---
title: 四叉树学习笔记
date: 2025-01-10
updated: 2025-01-10
permalink: articles/XZDXRZ/quadtree_nodes/
categories: 数据结构
tags: [数据结构, C++, 游戏开发]
---

# What is a Quadtree and why are we using it

Within the context of game development, we often encounter situations where we need to detect whether two objects has collided. Throughing implementing quadtree, we can reduce the number of objects that need to be checked for collisions by focusing only on objects within the same or neighboring quadrants.

Imaging that we have a 2D plane with several game objects on it. We define our game object in this class.

```cpp
class Object
{
    public:
        Point position;
        std::string name;
        Object(Point _position, std::string _name): position(_position), name(_name) {}
};
```

Where `Point` is defined through this,

```cpp
class Point
{
    public:
        int x, y;
        Point(int _x, int _y): x(_x), y(_y) {}
};
```

A quadtree is a tree structure in which each node has exactly four child node. Each node within the quadtree can be defined through this way:

```cpp
class Quadtree
{
    private:
        // Define the area where the node is allocated
        Point top_left, bottom_right;
        // Declare four child nodes
        Quadtree *top_left_child, *top_right_child, *bottom_left_child, *bottom_right_child;
        // The object stored in this region
        Object *object;

        // whether this node has any children
        bool hasChildren();
    public:
        Quadtree(Point _top_left, Point _bottom_right):\
            top_left(_top_left), bottom_right(_bottom_right),\
            object(NULL), top_left_child(NULL), top_right_child(NULL),\
            bottom_left_child(NULL), bottom_right_child(NULL) {}

        ~Quadtree();

        // insert an object into the quadtree
        void insert(Object *object);
        // search for an object in the quadtree
        Object* search(Point point);

        // whether the point is in a specific child node
        bool in_top_left_child(Point point);
        bool in_top_right_child(Point point);
        bool in_bottom_left_child(Point point);
        bool in_bottom_right_child(Point point);

        // whether the point is in the boundary of the node
        bool in_boundary(Point point);
};
```

We have created an immense class here. Don't worry, I'll eventually explain them in detail.

To define a rectangle region, the coordinate of its top left and bottom right corner is required. We use two points to define the area where the node is allocated to.

To define the four child nodes, we use four pointers to point to the child nodes.

Similarly, to define the object stored in this region, we use a pointer to point to the object.

Here is the implementation of some assisting functions.

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

BTW, we create four children at same time. Therefore, when we need to determine if a node has children, we can check any of its child and see if it exist.

And the destructor to free up the memory.

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

Through recursively partition a 2-dimensional plane into four quadrants, we can build a quadtree quite easily.

```cpp
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
```

To insert an object into the quadtree, we should ensure that the object is within the boundary of the quadtree.

Additionaly, if the object is already in a unit node of the quadtree, we simply store the object into this node.

If the node has not been partitioned yet, we need to partition the node and create four children. Then we insert the object into the corresponding child.

To search for an object in the quadtree, we recursively search for the region where the object is located.

```cpp
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
```

The logic for searching is simple: When we insert an object into the tree, we keep partitioning the tree until we reach a unit node. Therefore, when we search for an object in the tree, we keep searching until we reach a unit node.

Let us test the quadtree with some instances.

```cpp
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

The result should be:

```bash
Test Insert and Search:
Search (1, 1): Object 1
Search (2, 5): Object 2
Search (7, 7): Object 3
Search (3, 3): NULL
```

Finally, the complete code is,

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
