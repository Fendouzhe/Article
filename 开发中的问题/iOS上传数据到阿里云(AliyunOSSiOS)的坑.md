我们公司项目要做在线培训，在我们上传培训图片和语音的时候。伟大的后台告诉我，前台应该直接向阿里传递数据，这样的路径是：iOS端—>阿里。以往我们的传递路径都是：iOS端—>后台—>阿里。

* * *

# 正文

首先，给大家阿里sdk的[github地址](https://link.jianshu.com?t=https://github.com/sunyunfei/aliyun-oss-ios-sdk.git),在这里你可以下载他们的idk，看一下他们的介绍。

想上传到阿里，首先要有对应的阿里的账号参数，填写到这里

> NSString * const AccessKey = @"";
> 
> NSString * const SecretKey = @"";

OSSClient是OSS服务的iOS客户端，它为调用者提供了一系列的方法，用于和OSS服务进行交互。一般来说，全局内只需要保持一个OSSClient，用来调用各种操作。

上面这一句是阿里的说法，那我们就听从人家的建议吧：

```

@interface ViewController (){

OSSClient * client;

}

```

用明文AK/SK实现的加签器（官方建议只在测试模式时使用）

> NSString *endpoint = @"自己的参数";//比如http://ios.ali.comidcredential = [[OSSPlainTextAKSKPairCredentialProvider alloc] initWithPlainTextAccessKey:AccessKey                                                                                                     secretKey:SecretKey];
> 
> client = [[OSSClient alloc] initWithEndpoint:endpoint credentialProvider:credential];
> 
> OSSPutObjectRequest * put = [OSSPutObjectRequest new];

Bucket名称， Object名称

> put.bucketName = @"自己的数据";
> 
> put.objectKey = @"自己的数据";

上传数据，有两种方式：

1.data上传

> put.uploadingData = self.imageData;//自己的NSData数据

2.上传路径

> put.uploadingFileURL = [NSURL fileURLWithPath:fullPath];

当然，上传的时候你也可以看一看你的上传进度,还有一些参数配置（这一步不是必须的）：

> put.uploadProgress = ^(int64_t bytesSent, int64_t totalByteSent, int64_t totalBytesExpectedToSend) {
> 
> NSLog(@"%lld, %lld, %lld", bytesSent, totalByteSent, totalBytesExpectedToSend);
> 
> };

所有调用api的操作，都会立即获得一个OSSTask。这是官方的说法。这一步是最重要的一步，成不成功就看他了。

> OSSTask * putTask = [client putObject:put];
> 
> [putTask continueWithBlock:^id(OSSTask *task) {
> 
> task = [client presignPublicURLWithBucketName:@"同上面的bucketName"
> 
> withObjectKey:同上];
> 
> NSLog(@"objectKey: %@", put.objectKey);
> 
> if (!task.error) {
> 
> OSSPutObjectResult * result = task.result;
> 
> NSLog(@"upload object success!");
> 
> NSLog(@"Result - requestId: %@",
> 
> result.requestId);
> 
> } else {
> 
> NSLog(@"upload object failed, error: %@" , task.error);
> 
> OSSPutObjectResult * result = task.result;
> 
> NSLog(@"requestId: %@",
> 
> result.requestId);
> 
> }
> 
> return nil;
> 
> }];

好了，这就是全部的上传的代码。关于最后一步我要说一下，官方给的代码是：

> OSSTask * putTask = [client putObject:put];
> 
> [putTask continueWithBlock:^id(OSSTask *task) {
> 
> if (!task.error) {
> 
> NSLog(@"upload object success!");
> 
> } else {
> 
> NSLog(@"upload object failed, error: %@" , task.error);
> 
> }
> 
> return nil;
> 
> }];

没有这一句代码：

> task = [client presignPublicURLWithBucketName:@"同上面的bucketName"
> 
> withObjectKey:同上];


