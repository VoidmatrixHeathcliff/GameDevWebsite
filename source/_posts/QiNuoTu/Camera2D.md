---
title: 自己设计一个游戏通用的2D摄像机累。Camera2D
date: 2024-06-4
updated: 2024-06-4
permalink: articles/QiNuoTu/Camera2D/
categories: QiNuoTu
tags: [教程,游戏开发]
---
在2D游戏开发中，摄像机是一个至关重要的组件，允许开发者控制玩家的视角，决定他们能看到游戏世界的哪些部分。这对于引导玩家的注意力和提供沉浸式体验非常重要，在战斗场景中聚焦于战斗区域；而在探索场景中扩大视野以展示更多的环境细节，跟随主要角色移动，确保玩家始终能够看到控制中的角色和周围的环境，将摄像机焦点对准可交互对象，用来提示玩家这些对象的存在和重要性，还可以用来实现特殊效果，如震动或动态缩放等，在下文中我们将会实现这样一个摄像机类用于自己的游戏。
![ICON](articles/QiNuoTu/Camera2D/1.gif)
<!-- More -->
<div align="center">
    <p align="center">
        <img src=",articles/QiNuoTu/icon.png" alt="logo" width="200">
    </p>
    <h1>琪诺兔</h1>
    <p>
        <a href="https://space.bilibili.com/69720374" target="_blank">关注我的哔哩哔哩走进我的生活</a>
        &nbsp;|&nbsp;
        <a href="https://github.com/QiNuoTu" target="_blank">关注我的GitHub获得我的代码</a>
    </p>
</div>

## 一个合格的摄像机应该具有以下功能。
    本人习惯已屏幕中心为基准处理数据，并且正向使用坐标，所有以下实现已本人使用舒适维主进行实现，我们来弄一些碎片吧。
- **视口控制：**设置视口大小适应不同的分辨率和屏幕尺寸。
```cpp
    void SetCameraCenter(short int Viewport_Width, short int Viewport_Height) {
        ViewportWidth = Viewport_Width;
        ViewportHeight = Viewport_Height;
        ViewportCenterX = ViewportWidth * 0.5f;
        ViewportCenterY = ViewportHeight * 0.5f;
    }
```
- **焦点跟随：**视点跟随焦点移动。
```cpp
    void SetTarget(float targetX, float targetY) {
        TargetX = targetX;
        TargetY = targetY;
    }
        void SmoothMoveToPosition(float smoothing = 0.5f) {
        CameraFocusX += (targetX - CameraFocusX) * smoothing;
        CameraFocusY += (targetY - CameraFocusY) * smoothing;
    }
```
- **坐标转换：**屏幕坐标与世界坐标之间转换，便于处理用户输入和游戏对象的位置。
```cpp
    void ScreenToWorld(float screenX, float screenY, float& worldX, float& worldY) const {
        worldX = CameraFocusX - (screenX - ViewportCenterX) / Zoom;
        worldY = CameraFocusY - (screenY - ViewportCenterY) / Zoom;
    }

    void WorldToScreen(float worldX, float worldY, float& screenX, float& screenY) const {
        screenX = ViewportCenterX + (CameraFocusX - worldX) * Zoom;
        screenY = ViewportCenterY + (CameraFocusY - worldY) * Zoom;
    }
```
- **缩放功能：**适应不同的游戏场景和提供不同的视觉体验。
```cpp
    float GetScale() const { return Zoom; }

    void SetScale(float zoom = 1)
    {
        Zoom = zoom;
    }
```
- **边界限制：**设置移动边界，防止摄像机移动到游戏世界之外。
```cpp
    void SetWorldSize(float Width, float Height) {
        WorldBoundaryLeft = -Width * 0.5f;
        WorldBoundaryTop = -Height * 0.5f;
        WorldBoundaryRight = Width * 0.5f;
        WorldBoundaryBottom = Height * 0.5f;
    }

    bool SetWorldBoundaries(float left, float top, float right, float bottom) {
        if (left < right && top < bottom) {
            WorldBoundaryLeft = left;
            WorldBoundaryTop = top;
            WorldBoundaryRight = right;
            WorldBoundaryBottom = bottom;
            return true;
        }
        return false;
    }

    void ViewportCheckBoundaries() {
        float scaledOffsetX = ViewportCenterX / Zoom;
        float scaledOffsetY = ViewportCenterY / Zoom;
        CameraFocusX = std::max(WorldBoundaryLeft + scaledOffsetX, std::min(CameraFocusX, WorldBoundaryRight - scaledOffsetX));
        CameraFocusY = std::max(WorldBoundaryTop + scaledOffsetY, std::min(CameraFocusY, WorldBoundaryBottom - scaledOffsetY));

        if (ViewportWidth / Zoom > WorldBoundaryRight - WorldBoundaryLeft) {
            CameraFocusX = (WorldBoundaryLeft + WorldBoundaryRight) * 0.5f;
        }
        if (ViewportHeight / Zoom > WorldBoundaryBottom - WorldBoundaryTop) {
            CameraFocusY = (WorldBoundaryTop + WorldBoundaryBottom) * 0.5f;
        }
    }
```
- **平滑过渡：**避免视角突变给玩家带来不适。
```cpp
    void SmoothMoveToPosition(float targetX, float targetY, float smoothing = 0.5f) {
        CameraFocusX += (targetX - CameraFocusX) * smoothing;
        CameraFocusY += (targetY - CameraFocusY) * smoothing;
    }

    void Scale(float zoom = 1){
        Zoom += zoom;
    }
```
- **抖动效果：**模拟冲击爆炸等效果来增强游戏的氛围和反馈。
```cpp
    void Shake(float intensityX = 5.5f, float intensityY = 5.5f) {
        std::uniform_real_distribution<float> disX(-intensityX, intensityX);
        std::uniform_real_distribution<float> disY(-intensityY, intensityY);
        CameraFocusX += disX(gen);
        CameraFocusY += disY(gen);
    }

    void ShakeCircle(float intensityX = 5.5f, float intensityY = 5.5f) {
        std::uniform_real_distribution<float> disX(-intensityX, intensityX);
        std::uniform_real_distribution<float> disY(-intensityY, intensityY);
        std::uniform_real_distribution<float> angle(-360.0f, 360.0f);
        float radian = angle(gen) / 360 * m_PI * 2;
        CameraFocusX += disX(gen) * std::cos(radian);
        CameraFocusY += disY(gen) * std::sin(radian);
    }
```
## 此刻Camera2D类的所有碎片都已经获得，我们把他拼起来吧。
```cpp
class Camera2D {
public:
    Camera2D(float Viewport_Width, float Viewport_Height,
        float World_Width, float World_Height,
        float FocusX = 0, float FocusY = 0) :
        ViewportCenterX(Viewport_Width * 0.5f),
        ViewportCenterY(Viewport_Height * 0.5f),
        ViewportWidth(Viewport_Width),
        ViewportHeight(Viewport_Height),
        WorldBoundaryLeft(-World_Width * 0.5),
        WorldBoundaryTop(-World_Height * 0.5),
        WorldBoundaryRight(World_Width * 0.5),
        WorldBoundaryBottom(World_Height * 0.5),
        TargetX(0),
        TargetY(0),
        CameraFocusX(FocusX),
        CameraFocusY(FocusY),
        Zoom(1){}

    ~Camera2D() = default;

    void ScreenToWorld(float screenX, float screenY, float& worldX, float& worldY) const {
        worldX = CameraFocusX - (screenX - ViewportCenterX) / Zoom;
        worldY = CameraFocusY - (screenY - ViewportCenterY) / Zoom;
    }

    void WorldToScreen(float worldX, float worldY, float& screenX, float& screenY) const {
        screenX = ViewportCenterX + (CameraFocusX - worldX) * Zoom;
        screenY = ViewportCenterY + (CameraFocusY - worldY) * Zoom;
    }

    float GetScale() const { return Zoom; }

    void SetScale(float zoom = 1)
    {
        Zoom = zoom;
    }

    void Scale(float zoom = 1)
    {
        Zoom += zoom;
    }

    void SmoothMoveToPosition(float targetX, float targetY, float smoothing = 0.5f) {
        CameraFocusX += (targetX - CameraFocusX) * smoothing;
        CameraFocusY += (targetY - CameraFocusY) * smoothing;
    }

    void SetTarget(float targetX, float targetY) {
        TargetX = targetX;
        TargetY = targetY;
    }

    void SmoothMoveToTarget(float smoothing = 0.5f) {
        CameraFocusX += (TargetX - CameraFocusX) * smoothing;
        CameraFocusY += (TargetY - CameraFocusY) * smoothing;
    }

    void Shake(float intensityX = 5.5f, float intensityY = 5.5f) {
        std::uniform_real_distribution<float> disX(-intensityX, intensityX);
        std::uniform_real_distribution<float> disY(-intensityY, intensityY);
        CameraFocusX += disX(gen);
        CameraFocusY += disY(gen);
    }

    void ShakeCircle(float intensityX = 5.5f, float intensityY = 5.5f) {
        std::uniform_real_distribution<float> disX(-intensityX, intensityX);
        std::uniform_real_distribution<float> disY(-intensityY, intensityY);
        std::uniform_real_distribution<float> angle(-360.0f, 360.0f);
        float radian = angle(gen) / 360 * 3.1415926535 * 2;
        CameraFocusX += disX(gen) * std::cos(radian);
        CameraFocusY += disY(gen) * std::sin(radian);
    }

    void SetCameraCenter(short int Viewport_Width, short int Viewport_Height) {
        ViewportWidth = Viewport_Width;
        ViewportHeight = Viewport_Height;
        ViewportCenterX = ViewportWidth * 0.5f;
        ViewportCenterY = ViewportHeight * 0.5f;
    }

    void SetFocus(float FocusX, float FocusY) {
        CameraFocusX = FocusX;
        CameraFocusY = FocusY;
    }

    void Move(float deltaX, float deltaY) {
        CameraFocusX += deltaX;
        CameraFocusY += deltaY;
    }

    float GetFocusX() const { return CameraFocusX; }

    float GetFocusY() const { return CameraFocusY; }

    void SetWorldSize(float Width, float Height) {
        WorldBoundaryLeft = -Width * 0.5f;
        WorldBoundaryTop = -Height * 0.5f;
        WorldBoundaryRight = Width * 0.5f;
        WorldBoundaryBottom = Height * 0.5f;
    }

    bool SetWorldBoundaries(float left, float top, float right, float bottom) {
        if (left < right && top < bottom) {
            WorldBoundaryLeft = left;
            WorldBoundaryTop = top;
            WorldBoundaryRight = right;
            WorldBoundaryBottom = bottom;
            return true;
        }
        return false;
    }

    void ViewportCheckBoundaries() {
        float scaledOffsetX = ViewportCenterX / Zoom;
        float scaledOffsetY = ViewportCenterY / Zoom;
        CameraFocusX = std::max(WorldBoundaryLeft + scaledOffsetX, std::min(CameraFocusX, WorldBoundaryRight - scaledOffsetX));
        CameraFocusY = std::max(WorldBoundaryTop + scaledOffsetY, std::min(CameraFocusY, WorldBoundaryBottom - scaledOffsetY));

        if (ViewportWidth / Zoom > WorldBoundaryRight - WorldBoundaryLeft) {
            CameraFocusX = (WorldBoundaryLeft + WorldBoundaryRight) * 0.5f;
        }
        if (ViewportHeight / Zoom > WorldBoundaryBottom - WorldBoundaryTop) {
            CameraFocusY = (WorldBoundaryTop + WorldBoundaryBottom) * 0.5f;
        }
    }

    void GetFocusRect(float& left, float& top, float& right, float& bottom) {
         left = CameraFocusX - ViewportCenterX;
         top = CameraFocusY - ViewportCenterY;
         right = CameraFocusX + ViewportCenterX;
         bottom = CameraFocusY + ViewportCenterY;
    }

private:
    std::random_device rd;
    std::mt19937 gen(rd());
    float TargetX, TargetY;
    float CameraFocusX, CameraFocusY;
    float ViewportCenterX, ViewportCenterY, ViewportWidth, ViewportHeight;
    float Zoom, WorldBoundaryLeft, WorldBoundaryTop, WorldBoundaryRight, WorldBoundaryBottom;
};
```

## 这是什么?
![ICON](articles/QiNuoTu/Camera2D/2.png)
