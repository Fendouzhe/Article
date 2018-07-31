我们在开发APP的时候可能都会遇到在一个页面请求多个接口的情况，如果在每个接口请求成功后都刷新UI可能会出现闪屏的情况，也可能出现由于数据源不完整导致刷新UI时程序崩溃。那么该如何保证这多个请求之间异步执行且只有在所有请求都成功返回后再进行UI刷新呢？在网上搜索了很多例子基本都是用```dispatch_group_async```、```dispatch_group_t```与```dispatch_group_notify``` 组合来实现的。这种方式只是做对了一半，其实现如下：
```
dispatch_group_t  group = dispatch_group_create();
dispatch_queue_t  queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_async(group, queue, ^{
        NSLog(@"1");
    });
dispatch_group_async(group, queue, ^{
        NSLog(@"2");
    });
dispatch_group_async(group, queue, ^{
       NSLog(@"3");
    });
 dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"4");
    });
```
程序运行之后1、2、3先异步执行，然后1、2、3都返回后 4 再最后执行。到这里看似满足了 所有请求都返回后再刷新UI 的要求，但是当我按照这个思路来实现多个网络请求时，发现1、2、3、4的顺序却完全打乱了，4并不总是在最后执行。经过多次调试后才发现原因是因为上面每一个```NSLog(...)```本身都是同步执行的，而我想要的网路请求本身却是个异步操作。如下

```
dispatch_group_async(group, queue, ^{
       [[SFAPIManager sharedManager] requestDataWithPath:urlPath params:@{@"id":@"1"} completeBlock:^(id result, NSError *error) {
        NSLog(@"-------%@",result);
       }];
 });
```

这会导致在网络请求未回来之前block就已经提前返回了，所以以上代码实现是错误的。正确的方法应该是以上三个函数再配合```dispatch_group_enter(group)```和```dispatch_group_leave(group)```两个函数一起来使用，这样才能实现我们想要的最终效果。

**dispatch_group_enter(dispatch_group_t group);**
参数group不能为空，在异步任务开始前调用。它明确的表明了一个 block 被加入到了队列组group中，此时group中的任务的引用计数会加1(类似于OC的内存管理)，```dispatch_group_enter(group)```必须与```dispatch_group_leave(group)```配对使用，它们可以在使用```dispatch_group_async```时帮助你合理的管理队列组中任务的引用计数的增加与减少。

**dispatch_group_leave(dispatch_group_t group);**
参数group不能为空，在异步任务成功返回后调用。它明确的表明了队列组里的一个 block 已经执行完成，队列组中的任务的引用计数会减1，它必须与```dispatch_group_enter(group)```配对使用，```dispatch_group_leave(group)```的调用次数不能多于```dispatch_group_enter(group)```的调用次数。

当队列组里的任务的引用计数等于0时，会调用```dispatch_group_notify```函数。具体代码实现如下：

```
@property (nonatomic,strong) dispatch_group_t disGroup;

- (void)requestDatas {
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_enter(self.disGroup);
    dispatch_group_enter(self.disGroup);
    dispatch_group_enter(self.disGroup);
    
    dispatch_group_async(self.disGroup, queue, ^{
        [self requestHomeWorks];
    });
    
    dispatch_group_async(self.disGroup, queue, ^{
        [self requestHomeBanner];
    });
    
    dispatch_group_async(self.disGroup, queue, ^{
        [self requestHomeAdvInfos];
    });
       
    dispatch_group_notify(self.disGroup, dispatch_get_main_queue(), ^{
        NSLog(@"4");
        [self.collectionView reloadData];
    });
}
```
```
- (void)requestHomeWorks {
    [[SFAPIManager sharedManager] requestDataWithPath:kHomeWorksPath params:nil completeBlock:^(id result, NSError *error) {
        NSLog(@"1");
        dispatch_group_leave(self.disGroup);
    }];
}
- (void) requestHomeBanner {
    [[SFAPIManager sharedManager] requestDataWithPath:kHomeBannerPath params:nil completeBlock:^(id result, NSError *error) {
        NSLog(@"2");
        dispatch_group_leave(self.disGroup);
    }];
}
- (void) requestHomeAdvInfos {
    [[SFAPIManager sharedManager] requestDataWithPath:kHomeAdvInfosPath params:nil completeBlock:^(id result, NSError *error) {
        NSLog(@"3");
        dispatch_group_leave(self.disGroup);
    }];
}
```

这样就可以实现多个网络请求之间异步执行，且只有在所有请求都返回后才会执行UI刷新的效果。
