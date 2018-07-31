##线程保活工具类：LRPermenantThread
```
#import <Foundation/Foundation.h>

typedef void (^MJPermenantThreadTask)(void);

@interface LRPermenantThread : NSObject

/**
 开启线程
 */
//- (void)run;

/**
 在当前子线程执行一个任务
 */
- (void)executeTask:(MJPermenantThreadTask)task;

/**
 结束线程
 */
- (void)stop;

@end
```

```

/** MJThread **/
@interface LRThread : NSThread
@end
@implementation LRThread
- (void)dealloc
{
    NSLog(@"%s", __func__);
}
@end

/** MJPermenantThread **/
@interface LRPermenantThread()
@property (strong, nonatomic) LRThread *innerThread;
@property (assign, nonatomic, getter=isStopped) BOOL stopped;
@end

@implementation LRPermenantThread
#pragma mark - public methods
- (instancetype)init
{
    if (self = [super init]) {
        self.stopped = NO;
        
        __weak typeof(self) weakSelf = self;
        
        self.innerThread = [[LRThread alloc] initWithBlock:^{
            [[NSRunLoop currentRunLoop] addPort:[[NSPort alloc] init] forMode:NSDefaultRunLoopMode];
            
            while (weakSelf && !weakSelf.isStopped) {
                [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
            }
        }];
        
        [self.innerThread start];
    }
    return self;
}

//- (void)run
//{
//    if (!self.innerThread) return;
//
//    [self.innerThread start];
//}

- (void)executeTask:(MJPermenantThreadTask)task
{
    if (!self.innerThread || !task) return;
    
    [self performSelector:@selector(__executeTask:) onThread:self.innerThread withObject:task waitUntilDone:NO];
}

- (void)stop
{
    if (!self.innerThread) return;
    
    [self performSelector:@selector(__stop) onThread:self.innerThread withObject:nil waitUntilDone:YES];
}

- (void)dealloc
{
    NSLog(@"%s", __func__);
    
    [self stop];
}

#pragma mark - private methods
- (void)__stop
{
    self.stopped = YES;
    CFRunLoopStop(CFRunLoopGetCurrent());
    self.innerThread = nil;
}

- (void)__executeTask:(MJPermenantThreadTask)task
{
    task();
}

@end
```

##调用：
```
#import "ViewController.h"
#import "LRPermenantThread.h"

@interface ViewController ()
@property (strong, nonatomic) LRPermenantThread *thread;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.thread = [[LRPermenantThread alloc] init];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self.thread executeTask:^{
        NSLog(@"执行任务 - %@", [NSThread currentThread]);
    }];
}

- (IBAction)stop {
    [self.thread stop];
}

- (void)dealloc
{
    NSLog(@"%s", __func__);
}

@end
```

##C语言版本:
```
#import "LRPermenantThread.h"

/** MJThread **/
@interface LRThread : NSThread
@end
@implementation LRThread
- (void)dealloc
{
    NSLog(@"%s", __func__);
}
@end

/** MJPermenantThread **/
@interface LRPermenantThread()
@property (strong, nonatomic) LRThread *innerThread;
@end

@implementation LRPermenantThread
#pragma mark - public methods
- (instancetype)init
{
    if (self = [super init]) {
        self.innerThread = [[LRThread alloc] initWithBlock:^{
            NSLog(@"begin----");
            
            // 创建上下文（要初始化一下结构体）
            CFRunLoopSourceContext context = {0};
            
            // 创建source
            CFRunLoopSourceRef source = CFRunLoopSourceCreate(kCFAllocatorDefault, 0, &context);
            
            // 往Runloop中添加source
            CFRunLoopAddSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);
            
            // 销毁source
            CFRelease(source);
            
            // 启动
            CFRunLoopRunInMode(kCFRunLoopDefaultMode, 1.0e10, false);
            
            NSLog(@"end----");
        }];
        
        [self.innerThread start];
    }
    return self;
}

//- (void)run
//{
//    if (!self.innerThread) return;
//
//    [self.innerThread start];
//}

- (void)executeTask:(MJPermenantThreadTask)task
{
    if (!self.innerThread || !task) return;
    
    [self performSelector:@selector(__executeTask:) onThread:self.innerThread withObject:task waitUntilDone:NO];
}

- (void)stop
{
    if (!self.innerThread) return;
    
    [self performSelector:@selector(__stop) onThread:self.innerThread withObject:nil waitUntilDone:YES];
}

- (void)dealloc
{
    NSLog(@"%s", __func__);
    
    [self stop];
}

#pragma mark - private methods
- (void)__stop
{
    CFRunLoopStop(CFRunLoopGetCurrent());
    self.innerThread = nil;
}

- (void)__executeTask:(MJPermenantThreadTask)task
{
    task();
}

@end
```
