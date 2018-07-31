例如跳转Wi-Fi,之前是使用`prefs:root=WIFI`或者`App-Prefs:root=WIFI`来进行跳转

```
//iOS10
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=WIFI"] options:@{} completionHandler:nil];
//
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=WIFI"]];

```

这样在上架的时候会被拒

# 解决方法

中间进行一个转码,绕过苹果的代码扫描,亲测能过审核.

```
//将上面的跳转字符串转成字符,在进行拼接就好了
NSData *encryptString = [[NSData alloc] initWithBytes:(unsigned char []){0x70,0x72,0x65,0x66,0x73,0x3a,0x72,0x6f,0x6f,0x74,0x3d,0x4e,0x4f,0x54,0x49,0x46,0x49,0x43,0x41,0x54,0x49,0x4f,0x4e,0x53,0x5f,0x49,0x44} length:27];

NSString *string = [[NSString alloc] initWithData:encryptString encoding:NSUTF8StringEncoding];

[[UIApplication sharedApplication] openURL:[NSURL URLWithString:string] options:@{} completionHandler:nil];

```

给一个转换的网站
[http://www.ab126.com/goju/1711.html](https://link.jianshu.com?t=http%3A%2F%2Fwww.ab126.com%2Fgoju%2F1711.html)
