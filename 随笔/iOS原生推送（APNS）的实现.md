原文链接：http://www.jianshu.com/p/94cec7d435e8

#### 1.前期准备工作

创建你的APNs keys 或者 创建推送证书，这两个创建一个即可实现推送。

 1.  创建你的APNs keys

       首先来到你的开发者 Certificates, Identifiers & Profiles—>Keys—>点击+号，如下图

![](http://upload-images.jianshu.io/upload_images/1464492-9b00347a3856219c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


分别填写key的name，勾选用途，点击continue，如下图

![](http://upload-images.jianshu.io/upload_images/1464492-c0c09b5f53ddcf87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后点击confrim—> Download这里需要注意你下载好keys的.p8文件后一定要保存好，因为他只能下载一次。然后把keys的.p8交给后台用做推送。

2\. 证书的创建

（1.）进入苹果Apple Developer -> Member Center -> Certificates, Identifiers & Profiles – >Identifiers - >App IDs–>Edit

![](http://upload-images.jianshu.io/upload_images/1464492-8c0d9fc3a109342d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如上图所示，勾选Push Notifications，然后点击下面的Create Certificate，分别创建测试环境与生产环境的SSL推送证书。

（2）用证书文件与私钥合成.pem文件给后台的同学

完成第（1）步后，然后在进入苹果Apple Developer -> Member Center -> Certificates, Identifiers & Profiles – >Certificates，找到刚才的推送证书然后下载->双击安装。在钥匙串中找到这两个证书（production&developerment）。分别导出.p12文件（证书的p12与密钥的p12），如下图。

![](http://upload-images.jianshu.io/upload_images/1464492-11522ded1e31a421.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


得到证书的p12与密钥的p12后，打开命令行把p12文件转化为.pem文件

假设我的证书的p12与密钥的p12
分别命名为：apns-dev-cert.p12；apns-dev-key.p12

首先cd到这两个文件的目录下，使用下面的命令分别得到apns-dev-cert.pem;   apns-dev-key.pem;

```
openssl pkcs12 -clcerts -nokeys -out apns-dev-cert.pem -in apns-dev-cert.p12
openssl pkcs12 -nocerts -out apns-dev-key.pem -in apns-dev-key.p12
```

然后再用apns-dev-cert.pem;   apns-dev-key.pem;合成最终后台可以用来推送的apns-dev.pem文件。你把apns-dev.pem文件交给后台的小伙伴就可以了。

```
cat apns-dev-cert.pem apns-dev-key.pem > apns-dev.pem
```

production环境的.pem与developerment一样。

[这篇文章](https://www.jianshu.com/p/cc952ea07a08?mType=Group) 详细的介绍了.pem如何制作的。

#### 2.实现推送

打开你的项目->capabilication打开push notifications与background Modes，勾选最后一个remote  notifications。

![](http://upload-images.jianshu.io/upload_images/1464492-cfc313b50bcd6b38.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果是iOS10以上版本还需要引入UserNotifications.framework、UserNotificationsUI.framework这两个framework

然后再application引入头文件#import  <UserNotifications/UserNotifications.h>

（1）注册推送

```
    if (IOS_VERSION >= 10.0) {
        
        UNUserNotificationCenter * center = [UNUserNotificationCenter currentNotificationCenter];
        
        [center setDelegate:self];
        
        UNAuthorizationOptions type = UNAuthorizationOptionBadge|UNAuthorizationOptionSound|UNAuthorizationOptionAlert;
        
        [center requestAuthorizationWithOptions:type completionHandler:^(BOOL granted, NSError * _Nullable error) {
            
            if (granted) {
                
                DBLog(@"注册成功");
                
            }else{
                
                DBLog(@"注册失败");
                
            }
            
        }];
        
    }else if (IOS_VERSION >= 8.0){
        
        UIUserNotificationType notificationTypes = UIUserNotificationTypeBadge |
        
        UIUserNotificationTypeSound |
        
        UIUserNotificationTypeAlert;
        
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:notificationTypes categories:nil];
        
        [application registerUserNotificationSettings:settings];
        
    }else{//ios8一下
        
        UIRemoteNotificationType notificationTypes = UIRemoteNotificationTypeBadge |
        
        UIRemoteNotificationTypeSound |
        
        UIRemoteNotificationTypeAlert;
        
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:notificationTypes];
        
    }
    
    // 注册获得device Token
    
    [application registerForRemoteNotifications];
```

在application里面的```- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {}```

中添加上面的代码。

（2）获取Token

```
// 将得到的deviceToken传给SDK
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{

      NSString *deviceTokenStr = [[[[deviceToken description]

      stringByReplacingOccurrencesOfString:@"<" withString:@""]

      stringByReplacingOccurrencesOfString:@">" withString:@""]

      stringByReplacingOccurrencesOfString:@" " withString:@""];

      NSLog(@"deviceTokenStr:\n%@",deviceTokenStr);

}

// 注册deviceToken失败
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error{

      NSLog(@"error -- %@",error);
}
```

在application里面的添加上面两个方法。然后再获取设备token成功的方法里面，我们需要把获取到的设备的token发送给后台，然后后台拿token去推送。

（3）处理推送过来的消息

1.iOS10以上版本的处理

```
//在前台
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler{
// 需要执行这个方法，选择是否提醒用户，有Badge、Sound、Alert三种类型可以设置

   completionHandler(UNNotificationPresentationOptionBadge|

   UNNotificationPresentationOptionSound|

   UNNotificationPresentationOptionAlert);

}
```

上面的这个方法，加上```completionHandler(UNNotificationPresentationOptionBadge | UNNotificationPresentationOptionSound | UNNotificationPresentationOptionAlert)```;

用户即使在前台，收到推送时通知栏也会出现，有声音和角标。如果去掉应用在前台有推送时并不会收到。

```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler{
//处理推送过来的数据

  [self handlePushMessage:response.notification.request.content.userInfo];

  completionHandler();

}
```

这个方法是在用户点击了消息栏的通知，进入app后会来到这里。我们可以业务逻辑。比如跳转到相应的页面等。

2.iOS10以下的处理

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary * _Nonnull)userInfo fetchCompletionHandler:(void (^ _Nonnull)(UIBackgroundFetchResult))completionHandler{
NSLog(@"didReceiveRemoteNotification:%@",userInfo);

   /*

   UIApplicationStateActive 应用程序处于前台

   UIApplicationStateBackground 应用程序在后台，用户从通知中心点击消息将程序从后台调至前台

   UIApplicationStateInactive 用用程序处于关闭状态(不在前台也不在后台)，用户通过点击通知中心的消息将客户端从关闭状态调至前台

   */

   //应用程序在前台给一个提示特别消息

   if (application.applicationState == UIApplicationStateActive) {

      //应用程序在前台
      [self createAlertViewControllerWithPushDict:userInfo];

    }else{

       //其他两种情况，一种在后台程序没有被杀死，另一种是在程序已经杀死。用户点击推送的消息进入app的情况处理。
       [self handlePushMessage:userInfo];

    }

       completionHandler(UIBackgroundFetchResultNewData);

}
```

在application里面的添加上面的方法，iOS10以下，应用在前台的时候，有推送来，会直接来到这个方法。但是通知栏不会有提示，角标也不会有。应用如果在后台或者在关闭状态，点击推送来的消息也会来到这个方法。我们可以在这里处理业务逻辑。

#### 3.测试推送是否成功

到这里我们完成了基本的推送功能，但是是否能够成功还不知道？我们可以测试一下。[这里是一个测试推送的软件](https://link.jianshu.com?t=https://github.com/shaojiankui/SmartPush) 大家从github下载下来，直接运行。

1.把你的证书路径、token以及要推送的消息放到指定的地方。

2.点击连接服务器，这个时候会有访问钥匙串的请求，允许访问钥匙串。

3.点击发送。

#### 4.iOS10推送的进阶使用

iOS10还可以实现推送页面UI的自定义，以及添加事件，下一篇文章[《iOS原生推送（APNS）进阶iOS10推送图片、视频、音乐》](http://www.jianshu.com/p/9eae61bcc42e)会介绍。

![](http://upload-images.jianshu.io/upload_images/1464492-2c6241c097bc8124.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
