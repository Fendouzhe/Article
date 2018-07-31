最近在项目中有个这样的需求:整个APP中界面也竖屏为主,且不能自动横竖屏切换,在线学习视频播放界面可以根据手机的方向横竖屏切换;其实实现起来也并不难,关于视图是否能旋转主要还是有没有设置支持,在工程的General-->Device Orientation里可以进行这些设置:

![](http://upload-images.jianshu.io/upload_images/1464492-b4f0375db9069b4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这些设置后就可以在项目中用代码控制了,控制视图是否能够自动旋转,支持哪些方向主要是用了下面的三个方法:

```
// New Autorotation support.  
//是否自动旋转,返回YES可以自动旋转  
- (BOOL)shouldAutorotate NS_AVAILABLE_IOS(6_0) __TVOS_PROHIBITED;  
//返回支持的方向  
- (UIInterfaceOrientationMask)supportedInterfaceOrientations NS_AVAILABLE_IOS(6_0) __TVOS_PROHIBITED;  
// Returns interface orientation masks.  
//这个是返回优先方向  
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation NS_AVAILABLE_IOS(6_0) __TVOS_PROHIBITED;  
```
一般情况下实现前两个方法即可!!这些都是UIViewController的实例方法,直接在需要设置的控制器重写就行...
可能,你会发现,有时候重写了,但是并没有用,明明返回了NO,但是视图还是会根据手机方向自动旋转,这里就要注意一下了,当前的设置必须是影响了工程的跟视图控制器才行,如果你在跟视图控制器里做如下设置:

```
- (BOOL)shouldAutorotate{  
    return NO;  
}  
```

那么无论你怎么旋转手机,视图是不会跟着转的.很多时候,我们需要旋转的页面并不是跟视图,这就要想办法告诉跟视图,这个界面我设置了可以自动旋转,跟视图控制器再进行相应的设置,才会有效果!
怎么告诉跟视图控制器呢?这要视情况而定了:
1.当前viewcontroller就是跟视图控制器
这种最简单,直接重写上面的方法,设置可以支持的方向,和是否自动旋转即可;这种情况比较少见
2.项目的跟视图控制器是导航(UINavigationController)
这种情况也很简单,可以自定义一个导航类,在这个导航类里实现如下方法:

```
-(BOOL)shouldAutorotate  
{  
    return [[self.viewControllers lastObject] shouldAutorotate];  
      
}  
  
#if __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_9_0  
- (NSUInteger)supportedInterfaceOrientations  
#else  
- (UIInterfaceOrientationMask)supportedInterfaceOrientations  
#endif  
{  
     return [[self.viewControllers lastObject] supportedInterfaceOrientations];  
}  
  
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {  
      
    return [[self.viewControllers lastObject] preferredInterfaceOrientationForPresentation];  
}  
```

然后在需要特殊设置的控制器内,重写这几个方法即可.
上面的[self.viewControllers lastObject]是获取当前导航的栈顶的控制器,用他去调自己重写的方法,返回需要设置的值
这样也能达到控制某个页面是否支持旋转的目的;
还有一种方法,如果需要自动旋转的页面能够预知,可以在这个跟视图导航里做如下设置:

```
-(BOOL)shouldAutorotate  
{  
    if ([[self.viewControllers lastObject]isKindOfClass:[FirstViewController class]]) {  
        return YES;  
    }  
      
    return NO;  
//    return [[self.viewControllers lastObject] shouldAutorotate];  
      
} 
```

上面是设置FirstViewController可以自动旋转,其他界面不能自动旋转;这种方法的好处是不需要在需要旋转的视图控制器重写上面的方法了;
3.项目的跟视图是UITabBarController
很多情况下都是这样,项目的跟视图是一个tabBar,其实明白了原理,实现起来也不难,就是怎么告诉跟视图控制器,这个界面需要支持自动旋转;由于在tabBar里直接获取需要旋转的控制器比较困难,所以,本人采用通知的形式:
在跟视图控制器(tabBar)中设置一个全局的BOOL值

```
BOOL shouldAutorotate;  
```

用于接收通知发送来的结果,并设置跟视图的返回结果;

```
/** 
 * 
 *  @return 是否支持旋转 
 */  
-(BOOL)shouldAutorotate  
{  
    NSLog(@"======%d",shouldAutorotate);  
    if (!shouldAutorotate) {  
        return NO;  
    }else{  
        return YES;  
    }  
}  
  
/** 
 *  适配旋转的类型 
 * 
 *  @return 类型 
 */  
-(UIInterfaceOrientationMask)supportedInterfaceOrientations  
{  
    if (!shouldAutorotate) {  
        return UIInterfaceOrientationMaskPortrait;  
    }  
    return UIInterfaceOrientationMaskAllButUpsideDown;  
}  
```

最后添加一个通知监听,并实现通知方法:

```
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(AutorotateInterface:) name:@"InterfaceOrientation" object:nil];  
  
  
-(void)AutorotateInterface:(NSNotification *)notifition  
{  
    shouldAutorotate = [notifition.object boolValue];  
}  
```

在通知调用的方法里,将传过来的值赋给shouldAutorotate,然后跟视图会根据shouldAutorotate的值设置响应的结果;
最后在需要自动转屏的控制器里发送通知即可:

```
[[NSNotificationCenter defaultCenter] postNotificationName:@"InterfaceOrientation" object:@"YES"];  
```
最后介绍一个强制转屏的方法,这个方法可以在进入某个视图时,强制转成你需要的屏幕方向,用的比较多的是在一个竖屏的应用中强制转换某一个界面为横屏(例如播放视频):
```
//强制旋转屏幕  
- (void)orientationToPortrait:(UIInterfaceOrientation)orientation {  
    SEL selector = NSSelectorFromString(@"setOrientation:");  
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[UIDevice instanceMethodSignatureForSelector:selector]];  
    [invocation setSelector:selector];  
    [invocation setTarget:[UIDevice currentDevice]];  
    int val = orientation;  
    [invocation setArgument:&val atIndex:2];//前两个参数已被target和selector占用  
    [invocation invoke];  
      
} 
```
调用的时候只需把你想要的方向传过去即可!!
使用的时候有个点需要注意,从A进入B的时候,把B强制转换成横屏,返回的时候,需要在A出现的时候再转换为原来的方向,不然会有问题;个人建议可以在B的viewWillAppear调用这个方法,转换屏幕(例如转换为横屏),然后在A的viewWillAppear中转换回来;

最后附上以上内容的[Demo地址](https://github.com/Fendouzhe/LRInterfaceRotation)
