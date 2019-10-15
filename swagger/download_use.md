# swagger本地搭建

除了可以[在线编辑](https://link.jianshu.com/?t=https://editor.swagger.io//?&_ga=2.118897038.641439096.1505787945-1994390625.1504744265#/)swaggerAPI文档，还可以通过容器在本地进行编辑，并且也提供了swagger-ui的容器

使用docker

```shell
docker pull swaggerapi/swagger-editor 
docker run -d -p 88:8080 swaggerapi/swagger-editor 
```

88:8080 将容器的8080端口暴露给localhost的88端口，在浏览中输入：localhost:88，就可以在容器中编辑api文档

参考：

+   https://github.com/swagger-api/swagger-editor

+   https://www.jianshu.com/p/103ae5f80691

