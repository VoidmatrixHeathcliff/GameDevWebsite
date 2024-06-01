---
title: Boids集群算法浅析与实践
date: 2024-06-01
updated: 2024-06-01
permalink: articles/Voidmatrix/boids-algorithm/
categories: Voidmatrix
tags: [EasyX, 算法, C++]
---

> 鸟群算法 Boids是模拟鸟类群集行为的人工生命项目，由克雷格·雷诺兹（Craig Reynolds）于1986年开发。Boids 是涌现行为的典例，其复杂性来自于遵循一系列简单规则个体的相互作用。

Boids 通常用于计算机图形学，提供鸟群和其他生物（如鱼群）的逼真表现：

+ 《史丹利和史黛拉: 破冰》是第一部利用了 Boids 模型的动画片；
+ 《半条命》中，游戏结束时出现的类似鸟类的飞行生物就使用了该模型；
+ 《蝙蝠侠归来》电影中，蝙蝠群和成群的企鹅行进穿过哥谭市的街道时使用了该模型；
+ 《黑客帝国》/《重装前哨》中，章鱼外形的哨兵机器人的集群特效使用了该模型。

<div style="text-align:center">

![分离（左） - 聚集（中） - 对齐（右）](articles/Voidmatrix/boids-algorithm/231573721246935.png)

</div>

<!-- more -->

Boids 模型由三种规则描述：

1. **分离**：个体之间存在排斥趋向，自主移动避开群体拥挤处；
3. **聚集**：个体会朝向周围同伴所处的平均位置处移动；
2. **对齐**：个体会朝向周围同伴的平均移动方向运动。


> 下面将使用 EasyX 可视化该算法。

## 随机个体位置着色

```cpp
std::vector<Boid> boids(500);
for (Boid& b : boids)
{
    b.position.x = (float)(rand() % 1280);
    b.position.y = (float)(rand() % 720);
    b.color = RGB(rand() % 255, rand() % 255, rand() % 255);
}
```

<div style="text-align:center">

![随机个体位置着色](articles/Voidmatrix/boids-algorithm/403754021267101.png)

</div>

## 添加“聚集”规则

```cpp
Vector2D cohesion(const std::vector<Boid>& boids) 
{
    Vector2D center_of_mass(0, 0);
    int total_neighbors = 0;

    for (const Boid& b : boids)
    {
        float distance = (b.position - position).length();

        if (distance > 0 && distance < neighbour_distance)
        {
            center_of_mass = center_of_mass + b.position;
            total_neighbors++;
        }
    }

    if (total_neighbors > 0) 
    {
        center_of_mass = center_of_mass * (1.0f / total_neighbors);
        return (center_of_mass - position);
    }

    return Vector2D(0, 0);
}
```

<div style="text-align:center">

![聚集规则](articles/Voidmatrix/boids-algorithm/338824221259770.gif)

</div>

## 添加“分离”规则

```cpp
Vector2D separation(const std::vector<Boid>& boids) 
{
    Vector2D separation(0, 0);

    for (const Boid& b : boids)
    {
        float distance = (b.position - position).length();

        if (distance > 0 && distance < separation_distance) 
        {
            Vector2D diff = position - b.position;
            separation = separation + diff * (1.0f / distance);
        }
    }

    return separation;
}
```

<div style="text-align:center">

![聚集规则 + 分离规则](articles/Voidmatrix/boids-algorithm/294884321256325.gif)

</div>

## 添加“对齐”规则

```cpp
Vector2D alignment(const std::vector<Boid>& boids) 
{
    Vector2D avg_velocity(0, 0);
    int total_neighbors = 0;

    for (const Boid& b : boids)
    {
        float distance = (b.position - position).length();

        if (distance > 0 && distance < neighbour_distance) 
        {
            avg_velocity = avg_velocity + b.velocity;
            total_neighbors++;
        }
    }

    if (total_neighbors > 0) 
    {
        avg_velocity = avg_velocity * (1.0f / total_neighbors);
        return avg_velocity - velocity;
    }

    return Vector2D(0, 0);
}
```

<div style="text-align:center">

![聚集规则 + 分离规则 + 对齐规则](articles/Voidmatrix/boids-algorithm/101894421252079.gif)

</div>

## 完整代码

```cpp
#include <cmath>
#include <vector>
#include <iostream>
#include <graphics.h>

// 简单的二维向量
struct Vector2D 
{
    float x, y;

    Vector2D() = default;
    Vector2D(float _x, float _y) : x(_x), y(_y) {}

    Vector2D operator+(const Vector2D& other) const 
    {
        return Vector2D(x + other.x, y + other.y);
    }

    Vector2D operator-(const Vector2D& other) const 
    {
        return Vector2D(x - other.x, y - other.y);
    }

    Vector2D operator*(float scalar) const 
    {
        return Vector2D(x * scalar, y * scalar);
    }

    float length() const 
    {
        return std::sqrt(x * x + y * y);
    }

    void normalize() 
    {
        float len = length();
        x /= len;
        y /= len;
    }
};

// 集群单位
struct Boid 
{
    COLORREF color;
    Vector2D position;
    Vector2D velocity;

    // 聚集规则
    Vector2D cohesion(const std::vector<Boid>& boids) 
    {
        Vector2D center_of_mass(0, 0);
        int total_neighbors = 0;

        for (const Boid& b : boids)
        {
            float distance = (b.position - position).length();

            if (distance > 0 && distance < neighbour_distance)
            {
                center_of_mass = center_of_mass + b.position;
                total_neighbors++;
            }
        }

        if (total_neighbors > 0) 
        {
            center_of_mass = center_of_mass * (1.0f / total_neighbors);
            return (center_of_mass - position);
        }

        return Vector2D(0, 0);
    }

    // 分离规则
    Vector2D separation(const std::vector<Boid>& boids) 
    {
        Vector2D separation(0, 0);

        for (const Boid& b : boids)
        {
            float distance = (b.position - position).length();

            if (distance > 0 && distance < separation_distance) 
            {
                Vector2D diff = position - b.position;
                separation = separation + diff * (1.0f / distance);
            }
        }

        return separation;
    }

    // 对齐规则
    Vector2D alignment(const std::vector<Boid>& boids) 
    {
        Vector2D avg_velocity(0, 0);
        int total_neighbors = 0;

        for (const Boid& b : boids)
        {
            float distance = (b.position - position).length();

            if (distance > 0 && distance < neighbour_distance) 
            {
                avg_velocity = avg_velocity + b.velocity;
                total_neighbors++;
            }
        }

        if (total_neighbors > 0) 
        {
            avg_velocity = avg_velocity * (1.0f / total_neighbors);
            return avg_velocity - velocity;
        }

        return Vector2D(0, 0);
    }

    // 更新方法
    void update(const std::vector<Boid>& boids) 
    {
        Vector2D v1 = cohesion(boids);
        Vector2D v2 = separation(boids);
        Vector2D v3 = alignment(boids);

        // 根据规则权重调节最终行为
        v1 = v1 * cohesion_weight;
        v2 = v2 * separation_weight;
        v3 = v3 * align_weight;

        // 更新速度
        velocity = velocity + v1 + v2 + v3;

        // 限制速度
        float speed = velocity.length();
        if (speed > max_speed) velocity = velocity * (max_speed / speed);

        // 更新位置
        position = position + velocity;

        // 限制位置
        if (position.x < 0) position.x = 0;
        if (position.x > 1280) position.x = 1280;
        if (position.y < 0) position.y = 0;
        if (position.y > 720) position.y = 720;
    }

    float neighbour_distance = 100.0f;      // 判定为临近单位的距离
    float separation_distance = 50.0f;      // 临近单位的分离距离
    float cohesion_weight = 1.0f;           // "聚集" 规则强度
    float separation_weight = 1.0f;         // "分离" 规则强度 
    float align_weight = 1.0f;              // "对其" 规则强度
    float max_speed = 5.0f;                 // 集群最大速度
};

int main() 
{
    initgraph(1280, 720, EW_SHOWCONSOLE);
    BeginBatchDraw();

    // 初始化集群
    std::vector<Boid> boids(500);
    for (Boid& b : boids)
    {
        b.position.x = (float)(rand() % 1280);
        b.position.y = (float)(rand() % 720);
        b.color = RGB(rand() % 255, rand() % 255, rand() % 255);
    }

    // 循环模拟
    while (true) 
    {
        // 更新集群
        for (Boid& b : boids)
            b.update(boids);

        // 渲染集群
        cleardevice();
        for (Boid& b : boids)
        {
            setfillcolor(b.color);
            fillcircle((int)b.position.x, (int)b.position.y, 10);
        }
        FlushBatchDraw();

        Sleep(25);
    }

    return 0;
}
```

> 使用三角形等有指向性的元素，根据当前速度向量绘图模拟效果会更加清晰。

<div style="text-align:center">

\>\>\>  [在 Voidmatrix's Blog 上查看本文章](https://www.voidmatrix.work/articles/boids-algorithm/)  <<<

</div>