## 检测邮箱是否真实存在

例如验证 `shdwzhang@163.com` 是否真实存在

1. `nslookup -type=MX 163.com` 查找`163.com`的MX地址

```shell
$ nslookup -type=MX 163.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
163.com	mail exchanger = 10 163mx01.mxmail.netease.com.
163.com	mail exchanger = 10 163mx02.mxmail.netease.com.
163.com	mail exchanger = 10 163mx03.mxmail.netease.com.
163.com	mail exchanger = 50 163mx00.mxmail.netease.com.

Authoritative answers can be found from:

```

2. `telnet 163mx01.mxmail.netease.com 25` 远程连接

```shell
$ telnet 163mx01.mxmail.netease.com 25
Trying 220.181.14.137...
Connected to 163mx01.mxmail.netease.com.
Escape character is '^]'.
220 163.com Anti-spam GT for Coremail System (163com[20141201])

```

3. `helo mannix` 发送 `helo` 给smtp服务器

```shell
Trying 220.181.14.137...
Connected to 163mx01.mxmail.netease.com.
Escape character is '^]'.
220 163.com Anti-spam GT for Coremail System (163com[20141201])
helo mannix
250 OK
```

4. `mail from:<shdwzhang@gmail.com>` 指明发送者


```shell
mail from:<shdwzhang@gmail.com>
250 Mail OK
```

5. `rcpt to:<shdwzhang@163.com>` 发送 `rcpt to` 命令，验证是否存在

```shell
rcpt to:<shdwzhang@163.com>
250 Mail OK
```

