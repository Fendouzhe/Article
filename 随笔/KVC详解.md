> KVC（Key-value coding）键值编码，单看这个名字可能不太好理解。其实翻译一下就很简单了，就是指iOS的开发中，可以允许开发者通过Key名直接访问对象的属性，或者给对象的属性赋值。而不需要调用明确的存取方法。这样就可以在运行时动态地访问和修改对象的属性。而不是在编译时确定，这也是iOS开发中的黑魔法之一。很多高级的iOS开发技巧都是基于KVC实现的。目前网上关于KVC的文章在非常多，有的只是简单地说了下用法，有的讲得深入但是在使用场景和最佳实践没有说明，我写下这遍文章就是给大家详解一个最完整最详细的KVC。

## KVC在iOS中的定义

无论是`Swift`还是`Objective-C`，KVC的定义都是对`NSObject`的扩展来实现的(`Objective-C`中有个显式的`NSKeyValueCoding`类别名，而Swift没有，也不需要)。所以对于所有继承了`NSObject`的类型，也就是几乎所有的`Objective-C`对象都能使用`KVC`(一些纯`Swift`类和结构体是不支持KVC的)，下面是KVC最为重要的四个方法

```
- (nullable id)valueForKey:(NSString *)key;                          //直接通过Key来取值
- (void)setValue:(nullable id)value forKey:(NSString *)key;          //通过Key来设值
- (nullable id)valueForKeyPath:(NSString *)keyPath;                  //通过KeyPath来取值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;  //通过KeyPath来设值

```

当然NSKeyValueCoding类别中还有其他的一些方法，下面列举一些

```
+ (BOOL)accessInstanceVariablesDirectly;
//默认返回YES，表示如果没有找到Set<Key>方法的话，会按照_key，_iskey，key，iskey的顺序搜索成员，设置成NO就不这样搜索

- (BOOL)validateValue:(inout id __nullable * __nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;
//KVC提供属性值正确性�验证的API，它可以用来检查set的值是否正确、为不正确的值做一个替换值或者拒绝设置新值并返回错误原因。

- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;
//这是集合操作的API，里面还有一系列这样的API，如果属性是一个NSMutableArray，那么可以用这个方法来返回。

- (nullable id)valueForUndefinedKey:(NSString *)key;
//如果Key不存在，且没有KVC无法搜索到任何和Key有关的字段或者属性，则会调用这个方法，默认是抛出异常。

- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key;
//和上一个方法一样，但这个方法是设值。

- (void)setNilValueForKey:(NSString *)key;
//如果你在SetValue方法时面给Value传nil，则会调用这个方法

- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
//输入一组key,返回该组key对应的Value，再转成字典返回，用于将Model转到字典。

```

上面的这些方法在碰到特殊情况或者有特殊需求还是会用到的，所以也是可以c了解一下。后面的代码示例会有讲到其中的一些方法。

同时苹果对一些容器类比如`NSArray`或者`NSSet`等，KVC有着特殊的实现。建议有基础的或者英文好的开发者直接去看苹果的官方文档，相信你会对KVC的理解更上一个台阶。

> 可能有些读者不知道怎么查官方文档，在这里说明一下。打开Xcode，查看最上面的菜单，点最后一个Help -> Documentation and API Reference,然后就可以打开官方文档了。

![](http://upload-images.jianshu.io/upload_images/1464492-a232f68459d436ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## KVC是怎么寻找Key的

KVC是怎么使用的，我相信绝大多数的开发者都很清楚，我在这里就不再写简单的使用KVC来设值和取值的代码了，首�先我们来探讨KVC在内部是按什么样的顺序来寻找key的。

### 设值

当调用`setValue：属性值 forKey：@”name“`的代码时，底层的执行机制如下：

*   程序优先调用`set<Key>:属性值`方法，代码通过`setter`方法完成设置。**注意，这里的<key>是指成员变量名，首字母大小写要符合KVC的命名规则，下同**

*   如果没有找到`setName：`方法，KVC机制会检查`+ (BOOL)accessInstanceVariablesDirectly`方法有没有返回YES，默认该方法会返回YES，如果你重写了该方法让其返回NO的话，那么在这一步KVC会执行`setValue：forUndefinedKey：`方法，不过一般开发者不会这么做。所以KVC机制会搜索该类里面有没有名为`_<key>`的成员变量，无论该变量是在类接口处定义，还是在类实现处定义，也无论用了什么样的访问修饰符，只在存在以`_<key>`命名的变量，KVC都可以对该成员变量赋值。

*   如果该类即没有`set<key>：`方法，也没有`_<key>`成员变量，KVC机制会搜索`_is<Key>`的成员变量。

*   和上面一样，如果该类即没有`set<Key>：`方法，也没有`_<key>`和`_is<Key>`成员变量，KVC机制再会继续搜索`<key>`和`is<Key>`的成员变量。再给它们赋值。

*   如果上面列出的方法或者成员变量都不存在，系统将会执行该对象的`setValue：forUndefinedKey：`方法，默认是抛出异常。

如果开发者想让这个类禁用KVC里，那么重写`+ (BOOL)accessInstanceVariablesDirectly`方法让其返回NO即可，这样的话如果KVC没有找到`set<Key>:`属性名时，会直接用`setValue：forUndefinedKey：`方法。

下面我们来让代码来测试一下上面的KVC机制

```
@interface Dog : NSObject
@end
@implementation Dog
{
     NSString* toSetName;
    NSString* isName;
    //NSString* name;
    NSString* _name;
    NSString* _isName;
}
// -(void)setName:(NSString*)name{
//     toSetName = name;
// }
//-(NSString*)getName{
//    return toSetName;
//}
+(BOOL)accessInstanceVariablesDirectly{
    return NO;
}
-(id)valueForUndefinedKey:(NSString *)key{
    NSLog(@"出现异常，该key不存在%@",key);
    return nil;
}
-(void)setValue:(id)value forUndefinedKey:(NSString *)key{
     NSLog(@"出现异常，该key不存在%@",key);
}
@end
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        Dog* dog = [Dog new];
        [dog setValue:@"newName" forKey:@"name"];
        NSString* name = [dog valueForKey:@"toSetName"];
        NSLog(@"%@",name);
    }
    return 0;
}

```

首先我们先重写`accessInstanceVariablesDirectly`方法让其返回NO，再运行代码（注意上面注释的部分），Xcode直接打印出

```
2016-04-15 15:52:12.039 DemoKVC[9681:287627] 出现异常，该key不存在name
2016-04-15 15:52:12.040 DemoKVC[9681:287627] 出现异常，该key不存在toSetName
2016-04-15 15:52:12.040 DemoKVC[9681:287627] (null)

```

这说明了重写`+(BOOL)accessInstanceVariablesDirectly`方法让其返回NO后,KVC找不到`setName：`方法后，不再去找`name`系列成员变量，而是直接调用`setValue：forUndefinedKey：`方法

所以开发者如果不想让自己的类实现KVC，就可以这么做。

下面那两个`setter`和`getter`的注释取消掉，再把

```
NSString* name = [dog valueForKey:@"toSetName"]; 换成 NSString* name = [dog valueForKey:@"name"];

```

XCode就可以正确地打印出正确的值了

```
2016-04-15 15:56:22.130 DemoKVC[9726:289258] newName

```

下面再注释掉`accessInstanceVariablesDirectly`方法，就能测试其他的`key`查找顺序了，为了节省篇幅，剩下的的KVC对于`key`寻找机制就不在这里展示了，有兴趣的读者可以写代码去验证。

### 取值

当调用`valueForKey：@”name“`的代码时，KVC对`key`的搜索方式不同于`setValue：属性值 forKey：@”name“`，其搜索方式如下：

*   首先按`get<Key>`,`<key>`,`is<Key>`的顺序方法查找`getter`方法，找到的话会直接调用。如果是`BOOL`或者`Int`等值类型， 会将其包装成一个`NSNumber`对象。

*   如果上面的`getter`没有找到，KVC则会查找`countOf<Key>`,`objectIn<Key>AtIndex`或`<Key>AtIndexes`格式的方法。如果`countOf<Key>`方法和另外两个方法中的一个被找到，那么就会返回一个可以响应`NSArray`所�有方法的代理集合(它是`NSKeyValueArray`，是`NSArray`的子类)，调用这个代理集合的方法，或者说给这个代理集合发送属于`NSArray`的方法，就会以`countOf<Key>`,`objectIn<Key>AtIndex`�或`<Key>AtIndexes`这几个方法组合的形式调用。还有一个可选的`get<Key>:range:`方法。所以你想重新定义KVC的一些功能，你可以添加这些方法，需要注意的是你的方法名要符合KVC的标准命名方法，包括方法签名。

*   如果上面的方法没有找到，那么会同时查找`countOf<Key>`，`enumeratorOf<Key>`,`memberOf<Key>`格式的方法。如果这三个方法都找到，那么就返回一个可以响应`NSSet`所的方法的代理集合，和上面一样，给这个代理集合发`NSSet`的消息，就会以`countOf<Key>`，`enumeratorOf<Key>`,`memberOf<Key>`组合的形式调用。

> 可能上面的两条查找方案对读者不好理解，简单来说就是如果你在自己的类自定义了KVC的实现，并且实现了上面的方法，那么恭喜你，你可以�将返回的对象当数组(NSArray)用了，详情见下面的示例代码

*   如果还没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly`,如果返回YES(默认行为)，那么和先前的设值一样，会按`_<key>,_is<Key>,<key>,is<Key>`的顺序搜索成员变量名，这里不推荐这么做，因为这样直接访问实例变量破坏了封装性，使代码更脆弱。如果重写了类方法`+ (BOOL)accessInstanceVariablesDirectly`返回NO的话，那么会直接调用`valueForUndefinedKey:`

*   还没有找到的话，调用`valueForUndefinedKey:`

下面再上代码测试

```
@interface TwoTimesArray : NSObject
-(void)incrementCount;
-(NSUInteger)countOfNumbers;
-(id)objectInNumbersAtIndex:(NSUInteger)index;
@end
@interface TwoTimesArray()
@property (nonatomic,readwrite,assign) NSUInteger count;
@property (nonatomic,copy) NSString* arrName;
@end
@implementation TwoTimesArray
-(void)incrementCount{
    self.count ++;
}
-(NSUInteger)countOfNumbers{
    return self.count;

-(id)objectInNumbersAtIndex:(NSUInteger)index{     //当key使用numbers时，KVC会找到这两个方法。
    return @(index * 2);
}
-(NSInteger)getNum{                 //第一个,自己一个一个注释试
    return 10;
}
-(NSInteger)num{                       //第二个
    return 11;
}
-(NSInteger)isNum{                    //第三个
    return 12;
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        TwoTimesArray* arr = [TwoTimesArray new];
        NSNumber* num =   [arr valueForKey:@"num"];
        NSLog(@"%@",num);
        id ar = [arr valueForKey:@"numbers"];
        NSLog(@"%@",NSStringFromClass([ar class]));
         NSLog(@"0:%@     1:%@     2:%@     3:%@",ar[0],ar[1],ar[2],ar[3]);
        [arr incrementCount];                                                                            //count加1
        NSLog(@"%lu",(unsigned long)[ar count]);                                                         //打印出1
        [arr incrementCount];                                                                            //count再加1
        NSLog(@"%lu",(unsigned long)[ar count]);                                                         //打印出2

        [arr setValue:@"newName" forKey:@"arrName"];
        NSString* name = [arr valueForKey:@"arrName"];
        NSLog(@"%@",name);

    }
    return 0;
}
//打印结果 
2016-04-17 15:39:42.214 KVCDemo[1088:74481] 10
2016-04-17 15:39:42.215 KVCDemo[1088:74481] NSKeyValueArray
2016-04-17 15:41:24.713 KVCDemo[1102:75424] 0:0     1:2     2:4     3:6                 //太明显了，直接调用-(id)objectInNumbersAtIndex:(NSUInteger)index;方法
2016-04-17 15:39:42.215 KVCDemo[1088:74481] 1
2016-04-17 15:39:42.215 KVCDemo[1088:74481] 2
2016-04-17 15:39:42.215 KVCDemo[1088:74481] newName

```

很明显，上面的代码充分说明了说明了KVC在调用`ValueforKey：@”name“`时搜索`key`的机制。不过还有些功能没有全部列出，有兴趣的读者可以写代码去验证。

## 在KVC中使用keyPath

然而在开发过程中，一个类的成员变量有可能是自定义类或其他的复杂数据类型，你可以先用KVC获取该属性，然后再次用KVC来获取这个自定义类的属性，但这样是比较繁琐的，对此，KVC提供了一个解决方案，那就是键路径`keyPath`。

```
- (nullable id)valueForKeyPath:(NSString *)keyPath;                  //通过KeyPath来取值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;  //通过KeyPath来设值

```

```
@interface Address : NSObject

@end
@interface Address()
@property (nonatomic,copy)NSString* country;
@end
@implementation Address
@end
@interface People : NSObject
@end
@interface People()
@property (nonatomic,copy) NSString* name;
@property (nonatomic,strong) Address* address;
@property (nonatomic,assign) NSInteger age;
@end
@implementation People
@end
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        People* people1 = [People new];
        Address* add = [Address new];
        add.country = @"China";
        people1.address = add;
        NSString* country1 = people1.address.country;
        NSString * country2 = [people1 valueForKeyPath:@"address.country"];
        NSLog(@"country1:%@   country2:%@",country1,country2);
        [people1 setValue:@"USA" forKeyPath:@"address.country"];
         country1 = people1.address.country;
        country2 = [people1 valueForKeyPath:@"address.country"];
        NSLog(@"country1:%@   country2:%@",country1,country2);
    }
    return 0;
}
//打印结果 
2016-04-17 15:55:22.487 KVCDemo[1190:82636] country1:China   country2:China
2016-04-17 15:55:22.489 KVCDemo[1190:82636] country1:USA   country2:USA

```

上面的代码简单在展示了`keyPath`是怎么用的。如果你不小心错误的使用了`key`而非`keyPath`的话，比如上面的代码中KVC会直接查找`address.country`这个属性，很明显，这个属性并不存在，所以会再调用`undefinedKey`相关方法。而KVC对于`keyPath`是搜索机制第一步就是分离`key`，用小数点`.`来分割`key`，然后再像普通`key`一样按照先前介绍的顺序搜索下去。

## KVC如何处理异常

KVC中最常见的异常就是不小心使用了错误的`key`，或者在设值中不小心传递了`nil`的值，KVC中有专门的方法来处理这些异常。

通常在用KVC操作Model时，抛出异常的那两个方法是需要重写的。虽然一般很小出现传递了错误的Key值这种情况，但是如果不小心出现了，直接抛出异常让APP崩溃显然是不合理的。一般在这里直接让这个`key`打印出来即可，或者有些特殊情况需要特殊处理。通常情况下，KVC不允许你要在调用`setValue：属性值 forKey：@”name“`(或者keyPath)时**对非对象**传递一个`nil`的值。很简单，因为值类型是不能为`nil`的。如果你不小心传了，KVC会调用`setNilValueForKey:`方法。这个方法默认是抛出异常，所以一般而言最好还是重写这个方法。

```
  [people1 setValue:nil forKey:@"age"]
   *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '[<People 0x100200080> setNilValueForKey]: could not set nil as the value for the key age.' // 调用setNilValueForKey抛出异常

```

如果重写`setNilValueForKey:`就没问题了

```
@implementation People

-(void)setNilValueForKey:(NSString *)key{
    NSLog(@"不能将%@设成nil",key);
}

@end
//打印出
2016-04-17 16:19:55.298 KVCDemo[1304:92472] 不能将age设成nil

```

## KVC处理非对象和自定义对象

不是每一个方法都返回对象，但是`valueForKey：`总是返回一个id对象，如果原本的变量类型是值类型或者结构体，返回值会封装成`NSNumber`或者`NSValue`对象。这两个类会处理从数字，布尔值到指针和结构体任何类型。然后开以者需要手动转换成原来的类型。尽管`valueForKey：`会自动将值类型封装成对象，但是`setValue：forKey：`却不行。你必须手动将值类型转换成`NSNumber`或者`NSValue`类型，才能传递过去。

对于自定义对象，KVC也会正确地设值和取值。因为传递进去和取出来的都是`id`类型，所以需要开发者自己担保类型的正确性，运行时`Objective-C`在发送消息的会检查类型，如果错误会直接抛出异常。

```
Address* add2 = [Address new];
add2.country = @"England";
[people1 setValue:add2 forKey:@"address"];
NSString* country1 = people1.address.country;
NSString * country2 = [people1 valueForKeyPath:@"address.country"];
NSLog(@"country1:%@   country2:%@",country1,country2);
//打印结果
2016-04-17 16:29:36.349 KVCDemo[1346:95910] country1:England   country2:England

```

## KVC与容器类

对象的属性可以是一对一的，也可以是一对多的。一对多的属性要么是有序的(数组)，要么是无序的(集合)。

不可变的有序容器属性(`NSArray`)和无序容器属性(`NSSet`)一般可以使用`valueForKey:`来获取。比如有一个叫`items`的`NSArray`属性，你可以用`valurForKey:@"items"`来获取这个属性。前面`valueForKey:`的`key`搜索模式中，我们发现其实KVC使用了一种更灵活的方式来管理容器类。苹果的官方文档也推荐我们实现这些这些特殊的访问器。

而当对象的属性是可变的容器时，对于有序的容器，可以用下面的方法：

```
- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;

```

该方法返回一个可变有序数组，如果调用该方法，KVC的搜索顺序如下

*   搜索`insertObject:in<Key>AtIndex:` , `removeObjectFrom<Key>AtIndex:` 或者 `insert<Key>AdIndexes` , `remove<Key>AtIndexes` 格式的方法

    如果至少找到一个`insert`方法和一个`remove`方法，那么同样返回一个可以响应`NSMutableArray`所有方法代理集合(类名是`NSKeyValueFastMutableArray2`)，那么给这个代理集合发送`NSMutableArray`的方法，以`insertObject:in<Key>AtIndex:` , `removeObjectFrom<Key>AtIndex:` 或者 `insert<Key>AdIndexes` , `remove<Key>AtIndexes`组合的形式调用。还有两个可选实现的接口：`replaceOnjectAtIndex:withObject:`,`replace<Key>AtIndexes:with<Key>:`。

*   如果上步的方法没有找到，则搜索`set<Key>:` 格式的方法，如果找到，那么发送给代理集合的`NSMutableArray`最终都会调用`set<Key>:`方法。 也就是说，`mutableArrayValueForKey:`取出的代理集合修改后，用`set<Key>:` 重新赋值回去去。这样做效率会低很多。所以推荐实现上面的方法。

*   如果上一步的方法还还没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly`,如果返回YES(默认行为)，会按`_<key>`,`<key>`,的顺序搜索成员变量名，如果找到，那么发送的`NSMutableArray`消息方法直接交给这个成员变量处理。

*   如果还是找不到，则调用`valueForUndefinedKey:`。

*   关于`mutableArrayValueForKey:`的适用场景，我在网上找了很多，发现其一般是用在对`NSMutableArray`添加Observer上。如果对象属性是个`NSMutableArray、NSMutableSet、NSMutableDictionary`等集合类型时，你给它添加KVO时，你会发现当你添加或者移除元素时并不能接收到变化。因为KVO的本质是系统监测到某个属性的内存地址或常量改变时，会添加上`- (void)willChangeValueForKey:(NSString *)key`和`- (void)didChangeValueForKey:(NSString *)key`方法来发送通知，所以一种解决方法是手动调用者两个方法，但是并不推荐，你永远无法像系统一样真正知道这个元素什么时候被改变。另一种便是利用使用`mutableArrayValueForKey:`了。

```
@interface demo : NSObject
@property (nonatomic,strong) NSMutableArray* arr;
@end
@implementation demo
-(id)init{
    if (self == [super init]){
        _arr = [NSMutableArray new];
        [self addObserver:self forKeyPath:@"arr" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:nil];
    }
    return self;
}
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    NSLog(@"%@",change);
}
-(void)dealloc{
    [self removeObserver:self forKeyPath:@"arr"]; //一定要在dealloc里面移除观察
}
-(void)addItem{
    [_arr addObject:@"1"];
}
-(void)addItemObserver{
    [[self mutableArrayValueForKey:@"arr"] addObject:@"1"];
}
-(void)removeItemObserver{
    [[self mutableArrayValueForKey:@"arr"] removeLastObject];
}
@end
然后再:
demo* d = [demo new];
[d addItem];
[d addItemObserver];
[d removeItemObserver];

打印结果
2016-04-18 17:48:22.675 KVCDemo[32647:505864] {
    indexes = "<_NSCachedIndexSet: 0x100202c70>[number of indexes: 1 (in 1 ranges), indexes: (1)]";
    kind = 2;
    new =     (
        1
    );
}
2016-04-18 17:48:22.677 KVCDemo[32647:505864] {
    indexes = "<_NSCachedIndexSet: 0x100202c70>[number of indexes: 1 (in 1 ranges), indexes: (1)]";
    kind = 3;
    old =     (
        1
    );
}

```

从上面的代码可以看出，当只是普通地调用`[_arr addObject:@"1"]`时，`Observer`并不会回调，只有`[[self mutableArrayValueForKey:@"arr"] addObject:@"1"]`;这样写时才能正确地触发KVO。打印出来的数据中，可以看出这次操作的详情，`kind`可能是指操作方法(我还不是很确认)，`old`和`new`并不是成对出现的，当加添新数据时是`new`，删除数据时是`old`

而对于无序的容器，可以用下面的方法：

```
- (NSMutableSet *)mutableSetValueForKey:(NSString *)key;

```

该方法返回一个可变的无序数组如果调用该方法，KVC的搜索顺序如下

*   搜索`addObject<Key>Object:` , `remove<Key>Object:` 或者 `add<Key>` , `remove<Key>` 格式的方法

    如果至少找到一个`insert`方法和一个`remove`方法，那么同样返回一个可以响应`NSMutableSet`所有方法代理集合(类名是`NSKeyValueFastMutableSet2`)，那么给这个代理集合发送`NSMutableSet`的方法，以`addObject<Key>Object:` , `remove<Key>Object:` 或者 `add<Key>` , `remove<Key>`组合的形式调用。还有两个可选实现的接口：`intersect<Key> , set<Key>:` 。

*   如果`receiver`是`ManagedObject`，那么就不会继续搜索。

*   如果上一步的方法没有找到，则搜索`set<Key>`: 格式的方法，如果找到，那么发送给代理集合的`NSMutableSet`最终都会调用`set<Key>:`方法。 也就是说，`mutableSetValueForKey`取出的代理集合修改后，用`set<Key>:` 重新赋值回去去。这样做效率会低很多。所以推荐实现上面的方法。

*   如果上一步的方法还没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly`,如果返回`YES`(默认行为)，会按`_<key>`,`<key>`的顺序搜索成员变量名，如果找到，那么发送的`NSMutableSet`消息方法直接交给这个成员变量处理。

*   如果还是找不到，调用`valueForUndefinedKey:`

    可见，除了检查`receiver`是`ManagedObject`以外，其搜索顺序和`mutableArrayValueForKey`基本一至，

同样，它们也有对应的`keyPath`版本

```
- (NSMutableArray *)mutableArrayValueForKeyPath:(NSString *)keyPath;
- (NSMutableSet *)mutableSetValueForKeyPath:(NSString *)keyPath;

```

iOS5和OSX10.7以后还有个`mutableOrdered`版本

```
- (NSMutableOrderedSet *)mutableOrderedSetValueForKey:(NSString *)key

```

这两种KVC的用法我还不是清楚，目前只能找到用于KVO的例子。如果有读者能在项目中用到，希望可以告诉我。

## KVC和字典

当对`NSDictionary`对象使用KVC时，`valueForKey:`的表现行为和`objectForKey:`一样。所以使用`valueForKeyPath:`用来访问多层嵌套的字典是比较方便的。

KVC里面还有两个关于NSDictionary的方法

```
- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
- (void)setValuesForKeysWithDictionary:(NSDictionary<NSString *, id> *)keyedValues;

```

`dictionaryWithValuesForKeys:`是指输入一组`key`，返回这组`key`对应的属性，再组成一个字典。

`setValuesForKeysWithDictionary`是用来修改Model中对应`key`的属性。下面直接用代码会更直观一点

```
Address* add = [Address new];
add.country = @"China";
add.province = @"Guang Dong";
add.city = @"Shen Zhen";
add.district = @"Nan Shan";
NSArray* arr = @[@"country",@"province",@"city",@"district"];
NSDictionary* dict = [add dictionaryWithValuesForKeys:arr]; //把对应key所有的属性全部取出来
NSLog(@"%@",dict);

NSDictionary* modifyDict = @{@"country":@"USA",@"province":@"california",@"city":@"Los angle"};
[add setValuesForKeysWithDictionary:modifyDict];            //用key Value来修改Model的属性
NSLog(@"country:%@  province:%@ city:%@",add.country,add.province,add.city);

//打印结果
2016-04-19 11:54:30.846 KVCDemo[6607:198900] {
    city = "Shen Zhen";
    country = China;
    district = "Nan Shan";
    province = "Guang Dong";
}
2016-04-19 11:54:30.847 KVCDemo[6607:198900] country:USA  province:california city:Los angle

```

打印出来的结果完全符合预期。

## KVC的内部实现机制

前面我们对析了KVC是怎么搜索`key`的。所以如果明白了`key`的搜索顺序，是可以自己写代码实现KVC的。在考虑到集合和`keyPath`的情况下，KVC的实现会比较复杂，我们只写代码实现最普通的取值和设值即可。

```
@interface NSObject(MYKVC)
-(void)setMyValue:(id)value forKey:(NSString*)key;
-(id)myValueforKey:(NSString*)key;

@end
@implementation NSObject(MYKVC)
-(void)setMyValue:(id)value forKey:(NSString *)key{
    if (key == nil || key.length == 0) {  //key名要合法
        return;
    }
    if ([value isKindOfClass:[NSNull class]]) {
        [self setNilValueForKey:key]; //如果需要完全自定义，那么这里需要写一个setMyNilValueForKey，但是必要性不是很大，就省略了
        return;
    }
    if (![value isKindOfClass:[NSObject class]]) {
        @throw @"must be s NSObject type";
        return;
    }

    NSString* funcName = [NSString stringWithFormat:@"set%@:",key.capitalizedString];
    if ([self respondsToSelector:NSSelectorFromString(funcName)]) {  //默认优先调用set方法
        [self performSelector:NSSelectorFromString(funcName) withObject:value];
        return;
    }
    unsigned int count;
    BOOL flag = false;
    Ivar* vars = class_copyIvarList([self class], &count);
    for (NSInteger i = 0; i<count; i++) {
        Ivar var = vars[i];
        NSString* keyName = [[NSString stringWithCString:ivar_getName(var) encoding:NSUTF8StringEncoding] substringFromIndex:1];

        if ([keyName isEqualToString:[NSString stringWithFormat:@"_%@",key]]) {
            flag = true;
            object_setIvar(self, var, value);
            break;
        }

        if ([keyName isEqualToString:key]) {
            flag = true;
            object_setIvar(self, var, value);
            break;
        }
    }
    if (!flag) {
        [self setValue:value forUndefinedKey:key];//如果需要完全自定义，那么这里需要写一个self setMyValue:value forUndefinedKey:key，但是必要性不是很大，就省略了
    }
}

-(id)myValueforKey:(NSString *)key{
    if (key == nil || key.length == 0) {
        return [NSNull new]; //其实不能这么写的
    }
    //这里为了更方便，我就不做相关集合的方法查询了
    NSString* funcName = [NSString stringWithFormat:@"gett%@:",key.capitalizedString];
    if ([self respondsToSelector:NSSelectorFromString(funcName)]) {
       return [self performSelector:NSSelectorFromString(funcName)];
    }

    unsigned int count;
    BOOL flag = false;
    Ivar* vars = class_copyIvarList([self class], &count);
    for (NSInteger i = 0; i<count; i++) {
        Ivar var = vars[i];
        NSString* keyName = [[NSString stringWithCString:ivar_getName(var) encoding:NSUTF8StringEncoding] substringFromIndex:1];
        if ([keyName isEqualToString:[NSString stringWithFormat:@"_%@",key]]) {
            flag = true;
            return     object_getIvar(self, var);
            break;
        }
        if ([keyName isEqualToString:key]) {
            flag = true;
            return     object_getIvar(self, var);
            break;
        }
    }
    if (!flag) {
        [self valueForUndefinedKey:key];//如果需要完全自定义，那么这里需要写一个self myValueForUndefinedKey，但是必要性不是很大，就省略了
    }
   return [NSNull new]; //其实不能这么写的
}
@end

Address* add = [Address new];
add.country = @"China";
add.province = @"Guang Dong";
add.city = @"Shen Zhen";
add.district = @"Nan Shan";

[add setMyValue:nil forKey:@"area"];            //测试设置 nil value
[add setMyValue:@"UK" forKey:@"country"];
[add setMyValue:@"South" forKey:@"area"];
[add setMyValue:@"300169" forKey:@"postCode"];
NSLog(@"country:%@  province:%@ city:%@ postCode:%@",add.country,add.province,add.city,add._postCode);
NSString* postCode = [add myValueforKey:@"postCode"];
NSString* country = [add myValueforKey:@"country"];
NSLog(@"country:%@ postCode: %@",country,postCode);

//打印结果：

2016-04-19 14:29:39.498 KVCDemo[7273:275129] country:UK  province:South city:Shen Zhen postCode:300169
2016-04-19 14:29:39.499 KVCDemo[7273:275129] country:UK postCode: 300169

```

上面就是自己写代码实现KVC的部分功能。其中我省略了自定义KVC错误方法，省略了部分KVC搜索`key`的步骤，但是逻辑是很清晰明了的，后面的测试也符合预期。当然这只是我自己实现KVC的思路，Apple也许并不是这么做的。

## KVC的正确性验证

KVC提供了属性值,用来验证key对应的Value是否可用的方法

```
- (BOOL)validateValue:(inout id __nullable * __nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;

```

这个方法的默认实现是去探索类里面是否有一个这样的方法：`-(BOOL)validate<Key>:error:`如果有这个方法，就调用这个方法来返回，没有的话就直接返回`YES`

```
@implementation Address
-(BOOL)validateCountry:(id *)value error:(out NSError * _Nullable __autoreleasing *)outError{  //在implementation里面加这个方法，它会验证是否设了非法的value
    NSString* country = *value;
    country = country.capitalizedString;
    if ([country isEqualToString:@"Japan"]) {
        return NO;                                                                             //如果国家是日本，就返回NO，这里省略了错误提示，
    }
    return YES;
}
@end
NSError* error;
id value = @"japan";
NSString* key = @"country";
BOOL result = [add validateValue:&value forKey:key error:&error]; //如果没有重写-(BOOL)-validate<Key>:error:，默认返回Yes
if (result) {
    NSLog(@"键值匹配");
    [add setValue:value forKey:key];
}
else{
    NSLog(@"键值不匹配"); //不能设为日本，基他国家都行
}
NSString* country = [add valueForKey:@"country"];
NSLog(@"country:%@",country);
//打印结果 
2016-04-20 14:55:12.055 KVCDemo[867:58871] 键值不匹配
2016-04-20 14:55:12.056 KVCDemo[867:58871] country:China

```

如上面的代码，当开发者需要验证能不能用KVC设定某个值时，可以调用`validateValue: forKey:`这个方法来验证，如果这个类的开发者实现了`-(BOOL)validate<Key>:error:`这个方法，那么KVC就会直接调用这个方法来返回，如果没有，就直接返回`YES`，注意，KVC在设值时不会主动去做验证，需要开发者手动去验证。所以即使你在类里面写了验证方法，但是KVC因为不会去主动验证，所以还是能够设值成功。

## KVC的使用

KVC在iOS开发中是绝不可少的利器，这种基于运行时的编程方式极大地提高了灵活性，简化了代码，甚至实现很多难以想像的功能，KVC也是许多iOS开发黑魔法的基础。下面我来列举iOS开发中KVC的使用场景

#### 动态地取值和设值

利用KVC动态的取值和设值是最基本的用途了。相信每一个iOS开发者都能熟练掌握，

#### 用KVC来访问和修改私有变量

对于类里的私有属性，Objective-C是无法直接访问的，但是KVC是可以的，请参考本文前面的Dog类的例子。

#### Model和字典转换

这是KVC强大作用的又一次体现，请参考[iOS开发技巧系列---打造强大的BaseMod系列文章](https://www.jianshu.com/p/53b1e5785b24)，里面

充分地运用了KVC和Objc的`runtime`组合的技巧，只用了短短数行代码就是完成了很多功能。

#### 修改一些控件的内部属性

这也是iOS开发中必不可少的小技巧。众所周知很多UI控件都由很多内部UI控件组合而成的，但是Apple度没有提供这访问这些控件的API，这样我们就无法正常地访问和修改这些控件的样式。而KVC在大多数情况可下可以解决这个问题。最常用的就是个性化UITextField中的placeHolderText了。下面演示如果修改placeHolder的文字样式。这里的关键点是如果获取你要修改的样式的属性名，也就是key或者keyPath名。

![](http://upload-images.jianshu.io/upload_images/1464492-e1f5a56ed1b27f42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


一般情况下可以运用`runtime`来获取Apple不想开放的属性名

```
let count:UnsafeMutablePointer<UInt32> =  UnsafeMutablePointer<UInt32>()
var properties = class_copyIvarList(UITextField.self, count)
while properties.memory.debugDescription !=  "0x0000000000000000"{
    let t = ivar_getName(properties.memory)
    let n = NSString(CString: t, encoding: NSUTF8StringEncoding)
    print(n)                                                         //打印出所有属性，这里我用了Swift语言
    properties = properties.successor()
}

//上面省略了部分属性
Optional(_disabledBackgroundView)
Optional(_systemBackgroundView)
Optional(_floatingContentView)
Optional(_contentBackdropView)
Optional(_fieldEditorBackgroundView)
Optional(_fieldEditorEffectView)
Optional(_displayLabel)
Optional(_placeholderLabel)                                         //这个正是我想要修改的属性。
Optional(_dictationLabel)
Optional(_suffixLabel)
Optional(_prefixLabel)
Optional(_iconView)
//下面省略了部分属性

```

可以从里面看到其他还有很多东西可以修改，运用KVC设值可以获得自己想要的效果。

#### 操作集合

Apple对KVC的`valueForKey:`方法作了一些特殊的实现，比如说`NSArray`和`NSSet`这样的容器类就实现了这些方法。所以可以用KVC很方便地操作集合

##### 用KVC实现高阶消息传递

当对容器类使用KVC时，`valueForKey:`将会被传递给容器中的每一个对象，而不是容器本身进行操作。结果会被添加进返回的容器中，这样，开发者可以很方便的操作集合来返回另一个集合。

```
NSArray* arrStr = @[@"english",@"franch",@"chinese"];
NSArray* arrCapStr = [arrStr valueForKey:@"capitalizedString"];
for (NSString* str  in arrCapStr) {
    NSLog(@"%@",str);
}
NSArray* arrCapStrLength = [arrStr valueForKeyPath:@"capitalizedString.length"];
for (NSNumber* length  in arrCapStrLength) {
    NSLog(@"%ld",(long)length.integerValue);
}
打印结果
2016-04-20 16:29:14.239 KVCDemo[1356:118667] English
2016-04-20 16:29:14.240 KVCDemo[1356:118667] Franch
2016-04-20 16:29:14.240 KVCDemo[1356:118667] Chinese
2016-04-20 16:29:14.240 KVCDemo[1356:118667] 7
2016-04-20 16:29:14.241 KVCDemo[1356:118667] 6
2016-04-20 16:29:14.241 KVCDemo[1356:118667] 7

```

方法`capitalizedString`被传递到NSArray中的每一项，这样，NSArray的每一员都会执行`capitalizedString`并返回一个包含结果的新的NSArray。从打印结果可以看出，所有`String`都成功以转成了大写。

同样如果要执行多个方法也可以用`valueForKeyPath:`方法。它先会对每一个成员调用 `capitalizedString`方法，然后再调用`length`，因为`lenth`方法返回是一个数字，所以返回结果以`NSNumber`的形式保存在新数组里。

##### 用KVC中的函数操作集合

KVC同时还提供了很复杂的函数，主要有下面这些

①简单集合运算符

简单集合运算符共有`@avg， @count ， @max ， @min ，@sum5`种，都表示啥不用我说了吧， 目前还不支持自定义。

```
@interface Book : NSObject
@property (nonatomic,copy)  NSString* name;
@property (nonatomic,assign)  CGFloat price;
@end
@implementation Book
@end

Book *book1 = [Book new];
book1.name = @"The Great Gastby";
book1.price = 22;
Book *book2 = [Book new];
book2.name = @"Time History";
book2.price = 12;
Book *book3 = [Book new];
book3.name = @"Wrong Hole";
book3.price = 111;

Book *book4 = [Book new];
book4.name = @"Wrong Hole";
book4.price = 111;

NSArray* arrBooks = @[book1,book2,book3,book4];
NSNumber* sum = [arrBooks valueForKeyPath:@"@sum.price"];
NSLog(@"sum:%f",sum.floatValue);
NSNumber* avg = [arrBooks valueForKeyPath:@"@avg.price"];
NSLog(@"avg:%f",avg.floatValue);
NSNumber* count = [arrBooks valueForKeyPath:@"@count"];
NSLog(@"count:%f",count.floatValue);
NSNumber* min = [arrBooks valueForKeyPath:@"@min.price"];
NSLog(@"min:%f",min.floatValue);
NSNumber* max = [arrBooks valueForKeyPath:@"@max.price"];
NSLog(@"max:%f",max.floatValue);

打印结果
2016-04-20 16:45:54.696 KVCDemo[1484:127089] sum:256.000000
2016-04-20 16:45:54.697 KVCDemo[1484:127089] avg:64.000000
2016-04-20 16:45:54.697 KVCDemo[1484:127089] count:4.000000
2016-04-20 16:45:54.697 KVCDemo[1484:127089] min:12.000000
2016-04-20 16:45:54.697 KVCDemo[1484:127089] max:111.000000

```

②对象运算符

比集合运算符稍微复杂，能以数组的方式返回指定的内容，一共有两种：

`@distinctUnionOfObjects`

`@unionOfObjects`

它们的返回值都是NSArray，区别是前者返回的元素都是唯一的，是去重以后的结果；后者返回的元素是全集。

用法如下：

```
NSLog(@"distinctUnionOfObjects");
NSArray* arrDistinct = [arrBooks valueForKeyPath:@"@distinctUnionOfObjects.price"];
for (NSNumber *price in arrDistinct) {
    NSLog(@"%f",price.floatValue);
}
NSLog(@"unionOfObjects");
NSArray* arrUnion = [arrBooks valueForKeyPath:@"@unionOfObjects.price"];
for (NSNumber *price in arrUnion) {
    NSLog(@"%f",price.floatValue);
}

2016-04-20 16:47:34.490 KVCDemo[1522:128840] distinctUnionOfObjects
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 111.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 12.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 22.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] unionOfObjects
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 22.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 12.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 111.000000
2016-04-20 16:47:34.490 KVCDemo[1522:128840] 111.000000

```

前者会将重复的价格去除后返回所有价格，后者直接返回所有的图书价格。(因为只返回价格，没有返回图书，感觉用处不大。)

③Array和Set操作符

这种情况更复杂了，说的是集合中包含集合的情况，我们执行了如下的一段代码：

@distinctUnionOfArrays

@unionOfArrays

@distinctUnionOfSets

`@distinctUnionOfArrays：`该操作会返回一个数组，这个数组包含不同的对象，不同的对象是在从关键路径到操作器右边的被指定的属性里

`@unionOfArrays` 该操作会返回一个数组，这个数组包含的对象是在从关键路径到操作器右边的被指定的属性里和@distinctUnionOfArrays不一样，重复的对象不会被移除

`@distinctUnionOfSets` 和`@distinctUnionOfArrays`类似。因为`Set`本身就不支持重复。

#### KVO

你没看错，KVO是基于KVC实现的。那么是怎么用KVC实现KVO的呢，~~请期待下章~~。

## 总结

本文全方位介绍了KVC的原理和各种用法。相信读者看完后对会KVC会有更完全的理解，也会在项目里更好的运用KVC。其实这里面所有的东西在官方文档里都有详细的讲解说明。只不过全是英文的，我也看过几遍，但是英语不好会看得很吃力，比如官方在介绍@distinctUnionOfArrays时的那句话我想了很好久也不是很明白，而且官方的示例代码也做得不够好，所以很难找出某些功能的适用场景。但我还是推荐各位开发者能够学好英语去看官方文档。再结合StackOverFlow和Google。真的可以解决绝大多数开发中碰到的难题了。这篇文章就到这里，~~下篇我向大家介绍KVO~~。 现在网上KVO的文章非常多，质量也不错，读者可以搜索阅读。

