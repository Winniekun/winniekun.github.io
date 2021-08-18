---
title: https总结
tags: 应用层
categories: 网络
mathjax: false
date: 2021-05-21 22:32:09
---


## 序

引入一个新的概念的时候，必然先阐述其相关方面已有的实现有哪些缺点，新的概念解决了哪些缺点。所以～让我们看看HTTP有哪些却缺点

**缺点：**

1. 明文传输，传输信息容易被窃听 --- **窃听风险**
2. 不验证通信方的身份，因此可能会遭遇伪装 --- **冒充风险**
3. 无法验证报问的完整性，所以通信的信息可能已遭篡改 --- **篡改风险**
   1. 譬如强行植入的广告信息

而HTTPS的出现就正好解决了上述的三个问题。对于窃听风险使用加密措施解决，对于冒充风险使用证书认证解决，对于篡改风险使用完整性保护解决（信息摘要算法）

## HTTPS概述

![https](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521144232642.png)

HTTPS其实就是**http secure**的意思。HTTP是应用层协议，位于HTTP协议之下是传输协议TCP。TCP负责传输，HTTP则定义了数据如何进行包装。所以HTTP传输给TCP什么样的数据包，其都会原样的发送个通讯方（仅仅将数据包分为了多个报报问段）。

> HTTP --> TCP （明文传输）

而HTTPS相对于HTTP有哪些不同呢？其实就是在HTTP和TCP中间加多了一层加密层**TLS/SSL**。

> 通俗的讲，TLS、SSL其实是类似的东西，SSL是个加密套件，负责对HTTP的数据进行加密。TLS是SSL的升级版。现在提到HTTPS，加密套件基本指的是TLS。

原先是应用层将数据直接给到TCP进行传输，现在改成应用层将数据给到TLS/SSL，将数据加密后，再给到TCP进行传输。如下图所示

![https和http传输过程](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521144940926.png)

所以HTTPS本身并不是应用层的一个新的协议，只是HTTP在和TCP进行交接的中间，又新增了一层SSL/TLS协议。所以HTTPS可以理解为披着SSL/TLS协议外壳的HTTP而已。



## HTTPS的加密---混合加密

首先需要了解`对称加密`、`非对称加密`的区别。我们知道实际对通信加密的是SSL/TLS层。SSL/TLS采用是混合加密的机制，也就是对称加密和非对称加密的混合。使用非对称加密传输安全传输对称加密的密钥，之后使用对称加密的密钥进行通信从而提高整体的通讯效率。

接下来结合WireShark抓包详细阐述TLS的加密流程，TLS 是建立在 TCP 基础上的，因此必定需要先三次 TCP 握手建立 TCP 连接，然后再是建立 TLS。

1. **Client Hello**
   1. **Client Hello** 报文：客户端对加密算法的支持度不同，因此需要向服务端发送客户端支持的 **加密套件（Cipher Suite）** ，同时还要生成一个 **随机数** 同时保存在客户端和发送给服务
2. **Server Hello**
   1. **ServerCertificate** 报文：服务端收到 Client Hello 之后，向客户端发送 **CA 认证的数字证书**，用来鉴别服务端身份信息，同时还要生成一个 **随机数** 同时保存在服务端和发送给客户端
   2. **Server Hello Done** 报文：表示服务端宣告第一阶段的客户端服务端握手协商结束
   3. 可选：**Certificate Request** 报文：必要情况下，要求客户端发送证书验证身份
   4. 可选：**Server Key Exchange** 报文：如果 CA 认证的数字证书提供的信息不够，服务端还可发送提供补充信息
3. **Client Finish**
   1. **Client Key Exchange** 报文：客户端收到 CA 数字证书并通过验证，然后通过 CA 公钥解密获取到 **服务端公钥**。Client Key Exchange 报文包括有一个随机数，这个随机数被称为 **Pre-master key/secret**；一个表示随后的信息使用双方协商好的加密方法和密钥发送的 **通知** ；还有一个通过协商好的 HASH 算法对前面所有信息内容的 **HASH 计算值**，用来提供服务端校验。**这些信息都通过服务端公钥加密传送给服务端**
   2. **ExchangeCipherSpec** 报文：该报文通知服务端，此后的通信都将使用协商好的加密算法计算对称密钥进行加密通信（也就是使用两个随机数以及第三个 Pre-master key/secret 随机数一起算出一个对称密钥 **session key/secret**）
   3. **Finished** 报文：该报文包括连接至此的所有报文的校验值，使用已经生成的对称加密密钥加密
   4. 可选：**ClientCertificate** 报文：如果服务端请求，客户端需要发送 CA 数字证书
   5. 可选：**CertificateVerify** 报文：服务端如果要求 CA 数字证书，那么需要通过 HASH 算法计算一个服务端发送来的信息摘要
4. **Server Finish**
   1. 服务端先私钥得出**Pre-master key**， 同样使用这三个随机数得出对称加密密钥。服务端最后对客户端发送过来的 **Finished** 报文使用服务端对称密钥进行解密校验。
   2. **ClientCipherSpec** 报文：报文通知服务端，此后的通信都将使用协商好的加密算法计算对称密钥 session key/secret 进行加密通信
   3. **Finished** 报文：标志 TLS 连接建立成功
5. TLS 握手成功此后通过对称密钥 session key/secret 加密通信

> 1. 握手：证书下发，密钥协商（这个阶段都是明文的）
> 2. 数据传输：这个阶段才是加密的，用的就是握手阶段协商出来的对称密钥

进行整理，如下图所示

![image-20210521215733187](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521215733187.png)

### 使用wireshark实战证明

本地地址和LeetCode-CN的交互

![wireshark抓包展示](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521174931648.png)

1. 客户端想服务端发送https请求链接，同时将自身支持的密码套件（非对称加密算法、对称加密算法、摘要算法）发送个服务端

   ![客户端say hello阶段](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521181226105.png)

2. 服务端根据客户端获发送的数据，选择一套密码套件，以及自身的证书发送给客户端

   ![服务端选择的密码套件--服务端say hello阶段](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521182242937.png)

   ![服务端将自身的证书发送给客户端--- 服务端Say hello 阶段](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521182508161.png)

3. 客户端进行验证，验证正常自后，随机生成一串数组作为后续通信的对称加密密钥，然后使用服务端证书的公钥进行加密，同时使用摘要算法对本次的会话信息进行加密，然后一并发送给服务端。

   ![image-20210521185850059](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210521185850059.png)

4. 服务端验证，正常之后，剩下的忘截图了，后面补上

## HTTPS的认证---数字证书

了解了HTTPS加密通信的流程后，对于数据裸奔的疑虑应该基本打消了。然而，还有一个问题：如何确保证书时可信的？

证书不可信可能有两种情况：

1. 证书是伪造的：压根不是CA颁发的
2. 证书被篡改过：比如将XX网站的公钥给替换了

数字证书是由权威的CA（Certificate Authority）机构给服务端进行颁发，CA机构通过服务端提供的相关信息生成证书，证书内容包含了`持有人的相关信息`，`服务器的公钥`，`签署者签名信息（数字签名）`等，最重要的是**公钥在数字证书中**。

> 数字签名和摘要算法的关系
>
> 明文 --> hash运算 --> 摘要 --> 私钥加密 --> 数字签名

数字证书是如何保证公钥来自请求的服务器？

- 数字证书上由持有人的相关信息，通过这点可以确定其不是一个中间人

**证书包含的内容：**

- 一个证书中含有三个部分:`证书内容`，`散列算法`，`加密密文`，证书内容会被散列算法hash计算出摘要，然后使用CA机构提供的私钥进行RSA加密得到数组签名（加密密文）。

![证书内容](/Users/weikunkun/Library/Application Support/typora-user-images/image-20210521221149447.png)

### 证书伪造

浏览器内置的CA的根证书包含了**CA的公钥**

1. 证书颁发的机构是伪造的：浏览器不认识，直接认为是危险证书
2. 证书颁发的机构是确实存在的，于是根据CA名，找到对应内置的CA根证书、CA的公钥。
3. 用CA的公钥，对伪造的证书的摘要进行解密，发现解不了。认为是危险证书

### 证书篡改

1. 根据证书的加密密文（数字签名）使用CA的公钥得到证书摘要A
2. 根据证书内容使用摘要算法得到证书摘要B
3. 对比摘要A和摘要B是否相同



## HTTPS的完整性保护---摘要算法





## 总结

## references

* 图解HTTP
* 趣谈协议---极客时间
* [Why does the SSL/TLS handshake have a client and server random](https://security.stackexchange.com/questions/89383/why-does-the-ssl-tls-handshake-have-a-client-and-server-random)





