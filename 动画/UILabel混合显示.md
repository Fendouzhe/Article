动画效果
![](http://upload-images.jianshu.io/upload_images/1464492-d087235d5aed8792.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
实现思路：同一位置创建两个一样大小view，一个upView一个downView，其分别有一个和本身同样大小的子控件UILabel，展示相同的文字。upView在上面，红色背景色，其上面的label是白色字体，透明背景。downView在下面，透明背景色，红色边框，其上面的label是红色字体，透明背景。通过改变upView的宽度实现动画效果。

源码(源码没有进行封装,细节都没有处理,望见谅)代码如下：
```
- (void)viewDidLoad {
    [super viewDidLoad];
    /*
     给upView的frame值做动画才是label能够混色显示的核心
     
     upView(红色背景)   ===>  upLabel(白色底字)
     |                       |
     |                       |
     |                       |
     |                       |
     downView(白色背景) ===> downLabel(红色底字)
     
     */
    _downView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 220, 22)];
    _downView.center = self.view.center;
    [self.view addSubview:_downView];
    _downView.layer.cornerRadius = 2;
    _downView.layer.masksToBounds = YES;
    _downView.layer.borderColor = [UIColor redColor].CGColor;
    _downView.layer.borderWidth = 0.5f;
    _downView.backgroundColor = [UIColor whiteColor];
    
    _downLabel = [[UILabel alloc] initWithFrame:_downView.bounds];
    [_downView addSubview:_downLabel];
    _downLabel.textAlignment = NSTextAlignmentCenter;
    _downLabel.text = @"LeiLuRong - iOS Programmer";
    _downLabel.font = [UIFont systemFontOfSize:14];
    _downLabel.textColor = [UIColor redColor];
    
    _upView = [[UIView alloc] initWithFrame:_downView.bounds];
    _upView.center = self.view.center;
    [self.view addSubview:_upView];
    _upView.layer.cornerRadius = 2;
    _upView.layer.masksToBounds = YES;
    _upView.backgroundColor = [UIColor redColor];
    
    _upLabel = [[UILabel alloc] initWithFrame:_upView.bounds];
    [_upView addSubview:_upLabel];
    _upLabel.textAlignment = NSTextAlignmentCenter;
    _upLabel.text = _downLabel.text;
    _upLabel.font = _downLabel.font;
    _upLabel.textColor = [UIColor whiteColor];
    
    _timer = [[GCDTimer alloc] initInQueue:[GCDQueue mainQueue]];
    __weak typeof(self) weakSelf = self;
    [_timer event:^{
        [UIView animateWithDuration:0.5f delay:0.f usingSpringWithDamping:3.f initialSpringVelocity:0 options:0 animations:^{
            
            weakSelf.upView.width = arc4random()%220;
            
        } completion:^(BOOL finished) {
            
        }];
    } timeInterval:NSEC_PER_SEC];
    [_timer start];
}
```
代码已经放在[GitHub](https://github.com/Fendouzhe/LRAnimations),里面有更多动画效果，在不断更新，欢迎下载查看，喜欢可以star.
