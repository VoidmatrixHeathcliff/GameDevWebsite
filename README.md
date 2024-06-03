<div align="center">
    <p align="center">
        <img align="center" src="source/images/avatar.png" alt="logo" width="200">
    </p>
    <h1 align="center">VoidGameSpace</h1>
    <p align="center">游戏开发主题社区博客：
        <a href="https://www.voidgame.space/">
            https://www.voidgame.space
        </a>
    </p>
    </br>
</div>

> 文档正在完善，欢迎提交补充……

## 我该如何提交自己的文章？

### 第三方教程
[保姆级！使用网页版在VoidGameSpace中发表文章与图文。Ciallo​～](https://www.voidgame.space/articles/QiNuoTu/CorrectlyReleased/)
[使用GitHub Desktop在论坛中发表文章的方法及注意事项](https://www.voidgame.space/articles/suang/publish_article/)

### 教程
所有的文章均位于 [source/_posts](source/_posts) 目录下，拥有自己的命名空间是一个好习惯，这样不至于干扰到其他人的创作，所以如果你是首次在本站尝试提交自己的文章，应该仿照 [Demo](source/_posts/Demo) 目录使用自己的ID创建独一无二的目录，随后在新建的目录下放置自己的博客文章和与文章同名的资源目录。

元数据的配置和编写实例详见：[Demo/HelloWorld.md](source/_posts/Demo/HelloWorld.md)

+ `title`：你的文章标题，这通常是必要的；
+ `date` 与 `updated`：创作时间与更新时间，建议不要省略更新时间，避免网站生成时造成默认的更新时间错误；
+ `permalink`：永久链接，请确保你的文章位于 `articles/你的ID/` 下；
+ `categories`：必须为你的ID
+ `tags`：相关标签，可省略

> 图片资源的引用路径建议使用永久链接作为起始路径

更多使用细节可以查阅：

+ Hexo 文档：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)
+ Icarus 文档：[https://ppoffice.github.io/hexo-theme-icarus/categories/](https://ppoffice.github.io/hexo-theme-icarus/categories/)

在完美更新自己的文章后，可以直接提交推送到仓库，请确保不要修改其他无关文件，不符合规范的提交可能会审核失败。

## 我该如何展示自己的链接？

在 [_config.icarus.yml](_config.icarus.yml) `273 行`附近，可以看到以友链形式存在的个人网站链接展示配置，欢迎在后续追加自己的个人网站链接，如：

```
Voidmatrix: https://www.voidmatrix.work/
```

个人网站链接展示仅支持在本站更新分享文章的创作者，请确保个人链接键名与文章作者ID一致。
