---
title: 博客第一步：将Hexo部署在GitHub上
# author: shuo-liu16
date: 2024-4-10
permalink: articles/shuo-liu16/deploy-hexo/
categories: shuo-liu16
tags: hexo
---

Hexo 是一个快速、简单且强大的静态博客框架，与 GitHub 结合使用可轻松搭建个人博客网站。

<!-- more -->

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。其主题，模板也更加丰富多样，更适合创建功能丰富的个人博客或其他类型的静态网站。

## Quick Start

接下来，我将分享我在GitHub上部署Hexo的过程步骤，并给出其中我遇到的一些困难与解决方法。。

### 1.安装Node

Hexo是一个基于Node.js的静态网站生成器，需要在本地安装Node.js环境，并通过命令行来创建、编译和部署网站。

下载链接: [Node](https://nodejs.cn/)

### 2.安装Sublime text

这个算是文本编辑器，但功能更加强大，有许多妙用。

下载链接: [Sublime text](https://www.sublimetext.com/)

### 3.下载Git

下载链接: [Git](https://git-scm.com/)

#### 基本配置Git

打开gitbush终端输入一下指令（名字邮箱皆与GitHub相关），这将关系到你是否能将仓库推送到GitHub上

a.配置昵称，邮箱

``` bash
$ git config --global user.name "Your Name"
$ git config --global user.email "your.email@example.com"
```

b.生成ssh

``` bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"         //生成ssh密钥
```

c.添加 SSH 公钥到GitHub账户

#### 使用Git链接远程仓库（后续会使用到）

需要替换url

``` bash
$ git init          //初始化仓库
$ git remote add origin https://github.com/exampleuser/example.git    
$ git remote -v     //查看是否关联成功
```

### 4.创建GitHub仓库

建立一个公共仓库，仓库名：<你的 GitHub 用户名>.github.io
eg：Dream-verylively.github.io

### 5.配置并使用Hexo

a.配置hexo

这个步骤将生成blog文件夹，接下来的诸多操作将围绕这个文件夹进行

``` bash
$ npm install hexo-cli -g       //安装 Hexo CLI（命令行工具）到全局环境
$ hexo init blog                //始化 Hexo 博客，创建一个名为 "blog" 的目录，并在其中初始化 Hexo（名字可更改）
$ cd blog                       //进入到刚创建的 "blog" 目录中
$ npm install                   //npm install
$ hexo server                   //启动 Hexo 服务器，用于本地预览您的博客
```

More info: [hexo](https://hexo.io/zh-cn/)

b.编辑你的 _config.yml

第一处
![ICON](articles/shuo-liu16/deploy-hexo/1.png)
第二处
![ICON](articles/shuo-liu16/deploy-hexo/2.png)

c.在博客文件夹下打开终端执行以下命令

``` bash
$ hexo clean            //清理 Hexo 生成的静态文件和缓存
$ hexo g                //使用 Hexo 生成静态文件，将 Markdown 格式的文章转换为 HTML 等静态文件
$ hexo deploy           //部署生成的静态文件到指定的部署目标（例如 GitHub Pages、FTP 等）
```

### GithubPages 404问题

#### 1、github 仓库名称不匹配

在 github 上面创建的仓库名称没有使用自己的 github 账号名称，例如你的 github 账号名称是 zhanghao，而你创建的仓库名称是 suibian.github.io，这样你是无法访问你的博客网站的。

#### 2、配置文件错误

在你的本地博客目录下（我创建的名称是 hexo，注意不是 themes 主题目录下的那个配置文件），有个配置文件 _config.yml，注意观察里边的内容是否符合条件。

#### 3、分支错误

注意提交的分支要与GitHub仓库中的默认分支一致

#### 4、hexo 依赖不全

检查依赖安装是否有不全的情况，可以使用 npm list 命令

``` bash
$ npm list --depth 0
```

如果检查有问题，会在控制台提示类似 npm ERR! missing xxxxx。

#### 5.版本不匹配问题（node与hexo）

``` bash
$ node -v           //检查两者的版本
$ hexo -v
```

看这个：[版本限制](https://hexo.io/zh-cn/docs/#Node-js-%E7%89%88%E6%9C%AC%E9%99%90%E5%88%B6)

###### 写在最后：这个过程并不是一帆风顺的，很多时候甚至让我痛苦，但最后我坚持下来了，现在想想尽管痛苦，但是成功了，似乎还不错。