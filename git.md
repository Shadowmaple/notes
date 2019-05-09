# git clone速度太慢解决方案
速度由9kb/s提高到16-40kb/s，亲测有效

查找域名对应的ip地址，并修改hosts文件
```
$ nslookup github.global.ssl.fastly.Net
$ nslookup github.com 
```
将下列内容加入 /etc/hosts文件中
```
151.101.xx.xxx http://global-ssl.fastly.net
xxx.xxx.xxx.xxx http://github.com
```
刷新DNS缓存
```
$ sudo /etc/init.d/networking restart
```

# git pull 提示拉取成功，但本地代码却没有拉下来，没有更新
git stash 将本地修改储存起来，然后再Git pull 就可以了呢