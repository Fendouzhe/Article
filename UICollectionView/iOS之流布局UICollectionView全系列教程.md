### 一、简介

 UICollectionView是iOS6之后引入的一个新的UI控件，它和UITableView有着诸多的相似之处，其中许多代理方法都十分类似。简单来说，UICollectionView是比UITbleView更加强大的一个UI控件，有如下几个方面：

1、支持水平和垂直两种方向的布局

2、通过layout配置方式进行布局

3、类似于TableView中的cell特性外，CollectionView中的Item大小和位置可以自由定义

4、通过layout布局回调的代理方法，可以动态的定制每个item的大小和collection的大体布局属性

5、更加强大一点，完全自定义一套layout布局方案，可以实现意想不到的效果

这篇博客，我们主要讨论CollectionView使用原生layout的方法和相关属性，其他特点和更强的制定化，会在后面的博客中介绍

### 二、先来实现一个最简单的九宫格类布局

在了解UICollectionView的更多属性前，我们先来使用其进行一个最简单的流布局试试看，在controller的viewDidLoad中添加如下代码：

```
//创建一个layout布局类
UICollectionViewFlowLayout * layout = [[UICollectionViewFlowLayout alloc]init];
//设置布局方向为垂直流布局
layout.scrollDirection = UICollectionViewScrollDirectionVertical;
//设置每个item的大小为100*100
layout.itemSize = CGSizeMake(100, 100);
//创建collectionView 通过一个布局策略layout来创建
UICollectionView * collect = [[UICollectionView alloc]initWithFrame:self.view.frame collectionViewLayout:layout];
//代理设置
collect.delegate=self;
collect.dataSource=self;
//注册item类型 这里使用系统的类型
[collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
   
[self.view addSubview:collect];
```

这里有一点需要注意，collectionView在完成代理回调前，必须注册一个cell，类似如下:

```
[collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
```

这和tableView有些类似，又有些不同，因为tableView除了注册cell的方法外，还可以通过临时创建来做：

```

//tableView在从复用池中取cell的时候，有如下两种方法
//使用这种方式如果复用池中无，是可以返回nil的，我们在临时创建即可
- (nullable __kindof UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier;
//6.0后使用如下的方法直接从注册的cell类获取创建，如果没有注册 会崩溃
- (__kindof UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);

```

我们可以分析：因为UICollectionView是iOS6.0之前的新类，因此这里统一了从复用池中获取cell的方法，没有再提供可以返回nil的方式，并且在UICollectionView的回调代理中，只能使用从复用池中获取cell的方式进行cell的返回，其他方式会崩溃，例如：

```

//这是正确的方法
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
 
//这样做会崩溃
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
//    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
//    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    UICollectionViewCell * cell = [[UICollectionViewCell alloc]init];
    return cell;
}
```

上面错误的方式会崩溃，信息如下，让我们使用从复用池中取cell的方式：
![](https://upload-images.jianshu.io/upload_images/9610202-712b2926444778f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的设置完成后，我们来实现如下几个代理方法：

这里与TableView的回调方式十分类似

```

//返回分区个数
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
//返回每个分区的item个数
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
//返回每个item
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-4371fa9269a29d59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


同样，如果内容的大小超出一屏，和tableView类似是可以进行视图滑动的。

还有一点细节，我们在上面设置布局方式的时候设置了垂直布局：

```
layout.scrollDirection = UICollectionViewScrollDirectionVertical;
//这个是水平布局
//layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
```

这样系统会在一行充满后进行第二行的排列，如果设置为水平布局，则会在一列充满后，进行第二列的布局，这种方式也被称为流式布局

### 三、UICollectionView中的常用方法和属性

```
//通过一个布局策略初识化CollectionView
- (instancetype)initWithFrame:(CGRect)frame collectionViewLayout:(UICollectionViewLayout *)layout;
 
//获取和设置collection的layout
@property (nonatomic, strong) UICollectionViewLayout *collectionViewLayout;
 
//数据源和代理
@property (nonatomic, weak, nullable) id <UICollectionViewDelegate> delegate;
@property (nonatomic, weak, nullable) id <UICollectionViewDataSource> dataSource;
 
//从一个class或者xib文件进行cell(item)的注册
- (void)registerClass:(nullable Class)cellClass forCellWithReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forCellWithReuseIdentifier:(NSString *)identifier;
 
//下面两个方法与上面相似，这里注册的是头视图或者尾视图的类
//其中第二个参数是设置 头视图或者尾视图 系统为我们定义好了这两个字符串
//UIKIT_EXTERN NSString *const UICollectionElementKindSectionHeader NS_AVAILABLE_IOS(6_0);
//UIKIT_EXTERN NSString *const UICollectionElementKindSectionFooter NS_AVAILABLE_IOS(6_0);
- (void)registerClass:(nullable Class)viewClass forSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forSupplementaryViewOfKind:(NSString *)kind withReuseIdentifier:(NSString *)identifier;
 
//这两个方法是从复用池中取出cell或者头尾视图
- (__kindof UICollectionViewCell *)dequeueReusableCellWithReuseIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath;
- (__kindof UICollectionReusableView *)dequeueReusableSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier forIndexPath:(NSIndexPath *)indexPath;
 
//设置是否允许选中 默认yes
@property (nonatomic) BOOL allowsSelection;
 
//设置是否允许多选 默认no
@property (nonatomic) BOOL allowsMultipleSelection;
 
//获取所有选中的item的位置信息
- (nullable NSArray<NSIndexPath *> *)indexPathsForSelectedItems; 
 
//设置选中某一item，并使视图滑动到相应位置，scrollPosition是滑动位置的相关参数，如下：
/*
typedef NS_OPTIONS(NSUInteger, UICollectionViewScrollPosition) {
    //无
    UICollectionViewScrollPositionNone                 = 0,
    //垂直布局时使用的 对应上中下
    UICollectionViewScrollPositionTop                  = 1 << 0,
    UICollectionViewScrollPositionCenteredVertically   = 1 << 1,
    UICollectionViewScrollPositionBottom               = 1 << 2,
    //水平布局时使用的  对应左中右
    UICollectionViewScrollPositionLeft                 = 1 << 3,
    UICollectionViewScrollPositionCenteredHorizontally = 1 << 4,
    UICollectionViewScrollPositionRight                = 1 << 5
};
*/
- (void)selectItemAtIndexPath:(nullable NSIndexPath *)indexPath animated:(BOOL)animated scrollPosition:(UICollectionViewScrollPosition)scrollPosition;
 
//将某一item取消选中
- (void)deselectItemAtIndexPath:(NSIndexPath *)indexPath animated:(BOOL)animated;
 
//重新加载数据
- (void)reloadData;
 
//下面这两个方法，可以重新设置collection的布局，后面的方法多了一个布局完成后的回调，iOS7后可以用
//使用这两个方法可以产生非常炫酷的动画效果
- (void)setCollectionViewLayout:(UICollectionViewLayout *)layout animated:(BOOL)animated;
- (void)setCollectionViewLayout:(UICollectionViewLayout *)layout animated:(BOOL)animated completion:(void (^ __nullable)(BOOL finished))completion NS_AVAILABLE_IOS(7_0);
 
//下面这些方法更加强大，我们可以对布局更改后的动画进行设置
//这个方法传入一个布局策略layout，系统会开始进行布局渲染，返回一个UICollectionViewTransitionLayout对象
//这个UICollectionViewTransitionLayout对象管理动画的相关属性，我们可以进行设置
- (UICollectionViewTransitionLayout *)startInteractiveTransitionToCollectionViewLayout:(UICollectionViewLayout *)layout completion:(nullable UICollectionViewLayoutInteractiveTransitionCompletion)completion NS_AVAILABLE_IOS(7_0);
//准备好动画设置后，我们需要调用下面的方法进行布局动画的展示，之后会调用上面方法的block回调
- (void)finishInteractiveTransition NS_AVAILABLE_IOS(7_0);
//调用这个方法取消上面的布局动画设置，之后也会进行上面方法的block回调
- (void)cancelInteractiveTransition NS_AVAILABLE_IOS(7_0);
 
//获取分区数
- (NSInteger)numberOfSections;
 
//获取某一分区的item数
- (NSInteger)numberOfItemsInSection:(NSInteger)section;
 
//下面两个方法获取item或者头尾视图的layout属性，这个UICollectionViewLayoutAttributes对象
//存放着布局的相关数据，可以用来做完全自定义布局，后面博客会介绍
- (nullable UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath;
- (nullable UICollectionViewLayoutAttributes *)layoutAttributesForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath;
 
//获取某一点所在的indexpath位置
- (nullable NSIndexPath *)indexPathForItemAtPoint:(CGPoint)point;
 
//获取某个cell所在的indexPath
- (nullable NSIndexPath *)indexPathForCell:(UICollectionViewCell *)cell;
 
//根据indexPath获取cell
- (nullable UICollectionViewCell *)cellForItemAtIndexPath:(NSIndexPath *)indexPath;
 
//获取所有可见cell的数组
- (NSArray<__kindof UICollectionViewCell *> *)visibleCells;
 
//获取所有可见cell的位置数组
- (NSArray<NSIndexPath *> *)indexPathsForVisibleItems;
 
//下面三个方法是iOS9中新添加的方法，用于获取头尾视图
- (UICollectionReusableView *)supplementaryViewForElementKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(9_0);
- (NSArray<UICollectionReusableView *> *)visibleSupplementaryViewsOfKind:(NSString *)elementKind NS_AVAILABLE_IOS(9_0);
- (NSArray<NSIndexPath *> *)indexPathsForVisibleSupplementaryElementsOfKind:(NSString *)elementKind NS_AVAILABLE_IOS(9_0);
 
//使视图滑动到某一位置，可以带动画效果
- (void)scrollToItemAtIndexPath:(NSIndexPath *)indexPath atScrollPosition:(UICollectionViewScrollPosition)scrollPosition animated:(BOOL)animated;
 
//下面这些方法用于动态添加，删除，移动某些分区获取items
- (void)insertSections:(NSIndexSet *)sections;
- (void)deleteSections:(NSIndexSet *)sections;
- (void)reloadSections:(NSIndexSet *)sections;
- (void)moveSection:(NSInteger)section toSection:(NSInteger)newSection;
 
- (void)insertItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)deleteItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)reloadItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths;
- (void)moveItemAtIndexPath:(NSIndexPath *)indexPath toIndexPath:(NSIndexPath *)newIndexPath;

```

## iOS流布局UICollectionView系列二——UICollectionView的代理方法

### 一、引言

在上一篇博客中，介绍了最基本的UICollectionView的使用和其中我们常用的属性和方法，也介绍了瀑布流布局的过程与思路，这篇博客来讨论关于UICollectionView的代理方法的使用。

###二、UICollectionViewDataSource协议

这个协议主要用于collectionView相关数据的处理，包含方法如下：

首先，有两个方法是我们必须实现的：

设置每个分区的Item个数

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section;

设置返回每个item的属性

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;

下面的方法是可选实现的：

虽然这个方法是可选的，一般我们都会去实现，设置分区数

- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView;

对头视图或者尾视图进行设置

- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath;

设置某个item是否可以被移动，返回NO则不能移动

- (BOOL)collectionView:(UICollectionView *)collectionView canMoveItemAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(9_0);

移动item的时候，会调用这个方法

- (void)collectionView:(UICollectionView *)collectionView moveItemAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath*)destinationIndexPath；

### 三、UICollectionViewDelegate协议

 这个协议用来设置和处理collectionView的功能和一些逻辑，所有方法都是可选实现：

是否允许某个Item的高亮，返回NO，则不能进入高亮状态

- (BOOL)collectionView:(UICollectionView *)collectionView shouldHighlightItemAtIndexPath:(NSIndexPath *)indexPath;

当item高亮时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didHighlightItemAtIndexPath:(NSIndexPath *)indexPath;

结束高亮状态时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didUnhighlightItemAtIndexPath:(NSIndexPath *)indexPath;

是否可以选中某个Item，返回NO，则不能选中

- (BOOL)collectionView:(UICollectionView *)collectionView shouldSelectItemAtIndexPath:(NSIndexPath *)indexPath;

是否可以取消选中某个Item

- (BOOL)collectionView:(UICollectionView *)collectionView shouldDeselectItemAtIndexPath:(NSIndexPath *)indexPath;

已经选中某个item时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath;

取消选中某个Item时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didDeselectItemAtIndexPath:(NSIndexPath *)indexPath;

将要加载某个Item时调用的方法

- (void)collectionView:(UICollectionView *)collectionView willDisplayCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(8_0);

将要加载头尾视图时调用的方法

- (void)collectionView:(UICollectionView *)collectionView willDisplaySupplementaryView:(UICollectionReusableView *)view forElementKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(8_0);

已经展示某个Item时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didEndDisplayingCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath;

已经展示某个头尾视图时触发的方法

- (void)collectionView:(UICollectionView *)collectionView didEndDisplayingSupplementaryView:(UICollectionReusableView *)view forElementOfKind:(NSString *)elementKind atIndexPath:(NSIndexPath *)indexPath;

这个方法设置是否展示长按菜单

- (BOOL)collectionView:(UICollectionView *)collectionView shouldShowMenuForItemAtIndexPath:(NSIndexPath *)indexPath;

长按菜单中可以触发一下类复制粘贴的方法，效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-ef8d775b6998cd7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个方法用于设置要展示的菜单选项

- (BOOL)collectionView:(UICollectionView *)collectionView canPerformAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender;

这个方法用于实现点击菜单按钮后的触发方法,通过测试，只有copy，cut和paste三个方法可以使用

- (void)collectionView:(UICollectionView *)collectionView performAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender;

通过下面的方式可以将点击按钮的方法名打印出来：

```
-(void)collectionView:(UICollectionView *)collectionView performAction:(SEL)action forItemAtIndexPath:(NSIndexPath *)indexPath withSender:(id)sender{    
    NSLog(@"%@",NSStringFromSelector(action));
}
```

collectionView进行重新布局时调用的方法

- (nonnull UICollectionViewTransitionLayout *)collectionView:(UICollectionView *)collectionView transitionLayoutForOldLayout:(UICollectionViewLayout *)fromLayout newLayout:(UICollectionViewLayout *)toLayout;

## iOS流布局UICollectionView系列三——使用FlowLayout进行更灵活布局

### 一、引言

前面的博客介绍了UICollectionView的相关方法和其协议中的方法，但对布局的管理类UICollectionViewFlowLayout没有着重探讨，这篇博客介绍关于布局的相关设置和属性方法。通过layout的设置，我们可以编写更加灵活的布局效果。

### 二、将九宫格式的布局进行升级

 在第一篇博客中，通过UICollectionView，我们很轻松的完成了一个九宫格的布局，但是如此中规中矩的布局方式，有时候并不能满足我们的需求，有时我们需要每一个Item展示不同的大小，代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc]init];
    layout.scrollDirection = UICollectionViewScrollDirectionVertical;
    UICollectionView *collect = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
  ;
    [self.view addSubview:collect];
    
    
}
//设置每个item的大小，双数的为50*50 单数的为100*100
-(CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath{
    if (indexPath.row%2==0) {
        return CGSizeMake(50, 50);
    }else{
        return CGSizeMake(100, 100);
    }
}
 
//代理相应方法
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 100;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-7a333d4371bea6e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


现在的布局效果是不是炫酷了许多。

### 三、UICollectionViewFlowLayout相关属性方法

UICollectionViewFlowLayout是系统提供给我们一个封装好的流布局设置类，其中有一些布局属性我们可以进行设置： 

设置行与行之间的间距最小距离

[@property](http://my.oschina.net/property) (nonatomic) CGFloat minimumLineSpacing;

设置列与列之间的间距最小距离

[@property](http://my.oschina.net/property) (nonatomic) CGFloat minimumInteritemSpacing;

设置每个item的大小

[@property](http://my.oschina.net/property) (nonatomic) CGSize itemSize;

设置每个Item的估计大小，一般不需要设置

[@property](http://my.oschina.net/property) (nonatomic) CGSize estimatedItemSize NS_AVAILABLE_IOS(8_0);

设置布局方向

[@property](http://my.oschina.net/property) (nonatomic) UICollectionViewScrollDirection scrollDirection;

这个UICollectionViewScrollDirection的枚举如下：

```
typedef NS_ENUM(NSInteger, UICollectionViewScrollDirection) {    
    UICollectionViewScrollDirectionVertical,//水平布局    
    UICollectionViewScrollDirectionHorizontal//垂直布局
};
```

设置头视图尺寸大小

@property (nonatomic) CGSize headerReferenceSize;

设置尾视图尺寸大小

@property (nonatomic) CGSize footerReferenceSize;

设置分区的EdgeInset

@property (nonatomic) UIEdgeInsets sectionInset;

这个属性可以设置分区的偏移量，例如我们在刚才的例子中添加如下设置：

```
 layout.sectionInset = UIEdgeInsetsMake(20, 20, 20, 20);
```

效果如下，会看到分区的边界闪出了20像素

![](https://upload-images.jianshu.io/upload_images/9610202-29440db6d51f027e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面这两个方法设置分区的头视图和尾视图是否始终固定在屏幕上边和下边

 @property (nonatomic) BOOL sectionHeadersPinToVisibleBounds NS_AVAILABLE_IOS(9_0);

@property (nonatomic) BOOL sectionFootersPinToVisibleBounds NS_AVAILABLE_IOS(9_0);

### 四、动态的配置layout的相关属性UICollectionViewDelegateFlowLayout

上面的方法在创建FlowLayout时静态的进行设置，如果我们需要动态的设置这些属性，就像我们例子中的，每个item的大小会有差异，我们可以通过代理来实现。

UICollectionViewDelegateFlowLayout是UICollectionViewDelegate的子协议，其中常用方法如下，我们只需要实现我们需要的即可：

动态设置每个Item的尺寸大小

- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath;

动态设置每个分区的EdgeInsets

 - (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout insetForSectionAtIndex:(NSInteger)section;

动态设置每行的间距大小

- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section;

动态设置每列的间距大小

- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section;

动态设置某个分区头视图大小

- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section;

动态设置某个分区尾视图大小

- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section;

## iOS流布局UICollectionView系列四——自定义FlowLayout进行瀑布流布局

### 一、引言

前几篇博客从UICollectionView的基础应用到设置UICollectionViewFlowLayout更加灵活的进行布局，但都限制在系统为我们准备好的布局框架中，还是有一些局限性，例如，如果我要进行瀑布流似的不定高布局，前面的方法就很难满足我们的需求了，如下：

![](https://upload-images.jianshu.io/upload_images/9610202-457db0965e87c7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这种布局无疑在app的应用中更加广泛，商品的展示，书架书目的展示，都会倾向于采用这样的布局方式，当然，通过自定义FlowLayout，我们也很容易实现。

### 二、进行自定义瀑布流布局

首先，我们新建一个文件继承于UICollectionViewFlowLayout：

```
@interface MyLayout : UICollectionViewFlowLayout
```

为了演示的方便，这里我不做更多的封装，只添加一个属性，直接让外界将item个数传递进来，我们把重心方法重写布局的方法上：

```
@interface MyLayout : UICollectionViewFlowLayout
@property(nonatomic,assign)int itemCount;
@end
```

前面说过，UICollectionViewFlowLayout是一个专门用来管理collectionView布局的类，因此，collectionView在进行UI布局前，会通过这个类的对象获取相关的布局信息，FlowLayout类将这些布局信息全部存放在了一个数组中，数组中是UICollectionViewLayoutAttributes类，这个类是对item布局的具体设置，以后咱们在讨论这个类。总之，FlowLayout类将每个item的位置等布局信息放在一个数组中，在collectionView布局时，会调用FlowLayout类layoutAttributesForElementsInRect：方法来获取这个布局配置数组。因此，我们需要重写这个方法，返回我们自定义的配置数组，另外，FlowLayout类在进行布局之前，会调用prepareLayout方法，所以我们可以重写这个方法，在里面对我们的自定义配置数据进行一些设置。

简单来说，自定义一个FlowLayout布局类就是两个步骤：

1、设计好我们的布局配置数据 prepareLayout方法中

2、返回我们的配置数组 layoutAttributesForElementsInRect方法中

示例代码如下：

```
@implementation MyLayout
{
    //这个数组就是我们自定义的布局配置数组
    NSMutableArray * _attributeAttay;
}
//数组的相关设置在这个方法中
//布局前的准备会调用这个方法
-(void)prepareLayout{
    _attributeAttay = [[NSMutableArray alloc]init];
    [super prepareLayout];
    //演示方便 我们设置为静态的2列
    //计算每一个item的宽度
    float WIDTH = ([UIScreen mainScreen].bounds.size.width-self.sectionInset.left-self.sectionInset.right-self.minimumInteritemSpacing)/2;
    //定义数组保存每一列的高度
    //这个数组的主要作用是保存每一列的总高度，这样在布局时，我们可以始终将下一个Item放在最短的列下面
    CGFloat colHight[2]={self.sectionInset.top,self.sectionInset.bottom};
    //itemCount是外界传进来的item的个数 遍历来设置每一个item的布局
    for (int i=0; i<_itemCount; i++) {
        //设置每个item的位置等相关属性
        NSIndexPath *index = [NSIndexPath indexPathForItem:i inSection:0];
        //创建一个布局属性类，通过indexPath来创建
        UICollectionViewLayoutAttributes * attris = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:index];
        //随机一个高度 在40——190之间
        CGFloat hight = arc4random()%150+40;
        //哪一列高度小 则放到那一列下面
        //标记最短的列
        int width=0;
        if (colHight[0]<colHight[1]) {
            //将新的item高度加入到短的一列
            colHight[0] = colHight[0]+hight+self.minimumLineSpacing;
            width=0;
        }else{
            colHight[1] = colHight[1]+hight+self.minimumLineSpacing;
            width=1;
        }
        
        //设置item的位置
        attris.frame = CGRectMake(self.sectionInset.left+(self.minimumInteritemSpacing+WIDTH)*width, colHight[width]-hight-self.minimumLineSpacing, WIDTH, hight);
        [_attributeAttay addObject:attris];
    }
    
    //设置itemSize来确保滑动范围的正确 这里是通过将所有的item高度平均化，计算出来的(以最高的列位标准)
    if (colHight[0]>colHight[1]) {
        self.itemSize = CGSizeMake(WIDTH, (colHight[0]-self.sectionInset.top)*2/_itemCount-self.minimumLineSpacing);
    }else{
          self.itemSize = CGSizeMake(WIDTH, (colHight[1]-self.sectionInset.top)*2/_itemCount-self.minimumLineSpacing);
    }
    
}
//这个方法中返回我们的布局数组
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    return _attributeAttay;
}
 
 
@end
```

自定义完成FlowLayout后，我们在ViewController中进行使用：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
    layout.scrollDirection = UICollectionViewScrollDirectionVertical;
    layout.itemCount=100;
     UICollectionView * collect  = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
  
    [self.view addSubview:collect];
    
    
}
 
 
 
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 100;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

运行效果就是我们引言中的截图。

### 三、UICollectionViewLayoutAttributes类中我们可以配置的属性

通过上面的例子，我们可以了解，collectionView的item布局其实是LayoutAttributes类具体配置的，这个类可以配置的布局属性不止是frame这么简单，其中还有许多属性：

```
//配置item的布局位置
@property (nonatomic) CGRect frame;
//配置item的中心
@property (nonatomic) CGPoint center;
//配置item的尺寸
@property (nonatomic) CGSize size;
//配置item的3D效果
@property (nonatomic) CATransform3D transform3D;
//配置item的bounds
@property (nonatomic) CGRect bounds NS_AVAILABLE_IOS(7_0);
//配置item的旋转
@property (nonatomic) CGAffineTransform transform NS_AVAILABLE_IOS(7_0);
//配置item的alpha
@property (nonatomic) CGFloat alpha;
//配置item的z坐标
@property (nonatomic) NSInteger zIndex; // default is 0
//配置item的隐藏
@property (nonatomic, getter=isHidden) BOOL hidden; 
//item的indexpath
@property (nonatomic, strong) NSIndexPath *indexPath;
//获取item的类型
@property (nonatomic, readonly) UICollectionElementCategory representedElementCategory;
@property (nonatomic, readonly, nullable) NSString *representedElementKind; 
 
//一些创建方法
+ (instancetype)layoutAttributesForCellWithIndexPath:(NSIndexPath *)indexPath;
+ (instancetype)layoutAttributesForSupplementaryViewOfKind:(NSString *)elementKind withIndexPath:(NSIndexPath *)indexPath;
+ (instancetype)layoutAttributesForDecorationViewOfKind:(NSString *)decorationViewKind withIndexPath:(NSIndexPath *)indexPath;

```

通过上面的属性，可以布局出各式各样的炫酷效果，正如一句话：没有做不到，只有想不到。

## iOS流布局UICollectionView系列五——圆环布局的实现

### 一、引言

前边的几篇博客，我们了解了UICollectionView的基本用法以及一些扩展，在不定高的瀑布流布局中，我们发现，可以通过设置具体的布局属性类UICollectionViewLayoutAttributes来设置设置每个item的具体位置，我们可以再扩展一下，如果位置我们可以自由控制，那个布局我们也可以更加灵活，就比如创建一个如下的circleLayout：

![](https://upload-images.jianshu.io/upload_images/9610202-61f786273733d386.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种布局方式在apple的官方文档中也有介绍，是UICollectionView的一个应用示例。

### 二、设计一个圆环布局

先自定义一个layout类，这个类继承于UICollectionViewLayout，UICollectionLayout是一个布局抽象基类，我们要使用自定义的布局方式，必须将其子类化，可能你还记得，我们在进行瀑布流布局的时候使用过UICollectionViewFlowLayout类，这个类就是继承于UICollectionViewLayout类，系统为我们实现好的一个布局方案。

```
@interface MyLayout : UICollectionViewLayout
//这个int值存储有多少个item
@property(nonatomic,assign)int itemCount;
@end
```

我们需要重写这个类的三个方法，来进行圆环布局的设置，首先是prepareLayout，为布局做一些准备工作，使用collectionViewContentSize来设置内容的区域大小，最后使用layoutAttributesForElementsInRect方法来返回我们的布局信息字典，这个和前面瀑布流布局的思路是一样的：

```
@implementation MyLayout
{
    NSMutableArray * _attributeAttay;
}
-(void)prepareLayout{
    [super prepareLayout];
    //获取item的个数
    _itemCount = (int)[self.collectionView numberOfItemsInSection:0];
    _attributeAttay = [[NSMutableArray alloc]init];
    //先设定大圆的半径 取长和宽最短的
    CGFloat radius = MIN(self.collectionView.frame.size.width, self.collectionView.frame.size.height)/2;
    //计算圆心位置
    CGPoint center = CGPointMake(self.collectionView.frame.size.width/2, self.collectionView.frame.size.height/2);
    //设置每个item的大小为50*50 则半径为25
    for (int i=0; i<_itemCount; i++) {
        UICollectionViewLayoutAttributes * attris = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:[NSIndexPath indexPathForItem:i inSection:0]];
        //设置item大小
        attris.size = CGSizeMake(50, 50);
        //计算每个item的圆心位置
        /*
         .
         . .
         .   . r
         .     .
         .........
         */
        //计算每个item中心的坐标
        //算出的x y值还要减去item自身的半径大小
        float x = center.x+cosf(2*M_PI/_itemCount*i)*(radius-25);
        float y = center.y+sinf(2*M_PI/_itemCount*i)*(radius-25);
     
        attris.center = CGPointMake(x, y);
        [_attributeAttay addObject:attris];
    }
    
   
    
}
//设置内容区域的大小
-(CGSize)collectionViewContentSize{
    return self.collectionView.frame.size;
}
//返回设置数组
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    return _attributeAttay;
}
```

在viewController中代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
     UICollectionView * collect  = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
    [self.view addSubview:collect];
}
 
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.layer.masksToBounds = YES;
    cell.layer.cornerRadius = 25;
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    return cell;
}
```

如上非常简单的一些逻辑控制，我们就实现哦圆环布局，随着item的多少，布局会自动调整，如果不是UICollectionView的功劳，实现这样的功能，我们可能要写上一阵子了^_^。

![](https://upload-images.jianshu.io/upload_images/9610202-7f4a709117507668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## iOS流布局UICollectionView系列六——将布局从平面应用到空间

### 一、引言

前面，我们将布局由线性的瀑布流布局扩展到了圆环布局，这使我们使用UICollectionView的布局思路大大迈进了一步，这次，我们玩的更加炫一些，想办法将布局应用到空间。之前在管理布局的item的具体属性的类UICollectionViewLayoutAttributrs类中，有transform3D这个属性，通过这个属性的设置，我们真的可以在空间的坐标系中进行布局设计。iOS系统的控件中，也并非没有这样的先例，UIPickerView就是很好的一个实例，这篇博客，我们就通过使用UICollectionView实现一个类似系统的UIPickerView的布局视图，来体会UICollectionView在3D控件布局的魅力。系统的pickerView效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-83f6b3ad772cb424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 二、先来实现一个炫酷的滚轮空间布局

万丈的高楼也是由一砖一瓦堆砌而成，在我们完全模拟系统pickerView前，我们应该先将视图的布局摆放这一问题解决。我们依然来创建一个类，继承于UICollectionViewLayout：

```
@interface MyLayout : UICollectionViewLayout 

@end
```

对于.m文件的内容，前几篇博客中我们都是在prepareLayout中进行布局的静态设置，那是因为我们前几篇博客中的布局都是静态的，布局并不会随着我们的手势操作而发生太大的变化，因此我们全部在prepareLayout中一次配置完了。而我们这次要讨论的布局则不同，pickerView会随着我们手指的拖动而进行滚动，因此UICollectionView中的每一个item的布局是在不断变化的，所以这次，我们采用动态配置的方式，在layoutAttributesForItemAtIndexPath方法中进行每个item的布局属性设置。

至于layoutAttributesForItemAtIndexPath方法，它也是UICollectionViewLayout类中的方法，用于我们自定义时进行重写，至于为什么动态布局要在这里面配置item的布局属性，后面我们会了解到。

在编写我们的布局类之前，先做好准备工作，在viewController中，实现如下代码：

```

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
     UICollectionView * collect  = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
    [self.view addSubview:collect];
}
 
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 10;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, 250, 80)];
    label.text = [NSString stringWithFormat:@"我是第%ld行",(long)indexPath.row];
    [cell.contentView addSubview:label];
    return cell;
}
```

上面我创建了10个Item，并且在每个Item上添加了一个标签，标写是第几行。

在我们自定义的布局类中重写layoutAttributesForElementsInRect，在其中返回我们的布局数组：

```
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    NSMutableArray * attributes = [[NSMutableArray alloc]init];
    //遍历设置每个item的布局属性
    for (int i=0; i<[self.collectionView numberOfItemsInSection:0]; i++) {
        [attributes addObject:[self layoutAttributesForItemAtIndexPath:[NSIndexPath indexPathForItem:i inSection:0]]];
    }
    return attributes;
}
```

之后，在我们布局类中重写layoutAttributesForItemAtIndexPath方法：

```
-(UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    //创建一个item布局属性类
    UICollectionViewLayoutAttributes * atti = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    //获取item的个数
    int itemCounts = (int)[self.collectionView numberOfItemsInSection:0];
    //设置每个item的大小为260*100
    atti.size = CGSizeMake(260, 100);  
   /*
   后边介绍的代码添加在这里
   
   */ 
    return atti;
}
```

上面的代码中，我们什么都没有做，下面我们一步步来实现3D的滚轮效果。

首先，我们先将所有的item的位置都设置为collectionView的中心：

```
atti.center = CGPointMake(self.collectionView.frame.size.width/2, self.collectionView.frame.size.height/2);
```

这时，如果我们运行程序的话，所有item都将一层层贴在屏幕的中央，如下：

![](https://upload-images.jianshu.io/upload_images/9610202-eca5bc29e3fe67e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


很丑对吧，之后我们来设置每个item的3D效果,在上面的布局方法中添加如下代码:

```
    //创建一个transform3D类
    //CATransform3D是一个类似矩阵的结构体
    //CATransform3DIdentity创建空得矩阵
    CATransform3D trans3D = CATransform3DIdentity;
    //这个值设置的是透视度，影响视觉离投影平面的距离
    trans3D.m34 = -1/900.0;
    //下面这些属性 后面会具体介绍
    //这个是3D滚轮的半径
    CGFloat radius = 50/tanf(M_PI*2/itemCounts/2);
    //计算每个item应该旋转的角度
    CGFloat angle = (float)(indexPath.row)/itemCounts*M_PI*2;
    //这个方法返回一个新的CATransform3D对象，在原来的基础上进行旋转效果的追加
    //第一个参数为旋转的弧度，后三个分别对应x，y，z轴，我们需要以x轴进行旋转
    trans3D = CATransform3DRotate(trans3D, angle, 1.0, 0, 0);
    //进行设置
    atti.transform3D = trans3D;
```

对于上面的radius属性，运用了一些简单的几何和三角函数的知识。如果我们将系统的pickerView沿着y轴旋转90°，你会发现侧面的它是一个规则的正多边形，这里的radius就是这个多边形中心到其边的垂直距离，也是内切圆的半径，所有的item拼成了一个正多边形，示例如下：

![](https://upload-images.jianshu.io/upload_images/9610202-3f0fe8d1314d6621.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过简单的数学知识，h/2弦对应的角的弧度为2*pi/(边数)/2，在根据三角函数相关知识可知，这个角的正切值为h/2/radius，这就是我们radius的由来。 

对于angle属性，它是每一个item的x轴旋转度数，如果我们将所有item的中心都放在一点，通过旋转让它们散开如下图所示：

![](https://upload-images.jianshu.io/upload_images/9610202-54e6b69e7dc9109f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


每个item旋转的弧度就是其索引/(2*pi)。

通过上面的设置，我们再运行代码，效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-5a7312f5e4a0897e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


仔细观察我们可以发现，item以x中轴线进行了旋转平均布局，侧面的效果就是我们上面的简笔画那样，下面要进行我们的第三步了，将这个item，全部沿着其Z轴向前拉，就可以成为我们滚轮的效果，示例图如下：

![](https://upload-images.jianshu.io/upload_images/9610202-189bfaf545cbf3df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们继续在刚才的代码后面添加这行代码：

```
 //这个方法也返回一个transform3D对象，追加平移效果，后面三个参数，对应平移的x，y，z轴，我们沿z轴平移 
 trans3D = CATransform3DTranslate(trans3D, 0, 0, radius);
```

再次运行，效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-0d7085d0cc17ac2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


布局的效果我们已经完成了，离成功很近了对吧，只是现在的布局是静态的，我们不能滑动这个滚轮，我们还需要用动态滑动做一些处理。

### 三、让滚轮滑动起来

通过上面的努力，我们已经静态布局出了一个类似pickerView的滚轮，现在我们再来添加滑动滚动的效果

首先，我们需要给collectionView一个滑动的范围，我们以一屏collectionView的滑动距离来当做滚轮滚动一下的参照，我们在布局类中的如下方法中返回滑动区域：

```
-(CGSize)collectionViewContentSize{
    return CGSizeMake(self.collectionView.frame.size.width, self.collectionView.frame.size.height*[self.collectionView numberOfItemsInSection:0]);
}
```

这时我们的collectionView已经可以进行滑动，但是并不是我们想要的效果，滚轮并没有滚动，而是随着滑动出了屏幕，因此，我们需要在滑动的时候不停的动态布局，将滚轮始终固定在collectionView的中心，先需要在布局类中实现如下方法：

```
//返回yes，则一有变化就会刷新布局
-(BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds{
    return YES;
}
```

将上面的布局的中心点设置加上一个动态的偏移量：

```
 atti.center = CGPointMake(self.collectionView.frame.size.width/2, self.collectionView.frame.size.height/2+self.collectionView.contentOffset.y);
```

现在在运行，会发现滚轮会随着滑动始终固定在中间，但是还是不如人意，滚轮并没有转动起来，我们还需要动态的设置每个item的旋转角度，这样连续看起来，滚轮就转了起来，在上面设置布局的方法中，我们在添加一些处理：

```
//获取当前的偏移量
float offset = self.collectionView.contentOffset.y;
//在角度设置上，添加一个偏移角度
float angleOffset = offset/self.collectionView.frame.size.height;
CGFloat angle = (float)(indexPath.row+angleOffset)/itemCounts*M_PI*2;
```

再看看效果，没错，就是这么简单，滚轮已经转了起来。

### 四、让其循环滚动的逻辑

我们再进一步，如果滚动可以循环，这个控件将更加炫酷，添加这样的逻辑也很简单，通过监测scrollView的偏移量，我们可以对齐进行处理，因为collectionView继承于scrollView，我们可以直接在ViewController中实现其代理方法，如下：

```
-(void)scrollViewDidScroll:(UIScrollView *)scrollView{
    //小于半屏 则放到最后一屏多半屏
    if (scrollView.contentOffset.y<200) {
        scrollView.contentOffset = CGPointMake(0, scrollView.contentOffset.y+10*400);
    //大于最后一屏多一屏 放回第一屏
    }else if(scrollView.contentOffset.y>11*400){
        scrollView.contentOffset = CGPointMake(0, scrollView.contentOffset.y-10*400);
    }
}
```

因为咱们的环状布局，上面的逻辑刚好可以无缝对接，但是会有新的问题，一开始运行，滚轮就是出现在最后一个item的位置，而不是第一个，并且有些相关的地方，我们也需要一些适配：

在viewController中：

```
//一开始将collectionView的偏移量设置为1屏的偏移量
collect.contentOffset = CGPointMake(0, 400);
```

在layout类中：

```
//将滚动范围设置为(item总数+2)*每屏高度 
-(CGSize)collectionViewContentSize{
    return CGSizeMake(self.collectionView.frame.size.width, self.collectionView.frame.size.height*([self.collectionView numberOfItemsInSection:0]+2));
}
```

```
//将计算的具体item角度向前递推一个
CGFloat angle = (float)(indexPath.row+angleOffset-1)/itemCounts*M_PI*2;
```

OK，我们终于大功告成了，可以发现，实现这样一个布局效果炫酷的控件，代码其实并没有多少，相比，数学逻辑要比编写代码本身困难，这十分类似数学中的几何问题，如果你弄清了逻辑，解决是分分钟的事，我们可以通过这样的一个思路，设计更多3D或者平面特效的布局方案，抽奖的转动圆盘，书本的翻页，甚至立体的标签云，UICollectionView都可以实现，这篇博客中的代码在下面的连接中，疏漏之处，欢迎指正！

[http://pan.baidu.com/s/1jGCmbKM](http://pan.baidu.com/s/1jGCmbKM)

## iOS流布局UICollectionView系列七——三维中的球型布局

### 一、引言

通过6篇的博客，从平面上最简单的规则摆放的布局，到不规则的瀑布流布局，再到平面中的圆环布局，我们突破了线性布局的局限，在后面，我们将布局扩展到了空间，在Z轴上进行了平移，我们实现了一个类似UIPickerView的布局模型，其实我们还可以再进一步，类比于平面布局，picKerView只是线性排列布局在空间上的旋转与平移，这次，我们更加充分了利用一下空间的尺寸，来设计一个圆球的布局模型。

### 二、将布局扩展为空间球型

在viewController中先实现一些准备代码：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    MyLayout * layout = [[MyLayout alloc]init];
     UICollectionView * collect  = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, 320, 400) collectionViewLayout:layout];
    collect.delegate=self;
    collect.dataSource=self;
    //这里设置的偏移量是为了无缝进行循环的滚动，具体在上一篇博客中有解释
    collect.contentOffset = CGPointMake(320, 400);
    [collect registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:@"cellid"];
    [self.view addSubview:collect];
}
 
-(NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView{
    return 1;
}
//我们返回30的标签
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return 30;
}
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewCell * cell  = [collectionView dequeueReusableCellWithReuseIdentifier:@"cellid" forIndexPath:indexPath];
    cell.backgroundColor = [UIColor colorWithRed:arc4random()%255/255.0 green:arc4random()%255/255.0 blue:arc4random()%255/255.0 alpha:1];
    UILabel * label = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, 30, 30)];
    label.text = [NSString stringWithFormat:@"%ld",(long)indexPath.row];
    [cell.contentView addSubview:label];
    return cell;
}
 
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
//这里对滑动的contentOffset进行监控，实现循环滚动
-(void)scrollViewDidScroll:(UIScrollView *)scrollView{
    if (scrollView.contentOffset.y<200) {
        scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x, scrollView.contentOffset.y+10*400);
    }else if(scrollView.contentOffset.y>11*400){
        scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x, scrollView.contentOffset.y-10*400);
    }
    if (scrollView.contentOffset.x<160) {
        scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x+10*320,scrollView.contentOffset.y);
    }else if(scrollView.contentOffset.x>11*320){
        scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x-10*320,scrollView.contentOffset.y);
    }
}
```

这里面的代码比较上一篇博客中的并没有什么大的改动，只是做了横坐标的兼容。

在我们的layout类中，将代码修改成如下：

```
-(void)prepareLayout{
    [super prepareLayout];
    
}
//返回的滚动范围增加了对x轴的兼容
-(CGSize)collectionViewContentSize{
    return CGSizeMake( self.collectionView.frame.size.width*([self.collectionView numberOfItemsInSection:0]+2), self.collectionView.frame.size.height*([self.collectionView numberOfItemsInSection:0]+2));
}
-(BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds{
    return YES;
}
 
-(UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewLayoutAttributes * atti = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    //获取item的个数
    int itemCounts = (int)[self.collectionView numberOfItemsInSection:0];
    atti.center = CGPointMake(self.collectionView.frame.size.width/2+self.collectionView.contentOffset.x, self.collectionView.frame.size.height/2+self.collectionView.contentOffset.y);
    atti.size = CGSizeMake(30, 30);
    
    CATransform3D trans3D = CATransform3DIdentity;
    trans3D.m34 = -1/900.0;
    
    CGFloat radius = 15/tanf(M_PI*2/itemCounts/2);
    //根据偏移量 改变角度
    //添加了一个x的偏移量
    float offsety = self.collectionView.contentOffset.y;
    float offsetx = self.collectionView.contentOffset.x;
    //分别计算偏移的角度
    float angleOffsety = offsety/self.collectionView.frame.size.height;
    float angleOffsetx = offsetx/self.collectionView.frame.size.width;
    CGFloat angle1 = (float)(indexPath.row+angleOffsety-1)/itemCounts*M_PI*2;
    //x，y的默认方向相反
    CGFloat angle2 = (float)(indexPath.row-angleOffsetx-1)/itemCounts*M_PI*2;
    //这里我们进行四个方向的排列
   if (indexPath.row%4==1) {
        trans3D = CATransform3DRotate(trans3D, angle1, 1.0,0, 0);
    }else if(indexPath.row%4==2){
        trans3D = CATransform3DRotate(trans3D, angle2, 0, 1, 0);
    }else if(indexPath.row%4==3){
        trans3D = CATransform3DRotate(trans3D, angle1, 0.5,0.5, 0);
    }else{
        trans3D = CATransform3DRotate(trans3D, angle1, 0.5,-0.5,0);
    }
    
    trans3D = CATransform3DTranslate(trans3D, 0, 0, radius);
    
    atti.transform3D = trans3D;
    return atti;
}
 
 
-(NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    NSMutableArray * attributes = [[NSMutableArray alloc]init];
    //遍历设置每个item的布局属性
    for (int i=0; i<[self.collectionView numberOfItemsInSection:0]; i++) {
        [attributes addObject:[self layoutAttributesForItemAtIndexPath:[NSIndexPath indexPathForItem:i inSection:0]]];
    }
    return attributes;
}
```

布局效果如下：

![](https://upload-images.jianshu.io/upload_images/9610202-16ff61e331bac76f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


滑动屏幕，这个圆球是可以进行滚动的。

TIP：这里我们只平均分配了四个方向上的布局，如果item更加小也更加多，我们可以分配到更多的方向上，使球体更加充实。

