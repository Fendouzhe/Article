之前看过很多有关RunLoop的文章，其中要么是主要介绍RunLoop的基本概念，要么是主要讲解RunLoop的底层原理，很少用真正的实例来讲解RunLoop的，这其中有大部分原因是由于大家在项目中很少能用到RunLoop吧。基于这种原因，本文中将用很少的篇幅来对基础内容做以介绍，然后主要利用实例来加深大家对RunLoop的理解,[本文中的代码已经上传GitHub](https://github.com/Fendouzhe/RunLoopDemo.git),大家可以下载查看，有问题欢迎Issue我。本文主要分为如下几个部分:

*   RunLoop的基础知识

*   初识RunLoop，如何让RunLoop进驻线程

*   深入理解Perform Selector

*   一直"活着"的后台线程

*   深入理解NSTimer

*   让两个后台线程有依赖性的一种方式

*   NSURLConnetction的内部实现

*   AFNetWorking中是如何使用RunLoop的?

*   其它:利用GCD实现定时器功能

*   延伸阅读

* * *

## 一、RunLoop的基本概念:

什么是`RunLoop`？提到RunLoop，我们一般都会提到线程，这是为什么呢？先来看下[官方对`RunLoop`的定义](https://link.jianshu.com?t=https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1):`RunLoop`系统中和线程相关的基础架构的组成部分(**和线程相关**)，一个RunLoop是一个事件处理环，系统利用这个事件处理环来安排事务，协调输入的各种事件。`RunLoop`的目的是让你的线程在有工作的时候忙碌，没有工作的时候休眠(**和线程相关**)。可能这样说你还不是特别清楚`RunLoop`究竟是用来做什么的，打个比方来说明:我们把线程比作一辆跑车，把这辆跑车的主人比作`RunLoop`，那么在没有'主人'的时候，这个跑车的生命是**直线型**的，其启动，运行完之后就会废弃(没有人对其进行控制，'撞坏'被收回)，当有了`RunLoop`这个主人之后，‘线程’这辆跑车的生命就有了保障，这个时候，跑车的生命是**环形**的，并且在主人有比赛任务的时候就会被`RunLoop`这个主人所唤醒,在没有任务的时候可以休眠(在IOS中，开启线程是很消耗性能的，开启主线程要消耗1M内存，开启一个后台线程需要消耗512k内存，我们应当在线程没有任务的时候休眠，来释放所占用的资源，以便CPU进行更加高效的工作)，这样可以增加跑车的效率,也就是说`RunLoop`是为线程所服务的。这个例子有点不是很贴切，**线程和RunLoop之间是以键值对的形式一一对应的，其中key是thread，value是runLoop(这点可以从[苹果公开的源码中看出来](https://link.jianshu.com?t=http://opensource.apple.com/tarballs/CF/CF-855.17.tar.gz))**，**其实RunLoop是管理线程的一种机制，这种机制不仅在IOS上有，在Node.js中的EventLoop，Android中的Looper，都有类似的模式**。刚才所说的比赛任务就是**唤醒跑车这个线程**的一个`source`;`RunLoop Mode`就是，一系列输入的`source`,`timer`以及`observer`，`RunLoop Mode`包含以下几种: `NSDefaultRunLoopMode`,`NSEventTrackingRunLoopMode`,`UIInitializationRunLoopMode`,`NSRunLoopCommonModes`,`NSConnectionReplyMode`,`NSModalPanelRunLoopMode`,至于这些mode各自的含义，读者可自己查询，网上不乏这类资源;

## 二、初识RunLoop，如何让RunLoop进驻线程

我们在主线程中添加如下代码:

```
while (1) {
    NSLog(@"while begin");
    // the thread be blocked here
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
    // this will not be executed
    NSLog(@"while end");

}

```

这个时候我们可以看到主线程在执行完`[runLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];` 之后被阻塞而没有执行下面的`NSLog(@"while end")`;同时，我们利用GCD，将这段代码放到一个后台线程中:

```
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    while (1) {

        NSLog(@"while begin");
        NSRunLoop *subRunLoop = [NSRunLoop currentRunLoop];
        [subRunLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
        NSLog(@"while end");
    }

});

```

这个时候我们发现这个while循环会一直在执行；这是为什么呢?我们先将这两个`RunLoop`分别打印出来:

![主线程的RunLoop.png](http://upload-images.jianshu.io/upload_images/1464492-ed878ba36c82a6c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


由于这个日志比较长，我就只截取了上面的一部分。

我们再看我们新建的子线程中的`RunLoop`,打印出来之后:

![backGroundThreadRunLoop.png](http://upload-images.jianshu.io/upload_images/1464492-e7078dc0e249facf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从中可以看出来：我们新建的线程中:

```
sources0 = (null),
sources1 = (null),
observers = (null),
timers = (null),

```

我们看到虽然有Mode，但是我们没有给它`soures,observer,timer`，其实Mode中的这些`source,observer,timer`，统称为这个`Mode`的`item`，如果一个`Mode`中一个`item`都没有，则这个RunLoop会直接退出，不进入**循环**(其实线程之所以可以一直存在就是由于RunLoop将其带入了这个循环中)。下面我们为这个RunLoop添加个source:

```
     dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

        while (1) {

        NSPort *macPort = [NSPort port];
        NSLog(@"while begin");
        NSRunLoop *subRunLoop = [NSRunLoop currentRunLoop];
        [subRunLoop addPort:macPort forMode:NSDefaultRunLoopMode];
        [subRunLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
        NSLog(@"while end");
        NSLog(@"%@",subRunLoop);

    }    

});

```

这样我们可以看到能够实现了和主线程中相同的效果，线程在这个地方暂停了，为什么呢？我们明天让RunLoop在`distantFuture`之前都一直run的啊？相信大家已经猜出出来了。这个时候线程被`RunLoop`带到‘坑’里去了，这个‘坑’就是一个循环，在循环中这个线程可以在没有任务的时候休眠，在有任务的时候被唤醒；当然我们只用一个`while(1)`也可以让这个线程一直存在，但是这个线程会一直在唤醒状态，及时它没有任务也一直处于运转状态，这对于CPU来说是非常不高效的。

**小结:我们的RunLoop要想工作，必须要让它存在一个Item(source,observer或者timer)，主线程之所以能够一直存在，并且随时准备被唤醒就是应为系统为其添加了很多Item**

## 三、深入理解Perform Selector

我们先在主线程中使用下`performselector`:

```
- (void)tryPerformSelectorOnMianThread{

[self performSelector:@selector(mainThreadMethod) withObject:nil]; }

- (void)mainThreadMethod{

NSLog(@"execute %s",__func__);

// print: execute -[ViewController mainThreadMethod]
}

```

这样我们在ViewDidLoad中调用`tryPerformSelectorOnMianThread`,就会立即执行，并且输出:print: execute -[ViewController mainThreadMethod];

和上面的例子一样，我们使用GCD,让这个方法在后台线程中执行

```
 - (void)tryPerformSelectorOnBackGroundThread{

 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

});
}
- (void)backGroundThread{

NSLog(@"%u",[NSThread isMainThread]);

NSLog(@"execute %s",__FUNCTION__);

}

```

同样的，我们调用`tryPerformSelectorOnBackGroundThread`这个方法，我们会发现，下面的`backGroundThread`不会被调用，这是什么原因呢？

这是因为，在调用`performSelector:onThread: withObject: waitUntilDone`的时候，系统会给我们创建一个Timer的**source**，加到对应的RunLoop上去，然而这个时候我们没有`RunLoop`,如果我们加上RunLoop:

```
 - (void)tryPerformSelectorOnBackGroundThread{

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop run];

});
}

```

这时就会发现我们的方法正常被调用了。那么为什么主线程中的`perfom selector`却能够正常调用呢？通过上面的例子相信你已经猜到了，主线程的RunLoop是一直存在的，所以我们在主线程中执行的时候，无需再添加RunLoop。从Apple的文档中我们也可以得到验证：

> Each request to perform a selector is queued on the target thread’s run loop and the requests are then processed sequentially in the order in which they were received. 每个执行perform selector的请求都以队列的形式被放到目标线程的run loop中。然后目标线程会根据进入run loop的顺序来一一执行。

**小结:当perform selector在后台线程中执行的时候，这个线程必须有一个开启的runLoop**

## 四、一直"活着"的后台线程

现在有这样一个需求，每点击一下屏幕，让子线程做一个任务,然后大家一般会想到这样的方式:

```
@interface ViewController ()

@property(nonatomic,strong) NSThread *myThread;

@end

@implementation ViewController

 - (void)alwaysLiveBackGoundThread{

NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(myThreadRun) object:@"etund"];
self.myThread = thread;
[self.myThread start];

}
- (void)myThreadRun{

NSLog(@"my thread run");

}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    NSLog(@"%@",self.myThread);
    [self performSelector:@selector(doBackGroundThreadWork) onThread:self.myThread withObject:nil waitUntilDone:NO];
}
- (void)doBackGroundThreadWork{

    NSLog(@"do some work %s",__FUNCTION__);

}
@end

```

这个方法中，我们利用一个强引用来获取了后台线程中的thread,然后在点击屏幕的时候，在这个线程上执行`doBackGroundThreadWork`这个方法，此时我们可以看到，在`touchesBegin`方法中，self.myThread是存在的，但是这是为是什么呢？这就要从线程的五大状态来说明了:**新建状态、就绪状态、运行状态、阻塞状态、死亡状态**，这个时候尽管内存中还有线程，但是这个线程在执行完任务之后已经死亡了，经过上面的论述，我们应该怎样处理呢？我们可以给这个线程的RunLoop添加一个source，那么这个线程就会检测这个source等待执行，而不至于死亡(*有工作的强烈愿望而不死亡*):

```
 - (void)myThreadRun{

 [[NSRunLoop currentRunLoop] addPort:[[NSPort alloc] init] forMode:NSDefaultRunLoopMode]; 
 [[NSRunLoop currentRunLoop] run]

  NSLog(@"my thread run");

}

```

这个时候再次点击屏幕，我们就会发现，后台线程中执行的任务可以正常进行了。

**小结:正常情况下，后台线程执行完任务之后就处于死亡状态，我们要避免这种情况的发生可以利用RunLoop，并且给它一个Source这样来保证线程依旧还在**

## 五、深入理解NSTimer

我们平时使用NSTimer，一般是在主线程中的，代码大多如下:

```
 - (void)tryTimerOnMainThread{

NSTimer *myTimer = [NSTimer scheduledTimerWithTimeInterval:0.5 target:self       
    selector:@selector(timerAction) userInfo:nil repeats:YES];

[myTimer fire];

}

- (void)timerAction{

NSLog(@"timer action");

}

```

这个时候代码按照我们预定的结果运行，如果我们把这个Tiemr放到后台线程中呢?

```
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    NSTimer *myTimer = [NSTimer scheduledTimerWithTimeInterval:0.5 target:self selector:@selector(timerAction) userInfo:nil repeats:YES];

    [myTimer fire];

});

```

这个时候我们会发现，这个timer只执行了一次，就停止了。这是为什么呢？通过上面的讲解，想必你已经知道了，**NSTimer,只有注册到RunLoop之后才会生效，这个注册是由系统自动给我们完成的**,既然需要注册到RunLoop,那么我们就需要有一个`RunLoop`,我们在后台线程中加入如下的代码:

```
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop run];

```

这样我们就会发现程序正常运行了。在Timer注册到RunLoop之后，RunLoop会为其重复的时间点注册好事件，比如1：10，1：20，1：30这几个时间点。有时候我们会在这个线程中执行一个耗时操作，这个时候RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer，这就造成了误差(Timer有个冗余度属性叫做`tolerance`,它标明了当前点到后，容许有多少最大误差)，可以在执行一段循环之后调用一个耗时操作，很容易看到timer会有很大的误差，这说明在线程很闲的时候使用NSTiemr是比较傲你准确的，当线程很忙碌时候会有较大的误差。系统还有一个`CADisplayLink`,也可以实现定时效果，它是一个和屏幕的刷新率一样的定时器。如果在两次屏幕刷新之间执行一个耗时的任务，那其中就会有一个帧被跳过去，造成界面卡顿。另外GCD也可以实现定时器的效果，由于其和RunLoop没有关联，所以有时候使用它会更加的准确，这在[最后会给予说明](#anchor8)。

## 六、让两个后台线程有依赖性的一种方式

给两个后台线程添加依赖可能有很多的方式，这里说明一种利用`RunLoop`实现的方式。原理很简单，我们先让一个线程工作，当工作完成之后唤醒另外的一线程,通过上面对`RunLoop`的说明，相信大家很容易能够理解这些代码:

```
- (void)runLoopAddDependance{

self.runLoopThreadDidFinishFlag = NO;
NSLog(@"Start a New Run Loop Thread");
NSThread *runLoopThread = [[NSThread alloc] initWithTarget:self selector:@selector(handleRunLoopThreadTask) object:nil];
[runLoopThread start];

NSLog(@"Exit handleRunLoopThreadButtonTouchUpInside");
dispatch_async(dispatch_get_global_queue(0, 0), ^{

    while (!_runLoopThreadDidFinishFlag) {

        self.myThread = [NSThread currentThread];
        NSLog(@"Begin RunLoop");
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        NSPort *myPort = [NSPort port];
        [runLoop addPort:myPort forMode:NSDefaultRunLoopMode];
        [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
        NSLog(@"End RunLoop");
        [self.myThread cancel];
        self.myThread = nil;

    }
});

 }
- (void)handleRunLoopThreadTask
{
NSLog(@"Enter Run Loop Thread");
for (NSInteger i = 0; i < 5; i ++) {
    NSLog(@"In Run Loop Thread, count = %ld", i);
    sleep(1);
}
#if 0
// 错误示范
_runLoopThreadDidFinishFlag = YES;
// 这个时候并不能执行线程完成之后的任务，因为Run Loop所在的线程并不知道runLoopThreadDidFinishFlag被重新赋值。Run Loop这个时候没有被任务事件源唤醒。
// 正确的做法是使用 "selector"方法唤醒Run Loop。 即如下:
#endif
NSLog(@"Exit Normal Thread");
[self performSelector:@selector(tryOnMyThread) onThread:self.myThread withObject:nil waitUntilDone:NO];

// NSLog(@"Exit Run Loop Thread");
}

```

## 七、NSURLConnection的执行过程

在使用NSURLConnection时，我们会传入一个Delegate,当我们调用了`[connection start]`之后，这个Delegate会不停的收到事件的回调。实际上，**start这个函数的内部会获取CurrentRunloop**，然后在其中的DefaultMode中添加4个source。如下图所示，CFMultiplexerSource是负责各种Delegate回调的，CFHTTPCookieStorage是处理各种Cookie的。如下图所示:

![NSURLConnection的执行过程.png](http://upload-images.jianshu.io/upload_images/1464492-af830d8db3614a62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从中可以看出，当开始网络传输是，我们可以看到NSURLConnection创建了两个新的线程:com.apple.NSURLConnectionLoader和com.apple.CFSocket.private。其中CFSocket是处理底层socket链接的。NSURLConnectionLoader这个线程内部会使用RunLoop来接收底层socket的事件，并通过之前添加的source，来通知(`唤醒`)上层的Delegate。这样我们就可以理解我们平时封装网络请求时候常见的下面逻辑了:

```
    while (!_isEndRequest)
{
    NSLog(@"entered run loop");
    [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}

NSLog(@"main finished，task be removed");

- (void)connectionDidFinishLoading:(NSURLConnection *)connection
 {

  _isEndRequest = YES;

 } 

```

这里我们就可以解决下面这些疑问了:

1.  为什么这个While循环不停的执行，还需要使用一个RunLoop? 程序执行一个while循环是不会耗费很大性能的，我们这里的目的是想让子线程在有任务的时候处理任务，没有任务的时候休眠，来节约CPU的开支。

2.  如果没有为RunLoop添加item,那么它就会立即退出，这里的item呢? 其实系统已经给我们默认添加了4个source了。

3.  既然`[[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];`让线程在这里停下来，那么为什么这个循环会持续的执行呢？因为这个一直在处理任务，并且接受系统对这个Delegate的回调，也就是这个回调**唤醒**了这个线程，让它在这里循环。

## 八、AFNetWorking中是如何使用RunLoop的?

在AFN中AFURLConnectionOperation是基于NSURLConnection构建的，其希望能够在后台线程来接收Delegate的回调。

为此AFN创建了一个线程,然后在里面开启了一个RunLoop，然后添加`item`

```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
@autoreleasepool {
    [[NSThread currentThread] setName:@"AFNetworking"];
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    [runLoop run];
}

}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{ 
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}

```

这里这个`NSMachPort`的作用和上文中的一样，就是让线程不至于在很快死亡，然后RunLoop不至于退出(如果要使用这个MachPort的话，调用者需要持有这个NSMachPort，然后在外部线程通过这个port发送信息到这个loop内部,它这里没有这么做)。然后和上面的做法相似，在需要后台执行这个任务的时候，会通过调用:`[NSObject performSelector:onThread:..]`来将这个任务扔给后台线程的RunLoop中来执行。

```
- (void)start {
[self.lock lock];
if ([self isCancelled]) {
    [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
} else if ([self isReady]) {
    self.state = AFOperationExecutingState;
    [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
}
[self.lock unlock];
}

```

## GCD定时器的实现

```
 - (void)gcdTimer{

// get the queue
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

// creat timer
self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
// config the timer (starting time，interval)
// set begining time
dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
// set the interval
uint64_t interver = (uint64_t)(1.0 * NSEC_PER_SEC);

dispatch_source_set_timer(self.timer, start, interver, 0.0);

dispatch_source_set_event_handler(self.timer, ^{

    // the tarsk needed to be processed async
    dispatch_async(dispatch_get_global_queue(0, 0), ^{

        for (int i = 0; i < 100000; i++) {

            NSLog(@"gcdTimer");

        }

    });

});

dispatch_resume(self.timer);

}

```

## 九、延伸阅读

1.  [http://chun.tips/blog/2014/10/20/zou-jin-run-loopde-shi-jie-%5B%3F%5D-:shi-yao-shi-run-loop%3F/](https://link.jianshu.com?t=http://chun.tips/blog/2014/10/20/zou-jin-run-loopde-shi-jie-%5B%3F%5D-:shi-yao-shi-run-loop%3F/)

2.  [https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1](https://link.jianshu.com?t=https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1)

3.  [http://www.cocoachina.com/ios/20150601/11970.html](https://link.jianshu.com?t=http://www.cocoachina.com/ios/20150601/11970.html)

4.  [http://www.jianshu.com/p/de2716807570](https://www.jianshu.com/p/de2716807570)

5.  [http://blog.csdn.net/enuola/article/details/9163051](https://link.jianshu.com?t=http://blog.csdn.net/enuola/article/details/9163051)
