---
title: 2D 游戏光照分析
date: 2024-08-24
updated: 2024-08-24
author: Margoo
permalink: articles/Margoo/2d-light/
mathjax: true
tags: [Math, Computer Graphics, Lighting]
---

<div align="center">

![](articles/Margoo/2d-light/cover.png)

</div>

本文将会介绍一个我正在制作的项目 PaperCraft 中有关游戏 2D 光照的实现. 提供一个低成本的基于有符号距离场可用于实时 2D 光照渲染的可行思路. 本文将会提供一种可能的 SKSL（Skia Shader Language）代码来实现该方法.

<!-- More -->

## Ⅰ、基础知识简介

SDF（Signed distance function），有符号距离函数.是一个多元函数（具体几元取决于研究对象所处的维度），本文记作 $\varphi\left(\boldsymbol{x}\right)$.其返回一个值，该值为点 $\boldsymbol{x}$ 到目标几何体的最短距离.不同的几何体有不同的 SDF 函数.例如圆的 SDF 为 $\varphi\left(\boldsymbol{x}\right)=\left|\left|\boldsymbol{x}-\boldsymbol{p}\right|\right|_n-R$.其中 $R$ 为圆的半径.若 $\varphi\left(\boldsymbol{x}\right)<0$ 则说明点在圆内；若 $\varphi\left(\boldsymbol{x}\right)=0$ 则说明点在圆上；若 $\varphi\left(\boldsymbol{x}\right)>0$ 则说明点在园外.本文不提供任何几何体的 SDF 函数推导过程，读者若有兴趣可自行查找相关资料.

法线贴图（Normal Texture），是一种凹凸贴图（Bump Map）.可以表示物体的表面细节（如凹凸、划痕）.一个常见的法线贴图如下所示：

<div align="center">

![](articles/Margoo/2d-light/normal_texture.png)

</div>

事实上，法线贴图就是将法向量 $\boldsymbol{n}=\left(x,y,z\right)$ 映射到了 RGB 空间中；由于 $n_i\in\left[-1,1\right]$，于是就存在如下映射关系：

$$\boldsymbol{C}=0.5\boldsymbol{n}+0.5$$

因此，可以将物体表面粗糙的法向量 $\boldsymbol{n}$ 全部映射到 RGB 空间中，并储存为图片。在渲染中，我们可以用这一低成本的方法实现物体表面粗糙平面的渲染。得到更好的细节。事实上，如果放大文章封面：

<div align="center">

![](articles/Margoo/2d-light/normal_result.png)

</div>

你可以清晰地看到法线贴图的效果，红石块和石块表面看起来来凹凸不平，极具立体感。

关于法线贴图和 SDF 的详细描述，请分别参考 [LearnOpenGL - 法线贴图](https://learnopengl-cn.github.io/05%20Advanced%20Lighting/04%20Normal%20Mapping/) 和 [知乎 Jacks0n - Rendering (Signed) Distance Function](https://zhuanlan.zhihu.com/p/345227118).

## Ⅱ、渲染管线概述

下图简单直观地展示了本方法的渲染管线：

<div align="center">

![](articles/Margoo/2d-light/pipeline.png)

</div>

这里再给出这几个步骤的简述：

1. 基础场景渲染

    该流程使用原始贴图渲染出未经光照的原始地图.

2. 法线场景渲染

    该流程使用法线贴图渲染出未经光照的原始地图.

3. 光照贴图渲染

    该流程将会基于 SDF 函数计算出原始光照蒙版贴图.

4. 辉光处理

    该流程基于高斯模糊，对阶段三生成的光线蒙版贴图进行辉光处理.

5. 混合

    该步骤将会将步骤一、二、四得到的结果混合并计算出最终结果.

下文将会详细讲述步骤三至五.

## Ⅲ、基于 SDF 的蒙版光照计算

根据现实生活中的经验，可以发现，一个发光的物体照亮的区域形成一个圆圈.且先保持强度不变再衰减，具体示意图如下：

<div align="center">

![](articles/Margoo/2d-light/light.png)

</div>

那么便可以通过 SDF 模拟该过程：

假设某点处的光源强度 $\boldsymbol{I}=\left(x,y,z\right)$（其中 $x,y,z\in\left[0,1\right]$，$1$ 为最亮，$0$ 为最暗）

我们假设有一个光源 $A$，颜色为 $C$，其光强为 $I_A$， SDF 函数为 $\varphi_A\left(\boldsymbol{x}\right)$，衰减半径为小 $r$.$\boldsymbol{R}$ 为一满足在 $\boldsymbol{P}$ 方向上使得 $\varphi_A\left(\boldsymbol{R}\right)=0$ 的一点，则 $A$ 在 $\boldsymbol{P}$ 处的辐射度 $\boldsymbol{L_{A,P}}$ 可以为：

<div align="center">

![](articles/Margoo/2d-light/equation1.svg)

</div>

注意（1）中 $\frac{\varphi_A\left(\boldsymbol{P}\right)-R+r}{r}C$ 实际上是由：$\left(1-\frac{\varphi_A\left(\boldsymbol{P}\right)-R}{r}\right)C$ 推到得到的.显然，有 $\boldsymbol{I}_x\in\left[0,1\right]$，因此，我们可以考虑，使用一个非线性插值 $D\left(\boldsymbol{x}\right)$ 函数来对平滑光照衰减.特别地，我们需要确保 $D\left(0\right)=0$、$D\left(1\right)=1$. 此处采用 $D\left(x\right)=\sqrt[3]{x}$ 作为衰减平滑函数。

根据在 1986 年由 James T.Kajiya 提出的渲染方程：

$$L_O\left(\vec{v}\right)=L_e\left(\vec{v}\right)+\int_{\Omega}f\left(\vec{l_i}, \vec{v}\right)L_i\left(\vec{l_i}\right)\left(\vec{n}\cdot\vec{l_i}\right)\text{d}\vec{l_i}$$

这里我们无需过多关注这个公式本身（如果你想了解这个公式，可以参考 James T.Kajiya 的论文 *The Rendering Equation*），在 2D 平面中，我们可以认为所有光线都将射入摄像机，且任意点上收到的光照，等于所有光线辐射度的总和，可以将球面积分改写成很简洁的形式：

$$L_O\left(\vec{v}\right)=\int^{2\pi}_{0}L\left(\vec{v}, \theta\right)\text{d}\theta$$

其中 $L\left(\vec{v}, \theta\right)$ 表示 $\theta$ 方向上受到的光亮。然而，由于我们的光源是确定的，在不考虑阴影渲染的情况下，我们可以认为有：

<div align="center">

![](articles/Margoo/2d-light/equation2.svg)

</div>

之所以是在不考虑阴影的情况下才有上述等式成立，是因为当有一个物体遮挡时，光线就并不总是可以完全照射到指定点处。这时就要考虑可能的辐射度衰减。

因此，假设有 $n$ 个光源 $\{A_n\}$，则 $\boldsymbol{P}$ 点处的辐射度 $\boldsymbol{L}$ 则为：

<div align="center">

![](articles/Margoo/2d-light/equation3.svg)

</div>


此处给出一个可能的 SKSL 着色器实现：

```GLSL
// The light type of the shader
//      If LightType < 10. = Ellipse
//      Else = Rectangle
uniform float LightType;
// The X position of the light
uniform float2 Center;
// The radius of the light source object
uniform float Radius;
// The range of the light source
uniform float ValidRadius;
// The light brighteness level of the light source
uniform float Intensity;
// The color of the light source
uniform float3 Color;

// The texture shader for background texture
uniform shader BackgroundTexture;

// The SDF of ellipse
float EllipseSDF(in vec2 R, in float Radius) {
    return length(R) - Radius;
}
// The SDF of rectangle
float RectangleSDF(in vec2 R, in float Radius) {
    vec2 Box    = vec2(Radius, Radius);
    vec2 Delta  = abs(R) - Box;
    return length(max(Delta, vec2(0.))) + min(max(Delta.x, Delta.y), 0.0);
}
// The phi interpolation function
float Phi(in float Value) {
    return 1.0 - pow(Value, 0.3);
}

// Calculate the light brightness contribution from the SDF
vec3 SDFContribution(in float Distance, in float Intensity, in float ValidRadius, in vec3 Color) {
    vec3 col = vec3(0., 0., 0.);
    if (Distance <= 0.0) {
        float D = Intensity + 1.;
        col = Color * D;
    }
    else if (Distance <= ValidRadius) {
        float F = (Distance / ValidRadius);
        float D = Phi(F) * (Intensity + 1.);
        col = vec3(Color[0] * D, Color[1] * D, Color[2] * D);
    }

    return col;
}
// Sample the coord light from a ellipse object
vec3 SampleEllipse(in vec2  Center,
                   in vec3  Color,
                   in float Radius,
                   in float ValidRadius,
                   in float Intensity,
                   in vec2  Coord) {
    vec2 R = Center - Coord;

    float sdf = EllipseSDF(R, Radius);
    return SDFContribution(sdf, Intensity, ValidRadius, Color);
}
// Sample the coord light from a rectangle object
vec3 SampleRectangle(in vec2  Center,
                     in vec3  Color,
                     in float Radius,
                     in float ValidRadius,
                     in float Intensity,
                     in vec2  Coord) {
    vec2 R = Center - Coord;

    float sdf = RectangleSDF(R, Radius);
    return SDFContribution(sdf, Intensity, ValidRadius, Color);
}

// Fix the gamma value
vec3 GammaFixed(in vec3 R) {
    return vec3(pow(R[0], 0.9), pow(R[1], 0.9), pow(R[2], 0.9));
}

vec4 main(in vec2 fragCoord) {
    vec4 fragColor = BackgroundTexture.eval(fragCoord);
    if (LightType < 10.) fragColor += vec4(SampleEllipse(Center, Color, Radius, ValidRadius, Intensity, fragCoord), 1.);
    else fragColor += vec4(SampleRectangle(Center, Color, Radius, ValidRadius, Intensity, fragCoord), 1.);

	return fragColor;
}
```

## Ⅳ、辉光处理

尽管在章节 Ⅲ 中已经引入了光照平滑函数 $D\left(x\right)$ 来尽可能地让光照平滑自然，但如下图所示，如果要让光照足够的自然，最好还是模拟现实中发光物体的“辉光”效果：

<div align="center">

![](articles/Margoo/2d-light/bloom.png)

</div>

为了实现辉光效果，一个可行的思路就是通过高斯模糊来对原来的蒙蔽光照贴图进行处理.

高斯模糊使用到了二阶正态分布，通过在目标点指定半径大小内计算每个点相当对于该点的权重 $w_{x,y}$ 来实现模糊效果：

$$w_{x,y}=f(x, y) = \frac{1}{2\pi\sigma_1\sigma_2\sqrt{1-\rho^2}} e^{-\frac{1}{2(1-\rho^2)}\left[\frac{(x-\mu_1)^2}{\sigma_1^2} - 2\rho\frac{(x-\mu_1)(y-\mu_2)}{\sigma_1\sigma_2} + \frac{(y-\mu_2)^2}{\sigma_2^2}\right]}$$

关于高斯模糊的详细描述，请参考[维基百科](https://zh.wikipedia.org/wiki/%E9%AB%98%E6%96%AF%E6%A8%A1%E7%B3%8A).

一个可能的辉光 SKSL 实现代码如下：

```GLSL
uniform shader BackgroundTexture;

float pow2(in float x) {
    return ((x) * (x));
}

const float pi      = 3.14159265358979323846;
const float samples = 30;
const float sigma   = samples * 0.25;

float gaussian(float2 i) {
    return 1.0 / (2.0 * pi * pow2(sigma)) * exp(-((pow2(i.x) + pow2(i.y)) / (2.0 * pow2(sigma))));
}

vec4 main(float2 coord) {
    vec4  color = vec4(0.0);
    float accum = 0.0;

    for (float x = -samples / 2; x <= samples / 2; ++x) {
        for (float y = -samples / 2; y <= samples / 2; ++y) {
            float2 offset   = float2(float(x), float(y));
            float weight    = gaussian(offset);
            color           += BackgroundTexture.eval(coord + offset) * weight;
            accum           += weight;
        }
    }

    color /= accum;
    return color;
}
```

## Ⅵ、混合

在最后阶段中，我们需要将前几个步骤得到的结果混合起来，完成光照渲染。混合公式为：

<div align="center">

![](articles/Margoo/2d-light/equation4.svg)

</div>

实际上就是把 $m$ 个颜色的 r，g，b 数值相乘后组成新的颜色。事实上，这相当于 PhotoShop 中的 “正片叠底”效果。

可能的 SKSL 实现代码如下：

```GLSL
uniform shader LightMask;
uniform shader Normal;
uniform shader Origin;

vec4 main(in vec2 fragCoord) {
    return 1.5 * Normal.eval(fragCoord) * LightMask.eval(fragCoord) * Origin.eval(fragCoord);
}
```


## Ⅶ、总结

本文提供了一个低成本的基于有符号距离场可用于实时 2D 光照渲染的可行思路. 事实上，如果优化得当，该方法在 CPU 上依然可以使用。因为本方法并不考虑阴影遮挡，因此可以说只要光源是确定（即全为静态光源）就可以考虑采用该方法预先进行光线烘培。即使是需要动态光源，在确保光源本身属性不变的情况下，依然可以预先渲染蒙版光源。

本方法的很多渲染并不一定需要实时进行，这也是为什么在 CPU 上实现这个方法成为了可能。