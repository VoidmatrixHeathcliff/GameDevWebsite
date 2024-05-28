---
title: 使用GitHub Desktop在论坛中发表文章的方法及注意事项
date: 2024-05-28
update: 2024-05-29
permalink: articles/suang/publish_article/
categories: suang
tags: [发表文章]
---

使用 **GitHub Desktop** 向 **VoidGameSpace** 发表文章的方法以及注意事项

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

#### 5. 检查fork，拉取合并更改的请求

回到 **GitHub** 中并刷新网页，此时，一定要查看是否存在 `fork` ，如果存在，请按照下图点击完成同步。

![图1](articles/suang/publish_article/fork_img.png)

接下来点击 `Pull requests` ，再点击 `New pull request` 拉取合并更改的请求。最后点击 `Create pull request` 生成请求。

#### 6. 等待Vercel部署和大V老师的审核

完成了这一步，恭喜你已经成功将你的文章提交了，接下来就是耐心地等待你的文章通过审核了。
不过别急着高兴，关于 `markdown` 文件，还有一些注意事项。

### 二、markdown文件注意事项

***敲重点！！！！！***

- 元数据一定不能少，详情请参考 **VoidGameSpace** https://github.com/VoidmatrixHeathcliff/GameDevWebsite 的 `README.md` 文件

- 元数据中有 `title` 属性不需要写一级标题

- 最好使用 `<!-- More -->` 注释添加概述否则主页文章会全部展开显得很长

---

未完待续...... 明天更新有关图片资源的插入问题

> ~~兔姐~~ 魅魔兔子：你这个教程有毒 :D

呜呜
