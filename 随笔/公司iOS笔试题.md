公司iOS笔试题

1. block 是如何访问外部变量的？他在使用中需要注意什么？如何解决？

2. UIView与CLayer有什么区别？

3. 写几个你会的sqlite语句？

4. 介绍下内存的几大区域?

5. 多线程中并行和并发概念？NSOperation和GCD对比优缺点？多线程使用需注意哪些问题？

6. 什么情况使用 weak 关键字，相比 assign 有什么不同？

7. runtime 如何实现 weak 属性

8. runtime如何通过selector找到对应的IMP地址

9. 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

10.内存泄漏可能会出现的几种原因，聊聊你的看法？

11.如果项目开始容错处理没做？如何防止拦截潜在的崩溃？

12. 遇到tableView卡顿嘛？会造成卡顿的原因大致有哪些？

13. app优化你是从哪几方面着手？


公司面试题
1. http和scoket通信的区别。
答： http是客户端用http协议进行请求，发送请求时候需要封装http请求头，并绑定请求的数据，服务器一般有web服务器配合（当然也非绝对）。 http请求方式为客户端主动发起请求，服务器才能给响应，一次请求完毕后则断开连接，以节省资源。服务器不能主动给客户端响应（除非采取http长连接 技术）。iphone主要使用类是NSUrlConnection。
scoket是客户端跟服务器直接使用socket“套接字”进行连接，并没有规定连接后断开，所以客户端和服务器可以保持连接通道，双方 都可以主动发送数据。一般在游戏开发或股票开发这种要求即时性很强并且保持发送数据量比较大的场合使用。主要使用类是CFSocketRef。

1. block 是如何访问外部变量的？他在使用中需要注意什么？如何解决？

2. UIView与CLayer有什么区别？

3. C和obj-c 如何混用
答: 1).obj-c的编译器处理后缀为m的文件时，可以识别obj-c和c的代码，处理mm文件可以识别obj-c,c,c++代码，但cpp文件必须只能用c/c++代码，而且cpp文件include的头文件中，也不能出现obj-c的代码，因为cpp只是cpp
2).在mm文件中混用cpp直接使用即可，所以obj-c混cpp不是问题
3).在cpp中混用obj-c其实就是使用obj-c编写的模块是我们想要的。
如果模块以类实现，那么要按照cpp class的标准写类的定义，头文件中不能出现obj-c的东西，包括#import cocoa的。实现文件中，即类的实现代码中可以使用obj-c的东西，可以import,只是后缀是mm。
如果模块以函数实现，那么头文件要按c的格式声明函数，实现文件中，c++函数内部可以用obj-c，但后缀还是mm或m。
总结：只要cpp文件和cpp include的文件中不包含obj-c的东西就可以用了，cpp混用obj-c的关键是使用接口，而不能直接使用 实现代 码，实际上cpp混用的是obj-c编译后的o文件，这个东西其实是无差别的，所以可以用。obj-c的编译器支持cpp

4. Objective-C堆和栈的区别？
答: 管理方式：对于栈来讲，是由编译器自动管理，无需我们手工控制；对于堆来说，释放工作由程序员控制，容易产生memory leak。
申请大小：
栈：在Windows下,栈是向低地址扩展的数据结构，是一块连续的内存的区域。这句话的意思是栈顶的地址和栈的最大容量是系统预先规定好的，在 WINDOWS下，栈的大小是2M（也有的说是1M，总之是一个编译时就确定的常数），如果申请的空间超过栈的剩余空间时，将提示overflow。因 此，能从栈获得的空间较小。
堆：堆是向高地址扩展的数据结构，是不连续的内存区域。这是由于系统是用链表来存储的空闲内存地址的，自然是不连续的，而链表的遍历方向是由低地址向高地址。堆的大小受限于计算机系统中有效的虚拟内存。由此可见，堆获得的空间比较灵活，也比较大。
碎片问题：对于堆来讲，频繁的new/delete势必会造成内存空间的不连续，从而造成大量的碎片，使程序效率降低。对于栈来讲，则不会存在这个问题，因为栈是先进后出的队列，他们是如此的一一对应，以至于永远都不可能有一个内存块从栈中间弹出
分配方式：堆都是动态分配的，没有静态分配的堆。栈有2种分配方式：静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配由alloca函数进行分配，但是栈的动态分配和堆是不同的，他的动态分配是由编译器进行释放，无需我们手工实现。
分配效率：栈是机器系统提供的数据结构，计算机会在底层对栈提供支持：分配专门的寄存器存放栈的地址，压栈出栈都有专门的指令执行，这就决定了栈的效率比较高。堆则是C/C++函数库提供的，它的机制是很复杂的。

5 JC与OC如何交互？

6. 什么情况使用 weak 关键字，相比 assign 有什么不同？
什么情况使用 weak 关键字？
1. 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
2. 自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。在下文也有论述：***《IBOutlet连出来的视图属性为什么可以被设置成weak?》***
不同点：
1. weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。
2. assign 可以用非 OC 对象,而 weak 必须用于 OC 对象

7. 怎么用 copy 关键字？
用途：
1. NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
2. block 也经常使用 copy 关键字，具体原因见官方文档：Objects Use Properties to Keep Track of Blocks：
block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作。如果不写 copy ，该类的调用者有可能会忘记或者根本不知道“编译器会自动对 block 进行了 copy 操作”，他们有可能会在调用之前自行拷贝属性值。这种操作多余而低效。你也许会感觉我这种做法有些怪异，不需要写依然写。如果你这样想，其实是你“日用而不知”，你平时开发中是经常在用我说的这种做法的，比如下面的属性不写copy也行，但是你会选择写还是不写呢？
@property (nonatomic, copy) NSString *userId;

- (instancetype)initWithUserId:(NSString *)userId {
   self = [super init];
   if (!self) {
       return nil;
   }
   _userId = [userId copy];
   return self;
}

7. runtime如何通过selector找到对应的IMP地址
类对象中有类方法和实例方法的列表，列表中记录着方法的名词、参数和实现，而selector本质就是方法名称，runtime通过这个方法名称就可以在列表中找到该方法对应的实现。
类对象中声明了一个指向struct objc_method_list指针的指针，可以包含类方法列表和实例方法列表

8. runtime 如何实现 weak 属性
要实现 weak 属性，首先要搞清楚 weak 属性的特点：
weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。
那么 runtime 如何实现 weak 变量的自动置nil？
runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

9. 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？
1. 因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本.
2. 如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.

10.内存泄漏可能会出现的几种原因，聊聊你的看法？
第一种可能：第三方框架不当使用；
第二种可能：block循环引用；
第三种可能：delegate循环引用；
第四种可能：NSTimer循环引用
第五种可能：非OC对象内存处理
第六种可能：地图类处理
第七种可能：大次数循环内存暴涨
追问一：非OC对象如何处理？
非OC对象，其需要手动执行释放操作例：CGImageRelease(ref)，否则会造成大量的内存泄漏导致程序崩溃。
其他的对于CoreFoundation框架下的某些对象或变量需要手动释放、C语言代码中的malloc等需要对应free。
追问二：地图类内存若泄漏，如何处理？
地图是比较耗费App内存的，因此在根据文档实现某地图相关功能的同时，需要注意内存的正确释放，大体需要注意的有需在使用完毕时将地图、代理等滞空为nil；
注意地图中标注（大头针）的复用，并且在使用完毕时清空标注数组等。

11.容错处理你们一般是注意哪些？
在团队协作开发当中，由于每个团队成员的水平不一，很难控制代码的质量，保证代码的健壮性，经常会发生由于后台返回异常数据造成app崩溃闪退的情况，为了避免这样的情况项目中做一些容错处理，显得格外重要，极大程度上降低了因为数据容错不到位产生崩溃闪退的概率。
例如：
1.字典
2.数组；
3.野指针；
4.NSNull
等~

12.如果项目开始容错处理没做？如何防止拦截潜在的崩溃？
例：
1、category给类添加方法用来替换掉原本存在潜在崩溃的方法。
2、利用runtime方法交换技术，将系统方法替换成类添加的新方法。
3、利用异常的捕获来防止程序的崩溃，并且进行相应的处理。
总结：
1、不要过分相信服务器返回的数据会永远的正确。
2、在对数据处理上，要进行容错处理，进行相应判断之后再处理数据，这是一个良好的编程习惯。

12 一般开始做一个项目，你的架构是如何思考的？

13 遇到tableView卡顿嘛？会造成卡顿的原因大致有哪些？

可能造成tableView卡顿的原因有：
1.最常用的就是cell的重用， 注册重用标识符
如果不重用cell时，每当一个cell显示到屏幕上时，就会重新创建一个新的cell；
如果有很多数据的时候，就会堆积很多cell。
如果重用cell，为cell创建一个ID，每当需要显示cell 的时候，都会先去缓冲池中寻找可循环利用的cell，如果没有再重新创建cell
2.避免cell的重新布局
cell的布局填充等操作 比较耗时，一般创建时就布局好
如可以将cell单独放到一个自定义类，初始化时就布局好
3.提前计算并缓存cell的属性及内容
当我们创建cell的数据源方法时，编译器并不是先创建cell 再定cell的高度
而是先根据内容一次确定每一个cell的高度，高度确定后，再创建要显示的cell，滚动时，每当cell进入凭虚都会计算高度，提前估算高度告诉编译器，编译器知道高度后，紧接着就会创建cell，这时再调用高度的具体计算方法，这样可以方式浪费时间去计算显示以外的cell
4.减少cell中控件的数量
尽量使cell得布局大致相同，不同风格的cell可以使用不用的重用标识符，初始化时添加控件，
不适用的可以先隐藏
5.不要使用ClearColor，无背景色，透明度也不要设置为0
渲染耗时比较长
6.使用局部更新
如果只是更新某组的话，使用reloadSection进行局部更新
7.加载网络数据，下载图片，使用异步加载，并缓存
8.少使用addView 给cell动态添加view
9.按需加载cell，cell滚动很快时，只加载范围内的cell
10.不要实现无用的代理方法，tableView只遵守两个协议
11.缓存行高：estimatedHeightForRow不能和HeightForRow里面的layoutIfNeed同时存在，这两者同时存在才会出现“窜动”的bug。所以我的建议是：只要是固定行高就写预估行高来减少行高调用次数提升性能。如果是动态行高就不要写预估方法了，用一个行高的缓存字典来减少代码的调用次数即可
12.不要做多余的绘制工作。在实现drawRect:的时候，它的rect参数就是需要绘制的区域，这个区域之外的不需要进行绘制。例如上例中，就可以用CGRectIntersectsRect、CGRectIntersection或CGRectContainsRect判断是否需要绘制image和text，然后再调用绘制方法。
13.预渲染图像。当新的图像出现时，仍然会有短暂的停顿现象。解决的办法就是在bitmap context里先将其画一遍，导出成UIImage对象，然后再绘制到屏幕；
14.使用正确的数据结构来存储数据。

14. app优化你是从哪几方面着手？
一、首页启动速度
启动过程中做的事情越少越好（尽可能将多个接口合并）
不在UI线程上作耗时的操作（数据的处理在子线程进行，处理完通知主线程刷新节目）
在合适的时机开始后台任务（例如在用户指引节目就可以开始准备加载的数据）
尽量减小包的大小
优化方法：
量化启动时间
启动速度模块化
辅助工具（友盟，听云，Flurry）

二、页面浏览速度
json的处理（iOS 自带的NSJSONSerialization，Jsonkit，SBJson）
数据的分页（后端数据多的话，就要分页返回，例如网易新闻，或者 微博记录）
数据压缩（大数据也可以压缩返回，减少流量，加快反应速度）
内容缓存（例如网易新闻的最新新闻列表都是要缓存到本地，从本地加载，可以缓存到内存，或者数据库，根据情况而定）
延时加载tab（比如app有5个tab，可以先加载第一个要显示的tab，其他的在显示时候加载，按需加载）
算法的优化（核心算法的优化，例如有些app 有个 联系人姓名用汉语拼音的首字母排序）

三、操作流畅度优化：
Tableview 优化（tableview cell的加载优化）
ViewController加载优化（不同view之间的跳转，可以提前准备好数据）

四、数据库的优化：
数据库设计上面的重构
查询语句的优化
分库分表（数据太多的时候，可以分不同的表或者库）

五、服务器端和客户端的交互优化：
客户端尽量减少请求
服务端尽量做多的逻辑处理
服务器端和客户端采取推拉结合的方式（可以利用一些同步机制）
通信协议的优化。（减少报文的大小）
电量使用优化（尽量不要使用后台运行）

六、非技术性能优化
产品设计的逻辑性（产品的设计一定要符合逻辑，或者逻辑尽量简单，否则会让程序员抓狂，有时候用了好大力气，才可以完成一个小小的逻辑设计问题）
界面交互的规范（每个模块的界面的交互尽量统一，符合操作习惯）
代码规范（这个可以隐形带来app 性能的提高，比如 用if else 还是switch ，或者是用！还是 ＝＝）
code review（坚持code Review 持续重构代码。减少代码的逻辑复杂度）
日常交流（经常分享一些代码，或者逻辑处理中的坑）
