效果：
![](http://upload-images.jianshu.io/upload_images/1464492-a3b9d0c5288abd6c.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现代码：
```
#import "ViewController.h"

@interface ViewController ()

@property (nonatomic, strong) NSTimer      *timer;
@property (nonatomic, strong) CAShapeLayer *shapeLayer;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 创建shapeLayer
    _shapeLayer = [CAShapeLayer layer];
    _shapeLayer.frame         = (CGRect){CGPointMake(0, 0), CGSizeMake(200, 200)};
    _shapeLayer.position      = self.view.center;
    _shapeLayer.path          = [self getStar1BezierPath].CGPath;
    _shapeLayer.fillColor     = [UIColor clearColor].CGColor;
    _shapeLayer.strokeColor   = [UIColor redColor].CGColor;
    _shapeLayer.lineWidth     = 2.f;
    [self.view.layer addSublayer:_shapeLayer];
    
    // 创建定时器
    _timer = [NSTimer scheduledTimerWithTimeInterval:1.f
                                              target:self
                                            selector:@selector(pathAnimation)
                                            userInfo:nil
                                             repeats:YES];
}

/**
 *  执行path的动画
 */
- (void)pathAnimation {
    static int i = 0;
    if (i++ % 2 == 0) {
        CABasicAnimation *circleAnim = [CABasicAnimation animationWithKeyPath:@"path"];
        circleAnim.removedOnCompletion = NO;
        circleAnim.duration            = 1;
        circleAnim.fromValue           = (__bridge id)[self getStar1BezierPath].CGPath;
        circleAnim.toValue             = (__bridge id)[self getStar2BezierPath].CGPath;
        _shapeLayer.path               = [self getStar2BezierPath].CGPath;
        [_shapeLayer addAnimation:circleAnim forKey:@"animateCirclePath"];
    } else {
        CABasicAnimation *circleAnim = [CABasicAnimation animationWithKeyPath:@"path"];
        circleAnim.removedOnCompletion = NO;
        circleAnim.duration            = 1;
        circleAnim.fromValue           = (__bridge id)[self getStar2BezierPath].CGPath;
        circleAnim.toValue             = (__bridge id)[self getStar1BezierPath].CGPath;
        _shapeLayer.path               = [self getStar1BezierPath].CGPath;
        [_shapeLayer addAnimation:circleAnim forKey:@"animateCirclePath"];
    }
}

/**
 *  贝塞尔曲线1
 *
 *  @return 贝塞尔曲线
 */
-(UIBezierPath *)getStar1BezierPath {
    //// Star Drawing
    UIBezierPath* starPath = [UIBezierPath bezierPath];
    [starPath moveToPoint: CGPointMake(22.5, 2.5)];
    [starPath addLineToPoint: CGPointMake(28.32, 14.49)];
    [starPath addLineToPoint: CGPointMake(41.52, 16.32)];
    [starPath addLineToPoint: CGPointMake(31.92, 25.56)];
    [starPath addLineToPoint: CGPointMake(34.26, 38.68)];
    [starPath addLineToPoint: CGPointMake(22.5, 32.4)];
    [starPath addLineToPoint: CGPointMake(10.74, 38.68)];
    [starPath addLineToPoint: CGPointMake(13.08, 25.56)];
    [starPath addLineToPoint: CGPointMake(3.48, 16.32)];
    [starPath addLineToPoint: CGPointMake(16.68, 14.49)];
    [starPath closePath];
    
    return starPath;
}

/**
 *  贝塞尔曲线2
 *
 *  @return 贝塞尔曲线
 */
-(UIBezierPath *)getStar2BezierPath {
    //// Star Drawing
    UIBezierPath* starPath = [UIBezierPath bezierPath];
    [starPath moveToPoint: CGPointMake(22.5, 2.5)];
    [starPath addLineToPoint: CGPointMake(32.15, 9.21)];
    [starPath addLineToPoint: CGPointMake(41.52, 16.32)];
    [starPath addLineToPoint: CGPointMake(38.12, 27.57)];
    [starPath addLineToPoint: CGPointMake(34.26, 38.68)];
    [starPath addLineToPoint: CGPointMake(22.5, 38.92)];
    [starPath addLineToPoint: CGPointMake(10.74, 38.68)];
    [starPath addLineToPoint: CGPointMake(6.88, 27.57)];
    [starPath addLineToPoint: CGPointMake(3.48, 16.32)];
    [starPath addLineToPoint: CGPointMake(12.85, 9.21)];
    [starPath closePath];
    
    return starPath;
}

@end
```
