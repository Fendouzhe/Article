####LRStudent
```
@interface LRStudent : NSObject

+ (void)test;
- (void)test;

@end
```

```
#import "LRStudent.h"

@implementation LRStudent

+ (void)test
{
    NSLog(@"%s", __func__);
}

- (void)test
{
    NSLog(@"%s", __func__);
}
@end
```
####LRPerson
```
@interface LRPerson : NSObject

+ (void)test;

@end
```

```
#import "LRPerson.h"
#import <objc/runtime.h>
#import "LRStudent.h"

@implementation LRPerson

//+ (id)forwardingTargetForSelector:(SEL)aSelector
//{
//    //返回实例对象会调用LRStudent的实例方法 -(void)test
//    //if (aSelector == @selector(test)) return [[LRStudent alloc] init];
//    //返回类对象会调用LRStudent的类方法 +(void)test
//    if (aSelector == @selector(test)) return [LRStudent class];
//    return [super forwardingTargetForSelector:aSelector];
//}

+ (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    if (aSelector == @selector(test)) return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    
    return [super methodSignatureForSelector:aSelector];
}

+ (void)forwardInvocation:(NSInvocation *)anInvocation
{
    /* 1.获取参数
     // 参数顺序：receiver、selector、other arguments
     int age;
     [anInvocation getArgument:&age atIndex:2];//获取参数值
     NSLog(@"%d", age + 10);
     */
    
    //1.什么都不做 降低unrecognized selector崩溃率
    //NSLog(@"调用者：%@的%@方法没有找到",[anInvocation.target class],NSStringFromSelector(anInvocation.selector));
    
    //2.给消息转发对象处理
    //2.1 target传实例对象调用的是LRStudent的实例方法 -(void)test
    //[anInvocation invokeWithTarget:[[LRStudent alloc] init]];
    //2.2 target传类对象调用的是LRStudent的类方法 +(void)test
    [anInvocation invokeWithTarget:[LRStudent class]];
    
}

@end
```
####调用
```
[LRPerson test];
```
####控制台打印输出
```
2.1打印：
-[LRStudent test]
2.2打印：
+[LRStudent test]
```
