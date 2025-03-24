---
title: 自制ECS库-MyECS
date: 2025-3-24
updated: 2024-3-24
permalink: articles/Flaaax/MyECS/
categories: Flaaax
tags: [ECS, 游戏开发, C++]
---

## 0.前言
因受到entt启发，以学习为目的，我仿照entt制作了一个纯头文件ecs库，目前许多功能还在实验中。其大部分接口与逻辑与entt相同，但在底层实现上一些差别。该博客主要是对ECS的介绍，对MyECS库的介绍并附带一些对entt的一些思考之类的。最后本人水平知识有限，请大佬轻喷（doge），非常欢迎大家来交流学习。


**github网址 ↓**

MyEcs:  &emsp;https://github.com/Flaaax/MyECS

entt:   &emsp;&emsp;https://github.com/skypjack/entt

<!-- More -->

## 1.什么是ECS？
*ECS*是一种游戏架构，全称*Entity Component System*（实体组件系统），通常用于游戏开发，并显著区别于经典的 *OOP*(*面向对象编程*) 架构。其定义分为三个部分Entity，Component和System。
#### 1.Entity
每个*Entity*仅为一个**唯一标识符** ，通常是一个无符号整数，不包含逻辑或者额外的东西，而不是像 *OOP Style* 一样的有着繁杂接口，超多成员的基类。

在*OOP*中，一个*Entity*可能长这样：
```cpp
class Entity{
protected:
    int health;
    int damage;
    //100+ members...

public:
    virtual ~Entity(){}
    virtual void update(float dt){}
    virtual void handleEvent(const Event& e){}
    virtual void render(Window& w){}
    //1000+ virtual functions...
}
```
然而，*ECS*中的*Entity*仅作为一个标识符，看起来就简单多了：
```cpp
using Entity = size_t;          //unsigned integer
```
（当然，这只是个示例，实际上还要考虑版本控制等）

那么它的数据和逻辑放哪呢？在*ECS*，一个*Entity*会关联一些*Component*，它们负责存储数据，而*System*负责处理这些数据逻辑。请继续往下看。

 #### 2.Component
 *Component*是一些**仅存储数据**的容器，例如**位置**，**速度**，**血量**等。每个*Entity*都可以关联多种*Component*来为其添加属性，换言之，对于一类*Component*，每个*Entity*都唯一对应了这一类的一个实例（如果它们有关联的话）
 
 在*MyECS*的实现中，你不需要为*Components*做额外操作。直接定义它们，然后立刻使用。

 ```cpp
struct Position{         //无需任何修饰
    float x, y;
};

struct Health{
    int health = 0;
}

//example
void main(){
    using namespace myecs;
    Registry reg;
    entity e = reg.create();

    reg.emplace<Position>(e, 1.f, 2.f);       //直接使用
    reg.emplace<Health>(e, Health{10});       //Very OK

    Position& pos = reg.get<Position>(e);     //再次获取
    std::cout << "e's Position: " << pos.x << " " << pos.y << std::endl;
}
```

> 就像*entt*一样，我在*MyECS*舍弃了继承关系和一部分安全性，换来了非常大的便利。

为什么我们不像*OOP*一样把所有东西都放在一个类里呢？因为你可以选择一个*Entity*持有哪些*Components*，不再需要负担你不需要的东西。在*OOP*，很多时候你必须在基类添加过多数据成员，而它们很多都被子类浪费了。例如，大部分实体都需要“位置”和“渲染”，但不是所有都需要“生命”。*Component*的设计很好地解决了这一问题。

#### 3.System
有了数据，现在只需为它们添加逻辑。在*OOP*，这部分通过继承*Entity*的虚函数来处理，但在*ECS*这正是*System*做的事。一个*ECS*可以有多个*System*，比如下面这些：
```cpp
class PhysicsSystem{
public:
    void updatePosition(Registry& reg, float dt){
        for(entity e: reg.view<Position, Velocity>){
            reg.get<Position>(e) += reg.get<Velocity>(e) * dt;
            if(Acceleration* acc = reg.try_get<Acceleration>(e)){
                reg.get<Velocity>(e) += (*acc) * dt;
            }
        }
    }
}

class RenderSystem{
public:
    void render(Registry& reg, Window& window){
        for(entity e: reg.view<Sprite>){
            window.draw(reg.get<Sprite>(e));
        }
    }
}
```
在纯粹的*ECS*中，所有逻辑都应该交给*System*来做，这些*System*不持有任何数据，仅负责处理数据。如果你有额外需求，那么就多设计几个*System*。当然，*MyECS*库并不会在乎这一点，你仍可以在任何地方添加你的逻辑，只要确保你知道自己在做什么。


## 2.为什么选择ECS而不是OOP？
~~因为写着很爽（划掉~~

*ECS* 能够抛开繁琐的继承关系，将实体化为组件的组合，这样结构更清晰，对代码编写者更友好。事实上，这也是我选择学习*ECS*的一个重要原因。

其它原因也包括比*OOP*效率更高，缓存更友好等 （这在后面会解释）

总之，在某些场合下，*ECS*确实能比*OOP*发挥更大的优势


## 3.ECS的优缺点
#### 优点：
1.代码结构清晰，更好编写，别提多爽了

2.缓存命中率高，更好发挥cpu性能

#### 缺点：
1.适用范围有限。与许多知乎“大佬”的意见不同的是，并不是所有情况都应该使用*ECS*，*ECS*也绝无可能取代*OOP*。不适合的例子就如**卡牌游戏**，**UI系统**等，而适合的例子有类似《**Noita**》的复杂弹幕游戏。

2.学习难度略大，因为*ECS*的架构与传统*OOP*差别过大，需要一定努力来适应新的模式并抛弃以前的继承式思维，当然这点因人而异了。
***

总之，希望以上内容帮助你判断你是否需要学习*ECS*并在你的新游戏里实践它。

感觉这次也写的够多了，关于*MyECS*的介绍就放到下期吧（）   感谢阅读，最后再贴一遍网址：

MyEcs:  &emsp;https://github.com/Flaaax/MyECS

entt:   &emsp;&emsp;https://github.com/skypjack/entt
