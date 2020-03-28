# “apt” not found —— 寻踪记

# 迷雾

在安装Docker的时候，用apt安装输入`sudo apt-get install apt-transport-https`，安装成功`apt-transport-https`后执行下一步才惊觉—— apt 没了！怎么回事？！

查看 apt 的 log 文件，才发现竟然在`apt-transport-https`的安装过程中，把 apt 给移除了！并且还有apt的一些相关依赖，以及 vscode —— 这什么鬼？

```verilog
Start-Date: 2019-08-19  14:09:20
Commandline: apt-get install apt-transport-https ca-certificates curl software-properties-common
Requested-By: lawler (1000)
Install: apt-transport-https:amd64 (1.2.32)
Remove: update-notifier-common:amd64 (3.192.1.5), apt:amd64 (1.6.10), update-manager:amd64 (1:18.04.11.9), ubuntu-desktop:amd64 (1.417), flashplugin-installer:amd64 (32.0.0.223ubuntu0.16.04.1), ttf-mscorefonts-installer:amd64 (3.6ubuntu2), ubuntu-minimal:amd64 (1.417), apt-utils:amd64 (1.6.10), code:amd64 (1.37.0-1565227985), ubuntu-release-upgrader-gtk:amd64 (1:18.04.30), update-notifier:amd64 (3.192.1.5)
End-Date: 2019-08-19  14:10:11
```

有点心态爆炸，难怪无论怎么查找，都找不到 apt 的 可执行文件，所幸一些配置文件、log文件等都还有保留。既然知道了问题所在，那么就要想办法解决——把 apt 给装回来喽！还好，apt 还是能装回来的，否则只能重装系统了。。。

# 出路

## 获取镜像

进入[阿里开源镜像网站](https://opsx.alibaba.com/mirror/search?q=apt&lang=zh-CN)，根据当前系统版本和架构选择合适的 apt 镜像文件（格式为：apt_xxx_xxx.deb），我的计算机是ubuntu 18.04，x86架构，因而选择 ubuntu bionic 的`apt_1.6.1_amd64.deb`的镜像文件下载。

使用如下命令可以查看当前 Linux 系统的版本信息

```shell
$ cat /etc/os-release
```

不过 1.6 版本的 apt 算是比较老的了，阿里上面没有更新的版本，可以去网易或搜狐上面找，根据系统的不同将网址更改下即可。

> 网易开源镜像：http://mirrors.163.com/ubuntu/pool/main/a/apt/
>
> 搜狐开源镜像：http://mirrors.sohu.com/ubuntu/pool/main/a/apt/

不过要注意，不是下载的镜像越新越好，因为 apt 的安装运行也是有很多依赖的，版本的差异会导致一些比较大的问题。就比如下载了一个很新1.9.0 的 apt 版本，但在安装的时候会报错。

## 安装

使用 dpkg 进行安装，一定要用 root 权限。

```shell
$ sudo dpkg -i apt_1.6.1_amd64.deb
```

如果没什么问题，就表示 apt 重新归来了。可以 dpkg 查看下

```shell
$ dpkg -l apt
```



## 重装 apt-transport-https

之前为了合 apt 的版本，将 `apt-transport-https`给卸载了，现在我要重新安装。

还是之前的操作：

```shell
$ sudo apt-get install apt-transport-https
```

发现原来安装的时候已经有提示了……不看安装说明真不是一个好习惯，尤其是让你输入什么确认的时候……



![](https://raw.githubusercontent.com/Shadowmaple/mydocuments/master/images/Screenshot/001.png)



既然这样用 apt 安装`apt-transport-https`会把 apt 给移除掉，那么该如何安装 呢？apt 怎么安装，它就怎么安装。在之前的镜像网站中也可以看到有 `apt-transport-https` 的镜像文件，选择合适的下载下来即可，dpkg 安装

```shell
$ sudo dpkg -i apt-transport-https_1.8.3_all.deb
```

如果有什么问题，比如版本的问题，那就再下载其它版本的试试。年轻人就要多折腾～



## apt 相关软件包

虽然 apt已经重新安装好了，但是并不代表它是完整的，从之前的log文件中可以看到随 apt 移除的还有 `update-manager`，`flashplugin-installer` 等等，这些东西的缺失会使你在安装其它一些软件的时候受到阻碍，那么该如何恢复？现在就不用从镜像网站一个个下载了，直接一条指令便足以解决：

```shell
$ sudo apt --fix-broken install
```



![](https://raw.githubusercontent.com/Shadowmaple/mydocuments/master/images/Screenshot/003.png)



还有之前也同样被移除的 vscode ，就要自己去官网下重新下载了

这样 apt 完全修复了



# 寻踪启示

引以为鉴，在安装什么东西的时候稍微看下安装的说明，尤其是要确认什么的时候

附上一些有用的参考链接：

> [中国Linux开源镜像站大全](https://bbs.aliyun.com/simple/t345645.html)
>
> https://www.bbsmax.com/A/gVdnYx2X5W/
>
> https://blog.csdn.net/qq_38417808/article/details/85602476



最后附上一张某些人嘲笑我的证明：

![](https://raw.githubusercontent.com/Shadowmaple/myDocuments/master/images/Screenshot/from_phone/001.jpg)
