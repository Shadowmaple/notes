# wine QQ
在Linux上运行qq

> [Linux 上真正完美稳定的 wine QQ 方案](https://linux265.com/news/2674.html)



解决在非中文环境下中文无法正常显示的问题：

打开 `/opt/deepinwine/tools/run.sh` 文件，修改下述内容：

```shell
WINE_CMD="LC_ALL=zh_CN.UTF-8 deepin-wine"
```



经过体验，确实可以很顺畅地运行，并且发消息之类的功能大多都比较完善。

但要注意的是，这一套下来占用的磁盘空间会很大，并且运行时占用的内存也会很大（占用CPU 20%-60%），慎用

