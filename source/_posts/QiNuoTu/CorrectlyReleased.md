---
title: 保姆级！使用网页版在VoidGameSpace中发表文章与图文。Ciallo​～
date: 2024-05-30
updated: 2024-06-4
permalink: articles/QiNuoTu/CorrectlyReleased/
categories: QiNuoTu
tags: [教程]
---
保姆级！保姆级！保姆级！Ciallo​～！
每一步都有图文混合讲解！
<!-- More -->
<div align="center">
    <p align="center">
        <img src="/articles/QiNuoTu/icon.png" alt="logo" width="200">
    </p>
    <h1>琪诺兔</h1>
    <p>
        <a href="https://space.bilibili.com/69720374" target="_blank">关注我的哔哩哔哩走进我的生活</a>
        &nbsp;|&nbsp;
        <a href="https://github.com/QiNuoTu" target="_blank">关注我的GitHub获得我的代码</a>
    </p>
</div>

# 第一步，先分叉一个VoidGameSpace的原始库
- **1**: 点击蓝字进入[VoidGameSpace](https://github.com/VoidmatrixHeathcliff/GameDevWebsite)。
进入后，会看到这样的页面。
![ICON](articles/QiNuoTu/CorrectlyReleased/1.png)
- **2**: 点击右上角的，Fork，如果不明白全部默认，点击绿色按钮即可。
![ICON](articles/QiNuoTu/CorrectlyReleased/2.png)
![ICON](articles/QiNuoTu/CorrectlyReleased/3.png)
- **3**: 此刻你的库中就有了一份分叉文件，第一步完成。
# 第二步，来到自己的库中，并进入GameDevWebsite项目。
- **1**: 点击左上角的，点点点，标志。
![ICON](articles/QiNuoTu/CorrectlyReleased/4.png)
选择，自己名字加上斜杠的/GameDevWebsite项目，非常重要！
![ICON](articles/QiNuoTu/CorrectlyReleased/5.png)
- **2**: 这样就进入了自己库中的GameDevWebsite克隆项目。
![ICON](articles/QiNuoTu/CorrectlyReleased/6.png)
这里的头像变成自己的，就正确了。
# 第三步，来到，_posts，文件夹。
点击项目中的，source，文件夹。
![ICON](articles/QiNuoTu/CorrectlyReleased/7.png)
在点击，_posts，文件夹
![ICON](articles/QiNuoTu/CorrectlyReleased/8.png)
这里会有一大堆别人的文件夹，不需要管理。
![ICON](articles/QiNuoTu/CorrectlyReleased/9.png)
# 第四步，准备文章！
此刻，网页上的操作已经完成，回到本地，用自己的名字创建一个文件夹。
## 自己的名字！，不要抄我的名字！
![ICON](articles/QiNuoTu/CorrectlyReleased/10.png)
进入文件夹，使用你任何喜欢的md文件编辑器在其中创建一个文件，然后进入，开始编写。
编写参阅，[VoidGameSpace](https://github.com/VoidmatrixHeathcliff/GameDevWebsite)。下方的说明。
- **1**: 跳过上面的说明我们创建一个。
![ICON](articles/QiNuoTu/CorrectlyReleased/11.png)
- **2**: 打开，在头部添加以下内容。
![ICON](articles/QiNuoTu/CorrectlyReleased/12.png)
permalink: 不太好理解，不用担心，请往下看，
它是这样的结构，permalink: articles/QiNuoTu/Demo/
- **articles**: 可以理解为根目录，必须加上。
- **QiNuoTu**: 这个是我的名字，一个示意，需要修改为你名字文件夹的名称。
![ICON](articles/QiNuoTu/CorrectlyReleased/10.png)
- **Demo**: 这个是你创建的当前文章的文件名，不要扩展名，但要加上，/，斜杠在尾部。
![ICON](articles/QiNuoTu/CorrectlyReleased/13.png)
这样一来，文章头就准备完毕了。
![ICON](articles/QiNuoTu/CorrectlyReleased/14.png)
# 第五步，编写文章！
- **1**: 在上面的截图可以注意到，文章头下面有一些白字，没错，这个可以当作简介，或者先到言，具体的文章内容需要在，` <!-- More --> ` 标记下书写
![ICON](articles/QiNuoTu/CorrectlyReleased/15.png)
- **2**: 如何添加图片，在文章旁边使用同名创建一个文件夹。
![ICON](articles/QiNuoTu/CorrectlyReleased/16.png)
- **3**: 将图片丢进文件夹中，就可以使用，`![ICON](articles/自己名称文件夹名/文件夹名/文件名.后缀名)`表达式引用图片了。
![ICON](articles/QiNuoTu/CorrectlyReleased/17.png)
- **4**: 网页链接同理，`[名称](链接)`，表达式，即可。
![ICON](articles/QiNuoTu/CorrectlyReleased/27.png)
- **5**: 置入代码，只需要使用，` ```cpp 代码 ``` `将代码包裹其中即可。
![ICON](articles/QiNuoTu/CorrectlyReleased/28.png)
# 第六步，发表文章。
- **1**: 回到网页，那一堆名字的文件夹的页面，点击右上角的，上传文件。
![ICON](articles/QiNuoTu/CorrectlyReleased/18.png)
- **2**: 把文件夹拖住丢上去。
![ICON](articles/QiNuoTu/CorrectlyReleased/19.png)
- **3**: 点击绿色按钮。
![ICON](articles/QiNuoTu/CorrectlyReleased/20.png)
- **4**: 之后就可以看到自己的文件夹了。
![ICON](articles/QiNuoTu/CorrectlyReleased/21.png)
- **5**: 点进去！，就可以看到自己的md文件和文件夹什么的了，在提交之前，需要说明，建立文件夹克隆库一类都只需要做一次，每次发布只要弄文章md和图片文件夹上传到自己的文件夹中即可。
![ICON](articles/QiNuoTu/CorrectlyReleased/22.png)
# 第七步，提交！
- **1**: 点击右上角的GameDevWebsite。
![ICON](articles/QiNuoTu/CorrectlyReleased/23.png)
- **2**: 找到，Contribote，点击绿色按钮！
![ICON](articles/QiNuoTu/CorrectlyReleased/24.png)
- **2**: 填写标题，点击绿色按钮！
![ICON](articles/QiNuoTu/CorrectlyReleased/25.png)
# 等待审核即可！
![ICON](articles/QiNuoTu/CorrectlyReleased/26.png)

# 这是什么？
![ICON](articles/QiNuoTu/CorrectlyReleased/123415231.png)
