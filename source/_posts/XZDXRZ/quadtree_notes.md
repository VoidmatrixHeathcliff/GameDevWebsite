---
title: 四叉树学习笔记
date: 2025-01-10
updated: 2025-01-10
permalink: articles/XZDXRZ/quadtree_nodes/
categories: XZDXRZ
author: XZDXRZ
tags: [数据结构, C++, 游戏开发]
---

# 什么是四叉树，我们为什么使用它

在游戏开发的背景下，我们经常会遇到需要检测两个对象是否发生碰撞的情况。通过实现四叉树，我们可以减少需要检查碰撞的对象数量，只关注同一象限或相邻象限内的对象。

<!--more-->

想象一下，我们有一个包含多个游戏对象的二维平面。我们在这个类中定义我们的游戏对象。

```cpp
class Object
{
    public:
        Point position;
        std::string name;
        Object(Point _position, std::string _name): position(_position), name(_name) {}
};
```

其中 `Point` 类定义如下：

```cpp
class Point
{
    public:
        int x, y;
        Point(int _x, int _y): x(_x), y(_y) {}
};
```

四叉树是一种树结构，其中每个节点都有四个子节点。每个节点可以通过以下方式定义：

```cpp
class Quadtree
{
    private:
        // 定义节点分配的区域
        Point top_left, bottom_right;
        // 声明四个子节点
        Quadtree *top_left_child, *top_right_child, *bottom_left_child, *bottom_right_child;
        // 存储在该区域的对象
        Object *object;

        // 判断该节点是否有子节点
        bool hasChildren();
    public:
        Quadtree(Point _top_left, Point _bottom_right):\
            top_left(_top_left), bottom_right(_bottom_right),\
            object(NULL), top_left_child(NULL), top_right_child(NULL),\
            bottom_left_child(NULL), bottom_right_child(NULL) {}

        ~Quadtree();

        // 向四叉树中插入对象
        void insert(Object *object);
        // 在四叉树中搜索对象
        Object* search(Point point);

        // 判断点是否在特定子节点中
        bool in_top_left_child(Point point);
        bool in_top_right_child(Point point);
        bool in_bottom_left_child(Point point);
        bool in_bottom_right_child(Point point);

        // 判断点是否在节点的边界内
        bool in_boundary(Point point);
};
```

我们在这里创建了一个庞大的类。别担心，我会逐步详细解释它们。

要定义一个矩形区域，需要其左上角和右下角的坐标。我们使用两个点来定义节点分配的区域。

为了定义四个子节点，我们使用四个指针指向子节点。

同样，为了定义存储在该区域的对象，我们使用一个指针指向该对象。

以下是一些辅助函数的实现。

```cpp
bool Quadtree::hasChildren()
{
    return top_left_child != NULL;
}

bool Quadtree::in_top_left_child(Point point)
{
    return point.x <= top_left_child->bottom_right.x && point.y <= top_left_child->bottom_right.y;
}

bool Quadtree::in_top_right_child(Point point)
{
    return point.x > top_left_child->bottom_right.x && point.x <= top_right_child->bottom_right.x && point.y <= top_right_child->bottom_right.y;
}

bool Quadtree::in_bottom_left_child(Point point)
{
    return point.x <= bottom_left_child->bottom_right.x && point.y > top_left_child->bottom_right.y && point.y <= bottom_left_child->bottom_right.y;
}

bool Quadtree::in_bottom_right_child(Point point)
{
    return point.x > top_left_child->bottom_right.x && point.y > top_left_child->bottom_right.y && point.x <= bottom_right_child->bottom_right.x && point.y <= bottom_right_child->bottom_right.y;
}

bool Quadtree::in_boundary(Point point)
{
    return point.x >= top_left.x && point.x <= bottom_right.x && point.y >= top_left.y && point.y <= bottom_right.y;
}
```

顺便说一下，因为我们是同时创建四个子节点，所以，当我们需要确定一个节点是否有子节点时，我们可以检查它的任意一个子节点是否存在。

以及创建一个析构函数用于释放内存。

```cpp
Quadtree::~Quadtree()
{
    delete top_left_child;
    delete top_right_child;
    delete bottom_left_child;
    delete bottom_right_child;
    delete object;
}
```

通过递归地将二维平面划分为四个象限，我们可以很容易地构建一个四叉树。

```cpp
void Quadtree::insert(Object* object)
{
    // 如果对象不在四叉树的边界内
    if (!in_boundary(object->position))
        return;

    // 如果对象在四叉树的单元节点中
    // 即节点不能再被划分
    if (top_left.x == bottom_right.x && top_left.y == bottom_right.y)
    {
        if (this->object == NULL)
            this->object = object;
        return;
    }

    // 如果该节点尚未被划分
    if (!hasChildren())
    {
        // 划分节点并创建4个子节点
        int x = (top_left.x + bottom_right.x) / 2;
        int y = (top_left.y + bottom_right.y) / 2;
        top_left_child = new Quadtree(top_left, Point(x, y));
        top_right_child = new Quadtree(Point(x + 1, top_left.y), Point(bottom_right.x, y));
        bottom_left_child = new Quadtree(Point(top_left.x, y + 1), Point(x, bottom_right.y));
        bottom_right_child = new Quadtree(Point(x + 1, y + 1), bottom_right);
    }

    // 将对象插入到相应的子节点中
    if (in_top_left_child(object->position))
        top_left_child->insert(object);
    else if (in_top_right_child(object->position))
        top_right_child->insert(object);
    else if (in_bottom_left_child(object->position))
        bottom_left_child->insert(object);
    else
        bottom_right_child->insert(object);
}
```

要将对象插入四叉树，我们应确保对象在四叉树的边界内。

此外，如果对象已经在四叉树的单元节点中，我们只需将对象存储到该节点中。

如果节点尚未被划分，我们需要划分节点并创建四个子节点。然后我们将对象插入到相应的子节点中。

要在四叉树中搜索对象，我们递归地搜索对象所在的区域。

```cpp
Object* Quadtree::search(Point point)
{
    // 如果点不在四叉树的边界内
    if (!in_boundary(point))
        return NULL;

    // 如果点在四叉树的单元节点中
    // 即节点不能再被划分
    if (top_left.x == bottom_right.x && top_left.y == bottom_right.y)
        return object;

    // 如果该节点已经被划分
    if (hasChildren())
    {
        if (in_top_left_child(point))
            return top_left_child->search(point);
        else if (in_top_right_child(point))
            return top_right_child->search(point);
        else if (in_bottom_left_child(point))
            return bottom_left_child->search(point);
        else
            return bottom_right_child->search(point);
    }

    return NULL;
}
```

搜索的逻辑很简单：当我们将对象插入树中时，我们不断划分树，直到到达单元节点。因此，当我们在树中搜索对象时，我们不断搜索，直到到达单元节点。

让我们用一些实例来测试四叉树。

```cpp
int main()
{
    // 构建一个8x8的四叉树
    Quadtree qt(Point(0, 0), Point(8, 8));

    // 创建并插入三个对象到四叉树中
    Object obj1(Point(1, 1), "Object 1");
    Object obj2(Point(2, 5), "Object 2");
    Object obj3(Point(7, 7), "Object 3");

    qt.insert(&obj1);
    qt.insert(&obj2);
    qt.insert(&obj3);

    // 在四叉树中搜索对象
    Object* result1 = qt.search(Point(1, 1));
    Object* result2 = qt.search(Point(2, 5));
    Object* result3 = qt.search(Point(7, 7));
    // 搜索一个不在四叉树中的点
    Object* result4 = qt.search(Point(3, 3));

    std::cout << "Test Insert and Search:" << std::endl;
    std::cout << "Search (1, 1): " << (result1 != NULL ? result1->name : "NULL") << std::endl;
    std::cout << "Search (2, 5): " << (result2 != NULL ? result2->name : "NULL") << std::endl;
    std::cout << "Search (7, 7): " << (result3 != NULL ? result3->name : "NULL") << std::endl;
    std::cout << "Search (3, 3): " << (result4 != NULL ? result4->name : "NULL") << std::endl;
    return 0;
}
```

结果如下：

```bash
Test Insert and Search:
Search (1, 1): Object 1
Search (2, 5): Object 2
Search (7, 7): Object 3
Search (3, 3): NULL
```

完整代码如下

```cpp
#include <iostream>
#include <string>

class Point
{
    public:
        int x, y;
        Point(int _x, int _y): x(_x), y(_y) {}
};

class Object
{
    public:
        Point position;
        std::string name;
        Object(Point _position, std::string _name): position(_position), name(_name) {}
};

class Quadtree
{
    private:
        Point top_left, bottom_right;
        Quadtree *top_left_child, *top_right_child, *bottom_left_child, *bottom_right_child;
        Object *object;

        bool hasChildren();
    public:
        Quadtree(Point _top_left, Point _bottom_right):\
            top_left(_top_left), bottom_right(_bottom_right),\
            object(NULL), top_left_child(NULL), top_right_child(NULL),\
            bottom_left_child(NULL), bottom_right_child(NULL) {}

        ~Quadtree();

        void insert(Object *object);
        Object* search(Point point);

        bool in_top_left_child(Point point);
        bool in_top_right_child(Point point);
        bool in_bottom_left_child(Point point);
        bool in_bottom_right_child(Point point);

        bool in_boundary(Point point);
};

bool Quadtree::hasChildren()
{
    return top_left_child != NULL;
}

bool Quadtree::in_top_left_child(Point point)
{
    return point.x <= top_left_child->bottom_right.x && point.y <= top_left_child->bottom_right.y;
}

bool Quadtree::in_top_right_child(Point point)
{
    return point.x > top_left_child->bottom_right.x && point.x <= top_right_child->bottom_right.x && point.y <= top_right_child->bottom_right.y;
}

bool Quadtree::in_bottom_left_child(Point point)
{
    return point.x <= bottom_left_child->bottom_right.x && point.y > top_left_child->bottom_right.y && point.y <= bottom_left_child->bottom_right.y;
}

bool Quadtree::in_bottom_right_child(Point point)
{
    return point.x > top_left_child->bottom_right.x && point.y > top_left_child->bottom_right.y && point.x <= bottom_right_child->bottom_right.x && point.y <= bottom_right_child->bottom_right.y;
}

bool Quadtree::in_boundary(Point point)
{
    return point.x >= top_left.x && point.x <= bottom_right.x && point.y >= top_left.y && point.y <= bottom_right.y;
}

void Quadtree::insert(Object* object)
{
    // If the object is not in the boundary of the quadtree
    if (!in_boundary(object->position))
        return;

    // If the object is in a unit node of quadtree
    // that is, the node cannot be partitioned anymore
    if (top_left.x == bottom_right.x && top_left.y == bottom_right.y)
    {
        if (this->object == NULL)
            this->object = object;
        return;
    }

    // If this node has not been partitioned yet
    if (!hasChildren())
    {
        // Partition the node and create 4 children
        int x = (top_left.x + bottom_right.x) / 2;
        int y = (top_left.y + bottom_right.y) / 2;
        top_left_child = new Quadtree(top_left, Point(x, y));
        top_right_child = new Quadtree(Point(x + 1, top_left.y), Point(bottom_right.x, y));
        bottom_left_child = new Quadtree(Point(top_left.x, y + 1), Point(x, bottom_right.y));
        bottom_right_child = new Quadtree(Point(x + 1, y + 1), bottom_right);
    }

    // Insert the object into the corresponding child
    if (in_top_left_child(object->position))
        top_left_child->insert(object);
    else if (in_top_right_child(object->position))
        top_right_child->insert(object);
    else if (in_bottom_left_child(object->position))
        bottom_left_child->insert(object);
    else
        bottom_right_child->insert(object);
}

Object* Quadtree::search(Point point)
{
    // If the point is not in the boundary of the quadtree
    if (!in_boundary(point))
        return NULL;

    // If the point is in a unit node of quadtree
    // that is, the node cannot be partitioned anymore
    if (top_left.x == bottom_right.x && top_left.y == bottom_right.y)
        return object;

    // If this node has been partitioned
    if (hasChildren())
    {
        if (in_top_left_child(point))
            return top_left_child->search(point);
        else if (in_top_right_child(point))
            return top_right_child->search(point);
        else if (in_bottom_left_child(point))
            return bottom_left_child->search(point);
        else
            return bottom_right_child->search(point);
    }

    return NULL;
}

Quadtree::~Quadtree()
{
    delete top_left_child;
    delete top_right_child;
    delete bottom_left_child;
    delete bottom_right_child;
    delete object;
}

int main()
{
    // Build a 8x8 quadtree
    Quadtree qt(Point(0, 0), Point(8, 8));

    // Create and Insert three objects into the quadtree
    Object obj1(Point(1, 1), "Object 1");
    Object obj2(Point(2, 5), "Object 2");
    Object obj3(Point(7, 7), "Object 3");

    qt.insert(&obj1);
    qt.insert(&obj2);
    qt.insert(&obj3);

    // Search for the objects in the quadtree
    Object* result1 = qt.search(Point(1, 1));
    Object* result2 = qt.search(Point(2, 5));
    Object* result3 = qt.search(Point(7, 7));
    // search for a point that is not in the quadtree
    Object* result4 = qt.search(Point(3, 3));

    std::cout << "Test Insert and Search:" << std::endl;
    std::cout << "Search (1, 1): " << (result1 != NULL ? result1->name : "NULL") << std::endl;
    std::cout << "Search (2, 5): " << (result2 != NULL ? result2->name : "NULL") << std::endl;
    std::cout << "Search (7, 7): " << (result3 != NULL ? result3->name : "NULL") << std::endl;
    std::cout << "Search (3, 3): " << (result4 != NULL ? result4->name : "NULL") << std::endl;
    return 0;
}
```
