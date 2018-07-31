###第一种：key传地址
```
static const void *MJNameKey = &LRNameKey;
static const void *MJWeightKey = &LRWeightKey;
- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, LRNameKey, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name
{
    return objc_getAssociatedObject(self, LRNameKey);
}

- (void)setWeight:(int)weight
{
    objc_setAssociatedObject(self, LRWeightKey, @(weight), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (int)weight
{
    return [objc_getAssociatedObject(self, LRWeightKey) intValue];
}

```

###第二种：key传字符地址
```
// char只占用一个字节
static const char LRNameKey;
static const char LRWeightKey;
- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, &LRNameKey, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name
{
    return objc_getAssociatedObject(self, &LRNameKey);
}

- (void)setWeight:(int)weight
{
    objc_setAssociatedObject(self, &LRWeightKey, @(weight), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (int)weight
{
    return [objc_getAssociatedObject(self, &LRWeightKey) intValue];
}
```

###第三种：key传字符串常量地址
```
#define LRNameKey @"name"
#define LRWeightKey @"weight"
- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, LRNameKey, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name
{
    return objc_getAssociatedObject(self, LRNameKey);
}

- (void)setWeight:(int)weight
{
    objc_setAssociatedObject(self, LRWeightKey, @(weight), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (int)weight
{
    return [objc_getAssociatedObject(self, LRWeightKey) intValue];
}

```

###第四种：最优方案 key传方法地址
```
- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name
{
    // _cmd隐式参数 _cmd == @selector(name)
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setWeight:(int)weight
{
    objc_setAssociatedObject(self, @selector(weight), @(weight), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (int)weight
{
     // _cmd隐式参数 _cmd == @selector(weight)
    return [objc_getAssociatedObject(self, _cmd) intValue];
}
```
