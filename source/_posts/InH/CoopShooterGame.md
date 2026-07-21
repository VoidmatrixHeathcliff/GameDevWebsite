---
title: 多人联机合作射击探索
date: 2026-03-24
update: 2026-03-24
permalink: articles/InH/TopDownCoopShooterRuntime/
categories: InH
tags: ["C++", "Lua", "ENet", "Raylib", "SDL2", "GLSL", "联机射击"]
---

<div align="center">

![1](/articles/InH/TopDownCoopShooterRuntime/5.png)

</div>

本篇整合了我在学习PBR渲染和服务器权威联机模式后所完成的俯视角联机游戏项目。项目所做的工程拆分为 C++ 承接渲染、输入、碰撞、网络传输和材质系统，Lua 负责大厅、配装、战斗、任务逻辑和关卡运行时。

制作时我最深刻的印象就是联机射击项目很容易被一堆请求和状态同步搞乱。后来我把 RPC 名称集中到协议表，服务端只相信自己的状态，客户端做表现和 HUD，任务导演只通过事件改关卡节奏。这样的代码更易于理解和维护。

<!-- More -->

## 划分 C++ Lua 职责边界

我把 C++ 当作运行时底座，它负责 ENet 收发、窗口和输入、碰撞世界、材质加载、shader uniform 上传；Lua 只面对网络事件和玩法状态。这样做以后，玩法迭代不需要频繁改 C++，底层也不会被任务影响。

这个边界一开始看起来只是工程习惯，真正写到联机逻辑时才发现它很关键。只要 C++ 不直接参与具体任务判断，Lua 侧就可以比较自由地调整大厅流程、武器数据和关卡规则；只要 Lua 不绕过运行时接口，底层渲染和网络也不会被玩法脚本拖乱。

```cpp
void NetworkManager::Update() {
    ENetEvent event;
    while (enet_host_service(m_host, &event, 0) > 0) {
        if (event.type == ENET_EVENT_TYPE_RECEIVE) {
            HandlePacket(event);
            enet_packet_destroy(event.packet);
        }
    }
}
```

Lua 侧看到的是从 ENet packet 拆出来的 RPC 名称和 payload。后面移动、开火、补给、门禁、任务导演都能按同一种方式运作。也就是说，底层只负责把消息送到正确的入口，具体的游戏含义再由 Lua 解释。这个处理方式让后面的协议表有了存在的必要。



## 协议表 RPC

在开发项目时如果开火、移动、门禁、补给、扫描、导演事件都走网络，如果每个模块自己写 `"ReqShoot"`、`"SyncPos"`，后面改名和排查都会很痛。所以我把协议名和 JSON 编解码写到了 `protocol.lua`。

我当时最想避免的是同一个 RPC 名称散落在多个文件里。联机项目里的错误很多时候不是算法错，而是某个字符串、某个字段名、某个 payload 格式在两端没有对上。

```lua
Protocol.rpc = {
    request_move = "ReqMove",
    sync_position = "SyncPos",
    force_position = "ForcePos",
    request_shoot = "ReqShoot",
    spawn_bullet = "SpawnBullet",
    director_event = "DirectorEvent",
    director_state = "DirectorState"
}

function Protocol.broadcastTable(net, key, payload)
    net:broadcastRPC(Protocol.rpc[key] or key, JSON.encode(payload or {}))
end
```

C++ 的 `NetworkManager` 只处理拆包和回调派发。它按分隔符拿到 RPC 名称，再把 payload 交给 Lua 注册过的回调。

```cpp
void NetworkManager::HandlePacket(ENetEvent& event) {
    std::string data(
        static_cast<const char*>(event.packet->data),
        event.packet->dataLength);

    size_t splitPos = data.find('|');
    std::string rpcName = data.substr(0, splitPos);
    std::string payload = data.substr(splitPos + 1);

    auto it = m_rpcRegistry.find(rpcName);
    if (it != m_rpcRegistry.end()) {
        it->second(event.peer->incomingPeerID, payload);
    }
}
```

这个拆法让调试更直接。抓到一个包时，我能先看 RPC 名，再看 JSON payload，最后回到 Lua 的注册点。协议表定下来以后，后面每个系统都只是在这张表上继续挂具体行为。这样移动和开火就不会各自长出一套网络写法。



## 服务端权威处理移动

<div align="center">

![2](/articles/InH/TopDownCoopShooterRuntime/6.png)

</div>

多人合作射击里，移动同步是所有交互的地基。客户端可以提交自己想去的位置，服务端会按时间差、速度预算和导航场重新计算，再广播权威位置。偏差过大时，服务端额外发 `force_position`，让客户端明确知道自己被校正。

移动这里我没有把客户端输入完全当成最终结果，而是把它当成一次请求。客户端负责手感和响应，服务端负责判断这个位置能不能成立。

```lua
Protocol.register(net, "request_move", function(senderID, payload)
    local data = Protocol.decode(payload)
    local playerState = serverPlayers[tostring(senderID)]

    local syncData, wasClamped = ServerAuthority.applyMove(
        playerState,
        data,
        serverNavField)

    syncData.id = tonumber(senderID)
    Protocol.broadcastTable(net, "sync_position", syncData)

    if wasClamped then
        Protocol.sendTable(net, senderID, "force_position", syncData)
    end
end)
```

`ServerAuthority.applyMove()` 里还有一层速度预算和导航碰撞。玩家倒地、死亡、蹲伏、移动方向这些状态都在服务端统一生成同步包。

```lua
local budget = math.max(
    moveSpeed * elapsed * ServerAuthority.SPEED_TOLERANCE,
    ServerAuthority.MIN_BUDGET)

local limitedX, limitedY, wasClamped =
    clampMoveDistance(playerState.x, playerState.y, requestedX, requestedY, budget)

local solvedX, solvedY = ActorMotor.move(
    navField,
    playerState.x,
    playerState.y,
    limitedX - playerState.x,
    limitedY - playerState.y,
    runtimeMotor.radius,
    runtimeMotor)
```

这里的核心是把联机项目里的空间状态固定在服务端。客户端仍然可以做平滑、镜头、粒子和 HUD，结算依据只从服务端来。

移动链路处理清楚以后，开火也可以按同样思路走。两者的区别只是移动返回位置，开火返回命中和表现。



## 开火请求只生成权威结果

<div align="center">

![3](/articles/InH/TopDownCoopShooterRuntime/3.png)

</div>

开火链路也按同样思路处理。客户端发送方向和武器信息，服务端检查玩家状态、弹药、射击冷却和命中，再广播子弹表现。命中敌人后，服务端扣血并上报任务导演。

这里我比较在意的是“开火”和“命中”不要混成一个客户端结论。客户端可以告诉服务端自己按下了开火键，但它不能直接决定敌人已经掉血。

```lua
Protocol.register(net, "request_shoot", function(senderID, payload)
    local data = Protocol.decode(payload)
    local playerState = serverPlayers[tostring(senderID)]
    if not canShoot(playerState) then
        return
    end

    local shot = CombatValidator.registerShot(
        authorizedShots,
        senderID,
        data.x,
        data.y,
        data.dx,
        data.dy,
        data.dmg)

    local hitEnemyId, hitDistance = findNearestEnemyHit(shot.x, shot.y, shot.dx, shot.dy)
    Protocol.broadcastTable(net, "spawn_bullet", buildBulletPayload(shot, hitDistance))

    if hitEnemyId then
        applyEnemyShotDamage(senderID, hitEnemyId, serverEnemies[hitEnemyId], shot.dmg)
    end
end)
```

玩家看到的是即时反馈和弹道表现，服务端保存的是弹药、命中、敌人血量和任务压力。开火只是让客户端提交意图，服务端自己算结果。

这样写之后，枪声就可以自然变成任务导演的输入。战斗系统只负责一次攻击是否成立，至于这次攻击会不会引来下一波压力，则交给更上层的节奏系统处理。



## 任务导演将关卡节奏拆解

俯视角合作玩法里，敌人刷新、房间推进、终端解锁和撤离目标会互相影响。我把这些逻辑收进 `DirectorSystem`，让它根据房间进入、全员到达、威胁值、遭遇状态推动任务。

如果没有任务导演，这些逻辑很容易散在开门、刷怪、终端、撤离几个地方。短期看每个函数都能跑，长期看就是不知道某个关卡节奏到底被谁改了。

```lua
function DirectorSystem:update(dt)
    self.noise = clamp(self.noise - self.profile.threat.noiseDecay * dt, 0.0, self.profile.threat.maxNoise)

    self:updateRoomPresence()
    self:evaluateRoomTriggers()
    self:ensurePhaseStarted()
    self.playerPressure = self:calculatePlayerPressure()
    self:updatePacing(dt)

    if self.activeEncounter then
        self:maybeDispatchEvents(self.activeEncounter)
        self:maybeDispatchWaves(self.activeEncounter)
        self:maybeSpawnReinforcements(self.activeEncounter)
    end
end
```

服务端只给导演提供上下文函数：取玩家、刷敌人、开门、广播事件。导演本身不关心网络包怎么发。

```lua
director = DirectorSystem.new(profile, {
    getPlayers = function()
        return serverPlayers
    end,
    spawnEnemy = function(x, y, typeName, meta)
        return SpawnEnemy(x, y, typeName, meta)
    end,
    broadcastDirectorEvent = function(payload)
        Protocol.broadcastTable(net, "director_event", payload)
    end
})
```

这是我真正做项目时的取舍，战斗系统只管打中了谁，任务导演负责这次枪声让关卡压力发生了什么变化。

这一层拆出来以后，关卡推进就是一个持续更新的状态机。它可以接收战斗噪声，也可以根据房间状态主动发出事件。



## 材质和 HUD 服务玩法

<div align="center">

![4](/articles/InH/TopDownCoopShooterRuntime/4.png)

</div>

这个项目的视觉目标是服务俯视角射击里的信息表达。PBR 风格材质用 albedo、normal、roughness、AO 四张图；手电、环境灯、扫描波、墙体遮挡和实体高度统一上传到 shader。

视觉部分我没有把它当成独立展示效果来写，而是尽量让它回到玩法判断上。俯视角下玩家需要快速知道哪里危险、哪里被挡住、当前任务压力如何，材质和 HUD 都是围绕这些信息服务的。

```cpp
bool TextureManager::loadMaterial(const std::string& name, const std::string& basePath) {
    Material2D mat = {};
    mat.albedo = LoadTexture((basePath + "_Color.png").c_str());
    mat.normal = LoadTexture((basePath + "_NormalGL.png").c_str());
    mat.roughness = LoadTexture((basePath + "_Roughness.png").c_str());
    mat.ao = LoadTexture((basePath + "_AmbientOcclusion.png").c_str());

    if (mat.normal.id == 0) mat.normal = m_defaultNormal;
    if (mat.roughness.id == 0) mat.roughness = m_defaultRoughness;
    if (mat.ao.id == 0) mat.ao = m_defaultAO;

    m_materials[name] = mat;
    return true;
}
```

Lua 侧的 HUD 直接消费导演状态。目标、引导、补给、倒地、扫描冷却这些信息都来自服务端或本地运行时状态。

```lua
function ClientGame:onDirectorState(data)
    self.directorState.phase = data.phase or ""
    self.directorState.objective = data.objective or ""
    self.directorState.encounter = data.encounter or ""
    self.directorState.threat = tonumber(data.threat) or 0.0
    self.directorState.pacing = data.pacing or ""
end
```

材质和 HUD 都在服务玩法：手电/生物探测器可观察周围黑暗环境，导演告诉玩家当前目标。这一点和前面的服务端权威是连在一起的。
