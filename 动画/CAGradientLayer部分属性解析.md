iOS中Layer的坐标系统:
![](http://upload-images.jianshu.io/upload_images/1464492-3341b9491b8c6f2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
效果:
![](http://upload-images.jianshu.io/upload_images/1464492-d4bf097018833671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
- (void)viewDidLoad
{
    [super viewDidLoad];

    CAGradientLayer *colorLayer = [CAGradientLayer layer];
    colorLayer.frame    = (CGRect){CGPointZero, CGSizeMake(200, 200)};
    colorLayer.position = self.view.center;
    [self.view.layer addSublayer:colorLayer];

    // 颜色分配
    colorLayer.colors = @[(__bridge id)[UIColor redColor].CGColor,
                          (__bridge id)[UIColor greenColor].CGColor,
                          (__bridge id)[UIColor blueColor].CGColor];
    
    // 颜色分割线
    colorLayer.locations  = @[@(0.25), @(0.5), @(0.75)];
    
    // 起始点
    colorLayer.startPoint = CGPointMake(0, 0);
    
    // 结束点
    colorLayer.endPoint   = CGPointMake(1, 0);
}
```

![](http://upload-images.jianshu.io/upload_images/1464492-bd87ff68abe673ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1198)

颜色分配严格遵守Layer的坐标系统,locations,startPoint,endPoint都是以Layer坐标系统进行计算的.
而locations并不是表示颜色值所在位置,它表示的是颜色在Layer坐标系相对位置处要开始进行渐变颜色了.
![](http://upload-images.jianshu.io/upload_images/1464492-310c8e81db247b34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

CAGradientLayer 的这四个属性 colors locations startPoint endPoint 都是可以进行动画的哦.

稍微复杂点的动画效果:
![](http://upload-images.jianshu.io/upload_images/1464492-493c1e18b33438eb.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#import "RootViewController.h"
#import "LRGCD.h"

@interface RootViewController ()

@property (nonatomic, strong) GCDTimer  *timer;

@end

@implementation RootViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    CAGradientLayer *colorLayer = [CAGradientLayer layer];
    colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    colorLayer.frame    = (CGRect){CGPointZero, CGSizeMake(200, 200)};
    colorLayer.position = self.view.center;
    [self.view.layer addSublayer:colorLayer];

    // 颜色分配
    colorLayer.colors = @[(__bridge id)[UIColor cyanColor].CGColor,
                          (__bridge id)[UIColor orangeColor].CGColor,
                          (__bridge id)[UIColor magentaColor].CGColor];
    
    // 起始点
    colorLayer.startPoint = CGPointMake(0, 0);
    
    // 结束点
    colorLayer.endPoint   = CGPointMake(1, 0);
    
    _timer = [[GCDTimer alloc] initInQueue:[GCDQueue mainQueue]];
    [_timer event:^{
        
        static CGFloat test = - 0.1f;
        
        if (test >= 1.1)
        {
            test = - 0.1f;
            [CATransaction setDisableActions:YES];
            colorLayer.locations  = @[@(test), @(test + 0.05), @(test + 0.1)];
        }
        else
        {
            [CATransaction setDisableActions:NO];
            colorLayer.locations  = @[@(test), @(test + 0.05), @(test + 0.1)];
        }
        
        test += 0.1f;
        
    } timeInterval:NSEC_PER_SEC];
    [_timer start];
}

@end
```

![](http://upload-images.jianshu.io/upload_images/1464492-dd29ca8270a8bf7f.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
    _timer = [[GCDTimer alloc] initInQueue:[GCDQueue mainQueue]];
    [_timer event:^{
        
        static CGFloat test = - 0.1f;
        
        if (test >= 1.1)
        {
            test = - 0.1f;
            [CATransaction setDisableActions:NO];
            colorLayer.locations  = @[@(test), @(test + 0.01), @(test + 0.011)];
        }
        else
        {
            [CATransaction setDisableActions:NO];
            colorLayer.locations  = @[@(test), @(test + 0.01), @(test + 0.011)];
        }
        
        test += 0.1f;
        
    } timeInterval:NSEC_PER_SEC];
    [_timer start];
```
配合CAShapeLayer使用

![](http://upload-images.jianshu.io/upload_images/1464492-a840e0991b09d731.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#import "LRGlowViewController.h"
#import "GCDTimer.h"

#define DEGRESS(degress) ((M_PI*(degress))/180.f)
@interface LRGlowViewController ()

@property (nonatomic, strong)GCDTimer *timer;

@end

@implementation LRGlowViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor blackColor];
    
    CAGradientLayer *colorLayer = [CAGradientLayer layer];
    colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    colorLayer.frame = (CGRect){CGPointZero,CGSizeMake(200, 200)};
    colorLayer.position = self.view.center;
    [self.view.layer addSublayer:colorLayer];
    //颜色分配
    colorLayer.colors = @[(__bridge id)[UIColor redColor].CGColor,
                          (__bridge id)[UIColor whiteColor].CGColor,
                          (__bridge id)[UIColor redColor].CGColor];
    colorLayer.locations = @[@(-2),@(-1),@(0)];
    //开始点
    colorLayer.startPoint = CGPointMake(0, 0);
    //结束点
    colorLayer.endPoint = CGPointMake(1, 0);
    
    //绘制一个圆
    CAShapeLayer *circle = [CAShapeLayer layer];
    UIBezierPath *bezierpath = [UIBezierPath bezierPathWithArcCenter:CGPointZero radius:80 startAngle:DEGRESS(0) endAngle:DEGRESS(360) clockwise:YES];
    circle.path = bezierpath.CGPath;
    circle.position = CGPointMake(100, 100);
    // 设置填充颜色为透明
    circle.fillColor = [UIColor clearColor].CGColor;
    circle.strokeColor = [UIColor redColor].CGColor;
    circle.lineWidth = 2;
    //circle.lineDashPattern = @[@1,@5,@1,@5];
    circle.strokeEnd = 1.0f;
    colorLayer.mask = circle;
    
    _timer = [[GCDTimer alloc] initInQueue:[GCDQueue mainQueue]];
    [_timer event:^{
        
        CABasicAnimation *faceAnim = [CABasicAnimation animationWithKeyPath:@"locations"];
        faceAnim.fromValue = @[@(-0.2),@(-0.1),@(0)];
        faceAnim.toValue = @[@(1),@(1.1),@(1.2)];
        faceAnim.duration = 1.5;
        [colorLayer addAnimation:faceAnim forKey:nil];
        
    } timeInterval:2*NSEC_PER_SEC];
    
    [_timer start];
}

@end
```
代码已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，喜欢可以star.
