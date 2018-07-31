
![效果.gif](http://upload-images.jianshu.io/upload_images/1464492-74b0f05c598390ad.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

iOS7 开始苹果推出了自定义转场的 API 。从此，任何可以用 CoreAnimation 实现的动画，都可以出现在两个 ViewController 的切换之间。并且实现方式高度解耦，这也意味着在保证代码干净的同时想要替换其他动画方案时只需简单改一个类名就可以了，真正体会了一把高颜值代码带来的愉悦感。

其实网上关于自定义转场动画的教程很多，这里我是希望同学们能易懂，易上手。
转场分两种Push和Modal,所以自定义转场动画也就肯定分两种，今天我们讲的是Push

#自定义转场动画Push

首先搭建界面，添加4个按钮：
```
- (void)addButton{
    
    self.buttonArr = [NSMutableArray array];
    
    CGFloat margin=50;
    CGFloat width=(self.view.frame.size.width-margin*3)/2;
    CGFloat height = width;
    CGFloat x = 0;
    CGFloat y = 0;
    //列
    NSInteger col = 2;
    for (NSInteger i = 0; i < 4; i ++) {
        
        x = margin + (i%col)*(margin+width);
        y = margin + (i/col)*(margin+height) + 150;
        
        UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        button.frame = CGRectMake(x, y, width, height);
        button.layer.cornerRadius = width * 0.5;
        [button addTarget:self action:@selector(btnclick:) forControlEvents:UIControlEventTouchUpInside];
        button.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1.0];
        button.tag = i+1;
        [self.view addSubview:button];
        [self.buttonArr addObject:button];
    }

}
```
添加动画：
```
- (void)setupButtonAnimation{
    
    [self.buttonArr enumerateObjectsUsingBlock:^(UIButton * _Nonnull button, NSUInteger idx, BOOL * _Nonnull stop) {
        
        // positionAnimation
        CAKeyframeAnimation *positionAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
        positionAnimation.calculationMode = kCAAnimationPaced;
        positionAnimation.fillMode = kCAFillModeForwards;
        positionAnimation.repeatCount = MAXFLOAT;
        positionAnimation.autoreverses = YES;
        positionAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
        positionAnimation.duration = (idx == self.buttonArr.count - 1) ? 4 : 5+idx;

        UIBezierPath *positionPath = [UIBezierPath bezierPathWithOvalInRect:CGRectInset(button.frame, button.frame.size.width/2-5, button.frame.size.height/2-5)];
        positionAnimation.path = positionPath.CGPath;
        [button.layer addAnimation:positionAnimation forKey:nil];
        
        // scaleXAniamtion
        CAKeyframeAnimation *scaleXAniamtion = [CAKeyframeAnimation animationWithKeyPath:@"transform.scale.x"];
        scaleXAniamtion.values = @[@1.0,@1.1,@1.0];
        scaleXAniamtion.keyTimes = @[@0.0,@0.5,@1.0];
        scaleXAniamtion.repeatCount = MAXFLOAT;
        scaleXAniamtion.autoreverses = YES;
        scaleXAniamtion.duration = 4+idx;
        [button.layer addAnimation:scaleXAniamtion forKey:nil];
        
        // scaleYAniamtion
        CAKeyframeAnimation *scaleYAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform.scale.y"];
        scaleYAnimation.values = @[@1,@1.1,@1.0];
        scaleYAnimation.keyTimes = @[@0.0,@0.5,@1.0];
        scaleYAnimation.autoreverses = YES;
        scaleYAnimation.repeatCount = YES;
        scaleYAnimation.duration = 4+idx;
        [button.layer addAnimation:scaleYAnimation forKey:nil];
        
    }];
}
```

界面搭建好了：
![](http://upload-images.jianshu.io/upload_images/1464492-80423875febe5729.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后想在Push的时候实现自定义转场动画首先要遵守一个协议UINavigationControllerDelegate

苹果在 UINavigationControllerDelegate 中给出了几个协议方法，通过返回类型就可以很清楚地知道各自的具体作用。

```
//用来自定义转场动画
- (nullable id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC  NS_AVAILABLE_IOS(7_0);

```

```
//为这个动画添加用户交互

- (nullable id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController NS_AVAILABLE_IOS(7_0);

```

在第一个方法里只要返回一个准守UIViewControllerInteractiveTransitioning协议的对象,并在里面实现动画即可

1.  创建继承自 NSObject 并且声明 UIViewControllerAnimatedTransitioning 的的动画类。

2.  重载 UIViewControllerAnimatedTransitioning 中的协议方法。

```
//返回动画时间
- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;

//将动画的代码写到里面即可
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;

```

首先我自定义一个名为LRTransitionPushController的类继承于NSObject准守了UIViewControllerAnimatedTransitioning协议

```

 - (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext{
    
    self.transitionContext = transitionContext;
    
    //获取源控制器 注意不要写成 UITransitionContextFromViewKey
    LRTransitionPushController *fromVc = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    //获取目标控制器 注意不要写成 UITransitionContextToViewKey
    LRTransitionPopController *toVc = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    
    //获得容器视图
    UIView *containView = [transitionContext containerView];
    // 都添加到container中。注意顺序 目标控制器的view需要后面添加
    [containView addSubview:fromVc.view];
    [containView addSubview:toVc.view];
    
    UIButton *button = fromVc.button;
    //绘制圆形
    UIBezierPath *startPath = [UIBezierPath bezierPathWithOvalInRect:button.frame];
    
    //创建两个圆形的 UIBezierPath 实例；一个是 button 的 size ，另外一个则拥有足够覆盖屏幕的半径。最终的动画则是在这两个贝塞尔路径之间进行的
    //按钮中心离屏幕最远的那个角的点
    CGPoint finalPoint;
    //判断触发点在那个象限
    if(button.frame.origin.x > (toVc.view.bounds.size.width / 2)){
        if (button.frame.origin.y < (toVc.view.bounds.size.height / 2)) {
            //第一象限
            finalPoint = CGPointMake(0, CGRectGetMaxY(toVc.view.frame));
        }else{
            //第四象限
            finalPoint = CGPointMake(0, 0);
        }
    }else{
        if (button.frame.origin.y < (toVc.view.bounds.size.height / 2)) {
            //第二象限
            finalPoint = CGPointMake(CGRectGetMaxX(toVc.view.frame), CGRectGetMaxY(toVc.view.frame));
        }else{
            //第三象限
            finalPoint = CGPointMake(CGRectGetMaxX(toVc.view.frame), 0);
        }
    }
    
    CGPoint startPoint = CGPointMake(button.center.x, button.center.y);
    //计算向外扩散的半径 = 按钮中心离屏幕最远的那个角距离 - 按钮半径
    CGFloat radius = sqrt((finalPoint.x-startPoint.x) * (finalPoint.x-startPoint.x) + (finalPoint.y-startPoint.y) * (finalPoint.y-startPoint.y)) - sqrt(button.frame.size.width/2 * button.frame.size.width/2 + button.frame.size.height/2 * button.frame.size.height/2);
    UIBezierPath *endPath = [UIBezierPath bezierPathWithOvalInRect:CGRectInset(button.frame, -radius, -radius)];
    
    //赋值给toVc视图layer的mask
    CAShapeLayer *maskLayer = [CAShapeLayer layer];
    maskLayer.path = endPath.CGPath;
    toVc.view.layer.mask = maskLayer;
    
    CABasicAnimation *maskAnimation =[CABasicAnimation animationWithKeyPath:@"path"];
    maskAnimation.fromValue = (__bridge id)startPath.CGPath;
    maskAnimation.toValue = (__bridge id)endPath.CGPath;
    maskAnimation.duration = [self transitionDuration:transitionContext];
    maskAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    maskAnimation.delegate = self;
    [maskLayer addAnimation:maskAnimation forKey:@"path"];
    
}

```

在控制器里面用来自定义转场动画的方法里返回刚才自定义的动画类

```

- (id<UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController animationControllerForOperation:(UINavigationControllerOperation)operation fromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC{
    
    if (operation == UINavigationControllerOperationPush) {
        return [LRTranstionAnimationPush new];
    }else{
        return nil;
    }
}

```

到此为止自定义转场动画就完成了
pop的动画只是把push动画反过来做一遍这里就不细讲了，有疑问的可以去看代码

#添加滑动返回手势
上面说到这个方法是为这个动画添加用户交互的所以我们要在pop时实现滑动返回
最简单的方式应该就是利用UIKit提供的**UIPercentDrivenInteractiveTransition**类了，这个类已经实现了**UIViewControllerInteractiveTransitioning**协议，同学men可以通过这个类的对象指定转场动画的完成百分比。

```
//为这个动画添加用户交互

- (nullable id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController NS_AVAILABLE_IOS(7_0);

```

##### 第一步 添加手势

```

  UIPanGestureRecognizer *gestureRecognizer = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handlePan:)];
  [self.view addGestureRecognizer:gestureRecognizer];

```

##### 第二步 通过用户滑动的变化确定动画执行的比例

```

- (void)handlePan:(UIPanGestureRecognizer *)gestureRecognizer {
    
    /*调用UIPercentDrivenInteractiveTransition的updateInteractiveTransition:方法可以控制转场动画进行到哪了，
     当用户的下拉手势完成时，调用finishInteractiveTransition或者cancelInteractiveTransition，UIKit会自动执行剩下的一半动画，
     或者让动画回到最开始的状态。*/
    
    
    if ([gestureRecognizer translationInView:self.view].x>=0) {
        //手势滑动的比例
        CGFloat per = [gestureRecognizer translationInView:self.view].x / (self.view.bounds.size.width);
        per = MIN(1.0,(MAX(0.0, per)));
        
        if (gestureRecognizer.state == UIGestureRecognizerStateBegan) {
            
            self.interactiveTransition = [UIPercentDrivenInteractiveTransition new];
            [self.navigationController popViewControllerAnimated:YES];
            
        } else if (gestureRecognizer.state == UIGestureRecognizerStateChanged){
            
            if([gestureRecognizer translationInView:self.view].x ==0){
                
                [self.interactiveTransition updateInteractiveTransition:0.01];
                
            }else{
                
                [self.interactiveTransition updateInteractiveTransition:per];
            }
            
        } else if (gestureRecognizer.state == UIGestureRecognizerStateEnded || gestureRecognizer.state == UIGestureRecognizerStateCancelled){
            
            if([gestureRecognizer translationInView:self.view].x == 0){
                
                [self.interactiveTransition cancelInteractiveTransition];
                self.interactiveTransition = nil;
                
            }else if (per > 0.5) {
                
                [ self.interactiveTransition finishInteractiveTransition];

            }else{
                
                [ self.interactiveTransition cancelInteractiveTransition];
            }
            self.interactiveTransition = nil;
        }
        
        
    } else if (gestureRecognizer.state == UIGestureRecognizerStateChanged){

        [self.interactiveTransition updateInteractiveTransition:0.01];
        [self.interactiveTransition cancelInteractiveTransition];
  
    } else if ((gestureRecognizer.state == UIGestureRecognizerStateEnded || gestureRecognizer.state == UIGestureRecognizerStateCancelled)){
        
        self.interactiveTransition = nil;
    }
    
}

```

##### 第三步 在为动画添加用户交互的代理方法里返回UIPercentDrivenInteractiveTransition的实例

```
- (id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController {
    return self.interactiveTransition;
}

```

**如果感觉这篇文章对您有所帮助，顺手点个喜欢，谢谢啦**
代码已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，喜欢可以star.
