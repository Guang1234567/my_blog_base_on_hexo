---
title: 本博客写文章所用的一些特性和组件
aplayer: true
copyright: true
sticky: true
cover: >-
  https://cdn.jsdelivr.net/gh/Molunerfinn/test/Melody/8700af19ly1fjbc7xhpzmj212o0ha77e.jpg
top_img: >-
  https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/hexo-theme-butterfly-docs-error404.png
tags:
  - Java
  - Android
  - Swift
categories: Java
keywords: 'Java,Android,Swift'
description: 对美好人生的思考�
katex: false
abbrlink: e20f10f1
date: 2020-08-07 12:08:09
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash /source/_posts/hello-world.md https://github.com/Guang1234567/my_blog_base_on_hexo/blob/ec175f8dc049d1cf9f79909801221136d388ed72/source/_posts/hello-world.md#create-a-new-post
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

## 代码块进阶

- 某几行高亮

{% codeblock lang:bash 高亮代码块里面的某几行 https://hexo.io/zh-cn/docs/tag-plugins.html#%E4%BB%A3%E7%A0%81%E5%9D%97 高亮教程链接地址 mark:3,5-7%}
$ hexo server

我是代码高亮行示例3
我是代码高亮行示例4
我是代码高亮行示例5
我是代码高亮行示例6
我是代码高亮行示例7
{% endcodeblock %}

- gist

```markdown 请在 markdown 文档中插入下面代码 https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729 trial.key
{% gist b562756ea42cb36d21c54aa656798729 %}
```

{% gist b562756ea42cb36d21c54aa656798729 %}

or

```markdown 请在 markdown 文档中插入下面代码 https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729 trial.key
<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>
```

<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>

## 音乐

在文章中插入音乐播放列表:

```markdown
<!-- 简单示例 (id, server, type)  -->
{% meting "60198" "netease" "playlist" %}

<!-- 进阶示例 -->
{% meting "60198" "netease" "playlist" "autoplay" "mutex:false" "listmaxheight:340px" "preload:none" "theme:#ad7a86"%}
```

{% meting "60198" "netease" "playlist" %}

有关  `{% meting %}`  的选项列表如下:

选项          | 默认值     | 描述
--------------|------------|------------------------------------------------------
id            | **必须值** | 歌曲 id / 播放列表 id / 相册 id / 搜索关键字
server        | **必须值** | 音乐平台: `netease`, `tencent`, `kugou`, `xiami`, `baidu`
type          | **必须值** | `song`, `playlist`, `album`, `search`, `artist`
fixed         | `false`    | 开启固定模式
mini          | `false`    | 开启迷你模式
loop          | `all`      | 列表循环模式：`all`, `one`,`none`
order         | `list`     | 列表播放模式： `list`, `random`
volume        | 0.7        | 播放器音量
lrctype       | 0          | 歌词格式类型
listfolded    | `false`    | 指定音乐播放列表是否折叠
storagename   | `metingjs` | LocalStorage 中存储播放器设定的键名
autoplay      | `true`     | 自动播放，移动端浏览器暂时不支持此功能
mutex         | `true`     | 该选项开启时，如果同页面有其他 aplayer 播放，该播放器会暂停
listmaxheight | `340px`    | 播放列表的最大长度
preload       | `auto`     | 音乐文件预载入模式，可选项： `none`, `metadata`, `auto`
theme         | `#ad7a86`  | 播放器风格色彩设置

More info: [音乐](https://github.com/MoePlayer/hexo-tag-aplayer/blob/master/docs/README-zh_cn.md)


## 图片

![绝对路径引用图片](https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/hexo-theme-butterfly-doc-highlight-mac.png)

![相对路径引用图片](Swift_With_Android.png)

{% asset_path Swift_With_Android.png This is an example image %}
{% asset_img Swift_With_Android.png This is an example image %}
{% asset_link Swift_With_Android.png This is an example image %}


## 颜色标签

`````markdown
{% note default %}
default 提示块标籤
第二行
第三行
第四行

```markdown 请在 markdown 文档中插入下面代码 https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729 trial.key
<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>
```

<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>
{% endnote %}

{% note primary no-icon %}
primary 提示块标籤
{% endnote %}

{% note success %}
success 提示块标籤
{% endnote %}

{% note info %}
info 提示块标籤
{% endnote %}

{% note warning %}
warning 提示块标籤
{% endnote %}

{% note danger %}
danger 提示块标籤
{% endnote %}
`````

{% note default %}
default 提示块标籤
第二行
第三行
第四行

```markdown 请在 markdown 文档中插入下面代码 https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729 trial.key
<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>
```

<script src="https://gist.github.com/Guang1234567/b562756ea42cb36d21c54aa656798729.js"></script>
{% endnote %}

{% note primary no-icon %}
primary 提示块标籤
{% endnote %}

{% note success %}
success 提示块标籤
{% endnote %}

{% note info %}
info 提示块标籤
{% endnote %}

{% note warning %}
warning 提示块标籤
{% endnote %}

{% note danger %}
danger 提示块标籤
{% endnote %}


## 内容点击展开

**参数解释:**

```
content: 文本内容

display: 按钮显示的文字 (可选)

bg: 按钮的背景颜色 (可选)

color: 按钮文字的颜色 (可选)
```

### Block 形式

**语法:**

```markdown
{% hideBlock display,bg,color %}
content
{% endhideBlock %}
```
**Demo:**

```markdown
{% hideBlock 查看答案,#40A8F3,#fff %}
傻子，怎么可能有答案
{% endhideBlock %}
```

查看答案
{% hideBlock 查看答案,#40A8F3,#fff %}
傻子，怎么可能有答案
{% endhideBlock %}

---

```markdown
{% hideBlock 查看答案2 %}
傻子，怎么可能有答案2
{% endhideBlock %}
```

查看答案2
{% hideBlock 查看答案2 %}
傻子，怎么可能有答案2
{% endhideBlock %}


### Toggle 形式

**语法:**

```markdown
{% hideToggle display,bg,color %}
content
{% endhideToggle %}
```

**Demo:**

```markdown
{% hideToggle Butterfly安装方法 %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

如果想要安装比较新的dev分支，可以

git clone -b dev https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

{% endhideToggle %}
```

{% hideToggle Butterfly安装方法 %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

如果想要安装比较新的dev分支，可以

git clone -b dev https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

{% endhideToggle %}

---

```markdown
{% hideToggle Butterfly安装方法,#40A8F3,#fff %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

![](Swift_With_Android.png)

{% endhideToggle %}
```

{% hideToggle Butterfly安装方法,#40A8F3,#fff %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

![](Swift_With_Android.png)

{% endhideToggle %}


## 画图

使用 mermaid 标籤可以绘製 Flowchart（流程图）、Sequence diagram（时序图 ）、Class Diagram（类别图）、State Diagram（状态图）、Gantt（甘特图）和 Pie Chart（圆形图），具体可以查看 [mermaid 文档](https://mermaid-js.github.io/mermaid/#/)

**语法:**

```markdown
{% mermaid %}
内容
{% endmermaid %}
```

**Demo:**

```markdown
{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}
```

{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}


## Tabs

选项卡

```markdown
{% tabs test1 %}
<!-- tab 第一个Tab-->
**This is Tab 1.**

- 1
    - 11
    - 12
- 2
    - 22
- 3

{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}

{% hideToggle Butterfly安装方法,#40A8F3,#fff %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

![](Swift_With_Android.png)

{% endhideToggle %}

<!-- endtab -->

<!-- tab @fab fa-apple-pay -->
**This is Tab 2.**

![](Swift_With_Android.png)
<!-- endtab -->

<!-- tab 炸弹@fas fa-bomb -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs test1 %}
<!-- tab 第一个Tab-->
**This is Tab 1.**

- 1
    - 11
    - 12
- 2
    - 22
- 3

{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}

{% hideToggle Butterfly安装方法,#40A8F3,#fff %}
在你的博客根目录里

git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly

![](Swift_With_Android.png)

{% endhideToggle %}

<!-- endtab -->

<!-- tab @fab fa-apple-pay -->
**This is Tab 2.**

![](Swift_With_Android.png)
<!-- endtab -->

<!-- tab 炸弹@fas fa-bomb -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

## 按钮

```markdown
{% btn 'https://github.com/Guang1234567/Swift_Coroutine_NIO2',项目地址 Guang1234567/Swift_Coroutine_NIO2,far fa-hand-point-right,larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,blue larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,pink larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,red larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,purple larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,orange larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,green larger %}
```

{% btn 'https://github.com/Guang1234567/Swift_Coroutine_NIO2',项目地址 Guang1234567/Swift_Coroutine_NIO2,far fa-hand-point-right,larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,blue larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,pink larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,red larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,purple larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,orange larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,green larger %}


```markdown
<div class="btn-center">
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline blue larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline pink larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline red larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline purple larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline orange larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline green larger %}
</div>
```

<div class="btn-center">
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline blue larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline pink larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline red larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline purple larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline orange larger %}
{% btn 'http://www.jerryc.me',JerryC,far fa-hand-point-right,outline green larger %}
</div>