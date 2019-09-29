# Docker

https://docs.docker.com/install/linux/docker-ce/ubuntu/
[官方教程](https://docs.docker.com/install/linux/linux-postinstall/)

https://yeasy.gitbooks.io/docker_practice/introduction/what.html

列出镜像
```shell
docker image ls
docker images
```

删除镜像
```shell
docker image rm 501
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
docker 内的程序 ip 要用 **0.0.0.0** !!!

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