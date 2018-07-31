效果：

![2017-12-15 14_32_30.gif](http://upload-images.jianshu.io/upload_images/1464492-f9b7eb3402bb5a9c.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码：

```
#import "ViewController.h"

@interface ViewController ()

@property (nonatomic, strong) NSTimer          *timer;
@property (nonatomic, strong) CAGradientLayer  *gradientLayer;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    UIImageView *imageView1 = [[UIImageView alloc] initWithFrame:self.view.bounds];
    imageView1.image        = [UIImage imageNamed:@"bg2"];
    [self.view addSubview:imageView1];
    
    UIImageView *imageView2 = [[UIImageView alloc] initWithFrame:self.view.bounds];
    imageView2.image        = [UIImage imageNamed:@"bg1"];
    [self.view addSubview:imageView2];
    
    // 渐变图层
    self.gradientLayer       = [CAGradientLayer layer];
    self.gradientLayer.frame = self.view.bounds;
    
    // 设置颜色
    self.gradientLayer.colors = @[(id)[[UIColor clearColor] colorWithAlphaComponent:1.0].CGColor,
                                  (id)[[UIColor redColor] colorWithAlphaComponent:0.f].CGColor];
    self.gradientLayer.locations = @[[NSNumber numberWithFloat:0.7f],
                                     [NSNumber numberWithFloat:1.0f]];
    
    // 添加渐变图层
    imageView2.layer.mask = self.gradientLayer;
    
    // 添加定时器
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.f
                                                  target:self
                                                selector:@selector(timerEvent)
                                                userInfo:nil
                                                 repeats:YES];
}

- (void)timerEvent {
    self.gradientLayer.locations = @[[NSNumber numberWithFloat:arc4random()%100/100.f],
                                     [NSNumber numberWithFloat:1.0f]];
    
    self.gradientLayer.colors = @[(id)[[UIColor clearColor] colorWithAlphaComponent:1.0].CGColor,
                                  (id)[[UIColor colorWithRed:arc4random()%255/255.f
                                                       green:arc4random()%255/255.f
                                                        blue:arc4random()%255/255.f
                                                       alpha:1.0] colorWithAlphaComponent:0.f].CGColor];
}

@end
```
