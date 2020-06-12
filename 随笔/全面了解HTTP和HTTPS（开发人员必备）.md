## 序言

Http和Https属于计算机网络范畴，但作为开发人员，不管是后台开发或是前台开发，都很有必要掌握它们。
在学习Http和Https的过程中，主要是参考了[阮一峰老师的博客](http://www.ruanyifeng.com/blog/2016/08/http.html)，讲的很全面，并且通俗易懂，有兴趣的同学可以去学习学习。
这篇文章主要是按照自己的思路来讲解对Http和Https的理解。文章将会从以下几个方面介绍。

目录树（**暂时还不知道简书编辑器怎么通过目录树进行页面内跳转，哪位同学知道希望不吝告知**）：

*   **一、网络层结构**
*   **二、Http协议**
*   **三、Tcp三次握手**
*   **四、Https协议/SSL协议**
*   **五、SSL证书**
*   **六、RSA加密和DH加密**
*   **七、Http和Https对比**

从目录结构可以看出，每个标题展开来说都是一个很大的主题。但本文旨在让各位同学对Http和Https相关知识有一个全面的认知，不会太过深入探讨各个主题，有兴趣的同学可以进行针对性研究。

## 一、网络层结构

网络结构有两种主流的分层方式：**OSI七层模型**和**TCP/IP四层模型**。

### OSI七层模型和TCP/IP四层模型

OSI是指Open System Interconnect，意为开放式系统互联。
TCP/IP是指传输控制协议/网间协议，是目前世界上应用最广的协议。

| OSI层 | 对应TCP/IP层 | OSI各层功能 | 网络协议 | 设备 |
| --- | --- | --- | --- | --- |
| 应用层 | 应用层 | 应用程序（电子邮件，文件服务）,用户接口 | HTTP，FTP，TFTP，NFS | 网关 |
| 表示层 | 应用层 | 数据的表示，压缩和加密（数据格式化，代码转换，数据加密） | TELNET，SNMP | 网关 |
| 会话层 | 应用层 | 建立、管理和终止会话 | SMTP，DNS | 网关 |
| 传输层 | 传输层 | 提供端到端可靠报文段传递和错误恢复 | TCP，UDP | 网关 |
| 网络层 | 网际互联层 | 提供数据包从源到宿的传递和网际交互 | IP，ICMP，ARP，RARP，UUCP | 路由器 |
| 链路层 | 网络接口层 | 将比特组装成帧和点到点传递 | FDDI，SLIP，PPP，PDN | 交换机 |
| 物理层 | 网络接口层 | 传输比特流，以二进制数据形式在物理媒体上传输数据 | ISO2110，IEEE802，IEEE802.2 | 集线器，中继器 |

### 两种模型区别

1.  OSI采用七层模型，TCP/IP是四层模型
2.  TCP/IP网络接口层没有真正的定义，只是概念性的描述。OSI把它分为2层，每一层功能详尽。
3.  在协议开发之前，就有了OSI模型，所以OSI模型具有共通性，而TCP/IP是基于协议建立的模型，不适用于非TCP/IP的网络。
4.  实际应用中，OSI模型是理论上的模型，没有成熟的产品；而TCP/IP已经成为国际标准。

## 二、HTTP协议

Http是基于TCP/IP协议的应用程序协议，不包括数据包的传输，主要规定了客户端和服务器的通信格式，默认使用80端口。

### Http协议的发展历史

1.  1991年发布Http/0.9版本，只有Get命令，且服务端直返HTML格式字符串，服务器响应完毕就关闭TCP连接。
2.  1996年发布Http/1.0版本，**优点**：可以发送任何格式内容，包括文字、图像、视频、二进制。也丰富了命令Get，Post，Head。请求和响应的格式加入头信息。**缺点**：每个TCP连接只能发送一个请求，而新建TCP连接的成本很高，导致Http/1.0新能很差。
3.  1997发布Http/1.1版本，完善了Http协议，**直至20年后的今天仍是最流行的版本**。
    **优点**：`a`. 引入持久连接，TCP默认不关闭，可被多个请求复用，对于一个域名，多数浏览器允许同时建立6个持久连接。`b`. 引入管道机制，即在同一个TCP连接中，可以同时发送多个请求，不过服务器还是按顺序响应。`c`. 在头部加入Content-Length字段，一个TCP可以同时传送多个响应，所以就需要该字段来区分哪些内容属于哪个响应。`d`. 分块传输编码，对于耗时的动态操作，用流模式取代缓存模式，即产生一块数据，就发送一块数据。`e`. 增加了许多命令，头信息增加Host来指定服务器域名，可以访问一台服务器上的不同网站。
    **缺点**：TCP连接中的响应有顺序，服务器处理完一个回应才能处理下一个回应，如果某个回应特别慢，后面的请求就会排队等着（对头堵塞）。
4.  2015年发布Http/2版本，它有几个特性：二进制协议、多工、数据流、头信息压缩、服务器推送。

### Http请求和响应格式

Request格式：

```
GET /barite/account/stock/groups HTTP/1.1
QUARTZ-SESSION: MC4xMDQ0NjA3NTI0Mzc0MjAyNg.VPXuA8rxTghcZlRCfiAwZlAIdCA
DEVICE-TYPE: ANDROID
API-VERSION: 15
Host: shitouji.bluestonehk.com
Connection: Keep-Alive
Accept-Encoding: gzip
User-Agent: okhttp/3.10.0

```

Response格式：

```
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Mon, 15 Oct 2018 03:30:28 GMT
Content-Type: application/json;charset=UTF-8
Pragma: no-cache
Cache-Control: no-cache
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Content-Encoding: gzip
Transfer-Encoding: chunked
Proxy-Connection: Keep-alive

{"errno":0,"dialogInfo":null,"body":{"list":[{"flag":2,"group_id":1557,"group_name":"港股","count":1},{"flag":3,"group_id":1558,"group_name":"美股","count":7},{"flag":1,"group_id":1556,"group_name":"全部","count":8}]},"message":"success"}

```

说明一下请求头和响应头的部分字段：

*   `Host`：指定服务器域名，可用来区分访问一个服务器上的不同服务
*   `Connection`：`keep-alive`表示要求服务器不要关闭TCP连接，`close`表示明确要求关闭连接，默认值是keep-alive
*   `Accept-Encoding`：说明自己可以接收的压缩方式
*   `User-Agent`：用户代理，是服务器能识别客户端的操作系统（Android、IOS、WEB）及相关的信息。作用是帮助服务器区分客户端，并且针对不同客户端让用户看到不同数据，做不同操作。
*   `Content-Type`：服务器告诉客户端数据的格式，常见的值有`text/plain，image/jpeg，image/png，video/mp4，application/json，application/zip`。这些数据类型总称为`MIME TYPE`。
*   `Content-Encoding`：服务器数据压缩方式
*   `Transfer-Encoding`：`chunked`表示采用分块传输编码，有该字段则无需使用`Content-Length`字段。
*   `Content-Length`：声明数据的长度，请求和回应头部都可以使用该字段。

## Tcp三次握手

Http和Https协议请求时都会通过Tcp三次握手建立Tcp连接。那么，三次握手是指什么呢？

![image](//upload-images.jianshu.io/upload_images/9942787-afa31b861fb0f247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/960)

那么，为什么一定要三次握手呢，一次可以吗？两次可以吗？带着这些问题，我们来分析一下为什么必须是三次握手。

1.  第一次握手，A向B发送信息后，B收到信息。**B可确认A的发信能力和B的收信能力**
2.  第二次握手，B向A发消息，A收到消息。**A可确认A的发信能力和收信能力，A也可确认B的收信能力和发信能力**
3.  第三次握手，A向B发送消息，B接收到消息。**B可确认A的收信能力和B的发信能力**

通过三次握手，**A和B都能确认自己和对方的收发信能力，相当于建立了互相的信任**，就可以开始通信了。

下面，我们介绍一下三次握手具体发送的内容，用一张图描述如下：

![image](//upload-images.jianshu.io/upload_images/9942787-495afdbac8f2012c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/875)

首先，介绍一下几个概念：

*   `ACK`：响应标识，1表示响应，连接建立成功之后，所有报文段ACK的值都为1
*   `SYN`：连接标识，1表示建立连接，连接请求和连接接受报文段SYN=1，其他情况都是0
*   `FIN`：关闭连接标识，1标识关闭连接，关闭请求和关闭接受报文段FIN=1，其他情况都是0，跟SYN类似
*   `seq number`：序号，一个随机数X，请求报文段中会有该字段，响应报文段没有
*   `ack number`：应答号，值为请求seq+1，即X+1，除了连接请求和连接接受响应报文段没有该字段，其他的报文段都有该字段

知道了上面几个概念后，看一下三次握手的具体流程：

1.  第一次握手：建立连接请求。客户端发送连接请求报文段，将SYN置为1，seq为随机数x。然后，客户端进入SYN_SEND状态，等待服务器确认。
2.  第二次握手：确认连接请求。服务器收到客户端的SYN报文段，需要对该请求进行确认，设置ack=x+1（即客户端seq+1）。同时自己也要发送SYN请求信息，即SYN置为1，seq=y。服务器将SYN和ACK信息放在一个报文段中，一并发送给客户端，服务器进入SYN_RECV状态。
3.  第三次握手：客户端收到SYN+ACK报文段，将ack设置为y+1，向服务器发送ACK报文段，这个报文段发送完毕，客户端和服务券进入ESTABLISHED状态，完成Tcp三次握手。

从图中可以看出，建立连接经历了**三次握手**，当数据传输完毕，需要断开连接，而断开连接经历了**四次挥手**：

1.  第一次挥手：主机1（可以是客户端或服务器），设置seq和ack向主机2发送一个FIN报文段，此时主机1进入FIN_WAIT_1状态，表示没有数据要发送给主机2了
2.  第二次挥手：主机2收到主机1的FIN报文段，向主机1回应一个ACK报文段，表示同意关闭请求，主机1进入FIN_WAIT_2状态。
3.  第三次挥手：主机2向主机1发送FIN报文段，请求关闭连接，主机2进入LAST_ACK状态。
4.  第四次挥手：主机1收到主机2的FIN报文段，想主机2回应ACK报文段，然后主机1进入TIME_WAIT状态；主机2收到主机1的ACK报文段后，关闭连接。此时主机1等待主机2一段时间后，没有收到回复，证明主机2已经正常关闭，主机1页关闭连接。

下面是Tcp报文段首部格式图，对于理解Tcp协议很重要：

![image](//upload-images.jianshu.io/upload_images/9942787-69d610f456877e52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/669)

## Https协议/SSL协议

Https协议是以安全为目标的Http通道，简单来说就是Http的安全版。主要是在Http下加入SSL层（现在主流的是SLL/TLS），SSL是Https协议的安全基础。Https默认端口号为443。
前面介绍了Http协议，各位同学能说出Http存在的风险吗？

1.  窃听风险：Http采用明文传输数据，第三方可以获知通信内容
2.  篡改风险：第三方可以修改通信内容
3.  冒充风险：第三方可以冒充他人身份进行通信

SSL/TLS协议就是为了解决这些风险而设计，希望达到：

1.  所有信息加密传输，三方窃听通信内容
2.  具有校验机制，内容一旦被篡改，通信双发立刻会发现
3.  配备身份证书，防止身份被冒充

下面主要介绍SSL/TLS协议。

### SSL发展史（互联网加密通信）

1.  1994年NetSpace公司设计SSL协议（Secure Sockets Layout）1.0版本，但未发布。
2.  1995年NetSpace发布SSL/2.0版本，很快发现有严重漏洞
3.  1996年发布SSL/3.0版本，得到大规模应用
4.  1999年，发布了SSL升级版TLS/1.0版本，**目前应用最广泛的版本**
5.  2006年和2008年，发布了TLS/1.1版本和TLS/1.2版本

### SSL原理及运行过程

SSL/TLS协议基本思路是采用公钥加密法（最有名的是RSA加密算法）。大概流程是，**客户端向服务器索要公钥，然后用公钥加密信息，服务器收到密文，用自己的私钥解密**。
为了防止公钥被篡改，把公钥放在数字证书中，证书可信则公钥可信。公钥加密计算量很大，为了提高效率，**服务端和客户端都生成对话秘钥，用它加密信息，而对话秘钥是对称加密，速度非常快。而公钥用来机密对话秘钥**。

下面用一张图表示SSL加密传输过程：

![image](//upload-images.jianshu.io/upload_images/9942787-ed94df5036e93072.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

详细介绍一下图中过程：

1.  客户端给出协议版本号、一个客户端随机数A（Client random）以及客户端支持的加密方式
2.  服务端确认双方使用的加密方式，并给出数字证书、一个服务器生成的随机数B（Server random）
3.  客户端确认数字证书有效，生成一个新的随机数C（Pre-master-secret），**使用证书中的公钥对C加密**，发送给服务端
4.  **服务端使用自己的私钥解密出C**
5.  客户端和服务器根据约定的加密方法，使用三个随机数ABC，生成对话秘钥，之后的通信都用这个对话秘钥进行加密。

## SSL证书

上面提到了，Https协议中需要使用到SSL证书。
SSL证书是一个二进制文件，里面包含经过认证的网站公钥和一些元数据，需要从经销商购买。
证书有很多类型，按认证级别分类：

*   **域名认证（DV=Domain Validation）**：最低级别的认证，可以确认申请人拥有这个域名
*   **公司认证（OV=Organization Validation）**：确认域名所有人是哪家公司，证书里面包含公司的信息
*   **扩展认证（EV=Extended Validation）**：最高级别认证，浏览器地址栏会显示公司名称。

EV证书浏览器地址栏样式：

![image](//upload-images.jianshu.io/upload_images/9942787-0f852a848091326f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

OV证书浏览器地址栏样式：

![image](//upload-images.jianshu.io/upload_images/9942787-b7ecd5c6f53e7a58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/962)

DV证书浏览器样式：

![image](//upload-images.jianshu.io/upload_images/9942787-4b8628cffd5f8617.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/962)

按覆盖范围分类：

*   **单域名证书**：只能用于单域名，[foo.com](http://foo.com)证书不能用不[www.foo.com](http://www.foo.com)
*   **通配符证书**：可用于某个域名及所有一级子域名，比如*[.foo.com](http://.foo.com)的证书可用于[foo.com](http://foo.com)，也可用于[www.foo.com](http://www.foo.com)
*   **多域名证书**：可用于多个域名，比如foo.com和bar.com

**认证级别越高，覆盖范围越广的证书，价格越贵**。也有免费的证书，为了推广Https，电子前哨基金会成立了[Let's Encrypt](https://letsencrypt.org/)提供免费证书。
证书的经销商也很多，知名度比较高的有[亚洲诚信(Trust Asia)](https://www.trustasia.com/)。

## RSA加密和DH加密

### 加密算法分类

加密算法分为**对称加密**、**非对称加密**和**Hash加密**算法。

*   **对称加密**：甲方和乙方使用同一种加密规则对信息加解密
*   **非对称加密**：乙方生成两把秘钥（公钥和私钥）。公钥是公开的，任何人都可以获取，私钥是保密的，只存在于乙方手中。甲方获取公钥，然后用公钥加密信息，乙方得到密文后，用私钥解密。
*   **Hash加密**：Hash算法是一种单向密码体制，即只有加密过程，没有解密过程

对称加密算法加解密效率高，速度快，适合大数据量加解密。常见的堆成加密算法有`DES、AES、RC5、Blowfish、IDEA`
非对称加密算法复杂，加解密速度慢，但安全性高，一般与对称加密结合使用（对称加密通信内容，非对称加密对称秘钥）。常见的非对称加密算法有`RSA、DH、DSA、ECC`
Hash算法特性是：输入值一样，经过哈希函数得到相同的散列值，但并非散列值相同则输入值也相同。常见的Hash加密算法有`MD5、SHA-1、SHA-X系列`

下面着重介绍一下RSA算法和DH算法。

### RSA加密算法

Https协议就是使用RSA加密算法，可以说RSA加密算法是宇宙中最重要的加密算法。
RSA算法用到一些数论知识，包括**互质关系，欧拉函数，欧拉定理**。此处不具体介绍加密的过程，如果有兴趣，可以参照[RSA算法加密过程](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)。
RSA算法的安全保障基于**大数分解问题**，目前破解过的最大秘钥是700+位，也就代表1024位秘钥和2048位秘钥可以认为绝对安全。
大数分解主要难点在于计算能力，如果未来计算能力有了质的提升，那么这些秘钥也是有可能被破解的。

### DH加密算法

DH也是一种非对称加密算法，[DH加密算法过程](https://zh.wikipedia.org/wiki/%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E5%AF%86%E9%91%B0%E4%BA%A4%E6%8F%9B)。
DH算法的安全保障是基于**离散对数问题**。

## Http协议和Https协议的对比

Http和Https的区别如下：

*   https协议需要到CA申请证书，大多数情况下需要一定费用
*   Http是超文本传输协议，信息采用明文传输，Https则是具有安全性SSL加密传输协议
*   Http和Https端口号不一样，Http是80端口，Https是443端口
*   Http连接是无状态的，而Https采用Http+SSL构建可进行加密传输、身份认证的网络协议，更安全。
*   Http协议建立连接的过程比Https协议快。因为Https除了Tcp三次握手，还要经过SSL握手。连接建立之后数据传输速度，二者无明显区别。

## 总结

经过了3天的学习总结，总算完成了这篇文章，本文可以帮助读者大体上把握Http和Https的知识框架。并没有深入探讨每个主题的内容，当读者有了自己知识框架之后，可以自行深入了解每个知识点的内容。
这边提供一份总结资料：[计算机网络相关知识汇总](https://github.com/JeffyLu/JeffyLu.github.io/issues/22)。

原文作者：左大人
原文链接：https://www.jianshu.com/p/27862635c077
