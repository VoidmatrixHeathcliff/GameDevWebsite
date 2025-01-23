---
title: EasyX图片处理
date: 2025-01-23
updated: 2025-01-23
permalink: articles/zhong/easyx_image_processing/
categories: zhong
author: zhong
tags: [EasyX, C++, 游戏开发]
---

## 1.图片处理函数

````c++
void Resize(
	IMAGE* pImg,
	int width,
	int height
);
````

### 参数

**pImg**

指定要调整尺寸的绘图设备。如果为 NULL，则表示默认绘图窗口。

**width**

指定绘图设备的宽度。

**height**

指定绘图设备的高度。

<!-- More -->

## 2.获取存储图片像素色彩地址

```c++
DWORD* GetImageBuffer(IMAGE* pImg = NULL);
```

### 参数

**pImg**

绘图设备指针。如果为 NULL，表示默认的绘图窗口。



### 返回值

返回绘图设备的**显示缓冲区**指针。



### 注意

获取到的显示缓冲区指针可以直接读写。

在显示缓冲区中，每个点占用 4 个字节，因此：显示缓冲区的大小 = 宽度 × 高度 × 4 (字节)。像素点在显示缓冲区中按照从左到右、从上向下的顺序依次排列。访问显示缓冲区请勿越界，否则会造成难以预料的后果。

显示缓冲区中的每个点对应 `RGBTRIPLE` 类型的结构体：

```cpp
struct RGBTRIPLE {
	BYTE rgbtBlue;
	BYTE rgbtGreen;
	BYTE rgbtRed;
};
```

`RGBTRIPLE` 在内存中的表示形式为：`0xrrggbb` (`bb`=蓝，`gg`=绿，`rr`=红)，而常用的 `COLORREF` 在内存中的表示形式为：`0xbbggrr`。注意，两者的红色和蓝色是相反的，请用 `BGR` 宏交换红色和蓝色。

如果操作绘图窗口的显示缓冲区，请在操作完毕后，执行 `FlushBatchDraw()` 使操作生效。



## 3.获取某个像素点位置的方法

```c++
//分配图片内存
IMAGE* img = new IMAGE();

//设置图片宽高
int width;
int height;

//新建图片img调整画布，设置图片宽度高度(在EasyX中图片需要加载到缓冲区才能进行处理（渲染或调整），类似loadimage函数功能，但区别是该函数可以对图片进行修改)
Resize(img, width, height);

//获取存储图片像素色彩的首个像素的地址
DWORD* img_color_buffer=GetImageBuffer(img);

//图片矩阵image[y][x]像素索引(x,y从0开始)
int x,y;
int img_color_buffer_idx = y * width + x;

//获取某个像素点位置，该数据类型为DWORD
DWORD pix_color=img_color_buffer[img_color_buffer_idx]; 
```



## 4.获取`RGB`分量方法

### 方法一

```c++
//操作显示缓冲区中的颜色

//获取某个像素点位置，该数据类型为DWORD
DWORD pix_color=img_color_buffer[img_color_buffer_idx];
   
//获取显示缓冲区中的颜色ARGB分量（方便观看而加空格，编写不需要加）
BYTE A=(pix_color & 0xff 00 00 00)>>24;
BYTE R=(pix_color & 0x00 ff 00 00)>>16;
BYTE G=(pix_color & 0x00 00 ff 00)>>8;
BYTE B=(pix_color & 0x00000 00 ff);

//对获取了的ARGB分量的操作

//操作后组合（默认A,R,G,B数据类型为BYTE）
pix_color =(DWORD)( (A << 24) | (R << 16) | (G << 8) | B); 
```



### 方法二

+ 颜色在**内存中**的表示形式为：`0xbbggrr` (`bb`=蓝，`gg`=绿，`rr`=红)
+ 但是**显示缓冲区**中的颜色表现形式为 `0xrrggbb`。
+ 注意，两者的红色和蓝色是相反的。直接操作显示缓冲区时，可以通过`BGR` 宏交换颜色的红色和蓝色部分。



在C语言中，`COLORREF` 是一个常用的类型，主要用于表示颜色，特别是在 `Windows API` 中用于存储颜色值。

`COLORREF` 是一个32位的无符号整数，通常由三个字节组成，分别表示颜色的红色 (Red)、绿色 (Green)、蓝色 (Blue) 分量。

`COLORREF` 的结构如下：

```c++
typedef DWORD COLORREF;

// Color constants.
const COLORREF rgbRed   =  0x000000FF;
const COLORREF rgbGreen =  0x0000FF00;
const COLORREF rgbBlue  =  0x00FF0000;

const COLORREF rgbBlack =  0x00000000;
const COLORREF rgbWhite =  0x00FFFFFF;
```

它的每一部分通常是按以下顺序存储的：

- 高字节（`0x00FF0000`）：蓝色 (Blue)
- 中字节（`0x0000FF00`）：绿色 (Green)
- 低字节（`0x000000FF`）：红色 (Red)

```c++
BYTE GetRValue(COLORREF rgb);
BYTE GetGValue(COLORREF rgb);
BYTE GetBValue(COLORREF rgb);
```





````c++
//操作显示缓冲区中的颜色

//获取某个像素点位置，该数据类型为DWORD
DWORD pix_color=img_color_buffer[img_color_buffer_idx]
   
//获取显示缓冲区中的颜色RGB分量
BYTE R=GetBValue(pix_color);
BYTE G=GetGValue(pix_color);
BYTE B=GetRValue(pix_color);

//对获取了的RGB分量的操作

//操作后组合(不破坏原来Alpha透明度)
pix_color = BGR(RGB(R, G, B)) |  ((((DWORD)(BYTE)(255)) << 24) & pix_color);
````

