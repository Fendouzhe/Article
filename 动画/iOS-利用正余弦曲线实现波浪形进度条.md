一、显示效果

![](https://upload-images.jianshu.io/upload_images/9610202-8a819fcee26fc558.gif?imageMogr2/auto-orient/strip)


二、工作原理

实现原理主要是通过绘制出一条正弦曲线和一条余弦曲线，在两条曲线上下方添加不同的背景色，让曲线按照一个方向移动即可模拟出波浪的效果。首先需要了解曲线公式：

正弦曲线公式：y=Asin(ωx+φ)+k

A :振幅,曲线最高位和最低位的距离

ω :角速度,用于控制周期大小，单位x中起伏的个数

K :偏距,曲线上下偏移量

φ :初相,曲线左右偏移量

曲线图如下：

![](https://upload-images.jianshu.io/upload_images/9610202-d79883148bcb625a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果让曲线按照X轴以移动速度移动下去就会出现波浪的效果 ，按照以下步骤即可，和把大象放冰箱里的步骤差不多。

第一步:添加一条曲线----->第二步:让曲线沿x轴移动 ----->第三步:在曲线下部分添加填充色


![](https://upload-images.jianshu.io/upload_images/9610202-d7e922ce63523b32.gif?imageMogr2/auto-orient/strip)



到这里就完成了一层波浪，然后再添加第二层即可，如果第一层为余弦曲线，第二层则需添加正弦曲线，这样看起来就会有分层的感觉。

![](https://upload-images.jianshu.io/upload_images/9610202-f5393e1e15efefb5.gif?imageMogr2/auto-orient/strip)


然后稍加润色可以了，是不是很简单。

三、代码
```
#define BackGroundColor [UIColor colorWithRed:96/255.0f green:159/255.0f blue:150/255.0f alpha:1]
#define WaveColor1 [UIColor colorWithRed:136/255.0f green:199/255.0f blue:190/255.0f alpha:1]
#define WaveColor2 [UIColor colorWithRed:28/255.0 green:203/255.0 blue:174/255.0 alpha:1]
 
 
#import "XLWave.h"
 
 
@interface XLWave ()
{
    //前面的波浪
    CAShapeLayer *_waveLayer1;
    CAShapeLayer *_waveLayer2;
    
    CADisplayLink *_disPlayLink;
    
    /**
     曲线的振幅
     */
    CGFloat _waveAmplitude;
    /**
     曲线角速度
     */
    CGFloat _wavePalstance;
    /**
     曲线初相
     */
    CGFloat _waveX;
    /**
     曲线偏距
     */
    CGFloat _waveY;
    
    /**
     曲线移动速度
     */
    CGFloat _waveMoveSpeed;
}
@end
 
 
@implementation XLWave
 
-(instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        [self buildUI];
        [self buildData];
    }
    return self;
}
 
//初始化UI
-(void)buildUI
{
    //初始化波浪
    //底层
    _waveLayer1 = [CAShapeLayer layer];
    _waveLayer1.fillColor = WaveColor1.CGColor;
    _waveLayer1.strokeColor = WaveColor1.CGColor;
    
    [self.layer addSublayer:_waveLayer1];
    
    //上层
    _waveLayer2 = [CAShapeLayer layer];
    _waveLayer2.fillColor = WaveColor2.CGColor;
    _waveLayer2.strokeColor = WaveColor2.CGColor;
    [self.layer addSublayer:_waveLayer2];
    
    //画了个圆
    self.layer.cornerRadius = self.bounds.size.width/2.0f;
    self.layer.masksToBounds = true;
    self.backgroundColor = BackGroundColor;
}
 
//初始化数据
-(void)buildData
{
    //振幅
    _waveAmplitude = 10;
    //角速度
    _wavePalstance = M_PI/self.bounds.size.width;
    //偏距
    _waveY = 0;
    //初相
    _waveX = 0;
    //x轴移动速度
    _waveMoveSpeed = _wavePalstance * 10;
    //以屏幕刷新速度为周期刷新曲线的位置
    _disPlayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(updateWave:)];
    [_disPlayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
}
/**
 保持和屏幕的刷新速度相同，iphone的刷新速度是60Hz,即每秒60次的刷新
 */
-(void)updateWave:(CADisplayLink *)link
{
    //更新X
    _waveX += _waveMoveSpeed;
    [self updateWaveY];
    [self updateWave1];
    [self updateWave2];
}
 
//更新偏距的大小 直到达到目标偏距 让wave有一个匀速增长的效果
-(void)updateWaveY
{
    CGFloat targetY = self.bounds.size.height - _progress * self.bounds.size.height;
    if (_waveY < targetY) {
        _waveY += 2;
    }
    if (_waveY > targetY ) {
        _waveY -= 2;
    }
}
 
//更新第一层曲线
-(void)updateWave1
{
    //波浪宽度
    CGFloat waterWaveWidth = self.bounds.size.width;
    //初始化运动路径
    CGMutablePathRef path = CGPathCreateMutable();
    //设置起始位置
    CGPathMoveToPoint(path, nil, 0, _waveY);
    //初始化波浪其实Y为偏距
    CGFloat y = _waveY;
    //正弦曲线公式为： y=Asin(ωx+φ)+k;
    for (float x = 0.0f; x <= waterWaveWidth ; x++) {
        y = _waveAmplitude * cos(_wavePalstance * x + _waveX) + _waveY;
        CGPathAddLineToPoint(path, nil, x, y);
    }
    //填充底部颜色
    CGPathAddLineToPoint(path, nil, waterWaveWidth, self.bounds.size.height);
    CGPathAddLineToPoint(path, nil, 0, self.bounds.size.height);
    CGPathCloseSubpath(path);
    _waveLayer1.path = path;
    CGPathRelease(path);
}
 
//更新第二层曲线
-(void)updateWave2
{
    //波浪宽度
    CGFloat waterWaveWidth = self.bounds.size.width;
    //初始化运动路径
    CGMutablePathRef path = CGPathCreateMutable();
    //设置起始位置
    CGPathMoveToPoint(path, nil, 0, _waveY);
    //初始化波浪其实Y为偏距
    CGFloat y = _waveY;
    //正弦曲线公式为： y=Asin(ωx+φ)+k;
    for (float x = 0.0f; x <= waterWaveWidth ; x++) {
        y = _waveAmplitude * sin(_wavePalstance * x + _waveX) + _waveY;
        CGPathAddLineToPoint(path, nil, x, y);
    }
    //添加终点路径、填充底部颜色
    CGPathAddLineToPoint(path, nil, waterWaveWidth, self.bounds.size.height);
    CGPathAddLineToPoint(path, nil, 0, self.bounds.size.height);
    CGPathCloseSubpath(path);
    _waveLayer2.path = path;
    CGPathRelease(path);
    
}
 
//设置需要显示的进度，y轴的更新会在[updateWaveY]方法中实现
-(void)setProgress:(CGFloat)progress
{
    _progress = progress;
}
 
//停止动画
-(void)stop
{
    if (_disPlayLink) {
        [_disPlayLink invalidate];
        _disPlayLink = nil;
    }
}
//回收内存
-(void)dealloc
{
    [self stop];
    if (_waveLayer1) {
        [_waveLayer1 removeFromSuperlayer];
        _waveLayer1 = nil;
    }
    if (_waveLayer2) {
        [_waveLayer2 removeFromSuperlayer];
        _waveLayer2 = nil;
    }
}

```
本文Demo地址：[Github](https://github.com/Fendouzhe/LRAnimations.git)

