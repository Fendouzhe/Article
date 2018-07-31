#### 什么是RunLoop?

答:RunLoop是线程相关的基础框架中的一部分，是一个事件处理对象，每一个线程都有与之对应的RunLoop，但并不是线程创建时就有RunLoop，只有当前线程第一次主动获取RunLoop，系统才会创建当前线程相应的RunLoop。

#### RunLoop的作用是什么？

答:1).管理线程的生命周期及活动。                                           

     2).处理输入事件源以及通知观察者。

#### 如何使用RunLoop?

答:iOS/OSX提供了Core Foundation(CFRunLoopRef)和Cocoa(NSRunLoop)两套API来使用RunLoop，可以通过CFRunLoopGetMain() 和 CFRunLoopGetCurrent()或[NSRunLoop mainRunLoop]和[NSRunLoop currentRunLoop]来获取RunLoop对象，NSRunLoop是基于CFRunLoopRef的高层组件，CFRunLoopRef的API都是线程安全的，但NSRunLoop的API不是线程安全的。虽然CFRunLoopRef的操作都是线程安全的，但不建议跨线程处理。

#### 如何启动RunLoop？

答:可以通过NSRunLoop对象的run，runUntilDate:，runMode:beforeDate:方法，或通过CFRunLoopRef的CFRunLoopRun()，CFRunLoopRunInMode()来启动RunLoop。

#### 使用NSRunLoop和CFRunLoopRef来启动RunLoop有何不同？

答:虽然NSRunLoop是基于CFRunLoopRef的高层组件，对NSRunLoop的操作最终都会转换成对CFRunLoopRef的操作，但需要注意的是，NSRunLoop与CFRunLoopRef并不是简单的接口转换，就像启动RunLoop一样，NSRunLoop的Run方法与CFRunLoopRun()并不是对应着的，而且有着一定的区别。

![NSRunLoop Run](http://upload-images.jianshu.io/upload_images/1464492-7abe287fb8097a4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![CFRunLoopRun()](http://upload-images.jianshu.io/upload_images/1464492-fb1bb9e5ebc6dc65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 如何退出RunLoop？

答:退出RunLoop的方式有三种，分别是：方式一.给RunLoop设定超时时间；方式二.使用CFRunLoopStop()函数显式退出；方式三.移除CurrentMode的所有输入源和定时源。

方式一是官法推荐的方式，可以安全有效地退出RunLoop。方式二，用CFRunLoopStop()函数并不是绝对可以退出Run的，要看以什么方式启动RunLoop，如果用[[NSRunLoop currentRunLoop] run]来启动RunLoop，那么用CFRunLoopStop()是无法退出RunLoop的，正如上面的伪代码所示，以[[NSRunLoop currentRunLoop] run]启动的RunLoop，唯一退出的方式是移除CurrentMode的所有输入源和定时源，也就是方式三，但这种方式是不稳定的，因为系统会添加一些输入源或定时源来完成一些操作。

#### RunLoop是否自动运行的？

答:所有线程的RunLoop都是默认不启动的，但主线程的RunLoop会随着应用的运行而被启动。

#### RunLoop处理的输入事件源有哪些？

答:RunLoop处理的事件源有两种，分别是输入源(Input Source)和定时源(Timer Source)，而主线程的Main RunLoop还会处理GCD事件源。

#### 什么是RunLoopMode？

答:RunLoop可以有多个RunLoopMode，RunLoopMode包含了输入源(Input Source)，定时源(Timer Source)和观察者(Observer)。

![](http://upload-images.jianshu.io/upload_images/1464492-2d860447f67b6388.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


RunLoop每次进入Run时都需要指定一个RunLoopMode，指定RunLoopMode后，只有当前RunLoopMode内的源和观察者会被处理，其它的源和观察者需要等到RunLoop运行其RunLoopMode时才会被处理，切换RunLoopMode的唯一方式是，退出当前RunLoopMode，重新指定一个RunLoopMode进入Run。当RunLoop选择一个RunLoopMode进入Run时，若这个RunLoopMode中并没有需要处理的源(输入源或定时源)，RunLoop就会直接退出。

![RunLoopMode](http://upload-images.jianshu.io/upload_images/1464492-a621c48ccd99f63f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


RunLoopMode中的_timerPort是_timers中所有定时源的公共Port，_portToV1SourceMap记录了_sources1以及对应的Port，通过Port获取相应的Source1。

![通过Port获取Source1](http://upload-images.jianshu.io/upload_images/1464492-f7864c345b7e09d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 什么是CommonMode和CommonModeItem？

答:CommonModeItem是公共ModeItem，CommonMode是公共RunLoopMode，当把一个RunLoopMode注册为CommonMode时，CommonModeItem被会自动添加到CommonMode里，当CommonModeItem有所改动时，CommonMode也会作出相应的改动。

#### 什么是定时源(Timer Source)？

答:用于延时或重复的时间间隔处理事件。

#### 如何使用定时源？

答:CFRunLoopTimerRef是RunLoop中唯一的定时源，以定时器的形式表示和使用，可选择的定时器有，NSTimer，CADisplayLink，CFRunLoopTimerRef，GCD Timer。

#### 四种定时器的区别是什么？

答:NSTimer和CADisplayLink是Cocoa提供的高层定时器，CFRunLoopTimerRef是Core Foundation提供的基础定时器，NSTimer和CADisplayLink是建立在CFRunLoopTimerRef之上的高层组件，而CFRunLoopTimerRef是建立在mk_timer之上。NSTimer和CADisplayLink主要区别在于信号的发射频率不同，CADisplayLink的信号发射频率固定在16.67ms一次，而NSTimer的信号发射频率可自由定义(具体请看[iOS10定时消息的改动](https://www.jianshu.com/p/7045813769fd))。

![GCD Timer调用栈](http://upload-images.jianshu.io/upload_images/1464492-2639ddf1d4f9347b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


GCD Timer有别于前三种定时器，是由GCD系统所管理的定时器，通过一定的时间间隔dispatch任务到相应的队列中处理，以主线程为例，时间间隔到达后，GCD系统将Block dispatch到主线程对应的Main Queue，等待Main RunLoop检测和处理。

#### 如何选择使用哪一种定时器？

答:除非需要实现定时动画，否则不建议使用CADisplayLink作为定时器(具体请看[iOS10定时消息的改动](https://www.jianshu.com/p/7045813769fd)。什么是定时动画？请查看[iOS动画的基础知识](https://www.jianshu.com/p/406a14e5d9a5))；NSTimer适用于大部份情况，但需要注意循环引用的问题；GCD Timer的缺点在于，不能在自己所创建的子线程中使用。

#### CFRunLoopTimerRef的触发原理是怎样的？

答:具体请看[iOS10定时消息的改动](https://www.jianshu.com/p/7045813769fd)。

#### 什么是输入源？

答:是RunLoop所处理的事件源之一，主要用于线程或进程交互，输入源分为基于端口输入源(Source1)和非端口输入源(Source0)。

#### 基于端口输入源(Source1)与非端口输入源(Source0)的区别是什么？

答:1).Source0与Source1都是CFRunLoopSourceRef类型，但配置方式不同，Source0用CFRunLoopSourceContext来配置，Source1用CFRunLoopSourceContext1来配置。

     2).Source0与Source1都可用于线程(或进程)交互，但交互的形式有所不同，Source1监听端口，当端口有消息到达时，相应的Source1就会被触发回调，完成相应的操作；而Source0并不监听端口，让Source0执行回调需要手动标记Source0为待处理状态，还需要呼醒Source0所在的RunLoop。

![Source1交互](http://upload-images.jianshu.io/upload_images/1464492-df2508a16af3cf5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Source0交互](http://upload-images.jianshu.io/upload_images/1464492-74b360893d41ee6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


      3).从Source0与Source1的交式方式了解到，Source1的交互会主动呼醒所在的RunLoop，而Source0的交互则需要依赖其它线程来呼醒Source0所在的RunLoop。

      4).一次Loop只能执行一个Source1的回调，但一次Loop可以执行多个待处理的Source0的回调。

#### 如何创建Source0?

![创建Source0](http://upload-images.jianshu.io/upload_images/1464492-83c03ce7dc8a61c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 如何标记Source为待处理状态，且呼醒所在的RunLoop？

![Source0交互](http://upload-images.jianshu.io/upload_images/1464492-5b68f50a7691b70d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 如何创建Source1以及如何交互?

答:有Cocoa和Core Foundation两套API来配置和使用Source1；Cocoa有NSPort，NSMachPort，NSMessagePort，NSSocketPort等类，Core Foundation有CFMachPortRef，CFMessagePortRef，CFSocketRef等。其中用得比较多的是NSMachPort和CFMessagePortRef。

![CFMessagePortRef](http://upload-images.jianshu.io/upload_images/1464492-a2cf5b1670db856b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![NSMachPort](http://upload-images.jianshu.io/upload_images/1464492-b077e0bfe3a5d58d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Cocoa所提供的类只是建立在Core Foundation之上的高层组件，且提供了toll-free bridged。需要注意的是，NSMachPort接收和发送需要是同一个对象；CFMessagePortRef接收和发送的Port所用的name要相同，CFMessagePortSendRequest()函数通过CFMessagePortRef的name来查找相应的接收端口来进行消息发送(不建议直接使用mach_msg()来发送消息，关于Port可以查看[Inter-Process Communication](https://link.jianshu.com?t=http://nshipster.com/inter-process-communication/))。

#### RunLoop的内部逻辑是怎样的？

![Run内部逻辑](http://upload-images.jianshu.io/upload_images/1464492-9fc1d8000d5cb119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![官方文档的Run内部逻辑](http://upload-images.jianshu.io/upload_images/1464492-8a5ab0c6b6394706.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


需要注意的是第五步，官方文档写的是如果有基于端口的输入源待处理，就进入第九步，这跟CFRunLoop的源码不同。

![CFRunLoop Run源码第五步](http://upload-images.jianshu.io/upload_images/1464492-6ccd5234f9d8af0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从源码可以看到，第五步检测的是GCD端口事件，而不是官方文档所写的基于端口的输入源，但经过大量测试发现，第五步实际上会检测所有未处理的端口事件，而并非像官方文档或源码所展示的那样(太坑爹了，居然官方文档跟源码不同，源码又和实际测试结果不同😂)。

如果有什么地方写错的麻烦指出，如果有什么还想知道的请在评论留言，我会尽快补上的。

