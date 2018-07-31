效果：

![](http://upload-images.jianshu.io/upload_images/1464492-c9c3698f918fb00c.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码：

```
#import <QuartzCore/QuartzCore.h>
#import <UIKit/UIKit.h>

// 将常数转换为度数
#define   DEGREES(degrees)  ((M_PI * (degrees))/ 180.f)

@interface CAShapeLayer (Circle)

// 圆环
+ (instancetype)LayerWithCircleCenter:(CGPoint)point
                               radius:(CGFloat)radius
                           startAngle:(CGFloat)startAngle
                             endAngle:(CGFloat)endAngle
                            clockwise:(BOOL)clockwise
                      lineDashPattern:(NSArray *)lineDashPattern;

// 圆
+ (instancetype)LayerWithCircleCenter:(CGPoint)point
                               radius:(CGFloat)radius
                           startAngle:(CGFloat)startAngle
                             endAngle:(CGFloat)endAngle
                            clockwise:(BOOL)clockwise;
```

```
#import "CAShapeLayer+Circle.h"

@implementation CAShapeLayer (Circle)

+ (instancetype)LayerWithCircleCenter:(CGPoint)point
                               radius:(CGFloat)radius
                           startAngle:(CGFloat)startAngle
                             endAngle:(CGFloat)endAngle
                            clockwise:(BOOL)clockwise
                      lineDashPattern:(NSArray *)lineDashPattern
{
    CAShapeLayer *layer = [CAShapeLayer layer];
    
    // 贝塞尔曲线(创建一个圆)
    UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(0, 0)
                                                        radius:radius
                                                    startAngle:startAngle
                                                      endAngle:endAngle
                                                     clockwise:clockwise];
    
    // 获取path
    layer.path = path.CGPath;
    layer.position = point;
    
    // 设置填充颜色为透明
    layer.fillColor = [UIColor clearColor].CGColor;
    
    // 获取曲线分段的方式
    if (lineDashPattern)
    {
        layer.lineDashPattern = lineDashPattern;
    }
    
    return layer;
}

+ (instancetype)LayerWithCircleCenter:(CGPoint)point
                               radius:(CGFloat)radius
                           startAngle:(CGFloat)startAngle
                             endAngle:(CGFloat)endAngle
                            clockwise:(BOOL)clockwise
{
    CAShapeLayer *layer = [CAShapeLayer layer];
    
    // 贝塞尔曲线(创建一个圆)
    UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(0, 0)
                                                        radius:radius/2.f
                                                    startAngle:startAngle
                                                      endAngle:endAngle
                                                     clockwise:clockwise];
    
    // 获取path
    layer.path      = path.CGPath;
    layer.position  = point;
    layer.lineCap   = kCALineCapButt;
    layer.lineWidth = radius;
    
    // 设置填充颜色为透明
    layer.fillColor = [UIColor clearColor].CGColor;
    
    return layer;
}
```

```
#import "ViewController.h"
#import "CAShapeLayer+Circle.h"

@interface ViewController ()

@property (nonatomic, strong) NSTimer         *timer;
@property (nonatomic, strong) CAGradientLayer *faucet;
@property (nonatomic, strong) CAShapeLayer    *circleLayer;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 设置背景色
    self.view.backgroundColor = [UIColor colorWithRed:0.878 green:0.878 blue:0.878 alpha:1];
    
    // 创建形状遮罩
    self.circleLayer = [CAShapeLayer LayerWithCircleCenter:CGPointMake(82, 82)
                                                    radius:80
                                                startAngle:DEGREES(0)
                                                  endAngle:DEGREES(360)
                                                 clockwise:YES
                                           lineDashPattern:@[@10, @10]];
    self.circleLayer.strokeColor   = [UIColor blackColor].CGColor;  // 边缘线的颜色
    self.circleLayer.lineCap       = kCALineCapSquare;              // 边缘线的类型
    self.circleLayer.lineWidth     = 4.f;                           // 线条宽度
    self.circleLayer.strokeStart   = 0.0f;
    self.circleLayer.strokeEnd     = 1.0f;
    
    // 创建渐变图层
    self.faucet          = [CAGradientLayer layer];
    self.faucet.frame    = CGRectMake(0, 0, 200, 200);
    self.faucet.position = self.view.center;
    
    // 以CAShapeLayer的形状作为遮罩是实现特定颜色渐变的关键
    self.faucet.mask   = self.circleLayer;
    self.faucet.colors = @[(id)[UIColor greenColor].CGColor,
                           (id)[UIColor redColor].CGColor,
                           (id)[UIColor cyanColor].CGColor,
                           (id)[UIColor purpleColor].CGColor,
                           (id)[UIColor yellowColor].CGColor];
    
    // 设定动画时间
    self.faucet.speed = 0.5f;
    
    // 添加到系统图层中
    [self.view.layer addSublayer:self.faucet];
    
    // 创建定时器
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.f
                                                  target:self
                                                selector:@selector(timerEvent)
                                                userInfo:nil
                                                 repeats:YES];
}

/**
 *  定时器事件
 */
- (void)timerEvent {
    self.faucet.colors = @[(id)[UIColor colorWithRed:arc4random()%255/255.f
                                               green:arc4random()%255/255.f
                                                blue:arc4random()%255/255.f
                                               alpha:1].CGColor,
                           (id)[UIColor colorWithRed:arc4random()%255/255.f
                                               green:arc4random()%255/255.f
                                                blue:arc4random()%255/255.f
                                               alpha:1].CGColor,
                           (id)[UIColor colorWithRed:arc4random()%255/255.f
                                               green:arc4random()%255/255.f
                                                blue:arc4random()%255/255.f
                                               alpha:1].CGColor,
                           (id)[UIColor colorWithRed:arc4random()%255/255.f
                                               green:arc4random()%255/255.f
                                                blue:arc4random()%255/255.f
                                               alpha:1].CGColor,
                           (id)[UIColor colorWithRed:arc4random()%255/255.f
                                               green:arc4random()%255/255.f
                                                blue:arc4random()%255/255.f
                                               alpha:1].CGColor];
}

@end
```


