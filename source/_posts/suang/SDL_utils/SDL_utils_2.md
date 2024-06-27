---
title: 基于SDL的一些实用工具（二）：Atlas类以及Animation类的实现
date: 2024-06-27
update: 2024-06-27
permalink: articles/suang/SDL_utils/SDL_utils_2/
categories: suang
tags: [c++, SDL]
---

这里是suang的**基于SDL的一些实用工具**的第二部分—— **Atlas** 和 **Animation**，以及对他们的加载、更新、渲染等操作。

<div style="text-align:center">

![兔兔这么可爱！](articles/suang/SDL_utils/rabbit_hole.png)

</div>

<!-- More -->

## 一、 Atlas

首先，明确 **Atlas** 类需要的成员变量以及成员方法，我们需要一个容器去容纳我们所需要的 **Texture** ，定义私有变量 `std::vector<SDL_Texture*> tex_list`。在有了纹理的容器后，需要 `load_from_file` 函数来将所需的纹理加载到容器中，我们使用 `sprintf` 函数来实现对path模板进行操作，具体实现如下：

```cpp
    void load_from_file(SDL_Renderer* renderer, std::string path_template, int num)
	{
		tex_list.clear();
		tex_list.resize(num);

		char path_file[256];
		for (int i = 0; i < num; ++i)
		{
			sprintf(path_file, path_template.c_str(), i + 1);
			tex_list[i] = IMG_LoadTexture(renderer, path_file);
		}
	}
```

随后，我们定义 `clear` 成员函数来清空容器，定义 `get_size` 成员函数来获取容器的大小， 定义 `get_texture` 成员函数来获取索引为 **idx** 的纹理的指针，定义 `add_texture` 成员函数来向容器中添加纹理。

至此， **Atlas** 类便已经实现，完整代码如下：

```cpp
class Atlas
{
public:
	Atlas() = default;

	~Atlas() = default;

	void load_from_file(SDL_Renderer* renderer, std::string path_template, int num)
	{
		tex_list.clear();
		tex_list.resize(num);

		char path_file[256];
		for (int i = 0; i < num; ++i)
		{
			sprintf(path_file, path_template.c_str(), i + 1);
			tex_list[i] = IMG_LoadTexture(renderer, path_file);
		}
	}

	void clear()
	{
		tex_list.clear();
	}

	size_t get_size()
	{
		return tex_list.size();
	}

	SDL_Texture* get_texture(size_t idx)
	{
		if (idx < 0 || idx >= tex_list.size())
			return nullptr;
		return tex_list[idx];
	}

	void add_texture(SDL_Texture* texture)
	{
		tex_list.push_back(texture);
	}

private:
	std::vector<SDL_Texture*> tex_list;
};
```

## 二、 Animation

在提瓦特幸存者、植物全明星、村庄保卫战以及我制作的俄罗斯方块中，动画都是以连续的快速播放的图片的形式存在的，都是通过视觉暂留效应来达到连续的动画效果，因此，在 **Animation** 类中，我们需要持有动画所需的图集。因此，在私有变量中定义 `Atlas* atlas`，但是千万不要忘记在使用 **Animation** 类之前对 **atlas** 进行赋值。

接下来，我们定义bool类形变量 **is_loop** 来描述该动画是否为循环播放的动画、定义size_t类型变量 **idx_frame** 作为帧索引，定义Timer类 **timer** 来作为实现更新动画帧索引的计时器。

定义 `reset` 成员函数来重置动画的状态至初始状态，定义 `set_atlas` 成员函数来设置动画所使用的图集，定义 `set_loop` 成员函数来设置动画是否循环播放，定义 `set_interval` 成员函数来设置动画帧与帧之间的帧间隔，也就是 **timer** 的等待时间，定义 `on_update` 成员函数来实现动画类的更新，包括对 **timer** 的更新（随着 **Animation** 类的功能增加而增加），定义 `on_render` 成员函数来实现对动画当前帧的渲染。

完整实现如下：

```cpp
class Animation
{
public:
	typedef std::function<void()> PlayCallback;

public:
	Animation()
	{
		timer.set_one_shot(false);
		timer.set_on_timeout(
			[&]()
			{
				idx_frame++;
				if (idx_frame >= atlas->get_size())
					idx_frame = is_loop ? 0 : atlas->get_size() - 1;
			}
		);
	}

	~Animation() = default;

	void reset()
	{
		timer.restart();

		idx_frame = 0;
	}

	void set_atlas(Atlas* atlas)
	{
		reset();
		this->atlas = atlas;
	}

	void set_loop(bool is_loop)
	{
		this->is_loop = is_loop;
	}

	void set_interval(double interval)
	{
		timer.set_wait_time(interval);
	}

	void on_update(double delta)
	{
		timer.on_update(delta);
	}

	void on_render(SDL_Renderer* renderer, const Camera& camera, const SDL_Rect* dstrect, bool flag = true)
	{
		if (flag)
			SDL_RenderCopy_Camera(camera, renderer, atlas->get_texture(idx_frame), nullptr, dstrect);
		else
			SDL_RenderCopy(renderer, atlas->get_texture(idx_frame), nullptr, dstrect);
	}

private:
	Timer timer;
	bool is_loop = true;
	size_t idx_frame = 0;
	Atlas* atlas = nullptr;
	PlayCallback on_finished;
	int width_frame = 0, height_frame = 0;

};
```

其中，**Camera** 类以及 **Timer** 类在上一期**基于SDL的一些实用工具**中已经有详细的实现方法，这里不再赘叙。

-  **on_render** 中的 **flag** 变量为“是否使用摄像机进行渲染”

## 三、注意事项

学习过大V老师的村庄保卫战的伙伴们应该有了解到 **ResourcesManager** 类中的各种 **pool**，类似于“池”一样的哈希表用于存储游戏所使用的资源，那么我们是否可以在 **ResourcesManager** 类中也定义一个存储 **Atlas** 类的 **atlas_pool** 呢？

首先，我们在 **ResID** 枚举类中定义我们所要加载的图集类的ID，再为我们的存储 **Atlas** 类的哈希表定义一个别名 **AtlasPool** ，于是，在 `load_from_file`函数中，我们便可以像这样将 **Atlas** 类加载到 **atlas_pool** 中了：

```cpp
    bool load_from_file(SDL_Renderer* renderer)
    {
        Atlas* atlas_menu_background = new Atlas();
		atlas_menu_background->load_from_file(renderer, "resources/background/menu_background_%d.png", 10);
		atlas_pool[ResID::Atlas_MenuBackground] = atlas_menu_background;
    }
```