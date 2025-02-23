---
title: 让SDL程序不使用控制台
date: 2025-02-12
updated: 2025-02-23
permalink: articles/YeMinglv/SDL_no_console/
categories: YeMinglv
tags: [SDL2, 控制台]
---

本文以[生化危鸡](https://space.bilibili.com/25864506/lists/4191853?type=season)项目为例，展示如何取消掉程序运行后的出现的控制台（通常以黑框的形式出现）。  
需要工具：Visual Studio 2022，SDL2。

***注意：本文以SDL2作为演示。关于SDL3，参见文末SDL3部分。***
<!-- more -->

## 原理
当项目属性 -> 链接器 -> 系统 -> 子系统的值为控制台(/SUBSYSTEM:CONSOLE)时，操作系统为程序提供控制台。  
将其修改为窗口(/SUBSYSTEM:WINDOWS)后（建议将预处理器定义的_CONSOLE也换为_WINDOWS），
操作系统不再提供控制台，但是链接器默认会寻找WinMain函数作为入口（而非main函数）。  
而SDL2main.lib中有WinMain函数的定义，将SDL2main.lib添加到附加依赖项，正常定义main函数即可，例：int main(int argc, char* argv[])。

## 实践
右键项目，点击属性  
![项目属性](articles/YeMinglv/SDL_no_console/project_property.png)  
将子系统的值切换为窗口(/SUBSYSTEM:WINDOWS)

最好将预处理器定义的_CONSOLE也换为_WINDOWS
![预处理器](articles/YeMinglv/SDL_no_console/preprocessor.png)

添加依赖项，注意勾选从父级或项目默认设置继承
![附加依赖项](articles/YeMinglv/SDL_no_console/additional_dependencies.png)

运行程序，没有弹出控制台窗口
![结果](articles/YeMinglv/SDL_no_console/result.png)

## 常见错误
![错误1](articles/YeMinglv/SDL_no_console/error_1.png)
错因：没有添加依赖项SDL2main.lib，找不到入口函数WinMain。

![错误2](articles/YeMinglv/SDL_no_console/error_2.png)
错因：没有勾选从父级或项目默认设置继承，找不到CommandLineToArgvW函数所在的库shell32.lib。

## SDL3
在SDL3中，不再设置SDL2main.lib这个静态库，取而代之的是单头文件的库（single-header library）。
这意味着，我们不需要设置附加依赖项，在某一源文件（通常是main.cpp）中包含<SDL3/SDL_main.h>即可。

## 参考文献
[SDL2 Windows FAQ](https://wiki.libsdl.org/SDL2/FAQWindows)（英文）  
[/ENTRY（入口点符号）](https://learn.microsoft.com/zh-cn/cpp/build/reference/entry-entry-point-symbol?view=msvc-170)(机翻)  
[SDL3主函数](https://wiki.libsdl.org/SDL3/README/main-functions)（英文）
