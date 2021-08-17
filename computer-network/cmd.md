
查询域名对应的IP地址：
```shell
nslookup
```



在linux中如何查看知名端口号？

```shell
cat /etc/services
```



查看TCP状态数

```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

查看TCP连接状态信息

```shell
ss
```

