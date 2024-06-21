---
title: 随笔-如何提交文章到VoidGameSpace上
date: 2024-05-28
updated: 2024-05-28
permalink: articles/zExNocs/随笔-如何提交文章到VoidGameSpace上/
categories: zExNocs
tags:
  - VoidGameSpace
  - Hexo
  - Git
---

关于如果提交文章到VoidGameSpace论坛上的git教程。

<!-- More -->

---

本文章仅适用于[VoidGameSpace论坛](https://www.voidgame.space/)及其[GitHub库](https://github.com/VoidmatrixHeathcliff/GameDevWebsite)。

该教程目前仅适用于**Windows**系统。

# 如何使用github进行提交

## 一. 确认Git安装

Git下载地址: [Link](https://git-scm.com/downloads)

测试Git：打开 `cmd` 或者 `PowerShell`，输入 `git -v` 可以查看到当前Git的版本。

## 二. 配置Git设置和SSH

- 使用下面两个指令配置Git全局设置：

```
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

- 配置ssh设置：[可以参考这个文章](https://blog.csdn.net/Yaoyao2024/article/details/132123525)

1. 使用 `ssh-keygen -t rsa -C "你的邮箱"`，一路回车生成SSH。
2. 找到文件`C:\User\用户名\.ssh\id_rsa.pub`，使用记事本打开并复制里面的内容。
3. 打开GitHub在`Settings`界面左边找到`SSH and GPG keys`进入。
4. 点击 `New SSH key`，在`title`中填入合适的标题，在`SSH`中填入刚刚复制的内容。
5. 本地指令输入 `ssh -T git@github.com` 验证是否配置成功。

## 三. fork库到自己的库中

打开 [GitHub库](https://github.com/VoidmatrixHeathcliff/GameDevWebsite) 网页，点击右上角 `Fork` 按钮，复制该库到自己的库中。

选择`owner`为自己的账户，`Repository name`可以设置为默认`GameDevWebsite`。

`Description`用来描述这个库，可以随便写一些，例如：VoidGameSpace论坛的Fork库。

点击 `Create fork` 按钮。

## 四. clone库到本地

进入自己的库中找到刚刚fork的`GameDevWebsite`库，一般为 `https://github.com/用户名/GameDevWebsite`

找到 `<> Code` 绿色按钮，在本地中使用 `PowerShell` 打开想要部署的位置 or 在部署的位置中右键选择`在终端打开` or 使用`cmd` `cd` 到要部署的文件夹，使用下面三种方式之一克隆库到本地

1. 使用`HTTPS`，复制`web URL`，输入`git clone 复制的URL`部署。这个方法需要你在终端登录到Git中。
2. 使用`SSH`，复制`SSH key`，输入`git clone 复制的SSH_key`部署。
3. 点击`Download ZIP`，下载压缩包到要部署的文件夹并解压。

## 五. 编写自己的文档

在部署的项目目录`GameDevWebsite`中，路径`source\_posts\`创建自己的文件夹，并在文件夹中创建`.md`文件，参考[GitHub库](https://github.com/VoidmatrixHeathcliff/GameDevWebsite)中元数据说明，编写自己的文章。

另外，markdown的编写可以参考官方文档：[Link](https://markdown.com.cn/)

## 六. 提交文档到自己fork的库中

使用 `PowerShell` 打开项目目录`GameDevWebsite`，或者在目录中右键点击`在终端打开`，或者使用`cmd` `cd`到目录。

1. 创建自己的分支(可选，新手建议直接修改`main`分支)：使用 `git checkout -b 分支名` 创建并切换到新的分支。(分支的作用是保证`main`分支的干净，一般只有最终版本才会合并到`main`分支)。如果你已经创建过分支，就不需要再创建该分支了。
2. 添加所有文件到暂存区： `git add .`
3. 提交添加的文件： `git commit -m "修改描述"`。为了养成好习惯，修改描述要有一定的准则。例如你修改文档可以写 `"docs(你的名字): 添加了xxx文章"`。具体准则可以自行搜索学习一下。
4. push库：如果你使用的是新建的分支，使用`git push origin 分支名` 将新分支push到库中。如果你使用的是`main`分支或者是已经创建的库，那么使用 `git push` 将提交的内容push到库中。
5. 拉取申请：找到自己fork的库，点击左上方 `Pull requests`按钮，进入页面点击右上方`New pull request`按钮。左边是合并的基库，右边是申请合并的库。在右边申请的库中选择自己的库和分支(如果没有创建分支就选择`main`)，然后填写一些申请描述即可。
6. 等待审核。