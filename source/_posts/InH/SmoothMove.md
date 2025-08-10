---
title: 如何让2D角色丝滑的移动(C++/SDL2)开罐即食
date: 2025-08-10
update: 2025-08-10
permalink: articles/InH/SmoothMove/
categories: InH
tags: [学习心得]
---

<div align="center">

![](articles/InH/SmoothMove/cov.png)

</div>
本文你将会见到四种常见的让2D角色实现平滑移动的方式，用最简明的代码带来最佳的手感体验！

<!-- More --> 

## 前言

​   你是否觉得你的游戏角色移动的时候很死板，按住就走，松开就停，非常的死板没有活力，那就跟我一起从velocity的角度切入来看看有哪些有趣的算法可以优化我们的操作手感，让角色移动变的生动有趣~

#### 1.线性逼近（Lerp/Approach）

​	即每帧更新计算目标值与当前值的差，将差值按一定比例加回到当前值，使当前值逐步像目标值逼近，并在到达阈值的时候跳转到目标值避免无限逼近而影响状态机等外界对速度值的判断，这使人物速度启动拥有了缓入缓出的效果，避免了手感生硬。

```c++
target_velocity.x = move_dir.x * SPEED_RUN;
target_velocity.y = move_dir.y * SPEED_RUN;

auto approach = [&](float current, float target, float delta) -> float {
    float diff = target - current;
    if (std::abs(diff) < THRESHOLD) // 阈值
        return target;
    return current + diff * 6.0f * delta; // 数值越大越快
};
// 用法
velocity.x = approach(velocity.x, target_velocity.x, delta);
velocity.y = approach(velocity.y, target_velocity.y, delta);
```

​	所需要私有字段

```c++
    float SPEED_RUN = 200.0f;  //最大奔跑速度
    SDL_FPoint target_velocity = { 0 }; //目标速度
    const float THRESHOLD = 15.0f; //速度阈值，指接近这个数值直接跳转目标值
```

#### 2.指数平滑（Exponential Smoothing）

​	即每帧更新计算目标值与当前值的差，然后计算平滑因子factor，其原理是当低帧率时delta较大，factor也会较大，从而让后序计算更快的接近目标值，当高帧率时delta较小，factor也会较小，从而让后序计算更慢的接近目标值，但是因为一个是低帧一个是高帧本身刷新速度不一样，两项一结合就得到了此不受帧率影响的平滑移动的小算法，经过测试手感较优且稳定。

```c++
target_velocity.x = move_dir.x * SPEED_RUN;
target_velocity.y = move_dir.y * SPEED_RUN;

auto exp_smooth = [&](float current, float target) -> float {
    float diff = target - current;
    if (std::abs(diff) < THRESHOLD)
        return target;
    float factor = 1.0f - std::exp(-smoothing * delta);
    return current + diff * factor;
    };
// 用法
velocity.x = exp_smooth(velocity.x, target_velocity.x);
velocity.y = exp_smooth(velocity.y, target_velocity.y);
```

​	所需要私有字段

```c++
    float SPEED_RUN = 200.0f;  //最大奔跑速度
    SDL_FPoint target_velocity = { 0 }; //目标速度
    const float THRESHOLD = 15.0f; //速度阈值，指接近这个数值直接跳转目标值
    float smoothing = 6.0f; //控制收敛速度
```

#### 3.临界阻尼弹簧（SmoothDamp）

​	即每帧更新计算目标值与当前值的差，计算omega类似弹簧振动的角速度，x为单位时间的衰减量，之后通过多项式模拟exp指数来决定阻尼衰减速度，temp把当前位置偏差和当前速度结合成一个“预测的位移”,更新velRef时 ：(velRef - omega * temp) 是做一个速度衰减，(* exp)应用阻尼系数，逐渐趋近 0。(target + (change + temp) * exp)表示把衰减后的位移加到目标上。

```c++
target_velocity.x = move_dir.x * SPEED_RUN;
target_velocity.y = move_dir.y * SPEED_RUN;

auto smooth_damp = [&](float current, float target, float& velRef) -> float {
    // 阈值判断：如果足够接近，直接返回目标值并清零速度
    if (std::abs(target - current) < THRESHOLD) {
        velRef = 0.0f;
        return target;
    }
    float omega = 10.0f / smoothTime;
    float x = omega * delta;
    // 多项式近似 e^(-omega * delta)
    float exp = 1.0f / (1.0f + x + 0.48f * x * x + 0.235f * x * x * x);
    
    float change = current - target;
    float temp = (velRef + omega * change) * delta;
    velRef = (velRef - omega * temp) * exp;
    return target + (change + temp) * exp;
    };
// 用法
velocity.x = smooth_damp(velocity.x, target_velocity.x, velHelperX);
velocity.y = smooth_damp(velocity.y, target_velocity.y, velHelperY);
```

​	所需要私有字段

```c++
    float SPEED_RUN = 200.0f;  //最大奔跑速度
    SDL_FPoint target_velocity = { 0 }; //目标速度
    const float THRESHOLD = 15.0f; //速度阈值，指接近这个数值直接跳转目标值  

	float smoothTime = 0.2f;  //到达目标的大致时间（秒）
    float velHelperX = 0.0f;  //X轴内部阻尼速度
    float velHelperY = 0.0f;  //Y轴内部阻尼速度
```

#### 4.缓动（Smoothstep ）

​	t被传给 smoothstep，变成平滑的插值权重，smoothstep会根据 t 输出一个非线性缓动比例，起始和结束时速度较缓，中间较快，避免突变，这个平滑比例再用来线性插值当前值和目标值。

```c++
target_velocity.x = move_dir.x * SPEED_RUN;
target_velocity.y = move_dir.y * SPEED_RUN;

auto smoothstep = [&](float t) -> float {
    return t * t * (3.0f - 2.0f * t);
    };

auto ease_to = [&](float current, float target, float t) -> float {
    if (std::abs(current) < THRESHOLD)
        return target;
    float s = smoothstep(std::clamp(t, 0.0f, 1.0f));
    return current + (target - current) * s;
    };

// 用法
if (can_move) {
    velocity.x = ease_to(velocity.x, target_velocity.x, 0.1f);
    velocity.y = ease_to(velocity.y, target_velocity.y, 0.1f);//t为归一化进度系数
}
```

​	所需要私有字段

```c++
    float SPEED_RUN = 200.0f;  //最大奔跑速度
    SDL_FPoint target_velocity = { 0 }; //目标速度
    const float THRESHOLD = 15.0f; //速度阈值，指接近这个数值直接跳转目标值 
```

## 总结：

​	指数平滑 和 临界阻尼弹簧，这种基于指数衰减的算法，让2D角色获得了更自然、流畅且帧率无关的较好手感也比较符合我对游戏操作的直觉，所以给这两个小算法打满分！！，大家也快去试验一下吧！

