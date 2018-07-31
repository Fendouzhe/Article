> TouchID指纹识别是iPhone 5S设备中增加的一项重大功能.苹果的后续移动设备也相继添加了指纹功能,在实际使用中还是相当方便的,比如快捷登录,快捷支付等等.系统提供了相应框架,使用起来还是比较方便的.使用LAContext对象即可完成指纹识别,提高用户体验.

![TouchId](https://upload-images.jianshu.io/upload_images/1464492-7d10e42bdcaad61f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 提示：指纹识别必须用真机测试,并且在iOS8以上系统.

TouchID API使用

* * *

### 1.添加头文件

```
#import <LocalAuthentication/LocalAuthentication.h>

```

### 2.判断系统版本

```
 //首先判断版本
if (NSFoundationVersionNumber < NSFoundationVersionNumber_iOS_8_0) {
      NSLog(@"系统版本不支持TouchID");
      return;
}

```

### 3.LAPolicy

在这里简单介绍一下`LAPolicy`,它是一个枚举.我们根据自己的需要选择LAPolicy，它提供两个值:
`LAPolicyDeviceOwnerAuthenticationWithBiometrics`和`LAPolicyDeviceOwnerAuthentication`.
<1>. `LAPolicyDeviceOwnerAuthenticationWithBiometrics`是支持iOS8以上系统,使用该设备的TouchID进行验证,当输入TouchID验证5次失败后,TouchID被锁定,只能通过锁屏后解锁设备时输入正确的解锁密码来解锁TouchID。
<2>.`LAPolicyDeviceOwnerAuthentication`是支持iOS9以上系统,使用该设备的TouchID或设备密码进行验证，当输入TouchID验证5次失败后，TouchID被锁定，会触发设备密码页面进行验证。

### 4\. canEvaluatePolicy

使用`canEvaluatePolicy`方法判断设备是否支持TouchID，返回`BOOL`为`YES`，该设备支持TouchID。

```
 if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error]) {

```

error为返回验证错误码.具体不解释了.

### 4\. evaluatedPolicyDomainState

context.evaluatedPolicyDomainState用于判断设备上的指纹是否被更改，在LAContext被创建的时候，evaluatedPolicyDomainState才生效，可在TouchID验证成功时，将它记录下来，用于下次使用TouchID时校验，提高安全性。

### 5\. evaluatePolicy

evaluatePolicy方法是对TouchID进行验证,Block回调中如果success为YES则验证成功,为NO验证失败,并对error进行解析.

```
- (IBAction)loginButtonClick:(UIButton *)sender {

    //首先判断版本
    if (NSFoundationVersionNumber < NSFoundationVersionNumber_iOS_8_0) {
        NSLog(@"系统版本不支持TouchID");
        return;
    }

    LAContext *context = [[LAContext alloc] init];
    context.localizedFallbackTitle = @"输入密码";
    if (@available(iOS 10.0, *)) {
//        context.localizedCancelTitle = @"22222";
    } else {
        // Fallback on earlier versions
    }
    NSError *error = nil;

    if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error]) {

        [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:@"通过Home键验证已有手机指纹" reply:^(BOOL success, NSError * _Nullable error) {

            if (success) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSLog(@"TouchID 验证成功");
                });
            }else if(error){

                switch (error.code) {
                    case LAErrorAuthenticationFailed:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 验证失败");
                        });
                        break;
                    }
                    case LAErrorUserCancel:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 被用户手动取消");
                        });
                    }
                        break;
                    case LAErrorUserFallback:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"用户不使用TouchID,选择手动输入密码");
                        });
                    }
                        break;
                    case LAErrorSystemCancel:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 被系统取消 (如遇到来电,锁屏,按了Home键等)");
                        });
                    }
                        break;
                    case LAErrorPasscodeNotSet:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 无法启动,因为用户没有设置密码");
                        });
                    }
                        break;
                    case LAErrorTouchIDNotEnrolled:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 无法启动,因为用户没有设置TouchID");
                        });
                    }
                        break;
                    case LAErrorTouchIDNotAvailable:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 无效");
                        });
                    }
                        break;
                    case LAErrorTouchIDLockout:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"TouchID 被锁定(连续多次验证TouchID失败,系统需要用户手动输入密码)");
                        });
                    }
                        break;
                    case LAErrorAppCancel:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"当前软件被挂起并取消了授权 (如App进入了后台等)");
                        });
                    }
                        break;
                    case LAErrorInvalidContext:{
                        dispatch_async(dispatch_get_main_queue(), ^{
                            NSLog(@"当前软件被挂起并取消了授权 (LAContext对象无效)");
                        });
                    }
                        break;
                    default:
                        break;
                }
            }
        }];

    }else{
        NSLog(@"当前设备不支持TouchID");
    }
}

```

上面这个代码, 是整个TouchID的核心,也几乎是所有代码了.

### 验证

验证必须使用真机

![结果.jpg](https://upload-images.jianshu.io/upload_images/1464492-4a6a51e7df4a0133.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![错误的时候.jpg](https://upload-images.jianshu.io/upload_images/1464492-727a760836cab57b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 总结:TouchID使用起来不难,重要的是使用流程逻辑.

以登录为例,一般来说流程是这样的:
1.**开启指纹登录**:首次登陆使用密码登录,登录后,可以设置一个开启指纹ID登录的按钮,来进行指纹认证.
2.**验证**:检测是否支持TouchID.
3.**生成设备账号/密码**:TouchID验证通过后，根据当前已登录的账号和硬件设备Token，生成设备账号/密码（规则可自定，密码要长要复杂），并保存在keychain；
4.**绑定**：生成设备账号/密码后，将原账号及设备账号/密码，加密后（题主使用的是RSA加密）发送到服务端进行绑定；
5.**成功**：验证原账号及设备账号有效后，返回相应状态，绑定成功则完成整个TouchID（设备）绑定流程。

文章转载自：[https://www.jianshu.com/p/9990b0f48488](https://www.jianshu.com/p/9990b0f48488)
