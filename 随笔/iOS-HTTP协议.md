## 一. 网络编程基础

在移动互联网时代，几乎所有应用都需要用到网络，只有通过网络跟外界进行数据交互、数据更新，应用才能保持新鲜、活力。一个好的移动网络应用不仅要有良好的UI和良好的用户体验也要具备实时更新数据的能力。**网络编程便是一种实时更新应用数据的常用手段也是开发优秀网络应用的前提和基础。**

#### 1\. 在网络编程中，有几个必须掌握的基本概念

客户端（Client）：移动应用（iOS、android等应用）
服务器（Server）：为客户端提供服务、提供数据、提供资源的机器
请求（Request）：客户端向服务器索取数据的一种行为
响应（Response）：服务器对客户端的请求做出的反应，一般指返回数据给客户端

我们通过下面图片来理解这四者之间的关系

![](http://upload-images.jianshu.io/upload_images/1464492-1b5de575e8fe489d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 二. HTTP协议

HTTP协议是在网络开发中最常用的协议

#### 1\. 概念

> **协议：**协议是指计算机通信网络中两台计算机之间进行通信所必须共同遵守的规定或规则，超文本传输协议(HTTP)是一种通信协议。

> **HTTP协议：**即超文本传输协议(Hypertext transfer protocol)。是一种详细规定了浏览器和万维网(WWW = World Wide Web)服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

> **HTTP协议作用：**HTTP协议是用于从WWW服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示(如文本先于图形)等。

> **URL：**我们在浏览器的地址栏里输入的网站地址叫做URL (Uniform Resource Locator，统一资源定位符)。就像每家每户都有一个门牌地址一样，每个网页也都有一个Internet地址。当你在浏览器的地址框中输入一个URL或是单击一个超级链接时，URL就确定了要浏览的地址。浏览器通过超文本传输协议(HTTP)，将Web服务器上站点的网页代码提取出来，并翻译成漂亮的网页。

#### 2.HTTP版本区别

**HTTP/0.9和1.0** 使用非持续连接，限制每次连接只处理一个请求，服务器对客户端的请求做出响应后，马上断开连接，这种方式可以节省传输时间

**HTTP/1.1** 当前版本。持久连接被默认采用，并能很好地配合代理服务器工作。还支持以管道方式同时发送多个请求，以便降低线路负载，提高传输速度。

**HTTP/1.1相较于 HTTP/1.0 协议的区别主要体现在：**
1 缓存处理
2 带宽优化及网络连接的使用
3 错误通知的管理
4 消息在网络中的发送
5 互联网地址的维护
6 安全性及完整

#### 3.HTTP工作流程

**一次HTTP操作称为一个事务，其工作过程可分为四步：**
1.  首先客户机与服务器需要建立连接。只要单击某个超级链接，HTTP的工作就开始了。
2.  建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。
3.  服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。
4.  客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。

如果在以上过程中的某一步出现错误，那么产生错误的信息将返回到客户端，由显示屏输出。对于用户来说，这些过程是由HTTP自己完成的，用户只要用鼠标点击，等待信息显示就可以了。

**HTTP通信过程 - 请求详细内容**

> HTTP协议规定：1个完整的由客户端发给服务器的HTTP请求中包含以下内容
> 
> 请求头：包含了对客户端的环境描述、客户端请求信息等
> 
> GET /minion.png HTTP/1.1 // 包含了请求方法、请求资源路径、HTTP协议版本
> 
> Host: 120.25.226.186:32812 // 客户端想访问的服务器主机地址
> 
> User-Agent: Mozilla/5.0 // 客户端的类型，客户端的软件环境
> 
> Accept: text/html  // 客户端所能接收的数据类型
> 
> Accept-Language: zh-cn  // 客户端的语言环境
> 
> Accept-Encoding: gzip  // 客户端支持的数据压缩格式

> 请求体：客户端发给服务器的具体数据，比如文件数据(POST请求才会有)

**HTTP通信过程 - 响应详细内容**

> 客户端向服务器发送请求，服务器应当做出响应，即返回数据给客户端
> 
> HTTP协议规定：1个完整的HTTP响应中包含以下内容
> 
> 响应头：包含了对服务器的描述、对返回数据的描述
> 
> HTTP/1.1 200 OK // 包含了HTTP协议版本、状态码、状态英文名称
> 
> Server: Apache-Coyote/1.1 // 服务器的类型
> 
> Content-Type: image/jpeg // 返回数据的类型
> 
> Content-Length: 56811 // 返回数据的长度
> 
> Date: Mon, 23 Jun 2014 12:54:52 GMT // 响应的时间

> 响应体：服务器返回给客户端的具体数据，比如文件数据
> 
> 常见响应状态码
> 
> ![](http://upload-images.jianshu.io/upload_images/1464492-0e2102af481fdba9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**注意：HTTP是基于传输层的TCP协议，而TCP是一个端到端的面向连接的协议。所谓的端到端可以理解为进程到进程之间的通信。所以HTTP在开始传输之前，首先需要建立TCP连接，而TCP连接的过程需要所谓的“三次握手”。**

下图所示TCP连接的三次握手。

![](http://upload-images.jianshu.io/upload_images/1464492-0c52f216dd4b18e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


简单来说就是

1.  客户端向服务器发送消息，告诉服务器我将要发送数据。
2.  服务器端接收到客户端请求后，确认自己准备好接收数据，并告知客户端，我已经准备好，可以发送请求
3.  客户端接受到服务器端已准备好接收的消息后，发送数据给服务器端。

**在TCP三次握手之后，建立了TCP连接，此时HTTP就可以进行传输了。一个重要的概念是面向连接，既HTTP在传输完成之间并不断开TCP连接。在HTTP1.1中这是默认行为。**

#### 4\. HTTP协议的特点

HTTP协议永远都是客户端发起请求，服务器回送响应。这样就限制了使用HTTP协议，无法实现在客户端没有发起请求的时候，服务器将消息推送给客户端。

> HTTP协议的主要特点可概括如下：

1.  简单快速：因为HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。
2.  灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
3.  HTTP 0.9和1.0使用非持续连接：限制每次连接只处理一个请求，服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
    HTTP 1.1使用持续连接：不必为每个web对象创建一个新的连接，一个连接可以传送多个对象。

#### 5\. URL

**URL的全称是Uniform Resource Locator（统一资源定位符）**
URL就是资源的地址、位置。互联网上的每个资源都有一个唯一的URL，通过这个个URL，能找到互联网上唯一的一个资源。

URL的基本格式 = 协议://主机地址/路径
协议：不同的协议，代表着不同的资源查找方式、资源传输方式
主机地址：存放资源的主机（服务器）的IP地址（域名）
路径：资源在主机（服务器）中的具体位置

## 三. 发送HTTP请求

#### 1.发送HTTP请求的方法

在HTTP/1.1协议中，定义了8种发送HTTP请求的方法
GET、POST、OPTIONS、HEAD、PUT、DELETE、TRACE、CONNECT、PATCH
各个方法的解释如下（所有方法全为大写）：
**GET:       请求获取Request-URI所标识的资源**
**POST:     在Request-URI所标识的资源后附加新的数据**
HEAD:     请求获取由Request-URI所标识的资源的响应消息报头
PUT:    请求服务器存储一个资源，并用Request-URI作为其标识
DELETE:  请求服务器删除Request-URI所标识的资源
TRACE:   请求服务器回送收到的请求信息，主要用于测试或诊断
CONNECT: 保留将来使用
OPTIONS: 请求查询服务器的性能，或者查询与资源相关的选项和需求

根据HTTP协议的设计初衷，不同的方法对资源有不同的操作方式
PUT ：增
DELETE ：删
POST：改
GET：查
**最常用的是GET和POST（实际上GET和POST都能办到增删改查）**

#### 2\. GET和POST对比和区别

**GET和POST的主要区别表现在数据传递上**

GET：在请求URL后面以**`?`**的形式拼接发给服务器的参数，多个参数之间用&隔开。

比如`http://www.test.com/login?username=123&pwd=234&type=JSON`由于浏览器和服务器对URL长度有限制，因此在URL后面附带的参数是有限制的，通常不能超过1KB

POST：发给服务器的参数全部放在请求体中，理论上，POST传递的数据量没有限制（具体还得看服务器的处理能力）

**注意：GET和POST都可以向服务器传送数据，也都可以从服务器获取数据**

**关于URL长度的限制**
**首先，HTTP协议及URL官方说明均对URL长度限制没有说明，也就是说GET，POST都对URL长度没有限制**，但是HTTP客户端和服务器的实现对URL长度进行了限制，因此我们使用GET请求拼接参数，有时会导致URL过长而无法进行请求。

**关于安全问题**
并不是POST比GET特别安全，只不过GET传递的参数显示在URL中，我们一眼就可以看到，POST方式看不到是因为浏览器做了限制，我们同样可以用第三方工具看到POST方式传递的数据。

**GET 和POST 的选择**
选择GET和POST的建议
如果仅仅是索取数据（数据查询），建议使用GET
如果是增加、修改、删除数据或者传递大量数据，比如文件上传，建议用POST

**URL中多值参数和中文输出**

1.  多值参数
    有时候一个参数名，可能会对应多个值。例如
    **`http://120.25.226.186:32812/weather?place=Beijing&place=Henan&place=Hunan`**
    当一个参数有多个值的时候，需要使用下面这样方式来赋值
    **`place=beijing&place=shanghai`**
    服务器的place属性是一个数组

2.  中文参数
    当url中有汉字的时候，我们需要先进行中文转码
    GET需要转码，POST当参数有汉字的时候就不用转码了，因为POST参数在请求体中，但是当url有汉字的时候同样需要转码。
```
NSString *urlStr = @"http://120.25.226.186:32812/login2?username=汉字&pwd=520it";
urlStr =  [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

```

#### 3\. iOS中发送HTTP请求的方案

在iOS中，常见的发送HTTP请求的方案有
苹果原生（自带）
**NSURLConnection：用法简单，最古老最经典最直接的一种方案
NSURLSession：功能比NSURLConnection更加强大，苹果目前比较推荐使用这种技术（2013推出，iOS7开始使用的技术）**
CFNetwork：NSURL*的底层，纯C语言

第三方框架
**AFNetworking：简单易用，提供了基本够用的常用功能，维护和使用者多**
ASIHttpRequest：外号“HTTP终结者”，功能极其强大，可惜早已停止更新
MKNetworkKit：简单易用，产自印度，维护和使用者少

**为了提高开发效率，我们开发用的基本是第三方框架，但是我们同样也需要掌握苹果原生的请求方案**

## 四. 服务器返回的数据格式

服务器返回给客户端的数据，一般都是JSON格式或者XML格式（文件下载除外）

#### 1\. JSON

**什么是JSON**
JSON(JavaScript Object Notation)：一种轻量级的数据交换格式，具有良好的可读和便于快速编写的特性。可在不同平台之间进行数据交互。

**JSON的格式**
JSON的格式很像OC中的字典和数组
{"name" : "jack", "age" : 10}
{"names" : ["jack", "rose", "jim"]}
标准JSON格式的注意点：key必须用双引号

**JSON解析方案**
要想从JSON中挖掘出具体数据，需要对JSON进行解析，将JSON数据转换为OC数据类型
在iOS中，苹果为我们提供了JSON的解析方案 **NSJSONSerialization**。
NSJSONSerialization的常见方法

**JSON数据 转 OC对象**

```
/*
   参数一：JSON数据
   参数二：options 一般填kNilOptions
   参数三：错误信息 nil
*/
+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;

```

**OC对象 转 JSON数据**

```
+ (NSData *)dataWithJSONObject:(id)obj options:(NSJSONWritingOptions)opt error:(NSError **)error;

```

#### 2\. XML

**什么是XML**

扩展标记语言 (Extensible Markup Language, XML) ，用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言。 XML使用DTD(document type definition)文档类型定义来组织数据;格式统一，跨平台和语言，早已成为业界公认的标准。

**XML格式**

一个常见的XML文档一般由以下部分组成

1.  文档声明
    在XML文档的最前面，必须编写一个文档声明，用来声明XML文档的类型
    最简单的声明
    <?xml version="1.0" ?>
    用encoding属性说明文档的字符编码
    <?xml version="1.0" encoding="UTF-8" ?>

2.  元素（Element）
    一个元素包括了开始标签和结束标签
    拥有内容的元素：<video>小黄人</video>
    没有内容的元素：<video></video>
    一个元素可以嵌套若干个子元素（不能出现交叉嵌套）
    规范的XML文档最多只有1个根元素，其他元素都是根元素的子孙元素

3.  属性（Attribute）

**XML解析**
要想从XML中提取有用的信息，必须得学会解析XML
XML的解析方式有2种
DOM：一次性将整个XML文档加载进内存，比较适合解析小文件
SAX：从根元素开始，按顺序一个元素一个元素往下解析，比较适合解析大文件

解析XML的工具
苹果原生NSXMLParser:  使用SAX方式解析，使用简单
GDataXML:  采用DOM方式解析，该框架由Goole开发，是基于xml2的

**1\. 使用NSXMLParser解析XML方法和步骤**

```
//解析步骤：
//1 创建一个解析器
NSXMLParser *parser = [[NSXMLParser alloc]initWithData:data];
//2 设置代理
parser.delegate = self;
//3 开始解析
[parser parse];

```

**NSXMLParser代理方法**

```
//1.开始解析XML文档
-(void)parserDidStartDocument:(nonnull NSXMLParser *)parser
{
}
//2.开始解析XML中某个元素的时候调用，比如<video>
-(void)parser:(nonnull NSXMLParser *)parser didStartElement:(nonnull NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName attributes:(nonnull NSDictionary<NSString *,NSString *> *)attributeDict
{
    //可在此方法中做字典转模型操作，参数attributeDict存放着元素的属性
}
//3.当某个元素解析完成之后调用，比如</video>
-(void)parser:(nonnull NSXMLParser *)parser didEndElement:(nonnull NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName
{
}
//4.XML文档解析结束
-(void)parserDidEndDocument:(nonnull NSXMLParser *)parser
{
}

```

NSXMLParser采取的是SAX方式解析，特点是事件驱动，下面情况都会通知代理

当扫描到文档（Document）的开始与结束

当扫描到元素（Element）的开始与结束

**2\. GDataXML解析XML方法和步骤**

GDataXML需要配置环境

1.  设置libxml2的头文件搜索路径（为了能找到libxml2库的所有头文件）
    在Head Search Path中加入/usr/include/libxml2

2.  设置链接参数（自动链接libxml2库）
    在Other Linker Flags中加入-lxml2

使用方法

```
//1 加载XML文档（使用的是DOM的方式次性把整个XML加载完毕）
    GDataXMLDocument *doc = [[GDataXMLDocument alloc]initWithData:data options:kNilOptions error:nil];
//2 获取XML文档的根元素，根据根元素取出XML中的每个子元素
  NSArray * elements = [doc.rootElement elementsForName:@"video"];
//3 取出每个子元素的属性并转换为模型
for (GDataXMLElement *ele in elements) {
    //4 把转换好的模型添加到tableView的数据源self.videos数组中
    [self.videos addObject:video];
}

```

#### 3\. JSON和XML比较

同一份数据，既可以用JSON来表示，也可以用XML来表示
相比之下，JSON的体积小于XML，并且易于解析，传输速度也快，所以服务器返回给移动端的数据格式以JSON居多。

## 五. HTTP应用

#### 1\. 断点续传的实现原理

HTTP协议设置请求头内容，支持只请求某个资源的某一部分。
Range 请求的资源范围；
Content-Range 响应的资源范围；
**在连接断开重连时，客户端只请求该资源未下载的部分，而不是重新请求整个资源，来实现断点续传。**
分块请求资源实例：
Eg1：Range: bytes=306302- ：请求这个资源从306302个字节到末尾的部分；
Eg2：Content-Range: bytes 306302-604047/604048：响应中指示携带的是该资源的第306302-604047的字节，该资源共604048个字节；
**客户端通过并发的请求相同资源的不同片段，来实现对某个资源的并发分块下载。从而达到快速下载的目的。目前流行的FlashGet和迅雷基本都是这个原理。**

#### 2\. 多线程下载的原理

**下载工具开启多个发出HTTP请求的线程，每个http请求只请求资源文件的一部分，例如：Content-Range: bytes 20000-40000/47000；
最后合并每个线程下载的文件。**
## 六. HTTPS

#### 1\. HTTPS简介

HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。

#### 2\. HTTPS通信过程

![](http://upload-images.jianshu.io/upload_images/1464492-991b504090945c09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 3\. HTTPS与HTTP的区别

**超文本传输协议HTTP协议**被用于在Web浏览器和网站服务器之间传递信息。HTTP协议以明文方式发送内容，不提供任何方式的数据加密，如果攻击者截取了Web浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息，**因此HTTP协议不适合传输一些敏感信息，比如信用卡号、密码等。**

为了解决HTTP协议的这一缺陷，需要使用另一种协议：安全套接字层超文本传输协议HTTPS。**为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。**

HTTPS和HTTP的区别主要为以下四点：

> 一、https协议需要到ca申请证书，一般免费证书很少，需要交费。
> 
> 二、http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
> 
> 三、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
> 
> 四、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

