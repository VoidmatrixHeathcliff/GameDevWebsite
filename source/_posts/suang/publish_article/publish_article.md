---
title: 使用GitHub Desktop在论坛中发表文章的方法及注意事项
date: 2024-05-28
update: 2024-06-04
permalink: articles/suang/publish_article/publish_article/
categories: suang
tags: [suang, 教程, 发表文章]
---

**suang教你用GitHub Desktop发表文章！**
**超级简单！超级详细！超级耐心！**
**一定要看！一看就会！**

<!-- More -->

### 一、提交文章的流程

#### 1. 安装以下工具

- **GitHub Desktop**
- **VSCode**

#### 2. 将GitHub库中的资源clone到本地

进入 **VoidGameSpace** 论坛的 **GitHub** 库网站，点击 `< > Code` ，选择`Open with GitHub Desktop`将资源克隆到本地。

#### 3. 使用VSCode打开库中资源并进行编辑

切换到 **GitHub Desktop** ，点击 `Open in Visual Studio Code` 即可在 **VSCode** 中进行编辑。
在左侧资源管理器中打开目录 `/GAMEDEVWEBSITE/source/_posts` 在这个目录下新建一个独立的、专属于自己的文件夹。
在自己的文件夹中新建 `markdown` 文件，编辑完成保存后返回 **GitHub Desktop** 。 

#### 4. 提交本地代码，将本地所做的更改同步到远程代码库

返回 **GitHub Desktop** 后，会自动跳转到 `Changes` 界面，确认代码无误后，点击 `Commit to main` 进行提交，再点击 `Push origin` 将更改同步到远程代码库。

#### 5. 进入自己的fork，拉取合并更改的请求

回到 **GitHub** 中并刷新网页，此时，一定要在自己fork的文件中！！！
按照下图方式点击，进入自己的fork，才能进行之后的操作。

<div style="text-align:center">

![xdm一定要记得fork啊，不然我白教了](articles/suang/publish_article/fork_img.png)

</div>

接下来点击 `Pull requests` ，再点击 `New pull request` 拉取合并更改的请求。最后点击 `Create pull request` 生成请求。

#### 6. 等待Vercel部署和大V老师的审核

完成了这一步，恭喜你已经成功将你的文章提交了，接下来就是耐心地等待你的文章通过审核了。
不过别急着高兴，关于 `markdown` 文件，还有一些注意事项。

### 二、关于markdown文件

#### 1. markdown文件的写法

markdown的语法很简单，可以参考 [Markdown 官方教程](https://markdown.com.cn/basic-syntax/) 来进行查找和学习。

#### 2. 注意事项

***敲重点！！！！！***

- **元数据一定不能少！** **元数据一定不能少！** **元数据一定不能少！**

重要的事情说一万遍！

元数据是用两个`---`分割线包围的数据，记录你的文章的**标题**、**上传时间**，**修改时间**、**永久链接**、**目录**、**标签** 这些信息，缺少了元数据的文章无法通过审核。写法如下：

```md
---
title: 使用GitHub Desktop在论坛中发表文章的方法及注意事项
date: 2024-05-28
update: 2024-05-29
permalink: articles/suang/publish_article/
categories: suang
tags: [suang, 教程, 发表文章]
---
```
- 其中，`permalink` 项要填写 `article/你的专属目录/子目录（可选）/`

虽然子目录是可选项，但是，你也不想把所有的文章都丢在同一个文件夹下吧。真的不要这样做啊！！！

- 元数据中有 `title` 属性不需要写一级标题

- 最好使用 `<!-- More -->` 注释添加概述否则主页文章会全部展开显得很长

### 三、关于图片的插入

#### 1. 导入图片资源

<div style="text-align:center">

![图片在哪里呀？](articles/suang/publish_article/image_pos.png)

</div>

首先，将你所需要的图片资源导入到你的专属文件夹中，然后就可以修改markdown文件啦~

像这样
```md
<div style="text-align:center">

![图片在哪里呀？](articles/suang/publish_article/image_pos.png)

</div>
```

括号里面写上你的图片的相对路径就可以将图片导入啦！

---

真的是保姆级了，正常来说照着我的教程一步一步来应该不会出问题了。
如果遇到了问题记得在群里@我

**Yeah~**
