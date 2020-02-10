# Docker

# 镜像

## 获取

```shell
docker pull [选项] [Docker Registry 地址]/仓库名:标签
```

## 顶层镜像

```shell
docker images
docker image ls
```

## 中间层镜像

```shell
docker images -a
```

## 虚悬镜像

```shell
# 列出
docker images -f dangling=true

# 删除
docker image prune
docker rmi $(docker images -q -f dangling=true)
docker image rm $(docker images -q -f dangling=true)
```

## 构建镜像

```shell
docker build [选项] <上下文路径/URL/->

docker build -t homeworks:v1 .
docker build -t homeworks:v1 -f Dockerfile .
```

上下文路径：不是本地目录的上下文路径，而是在Docker引擎中展开的上下文。

>   当构建的时候，用户会指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

不指定构建的文本，便默认为本目录下的`Dockerfile`文件

其它构建方法：

+   Git repo（url中构建）

+   tar 压缩包构建

+   从标准输入中读取上下文压缩包进行构建

+   从标准输入中读取 Dockerfile 进行构建

    ```shell
    docker build - < Dockerfile
    cat Dockerfile | docker build -
    ```

