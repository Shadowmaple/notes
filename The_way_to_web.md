# The Way To Web

##### 什么是Web

WWW可以让Web客户端（常用浏览器）访问Web服务器上的页面。 是一个由许多互相链接的超文本组成的系统，通过互联网访问。

在这个系统中，每个有用的事物(文字、图片、视频、声音等)，称为一个“资源”；并且由一个“统一资源标识符”（URI）标识；这些资源通过超文本传输协议（Hypertext Transfer Protocol，简称HTTP）传送给用户，而后者通过点击链接来获得资源。

### Internet 

Internet 不等于 Web ！！！

Web是Internet的一部分，虽然Web是Internet中最被人熟知的那一部分服务。

Internet是互联网，又称网际网路，或音译因特网、英特网，是网络与网络之间所串连成的庞大网络，这些网络以一组通用的协议相连，形成逻辑上的单一巨大国际网络。这种将计算机网络互相联接在一起的方法可称作“网络互联”，在这基础上发展出覆盖全世界的全球性互联网络称互联网，即是互相连接一起的网络结构。

今天，人们有时候比较容易混淆Internet和Web的概念，是因为现在越来越多的Internet的服务（e-mail，FTP，newsgroups等）都通过Web这个接口来呈现给用户，这些服务中的很多都已经整合到Web中。

![img](https://upload-images.jianshu.io/upload_images/1783214-d91f80bb55af2b41.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/540/format/webp)



### Web 的诞生

+ 1969年，美国国防部高级研究计划局（Advance Research Projects Agency, 简称ARPA）开始建立一个命名为ARPAnet的网络。ARPAnet被称为Internet的雏形。



Web由一个叫tim berners-lee (蒂姆·伯纳斯·李) 的人发明.

他在1989年发明了万维网，在1990年写出了第一个网页浏览器.

在1994年创立了著名的W3C（World Wide Web Consortium，万维网联盟）组织，因为他觉得Web发展迅猛，需要有一个类似基金会或委员会的机构来规范，以达成全球统一标准。W3C后来发明了一系列的语言和规范：HTML，CSS，XML等。近几年的HTML5也是他们规定的。

![img](https://upload-images.jianshu.io/upload_images/1783214-55d9c05075638cfb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/450/format/webp)



***



### 客户端与服务端

#### Client-Server模型

现在的万维网广泛采用的是Client-Server模型。Client在英语中是“客户”，“委托人”的意思，而Server是“服务器”，“服务者”的意思。

在信息技术领域，这种模型就是最著名和使用最广泛的“客户机-服务器”（Client-Server）模型。

客户机是什么呢？客户机就是获取服务的机器，一般就是我们自己的家用电脑、手机或平板电脑上等。而服务器通常是比较强大的电脑（当然了，你也可以用自己的个人电脑或者一个小小的树莓派（Raspberry PI）来搭建一个属于自己的服务器），专门服务我们的客户，一个服务器可以同时服务多个客户。

在Web领域，我们主要是用客户机来浏览网页，而客户机要能浏览网页，还必须装备一个叫做“浏览器”（英语是“browser”）的软件。

举例来说，当你的网页浏览器向维基百科请求一个指定的文章时，维基百科服务器从维基百科的数据库中找出所有该文章需要的信息，结合成一个网页，再发送回你的浏览器。



比较常用的浏览器有以下五个：Chrome，Firefox，IE，Safari，Opera。

![img](https://upload-images.jianshu.io/upload_images/1783214-8ea2a5260df8c0aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/737/format/webp)







![img](https://upload-images.jianshu.io/upload_images/1783214-2ba43665addd8ba9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/480/format/webp)

#### 客户端

我们的服务器和客户端要正常运行来为我们呈现网页，是需要借助一些编程语言的，毕竟客户端和服务器端运行的都是程序（Program）么。

上一部分讲到的W3C（由Web的发明人Tim Berners Lee创立的Web标准化机构）就制定了一系列统一的语言，主要是客户端的。

我们所看到的每一个网页，其实归根结底都是一个个文件。而我们的浏览器可以把这些文件解析成我们人类看得懂的各种样式：图片，文字，超链接，视频，音频等等。而这些网页文件本身是要由特定语言写成的。

客户端的语言有：HTML，CSS和JavaScript（简称JS）。

![img](https://upload-images.jianshu.io/upload_images/1783214-d6ea2c2fc457c380.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/501/format/webp)

+ HTML：HyperText Markup Language （超文本标记语言）的缩写。W3C制定的标记语言，用来表述网页的整体样式。“超文本”就是指页面内可以包含图片、链接，甚至音乐、程序等非文字元素。HTML不是一种编程语言，而是一种标记语言 (markup language)
+ 

<br>



+ CSS：Cascading Style Sheets（层叠样式表）的缩写。W3C制定的编程语言。既然叫“样式”表，那么它就是用于定义如何显示 HTML 元素。CSS使得HTML写成的页面不那么单调，可以有各种颜色，大小等。
+ JavaScript：一种脚本语言。与Java语言没有关系，不要因为看到名字中包含一个Java就以为JavaScript是Java的变体或者什么。可插入HTML页面，使网页具有动态/交互性。

浏览器要将由HTML，CSS和JS写成的网页文件翻译成我们用户能看懂的内容。过程类似如下：

![img](https://upload-images.jianshu.io/upload_images/1783214-f69eb863ed535b01.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/791/format/webp)

其实我们可以在我们所浏览的网页上点击鼠标右键，选择“查看网页源代码”，就可以看到被浏览器解释之前的这个页面原来的样子了：



![img](https://upload-images.jianshu.io/upload_images/1783214-79162b91c5305b83.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/146/format/webp)

那么我们一般的网页文件（主要由HTML语言写成，可能还包含了内嵌的CSS和JS，或者外部引用CSS和JS）的内容大致是什么样子的呢？

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
    <link rel="shortcut icon" href="https://mp.toutiao.com/static/resource/pgc_web/static/style/image/favicon.75200df.png" type="image/x-icon"/>
    <title>手动更新 - 头条号</title>
    <link rel="stylesheet" type="text/css" href="https://mp.toutiao.com/static/resource/pgc_web/static/pkg/common.c8103d9.css">
    <script type="text/javascript" charset="utf-8" src="https://mp.toutiao.com/static/resource/pgc_web/static/js/lib/pre.2dc26ef.js"></script>
  </head>
  <body >
    <div id="pagelet-header" gap="^用户profile">
    <div class="shead">
  </body >
</html>

```

以上就是一个网页文件的样例。可以看到它是HTML语言写成的，调用了css和javascript文件。

HTML这样的标记语言的一大特点就是有这样一对对的<>尖括号构成的结构，叫做tag（标签）。可以说HTML文件是由文本信息加标签组成，标签包裹了每一个文本，使得浏览器在翻译HTML文件时可以知道：“噢，这里是一个段落”，“噢，那里是一个标题”，“这里是一个超链接”，“那里是一张图片或一个视频”，等等：

```html
<p>这是一个段落。</p>
<h1>这是一个标题</h1>
```



#### 服务端

如果说客户端的语言编写的程序决定了我们的网页的外观，那么服务器端的语言编写的程序决定了网页的功能和如何与用户交互。你也许会问：“既然我们可以用HTML，CSS和JavaScript直接写出客户端的Web网站，那为什么还要多此一举用服务器端的语言来编写网站呢？”

首先我们来看两个概念:

+ 静态网页

```
在网站设计中，纯粹HTML格式的网页通常被称为“静态网页”，静态网页是标准的HTML文件，它的文件扩展名是.htm、.html，可以包含文本、图像、声音、FLASH动画等。静态网页是网站建设的基础，早期的网站一般都是由静态网页制作的。静态网页是相对于动态网页而言，是指没有后台数据库、不含程序和不可交互的网页。静态网页相对更新起来比较麻烦，适用于一般更新较少的展示型网站。容易误解的是静态页面都是htm这类页面，实际上静态也不是完全静态，他也可以出现各种动态的效果，如GIF格式的动画、FLASH、滚动字幕等。
```



+ 动态网页

```
动态网站并不是指具有动画功能的网站，而是指网站内容可根据不同情况动态变更的网站，一般情况下动态网站通过数据库进行架构。 动态网站除了要设计网页外，还要通过数据库和编程来使网站具有更多自动的和高级的功能。动态网站服务器空间配置要比静态的网页要求高，费用也相应的高，不过动态网页利于网站内容的更新，适合企业建站。动态是相对于静态网站而言。
```

简单来说，静态网页，你一旦用HTML和CSS写好，上传到服务器空间，以后每个用户访问你的网址看到的网页都是一样的。

动态网页展示给每个用户一般是不一样的，例如可以注册用户的那些网站，肯定是动态网页。因为你登录后就看到自己的信息，其他人登录则看到他们自己的信息。

与客户端不同的是，服务器端没有一种语言是必须使用的。对于客户端来说，HTML语言是必须的。对于服务器端，我们可以选择适合自己的编程语言来开发。

常见的服务器端编程语言有：

1. PHP
2. Java
3. Python
4. Ruby
5. C#

等等. 很多语言都可以用于写后端.



#### 前后端合并与前后端分离的开发方式

前后端合并的开发方式，直白来说就是服务端在程序中读取数据库之后，根据写好的模板文件，在模板中填充数据库中查询的信息，来生成一个web页面，之后返回给客户端.

而前后端分离的开发方式是指，前端和后端分开来部署。其中前端也需要用服务端语言写一些程序，专门用于请求后端；而后端一般通过json的数据格式将信息返回给前端；前端拿到信息之后，填充到自己的页面中返回给客户端。

前后端分离的好处有很多，最重要的是前后端解耦了，前端可以使用自己喜欢的框架来写，而后端也可以用自己喜欢的语言来构建，只要双方约定好信息交换的接口即可无缝对接。

