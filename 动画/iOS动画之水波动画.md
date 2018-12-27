- 前言:现在很多app为了提供好的交互效果给用户，通常都会通过添加动画效果来达到目的。一个好的动画效果往往会起到神来之笔的作用。而在IOS中动画效果也是丰富多彩的,我准备在后续的笔记中详细的记录那些动画。
- 今天我们就来实现一下在IOS中的水波动画。先上效果图: 

![](https://upload-images.jianshu.io/upload_images/9610202-de295289eeaf066c.gif?imageMogr2/auto-orient/strip)

- 最开始看到这个效果确实有点难的，不过如果对正弦公式比较了解的话实现起来也挺简单的。

  -  高中知识:y = Asin(wx + θ) + k;
  - 其中A：代表振幅; w:角频率，和周期的关系是:T = 2π/|w| ;θ:初相,相对于标准的正弦公式y = sin(x)而言,θ代表 
  - 标准的正弦公式y = sin(x)在水平(x轴)方向上的整体移动，即左加右减;k:偏距,代表标准的正弦公式y = sin(x)在 垂直(y轴)方向上的整体移动,即上加下减。
  - 由表达式 y = sin(x)我们知道:在[0,2π]的区间上绘制一个完整的正弦波形,周期刚好是2π(T= 2π/|w|) 这里w = 1;绘制2个完整的正弦波形,周期为π。所以可以把w理解成完整波形的个数。由此可以得出:在区间[0,waveWidth]上绘制n个完整的波形,w = 2π * n / waveWidth。另外还要注意一点就是UIKit框架坐标轴是向下的。
  - 上面的知识也只是对正弦公式进行回顾。怎么样才能得到波形效果呢？关键点就在初相θ身上。
  - 下面这张图是正弦公式:y = sin(x)的图谱: 
 ![](https://upload-images.jianshu.io/upload_images/9610202-6c585cb68c96a2e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    我们把初相θ向左移动1个单位长度得到公式:y = sin(x + 1)的图谱: 
 ![](https://upload-images.jianshu.io/upload_images/9610202-f81ec89e4ad9da5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    比较这两张图谱，我们可以从中发现：要想实现波形震荡效果，我们只需不断变化初相θ的值，然后不断刷新屏幕即可。
  - CADisplayLink：这个定时器的刷新频率是60HZ，即每秒可以对屏幕进行60次刷新，我们肉眼是感觉不出刷新间隔的时差的。 
整个实现过程的代码如下: 
```
@interface JGWaterWaveAnimation()
{
    //振幅--这个决定波形的起伏高度
    CGFloat _waterAmplitude;
    //频率--这个决定波形的宽度
    CGFloat _waterFrequency;
    //初相:这个决定了波形水平移动的速度
    CGFloat _waterEpoch;
    //偏距--调节距离顶部的高度
    CGFloat _waterSetover;
    //定时器
    CADisplayLink *_timer;
    
    //波形整个的宽度
    CGFloat _waterWaveWidth;
    //波形的整个高度
    CGFloat _waterWaveHeight;
}
/**layer*/
@property(strong,nonatomic)CAShapeLayer *waterShapeLayer;
@end
@implementation JGWaterWaveAnimation
- (instancetype)initWithFrame:(CGRect)frame{
    if (self = [super initWithFrame:frame]) {
        
        //default
        _waterAmplitude = 15.0;
        //假设在frame的长度上出现3个完整的波形:注意这里乘以0.5出现震荡效果,如果不乘以0.5只会出现波形平移的效果。
        _waterFrequency = 2 *M_PI * 3 / frame.size.width *0.5;
        _waterEpoch = 0.0;
        _waterSetover = 20.0;
        
        _waterWaveWidth = CGRectGetWidth(self.frame);
        _waterWaveHeight = CGRectGetHeight(self.frame);
        
        [self.layer addSublayer:self.waterShapeLayer];
        //初始化定时器
        _timer = [CADisplayLink displayLinkWithTarget:[YYWeakProxy proxyWithTarget:self] selector:@selector(waterWaveAnimation)];
        [_timer addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
    }
    return self;
}
- (void)waterWaveAnimation{
    
    //核心代码:
    _waterEpoch += 0.08;
    //path
    UIBezierPath *waterWavePath = [UIBezierPath bezierPath];
    [waterWavePath moveToPoint:CGPointMake(0, 0)];
    for (CGFloat x = 0; x < _waterWaveWidth; x ++) {
        CGFloat y = _waterAmplitude * sinf(_waterFrequency * x + _waterEpoch) + _waterSetover;
        [waterWavePath addLineToPoint:CGPointMake(x, y)];
    }
    [waterWavePath addLineToPoint:CGPointMake(_waterWaveWidth, _waterWaveHeight)];
    [waterWavePath addLineToPoint:CGPointMake(0, _waterWaveHeight)];
    [waterWavePath closePath];
    
    self.waterShapeLayer.path = waterWavePath.CGPath;
}
- (CAShapeLayer *)waterShapeLayer{
    if (!_waterShapeLayer) {
        _waterShapeLayer = [CAShapeLayer layer];
        _waterShapeLayer.frame = self.bounds;
        _waterShapeLayer.fillColor = [UIColor colorWithRed:52/255.0 green:152/255.0 blue:219/255.0 alpha:1.0].CGColor;
        _waterShapeLayer.strokeColor = [UIColor clearColor].CGColor;
    }
    return _waterShapeLayer;
}
- (void)dealloc{
    
    [_timer invalidate];
    _timer = nil;
}
@end
```
--------------------- 
