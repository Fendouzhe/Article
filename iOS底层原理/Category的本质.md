## iOS底层原理总结 - Category的本质

### 面试题

1.  Category的实现原理，以及Category为什么只能加方法不能加属性。
2.  Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？
3.  load、initialize的区别，以及它们在category重写的时候的调用的次序。

### Category的本质

首先我们写一段简单的代码，之后的分析都基于这段代码。

```
Presen类 
// Presen.h
#import <Foundation/Foundation.h>
@interface Preson : NSObject
{
    int _age;
}
- (void)run;
@end

// Presen.m
#import "Preson.h"
@implementation Preson
- (void)run
{
    NSLog(@"Person - run");
}
@end

Presen扩展1
// Presen+Test.h
#import "Preson.h"
@interface Preson (Test) <NSCopying>
- (void)test;
+ (void)abc;
@property (assign, nonatomic) int age;
- (void)setAge:(int)age;
- (int)age;
@end

// Presen+Test.m
#import "Preson+Test.h"
@implementation Preson (Test)
- (void)test
{
}

+ (void)abc
{
}
- (void)setAge:(int)age
{
}
- (int)age
{
    return 10;
}
@end

Presen分类2
// Preson+Test2.h
#import "Preson.h"
@interface Preson (Test2)
@end

// Preson+Test2.m
#import "Preson+Test2.h"
@implementation Preson (Test2)
- (void)run
{
    NSLog(@"Person (Test2) - run");
}
@end

```

我们之前讲到过实例对象的isa指针指向类对象，类对象的isa指针指向元类对象，当p调用run方法时，通过实例对象的isa指针找到类对象，然后在类对象中查找对象方法，如果没有找到，就通过类对象的superclass指针找到父类对象，接着去寻找run方法。

那么当调用分类的方法时，步骤是否和调用对象方法一样呢？
**分类中的对象方法依然是存储在类对象中的，同本类对象方法在同一个地方，调用步骤也同调用对象方法一样。如果是类方法的话，也同样是存储在元类对象中。**
那么分类方法是如何存储在类对象中的，我们来通过源码看一下分类的底层结构。

### 分类的底层结构

如何验证上述问题？通过查看分类的源码我们可以找到category_t 结构体。

```
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods; // 对象方法
    struct method_list_t *classMethods; // 类方法
    struct protocol_list_t *protocols; // 协议
    struct property_list_t *instanceProperties; // 属性
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};

```

从源码基本可以看出我们平时使用categroy的方式，对象方法，类方法，协议，和属性都可以找到对应的存储方式。并且我们发现分类结构体中是不存在成员变量的，因此分类中是不允许添加成员变量的。分类中添加的属性并不会帮助我们自动生成成员变量，只会生成get set方法的声明，需要我们自己去实现。

通过源码我们发现，分类的方法，协议，属性等好像确实是存放在categroy结构体里面的，那么他又是如何存储在类对象中的呢？
我们来看一下底层的内部方法探寻其中的原理。
首先我们通过命令行将Preson+Test.m文件转化为c++文件，查看其中的编译过程。

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Preson+Test.m

```

在分类转化为c++文件中可以看出_category_t结构体中，存放着类名，对象方法列表，类方法列表，协议列表，以及属性列表。

![image](//upload-images.jianshu.io/upload_images/1434508-78990f54c5edb32e.png?imageMogr2/auto-orient/strip|imageView2/2/w/820)

紧接着，我们可以看到_method_list_t类型的结构体，如下图所示

![image](//upload-images.jianshu.io/upload_images/1434508-7fecec0e35cbfe3e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1179)

上图中我们发现这个结构体**`_OBJC_$_CATEGORY_INSTANCE_METHODS_Preson_$_Test`**从名称可以看出是INSTANCE_METHODS对象方法，并且一一对应为上面结构体内赋值。我们可以看到结构体中存储了方法占用的内存，方法数量，以及方法列表。并且从上图中找到分类中我们实现对应的对象方法，test , setAge, age三个方法

接下来我们发现同样的_method_list_t类型的类方法结构体，如下图所示

![image](//upload-images.jianshu.io/upload_images/1434508-e81b6858a16dc4cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1135)

同上面对象方法列表一样，这个我们可以看出是类方法列表结构体 `_OBJC_$_CATEGORY_CLASS_METHODS_Preson_$_Test`，同对象方法结构体相同，同样可以看到我们实现的类方法，abc。

接下来是协议方法列表

![image](//upload-images.jianshu.io/upload_images/1434508-19ef40f48eeebfd3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1189)

通过上述源码可以看到先将协议方法通过_method_list_t结构体存储，之后通过_protocol_t结构体存储在**`_OBJC_CATEGORY_PROTOCOLS_$_Preson_$_Test`**中同_protocol_list_t结构体一一对应，分别为protocol_count 协议数量以及存储了协议方法的_protocol_t结构体。

最后我们可以看到属性列表

![image](//upload-images.jianshu.io/upload_images/1434508-e5878cf5fc015c7c.png?imageMogr2/auto-orient/strip|imageView2/2/w/962)

属性列表结构体**`_OBJC_$_PROP_LIST_Preson_$_Test`**同_prop_list_t结构体对应，存储属性的占用空间，属性属性数量，以及属性列表，从上图中可以看到我们自己写的age属性。

最后我们可以看到定义了**`_OBJC_$_CATEGORY_Preson_$_Test`**结构体，并且将我们上面着重分析的结构体一一赋值，我们通过两张图片对照一下。

![image](//upload-images.jianshu.io/upload_images/1434508-ed3e04186e33e0d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/689)

![image](//upload-images.jianshu.io/upload_images/1434508-a2dffad1c81f5d72.png?imageMogr2/auto-orient/strip|imageView2/2/w/1194)

上下两张图一一对应，并且我们看到定义`_class_t`类型的`OBJC_CLASS_$_Preson`结构体，最后将`_OBJC_$_CATEGORY_Preson_$_Test`的`cls`指针指向`OBJC_CLASS_$_Preson`结构体地址。我们这里可以看出，`cls`指针指向的应该是分类的主类类对象的地址。

通过以上分析我们发现。分类源码中确实是将我们定义的对象方法，类方法，属性等都存放在catagory_t结构体中。接下来我们在回到runtime源码查看catagory_t存储的方法，属性，协议等是如何存储在类对象中的。

首先来到runtime初始化函数

![image](//upload-images.jianshu.io/upload_images/1434508-0b43b0c4f1a2f9e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/812)

接着我们来到 &map_images读取模块（images这里代表模块），来到map_images_nolock函数中找到_read_images函数，在_read_images函数中我们找到分类相关代码

![image](//upload-images.jianshu.io/upload_images/1434508-89ef8494e5741ac2.png?imageMogr2/auto-orient/strip|imageView2/2/w/866)

从上述代码中我们可以知道这段代码是用来查找有没有分类的。通过_getObjc2CategoryList函数获取到分类列表之后，进行遍历，获取其中的方法，协议，属性等。可以看到最终都调用了remethodizeClass(cls);函数。我们来到remethodizeClass(cls);函数内部查看。

![image](//upload-images.jianshu.io/upload_images/1434508-81023f6e55b830af.png?imageMogr2/auto-orient/strip|imageView2/2/w/852)

通过上述代码我们发现attachCategories函数接收了类对象cls和分类数组cats，如我们一开始写的代码所示，一个类可以有多个分类。之前我们说到分类信息存储在category_t结构体中，那么多个分类则保存在category_list中。

我们来到attachCategories函数内部。

![image](//upload-images.jianshu.io/upload_images/1434508-0c16f9a121a704c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/942)

上述源码中可以看出，首先根据方法列表，属性列表，协议列表，malloc分配内存，根据多少个分类以及每一块方法需要多少内存来分配相应的内存地址。之后从分类数组里面往三个数组里面存放分类数组里面存放的分类方法，属性以及协议放入对应mlist、proplists、protolosts数组中，这三个数组放着所有分类的方法，属性和协议。
之后通过类对象的data()方法，拿到类对象的class_rw_t结构体rw，在class结构中我们介绍过，class_rw_t中存放着类对象的方法，属性和协议等数据，rw结构体通过类对象的data方法获取，所以rw里面存放这类对象里面的数据。
之后分别通过rw调用方法列表、属性列表、协议列表的attachList函数，将所有的分类的方法、属性、协议列表数组传进去，我们大致可以猜想到在attachList方法内部将分类和本类相应的对象方法，属性，和协议进行了合并。

我们来看一下attachLists函数内部。

![image](//upload-images.jianshu.io/upload_images/1434508-5b7b751278a9de70.png?imageMogr2/auto-orient/strip|imageView2/2/w/1009)

**上述源代码中有两个重要的数组**
**array()->lists： 类对象原来的方法列表，属性列表，协议列表。**
**addedLists：传入所有分类的方法列表，属性列表，协议列表。**

attachLists函数中最重要的两个方法为memmove内存移动和memcpy内存拷贝。我们先来分别看一下这两个函数

```
// memmove ：内存移动。
/*  __dst : 移动内存的目的地
*   __src : 被移动的内存首地址
*   __len : 被移动的内存长度
*   将__src的内存移动__len块内存到__dst中
*/
void    *memmove(void *__dst, const void *__src, size_t __len);

// memcpy ：内存拷贝。
/*  __dst : 拷贝内存的拷贝目的地
*   __src : 被拷贝的内存首地址
*   __n : 被移动的内存长度
*   将__src的内存移动__n块内存到__dst中
*/
void    *memcpy(void *__dst, const void *__src, size_t __n);

```

下面我们图示经过memmove和memcpy方法过后的内存变化。

![image](//upload-images.jianshu.io/upload_images/1434508-de4cc8308b362d72.png?imageMogr2/auto-orient/strip|imageView2/2/w/743)

经过memmove方法之后，内存变化为

```
// array()->lists 原来方法、属性、协议列表数组
// addedCount 分类数组长度
// oldCount * sizeof(array()->lists[0]) 原来数组占据的空间
memmove(array()->lists + addedCount, array()->lists, 
                  oldCount * sizeof(array()->lists[0]));

```

![image](//upload-images.jianshu.io/upload_images/1434508-d57c524988754a9c.png?imageMogr2/auto-orient/strip|imageView2/2/w/917)

经过memmove方法之后，我们发现，虽然本类的方法，属性，协议列表会分别后移，但是本类的对应数组的指针依然指向原始位置。

memcpy方法之后，内存变化

```
// array()->lists 原来方法、属性、协议列表数组
// addedLists 分类方法、属性、协议列表数组
// addedCount * sizeof(array()->lists[0]) 原来数组占据的空间
memcpy(array()->lists, addedLists, 
               addedCount * sizeof(array()->lists[0]));

```

![image](//upload-images.jianshu.io/upload_images/1434508-2f70771f3deffad7.png?imageMogr2/auto-orient/strip|imageView2/2/w/916)

我们发现原来指针并没有改变，至始至终指向开头的位置。并且经过memmove和memcpy方法之后，分类的方法，属性，协议列表被放在了类对象中原本存储的方法，属性，协议列表前面。

那么为什么要将分类方法的列表追加到本来的对象方法前面呢，这样做的目的是为了保证分类方法优先调用，我们知道当分类重写本类的方法时，会覆盖本类的方法。
其实经过上面的分析我们知道本质上并不是覆盖，而是优先调用。本类的方法依然在内存中的。我们可以通过打印所有类的所有方法名来查看

```
- (void)printMethodNamesOfClass:(Class)cls
{
    unsigned int count;
    // 获得方法数组
    Method *methodList = class_copyMethodList(cls, &count);
    // 存储方法名
    NSMutableString *methodNames = [NSMutableString string];
    // 遍历所有的方法
    for (int i = 0; i < count; i++) {
        // 获得方法
        Method method = methodList[i];
        // 获得方法名
        NSString *methodName = NSStringFromSelector(method_getName(method));
        // 拼接方法名
        [methodNames appendString:methodName];
        [methodNames appendString:@", "];
    }
    // 释放
    free(methodList);
    // 打印方法名
    NSLog(@"%@ - %@", cls, methodNames);
}
- (void)viewDidLoad {
    [super viewDidLoad];    
    Preson *p = [[Preson alloc] init];
    [p run];
    [self printMethodNamesOfClass:[Preson class]];
}

```

通过下图中打印内容可以发现，调用的是Test2中的run方法，并且Person类中存储着两个run方法。

![image](//upload-images.jianshu.io/upload_images/1434508-e918e62841729d82.png?imageMogr2/auto-orient/strip|imageView2/2/w/919)

#### 总结：

问： Category的实现原理，以及Category为什么只能加方法不能加属性?

答：分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。

### load 和 initialize

load方法会在程序启动就会调用，当装载类信息的时候就会调用。
调用顺序看一下源代码。

![image](//upload-images.jianshu.io/upload_images/1434508-708c460c656a6f6c.png?imageMogr2/auto-orient/strip|imageView2/2/w/819)

通过源码我们发现是优先调用类的load方法，之后调用分类的load方法。

我们通过代码验证一下：
我们添加Student继承Presen类，并添加Student+Test分类，分别重写只+load方法，其他什么都不做通过打印发现

![image](//upload-images.jianshu.io/upload_images/1434508-1d4aee32a833695e.png?imageMogr2/auto-orient/strip|imageView2/2/w/794)

确实是优先调用类的load方法之后调用分类的load方法，不过调用类的load方法之前会保证其父类已经调用过load方法。

之后我们为Preson、Student 、Student+Test 添加initialize方法。
我们知道当类第一次接收到消息时，就会调用initialize，相当于第一次使用类的时候就会调用initialize方法。调用子类的initialize之前，会先保证调用父类的initialize方法。如果之前已经调用过initialize，就不会再调用initialize方法了。当分类重写initialize方法时会先调用分类的方法。但是load方法并不会被覆盖，首先我们来看一下initialize的源码。

![image](//upload-images.jianshu.io/upload_images/1434508-ec80b8d39337c614.png?imageMogr2/auto-orient/strip|imageView2/2/w/669)

上图中我们发现，initialize是通过消息发送机制调用的，消息发送机制通过isa指针找到对应的方法与实现，因此先找到分类方法中的实现，会优先调用分类方法中的实现。

我们再来看一下load方法的调用源码

![image](//upload-images.jianshu.io/upload_images/1434508-6e323496c3792af8.png?imageMogr2/auto-orient/strip|imageView2/2/w/768)

我们看到load方法中直接拿到load方法的内存地址直接调用方法，不在是通过消息发送机制调用。

![image](//upload-images.jianshu.io/upload_images/1434508-ad02c1839e1ec5e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/711)

我们可以看到分类中也是通过直接拿到load方法的地址进行调用。因此正如我们之前试验的一样，分类中重写load方法，并不会优先调用分类的load方法，而不调用本类中的load方法了。

#### 总结

问：Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？
答：Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法

问：load、initialize的区别，以及它们在category重写的时候的调用的次序。
答：区别在于调用方式和调用时刻
调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用
调用时刻：load是runtime加载类、分类的时候调用（只会调用1次），initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）

调用顺序：先调用类的load方法，先编译那个类，就先调用load。在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次），如果分类实现了+initialize，就覆盖类本身的+initialize调用。

