---
title: Git使用方法学习
date: 2024-05-29
updated: 2024-05-29
permalink: articles/FlyingfishFantasticfan/Git学习记录
categories: FlyingfishFantasticfan
tags: [Git][学习]
---


这篇文章既是记录Git的学习过程，同时也是markdown的练习，操作系统为Win11。

## Git的安装和初始化配置
1. 安装Git，[Git下载连接](https://git-scm.com/download),根据自己的操作系统进行选择，下载后运行exe文件，我下载选项暂时全部都选择默认
2. 检查是否成功安装Git，下载完成后，打开控制面板（win+R 输入cmd），查看Git版本信息
<!-- More -->
    ```
    // 输入此命令进行查看
    git --version
    ```

3. 配置Git，配置用户名和邮箱，在命令行中输入以下指令

    ```
    git config --global user.name <此处替换为你的用户名>
    git config --global user.email <此处替换成你的邮箱>

    ```
4. 检测是否配置成功，在命令行中输入以下指令
    ```
    git config user.name
    git config user.email
    ```

## Git基础使用
[中文官方文档链接](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%8E%B7%E5%8F%96-Git-%E4%BB%93%E5%BA%93)  

### 获取Git仓库
- 在已存在目录中初始化仓库  

    首先，新建一个文件夹，例如testGit，记录该文件夹的路径例如:"D:\testGit"，接下来点击鼠标右键，选择Git Bash，输入指令
    ```
    $ cd D:\testGit      //输入你自己文件夹的路径，该步骤是转到此文件夹下
    ```
    再输入
    ```
    $ git init        //该步骤将当前文件夹转化为一个Git仓库
    ```
    若显示
    ```
    Initialized empty Git repository in D:/testGit/.git/
    ```
    即为操作成功  

    接下来可以在文件夹中创建两个文本文档，命名为test1和test2.  
    通过 git add 命令来指定所需的文件来进行追踪，然后执行 git commit ：      
    `$ git add *.txt`  
    `$ git commit -m 'initial project version'`  


- 克隆现有的仓库  

    使用命令
    ```
    git clone <url>
    ```
    详情参考中文官方文档  

### 记录每次更新到仓库
工作目录下的每一个文件都不外乎这两种状态：已跟踪 或 未跟踪。  

1. 现在开始检查Git仓库的状态  
   
   在Git Bash中输入以下指令，便可查看仓库当前状态
    ```
    $ git status
    ```
    若看到以下输出
    ```
    On branch master
    nothing to commit, working tree clean
    ```
    说明你现在的工作目录相当干净。所有已跟踪文件在上次提交后都未被更改过且当前目录下没有出现任何处于未跟踪状态的新文件，否则 Git 会在这里列出来。  

2. 然后在testGit中新建一个名为README的文本文档，再次输入`$ git status`  
    可以看到以下输出
    ```
    On branch master
    Untracked files:
    (use "git add <file>..." to include in what will be committed)
        README.txt
    nothing added to commit but untracked files present (use "git add" to track)
    ```
    可以看到新建的 README 文件出现在 Untracked files 下面。除非明确表示要跟踪某文件，否则Git不会自动纳入跟踪范围，这样做生成的二进制文件或其它不想被跟踪的文件包含进来。

3. 现在让我们跟踪README文件  

   使用命令 git add 开始跟踪一个文件
    ```
    $ git add README.txt
    ```
    输入`$ git status`，会看到 README 文件已被跟踪，并处于暂存状态
    ```
    $ git status
    On branch master
    Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
        new file:   README.txt
    ```
    只要在 Changes to be committed 这行下面的，就说明是已暂存状态。 

4. 接着，尝试暂存已修改的文件
    
    打开test1，在其中随意写一些文字，例如"Hello World!"并保存修改，输入`$ git status`，可以看到
    ```
    ........  

    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
        modified:   test1.txt
    ```
    test1.txt出现在 Changes not staged for commit 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区.  
    
    要暂存这次更新，需要运行 `git add test1.txt` 命令, 接着再运行`git status`命令  
    可以看到以下输出
    ```
    $ git status
    On branch master
    Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
            new file:   README.txt
            modified:   test1.txt
    ```
    可以看到现在两个文件都已暂存，下次提交时就会一并记录到仓库。 

5. 提交更新，输入
    ```
    $ git commit
    ```
    屏幕显示
    ```
    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    #
    # On branch master
    # Changes to be committed:
    #       new file:   README.txt
    #       modified:   test1.txt
    #
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    .git/COMMIT_EDITMSG [unix] (20:11 29/05/2024)                          1,0-1 All
    "/d/testGit/.git/COMMIT_EDITMSG" [unix] 9L, 235B
    ```
    按“i”键，插入`git commit`，再按“esc”键，输入`wq`,再按回车键，便可以成功提交。  

### 查看提交历史
输入`git log`即可查看该仓库提交历史。详情指令集可查看官方文档。  



---


最后写一丢丢随笔：
我是一个普通的大学生，从小就喜欢玩电子游戏，从4399到Steam，玩了许多电子游戏，不过那时也就仅限于玩一下而已，对于怎么制作电子游戏说不上感兴趣，让我想要学习制作电子游戏的契机说起来还挺难过的，或许是长大了，又或许是因为生活中的一些不快，我慢慢的不能像儿时那样，一整天坐在电脑桌前，全身心的投入到游戏缤纷的世界里，但是我仍然热爱电子游戏，我喜欢这些用代码和美术构成的虚拟而又缤纷的世界，所以我决定把自己对游戏的热爱从玩电子游戏到制作电子游戏。  

但是万事开头难，回忆自己在大学中一年多的学习，都是像数据结构，软件工程概论这样的理论上的学习，理论学习和技术学习有着巨大的鸿沟，我根本无法从命令行开始去想象如何制作一个游戏，在我为此绞尽脑汁时，意外的在EasyX的社区中看到了大V老师的视频，这才真正开启了我学习技术的道路。

跟着大V老师学习差不多快两个月了，从刚开始跟着敲井字棋，到现在见证了社区的建立，大V老师质量优秀的视频和建立的学习交流群，提供给我游戏制作技术入门的途径，对大V老师的感谢难以言表，祝大V老师的教程越来越好，吸引更多对游戏制作感兴趣的人来观看，也祝社区越办越好，成为中国游戏制作的绿洲，滋养中国游戏界。
