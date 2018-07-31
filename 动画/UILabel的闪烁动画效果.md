效果如下：
![](http://upload-images.jianshu.io/upload_images/1464492-0a269cba6bfc32ce.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，喜欢可以star.

实现如下：
LRShineLabel.h
```
#import <UIKit/UIKit.h>

@interface LRShineLabel : UIView

@property (nonatomic, strong) NSString *text;       // 文本的文字
@property (nonatomic, strong) UIFont   *font;       // 文本的字体

@property (nonatomic, assign) CGFloat   startScale; // 最初处于alpha = 0状态时的scale值
@property (nonatomic, assign) CGFloat   endScale;   // 最后处于alpha = 0状态时的scale值

@property (nonatomic, strong) UIColor  *backedLabelColor; // 不会消失的那个label的颜色
@property (nonatomic, strong) UIColor  *colorLabelColor;  // 最终会消失的那个label的颜色

- (void)startAnimation;

@end
```

LRShineLabel.m

```
#import "LRShineLabel.h"

@interface LRShineLabel()

@property (nonatomic, strong) UILabel  *backedLabel;
@property (nonatomic, strong) UILabel  *colorLabel;

@end

@implementation LRShineLabel

- (instancetype)initWithFrame:(CGRect)frame{
    if (self = [super initWithFrame:frame]) {
        _backedLabel = [[UILabel alloc] initWithFrame:self.bounds];
        _colorLabel  = [[UILabel alloc] initWithFrame:self.bounds];
        
        // 初始时的alpha值为0
        _backedLabel.alpha = 0;
        _colorLabel.alpha  = 0;
        
        // 文本居中
        _backedLabel.textAlignment = NSTextAlignmentCenter;
        _colorLabel.textAlignment  = NSTextAlignmentCenter;
        
        [self addSubview:_backedLabel];
        [self addSubview:_colorLabel];
    }
    return self;
}

- (void)startAnimation{
    if (_endScale == 0) {
        _endScale = 2.f;
    }
    
    [UIView animateWithDuration:1 delay:0.5 usingSpringWithDamping:7 initialSpringVelocity:4 options:UIViewAnimationOptionCurveEaseInOut animations:^{
        //恢复正常尺寸
        _backedLabel.alpha = 1.f;
        _backedLabel.transform = CGAffineTransformMake(1, 0, 0, 1, 0, 0);
        
        _colorLabel.alpha = 1.F;
        _colorLabel.transform = CGAffineTransformMake(1, 0, 0, 1, 0, 0);
        
    } completion:^(BOOL finished) {
        
        [UIView animateWithDuration:2 delay:0.5 usingSpringWithDamping:7 initialSpringVelocity:4 options:UIViewAnimationOptionCurveEaseInOut animations:^{
            //放大消失
            _colorLabel.alpha = 0.f;
            _colorLabel.transform = CGAffineTransformMake(_endScale, 0, 0, _endScale, 0, 0);
        } completion:^(BOOL finished) {
            
        }];
        
    }];
}

#pragma mark - 重写setter方法
@synthesize text = _text;
- (void)setText:(NSString *)text
{
    _text             = text;
    _backedLabel.text = text;
    _colorLabel.text  = text;
}
- (NSString *)text
{
    return _text;
}

@synthesize startScale = _startScale;
- (void)setStartScale:(CGFloat)startScale
{
    _startScale = startScale;
    _backedLabel.transform = CGAffineTransformMake(startScale, 0, 0, startScale, 0, 0);
    _colorLabel.transform  = CGAffineTransformMake(startScale, 0, 0, startScale, 0, 0);
}

- (CGFloat)startScale
{
    return _startScale;
}

@synthesize font = _font;
- (void)setFont:(UIFont *)font
{
    _font = font;
    _backedLabel.font = font;
    _colorLabel.font  = font;
}
- (UIFont *)font
{
    return _font;
}

@synthesize backedLabelColor = _backedLabelColor;
- (void)setBackedLabelColor:(UIColor *)backedLabelColor
{
    _backedLabelColor = backedLabelColor;
    _backedLabel.textColor = backedLabelColor;
}

@synthesize colorLabelColor = _colorLabelColor;
- (void)setColorLabelColor:(UIColor *)colorLabelColor
{
    _colorLabelColor = colorLabelColor;
    _colorLabel.textColor = colorLabelColor;
}

@end
```

调用

```
#import "LRShineLabelController.h"
#import "LRShineLabel.h"

@interface LRShineLabelController ()

@end

@implementation LRShineLabelController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor blackColor];
    
    LRShineLabel *shine = [[LRShineLabel alloc] initWithFrame:CGRectMake(0, 0, 320, 100)];
    shine.center = self.view.center;
    shine.text = @"LeiLuRong";
    //初始大小系数
    shine.startScale = 0.3f;
    shine.endScale = 2.f;
    shine.backedLabelColor = [UIColor redColor];
    shine.colorLabelColor = [UIColor cyanColor];
    shine.font = [UIFont systemFontOfSize:30.f];
    [self.view addSubview:shine];
    
    [[GCDQueue mainQueue] execute:^{
        [shine startAnimation];
    }];
}

@end
```
