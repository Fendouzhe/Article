oc发送消息时，在类对象和父类中都没有找到方法实现就会进入动态方法解析：
```
#import <Foundation/Foundation.h>

@interface LRPerson : NSObject

- (void)instanceMethodTest0;

- (void)instanceMethodTest1;

+ (void)classMethodTest0;

+ (void)classMethodTest1;

@end
```

```
#import "LRPerson.h"
#import <objc/runtime.h>

@implementation LRPerson

+ (void)oc_classMethod{
    NSLog(@"%s", __func__);
}

- (void)oc_instanceMethod{
    NSLog(@"%s", __func__);
}

void c_method(id self, SEL _cmd)
{
    NSLog(@"c_other - %@ - %@", self, NSStringFromSelector(_cmd));
}

///实例方法消息转发
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(instanceMethodTest0)) {///调用c语言函数
        // 动态添加test方法的实现
        // "v16@0:8" v代表void返回类型，16代表参数占用总字节数，@代表id类型，0代表参数self从0字节开始，“:”代表SEL类型，8代表_cmd从第8个字节开始
        class_addMethod(self, sel, (IMP)c_method, "v16@0:8");
        // 返回YES代表有动态添加方法
        return YES;
    }else if (sel == @selector(instanceMethodTest1)){///调用oc函数
        
        Method method = class_getInstanceMethod(self, @selector(oc_instanceMethod));
        // 给类对象添加instanceMethodTest1方法，使用oc_instanceMethod的方法实现和方法类型
        class_addMethod(self, sel, method_getImplementation(method), method_getTypeEncoding(method));
        
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

///类方法消息转发
+ (BOOL)resolveClassMethod:(SEL)sel
{
    if (sel == @selector(classMethodTest0)) {///调用c语言函数
        
        // 第一个参数是获取元类object_getClass(self) 类方法是添加到元类中
        // "v16@0:8" v代表void返回类型，16代表参数占用总字节数，@代表id类型，0代表参数self从0字节开始，“:”代表SEL类型，8代表_cmd从第8个字节开始
        class_addMethod(object_getClass(self), sel, (IMP)c_method, "v16@0:8");
        // 返回YES代表有动态添加方法
        return YES;
        
    }else if (sel == @selector(classMethodTest1)) {///调用oc函数
        
        //Method method = class_getInstanceMethod(self, @selector(oc_instanceMethod));        
        //class_addMethod(self, sel, method_getImplementation(method), method_getTypeEncoding(method));
        Method method = class_getClassMethod(self, @selector(oc_classMethod));
        // 第一个参数是获取元类object_getClass(self) 类方法是添加到元类中
        class_addMethod(object_getClass(self), sel, method_getImplementation(method), method_getTypeEncoding(method));
        return YES;
    }
    return [super resolveClassMethod:sel];
}


//+ (BOOL)resolveInstanceMethod:(SEL)sel
//{
//    if (sel == @selector(instanceMethodTest0)) {
//        // 获取其他方法
//        Method method = class_getInstanceMethod(self, @selector(oc_instanceMethod));
//
//        // 动态添加test方法的实现
//        class_addMethod(self, sel,
//                        method_getImplementation(method),
//                        method_getTypeEncoding(method));
//
//        // 返回YES代表有动态添加方法
//        return YES;
//    }
//    return [super resolveInstanceMethod:sel];
//}

// typedef struct objc_method *Method;
// struct objc_method == struct method_t
//        struct method_t *otherMethod = (struct method_t *)class_getInstanceMethod(self, @selector(oc_instanceMethod));

struct method_t {
    SEL sel;
    char *types;
    IMP imp;
};

//+ (BOOL)resolveInstanceMethod:(SEL)sel
//{
//    if (sel == @selector(instanceMethodTest0)) {
//        // 获取其他方法
//        struct method_t *method = (struct method_t *)class_getInstanceMethod(self, @selector(oc_instanceMethod));
//
//        // 动态添加test方法的实现
//        class_addMethod(self, sel, method->imp, method->types);
//
//        // 返回YES代表有动态添加方法
//        return YES;
//    }
//    return [super resolveInstanceMethod:sel];
//}

@end
```

![Snip20180709_3.png](https://upload-images.jianshu.io/upload_images/1464492-831563811d5770bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
