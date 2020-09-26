# Docker

## 文档

>   https://vuepress.mirror.docker-practice.com/
>
>    https://docs.docker.com/install/linux/docker-ce/ubuntu/
>
>   [官方教程](https://docs.docker.com/install/linux/linux-postinstall/)

## 基本操作

列出镜像

```shell
docker image ls
docker images
```

删除镜像
```shell
docker image rm <image id>
docker rmi <image id>
```

制作镜像
```shell
docker build -t homeworks:v1 .
# docker build -t homeworks:v1 -f Dockerfile .

# 启动
docker run -p 8000:5000 -it --rm homeworks:v1
docker run -p 8000:5000 --name HomeWorks -it --rm homeworks:v1
docker run -p 8000:5000 --name HomeWorks -it --rm homeworks:v1 flask run
```
docker 内的程序暴露的 ip 地址要用 **0.0.0.0** !!!

清除所有镜像文件
```shell
sudo service docker stop
sudo rm -rf /var/lib/docker
sudo service docker start
```

Dockerfile例子，以HomeWorks为例：
```dockerfile
# FROM python:3.6-alpine
FROM python:3.6
RUN mkdir /app
ADD . /app
WORKDIR /app
EXPOSE 5000
RUN pip3 install -r requirements.txt
CMD flask run -h 0.0.0.0 -p 5000
```

查看容器

```shell
# 正在运行的容器
docker ps

# 全部容器
docker ps -a
```

停止运行

```shell
docker stop [CONTAINER ID]
```

删除容器

```shell
docker rm [CONTAINER ID]

# 删除全部
docker rm $(docker ps -a -q)
```

ps：容器停止后还存在，想删除镜像时要先删除容器。

## 镜像

### 获取

```shell
docker pull [选项] [Docker Registry 地址]/仓库名:标签
```

### 顶层镜像

```shell
docker images
docker image ls
```

### 中间层镜像

```shell
docker images -a
```

### 虚悬镜像

```shell
# 列出
docker images -f dangling=true

# 删除
docker image prune
docker rmi $(docker images -q -f dangling=true)
docker image rm $(docker images -q -f dangling=true)
```

### 构建镜像

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

