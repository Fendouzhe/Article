效果：

![](http://upload-images.jianshu.io/upload_images/1464492-33fc6baa3aef39c9.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


代码：

LRColorUIImageView.h
```
#import <UIKit/UIKit.h>

typedef enum : NSUInteger {
    UP,    // 从上往下
    DOWN,  // 从下往上
    RIGHT, // 从右往左
    LEFT,  // 从左往右
} EColorDirection;


@interface LRColorUIImageView : UIImageView

/**
 *  确定方向（可以做动画）
 */
@property (nonatomic, assign) EColorDirection  direction;

/**
 *  颜色（可以做动画）
 */
@property (nonatomic, strong) UIColor  *color;

/**
 *  百分比（可以做动画）
 */
@property (nonatomic, assign) CGFloat   percent;

@end

```
LRColorUIImageView.m
```
#import "LRColorUIImageView.h"

@interface LRColorUIImageView ()

@property (nonatomic, strong) CAGradientLayer *gradientLayer;

@end

@implementation LRColorUIImageView

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        // 初始化CAGradientLayer
        self.gradientLayer           = [CAGradientLayer layer];
        self.gradientLayer.frame     = self.bounds;
        
        self.gradientLayer.colors    = @[(__bridge id)[UIColor clearColor].CGColor,
                                         (__bridge id)[UIColor redColor].CGColor];
        self.gradientLayer.locations = @[@(1), @(1)];
        
        [self.layer addSublayer:self.gradientLayer];
    }
    return self;
}

#pragma mark - 重写setter，getter方法
@synthesize color = _color;
- (void)setColor:(UIColor *)color {
    _color = color;
    self.gradientLayer.colors = @[(__bridge id)[UIColor clearColor].CGColor,
                                  (__bridge id)color.CGColor];
}
- (UIColor *)color {
    return _color;
}

@synthesize percent = _percent;
- (void)setPercent:(CGFloat)percent {
    _percent = percent;
    self.gradientLayer.locations = @[@(percent), @(1)];
}
- (CGFloat)percent {
    return _percent;
}

@synthesize direction = _direction;
- (void)setDirection:(EColorDirection)direction {
    _direction = direction;
    if (direction == UP) {
        self.gradientLayer.startPoint = CGPointMake(0, 0);
        self.gradientLayer.endPoint   = CGPointMake(0, 1);
    } else if (direction == DOWN) {
        self.gradientLayer.startPoint = CGPointMake(0, 1);
        self.gradientLayer.endPoint   = CGPointMake(0, 0);
    } else if (direction == RIGHT) {
        self.gradientLayer.startPoint = CGPointMake(1, 0);
        self.gradientLayer.endPoint   = CGPointMake(0, 0);
    } else if (direction == LEFT) {
        self.gradientLayer.startPoint = CGPointMake(0, 0);
        self.gradientLayer.endPoint   = CGPointMake(1, 0);
    } else {
        self.gradientLayer.startPoint = CGPointMake(0, 0);
        self.gradientLayer.endPoint   = CGPointMake(0, 1);
    }
}
- (EColorDirection)direction {
    return _direction;
}
```

控制器调用：
```
#import "ViewController.h"
#import "LRColorUIImageView.h"

@interface ViewController ()

@property (nonatomic, strong) LRColorUIImageView *colorView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.colorView        = [[LRColorUIImageView alloc] initWithFrame:self.view.bounds];
    self.colorView.center = self.view.center;
    self.colorView.image  = [UIImage imageNamed:@"bg"];
    [self.view addSubview:self.colorView];
    
    [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(event) userInfo:nil repeats:YES];
}

- (void)event {
    self.colorView.direction = UP;
    self.colorView.color     = [UIColor cyanColor];
    self.colorView.percent   = arc4random()%100/100.f;//0.5;
}
```
