效果：

![1](http://upload-images.jianshu.io/upload_images/1464492-29b0bb480fc8b93b.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](http://upload-images.jianshu.io/upload_images/1464492-29d472ff413a2d4c.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


代码已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，喜欢可以star.

实现：

LRMulticolorView.h
```
#import <UIKit/UIKit.h>

@interface LRMulticolorView : UIView

@property (nonatomic, assign) CGFloat          lineWidth;  // 圆的线宽
@property (nonatomic, assign) CFTimeInterval   sec;        // 秒
@property (nonatomic, assign) CGFloat          percent;    // 百分比

@property (nonatomic, strong) NSArray         *colors;     // 颜色组(CGColor)

@property (nonatomic, strong) NSArray         *lineDashPattern;//断点组

- (void)startAnimation;
- (void)endAnimation;
```

LRMulticolorView.m
```
#import "LRMulticolorView.h"

@interface LRMulticolorView ()

@property (nonatomic, strong) CAShapeLayer *circleLayer;

@end

@implementation LRMulticolorView

#pragma mark - 将当前view的layer替换成渐变色layer
+ (Class)layerClass
{
    return [CAGradientLayer class];
}

#pragma mark - 初始化
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self)
    {
        _circleLayer = [CAShapeLayer layer];
    }
    return self;
}

#pragma mark - 配置颜色
- (void)setupMulticolor
{
    // 获取当前的layer
    CAGradientLayer *gradientLayer = (CAGradientLayer *)[self layer];
    
    // 创建颜色数组
    NSMutableArray *colors = [NSMutableArray array];
    
    // 如果自定义颜色为空
    if (_colors == nil)
    {
        for (NSInteger hue = 0; hue <= 360; hue += 10)
        {
            [colors addObject:(id)[UIColor colorWithHue:1.0*hue/360.0
                                             saturation:1.0
                                             brightness:1.0
                                                  alpha:1.0].CGColor];
        }
        
        // 给渐变色layer设置颜色
        [gradientLayer setColors:[NSArray arrayWithArray:colors]];
    }
    else
    {
        // 给渐变色layer设置颜色
        [gradientLayer setColors:_colors];
    }
}

#pragma mark - 配置圆形
- (CAShapeLayer *)produceCircleShapeLayer
{
    // 生产出一个圆的路径
    CGPoint circleCenter = CGPointMake(CGRectGetMidX(self.bounds),
                                       CGRectGetMidY(self.bounds));
    
    CGFloat circleRadius = 0;
    
    if (_lineWidth == 0)
    {
        circleRadius = self.bounds.size.width/2.0 - 2;
    }
    else
    {
        circleRadius = self.bounds.size.width/2.0 - 2*_lineWidth;
    }
    
    UIBezierPath *circlePath = [UIBezierPath bezierPathWithArcCenter:circleCenter
                                                              radius:circleRadius
                                                          startAngle:M_PI
                                                            endAngle:-M_PI
                                                           clockwise:NO];
    
    // 生产出一个圆形路径的Layer
    _circleLayer.path          = circlePath.CGPath;
    _circleLayer.strokeColor   = [UIColor whiteColor].CGColor;
    _circleLayer.fillColor     = [[UIColor clearColor] CGColor];
    _circleLayer.lineDashPattern = self.lineDashPattern;

    if (_lineWidth == 0)
    {
        _circleLayer.lineWidth     = 1;
    }
    else
    {
        _circleLayer.lineWidth     = _lineWidth;
    }
    
    // 可以设置出圆的完整性
    _circleLayer.strokeStart = 0;
    _circleLayer.strokeEnd = 1.0;
    
    return _circleLayer;
}

#pragma mark - Animation

- (void)startAnimation
{
    // 设置渐变layer以及其颜色值
    [self setupMulticolor];
    
    // 生产一个圆形路径并设置成遮罩
    self.layer.mask = [self produceCircleShapeLayer];
    
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    
    if (_sec == 0)
    {
        animation.duration = 5;
    }
    else
    {
        animation.duration = _sec;
    }
    
    animation.repeatCount       = MAXFLOAT;
    animation.fromValue         = [NSNumber numberWithDouble:0];
    animation.toValue           = [NSNumber numberWithDouble:M_PI*2];
    [self.layer addAnimation:animation forKey:nil];
}

@synthesize percent = _percent;
-(CGFloat)percent
{
    return _percent;
}

- (void)setPercent:(CGFloat)percent
{
    if (_circleLayer)
    {
        _circleLayer.strokeEnd = percent;
    }
}

- (void)endAnimation
{
    [self.layer removeAllAnimations];
}

@end
```
控制器调用：
```
#import "LRMulticolorViewController.h"
#import "LRMulticolorView.h"

@interface LRMulticolorViewController ()

@property (nonatomic, strong) NSTimer         *timer;
@property (nonatomic, strong) LRMulticolorView  *showView;

@end

@implementation LRMulticolorViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor colorWithRed:0.878 green:0.878 blue:0.878 alpha:1];
    
    _showView           = [[LRMulticolorView alloc] initWithFrame:CGRectMake(0, 0, 200, 200)];
    _showView.lineWidth = 6.f;
    _showView.sec       = 2.f;
    _showView.colors    = @[(id)[UIColor cyanColor].CGColor,
                            (id)[UIColor yellowColor].CGColor,
                            (id)[UIColor cyanColor].CGColor];
    _showView.center    = self.view.center;
    //_showView.lineDashPattern = @[@10, @10];
    _timer              = [NSTimer scheduledTimerWithTimeInterval:1
                                                           target:self
                                                         selector:@selector(event:)
                                                         userInfo:nil
                                                          repeats:YES];
    
    [self.view addSubview:_showView];
    [_showView startAnimation];
}

- (void)event:(id)object
{
    _showView.percent = arc4random()%100/100.f;
}


@end
```
