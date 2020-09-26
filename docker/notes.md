# Docker

## 容器外部访问容器内

方法：

将容器内部端口暴露给外部，使用 -p 映射本机和容器内程序的端口

```shell
docker run -p 88:8081 grade_service_v2:0.2
```

`88:8081` 中左边是外部端口，右边是容器内端口

注：容器内的程序需要走 0.0.0.0 

## 容器内访问容器外部

方法：

将 docker 网络设为 host 的网络，让容器内部程序走本机 host 的网络，即可使用 localhost 访问外部 host 的程序。

```shell
docker run --network host grade_service_v2:0.2
```

## 容器间访问

...