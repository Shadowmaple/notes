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

# 修改git链接的url地址
在仓库根目录下进入`.git/config`文件，修改 url 即可
```
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    url = https://github.com/Shadowmaple/notes
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

# git commit 修改最后一次提交
修改你最近一次提交可能是所有修改历史提交的操作中最常见的一个。 对于你的最近一次提交，你往往想做两件事情：修改提交信息，或者修改你添加、修改和移除的文件的快照。

如果，你只是想修改最近一次提交的提交信息，那么很简单：
```
$ git commit --amend
```
这会把你带入文本编辑器，里面包含了你最近一条提交信息，供你修改。 当保存并关闭编辑器后，编辑器将会用你输入的内容替换最近一条提交信息。

如果你已经完成提交，又因为之前提交时忘记添加一个新创建的文件，想通过添加或修改文件来更改提交的快照，也可以通过类似的操作来完成。 通过修改文件然后运行 `git add` 或 `git rm` 一个已追踪的文件，随后运行 `git commit --amend` 拿走当前的暂存区域并使其做为新提交的快照。

使用这个技巧的时候需要小心，因为修正会改变提交的 SHA-1 校验。如果已经推送了最后一次提交就不要修正它。
