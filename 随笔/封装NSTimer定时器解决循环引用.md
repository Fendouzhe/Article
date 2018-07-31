##LRWeakTimer
```
#import <Foundation/Foundation.h>

@interface LRWeakTimer : NSObject


+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats userInfo:(nullable id)userInfo mode:(NSRunLoopMode)mode block:(void (^)(NSTimer *timer))block;

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)repeats mode:(NSRunLoopMode)mode;


@end
```

```
#import "LRWeakTimer.h"

@interface LRWeakTimer()

@property (nonatomic, copy)void (^block)(NSTimer *timer);
///弱引用
@property (nonatomic, weak)id target;

@property (nonatomic, assign)SEL selector;

@end


@implementation LRWeakTimer

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats userInfo:(nullable id)userInfo mode:(NSRunLoopMode)mode block:(void (^)(NSTimer *timer))block
{
    LRWeakTimer *timerObj = [[self alloc] init];
    NSTimer *timer = [NSTimer timerWithTimeInterval:interval target:timerObj selector:@selector(timeEvent:) userInfo:userInfo repeats:repeats];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:mode];
    timerObj.block = block;
    return timer;
}
- (void)timeEvent:(NSTimer *)timer
{
    //NSLog(@"userInfo = %@",timer.userInfo);
    if (self.block) {
        self.block(timer);
    }
}


+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)repeats mode:(NSRunLoopMode)mode
{
    LRWeakTimer *timerObj = [[self alloc] init];
    timerObj.target = aTarget;
    timerObj.selector = aSelector;
    NSTimer *timer = [NSTimer timerWithTimeInterval:interval target:timerObj selector:@selector(fire:) userInfo:userInfo repeats:repeats];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:mode];
    return timer;
}

- (void)fire:(NSTimer *)timer{
    //NSLog(@"userInfo = %@",timer.userInfo);
    if ([self.target respondsToSelector:self.selector]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
        [self.target performSelector:self.selector withObject:timer];
#pragma clang diagnostic pop
    }
}


- (void)dealloc{
    NSLog(@"%s",__func__);
}

@end
```

##调用
```
#import "ViewController.h"
#import "MJProxy.h"
#import "MJProxy1.h"
#import "LRWeakTimer.h"

@interface ViewController ()
@property (strong, nonatomic) NSTimer *timer0;
@property (strong, nonatomic) NSTimer *timer1;
@property (nonatomic, copy)NSString *name;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.name = @"名字";
    __weak typeof(self) weakSelf = self;
    self.timer0 = [LRWeakTimer scheduledTimerWithTimeInterval:1.0 repeats:YES userInfo:@{@"BlockTimer":@"666666"} mode:NSRunLoopCommonModes block:^(NSTimer *timer) {
        NSLog(@"%s name = %@ timer.userInfo = %@", __func__,weakSelf.name,timer.userInfo);
    }];
    
    self.timer1 = [LRWeakTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timerTest:) userInfo:@{@"LRWeakTimer":@"666666"} repeats:YES mode:NSRunLoopCommonModes];
}

- (void)timerTest:(NSTimer *)timer
{
    NSLog(@"%s userInfo = %@", __func__,timer.userInfo);
}

- (void)dealloc
{
    NSLog(@"%s", __func__);
    [self.timer0 invalidate];
    [self.timer1 invalidate];
}

@end
```
