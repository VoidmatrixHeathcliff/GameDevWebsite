---
title: 血液喷溅特效模拟
date: 2025-09-27
update: 2025-09-27
permalink: articles/InH/GodotBloodMaterialStory/
categories: InH
tags: ["Godot", "GDScript", "SpriteFrames", "ShaderMaterial", "屏幕空间", "贴花", "特效系统"]
---

<div align="center">

![1](/articles/InH/GodotBloodMaterialStory/1.png)

</div>

这篇文章旨在我对于《武士零》血液特效的细节探索。项目从角色受击生成血液贴花继续推进，重点在受击方向、喷射节奏、血滴飞行、溶解、落点贴花和场景材质融合。

迭代到当前版本时，我觉得血滴飞出去以后要有重量感，消失时不能像突然删节点，落到场景里的痕迹要和背景有关系，多个血迹还要能各自保存喷射方向。

<!-- More -->

## 如何产生血液特效

<div align="center">

![2](/articles/InH/GodotBloodMaterialStory/2.png)

</div>

这个项目最核心的就是一次受击事件怎样一路传到最终画面。角色只负责产生受击方向和喷射节奏，管理器决定生成哪类血滴，血滴节点负责飞行和溶解，并在飞行过程中抛出快照，最后贴花节点把这个快照压到场景画面里。

这条链路可以概括成：

```text
角色受击 -> 计算喷射方向
BloodFXManager -> 选择血滴类型
DissolvingSprite -> 飞行、溶解、生成快照
BloodDecal -> 独立材质、记录喷射角度
decal_composite.gdshader -> 根据背景亮度融合
```

<div align="center">

![3](/articles/InH/GodotBloodMaterialStory/3.png)

</div>

这样拆的好处是生命周期清楚。临时血滴可以自然消失，贴花可以长期留在 `DecalContainer`，角色脚本也不会被材质参数和贴花管理污染。



## 受击事件只输出方向和节奏

角色脚本是入口。它继承 `Area2D`，通过鼠标点击触发受击，状态从 `IDLE` 切到 `HURT`，播放受击动画，同时启动 `BloodSpawnTimer`。角色只保存受击方向、击退速度和喷血频率。

```gdscript
func take_damage(mouse_pos: Vector2) -> void:
    _current_state = State.HURT
    animated_sprite.play("hurt")
    blood_spawn_timer.start()

    var direction = (global_position - mouse_pos).normalized()
    if direction.is_zero_approx():
        direction = Vector2.UP

    _last_hit_direction = direction
    _velocity = direction * knockback_power
```

喷血时再在受击方向上加一个小随机角度。这里我控制在正负 15 度：范围太小会像一条直线，范围太大又会丢掉命中方向。

```gdscript
func _on_blood_spawn_timer_timeout() -> void:
    var base_angle = _last_hit_direction.angle()
    var random_offset = deg_to_rad(randf_range(-15, 15))
    var final_direction = Vector2.from_angle(base_angle + random_offset)

    var spawn_position = global_position + final_direction * 100
    blood_fx_manager.spawn_effect(spawn_position, final_direction, blood_gravity_scale)
```

这里解决的是角色只告诉外部血液贴图从哪里喷、往哪里喷、重力比例是多少。



## 管理器负责生成策略

<div align="center">

![4](/articles/InH/GodotBloodMaterialStory/4.png)

</div>

`BloodFXManager` 是调度层。它持有动画血滴、溶解血滴、固定角度血滴和贴花场景。所有持久贴花最后都进入 `DecalContainer`，这样后面做数量上限、清理、对象池都能从一个容器处理。

生成策略用 `_spawn_counter` 做简单分流：每三次出一个动画粒子，其余走静态或溶解 Sprite。动画粒子适合表现一团血的形状变化，静态高速血滴适合表现甩出去的长距离喷溅。

```gdscript
func spawn_effect(spawn_position: Vector2, target_direction: Vector2, gravity_scale: float):
    _spawn_counter += 1

    var particle = _pick_particle_scene(_spawn_counter).instantiate()
    add_child(particle)

    particle.global_position = spawn_position
    particle.request_stamp.connect(_on_particle_request_stamp)
    particle.launch_and_dissolve(target_direction, decal_speed, decal_dissolve_duration, gravity_scale)
```

方向动画选择也放在管理器里。角色只传 `Vector2`，管理器根据角度选择最接近的一组 SpriteFrames。这样美术资源和角色状态机就解耦了。

<div align="center">

![7](/articles/InH/GodotBloodMaterialStory/7.png)

</div>

```gdscript
func _get_animation_name_for_direction(direction: Vector2) -> StringName:
    var lookup_angle = abs(rad_to_deg(direction.angle()))
    var best_key = "0"

    for key in ANIMATION_MAP.keys():
        if key.is_valid_int() and abs(lookup_angle - key.to_int()) < abs(lookup_angle - best_key.to_int()):
            best_key = key

    return ANIMATION_MAP[best_key]
```



## 血滴节点负责飞行、溶解和快照

`DissolvingSprite.gd` 是临时血滴的基类。它有速度、重力、溶解时间、贴花检测频率和贴花概率。飞行阶段每帧叠加重力，位置按速度推进，静态血滴根据速度方向旋转。

```gdscript
func _process(delta: float):
    velocity += Vector2.DOWN * gravity_scale * delta
    global_position += velocity * delta

    if not use_animation:
        rotation = velocity.angle() + fixed_angle_offset
```

溶解使用独立 `ShaderMaterial`。每个血滴在 `_ready()` 时创建自己的材质实例，然后用 Tween 推 `progress`。这样多个血滴同时存在时，溶解进度不会互相覆盖。

```gdscript
func launch_and_dissolve(direction: Vector2, speed: float, duration: float, gravity: float):
    velocity = direction * speed
    gravity_scale = gravity

    var mat := _get_material()
    _tween = create_tween().set_trans(Tween.TRANS_SINE)
    _tween.tween_property(mat, "shader_parameter/progress", 1.0, duration)
    _tween.tween_callback(queue_free)

    _stamp_timer.start()
```

这里最关键的是“快照”。血滴本身会销毁，但它可以把当前 transform、贴图、材质进度、翻转状态和运动方向打包发出去。管理器收到以后生成独立贴花。

```gdscript
func _create_stamp_snapshot():
    var live_material := _get_material()
    var stamp_material := live_material.duplicate() as ShaderMaterial

    var snapshot_data = {
        "transform": global_transform,
        "texture": _get_current_texture(),
        "material": stamp_material,
        "flip_v": _get_current_flip_v(),
        "direction": velocity.normalized()
    }

    request_stamp.emit(snapshot_data)
```

这个设计解决了生命周期冲突：临时血滴负责动态效果，持久贴花负责环境痕迹。



## 贴花需要独立材质保存喷射方向

管理器收到快照后，实例化一个 `Sprite2D` 贴花，把 transform、texture、flip 状态恢复出来。然后给这个贴花复制一份材质，并写入自己的 `spray_angle`。

```gdscript
func _on_particle_request_stamp(snapshot_data: Dictionary):
    var decal = blood_decal_scene.instantiate() as Sprite2D

    decal.global_transform = snapshot_data["transform"]
    decal.texture = snapshot_data["texture"]
    decal.flip_v = snapshot_data.get("flip_v", false)

    var mat = decal.material.duplicate(true) as ShaderMaterial
    mat.set_shader_parameter("spray_angle", rad_to_deg(snapshot_data["direction"].angle()))
    decal.material = mat

    decal_container.add_child(decal)
```

我一开始没有足够重视材质实例化。多个 Sprite 共用同一份 `ShaderMaterial` 时，后写入的 `spray_angle` 会影响前面的血迹，画面上会出现方向统一的问题。每个贴花独立材质后，喷射方向就能被保存下来。



## 溶解和屏幕空间融合

<div align="center">

![5](/articles/InH/GodotBloodMaterialStory/5.png)

</div>

飞行血滴的消失使用一个很小的 `canvas_item` Shader。它读取噪声纹理，根据 `progress` 和 `softness` 用 `smoothstep` 计算 alpha。

贴花如果只做普通 alpha 混合，会像浮在屏幕上的红色贴纸。`decal_composite.gdshader` 里我读取 `SCREEN_TEXTURE`，用背景亮度参与遮蔽、扭曲、暗部裁剪等风格混合，详细带注释的shader参考如下：

<div align="center">

![6](/articles/InH/GodotBloodMaterialStory/6.png)

</div>

这套材质里贴花知道自己从哪里来。方向从角色受击一路传到血滴速度，再进入快照，最后写进贴花 Shader。一个小参数贯穿了事件层、运动层和材质层。

## 总结

这个专题展示的是血液喷溅特效系统。角色状态、特效生成、血滴生命周期、持久贴花和屏幕空间材质各自有职责。受击有方向，喷射有节奏，飞行有重力，消失有材质过程，落点能留下独立痕迹，痕迹还能根据背景产生融合感，我觉得是一个不错的模拟示例。

