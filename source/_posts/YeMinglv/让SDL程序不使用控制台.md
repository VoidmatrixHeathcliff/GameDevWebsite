---
title: 让SDL程序不使用控制台
date: 2025-02-12
updated: 2025-02-12
permalink: articles/YeMinglv/SDL_no_console/
categories: YeMinglv
tags: [SDL2, 控制台]
---

本文以[生化危鸡](https://space.bilibili.com/25864506/lists/4191853?type=season)项目为例，展示如何取消掉程序运行后的出现的控制台（通常以黑框的形式出现）。  
需要工具：Visual Studio 2022，SDL2。

<!-- more -->

## 原理
当项目属性 -> 链接器 -> 系统 -> 子系统的值为控制台(/SUBSYSTEM:CONSOLE)时，操作系统为程序提供控制台。  

将其修改为窗口(/SUBSYSTEM:WINDOWS)后（建议将预处理器定义的_CONSOLE也换为_WINDOWS），操作系统不再提供控制台，但是链接器默认会寻找WinMain函数作为入口（而非main函数）。  

而SDL2main.lib中有WinMain函数的定义，将SDL2main.lib添加附加依赖项，正常定义main函数即可，例：int main(int argc, char* argv[])。

## 实践
右键项目，点击属性
![项目属性](articles/YeMinglv/SDL_no_console/project_property.png)
将子系统的值切换为窗口(/SUBSYSTEM:WINDOWS)

最好将预处理器定义的_CONSOLE也换为_WINDOWS
![预处理器](articles/YeMinglv/SDL_no_console/preprocessor.png)

添加依赖项，注意勾选从父级或项目默认设置继承
![附加依赖项](articles/YeMinglv/SDL_no_console/additional_dependencies.png)

## 常见错误
![错误1](articles/YeMinglv/SDL_no_console/error_1.png)
错因：没有添加依赖项SDL2main.lib，找不到入口函数WinMain。  

![错误2](articles/YeMinglv/SDL_no_console/error_2.png)
错因：没有勾选从父级或项目默认设置继承，找不到CommandLineToArgvW函数所在的库shell32.lib。
## 参考文献
[SDL2 Windows FAQ](https://wiki.libsdl.org/SDL2/FAQWindows)（英文）
