效果（gift掉帧显得卡，实际效果见[GitHub](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FFendouzhe%2FLRAnimations)）：

![](http://upload-images.jianshu.io/upload_images/1464492-f4bb612c9d723ed5.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 实现步骤：
##### 1 创建粒子Layer
```
    // 创建粒子Layer
    CAEmitterLayer *snowEmitter = [CAEmitterLayer layer];
    // 粒子发射位置
    snowEmitter.emitterPosition = CGPointMake(120,0);
    // 发射源的尺寸大小
    snowEmitter.emitterSize     = self.view.bounds.size;
    // 发射模式
    snowEmitter.emitterMode     = kCAEmitterLayerSurface;
    // 发射源的形状
    snowEmitter.emitterShape    = kCAEmitterLayerLine;
    snowEmitter.shadowOpacity = 1.0;
    snowEmitter.shadowRadius  = 0.0;
    snowEmitter.shadowOffset  = CGSizeMake(0.0, 0.0);
    // 粒子边缘的颜色
    snowEmitter.shadowColor  = [[UIColor whiteColor] CGColor];
    // 将粒子Layer添加进图层中
    [self.view.layer addSublayer:snowEmitter];
```
##### 2 创建粒子添加到CAEmitterLayer
```
    // 创建雪花类型的粒子
    CAEmitterCell *snowflake    = [CAEmitterCell emitterCell];
    // 粒子的名字
    snowflake.name = @"snow";
    // 粒子参数的速度乘数因子
    snowflake.birthRate = 20.0;
    // 粒子生命周期
    snowflake.lifetime  = 120.0;
    // 粒子速度
    snowflake.velocity  = 10.0;
    // 粒子的速度范围
    snowflake.velocityRange = 10;
    // 粒子y方向的加速度分量
    snowflake.yAcceleration = 2;
    // 周围发射角度
    snowflake.emissionRange = 0.5 * M_PI;
    // 子旋转角度范围
    snowflake.spinRange = 0.25 * M_PI;
    snowflake.contents  = (id)[[UIImage imageNamed:@"snow"] CGImage];
    // 设置雪花形状的粒子的颜色
    snowflake.color      = [[UIColor whiteColor] CGColor];
    snowflake.redRange   = 2.f;
    snowflake.greenRange = 2.f;
    snowflake.blueRange  = 2.f;
    snowflake.scaleRange = 0.6f;
    snowflake.scale      = 0.7f;
    // 添加粒子
    snowEmitter.emitterCells = @[snowflake];
```
运行效果:

![](http://upload-images.jianshu.io/upload_images/1464492-d9fd3e87c175b149.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3 给CAEmitterLayer创建遮盖
```
    UIImage *image      = [UIImage imageNamed:@"alpha"];
    _movedMask          = [CALayer layer];
    _movedMask.frame    = (CGRect){CGPointZero, image.size};
    _movedMask.contents = (__bridge id)(image.CGImage);
    _movedMask.position = self.view.center;
    snowEmitter.mask    = _movedMask;
```
运行：

![](http://upload-images.jianshu.io/upload_images/1464492-c086244913cd5ae6.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4 创建拖拽的View添加拖动手势
```
   // 拖拽的View 和遮罩一样位置
    UIView *dragView = [[UIView alloc] initWithFrame:_movedMask.frame];
    [self.view addSubview:dragView];
    // 给dragView添加拖拽手势
    UIPanGestureRecognizer *recognizer = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handlePan:)];
    [dragView addGestureRecognizer:recognizer];
```

##### 5 实现拖动方法，拖动view也让遮罩一起拖动。
```
- (void)handlePan:(UIPanGestureRecognizer *)recognizer {
    // 拖拽
    CGPoint translation    = [recognizer translationInView:self.view];
    recognizer.view.center = CGPointMake(recognizer.view.center.x + translation.x,
                                         recognizer.view.center.y + translation.y);
    [recognizer setTranslation:CGPointMake(0, 0) inView:self.view];
    
    // 关闭CoreAnimation实时动画绘制(核心)
    [CATransaction setDisableActions:YES];
    _movedMask.position = recognizer.view.center;
}
```
最终效果：

![](http://upload-images.jianshu.io/upload_images/1464492-2a2619df43ae539a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

源码已经放在[GitHub](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FFendouzhe%2FLRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，有帮助可以star，谢谢！
不熟悉CAEmitterLayer属性的可参见文章[《CAEmitterLayer属性详解》](https://www.jianshu.com/p/3222ce78c24a)

