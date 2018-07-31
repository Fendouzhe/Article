#序言

微信现在这么普及，功能也做的越来越强大，不知大家对于微信朋友圈发视频截取的功能或者苹果拍视频对视频编辑的功能有没有了解（作者这里也猜测，微信的这个功能也是仿苹果的）。感觉这个功能确实很方便实用，近来作者也在研究音视频功能，所以就实现了一下这个功能。

功能其实看着挺简单，实现过程也踩了不少坑。一方面记录一下；另一方面也算是对实现过程的再一次梳理，这样大家看代码也会比较明白。

#效果

我们先看看我实现的效果

![image](http://upload-images.jianshu.io/upload_images/1464492-14114801c3b13765?imageMogr2/auto-orient/strip)

#实现

###实现过程分析

整个功能可以分为三部分：

*   视频播放

    这部分我们单独封装一个视频播放器即可

*   下边的滑动视图

    这部分实现过程比较复杂，一共分成了4部分。灰色遮盖、左右把手滑块、滑块中间上下两条线、图片管理视图

*   控制器视图逻辑组装和功能实现

###视频播放器的封装

这里使用AVPlayer、playerLayer、AVPlayerItem这三个类实现了视频播放功能；由于整个事件都是基于KVO监听的，所以增加了Block代码提供了对外监听使用。
```
#import "FOFMoviePlayer.h"
@interface FOFMoviePlayer()
{
    AVPlayerLooper *_playerLooper;
    AVPlayerItem *_playItem;
    BOOL _loop;
}
@property(nonatomic,strong)NSURL *url;

@property(nonatomic,strong)AVPlayer *player;
@property(nonatomic,strong)AVPlayerLayer *playerLayer;
@property(nonatomic,strong)AVPlayerItem *playItem;

@property (nonatomic,assign) CMTime duration;
@end
@implementation FOFMoviePlayer

-(instancetype)initWithFrame:(CGRect)frame url:(NSURL *)url superLayer:(CALayer *)superLayer{
    self = [super init];
    if (self) {
        [self initplayers:superLayer];
        _playerLayer.frame = frame;
        self.url = url;
    }
    return self;
}
-(instancetype)initWithFrame:(CGRect)frame url:(NSURL *)url superLayer:(CALayer *)superLayer loop:(BOOL)loop{
    self = [self initWithFrame:frame url:url superLayer:superLayer];
    if (self) {
        _loop = loop;
    }
    return self;
}
- (void)initplayers:(CALayer *)superLayer{
    self.player = [[AVPlayer alloc] init];
    self.playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];
    self.playerLayer.videoGravity = AVLayerVideoGravityResize;
    [superLayer addSublayer:self.playerLayer];
}
- (void)initLoopPlayers:(CALayer *)superLayer{
    self.player = [[AVQueuePlayer alloc] init];
    self.playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];
    self.playerLayer.videoGravity = AVLayerVideoGravityResize;
    [superLayer addSublayer:self.playerLayer];
}
-(void)fof_play{
    [self.player play];
}
-(void)fof_pause{
    [self.player pause];
}

#pragma mark - Observe
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    if ([keyPath isEqualToString:@"status"]) {
        AVPlayerItem *item = (AVPlayerItem *)object;
        AVPlayerItemStatus status = [[change objectForKey:@"new"] intValue]; // 获取更改后的状态
        if (status == AVPlayerItemStatusReadyToPlay) {
            _duration = item.duration;//只有在此状态下才能获取，不能在AVPlayerItem初始化后马上获取
            NSLog(@"准备播放");
            if (self.blockStatusReadyPlay) {
                self.blockStatusReadyPlay(item);
            }
        } else if (status == AVPlayerItemStatusFailed) {
            if (self.blockStatusFailed) {
                self.blockStatusFailed();
            }
            AVPlayerItem *item = (AVPlayerItem *)object;
            NSLog(@"%@",item.error);
            NSLog(@"AVPlayerStatusFailed");
        } else {
            self.blockStatusUnknown();
            NSLog(@"%@",item.error);
            NSLog(@"AVPlayerStatusUnknown");
        }
    }else if ([keyPath isEqualToString:@"tracking"]){
        NSInteger status = [change[@"new"] integerValue];
        if (self.blockTracking) {
            self.blockTracking(status);
        }
        if (status) {//正在拖动
            [self.player pause];
        }else{//停止拖动

        }
    }else if ([keyPath isEqualToString:@"loadedTimeRanges"]){
        NSArray *array = _playItem.loadedTimeRanges;
        CMTimeRange timeRange = [array.firstObject CMTimeRangeValue];//本次缓冲时间范围
        CGFloat startSeconds = CMTimeGetSeconds(timeRange.start);
        CGFloat durationSeconds = CMTimeGetSeconds(timeRange.duration);
        NSTimeInterval totalBuffer = startSeconds + durationSeconds;//缓冲总长度
        double progress = totalBuffer/CMTimeGetSeconds(_duration);
        if (self.blockLoadedTimeRanges) {
            self.blockLoadedTimeRanges(progress);
        }
        NSLog(@"当前缓冲时间：%f",totalBuffer);
    }else if ([keyPath isEqualToString:@"playbackBufferEmpty"]){
        NSLog(@"缓存不够，不能播放！");
    }else if ([keyPath isEqualToString:@"playbackLikelyToKeepUp"]){
        if (self.blockPlaybackLikelyToKeepUp) {
            self.blockPlaybackLikelyToKeepUp([change[@"new"] boolValue]);
        }
    }

}

-(void)setUrl:(NSURL *)url{
    _url = url;
    [self.player replaceCurrentItemWithPlayerItem:self.playItem];
}

-(AVPlayerItem *)playItem{
    _playItem = [[AVPlayerItem alloc] initWithURL:_url];
    //监听播放器的状态，准备好播放、失败、未知错误
    [_playItem addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:nil];
    //    监听缓存的时间
    [_playItem addObserver:self forKeyPath:@"loadedTimeRanges" options:NSKeyValueObservingOptionNew context:nil];
    //    监听获取当缓存不够，视频加载不出来的情况：
    [_playItem addObserver:self forKeyPath:@"playbackBufferEmpty" options:NSKeyValueObservingOptionNew context:nil];
    //    用于监听缓存足够播放的状态
    [_playItem addObserver:self forKeyPath:@"playbackLikelyToKeepUp" options:NSKeyValueObservingOptionNew context:nil];

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(private_playerMovieFinish) name:AVPlayerItemDidPlayToEndTimeNotification object:nil];

    return _playItem;
}
- (void)private_playerMovieFinish{
    NSLog(@"播放结束");
    if (self.blockPlayToEndTime) {
        self.blockPlayToEndTime();
    }
    if (_loop) {//默认提供一个循环播放的功能
        [self.player pause];
        CMTime time = CMTimeMake(1, 1);
        __weak typeof(self)this = self;
        [self.player seekToTime:time completionHandler:^(BOOL finished) {
            [this.player play];
        }];
    }
}
-(void)dealloc{
    NSLog(@"-----销毁-----");
}
@end
```
视频播放器就不重点讲了，作者计划单独写一篇有关视频播放器的。

###下边的滑动视图

####灰色遮盖

灰色遮盖比较简单这里作者只是用了UIView
```
self.leftMaskView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, height)];
self.leftMaskView.backgroundColor = [UIColor grayColor];
self.leftMaskView.alpha = 0.8;
[self addSubview:self.leftMaskView];
self.rightMaskView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, height)];
self.rightMaskView.backgroundColor = [UIColor grayColor];
self.rightMaskView.alpha = 0.8;
```
###滑块中间上下两条线

这两根线单独封装了一个视图Line,一开始也想到用一个UIView就好了，但是发现一个问题，就是把手的滑动与线的滑动速度不匹配,线比较慢。
```
@implementation Line

-(void)setBeginPoint:(CGPoint)beginPoint{
    _beginPoint = beginPoint;
    [self setNeedsDisplay];
}
-(void)setEndPoint:(CGPoint)endPoint{
    _endPoint = endPoint;
    [self setNeedsDisplay];
}
- (void)drawRect:(CGRect)rect {
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetLineWidth(context, 3);
    CGContextSetStrokeColorWithColor(context, [UIColor colorWithWhite:0.9 alpha:1].CGColor);
    CGContextMoveToPoint(context, self.beginPoint.x, self.beginPoint.y);
    CGContextAddLineToPoint(context, self.endPoint.x, self.endPoint.y);
    CGContextStrokePath(context);
}
```

###图片管理视图

这里封装了一个VideoPieces，用来组装把手、线、遮盖的逻辑，并且用来显示图片。由于图片只有10张，所以这里紧紧是一个for循环，增加了10个UIImageView
```
@interface VideoPieces()
{
    CGPoint _beginPoint;
}
@property(nonatomic,strong) Haft *leftHaft;
@property(nonatomic,strong) Haft *rightHaft;
@property(nonatomic,strong) Line *topLine;
@property(nonatomic,strong) Line *bottomLine;
@property(nonatomic,strong) UIView *leftMaskView;
@property(nonatomic,strong) UIView *rightMaskView;
@end
@implementation VideoPieces
-(instancetype)initWithFrame:(CGRect)frame{
    self = [super initWithFrame:frame];
    if (self) {
        [self initSubViews:frame];
    }
    return self;
}
- (void)initSubViews:(CGRect)frame{
    CGFloat height = CGRectGetHeight(frame);
    CGFloat width = CGRectGetWidth(frame);
    CGFloat minGap = 30;
    CGFloat widthHaft = 10;
    CGFloat heightLine = 3;
    _leftHaft = [[Haft alloc] initWithFrame:CGRectMake(0, 0, widthHaft, height)];
    _leftHaft.alpha = 0.8;
    _leftHaft.backgroundColor = [UIColor colorWithWhite:0.9 alpha:1];
    _leftHaft.rightEdgeInset = 20;
    _leftHaft.lefEdgeInset = 5;
    __weak typeof(self) this = self;
    [_leftHaft setBlockMove:^(CGPoint point) {
        CGFloat maxX = this.rightHaft.frame.origin.x-minGap;
        if (point.x<maxX) {
            this.topLine.beginPoint = CGPointMake(point.x, heightLine/2.0);
            this.bottomLine.beginPoint = CGPointMake(point.x, heightLine/2.0);
            this.leftHaft.frame = CGRectMake(point.x, 0, widthHaft, height);
            this.leftMaskView.frame = CGRectMake(0, 0, point.x, height);
            if (this.blockSeekOffLeft) {
                this.blockSeekOffLeft(point.x);
            }
        }
    }];
    [_leftHaft setBlockMoveEnd:^{
        if (this.blockMoveEnd) {
            this.blockMoveEnd();
        }
    }];
    _rightHaft = [[Haft alloc] initWithFrame:CGRectMake(width-widthHaft, 0, widthHaft, height)];
    _rightHaft.alpha = 0.8;
    _rightHaft.backgroundColor = [UIColor colorWithWhite:0.9 alpha:1];
    _rightHaft.lefEdgeInset = 20;
    _rightHaft.rightEdgeInset = 5;
    [_rightHaft setBlockMove:^(CGPoint point) {
        CGFloat minX = this.leftHaft.frame.origin.x+minGap+CGRectGetWidth(this.rightHaft.bounds);
        if (point.x>=minX) {
            this.topLine.endPoint = CGPointMake(point.x-widthHaft, heightLine/2.0);
            this.bottomLine.endPoint = CGPointMake(point.x-widthHaft, heightLine/2.0);
            this.rightHaft.frame = CGRectMake(point.x, 0, widthHaft, height);
            this.rightMaskView.frame = CGRectMake(point.x+widthHaft, 0, width-point.x-widthHaft, height);
            if (this.blockSeekOffRight) {
                this.blockSeekOffRight(point.x);
            }
        }
    }];
    [_rightHaft setBlockMoveEnd:^{
        if (this.blockMoveEnd) {
            this.blockMoveEnd();
        }
    }];
    _topLine = [[Line alloc] init];
    _topLine.alpha = 0.8;
    _topLine.frame = CGRectMake(widthHaft, 0, width-2*widthHaft, heightLine);
    _topLine.beginPoint = CGPointMake(0, heightLine/2.0);
    _topLine.endPoint = CGPointMake(CGRectGetWidth(_topLine.bounds), heightLine/2.0);
    _topLine.backgroundColor = [UIColor clearColor];
    [self addSubview:_topLine];

    _bottomLine = [[Line alloc] init];
    _bottomLine.alpha = 0.8;
    _bottomLine.frame = CGRectMake(widthHaft, height-heightLine, width-2*widthHaft, heightLine);
    _bottomLine.beginPoint = CGPointMake(0, heightLine/2.0);
    _bottomLine.endPoint = CGPointMake(CGRectGetWidth(_bottomLine.bounds), heightLine/2.0);
    _bottomLine.backgroundColor = [UIColor clearColor];
    [self addSubview:_bottomLine];

    [self addSubview:_leftHaft];
    [self addSubview:_rightHaft];

    self.leftMaskView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, height)];
    self.leftMaskView.backgroundColor = [UIColor grayColor];
    self.leftMaskView.alpha = 0.8;
    [self addSubview:self.leftMaskView];
    self.rightMaskView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, height)];
    self.rightMaskView.backgroundColor = [UIColor grayColor];
    self.rightMaskView.alpha = 0.8;
    [self addSubview:self.rightMaskView];
}
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    UITouch *touch = touches.anyObject;
    _beginPoint = [touch locationInView:self];

}
```

###把手的实现

把手的实现这里优化了一点，就是滑动的时候比较灵敏，一开始用手指滑动的时候不是非常灵敏，经常手指滑动了，但是把手没有动。

增加了灵敏度的方法其实就是增加了接收事件区域的大小，重写了-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event这个方法
```
@implementation Haft
-(instancetype)initWithFrame:(CGRect)frame{
    self = [super initWithFrame:frame];
    if (self) {
        self.userInteractionEnabled = true;
    }
    return self;
}

-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
    CGRect rect = CGRectMake(self.bounds.origin.x-self.lefEdgeInset, self.bounds.origin.y-self.topEdgeInset, CGRectGetWidth(self.bounds)+self.lefEdgeInset+self.rightEdgeInset, CGRectGetHeight(self.bounds)+self.bottomEdgeInset+self.topEdgeInset);
    if (CGRectContainsPoint(rect, point)) {
        return YES;
    }
    return NO;
}
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    NSLog(@"开始");
}
-(void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    NSLog(@"Move");
    UITouch *touch = touches.anyObject;
    CGPoint point = [touch locationInView:self.superview];
    CGFloat maxX = CGRectGetWidth(self.superview.bounds)-CGRectGetWidth(self.bounds);
    if (point.x>maxX) {
        point.x = maxX;
    }
    if (point.x>=0&&point.x<=(CGRectGetWidth(self.superview.bounds)-CGRectGetWidth(self.bounds))&&self.blockMove) {
        self.blockMove(point);
    }
}
-(void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    if (self.blockMoveEnd) {
        self.blockMoveEnd();
    }
}
- (void)drawRect:(CGRect)rect {

    CGFloat width = CGRectGetWidth(self.bounds);
    CGFloat height = CGRectGetHeight(self.bounds);
    CGFloat lineWidth = 1.5;
    CGFloat lineHeight = 12;
    CGFloat gap = (width-lineWidth*2)/3.0;
    CGFloat lineY = (height-lineHeight)/2.0;

    CGContextRef context  = UIGraphicsGetCurrentContext();
    CGContextSetLineWidth(context, lineWidth);
    CGContextSetStrokeColorWithColor(context, [[UIColor grayColor] colorWithAlphaComponent:0.8].CGColor);
    CGContextMoveToPoint(context, gap+lineWidth/2, lineY);
    CGContextAddLineToPoint(context, gap+lineWidth/2, lineY+lineHeight);
    CGContextStrokePath(context);

    CGContextSetLineWidth(context, lineWidth);
    CGContextSetStrokeColorWithColor(context, [[UIColor grayColor] colorWithAlphaComponent:0.8].CGColor);
    CGContextMoveToPoint(context, gap*2+lineWidth+lineWidth/2, lineY);
    CGContextAddLineToPoint(context, gap*2+lineWidth+lineWidth/2, lineY+lineHeight);
    CGContextStrokePath(context);

}
```

###控制器视图逻辑组装和功能实现

这部分逻辑是最重要也是最复杂的。

*   获取10张缩略图
```
- (NSArray *)getVideoThumbnail:(NSString *)path count:(NSInteger)count splitCompleteBlock:(void(^)(BOOL success, NSMutableArray *splitimgs))splitCompleteBlock {
    AVAsset *asset = [AVAsset assetWithURL:[NSURL fileURLWithPath:path]];
    NSMutableArray *arrayImages = [NSMutableArray array];
    [asset loadValuesAsynchronouslyForKeys:@[@"duration"] completionHandler:^{
        AVAssetImageGenerator *generator = [AVAssetImageGenerator assetImageGeneratorWithAsset:asset];
//        generator.maximumSize = CGSizeMake(480,136);//如果是CGSizeMake(480,136)，则获取到的图片是{240, 136}。与实际大小成比例
        generator.appliesPreferredTrackTransform = YES;//这个属性保证我们获取的图片的方向是正确的。比如有的视频需要旋转手机方向才是视频的正确方向。
        /**因为有误差，所以需要设置以下两个属性。如果不设置误差有点大，设置了之后相差非常非常的小**/
        generator.requestedTimeToleranceAfter = kCMTimeZero;
        generator.requestedTimeToleranceBefore = kCMTimeZero;
        Float64 seconds = CMTimeGetSeconds(asset.duration);
        NSMutableArray *array = [NSMutableArray array];
        for (int i = 0; i<count; i++) {
            CMTime time = CMTimeMakeWithSeconds(i*(seconds/10.0),1);//想要获取图片的时间位置
            [array addObject:[NSValue valueWithCMTime:time]];
        }
        __block int i = 0;
        [generator generateCGImagesAsynchronouslyForTimes:array completionHandler:^(CMTime requestedTime, CGImageRef  _Nullable imageRef, CMTime actualTime, AVAssetImageGeneratorResult result, NSError * _Nullable error) {

            i++;
            if (result==AVAssetImageGeneratorSucceeded) {
                UIImage *image = [UIImage imageWithCGImage:imageRef];
                [arrayImages addObject:image];
            }else{
                NSLog(@"获取图片失败！！！");
            }
            if (i==count) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    splitCompleteBlock(YES,arrayImages);
                });
            }
        }];
    }];
    return arrayImages;

}
```

10张图片很容易获取到，不过这里要注意一点：回调的时候要放到异步主队列回调！要不会出现图片显示延迟比较严重的问题。

*   监听左右滑块事件
```
[_videoPieces setBlockSeekOffLeft:^(CGFloat offX) {
    this.seeking = true;
    [this.moviePlayer fof_pause];
    this.lastStartSeconds = this.totalSeconds*offX/CGRectGetWidth(this.videoPieces.bounds);
    [this.moviePlayer.player seekToTime:CMTimeMakeWithSeconds(this.lastStartSeconds, 1) toleranceBefore:kCMTimeZero toleranceAfter:kCMTimeZero];
}];
[_videoPieces setBlockSeekOffRight:^(CGFloat offX) {
    this.seeking = true;
    [this.moviePlayer fof_pause];
    this.lastEndSeconds = this.totalSeconds*offX/CGRectGetWidth(this.videoPieces.bounds);
    [this.moviePlayer.player seekToTime:CMTimeMakeWithSeconds(this.lastEndSeconds, 1) toleranceBefore:kCMTimeZero toleranceAfter:kCMTimeZero];
}];
```

这里通过监听左右滑块的事件，将偏移距离转换成时间，从而设置播放器的开始时间和结束时间。

*   循环播放
```
self.timeObserverToken = [self.moviePlayer.player addPeriodicTimeObserverForInterval:CMTimeMakeWithSeconds(0.5, NSEC_PER_SEC) queue:dispatch_get_main_queue() usingBlock:^(CMTime time) {
    if (!this.seeking) {
        if (fabs(CMTimeGetSeconds(time)-this.lastEndSeconds)<=0.02) {
                [this.moviePlayer fof_pause];
                [this private_replayAtBeginTime:this.lastStartSeconds];
            }
    }
}];
```

这里有两个注意点：

1\. addPeriodicTimeObserverForInterval要进行释放，否则会有内存泄漏。
```
-(void)dealloc{
    [self.moviePlayer.player removeTimeObserver:self.timeObserverToken];
}
```

2.这里监听了播放时间，进而计算是否达到了我们右边把手拖动的时间，如果达到了则重新播放。***这个问题作者思考了很久，怎么实现边播放边截取？差点进入了一个误区，真去截取视频。其实这里不用截取视频，只是控制播放时间和结束时间就可以了，最后只截取一次就行了。***

#总结

这次微信小视频编辑实现过程中，确实遇到了挺多的小问题。不过通过仔细的研究，最终完美实现了，有种如释重负的感觉。哈哈。

文章转载：[《iOS完美实现微信朋友圈视频截取》](http://flyoceanfish.top/2018/07/13/iOS完美实现微信朋友圈视频截取/)
[GitHub源码](https://github.com/FlyOceanFish/StudyAVFoundation)
