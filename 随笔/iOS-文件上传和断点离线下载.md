## 一. iOS中发送HTTP请求的方案

在iOS中，我们常用发送HTTP请求的方案有
苹果原生（自带）
**NSURLConnection：用法简单，最古老最经典最直接的一种方案 （iOS 9.0弃用）**
**NSURLSession：功能比NSURLConnection更加强大，苹果目前比较推荐使用这种技术**
**第三方框架AFNetworking：简单易用，提供了基本够用的常用功能，维护和使用者多**

## 二. NSURLConnection （已弃用）

虽然NSURLConnection已经被弃用，但是我们还是要了解NSURLConnection的用法，便于我们之后更好的理解NSURLSession。

### 1\. NSURLConnection的使用

使用NSURLConnection发送请求的步骤很简单

1.  创建一个NSURL对象，设置请求路径
    NSURL：请求地址
2.  传入NSURL创建一个NSURLRequest对象，设置请求头和请求体
    NSURLRequest：一个NSURLRequest对象就代表一个请求，它包含的信息有
    一个NSURL对象、请求方法、请求头、请求体、请求超时等
    NSMutableURLRequest：NSURLRequest的子类，NSURLRequest默认的请求方法是GET，当我们需要修改请求方法时，请求头的时候就要用可变的NSMutableURLRequest

3.  使用NSURLConnection发送请求
    NSURLConnection负责发送请求，建立客户端和服务器的连接，同时发送数据给服务器，并收集来自服务器的响应数据

### 2\. NSURLConnection发送请求

##### 2.1 创建NSURLRequest

![](http://upload-images.jianshu.io/upload_images/1464492-6aa4273ed61dca6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


NSURLRequest默认的请求方法是GET，当我们需要修改请求方法为POST的时候就要用可变的NSMutableURLRequest，并设置请求方式，请求头和请求体。

![](http://upload-images.jianshu.io/upload_images/1464492-86c18c376513ea5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 2.2 发送请求

**NSURLConnection常见的发送请求方法分为同步和异步请求**

**注意：同步请求和异步请求的区别在于是否会阻塞线程，同步请求会阻塞线程等请求完毕以后再执行后面的任务，异步请求不会阻塞线程，会等后面的任务执行完毕之后回头执行请求，异步请求有开子线程的能力，但并不一定会开启子线程**

##### 2.2.1 同步请求

![](http://upload-images.jianshu.io/upload_images/1464492-cd754b7f50e8baff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们可以通过打印 data response error 的值来查看返回的数据，响应头，和错误信息

##### 2.2.2 异步请求

异步请求根据对服务器返回数据的处理方式的不同，block回调和代理。

**异步请求block回调**

![](http://upload-images.jianshu.io/upload_images/1464492-6a8241302f08c691.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**异步请求代理方法**

成为NSURLConnection的代理，需要遵守**NSURLConnectionDataDelegate**协议

使用代理异步请求的方法有三种

![](http://upload-images.jianshu.io/upload_images/1464492-0ff326534e2ba580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**NSURLConnectionDataDelegate**的代理方法

![](http://upload-images.jianshu.io/upload_images/1464492-9015e3f5cc71e1b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**注意：**

** 1\. 苹果为了方便我们拿到数据以后显示或者刷新UI，默认代理方法在主线程中调用，我们可以通过对象方法setDelegateQueue来设置代理执行的队列。**

** 2\. 请求数据的过程也可能非常耗时，我们能否将请求数据的操作也放在子线程中进行呢？答案是可以的但是需要注意，initWithRequest会将方法会将NSURLConnection对象加入当前对应的RunLoop中，当我们在子线程中进行网络请求，默认子线程的RunLoop不会自动创建，NSURLConnection对象会被释放，因此我们需要开启子线程中的RunLoop，保证NSURLConnection对象不会被释放。另外，当在子线程中设置请求手动开启调用start方法，就不需要开启子线程RunLoop了，因为start方法内部如果发现RunLoop不存在就会自动创建。**

### 3\. NSURLConnection 文件下载

##### 3.1 小文件下载

当我们下载很小的文件的时候，例如一张很小的图片，不会占用太大内存的话我们可以通过URL直接进行下载

```
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_02.png"];
NSData *data = [NSData dataWithContentsOfURL:url];
self.imageView.image = [UIImage imageWithData:data];

```

也可以通过NSURLConnection进行下载

```
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_02.png"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[NSURLConnection sendAsynchronousRequest:request queue[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
    self.imageView.image = [UIImage imageWithData:data];
}];

```

##### 3.2 较大文件下载

当我们需要下载一个较大文件的话，需要考虑的东西就很多了首先下载较大文件是一个耗时操作，我们应该肯定要通过什么方法来下载数据，第二，大文件需要时间较长，如果在下载过程中用户想要取消或者暂停应该怎么做，第三，下载文件较大应该怎么做存储，放在内存中？还是保存在沙盒中，都是我们需要考虑的。那么我们一个一个开始解决这些问题

第一：用什么方法请求数据？

因为文件较大，比较耗时，首先我们肯定要使用异步请求数据，另外同时在下载过程中我们同样需要拿到下载的数据，下载的进度，还要判断文件是否下载完成，因此使用异步下载代理方法

```
#import "ViewController.h"
@interface ViewController ()<NSURLConnectionDataDelegate>
//下载的文件
@property (nonatomic, strong) NSMutableData *fileData;
//当前已经下载文件的大小
@property (nonatomic, assign) NSInteger currentLength;
//下载文件的总大小
@property (nonatomic, assign) NSInteger totalLength;
@end
@implementation ViewController
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_02.mp4"];
    //2.创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //3.设置代理,发送请求
    [NSURLConnection connectionWithRequest:request delegate:self];
}
#pragma mark  NSURLConnectionDataDelegate  start
//1.接收到服务器响应的时候调用
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    self.fileData = [NSMutableData data];
    //拿到文件的总大小
    self.totalLength = response.expectedContentLength;
    NSLog(@"%zd",self.totalLength);
}
//2.接收到服务器返回的数据,会调用多次
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 将下载的文件拼接到fileData中
    [self.fileData appendData:data];
    // 记录当前下载的多少
    self.currentLength = self.fileData.length;
    NSLog(@"%f",1.0 * self.currentLength / self.totalLength);
}
//3.当请求完成之后调用该方法
-(void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    //保存下载的文件到沙盒
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    //拼接文件全路径
    NSString *fullPath = [caches stringByAppendingPathComponent:@"abc.mp4"];
    //写入数据到文件
    [self.fileData writeToFile:fullPath atomically:YES];
    NSLog(@"%@",fullPath);
}
// 4.当请求失败的适合调用该方法,如果失败那么error有值
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    NSLog(@"didFailWithError");
}

```

至此我们已经已经实现了一个简单的文件下载，我们可以看到下载进度，也可以打印出沙盒存储目录找到下载的文件，但是还存在一些问题，第一，我们没有办法控制文件下载暂停。第二，我们发现开始下载后工程占用内存开始飙升，大约上升了我们下载的文件大小，这是因为fileData 这个属性在内存中也存储了一份我们下载的文件。

第一：暂停下载

当我们点击暂停的时候下载暂停，当点击开始的时候接着之前的下载，请求头中有属性可以设置要请求的内容，因此我们需要设置请求头，直接来看代码

```
    // 断点下载需要设置请求头 因此request 要可变的 NSMutableURLRequest;
    // 设置请求头
    /*
     表示头500个字节：Range: bytes=0-499
     表示第二个500字节：Range: bytes=500-999
     表示最后500个字节：Range: bytes=-500
     表示500字节以后的范围：Range: bytes=500-
     */
     // 传入已经下载文件的大小，表示从已经下载以后开始下载
    NSString *range = [NSString stringWithFormat:@"bytes=%zd-",self.currentLength];
    NSLog(@"%@",range);
    [request setValue:range forHTTPHeaderField:@"Range"];

```

第二：解决fileData占用内存问题，如果不用fileData每次拼接下载的数据，我们可以越过内存存储这一环节，直接边下载边往沙盒中存储，首先在didReceiveResponse方法中创建文件用来存储文件。

实现代码

```
// 注意：获取总文件大小 这个获取的是每次返回数据时的数据大小,但是当我们暂停，在重新开始下载的时候，返回的就是剩余数据文件的大小，因此在当我们计算进度的时候就不准确了
// 所以我们需要当再次回到这个方法的时候，判断self.currentLength 是否为0 如果说明是第一次下载，我们需要创建文件并写入沙盒，如果不为零，说明是暂停以后重新开始的，那个就不需要重新创建文件了，直接return就好了
if (self.currentLength > 0) {
   return;
}
self.totalLength = response.expectedContentLength;
NSFileManager *manager = [NSFileManager defaultManager];
NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
// response.suggestedFilename 获得下载文件名
NSString *filePath = [caches stringByAppendingPathComponent:response.suggestedFilename];
[manager createFileAtPath:filePath contents:nil attributes:nil];
self.filePath = filePath;
NSLog(@"%@",self.filePath);

```

其次我们需要设置后续下载内容拼接在之前下载好的内容之后，这需要用到文件句柄，在didReceiveData（接收到服务器返回数据的方法）中设置每次下载的数据拼接在已经下载好的数据之后。如果直接在didReceiveData方法中写入文件，会覆盖之前下载好的文件内容。

```
// 文件句柄
NSFileHandle *handle = [NSFileHandle fileHandleForWritingAtPath:self.filePath];
self.handle = handle;
//设置指向文件的末尾
[self.handle seekToEndOfFile];
// 写数据
[self.handle writeData:data];
// 也可以设置指定位置写入文件
// [handle seekToFileOffset:(unsigned long long)];

```

当然，文件句柄的创建我们可以写在didReceiveResponse接受到服务器响应的时候创建，然后用属性强引用，不必再每次返回数据的时候重新创建。文件句柄需要在connectionDidFinishLoading（请求完成之后）关闭并置空。

```
[self.handle closeFile];
self.handle = nil;

```

除了文件句柄，我们也可以使用输出流来写数据，达到和文件句柄一样的效果

```
// 输出流 
// 第一个参数：文件路径  第二个参数：是否拼接 YES表示往后拼接数据，NO表示覆盖
self.stream = [NSOutputStream outputStreamToFileAtPath:self.filePath append:YES];
// 输出流需要开启
[self.stream open];
// 输出流写数据
// 参数一：要写入的二进制数据，bytes类型 参数二：数据的大小
[self.stream write:data.bytes maxLength:data.length];  

```

输出流一样需要关闭

```
//关闭输出流
[self.stream close];
self.stream = nil;

```

至此我们就使用NSURLConnection实现了简单较大文件下载。配合简单的UI可以实现断点下载。

**总结：**
**1\. 通过设置请求头Range设置请求数据的范围**
**2\. 通过响应头获取下载文件的一些基本信息，文件大小，名字等。**
**3\. 使用文件句柄或者输出流来实现拼接文件**

### 3\. NSURLConnection 文件上传

**文件上传步骤**
1.  确定请求路径
2.  根据URL创建一个可变的请求对象
3.  设置请求对象，修改请求方式为POST
4.  设置请求头，告诉服务器我们将要上传文件（Content-Type）
5.  设置请求体（在请求体中按照既定的格式拼接要上传的文件参数和非文件参数等数据）
    5.1 拼接文件参数
    5.2 拼接非文件参数
    5.3 添加结尾标记
6.  使用NSURLConnection sendAsync发送异步请求上传文件
7.  解析服务器返回的数据

**文件上传设置请求体的数据格式**

```
  //请求体拼接格式
  //分隔符：----WebKitFormBoundaryhBDKBUWBHnAgvz9c
  //01.文件参数拼接格式
   --分隔符
   Content-Disposition:参数
   Content-Type:参数
   空行
   文件参数
  //02.非文件拼接参数
   --分隔符
   Content-Disposition:参数
   空行
   非文件的二进制数据
  //03.结尾标识
  --分隔符--

```

关于文件上传NSURLConnection 与 NSURLSession 上传方式差不多，我们在NSURLSession中在做详细介绍。

## 三. NSURLSesscion

### 1\. NSURLSesscion使用步骤

1.  使用NSURLSession对象创建Task
2.  执行Task

Task的类型

![](http://upload-images.jianshu.io/upload_images/1464492-2d51d24310c63486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2\. NSURLSesscion 常用方法

获得Session

```
获得共享的Session
+ (NSURLSession *)sharedSession;

自定义Session
+ (NSURLSession *)sessionWithConfiguration:(NSURLSessionConfiguration *)configuration delegate:(id <NSURLSessionDelegate>)delegate delegateQueue:(NSOperationQueue *)queue;

```

Session常用方法

```
常见方法
- (void)suspend; // 暂停
- (void)resume; // 恢复
- (void)cancel; // 取消
@property (readonly, copy) NSError *error; // 错误
@property (readonly, copy) NSURLResponse *response; // 响应

// 取消任务 这个方法可以拿到恢复下载需要的数据
- (void)cancelByProducingResumeData:(void (^)(NSData *resumeData))completionHandler; 

```

### 3\. NSURLSesscion 简单使用

##### 1\. GET请求

```
NSURLSession *session = [NSURLSession sharedSession];
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
/**
参数一:请求对象
参数二:block块
data :响应体
response:响应头
error :错误信息
*/
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
     NSLog(@"%@---%@",[NSJSONSerialization JSONObjectWithData: data options:kNilOptions error:nil],[NSThread currentThread]);
}];
// 也可以使用下面方法直接传递url，这个方法会自动将url包装成请求对象，但是这种方法我们没有办法拿到请求对象，设置请求方式，因此这种方法只能GET请求
// NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
// NSLog(@"%@---%@",[NSJSONSerialization JSONObjectWithData: data options:kNilOptions error:nil],[NSThread currentThread]);
// }];
// 开启
[dataTask resume];

```

##### 2\. POST请求

```
NSURLSession *session = [NSURLSession sharedSession];
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
NSMutableURLRequest *request =[NSMutableURLRequest requestWithURL:url];
request.HTTPMethod = @"POST";
request.HTTPBody = [@"username=520it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {        
     NSLog(@"%@---%@",[NSJSONSerialization JSONObjectWithData: data options:kNilOptions error:nil],[NSThread currentThread]);
}];    
[dataTask resume];

```

**注意：通过打印可以看出回调方法在子线程中调用，如果在回调方法中拿到数据刷新UI，必须要回到主线程刷新UI。**

##### 3\. 代理方法请求 需要遵循NSURLSessionDataDelegate

```
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    // 使用代理方法请求
    /** 
     参数一：配置信息
     参数二：代理
     参数三：控制代理方法在哪个线程中调用
     遵守代理:NSURLSessionDataDelegate
     */
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];    
    NSURL *url =[NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];    
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request];
    [dataTask resume]; 
}
#pragma mark  NSURLSessionDataDelegate代理方法
// 接收到服务器响应的时候调用
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    NSLog(@"didReceiveResponse 接受到服务器响应");
    // completionHandler 控制是否接受服务器返回的数据
    /** 
     typedef NS_ENUM(NSInteger, NSURLSessionResponseDisposition) {
     NSURLSessionResponseCancel = 0, // 默认，表示不接收数据
     NSURLSessionResponseAllow = 1,   // 接受数据
     NSURLSessionResponseBecomeDownload = 2,
     NSURLSessionResponseBecomeStream NS_ENUM_AVAILABLE(10_11, 9_0) = 3,
     }
     */
    completionHandler(NSURLSessionResponseAllow);
}
// 接收到服务器返回数据时调用，会调用多次
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    NSLog(@"didReceiveData 接受到服务器返回数据");
}
// 当请求完成之后调用，如果请求失败error有值
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"didCompleteWithError 请求完成");
}

```

##### 4\. NSURLSessionConfiguration 配置简单说明

**NSURLSessionConfiguration对象用于初始化NSURLSession对象。当NSURLSession开启多个任务Task的时候可以用NSURLSessionConfiguration对象统一配置。**
会话在初始化时复制它们的配置，NSURLSession有一个只读的配置属性，使得该配置对象上的变化对这个会话无效。配置在初始化时被读取一次，之后都是不会变化的。
NSURLSessionConfiguration有三个类构造函数
`defaultSessionConfiguration`返回标准默认配置，一般我们都使用这个
`ephemeralSessionConfiguration`返回一个预设配置，没有持久性存储的缓存，Cookie或证书。可以用来实现像"无痕浏览"功能的功能。
`backgroundSessionConfiguration`：独特之处在于，它会创建一个后台会话。它甚至可以在应用程序挂起，退出，崩溃的情况下运行上传和下载任务。

```
 NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
// 请求超时时间
configuration.timeoutIntervalForRequest = 10;
// 加载资源超时时间
configuration.timeoutIntervalForResource = 10;
// 蜂窝网络状态下是否可用
configuration.allowsCellularAccess = YES;

```

### 3\. NSURLSesscion 文件下载

##### 1\. NSURLSessionDownloadTask实现断点下载

NSURLSession给提供了专用用来下载的Task，NSURLSessionDownloadTask，使用NSURLSessionDownloadTask的代理方法或者本身提供的方法可以很轻松的实现断点下载。

NSURLSessionDownloadTask的创建

```
NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc]init]];
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_03.mp4"];
NSURLSessionDownloadTask *downLoadTask = [session downloadTaskWithURL:url];

```

NSURLSessionDownloadTask也提供了一些方法

```
//这个方法可以拿到恢复下载需要的数据 resumeData 暂停下载时 已经下载完成的数据
[self.downloadTask cancelByProducingResumeData:^(NSData * _Nullable resumeData) {
     self.data = resumeData;
}];

// 创建一个恢复下载的任务，需要重新启动
self.downloadTask = [self.session downloadTaskWithResumeData:self.data];
// 需要启动
[self.downloadTask resume];

```

NSURLSessionDownloadDelegate也提供了非常好用的代理方法

```
#pragma mark NSURLSessionDownloadDelegate
// 1.当接收到数据的时候,写数据,该方法会调用多次
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
 //bytesWritten:本次写入数据的大小
 //totalBytesWritten:已经下载完成的数据大小
 //totalBytesExpectedToWrite:文件大小
 //可以在这个方法中监听下载的进度 totalBytesWritten/totalBytesExpectedToWrite
}
// 2.恢复下载的时候调用
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
{
    NSLog(@"didResumeAtOffset");
}
// 3.下载完成之后调用
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{ 
// location :下载文件的存储位置，在沙盒tmp文件中。
// tmp文件保存应用运行时所需的临时数据，使用完毕后会将相应的文件从该目录中删除，应用程序关闭时，系统会清除该目录下的文件
// 程序下载完成之后我们可以将tmp中下载的文件移动到沙盒中保存。
}
// 4.请求完成之后调用
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"didCompleteWithError---%@",error);
}

```

通过以上方法可以很轻松的实现断点下载，但是使用NSURLSessionDownloadTask实现还有一些缺点，我们只有在下载完成之后才能拿到下载完成的文件，那么当我们下载到一半的时候，我们点击暂停，或者在下载过程中，直接关闭退出程序，此时因为文件是保存在内存中的，所以之前下载的文件已经不存在了，当我们重新运行程序，就需要重新下载。这种不可操纵性显然不是我们想要的。因此我们还是要使用 NSURLSessiondataTask来实现离线断点下载。

##### 2\. NSURLSessiondataTask实现文件离线断点下载

**原理：首先利用输出流实现边下载边存储数据到沙盒，另外在第一次接收到响应的时候将下载文件的大小也存储在沙盒中。然后当退出程序重新运行的时候，查看沙盒中是否有已经下载的文件，如果有就获取已经下载文件的大小，并取出沙盒中存储的文件总大小，将下载进度显示在界面，然后接着拼接下载。如果没有，则从0开始下载。**

```
#import "ViewController.h"
#import <MediaPlayer/MediaPlayer.h>

#define FileName @"xx_cc.mp4"
#define FileLength @"xx_cc.xx"
@interface ViewController ()<NSURLSessionDataDelegate>

@property(nonatomic,strong)NSOutputStream *stream;//输出流
@property(nonatomic,assign)NSInteger totalLength;// 文件总大小
@property(nonatomic,assign)NSInteger currentLength;// 已经下载大小
@property(nonatomic,strong)NSURLSession *session; 
@property(nonatomic,strong)NSURLSessionDataTask *dataTask;
@property (weak, nonatomic) IBOutlet UIProgressView *progressView;
@property (weak, nonatomic) IBOutlet UIButton *playBtn;

@end

@implementation ViewController

#pragma mark 懒加载
#pragma mark --------------------
-(NSURLSessionDataTask *)dataTask
{
    if (_dataTask == nil) {
        self.currentLength = [self getCurrent];
        NSURL *url =[NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_02.mp4"];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        NSString *range =[NSString stringWithFormat:@"bytes=%zd-",self.currentLength];
        [request setValue:range forHTTPHeaderField:@"Range"];
        _dataTask = [self.session dataTaskWithRequest:request];
    }
    return _dataTask;
}
-(NSURLSession *)session
{
    if (_session == nil) {
        // 使用代理方法请求
        /**
         参数一：配置信息
         参数二：代理
         参数三：控制代理方法在那个队列中调用
         遵守代理:NSURLSessionDataDelegate
         */
        _session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    }
    return _session;
}
#pragma mark viewDIdLoad
#pragma mark --------------------
-(void)viewDidLoad
{
    [super viewDidLoad];
    self.playBtn.enabled = NO;
    NSString *caches =[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [caches stringByAppendingPathComponent:FileLength];
    NSMutableDictionary *dict = [NSMutableDictionary dictionaryWithContentsOfFile:filePath];
    if (dict) {
        self.progressView.progress = 1.0 * [self getCurrent]/[dict[FileLength] integerValue];
        if (self.progressView.progress == 1) {
            self.playBtn.enabled = YES;
        }
    }
    NSLog(@"%@",dict);
}
#pragma mark Btn点击事件
#pragma mark --------------------
- (IBAction)startBtn:(id)sender {
    [self.dataTask resume];
}
- (IBAction)stopBtn:(id)sender {
    [self.dataTask suspend];
}
- (IBAction)playBtn:(id)sender {
    NSString *caches =[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [caches stringByAppendingPathComponent:FileName];

    NSURL*videoPathURL=[[NSURL alloc] initFileURLWithPath:filePath];

    MPMoviePlayerViewController *vc =[[MPMoviePlayerViewController alloc]initWithContentURL:videoPathURL];
    [self presentViewController:vc animated:YES completion:nil];
}
#pragma mark 方法
#pragma mark --------------------
-(NSInteger )getCurrent
{
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [caches stringByAppendingPathComponent:FileName];
    NSFileManager *manager = [NSFileManager defaultManager];
    NSDictionary *dict = [manager attributesOfItemAtPath:filePath error:nil];
    return [dict[@"NSFileSize"] integerValue];
}
-(void)saveTotal:(NSInteger )length
{
    NSLog(@"开始存储文件大小");
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [caches stringByAppendingPathComponent:FileLength];
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    [dict setObject:@(length) forKey:FileLength];
    [dict writeToFile:filePath atomically:YES];
}
#pragma mark  NSURLSessionDataDelegate代理方法
#pragma mark --------------------
// 接收到服务器响应的时候调用
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    // 拿到文件总大小 获得的是当次请求的数据大小，当我们关闭程序以后重新运行，开下载请求的数据是不同的 ,所以要加上之前已经下载过的内容
    NSLog(@"接收到服务器响应");
    self.totalLength = response.expectedContentLength + self.currentLength;
    // 把文件总大小保存的沙盒 没有必要每次都存储一次,只有当第一次接收到响应，self.currentLength为零时，存储文件总大小就可以了
    if (self.currentLength == 0) {
        [self saveTotal:self.totalLength];
    }
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [caches stringByAppendingPathComponent:FileName];
    NSLog(@"%@",filePath);
    // 创建输出流 如果没有文件会创建文件，YES：会往后面进行追加
    NSOutputStream *stream = [[NSOutputStream alloc]initToFileAtPath:filePath append:YES];
    [stream open];
    self.stream = stream;
    //NSLog(@"didReceiveResponse 接受到服务器响应");
    completionHandler(NSURLSessionResponseAllow);
}
// 接收到服务器返回数据时调用，会调用多次
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    self.currentLength += data.length;
    // 输出流写数据
    [self.stream write:data.bytes maxLength:data.length];
    NSLog(@"%f",1.0 * self.currentLength / self.totalLength);
    self.progressView.progress = 1.0 * self.currentLength / self.totalLength;
    //NSLog(@"didReceiveData 接受到服务器返回数据");
}
// 当请求完成之后调用，如果请求失败error有值
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    // 关闭stream
    [self.stream close];
    self.stream = nil;
    NSLog(@"didCompleteWithError 请求完成");
    self.playBtn.enabled = YES;
}

@end

```

![](http://upload-images.jianshu.io/upload_images/1464492-21e37a4221fa4579.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 4\. NSURLSessiond上传文件

##### 1\. NSURLSessionUploadTask上传文件

```
#import "ViewController.h"
#define Kboundary  @"----WebKitFormBoundary35cxmtFcIglrlsad"
#define KNewLine [@"\r\n" dataUsingEncoding:NSUTF8StringEncoding] 
@interface ViewController ()
@end
@implementation ViewController
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self upload];
}
-(void)upload
{
    NSURLSession *session = [NSURLSession sharedSession];
    NSURL *url =[NSURL URLWithString:@"http://120.25.226.186:32812/upload"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";
    //2.3.设置请求头
    NSString *header = [NSString stringWithFormat:@"multipart/form-data; boundary=%@",Kboundary];
    [request setValue:header forHTTPHeaderField:@"Content-Type"];

    // session上传不需要设置请求体,如果数据在request中会被忽略。
    NSURLSessionUploadTask *upLoadTask = [session uploadTaskWithRequest:request fromData:[self getBody] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {

        NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    // 开启执行
    [upLoadTask resume];
}
-(NSData *)getBody
{
    //5.设置请求体
    NSMutableData *fileData = [NSMutableData data];
    //5.1 文件参数
    /*
     --分隔符
     Content-Disposition: form-data; name="file"; filename="123.png"
     Content-Type: image/png
     空行
     文件数据
     */
    NSString *str = [NSString stringWithFormat:@"--%@",Kboundary];
    [fileData appendData:[str dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Disposition: form-data; name=\"file\"; filename=\"123.png\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Type: image/png" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];

    UIImage *image = [UIImage imageNamed:@"123"];
    NSData *imageData = UIImagePNGRepresentation(image);
    [fileData appendData:imageData];
    [fileData appendData:KNewLine];
    //5.2 非文件参数
    /*
     --分隔符
     Content-Disposition: form-data; name="username"
     空行
     yy
     */
    [fileData appendData:[str dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Disposition: form-data; name=\"username\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"yy" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];

    //5.3 结尾标识
    /*
     --分隔符--
     */
    [fileData appendData:[[NSString stringWithFormat:@"--%@--",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    return fileData;
}
@end

```

##### 2\. NSURLSessionUploadTask代理方法上传方法

```
#import "ViewController.h"
#define Kboundary  @"----WebKitFormBoundary35cxmtFcIglrlsad"
#define KNewLine [@"\r\n" dataUsingEncoding:NSUTF8StringEncoding]
@interface ViewController ()<NSURLSessionDataDelegate>
@end
@implementation ViewController
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self uploadDelegate];
}
// 代理方法上传，可以监控上传过程和结束
-(void)uploadDelegate
{
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    NSURL *url =[NSURL URLWithString:@"http://120.25.226.186:32812/upload"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";

    //2.3.设置请求头
    NSString *header = [NSString stringWithFormat:@"multipart/form-data; boundary=%@",Kboundary];
    [request setValue:header forHTTPHeaderField:@"Content-Type"];

    // session上传不需要设置请求体,如果数据在request中会被忽略。
    // 使用代理方法可以监控上传过程
    [[session uploadTaskWithRequest:request fromData:[self getBody]]resume];
}
// 设置请求体，必须严格按照格式拼接
-(NSData *)getBody
{
    //5.设置请求体
    NSMutableData *fileData = [NSMutableData data];

    //5.1 文件参数
    /*
     --分隔符
     Content-Disposition: form-data; name="file"; filename="123.png"
     Content-Type: image/png
     空行
     文件数据
     */
    NSString *str = [NSString stringWithFormat:@"--%@",Kboundary];
    [fileData appendData:[str dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Disposition: form-data; name=\"file\"; filename=\"123.png\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Type: image/png" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];

    UIImage *image = [UIImage imageNamed:@"123"];
    NSData *imageData = UIImagePNGRepresentation(image);
    [fileData appendData:imageData];
    [fileData appendData:KNewLine];

    //5.2 非文件参数
    /*
     --分隔符
     Content-Disposition: form-data; name="username"
     空行
     yy
     */

    [fileData appendData:[str dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"Content-Disposition: form-data; name=\"username\"" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:KNewLine];
    [fileData appendData:[@"yy" dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];

    //5.3 结尾标识
    /*
     --分隔符--
     */
    [fileData appendData:[[NSString stringWithFormat:@"--%@--",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
    [fileData appendData:KNewLine];

    return fileData;
}
#pragma mark NSURLSessionDataDelegate 代理方法
/*
 bytesSent:本次上传数据大小
 totalBytesSent:总共上传了多少
 totalBytesExpectedToSend:文件大小
 */
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
{
    NSLog(@"%f",1.0 *totalBytesSent /totalBytesExpectedToSend);
}
// 上传结束
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"上传结束");
}
@end

```

### 5\. NSURLSession 内存释放问题

NSURLSession 需要释放，不然会引起内存泄漏

```
-(void)dealloc
{
    //注意:在不用的时候一定要调用该方法来释放,不然会出现内存泄露问题
    //方法一：取消所有过去的会话和任务
    [self.session invalidateAndCancel];
    //方法二：可在释放时做一些操作
    [self.session resetWithCompletionHandler:^{
         // 释放时做的操作
    }];
}

```

