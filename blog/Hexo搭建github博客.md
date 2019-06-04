# 什么是 Hexo？
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

搭建过程或许有那么点小繁琐~~（其实挺麻烦的）~~，但一旦搭建完成，发表博文是极简单，极舒服的。
只需要几个简单命令，你就可以完成一切。
```shell
$ hexo n #写文章
$ hexo g #生成
$ hexo d #部署，可与hexo g合并为 hexo d -g
```
# 前期准备
+ 已安装git
+ 已有github账号
+ 已有node.js

# 安装npm
可以直接apt安装，也可以去官网下载node.js包，然后解压安装。在此用apt安装：
```shell
$ sudo apt-get install npm 
```
因为npm的源在国外，没翻墙的话速度会很慢（感觉翻墙了效果也不是很大），可以改成国内的淘宝源
```shell
$ npm config list   #查看配置
$ sudo npm config set registry https://registry.npm.taobao.org  #换源
```

# hexo本地搭建
## 安装hexo
所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。一条命令即可解决：
```shell
$ sudo npm install -g hexo-cli
```

如果 npm 安装 hexo 失败, 则很有可能是权限问题, 或者npm与node的版本不兼容（很少出现）
> [npm install安装失败问题](https://blog.csdn.net/cc18868876837/article/details/81542282)

## Hexo初始化
创建一个目录用来作为你的 blog 目录，并在该目录中进行Hexo的初始化：
```shell
$ mkdir <folder>
$ cd <folder>
$ hexo init
```
也可以直接
```shell
$ hexo init <folder>
```
等一会儿如果出现橙色的 `WARN` 没关系，只要不出现红色的 `ERROR` 就行
执行后，可能会出现如下的警告，安装依赖项失败，则要手动运行 npm 安装
```shell
WARN  Failed to install dependencies. Please run 'npm install' manually!
```
手动运行安装(直接输入即可)
```shell
$ npm install
```
这样，hexo会帮你在该目录下生成相应的各种文件：
```shell
.
├── _config.yml
├── package.json
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
|   ├── _drafts
|   └── _posts
└── themes
    └── landscape
```
至此，完成了Hexo的初始化
## 本地启动
Hexo 3.0 把服务器独立成了个别模块，但要先安装 hexo-server 才能使用。
```shell
$ sudo npm install hexo-server --save
```
之后就可以进行本地的预览了 
```shell
$ hexo g    #等同于hexo generate, 生成静态文件
$ hexo s    #等同于hexo server, 在本地服务器运行
```
指定端口
```shell
$ hexo s -p 2333
```
至此，Hexo博客已经在本地成功搭建
停止：`ctrl+c`

# 将本地Hexo博客推送到GithubPages
## 创建仓库
仓库名为：`<Github账号名称>.github.io`，github会自动将它识别为Ｇithub Pages所属的仓库，并开启博客的站点链接

## 安装 hexo-deployer-git 插件
```
$ sudo npm install hexo-deployer-git --save
```
## 修改站点目录下的`_config.yml`
在文件末尾修改为：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:<Github账号名称>/<Github账号名称>.github.io.git
  branch: master
```
注：上面的地址可以是ssh地址，也可以是https地址
## 添加SSH key
### 生成ssh密钥
输入以下命令，回车三下（默认值）即可：
```
$ ssh-keygen -t rsa -C "邮箱地址"
```
### 将密钥添加到 github
1. 打开github --> Settings --> SSH and GPG keys --> New SSH key
2. 复制密钥文件内容（路径：`~/.ssh/id_rsa.pub`），粘贴到 New SSH Key 打开的 “Key” 之中即可，title 可随意，可不填。
3. 测试是否添加成功，在命令行依次输入以下命令，返回`You’ve successfully authenticated`即成功：

    ```
    $ ssh -T git@github.com
    ```

## 推送到GithubPages
输入以下命令， 返回`INFO Deploy done: git`即成功推送
```shell
$ hexo g
$ hexo d
```
等待1分钟左右，访问博客地址即可： `https://<Github账号名称>.github.io`

可以看到初始便有一篇《Hello world》文章

## 错误分析
+ 当遇到部署出错或部署后的网页存在偏差时，建议先清理下缓存：`hexo clean`

我遇到 Permission denied 问题，可能你们也会踩到这个坑
> [Permission denied问题](https://blog.csdn.net/Zzz_Zzz_Z/article/details/80730921)
[针对github权限导致hexo部署失败的解决方案](http://www.cnblogs.com/xsilence/p/6001938.html)

还有这一篇[博文](https://www.simon96.online/2018/10/12/hexo-tutorial/)关于错误分析罗列得比较全

## hexo指令
```
hexo help       #查看帮助
hexo init       #初始化一个目录
hexo clean      #清除缓存，最好每次执行命令前先清理缓存
hexo generate   #生成网页，可以在 public 目录查看整个网站的文件
hexo server     #本地预览，'Ctrl+C'关闭
hexo deploy     #部署.deploy目录
hexo new "postName"         #新建文章
hexo new page "pageName"    #新建页面
```
# 基础配置
## 配置站点信息
### 更改标题，作者，语言等
修改站点配置文件(`_config.yml`)，以博主的设置为例
```
# Site
title: 玄著 
subtitle: 记录点滴  #副标题，在主标题之下显示
description: Coding everyday, change everyday   #描述，在”作者“之下显示
keywords:           #站点关键字，可用于被搜索引擎捕获
author: Shadow Zhang    #作者名
language: zh-CN     #语言，简体中文，更多请见官方文档
timezone:           #时区
```
这样就算成自己的博客了2333
# 写博文
## 新建文章
执行命令，生成指定名称的文章至`source/_posts/postName.md`
```
$ hexo new [layout] "postName"
```
其中layout是可选参数，默认值为post。有哪些 layout 呢，请到 scaffolds 目录下查看，这些文件名称就是 layout 名称。当然你可以添加自己的 layout，方法就是添加一个文件即可，同时你也可以编辑现有的 layout，比如 post 的 layout 默认是`scaffolds/post.md`

## Front-matter
一开始新建起来的是这样的
```
title: postName             #文章标题，可以修改，但最好不要改，后面会说到
date: 2019-04-13 19:43:07   #文章生成时间，可以任意修改
`tags`:                     #文章标签，可为空
---
这里开始使用markdown格式输入你的正文。
```
多个标签（yaml格式）
```yaml
'tags':
  - python
  - linux
```
我们可以添加其它参数，如更新时间，分类等，详见[官方文档](https://hexo.io/zh-cn/docs/front-matter.html)

## 文章部分显示设置
打开主页，我们会发现主页显示的文章都是整篇显示的，这样子不太符合我们的需求。

### `<!-- more -->`方法
通常情况下，在文章中可以使用 `<!-- more -->` 手动进行截断，这是Hexo提供的方式。这种方式可以精确控制需要显示的摘录内容，也可以让 Hexo 中的插件更好地对文章进行识别，比如markdown的显示。

使用：在需要的地方插入`<!--more-->`即可。

### 其它方法
除了Hexo提供的传统方法，大多主题都提供有其它的方式，比如 NexT 主题，提供了另外两种方式：

1. 在文章的 front-matter 中添加 description，输入文章摘录即可
2. 自动形成摘要，将主题配置中的`"auto_excerpt`项，改为true即可
```
auto_excerpt:
  enable: true
  length: 150   #默认截取的长度为，可以自行设定
```
# 更改主题
网上主题有很多，前人栽树，后人乘凉，我们只要选择一个适合自己的主题即可。可以查看[主题列表](https://hexo.io/themes/)，但是更多的主题未被官方收录。

我选择的主题是NexT，简洁美观，有三种外观，并且现在已经更新到了v7.1了，内部已经集成了很多模块，可以很方便地实现一些动能而无需扒前端代码（卑微）。另外还有其它人气比较高的主题，诸如：material，[pacman](https://github.com/A-limon/pacman)，[modernist](https://github.com/orderedlist/modernist)，[yilia](https://github.com/litten/hexo-theme-yilia)，[indigo](https://github.com/yscoder/hexo-theme-indigo)。不多说，进入正题。

更换主题其实很简单，只要克隆主题——更改设置，即可，简单吧，~~呵呵~~，但实践起来还是会遇到麻烦
## 克隆主题
进入该主题的github仓库，比如[Next](https://github.com/theme-next/hexo-theme-next)。不过可能会遇到一个主题有两个仓库的情况，比如next还有一个[仓库](https://github.com/iissnan/hexo-theme-next)，但是看它的发行版本，只有v5.14，并也宣告该仓库已停止维护。所以下载哪个，看各自的版本吧。

在hexo顶级目录下，克隆到 themes 目录
```shell
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

可能在克隆过程中会因为速度过慢而失败，（此坑已踩，9kb/s...），解决办法参考[此文](https://blog.csdn.net/hzwwpgmwy/article/details/79043251)

## 启用主题
打开站点配置文件（_config.yml）, 找到 theme 字段, 并将其值更改为 next
```
theme: next
```
## 更改样式
NexT有四种模式（其实只有三种，第三和第四个差不太多），直接更改主题配置文件（`themes/next/_config.yml`）的 scheme 参数即可
```
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```
然后`hexo s`即可预览效果

# 主题优化(基于next)
## 添加「标签」和「分类」页面
（以添加标签页面为例）

新建标签页面
```shell
$ hexo new page tags
```
编辑刚新建的页面，将页面的类型设置为 tags ，主题将自动为这个页面显示标签页。文件内容如下：
```
title: tags
date: 2019-04-13 16:31:39
type: "tags"
---
```
在菜单中添加链接。编辑 主题配置文件 ， 添加 tags 到 menu 中
```
menu:
  home: / || home 
  #about: /about/ || user
  tags: /tags/ || tags 
  #categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```
添加“分类”或其它的菜单同理，只需把 tags 改为 categories 或其它即可。

若只是修改了主题配置文件（此坑亲测），那么点开 tags 链接会出现这样的画面：
![tags not found][1]

## 添加头像
在`themes/next/_config.yml`件中搜索 avatar ：
```
# Sidebar Avatar
avatar:
  # In theme directory (source/images): /images/avatar.gif
  # In site directory (source/uploads): /uploads/avatar.gif
  # You can also use other linking images.
  url: /uploads/avatar.jpg
  # If true, the avatar would be dispalyed in circle.
  rounded: true     # true为圆形，false为方形
  # The value of opacity should be choose from 0 to 1 to set the opacity of the avatar.
  opacity: 1        # 不透明度，0-1，0为透明
  # If true, the avatar would be rotated with the cursor.
  rotated: false    # true为旋转
```
头像图片可以放在主题目录下，也可以放在站点目录下，我选择放在站点目录下。在站点目录的 source 目录中新建 uploads 目录，用来存放头像，url 为图片地址

+ 注意：如果图片不是正方形的话，那么 rounded 参数设为 true 的话会变成椭圆形哦（亲测..）

## 返回文章顶部并显示当前浏览进度
修改`themes/next/_config.yml`，把 false 改为 true：
```
# Back to top in sidebar
b2t: true       # 返回文章顶部

# Scroll percent label in b2t button
scrollpercent: true     # 当前浏览进度
```
## 侧边栏社交小图标设置
打开主题配置文件`_config.yml`，搜索 Social，去掉你要添加的图标前面的#号，也可以自己添加其它的社交地址。其格式为：
```
[社交平台名]: [社交地址] || [图标名称]
```
```
social:
  GitHub: https://github.com/Shadowmaple || github
  E-Mail: mailto:shdwzhang@gmail.com || envelope
  #Weibo: https://weibo.com/yourname || weibo
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype
  
# 图标设置
social_icons:
  enable: true 
  icons_only: false     #只显示图标
  transition: false
  GitHub: github
```
+ 图标可以去[Font Awesome Icon](https://fontawesome.com/icons?from=io)网站去找，找到后复制名字到相应的位置即可。

## 文章代码主题设置
NexT 共有5款主题供你选择，默认使用的是 白色的 normal 主题，可选的值有 normal，night， night blue， night bright， night eighties。
在主题文件夹的 `_config.yml` 配置文件中，定位到 `highlight_theme`，根据需求修改相应的值即可

## 添加nest动态背景特效
背景的几何线条是采用的nest效果，一个基于html5 canvas绘制的网页背景效果，非常赞！来自github的开源项目[canvas-nest](https://github.com/hustcc/canvas-nest.js)

效果图：
![canvas动态效果图][2]

而幸运的是，next主题已经内部集成了这个模块，所以就不用那么麻烦地去改js代码了
> [查看教程](https://github.com/theme-next/theme-next-canvas-nest)

## 主页文章添加边框阴影效果
其它主题都可以这样设置。打开 `themes/*/source/css/_custom/custom.styl` ,向里面加代码:
```
// 主页文章添加阴影效果
.post {
   margin-top: 0px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```
## 修改底部标签样式
修改`themes/next/layout/_macro/post.swig`文件，搜索`rel="tag">#`，将`#`替换成`<i class="fa fa-tag"></i>`。查看效果：
![底部样式效果][3]

如果不成功，则把原来的静态资源给清除掉（`hexo clean`），重启服务。

## 底部隐藏由Hexo强力驱动、主题--NexT.Mist
在主题配置文件中，搜索 powered，修改 enable 即可
```
  powered:
    # Hexo link (Powered by Hexo).
    enable: false
    # Version info of Hexo after Hexo link (vX.X.X).
    version: true 

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: false 
    # Version info of NexT after scheme info (vX.X.X).
    version: true 
```
> 更多酷炫效果请见参考文档

# 参考文档
+ [Hexo官方文档](https://hexo.io/zh-cn/docs/)
+ [NexT官方文档](https://theme-next.org/docs/getting-started/)
+ [【持续更新】最全Hexo博客搭建+主题优化+插件配置+常用操作+错误分析](https://www.simon96.online/2018/10/12/hexo-tutorial/)
+ [码云+Hexo搭建个人博客+评论功能接入](http://zwd596257180.gitee.io/blog/2019/04/15/hexo_manong_bog/)
+ [打造个性超赞博客Hexo+NexT+GitHubPages的超深度优化](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html#fn:2)

  [1]: https://upload-images.jianshu.io/upload_images/3072214-a1cce579d49cb369.png?imageMogr2/auto-orient/
  [2]: https://yixiuer.oss-cn-shanghai.aliyuncs.com/images/hexo-next-optimization-2.gif
  [3]: https://qqadapt.qpic.cn/txdocpic/0/f30051c7389908eb2da8aecf0fe8af5a/0

