# 概述
RunLoop作为iOS中一个基础组件和线程有着千丝万缕的关系，同时也是很多常见技术的幕后功臣。尽管在平时多数开发者很少直接使用RunLoop，但是理解RunLoop可以帮助开发者更好的利用多线程编程模型，同时也可以帮助开发者解答日常开发中的一些疑惑。本文将从RunLoop源码着手，结合RunLoop的实际应用来逐步解开它的神秘面纱。

# 开源的RunloopRef
通常所说的RunLoop指的是NSRunloop或者CFRunloopRef，CFRunloopRef是纯C的函数，而NSRunloop仅仅是CFRunloopRef的OC封装，并未提供额外的其他功能，因此下面主要分析CFRunloopRef，苹果已经开源了CoreFoundation源代码，因此很容易找到CFRunloop源代码。
从代码可以看出CFRunloopRef其实就是**__CFRunloop这个结构体指针（按照OC的思路我们可以将RunLoop看成一个对象），这个对象的运行才是我们通常意义上说的运行循环，核心方法是__CFRunloopRun()**，为了便于阅读就不再直接贴源代码，放一段伪代码方便大家阅读：
```
int32_t __CFRunLoopRun()
{
    // 通知即将进入runloop
    __CFRunLoopDoObservers(KCFRunLoopEntry);
    
    do
    {
        // 通知将要处理timer和source
        __CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
        __CFRunLoopDoObservers(kCFRunLoopBeforeSources);
        
        // 处理非延迟的主线程调用
        __CFRunLoopDoBlocks();
        // 处理Source0事件
        __CFRunLoopDoSource0();
        
        if (sourceHandledThisLoop) {
            __CFRunLoopDoBlocks();
         }
        /// 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
        if (__Source0DidDispatchPortLastTime) {
            Boolean hasMsg = __CFRunLoopServiceMachPort();
            if (hasMsg) goto handle_msg;
        }
            
        /// 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
        if (!sourceHandledThisLoop) {
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
        }
            
        // GCD dispatch main queue
        CheckIfExistMessagesInMainDispatchQueue();
        
        // 即将进入休眠
        __CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);
        
        // 等待内核mach_msg事件
        mach_port_t wakeUpPort = SleepAndWaitForWakingUpPorts();
        
        // 等待。。。
        
        // 从等待中醒来
        __CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
        
        // 处理因timer的唤醒
        if (wakeUpPort == timerPort)
            __CFRunLoopDoTimers();
        
        // 处理异步方法唤醒,如dispatch_async
        else if (wakeUpPort == mainDispatchQueuePort)
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()
            
        // 处理Source1
        else
            __CFRunLoopDoSource1();
        
        // 再次确保是否有同步的方法需要调用
        __CFRunLoopDoBlocks();
        
    } while (!stop && !timeout);
    
    // 通知即将退出runloop
    __CFRunLoopDoObservers(CFRunLoopExit);
}
```
源代码尽管不算太长，但是如果不太熟悉的话面对这么一堆不知道做什么的函数调用还是会给人一种神秘感。但是现在可以不用逐行阅读，后面慢慢解开这层神秘面纱。现在只要了解上面的伪代码知道核心的方法**__CFRunLoopRun**内部其实是一个_do while_循环，这也正是Runloop运行的本质。执行了这个函数以后就一直处于“等待-处理”的循环之中，直到循环结束。只是不同于我们自己写的循环它在休眠时几乎不会占用系统资源，当然这是由于系统内核负责实现的，也是Runloop精华所在。

>随着Swift的开源苹果也维护了一个Swift版本的跨平台CoreFoundation版本，除了mac平台它还是适配了Linux和Windows平台。但是鉴于目前很多关于Runloop的讨论都是以OC版展开的，所以这里也主要分析OC版本。

下图描述了Runloop运行流程（基本描述了上面Runloop的核心流程，当然可以查看官方[The Run Loop Sequence of Events](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23)描述）：

![](http://upload-images.jianshu.io/upload_images/1464492-3e9619940d7b7815.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个流程并不复杂（需要注意的就是_黄色_区域的消息处理中并不包含source0，因为它在循环开始之初就会处理），整个流程其实就是一种Event Loop的实现，其他平台均有类似的实现，只是这里叫做Runloop。但是既然RunLoop是一个消息循环，谁来管理和运行Runloop？那么它接收什么类型的消息？休眠过程是怎么样的？如何保证休眠时不占用系统资源？如何处理这些消息以及何时退出循环？还有一系列问题需要解开。

>注意的是尽管CFRunLoopPerformBlock在上图中作为唤醒机制有所体现，但事实上执行CFRunLoopPerformBlock只是入队，下次RunLoop运行才会执行，而如果需要立即执行则必须调用CFRunLoopWakeUp。

Runloop Mode
从源码很容易看出，Runloop总是运行在某种特定的CFRunLoopModeRef下（每次运行**__CFRunLoopRun()函数时必须指定Mode）。而通过CFRunloopRef对应结构体的定义可以很容易知道每种Runloop都可以包含若干个Mode，每个Mode又包含Source/Timer/Observer。每次调用Runloop的主函数__CFRunLoopRun()时必须指定一种Mode，这个Mode称为 _currentMode**，当切换Mode时必须退出当前Mode，然后重新进入Runloop以保证不同Mode的Source/Timer/Observer互不影响。
```
 struct __CFRunLoop {
        CFRuntimeBase _base;
        pthread_mutex_t _lock;          /* locked for accessing mode list */
        __CFPort _wakeUpPort;           // used for CFRunLoopWakeUp 
        Boolean _unused;
        volatile _per_run_data *_perRunData;              // reset for runs of the run loop
        pthread_t _pthread;
        uint32_t _winthread;
        CFMutableSetRef _commonModes;
        CFMutableSetRef _commonModeItems;
        CFRunLoopModeRef _currentMode;
        CFMutableSetRef _modes;
        struct _block_item *_blocks_head;
        struct _block_item *_blocks_tail;
        CFAbsoluteTime _runTime;
        CFAbsoluteTime _sleepTime;
        CFTypeRef _counterpart;
    };
    
    struct __CFRunLoopMode {
        CFRuntimeBase _base;
        pthread_mutex_t _lock;  /* must have the run loop locked before locking this */
        CFStringRef _name;
        Boolean _stopped;
        char _padding[3];
        CFMutableSetRef _sources0;
        CFMutableSetRef _sources1;
        CFMutableArrayRef _observers;
        CFMutableArrayRef _timers;
        CFMutableDictionaryRef _portToV1SourceMap;
        __CFPortSet _portSet;
        CFIndex _observerMask;
    #if USE_DISPATCH_SOURCE_FOR_TIMERS
        dispatch_source_t _timerSource;
        dispatch_queue_t _queue;
        Boolean _timerFired; // set to true by the source when a timer has fired
        Boolean _dispatchTimerArmed;
    #endif
    #if USE_MK_TIMER_TOO
        mach_port_t _timerPort;
        Boolean _mkTimerArmed;
    #endif
    #if DEPLOYMENT_TARGET_WINDOWS
        DWORD _msgQMask;
        void (*_msgPump)(void);
    #endif
        uint64_t _timerSoftDeadline; /* TSR */
        uint64_t _timerHardDeadline; /* TSR */
    };
```
系统默认提供的Run Loop Modes有kCFRunLoopDefaultMode(NSDefaultRunLoopMode)和UITrackingRunLoopMode，需要切换到对应的Mode时只需要传入对应的名称即可。前者是系统默认的Runloop Mode，例如进入iOS程序默认不做任何操作就处于这种Mode中，此时滑动UIScrollView，主线程就切换Runloop到到UITrackingRunLoopMode，不再接受其他事件操作（除非你将其他Source/Timer设置到UITrackingRunLoopMode下）。
但是对于开发者而言经常用到的Mode还有一个kCFRunLoopCommonModes（NSRunLoopCommonModes）,其实这个并不是某种具体的Mode，而是一种模式组合，在iOS系统中默认包含了
 NSDefaultRunLoopMode和 UITrackingRunLoopMode（注意：并不是说Runloop会运行在kCFRunLoopCommonModes这种模式下，而是相当于分别注册了 NSDefaultRunLoopMode和 UITrackingRunLoopMode。当然你也可以通过调用CFRunLoopAddCommonMode()方法将自定义Mode放到 kCFRunLoopCommonModes组合）。

>注意：我们常常还会碰到一些系统框架自定义Mode，例如Foundation中NSConnectionReplyMode。还有一些系统私有Mode，例如：GSEventReceiveRunLoopMode接受系统事件，UIInitializationRunLoopMode App启动过程中初始化Mode。更多系统或框架Mode查看这里

CFRunLoopRef和CFRunloopMode、CFRunLoopSourceRef/CFRunloopTimerRef/CFRunLoopObserverRef关系如下图：

![](http://upload-images.jianshu.io/upload_images/1464492-c5737b448ba869ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么CFRunLoopSourceRef、CFRunLoopTimerRef和CFRunLoopObserverRef究竟是什么？它们在Runloop运行流程中起到什么作用呢？
# Source
首先看一下官方Runloop结构图（注意下图的Input Source Port和前面流程图中的Source0并不对应，而是对应Source1。Source1和Timer都属于端口事件源，不同的是所有的Timer都共用一个端口“Mode Timer Port”，而每个Source1都有不同的对应端口）：

![](http://upload-images.jianshu.io/upload_images/1464492-58747d97ff3b446c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再结合前面RunLoop核心运行流程可以看出Source0(负责App内部事件，由App负责管理触发，例如UITouch事件)和Timer（又叫Timer Source，基于时间的触发器，上层对应NSTimer）是两个不同的Runloop事件源（当然Source0是Input Source中的一类，Input Source还包括Custom Input Source，由其他线程手动发出），RunLoop被这些事件唤醒之后就会处理并调用事件处理方法（CFRunLoopTimerRef的回调指针和CFRunLoopSourceRef均包含对应的回调指针）。
但是对于CFRunLoopSourceRef除了Source0之外还有另一个版本就是Source1，Source1除了包含回调指针外包含一个mach port，和Source0需要手动触发不同，Source1可以监听系统端口和其他线程相互发送消息，它能够主动唤醒RunLoop(由操作系统内核进行管理，例如CFMessagePort消息)。官方也指出可以自定义Source，因此对于CFRunLoopSourceRef来说它更像一种协议，框架已经默认定义了两种实现，如果有必要开发人员也可以自定义，详细情况可以查看[官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)。

# Observer
```
    struct __CFRunLoopObserver {
        CFRuntimeBase _base;
        pthread_mutex_t _lock;
        CFRunLoopRef _runLoop;
        CFIndex _rlCount;
        CFOptionFlags _activities;      /* immutable */
        CFIndex _order;         /* immutable */
        CFRunLoopObserverCallBack _callout; /* immutable */
        CFRunLoopObserverContext _context;  /* immutable, except invalidation */
    };
```
相对来说CFRunloopObserverRef理解起来并不复杂，它相当于消息循环中的一个监听器，随时通知外部当前RunLoop的运行状态（它包含一个函数指针_callout_将当前状态及时告诉观察者）。具体的Observer状态如下：
```
    /* Run Loop Observer Activities */
    typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
        kCFRunLoopEntry = (1UL << 0), // 进入RunLoop 
        kCFRunLoopBeforeTimers = (1UL << 1), // 即将开始Timer处理
        kCFRunLoopBeforeSources = (1UL << 2), // 即将开始Source处理
        kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
        kCFRunLoopAfterWaiting = (1UL << 6), //从休眠状态唤醒
        kCFRunLoopExit = (1UL << 7), //退出RunLoop
        kCFRunLoopAllActivities = 0x0FFFFFFFU
    };
```
# Call out
在开发过程中几乎所有的操作都是通过Call out进行回调的(无论是Observer的状态通知还是Timer、Source的处理)，而系统在回调时通常使用如下几个函数进行回调(换句话说你的代码其实最终都是通过下面几个函数来负责调用的，即使你自己监听Observer也会先调用下面的函数然后间接通知你，所以在调用堆栈中经常看到这些函数)：
```
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__();
    static void __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__();
```
例如在控制器的touchBegin中打入断点查看堆栈（由于UIEvent是Source0，所以可以看到一个Source0的Call out函数CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION调用）：

![](http://upload-images.jianshu.io/upload_images/1464492-2c94911c09b131fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# RunLoop休眠
其实对于Event Loop而言RunLoop最核心的事情就是保证线程在没有消息时休眠以避免占用系统资源，有消息时能够及时唤醒。RunLoop的这个机制完全依靠系统内核来完成，具体来说是苹果操作系统核心组件Darwin中的Mach来完成的（[Darwin](https://opensource.apple.com)是开源的）。可以从下图最底层Kernel中找到Mach：

![](http://upload-images.jianshu.io/upload_images/1464492-5e7f60ca3b3f596b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Mach是Darwin的核心，可以说是内核的核心，提供了进程间通信（IPC）、处理器调度等基础服务。在Mach中，进程、线程间的通信是以消息的方式来完成的，消息在两个Port之间进行传递（这也正是Source1之所以称之为Port-based Source的原因，因为它就是依靠系统发送消息到指定的Port来触发的）。消息的发送和接收使用<mach/message.h>中的mach_msg()函数（事实上苹果提供的Mach API很少，并不鼓励我们直接调用这些API）：
```
    /*
     *  Routine:    mach_msg
     *  Purpose:
     *      Send and/or receive a message.  If the message operation
     *      is interrupted, and the user did not request an indication
     *      of that fact, then restart the appropriate parts of the
     *      operation silently (trap version does not restart).
     */
    __WATCHOS_PROHIBITED __TVOS_PROHIBITED
    extern mach_msg_return_t    mach_msg(
                        mach_msg_header_t *msg,
                        mach_msg_option_t option,
                        mach_msg_size_t send_size,
                        mach_msg_size_t rcv_size,
                        mach_port_name_t rcv_name,
                        mach_msg_timeout_t timeout,
                        mach_port_name_t notify);
 ```                       
而mach_msg()的本质是一个调用mach_msg_trap(),这相当于一个系统调用，会触发内核状态切换。当程序静止时，RunLoop停留在**__CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy),而这个函数内部就是调用了mach_msg**让程序处于休眠状态。

# Runloop和线程的关系
Runloop是基于pthread进行管理的，pthread是基于c的跨平台多线程操作底层API。它是mach thread的上层封装（可以参见[Kernel Programming Guide](https://developer.apple.com/library/content/documentation/Darwin/Conceptual/KernelProgramming/Mach/Mach.html)），和NSThread一一对应（而NSThread是一套面向对象的API，所以在iOS开发中我们也几乎不用直接使用pthread）。

![](http://upload-images.jianshu.io/upload_images/1464492-a5a45864fc59c202.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
苹果开发的接口中并没有直接创建Runloop的接口，如果需要使用Runloop通常CFRunLoopGetMain()和CFRunLoopGetCurrent()两个方法来获取（通过上面的源代码也可以看到，核心逻辑在_CFRunLoopGet_当中）,通过代码并不难发现其实只有当我们使用线程的方法主动get Runloop时才会在第一次创建该线程的Runloop，同时将它保存在全局的Dictionary中（线程和Runloop二者一一对应），默认情况下线程并不会创建Runloop（主线程的Runloop比较特殊，任何线程创建之前都会保证主线程已经存在Runloop），同时在线程结束的时候也会销毁对应的Runloop。

iOS开发过程中对于开发者而言更多的使用的是NSRunloop,它默认提供了三个常用的run方法：
```
     - (void)run; 
     - (BOOL)runMode:(NSRunLoopMode)mode beforeDate:(NSDate *)limitDate;
     - (void)runUntilDate:(NSDate *)limitDate;
```
run方法对应上面CFRunloopRef中的CFRunLoopRun并不会退出，除非调用CFRunLoopStop();通常如果想要永远不会退出RunLoop才会使用此方法，否则可以使用runUntilDate。
runMode:beforeDate:则对应CFRunLoopRunInMode(mode,limiteDate,true)方法,只执行一次，执行完就退出；通常用于手动控制RunLoop（例如在while循环中）。
runUntilDate:方法其实是CFRunLoopRunInMode(kCFRunLoopDefaultMode,limiteDate,false)，执行完并不会退出，继续下一次RunLoop直到timeout。
# RunLoop应用
## NSTimer
前面一直提到Timer Source作为事件源，事实上它的上层对应就是NSTimer（其实就是CFRunloopTimerRef）这个开发者经常用到的定时器（底层基于使用mk_timer实现），甚至很多开发者接触RunLoop还是从NSTimer开始的。其实NSTimer定时器的触发正是基于RunLoop运行的，所以使用NSTimer之前必须注册到RunLoop，但是RunLoop为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行（NSTimer提供了一个tolerance属性用于设置宽容度，如果确实想要使用NSTimer并且希望尽可能的准确，则可以设置此属性）。

NSTimer的创建通常有两种方式，尽管都是类方法，一种是timerWithXXX，另一种scheduedTimerWithXXX。
```
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block ;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block ;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
二者最大的区别就是后者除了创建一个定时器外会自动以NSDefaultRunLoopModeMode添加到当前线程RunLoop中，不添加到RunLoop中的NSTimer是无法正常工作的。例如下面的代码中如果timer2不加入到RunLoop中是无法正常工作的。同时注意如果滚动UIScrollView（UITableView、UICollectionview是类似的）二者是无法正常工作的，但是如果将NSDefaultRunLoopMode改为NSRunLoopCommonModes则可以正常工作，这也解释了前面介绍的Mode内容。
```
    #import "ViewController1.h"

    @interface ViewController1 ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,weak) NSTimer *timer2;
    @end
    
    @implementation ViewController1
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor blueColor];
        // timer1创建后会自动以NSDefaultRunLoopMode默认模式添加到当前RunLoop中，所以可以正常工作
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timeInterval:) userInfo:nil repeats:YES];
        NSTimer *tempTimer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timeInterval:) userInfo:nil repeats:YES];
        // 如果不把timer2添加到RunLoop中是无法正常工作的(注意如果想要在滚动UIScrollView时timer2可以正常工作可以将NSDefaultRunLoopMode改为NSRunLoopCommonModes)
        [[NSRunLoop currentRunLoop] addTimer:tempTimer forMode:NSDefaultRunLoopMode];
        self.timer2 = tempTimer;
        
        CGRect rect = [UIScreen mainScreen].bounds;
        UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:CGRectInset(rect, 0, 200)];
        [self.view addSubview:scrollView];
        
        UIView *contentView = [[UIView alloc] initWithFrame:CGRectInset(scrollView.bounds, -100, -100)];
        contentView.backgroundColor = [UIColor redColor];
        [scrollView addSubview:contentView];
        scrollView.contentSize = contentView.frame.size;
    }
    
    - (void)timeInterval:(NSTimer *)timer {
        if (self.timer1 == timer) {
            NSLog(@"timer1...");
        } else {
            NSLog(@"timer2...");
        }
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self dismissViewControllerAnimated:true completion:nil];
    }
    
    - (void)dealloc {
        NSLog(@"ViewController1 dealloc...");
    }
    @end
```
注意上面代码中UIViewController1对timer1和timer2并没有强引用，对于普通的对象而言，执行完viewDidLoad方法之后（准确的说应该是执行完viewDidLoad方法后的的一个RunLoop运行结束）二者应该会被释放，但事实上二者并没有被释放。原因是：为了确保定时器正常运转，当加入到RunLoop以后系统会对NSTimer执行一次retain操作（特别注意：timer2创建时并没直接赋值给timer2，原因是timer2是weak属性，如果直接赋值给timer2会被立即释放，因为timerWithXXX方法创建的NSTimer默认并没有加入RunLoop，只有后面加入RunLoop以后才可以将引用指向timer2）。
但是即使使用了弱引用，上面的代码中ViewController1也无法正常释放，原因是在创建NSTimer2时指定了target为self，这样一来造成了timer1和timer2对ViewController1有一个强引用。解决这个问题的方法通常有两种：一种是将target分离出来独立成一个对象（在这个对象中创建NSTimer并将对象本身作为NSTimer的target），控制器通过这个对象间接使用NSTimer；另一种方式的思路仍然是转移target，只是可以直接增加NSTimer扩展（分类），让NSTimer自身做为target，同时可以将操作selector封装到block中。后者相对优雅，也是目前使用较多的方案（目前有大量类似的封装，例如：[NSTimer+Block](https://github.com/mBrissman/NSTimer-Block)）。显然Apple也认识到了这个问题，如果你可以确保代码只在iOS 10下运行就可以使用iOS 10新增的系统级block方案（上面的代码中已经贴出这种方法）。
当然使用上面第二种方法可以解决控制器无法释放的问题，但是会发现即使控制器被释放了两个定时器仍然正常运行，要解决这个问题就需要调用NSTimer的invalidate方法（注意：无论是重复执行的定时器还是一次性的定时器只要调用invalidate方法则会变得无效，只是一次性的定时器执行完操作后会自动调用invalidate方法）。修改后的代码如下：
```
    #import "ViewController1.h"
    
    @interface ViewController1 ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,weak) NSTimer *timer2;
    @end
    
    @implementation ViewController1
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor blueColor];
    
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            NSLog(@"timer1...");
        }];
        NSTimer *tempTimer = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            NSLog(@"timer2...");
        }];
        [[NSRunLoop currentRunLoop] addTimer:tempTimer forMode:NSDefaultRunLoopMode];
        self.timer2 = tempTimer;
        
        CGRect rect = [UIScreen mainScreen].bounds;
        UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:CGRectInset(rect, 0, 200)];
        [self.view addSubview:scrollView];
        
        UIView *contentView = [[UIView alloc] initWithFrame:CGRectInset(scrollView.bounds, -100, -100)];
        contentView.backgroundColor = [UIColor redColor];
        [scrollView addSubview:contentView];
        scrollView.contentSize = contentView.frame.size;
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self dismissViewControllerAnimated:true completion:nil];
    }
    
    - (void)dealloc {
        [self.timer1 invalidate];
        [self.timer2 invalidate];
        NSLog(@"ViewController1 dealloc...");
    }
    @end
```
其实和定时器相关的另一个问题大家也经常碰到，那就是NSTimer不是一种实时机制，[官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Timers/Articles/timerConcepts.html)明确说明在一个循环中如果RunLoop没有被识别（这个时间大概在50-100ms）或者说当前RunLoop在执行一个长的call out（例如执行某个循环操作）则NSTimer可能就会存在误差，RunLoop在下一次循环中继续检查并根据情况确定是否执行（NSTimer的执行时间总是固定在一定的时间间隔，例如1:00:00、1:00:01、1:00:02、1:00:05则跳过了第4、5次运行循环）。
要演示这个问题请看下面的例子（注意：有些示例中可能会让一个线程中启动一个定时器，再在主线程启动一个耗时任务来演示这个问，如果实际测试可能效果不会太明显，因为现在的iPhone都是多核运算的，这样一来这个问题会变得相对复杂，因此下面的例子选择在同一个RunLoop中即加入定时器和执行耗时任务）
```
 #import "ViewController.h"
    
    @interface ViewController ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,strong) NSThread *thread1;
    @end
    
    @implementation ViewController
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor redColor];
        
        // 由于下面的方法无法拿到NSThread的引用，也就无法控制线程的状态
        //[NSThread detachNewThreadSelector:@selector(performTask) toTarget:self withObject:nil];
        self.thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(performTask) object:nil];
        [self.thread1 start];
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self.thread1 cancel];
        [self dismissViewControllerAnimated:YES completion:nil];
    }
    
    - (void)dealloc {
        [self.timer1 invalidate];
        NSLog(@"ViewController dealloc.");
    }
    
    - (void)performTask {
        // 使用下面的方式创建定时器虽然会自动加入到当前线程的RunLoop中，但是除了主线程外其他线程的RunLoop默认是不会运行的，必须手动调用
        __weak typeof(self) weakSelf = self;
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            if ([NSThread currentThread].isCancelled) {
                //[NSObject cancelPreviousPerformRequestsWithTarget:weakSelf selector:@selector(caculate) object:nil];
                //[NSThread exit];
                [weakSelf.timer1 invalidate];
            }
            NSLog(@"timer1...");
        }];
        
        NSLog(@"runloop before performSelector:%@",[NSRunLoop currentRunLoop]);
        
        // 区分直接调用和「performSelector:withObject:afterDelay:」区别,下面的直接调用无论是否运行RunLoop一样可以执行，但是后者则不行。
        //[self caculate];
        [self performSelector:@selector(caculate) withObject:nil afterDelay:2.0];
    
        // 取消当前RunLoop中注册测selector（注意：只是当前RunLoop，所以也只能在当前RunLoop中取消）
        // [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(caculate) object:nil];
        NSLog(@"runloop after performSelector:%@",[NSRunLoop currentRunLoop]);
        
        // 非主线程RunLoop必须手动调用
        [[NSRunLoop currentRunLoop] run];
        
        NSLog(@"注意：如果RunLoop不退出（运行中），这里的代码并不会执行，RunLoop本身就是一个循环.");
        
        
    }
    
    - (void)caculate {
        for (int i = 0;i < 9999;++i) {
            NSLog(@"%i,%@",i,[NSThread currentThread]);
            if ([NSThread currentThread].isCancelled) {
                return;
            }
        }
    }
    
    @end
```
如果运行并且不退出上面的程序会发现，前两秒NSTimer可以正常执行，但是两秒后由于同一个RunLoop中循环操作的执行造成定时器跳过了中间执行的机会一直到caculator循环完毕，这也正说明了NSTimer不是实时系统机制的原因。

但是以上程序还有几点需要说明一下：

1. NSTimer会对Target进行强引用直到任务结束或exit之后才会释放。如果上面的程序没有进行线程cancel而终止任务则及时关闭控制器也无法正确释放。
2. 非主线程的RunLoop并不会自动运行（同时注意默认情况下非主线程的RunLoop并不会自动创建，直到第一次使用），RunLoop运行必须要在加入NSTimer或Source0、Sourc1、Observer输入后运行否则会直接退出。例如上面代码如果run放到NSTimer创建之前则既不会执行定时任务也不会执行循环运算。
3. performSelector:withObject:afterDelay:执行的本质还是通过创建一个NSTimer然后加入到当前线程RunLoop（通而过前后两次打印RunLoop信息可以看到此方法执行之后RunLoop的timer会增加1个。类似的还有performSelector:onThread:withObject:afterDelay:，只是它会在另一个线程的RunLoop中创建一个Timer），所以此方法事实上在任务执行完之前会对触发对象形成引用，任务执行完进行释放（例如上面会对ViewController形成引用，注意：performSelector: withObject:等方法则等同于直接调用，原理与此不同）。
4. 同时上面的代码也充分说明了RunLoop是一个循环事实，run方法之后的代码不会立即执行，直到RunLoop退出。
5. 上面程序的运行过程中如果突然dismiss，则程序的实际执行过程要分为两种情况考虑：如果循环任务caculate还没有开始则会在timer1中停止timer1运行（停止了线程中第一个任务），然后等待caculate执行并break（停止线程中第二个任务）后线程任务执行结束释放对控制器的引用；如果循环任务caculate执行过程中dismiss则caculate任务执行结束，等待timer1下个周期运行（因为当前线程的RunLoop并没有退出，timer1引用计数器并不为0）时检测到线程取消状态则执行invalidate方法（第二个任务也结束了），此时线程释放对于控制器的引用。

>CADisplayLink是一个执行频率（fps）和屏幕刷新相同（可以修改preferredFramesPerSecond改变刷新频率）的定时器，它也需要加入到RunLoop才能执行。与NSTimer类似，CADisplayLink同样是基于CFRunloopTimerRef实现，底层使用mk_timer（可以比较加入到RunLoop前后RunLoop中timer的变化）。和NSTimer相比它精度更高（尽管NSTimer也可以修改精度），不过和NStimer类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下CADisaplayLink用于构建帧动画，看起来相对更加流畅，而NSTimer则有更广泛的用处。


# AutoreleasePool
AutoreleasePool是另一个与RunLoop相关讨论较多的话题。其实从RunLoop源代码分析，AutoreleasePool与RunLoop并没有直接的关系，之所以将两个话题放到一起讨论最主要的原因是因为在iOS应用启动后会注册两个Observer管理和维护AutoreleasePool。不妨在应用程序刚刚启动时打印currentRunLoop可以看到系统默认注册了很多个Observer，其中有两个Observer的callout都是** _ wrapRunLoopWithAutoreleasePoolHandler**，这两个是和自动释放池相关的两个监听。

```
    <CFRunLoopObserver 0x6080001246a0 [0x101f81df0]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
    '' <CFRunLoopObserver 0x608000124420 [0x101f81df0]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
```

第一个Observer会监听RunLoop的进入，它会回调objc_autoreleasePoolPush()向当前的AutoreleasePoolPage增加一个哨兵对象标志创建自动释放池。这个Observer的order是-2147483647优先级最高，确保发生在所有回调操作之前。
第二个Observer会监听RunLoop的进入休眠和即将退出RunLoop两种状态，在即将进入休眠时会调用objc_autoreleasePoolPop() 和 objc_autoreleasePoolPush() 根据情况从最新加入的对象一直往前清理直到遇到哨兵对象。而在即将退出RunLoop时会调用objc_autoreleasePoolPop() 释放自动自动释放池内对象。这个Observer的order是2147483647，优先级最低，确保发生在所有回调操作之后。
主线程的其他操作通常均在这个AutoreleasePool之内（main函数中），以尽可能减少内存维护操作(当然你如果需要显式释放【例如循环】时可以自己创建AutoreleasePool否则一般不需要自己创建)。
其实在应用程序启动后系统还注册了其他Observer（例如即将进入休眠时执行注册回调_UIGestureRecognizerUpdateObserver用于手势处理、回调为_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv的Observer用于界面实时绘制更新）和多个Source1（例如context为CFMachPort的Source1用于接收硬件事件响应进而分发到应用程序一直到UIEvent），这里不再一一详述。

# UI更新
如果打印App启动之后的主线程RunLoop可以发现另外一个callout为**_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv**的Observer，这个监听专门负责UI变化后的更新，比如修改了frame、调整了UI层级（UIView/CALayer）或者手动设置了setNeedsDisplay/setNeedsLayout之后就会将这些操作提交到全局容器。而这个Observer监听了主线程RunLoop的即将进入休眠和退出状态，一旦进入这两种状态则会遍历所有的UI更新并提交进行实际绘制更新。
通常情况下这种方式是完美的，因为除了系统的更新，还可以利用setNeedsDisplay等方法手动触发下一次RunLoop运行的更新。但是如果当前正在执行大量的逻辑运算可能UI的更新就会比较卡，因此facebook推出了[AsyncDisplayKit](https://github.com/facebookarchive/AsyncDisplayKit)来解决这个问题。AsyncDisplayKit其实是将UI排版和绘制运算尽可能放到后台，将UI的最终更新操作放到主线程（这一步也必须在主线程完成），同时提供一套类UIView或CALayer的相关属性，尽可能保证开发者的开发习惯。这个过程中AsyncDisplayKit在主线程RunLoop中增加了一个Observer监听即将进入休眠和退出RunLoop两种状态,收到回调时遍历队列中的待处理任务一一执行。

# NSURLConnection
在前面的[网络开发](http://www.cnblogs.com/kenshincui/p/4042190.html#requestAndResponse)的文章中已经介绍过NSURLConnection的使用，一旦启动NSURLConnection以后就会不断调用delegate方法接收数据，这样一个连续的的动作正是基于RunLoop来运行。
一旦NSURLConnection设置了delegate会立即创建一个线程com.apple.NSURLConnectionLoader，同时内部启动RunLoop并在NSDefaultMode模式下添加4个Source0。其中CFHTTPCookieStorage用于处理cookie ;CFMultiplexerSource负责各种delegate回调并在回调中唤醒delegate内部的RunLoop（通常是主线程）来执行实际操作。
早期版本的AFNetworking库也是基于NSURLConnection实现，为了能够在后台接收delegate回调AFNetworking内部创建了一个空的线程并启动了RunLoop，当需要使用这个后台线程执行任务时AFNetworking通过performSelector: onThread: 将这个任务放到后台线程的RunLoop中。

# GCD和RunLoop的关系
在RunLoop的源代码中可以看到用到了GCD的相关内容，但是RunLoop本身和GCD并没有直接的关系。当调用了dispatch_async(dispatch_get_main_queue(), <#^(void)block#>)时libDispatch会向主线程RunLoop发送消息唤醒RunLoop，RunLoop从消息中获取block，并且在__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__回调里执行这个block。不过这个操作仅限于主线程，其他线程dispatch操作是全部由libDispatch驱动的。

# 更多RunLoop使用
前面看了很多RunLoop的系统应用和一些知名第三方库使用，那么除了这些究竟在实际开发过程中我们自己能不能适当的使用RunLoop帮我们做一些事情呢？

思考这个问题其实只要看RunLoopRef的包含关系就知道了，RunLoop包含多个Mode，而它的Mode又是可以自定义的，这么推断下来其实无论是Source1、Timer还是Observer开发者都可以利用，但是通常情况下不会自定义Timer，更不会自定义一个完整的Mode，利用更多的其实是Observer和Mode的切换。
例如很多人都熟悉的使用perfromSelector在默认模式下设置图片，防止UITableView滚动卡顿（[[UIImageView alloc initWithFrame:CGRectMake(0, 0, 100, 100)] performSelector:@selector(setImage:) withObject:myImage afterDelay:0.0 inModes:@NSDefaultRunLoopMode]）。还有sunnyxx的[UITableView+FDTemplateLayoutCell](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell)利用Observer在界面空闲状态下计算出UITableViewCell的高度并进行缓存。再有老谭的[PerformanceMonitor](http://storage.tanhao.me/2015/11/PerformanceMonitor.zip)关于iOS实时卡顿监控，同样是利用Observer对RunLoop进行监视。

>关于如何自定义一个Custom Input Source[官网](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW3)给出了详细的流程。

文章转载自崔江涛（KenshinCui）http://www.cnblogs.com/kenshincui/p/6823841.html#3860302。
