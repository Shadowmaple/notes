# Linux的文件压缩

# 文件压缩

## gzip

```shell
$ gzip [option] {filename}

选项和参数：
-c	: 将压缩数据输出到屏幕上
-d	: 解压缩
-k	: 保留原始文件
-v	: 显示压缩信息
-n	: 1-9的数字，代表压缩等级，1最差，9最好，默认为6

# 压缩文件
# 这样之后压缩的文件会覆盖原始文件
$ gzip example.md

# 压缩文件，显示压缩信息，并保留源文件
# 第一种方法
$ gzip -kv example.md
# 第二种方法，使用数据流重定向
$ gzip -cv example.md > example.md.gz

# 解压缩文件
$ gzip -dv example.md.gz

# 使用9级压缩
$ gzip -9v example.md
```

一般来说无论压缩还是解压缩，默认都是替换原有文件，要想保留原始文件就必须另加选项或参数。

至于压缩等级，虽然9级压缩率最好，但是时间成本会比较高，酌情使用。



## bzip2

## xz



# 打包压缩

