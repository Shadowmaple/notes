## Socket编程

### 概述



### socket()

```c
int socket(int family, int type, int protocol);
```

出错返回-1，成功则返回一个非负的sockfd（socket file description，socket文件描述符）



### connect()

建立连接，进行三次握手，握手过程贯穿该函数的始终。

调用connect则将socket的状态从CLOSED转为SYN-SENT状态，若成功返回，则套接字状态变为ESTABLISHED状态。



### bind()



### listen()

该函数仅由服务器调用。

```c
# include <sys/socket.h>
int listen(int sockfd, int backlog);
// 成功则返回0，出错返回-1
// backlog 为该套接字指定的最大连接个数
```

listen函数调用会发生：

-   会把一个未连接的套接字从主动套接字变为被动套接字；
-   套接字的状态从CLOSED变为LISTEN状态。

>   关于被动套接字：
>
>   当socket()创建套接字时，是假设其为一个主动套接字的，即一个即将调用connect发起连接的客户端套接字。
>
>   而被动套接字则是会指示内核接受指向该套接字的连接请求。

backlog参数指定内核应该为该套接字排队的最大连接个数（半连接队列和已连接队列中数量的总和）。

接收到sync后，会将该套接字放入内核为该被动套接字维护的一个半连接队列，收到ACK，完成握手后，会放入已连接队列的队尾，同时accept()返回。

>   注：
>
>   listen函数是内核级的系统调用，不能人为传一个backlog值，只能通过修改系统一个backlog的参数。通常为5。
>
>   通常backlog为两个队列数量的总和，但不同的操作系统可能会超过这个值，比如最大连接数量为backlog的1.5倍因子。
>
>   backlog最好不要设为0，因为不同操作系统对此的解释不一样，如在Linux2.4.7中最大连接数为3，而MacOS10.2.6中为1。



### accept()

accept用于从已连接队列的队头返回下一个已完成连接socket，若队列为空，则进程进入睡眠（默认的阻塞方式）。

```c
# include <sys/socket.h>
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
// 成功返回非负描述符，出错则返回-1
// 除了sockfd是传入的参数，其它两个参数都是返回的
// cliaddr是客户进程的协议地址，addrlen是该地址的大小，都可为空指针
// 所以实际上accept返回三个值
```

accept返回的非负描述符是一个**已连接套接字描述符**（connect socket file description），是由内核自动生成的，代表与客户机的TCP连接，和传入的**监听套接字描述符**（listening socket fd）相区别。

监听套接字描述符sockfd存在于该服务器进程的整个生命周期，而已连接套接字描述符connfd只存在于该TCP连接中，完成特定服务关闭后就消失。



### fork()

fork()调用一次，返回两次，父进程返回子进程的pid，子进程返回0。

>   子进程可以通过getppid()获取父进程的pid，但父进程无法获取子进程的pid，所以需要记录子进程的pid。

服务器进程接受到accept()返回的connfd后，会fork一个子进程用来处理该客户请求，父进程和子进程会共享connfd，但之后父进程会关闭这个connfd，但子进程不会受到影响。



### close()

```c
# include <unistd.h>
int close(int sockfd);
```

用于显示地关闭一个套接字。每个套接字有一个引用计数，实际上是将套接字的引用计数-1，当connfd的引用计数为0的时候，就会发送FIN。

如果确实想直接在TCP连接上发送FIN，那么可以使用shutdown()

