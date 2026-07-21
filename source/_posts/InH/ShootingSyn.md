---
title: 联机射击中开火与命中的RPC同步流程
date: 2026-01-16
updated: 2026-01-16
permalink: articles/InH/NetworkProjectileSyncStory/
categories: InH
tags: ["Godot", "GDScript", "ENetMultiplayerPeer", "RPC", "网络同步", "客户端预测", "Hitscan"]
---

<div align="center">

![项目截图1](/articles/InH/NetworkProjectileSyncStory/1.png)

</div>


本文为针对联机射击游戏的实验探索。该项目用 `ENetMultiplayerPeer` 和 RPC 搭一个主机/客户端射击流程，把开火请求、模拟延迟、服务端计算、命中确认结合，并完整实现了过程可视化。

<!-- More -->

## 将开火变成 Transaction

<div align="center">

![项目截图2](/articles/InH/NetworkProjectileSyncStory/2.png)

</div>

联机射击游戏一帧画面背后可能有客户端输入、服务端收包、逻辑判定、广播表现、命中回包好几个阶段。这个实验里我先把每次开火变成一个 Transaction，核心就是生成一个 `tid`，后面所有 RPC 和 UI 更新都带着它走。

```gdscript
func _initiate_fire():
    var tid = Time.get_ticks_msec()
    var rot = gun_pivot.rotation
    var pos = muzzle.global_position
    var my_id = multiplayer.get_unique_id()

    Global.rpc("request_broadcast_trans", tid, 0, "主机A: 发送开火请求")

    _dispatch_fire_by_mode(tid, pos, rot, my_id)
```

注：这个 `tid` 只是调试编号。它让一次开火的请求、处理、广播、命中能被串回同一条链路，不会在连续开火时混在一起。



## 三种同步模式的区分

<div align="center">

![策略模式](/articles/InH/NetworkProjectileSyncStory/3.png)

</div>

<div align="center">

![武器](/articles/InH/NetworkProjectileSyncStory/4.png)

</div>

分别为哑客户端模式、客户端预测模式、射线检测模式。武器也做了两类：普通枪和火箭筒。延迟用 `latency_ms` 控制，所有 RPC 链路统一走 `delay_call()` 模拟。

```gdscript
enum Mode { NAIVE, CLIENT_PREDICTION, HITSCAN_INSTANT }
enum WeaponType { GUN, ROCKET }

var current_mode: Mode = Mode.NAIVE
var current_weapon: WeaponType = WeaponType.GUN
var latency_ms: float = 0.0

signal transaction_update(trans_id: int, stage: int, info: String)
signal hit_confirmed(trans_id: int)
```

这里我加了一个 `MIN_ANIM_TIME`。真实延迟为 0 时，UI 动画会瞬间结束，很难观察过程。

```gdscript
const MIN_ANIM_TIME = 0.15

func delay_call(duration_ms: float, callback: Callable):
    var ui_time = max(duration_ms / 1000.0, MIN_ANIM_TIME)
    await get_tree().create_timer(ui_time).timeout
    callback.call()
```

模式、武器、延迟都由主机同步，这样测试时看到的差异主要来自同步策略，而不是每台机器的局部状态。



## 三种同步模式的差异

同一次开火在三种模式下走的链路不同。

哑客户端模式什么都等服务端:

<div align="center">

![哑客户端模式](/articles/InH/NetworkProjectileSyncStory/2.png)

</div>

预测模式提前播放本地表现:

<div align="center">

![预测模式](/articles/InH/NetworkProjectileSyncStory/5.png)

</div>

射线检测模式提前给高速视觉反馈，但命中结果仍然等服务端射线判定:

<div align="center">

![射线检测模式](/articles/InH/NetworkProjectileSyncStory/6.png)

</div>

```gdscript
match Global.current_mode:
    Global.Mode.NAIVE:
        _send_fire_request_after_latency(tid, pos, rot, my_id)

    Global.Mode.CLIENT_PREDICTION:
        _play_shoot_anim()
        _spawn_visual_bullet_after_fire_delay(pos, rot, tid, SPEED_VISUAL_NORMAL)
        _send_fire_request_after_latency(tid, pos, rot, my_id)

    Global.Mode.HITSCAN_INSTANT:
        _play_shoot_anim()
        _spawn_visual_bullet_after_fire_delay(pos, rot, tid, SPEED_VISUAL_HITSCAN, true)
        _send_fire_request_after_latency(tid, pos, rot, my_id)
```

这里最能体现出手感和权威结果之间的冲突。玩家希望按下鼠标立刻有反馈，但命中结果应该由服务端确认。预测模式把视觉表现提前，命中逻辑仍然留给服务端。



## 服务端处理普通弹和射线检测

普通弹模式会在服务端生成逻辑子弹，由碰撞回调决定命中。射线检测模式直接用 `direct_space_state.intersect_ray()` 做一次射线检测，然后广播命中结果。两条路径都通过 `tid` 更新 UI，所以能清楚看到模式差异。

```gdscript
func _server_raycast_hit(pos: Vector2, rot: float) -> int:
    var space = get_world_2d().direct_space_state
    var query = PhysicsRayQueryParameters2D.create(
        pos,
        pos + Vector2.RIGHT.rotated(rot) * 2000)

    var result = space.intersect_ray(query)
    if result and result.collider is CharacterBody2D:
        return result.collider.name.to_int()
    return 0
```

这个阶段我主要处理了两个细节。第一，服务端逻辑和客户端视觉要分离；第二，预测模式下开枪方已经生成过视觉子弹，收到广播时需要跳过重复生成。



## 逻辑子弹和视觉子弹分开

`Bullet.gd` 里有一个重要字段 `is_server_logic`。服务端逻辑子弹隐藏 Sprite，开启碰撞；客户端视觉子弹显示 Sprite，关闭碰撞。这样画面表现不会直接决定命中。

```gdscript
if is_server_logic:
    $Sprite2D.visible = false
    monitoring = true
    collision_mask = 1
    collision_layer = 2
else:
    $Sprite2D.visible = true
    monitoring = false
    collision_mask = 0
    collision_layer = 0
```

火箭筒使用了波形轨迹。这里规则仍然很简单：直线位移加一个垂直方向的 `sin` 偏移，旋转方向用前进速度和波动速度合成。射线检测模式下也可以强制使用波形视觉，让高速反馈仍然有武器特征。

<div align="center">

![火箭筒预测模式](/articles/InH/NetworkProjectileSyncStory/7.png)

</div>

```gdscript
var linear_pos = spawn_pos + base_direction * speed * time_elapsed
var perp_dir = Vector2(-base_direction.y, base_direction.x)
var offset = perp_dir * sin(time_elapsed * frequency) * amplitude

global_position = linear_pos + offset
rotation = (base_direction * speed + perp_dir * cos(time_elapsed * frequency)).angle()
```

这一层的重点是谁负责结算，逻辑弹只在服务端参与碰撞，视觉弹只服务表现，两个对象用同一份位置、方向和 `tid` 对齐。

我在这个项目里最大的收获是调试工具要跟机制一起写。网络同步的问题经常不在某一行 RPC，而在于同步是否有错位出现。有了 Transaction 和阶段面板以后，模式差异就能被看见。
