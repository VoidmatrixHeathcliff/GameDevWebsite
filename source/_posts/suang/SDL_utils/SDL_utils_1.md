---
title: 基于SDL的一些实用工具（一）：Camera以及SDL_RenderCopy_Camera函数
date: 2024-06-26
update: 2024-06-26
permalink: articles/suang/SDL_utils/SDL_utils_1/
categories: suang
tags: [c++, SDL]
---

看了大V老师的SDL课程之后，突然心血来潮，抛弃了EasyX而投向了SDL的怀抱中。
过程总是困难重重，所以少不了轮子的制造。
**基于SDL的一些实用工具** 是一个大的合集，可能会有后续更新。

那么这次的内容就是在SDL中实现Camera的基础操作，以及实现有关Camera的SDL_RenderCopy_Camera函数。

<!-- More -->

## 一、Camera

首先，实现一个 **Camera** 类，我们需要先实现描述摄像机位置的**Vector2**类。

我们定义`class Vector2`，并定义public变量`double x`和`double y`：

```cpp
class Vector2
{
public:
    double x = 0.0;
    double y = 0.0;

public:
    Vector2() = default;
    ~Vector2() = default;

    Vector2(double x, double y) : x(x), y(y) { }
};
```

随后重载二维向量的基本运算符号，定义模长，标准化等成员函数：

```cpp
    Vector2 operator+(const Vector2& vec) const
	{
		return Vector2(x + vec.x, y + vec.y);
	}

	void operator+=(const Vector2& vec)
	{
		x += vec.x, y += vec.y;
	}

	Vector2 operator-(const Vector2& vec) const
	{
		return Vector2(x - vec.x, y - vec.y);
	}

	void operator-=(const Vector2& vec)
	{
		x -= vec.x, y -= vec.y;
	}

	double operator*(const Vector2& vec) const
	{
		return x * vec.x + y * vec.y;
	}

	Vector2 operator*(double val) const
	{
		return Vector2(x * val, y * val);
	}

	void operator*=(double val)
	{
		x *= val, y *= val;
	}

	bool operator==(const Vector2& vec) const
	{
		return x == vec.x && y == vec.y;
	}

	bool operator>(const Vector2& vec) const
	{
		return length() > vec.length();
	}

	bool operator<(const Vector2& vec) const
	{
		return length() < vec.length();
	}

	double length() const
	{
		return sqrt(x * x + y * y);
	}

	Vector2 normalize() const
	{
		double len = length();

		if (len == 0)
			return Vector2(0, 0);

		return Vector2(x / len, y / len);
	}

	bool approx_zero() const
	{
		return length() < 0.0001;
	}
```

至此，我们需要的的 **Vector2** 类已经基本实现，接下来便是 **Camera** 类的实现：

首先，在私有变量中定义 `Vector2 position` 变量，用于记录摄像机的位置，定义 `get_position` 函数用于获得摄像机的位置，定义 `reset` 函数用于重置摄像机的位置。

```cpp
class Camera
{
public:
    Camera() = default;
    ~Camera() = default;

    const Vector2& get_position() const
    {
        return position;
    }

    void reset() 
    {
        position.x = 0.0, position.y = 0.0;
    }

private:
    Vector2 position = { 0.0,0.0 };
};
```

我们的摄像机最最最基本的功能便已经实现，接下来便是如何使用摄像机来渲染所在位置的画面。

## 二、SDL_RenderCopy_Camera

想要使用摄像机的功能，渲染摄像机所在位置的画面，我们需要了解相对坐标系的概念：

<div style="text-align:center">

**窗口坐标系 = 世界坐标系 - 摄像机坐标系**

</div>


这一概念曾在植物全明星的视频中提及到。
了解了这一点，我们就可以开始着手SDL_RenderCopy_Camera函数了。
该函数接受五个参数：

```cpp
void SDL_RenderCopy_Camera(
    const Camera& camera,       // 使用的摄像机
    SDL_Renderer* renderer,     // 渲染器
    SDL_Texture* texture,       // 要渲染的纹理
    const SDL_Rect* srcrect,    // 选择的纹理区域
    const SDL_Rect* dstrect,    // 渲染目标的位置
)
```

我们需要考虑特殊情况，当 **dstrect** 为 **nullptr** 时，我们让目标位置为(-pos_camera.x, -pos_camera.y)，非特殊情况下，目标位置为(dstrect->x - pos_camera.x, dstrect->y - pos_camera.y)。由此，可得以下代码：

```cpp
void SDL_RenderCopy_Camera(
	const Camera& camera,
	SDL_Renderer* renderer,
	SDL_Texture* texture,
	const SDL_Rect* srcrect,
	const SDL_Rect* dstrect
)
{
	const Vector2& pos_camera = camera.get_position();
	SDL_Rect rect;
	if (dstrect == nullptr)
	{
		rect.x = -pos_camera.x;
		rect.y = -pos_camera.y;
	}
	else
	{
		rect.x = (dstrect->x - pos_camera.x);
		rect.y = (dstrect->y - pos_camera.y);
	}
	SDL_QueryTexture(texture, NULL, NULL, &rect.w, &rect.h);

	SDL_RenderCopy(renderer, texture, srcrect, &rect);
}
```

以上，便是全部的 **Camera** 以及 **SDL_RenderCopy_Camera** 的基础操作。
当然，我们可以加一些细节，比如加入一个 **Timer** 类，令摄像机实现屏幕震动的效果，当然，在植物全明星中，大V老师已经详细的进行了讲解，在此不过多赘述，直接上代码。

**Timer** 类：
```cpp
class Timer
{
public:
	Timer() = default;
	~Timer() = default;

	void restart()
	{
		pass_time = 0;
		shotted = false;
	}

	void set_wait_time(double val)
	{
		wait_time = val;
	}

	void set_one_shot(bool flag)
	{
		one_shot = flag;
	}

	void set_on_timeout(std::function<void()> on_timeout)
	{
		this->on_timeout = on_timeout;
	}

	void pause()
	{
		paused = true;
	}

	void resume()
	{
		paused = false;
	}

	void on_update(double delta)
	{
		if (paused) return;

		pass_time += delta;
		if (pass_time >= wait_time)
		{
			bool can_shot = (!one_shot || (one_shot && !shotted));
			shotted = true;
			if (can_shot && on_timeout)
				on_timeout();

			pass_time -= wait_time;
		}
	}

private:
	double pass_time = 0;
	double wait_time = 0;
	bool paused = false;
	bool shotted = false;
	bool one_shot = false;
	std::function<void()> on_timeout;

};
```

**Camera** 类：
```cpp
class Camera
{
public:
	Camera()
	{
		timer_shake.set_one_shot(true);
		timer_shake.set_on_timeout(
			[&]()
			{
				is_shaking = false;
				reset();
			}
		);
	}

	~Camera() = default;

	const Vector2& get_position() const
	{
		return position;
	}

	void reset()
	{
		position.x = 0;
		position.y = 0;
	}

	void on_update(double delta)
	{
		timer_shake.on_update(delta);

		if (is_shaking)
		{
			position.x = (-50 + rand() % 100) / 50.0f * shaking_strength;
			position.y = (-50 + rand() % 100) / 50.0f * shaking_strength;
		}
	}

	void shake(float strength, int duration)
	{
		is_shaking = true;
		shaking_strength = strength;

		timer_shake.set_wait_time(duration);
		timer_shake.restart();
	}

private:
	Vector2 position = { 0,0 };		// 摄像机位置
	Timer timer_shake;				// 摄像机抖动定时器
	bool is_shaking = false;		// 摄像机是否正在抖动
	float shaking_strength = 0;		// 摄像机抖动幅度
};
```

by——suang