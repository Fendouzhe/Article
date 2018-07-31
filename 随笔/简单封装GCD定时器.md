##LRGCDTimer
```
#import <Foundation/Foundation.h>

@interface LRGCDTimer : NSObject

+ (NSString *)execTask:(void(^)(void))task
           start:(NSTimeInterval)start
        interval:(NSTimeInterval)interval
         repeats:(BOOL)repeats
           async:(BOOL)async;

+ (NSString *)execTask:(id)target
              selector:(SEL)selector
                 start:(NSTimeInterval)start
              interval:(NSTimeInterval)interval
               repeats:(BOOL)repeats
                 async:(BOOL)async;

+ (void)cancelTask:(NSString *)name;

@end
```

```
#import "LRGCDTimer.h"

@implementation LRGCDTimer

static NSMutableDictionary *timers;
dispatch_semaphore_t semaphore;
+ (void)initialize
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        timers = [NSMutableDictionary dictionary];
        semaphore = dispatch_semaphore_create(1);
    });
}

+ (NSString *)execTask:(void (^)(void))task start:(NSTimeInterval)start interval:(NSTimeInterval)interval repeats:(BOOL)repeats async:(BOOL)async
{
    if (!task || start < 0 || (interval <= 0 && repeats)) return nil;
    
    // 队列
    dispatch_queue_t queue = async ? dispatch_get_global_queue(0, 0) : dispatch_get_main_queue();
    
    // 创建定时器
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    
    // 设置时间
    dispatch_source_set_timer(timer,
                              dispatch_time(DISPATCH_TIME_NOW, start * NSEC_PER_SEC),
                              interval * NSEC_PER_SEC, 0);
    
    ///多线程同步处理，加锁、解锁
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    // 定时器的唯一标识
    NSString *name = [NSString stringWithFormat:@"%zd", timers.count];
    // 存放到字典中
    timers[name] = timer;
    dispatch_semaphore_signal(semaphore);
    
    // 设置回调
    dispatch_source_set_event_handler(timer, ^{
        task();
        
        if (!repeats) { // 不重复的任务
            [self cancelTask:name];
        }
    });
    
    // 启动定时器
    dispatch_resume(timer);
    
    return name;
}

+ (NSString *)execTask:(id)target selector:(SEL)selector start:(NSTimeInterval)start interval:(NSTimeInterval)interval repeats:(BOOL)repeats async:(BOOL)async
{
    if (!target || !selector) return nil;
    
    return [self execTask:^{
        if ([target respondsToSelector:selector]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            [target performSelector:selector];
#pragma clang diagnostic pop
        }
    } start:start interval:interval repeats:repeats async:async];
}

+ (void)cancelTask:(NSString *)name
{
    if (name.length == 0) return;
    
    ///多线程同步处理，加锁、解锁
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    // 取出定时器
    dispatch_source_t timer = timers[name];
    if (timer) {
        // 取消定时器
        dispatch_source_cancel(timer);
        [timers removeObjectForKey:name];
    }
    dispatch_semaphore_signal(semaphore);
}

@end
```
##调用：
```
#import "ViewController.h"
#import "LRGCDTimer.h"

@interface ViewController ()
@property (copy, nonatomic) NSString *task;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 接口设计
    self.task = [LRGCDTimer execTask:self
                         selector:@selector(doTask)
                            start:2.0
                         interval:1.0
                          repeats:YES
                            async:NO];
    
}

- (void)doTask
{
    NSLog(@"doTask - %@", [NSThread currentThread]);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    //取消定时
    [LRGCDTimer cancelTask:self.task];
}
@end
```
