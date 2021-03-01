# HTTPS

HTTPS，全称为 HyperText Transfer Protocol over Secure Socket Layer

HTTPS = HTTP + SSL/TLS

SSL（Secure Socket Layer，安全套接字层）：1994年为 Netscape 所研发，SSL 协议位于 TCP/IP 协议与各种应用层协议之间，为数据通讯提供安全支持。

TLS（Transport Layer Security，传输层安全）：其前身是 SSL，它最初的几个版本（SSL 1.0、SSL 2.0、SSL 3.0）由网景公司开发，1999年从 3.1 开始被 IETF 标准化并改名，发展至今已经有 TLS 1.0、TLS 1.1、TLS 1.2 三个版本。SSL3.0和TLS1.0由于存在安全漏洞，已经很少被使用到。TLS 1.3 改动会比较大，目前还在草案阶段，目前使用最广泛的是TLS 1.1、TLS 1.2。

## 数字证书 Certificate

数字证书是为了确保对方（服务器）发送的公钥是正确的，需要一个第三方权威机构（Certificate Authority）进行担保背书。

包含的内容：公钥、证书所有者、颁发机构、有效期

证书是使用 CA 的私钥进行加密的，在校验的时候需要使用 CA 的公钥进行解密。

校验证书需要 CA 的公钥，那怎么确保 CA 的公钥是正确的？那也就需要校验 CA 的数字证书，需要更加权威的机构进行担保。如果还不信任该机构，那就层层向上，一直到最顶级的几个大 CA（root CA），而这几个 CA 是默认保存在操作系统上的。

签名就是在信息的后面再加上一段内容（信息经过hash后的值），可以证明信息没有被修改过。hash值一般都会加密后（也就是签名）再和信息一起发送，以保证这个hash值不被修改。

## 工作模式

**HTTPS 混合使用了非对称加密和对称加密，约定对称加密的密钥时使用非对称加密，之后的通信皆采用对称加密。**

工作过程：

1.  C->S，发送 Client Hello 消息，明文传输 SSL/TLS 版本信息、加密算法、压缩算法、一个随机数 c 等信息（发送约定的加密方法）
2.  S->C，返回 Server Hello 消息，选择使用的协议版本、加密算法、压缩算法，以及一个随机数 s
3.  S->C，发送服务器的证书 certificate
4.  S->C，发送 Server Hello Done 消息
5.  Client 校验证书，获取公钥
6.  Client 产生一个随机数字 pre-master
7.  C->S，将随机数字 pre-master 公钥加密传输
8.  Server 使用私钥解密，获取随机数字 pre-master。
9.  client 和 server 同时使用**随机数c + 随机数s + pre-master** 生成对称密钥。
10.  C->S，发送“Change Cipher Spec”消息，通知以后用协商的通信密钥和加密算法进行通信
11.  C->S，发送Encrypted Handshake Message” 消息，将商定好的参数通过协商密钥进行发送测试
12.  S->C，发送“Change Cipher Spec”消息，表示以后用协商的密钥和加密算法进行通信
13.  S->C，发送Encrypted Handshake Message” 消息，将参数发送进行测试
14.  SSL 握手完成
15.  HTTP 的流程……



>   上面的过程只包含了HTTPS 的单向认证，也即客户端验证服务端的证书，是大部分的场景，也可以在更加严格安全要求的情况下，启用双向认证，双方互相验证证书。

