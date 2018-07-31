效果(gif掉真实际效果可以运行demo)：

![](http://upload-images.jianshu.io/upload_images/1464492-3084654c0bf21717.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

demo已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，有帮助可以star，谢谢！
时间问题原理在这里不再多叙述，具体实际实现见代码：

LRProgressColor.h
```
#import <Foundation/Foundation.h>

@interface LRProgressColor : NSObject

/**
 存储的是CGColor的数组元素
 */
@property (nonatomic, strong)NSArray *cgColors;

/**
 颜色起始点
 */
@property (nonatomic, assign)CGPoint startPoint;

/**
 颜色结束点
 */
@property (nonatomic, assign)CGPoint endPoint;

/**
 颜色移位一次的动画时间
 */

@property (nonatomic, assign)NSTimeInterval duration;

/**
 *  -----------[ 子类可以重写该方法 ]-----------
 *
 *  转换颜色的算法
 *
 *  @return 移位后的颜色数组
 */
- (NSArray *)accessColors;

#pragma mark - 便利构造器方法(自己添加方法) --

+ (instancetype)redGradientColor;

@end
```
LRProgressColor.m
```
#import "LRProgressColor.h"

@implementation LRProgressColor

- (instancetype)init{
    if (self = [super init]) {
        
        self.startPoint = CGPointMake(0.f, 0.5f);
        self.endPoint   = CGPointMake(1.f, 0.5f);
        self.duration   = 0.1f;
        
    }
    return self;
}

- (NSArray *)accessColors{
    
    NSMutableArray *mutableArr = [_cgColors mutableCopy];
    //第一个和最后一个颜色交换实现位移
    id last = mutableArr.lastObject;
    [mutableArr removeLastObject];
    [mutableArr insertObject:last atIndex:0];
    
    NSArray *colors = [NSArray arrayWithArray:mutableArr];
    return colors;
}

+ (instancetype)redGradientColor{
    
    LRProgressColor *color = [[self alloc] init];
    
    NSMutableArray *cgColors = [NSMutableArray array];
    [cgColors addObject:(id)[UIColor colorWithRed:0.2f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.2f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.3f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.4f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.5f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.6f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.7f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.8f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.9f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:1.0f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.9f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.8f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.7f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.6f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.5f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.4f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.3f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.2f green:0.f blue:0.f alpha:1.f].CGColor];
    [cgColors addObject:(id)[UIColor colorWithRed:0.2f green:0.f blue:0.f alpha:1.f].CGColor];
    
    color.cgColors = cgColors;
    color.duration = 0.1f;
    return color;
}


@end
```

LRColorFullProgressView.h
```
#import <UIKit/UIKit.h>
#import "LRProgressColor.h"

@interface LRColorFullProgressView : UIView


/**
 进度
 */
@property (nonatomic, assign)CGFloat progress;


/**
 进度颜色
 */
@property (nonatomic, strong)LRProgressColor *progressColor;



/**
 配置生效以及开始运行
 */
- (void)configAvailableAndBegin;


/**
 便利构造器方法

 @param  frame         尺寸
 @param  progressColor 颜色值,可以为空
 @return 实例对象
 */
+ (instancetype)colorfulProgressViewWithFrame:(CGRect)frame progressColor:(LRProgressColor *)progressColor;
@end
```

LRColorFullProgressView.m
```
#import "LRColorFullProgressView.h"

@interface LRColorFullProgressView()<CAAnimationDelegate>{
    
    /**
     *  当前view宽度
     */
    CGFloat _width;
    
    /**
     *  当前view高度 用于记录原始高度
     */
    CGFloat _height;
}

@property (nonatomic, strong)UIView *baseView;

@property (nonatomic, strong)CAGradientLayer *gradientLayer;

@end

@implementation LRColorFullProgressView

- (instancetype)initWithFrame:(CGRect)frame{
    if (self = [super initWithFrame:frame]) {
        
        _width = self.frame.size.width;
        _height = self.frame.size.height;
        
        // baseView
        self.baseView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, _height)];
        _baseView.layer.masksToBounds = YES;
        [self addSubview:_baseView];
        
        // gradientLayer
        self.gradientLayer = [CAGradientLayer layer];
        _gradientLayer.frame = self.bounds;
        [self.baseView.layer addSublayer:_gradientLayer];
        
        
    }
    return self;
}

- (void)configAvailableAndBegin{
    
    // 如果没有设置ProgressColor,则自己生成一个
    if (_progressColor == nil) {
        
        _progressColor = [[LRProgressColor alloc] init];
        
        NSMutableArray *cgColors = [NSMutableArray array];
        for (NSInteger i = 0; i < 360; i+=5) {
            //指定HSB，参数是：色调（hue），饱和的（saturation），亮度（brightness）
            UIColor *color = [UIColor colorWithHue:i/360.0 saturation:1.0 brightness:1.0 alpha:1.0];
            [cgColors addObject:(__bridge id)color.CGColor];
        }
        _progressColor.cgColors = cgColors;
    }
    
    _gradientLayer.colors = _progressColor.cgColors;
    _gradientLayer.startPoint = _progressColor.startPoint;
    _gradientLayer.endPoint = _progressColor.endPoint;
    
    [self animation];
}

- (void)animation{
    
    NSArray *fromColors = _progressColor.cgColors;
    //获取最后一帧
    NSArray *toClors = [_progressColor accessColors];
    _progressColor.cgColors = toClors;
    
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"colors"];
    animation.fromValue = fromColors;
    animation.toValue = toClors;
    animation.duration = _progressColor.duration;
    animation.fillMode = kCAFillModeForwards;
    animation.removedOnCompletion = YES;
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
    animation.delegate = self;
    //layerhuadh
    //防止复原 removedOnCompletion == YES 则需要设置动画完后最终颜色
    _gradientLayer.colors = toClors;
    [_gradientLayer addAnimation:animation forKey:@"colorsAnimation"];

}

#pragma mark-- CAAnimationDelegate --

-(void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
    [self animation];
}

+ (instancetype)colorfulProgressViewWithFrame:(CGRect)frame progressColor:(LRProgressColor *)progressColor{
    
    LRColorFullProgressView *progressView = [[self alloc] initWithFrame:frame];
    if (progressColor) {
        progressView.progressColor = progressColor;
    }
    [progressView configAvailableAndBegin];
    
    return progressView;
}

#pragma mark -- 重写getter,setter方法 --

@synthesize progress = _progress;
- (void)setProgress:(CGFloat)progress{
    _progress = progress;
    if (progress <= 0) {
        _baseView.frame = CGRectMake(0, 0, 0, _height);
    }else if (progress <= 1){
        _baseView.frame = CGRectMake(0, 0, _width*progress, _height);
    }else{
        _baseView.frame = CGRectMake(0, 0, _width, _height);
    }
}

- (CGFloat)progress{
    return _progress;
}
@end
```
控制器调用：
```
#import "LRColorProgressController.h"
#import "LRColorFullProgressView.h"
#import "LRProgressColor.h"

@interface LRColorProgressController ()

@property (nonatomic, strong)GCDTimer *timer;

@property (nonatomic, strong)LRColorFullProgressView *progressView0;

@property (nonatomic, strong)LRColorFullProgressView *progressView1;

@end

@implementation LRColorProgressController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor blackColor];
    
    /*
    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.frame = CGRectMake(0, 80, 100, 200);
    [self.view.layer addSublayer:gradientLayer];
    gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor,
                             (__bridge id)[UIColor greenColor].CGColor,
                             (__bridge id)[UIColor blueColor].CGColor];
    gradientLayer.locations = @[@0.25,@0.5,@0.75];
    gradientLayer.startPoint = CGPointMake(0, 0.5);
    gradientLayer.endPoint = CGPointMake(1, 0.5);
     */
    
    // progressView0
    self.progressView0 = [LRColorFullProgressView colorfulProgressViewWithFrame:CGRectMake(0, 160, self.view.width, 2.f)
                                                                  progressColor:nil];
    
    [self.view addSubview:self.progressView0];
    
    // progressView1
    self.progressView1 = [LRColorFullProgressView colorfulProgressViewWithFrame:CGRectMake(0, 180, self.view.width, 2.f)
                                                                  progressColor:[LRProgressColor redGradientColor]];
    
    [self.view addSubview:self.progressView1];
    
    // timer
    _timer = [[GCDTimer alloc] initInQueue:[GCDQueue mainQueue]];
    [_timer event:^{
        
        CGFloat percent0 = arc4random() % 101 / 100.f;
        CGFloat percent1 = arc4random() % 101 / 100.f;
        [UIView animateWithDuration:0.5 delay:0 usingSpringWithDamping:1.f initialSpringVelocity:0 options:0 animations:^{
            _progressView0.progress = percent0;
            _progressView1.progress = percent1;
        } completion:^(BOOL finished) {
            
        }];
    } timeInterval:NSEC_PER_SEC];
    [_timer start];
}

@end

```

