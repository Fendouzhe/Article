Runtime如果在动态方法解析阶段没有实现`(BOOL)resolveInstanceMethod:(SEL)sel`处理消息的话就会进入消息转发阶段：首先会调用`(id)forwardingTargetForSelector:(SEL)aSelector`看看是否有返回处理对象,有的话就交给返回对象处理，没有的话就调用`(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`返回方法签名，然后在`(void)forwardInvocation:(NSInvocation *)anInvocation`处理消息。如果`methodSignatureForSelector:`也没有返回方法签名的话就会报一个`unrecognized selector to instance`的错误；
消息转发作用：我们项目中可以在UIControl分类添加上述代码，可以有效降低unrecognized selector崩溃率，只是我们实际开发中很少会这么干的。

####LRStudent.h
```
@interface LRStudent : NSObject
- (void)test;
- (void)testArgument:(int)age;
@end
```

####LRStudent.m
```
#import "LRStudent.h"

@implementation LRStudent

- (void)test{
    NSLog(@"%s",__func__);
}
- (void)testArgument:(int)age{
    NSLog(@"%s age = %d",__func__,age);
}

@end
```

####LRPerson.h
```
@interface LRPerson : NSObject
- (void)run;
- (void)test;
- (void)testArgument:(int)age;
@end
```
####LRPerson.m
```
#import "LRPerson.h"
#import "LRStudent.h"

@implementation LRPerson

- (void)run
{
    NSLog(@"%s",__func__);
}


/**
 方式1 直接返回处理对象，但是不能自定义
 */
//返回消息处理对象
//- (id)forwardingTargetForSelector:(SEL)aSelector{
//    /*
//    if (aSelector == @selector(test) || aSelector == @selector(testArgument:)) {
//        return [[LRStudent alloc] init];
//    }
//    return [super forwardingTargetForSelector:aSelector];
//     */
//    return [[LRStudent alloc] init];
//}


/**
 方式2 返回方法签名和实现(void)forwardInvocation:(NSInvocation *)anInvocation
 在里面可以自定义处理消息
 */
//返回方法签名：返回值类型、参数类型
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    // 有实现的方法
    if ([self respondsToSelector:aSelector]) {
        return [super methodSignatureForSelector:aSelector];
    }
    // 找不到的方法
    return [NSMethodSignature signatureWithObjCTypes:"v@:"];
}
// 来这里处理消息转发，也可以什么都不做
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    /* 1.获取参数
    // 参数顺序：receiver、selector、other arguments
    int age;
    [anInvocation getArgument:&age atIndex:2];//获取参数值
    NSLog(@"%d", age + 10);
     */
    
    /* 2.给消息转发对象处理
        [anInvocation invokeWithTarget:[[LRStudent alloc] init]];
     */
    //3.什么都不做 降低unrecognized selector崩溃率
    NSLog(@"调用者：%@的%@方法没有找到",[anInvocation.target class],NSStringFromSelector(anInvocation.selector));
    
}

@end
```
####调用：
```
LRPerson *person = [[LRPerson alloc] init];
[person run];
[person test];
[person testArgument:10];
```
####运行打印：
```
打印2：
 -[LRPerson run]
 -[LRStudent test]
 -[LRStudent testArgument:] age = 10

打印3：
 -[LRPerson run]
 调用者：LRPerson的test方法没有找到
 调用者：LRPerson的testArgument:方法没有找到
```
