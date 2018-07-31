#一、CAEmitterLayer 粒子发生器
1. CAEmitterLayer用于实现基于Core Animation的粒子发生器系统。
2. 在粒子系统中，CAEmitterLayer负责发射粒子（当然粒子也可以发射粒子），而这些所谓的粒子，就是CAEmitterCell，我们可以将CAEmitterLayer比作是CAEmitterCell的工厂，它会按照你的设置来以不同的样式不断产生粒子，也就是CAEmitterCell。
3. CAEmitterLayer决定了粒子从什么样的几何特性上发射出来，这个几何特性包括了位置，形状，大小。另外还有一些渲染相关的特性。另外的一些属性是CAEmitterLayer和CAEmiiterCell都有的，CAEmitterLayer的这些属性会作为CAEmitterCell相同属性的系数，例如当CAEmitterCell的lifetime（生命周期）为1，其所属CAEmitterLayer的lifetime为2时，在其它参数选择默认值的情况下，这个CAEmitterCell的生命周期就是1*2=2秒，2秒后，CAEmitterCell就会从粒子系统中被移除。
4. CAEmitterCell则决定了粒子自身的一些特征，例如速度，加速度，发射的范围，颜色等等。这些属性大多是以“中间值”搭配一个“范围”（A mean and a “cone”）的方式来表示的，例如velocity和velocityRange。表示CAEmitterCell的初始速度为velocity ± velocityRange。

#二、CAEmitterLayer 常用属性
```
1、粒子产生系数，默认1.0；每个cell的产生率乘以这个粒子产生系数，得出每一秒粒子的创建个数。 即：cell.birthRate 乘以 layer.birthRate =每秒粒子产生个数
@property float birthRate;
2、发射源的中心位置，默认(0,0,0)
@property CGPoint emitterPosition; //决定了粒子发射形状的中心点
@property CGFloat emitterZPosition;
3、emitterSize发射源的尺寸大小，emitterDepth决定粒子形状的深度联系(不太清楚，待补充)
@property CGSize emitterSize;//决定了粒子发射形状的大小
@property CGFloat emitterDepth;
4、发射源的形状，默认 kCAEmitterLayerPoint
@property(copy) NSString *emitterShape;//粒子从什么形状发射出来，它并不是表示粒子自己的形状。
为什么叫做“发射形状”呢？ 看看emitterShape的枚举，你就会大概能明白了
emitterShape的枚举：
kCAEmitterLayerPoint //点
kCAEmitterLayerLine //直线
kCAEmitterLayerRectangle //矩形
kCAEmitterLayerCuboid //3D立方形
kCAEmitterLayerCircle //圆形
kCAEmitterLayerSphere //3D球
5、发射模式, 默认kCAEmitterLayerVolume
@property(copy) NSString *emitterMode;
有以下取值：
kCAEmitterLayerPoints // 顶点
kCAEmitterLayerOutline // 轮廓，即边上
kCAEmitterLayerSurface // 表面，即图形的面积内
kCAEmitterLayerVolume // 容积，即3D图形的体积内
6、渲染模式, 默认kCAEmitterLayerUnordered
@property(copy) NSString *renderMode;
有以下取值：
kCAEmitterLayerUnordered
kCAEmitterLayerOldestFirst
kCAEmitterLayerOldestLast
kCAEmitterLayerBackToFront
kCAEmitterLayerAdditive
7、粒子的生命周期系数，默认1.0
即：（cell.lifetime 乘以 layer.lifetime）等于粒子的生命周期
@property float lifetime;
8、装粒子的数组
@property(nullable, copy) NSArray<CAEmitterCell *> *emitterCells;
9、不清楚待续，默认NO
@property BOOL preservesDepth;
10、粒子速度系数, 默认1.0
即：（cell.velocity 乘以 layer.velocity）等于粒子的速度
@property float velocity;
11、粒子的缩放比例系数, 默认1.0
即：（cell.scale 乘以 layer.scale）等于粒子的缩放比例
@property float scale;
12、自旋转速度系数, 默认1.0
即：（cell.spin 乘以 layer.spin）等于粒子的自旋转速度
13、随机数发生器
@property unsigned int seed;
```
#三、CAEmitterCell 常用属性
```
1、粒子的创建
＋ (instancetype)emitterCell;
2、根据键获得值
＋(nullable id)defaultValueForKey:(NSString *)key;
是否归档键值
－ (BOOL)shouldArchiveValueForKey:(NSString *)key;
3、粒子的名字，默认nil.
@property(nullable, copy) NSString *name;
4、不清楚待续...
@property(getter=isEnabled) BOOL enabled;
5、粒子的产生率，默认0
@property float birthRate;
6、粒子的生命周期和生命周期的范围，以秒为单位。两者默认0
@property float lifetime;
@property float lifetimeRange;
7、emissionLongitude指定了经度,经度角代表了x-y轴平面上与x轴之间的夹角
emissionLatitude指定了纬度,纬度角代表了x-z轴平面上与x轴之间的夹角，两者默认0:
@property CGFloat emissionLatitude;
@property CGFloat emissionLongitude;
8、周围发射角度,默认0
@property CGFloat emissionRange;
9、速度和速度范围，两者默认0
@property CGFloat velocity;
@property CGFloat velocityRange;
10、x,y,z方向上的加速度分量,三者默认都是0
@property CGFloat xAcceleration;
@property CGFloat yAcceleration;
@property CGFloat zAcceleration;
11、缩放比例,缩放比例范围缩放比例范围缩放比例范围,缩放比例和缩放范围默认是0
@property CGFloat scale;
@property CGFloat scaleRange;
@property CGFloat scaleSpeed;
12、自旋转角度，自旋转角度的范围,默认都是0
@property CGFloat spin;
@property CGFloat spinRange;
13、粒子的颜色,默认白色
@property(nullable) CGColorRef color;
14、粒子颜色blue,red,green,alpha能改变的范围,默认0
@property float redRange;
@property float greenRange;
@property float blueRange;
@property float alphaRange;
15、粒子颜色透明度，blue，green，red在生命周期内的改变速度,默认都是0
@property float redSpeed;
@property float greenSpeed;
@property float blueSpeed;
@property float alphaSpeed;
16、粒子的内容，大多都是个CGImageRef的对象,既粒子要展现的图片,默认nil
@property(nullable, strong) id contents;
17、应该画在contents里的子rectangle,默认(0,0,1,1)
@property CGRect contentsRect;
18、画在contents里的内容的比例因子
@property CGFloat contentsScale;
19、渲染'内容'图像时使用的滤波器参数。
@property(copy) NSString *minificationFilter;
@property(copy) NSString *magnificationFilter;
@property float minificationFilterBias;
20、粒子发射的粒子
@property(nullable, copy) NSArray<CAEmitterCell *> *emitterCells;
21、不清楚待续...
@property(nullable, copy) NSDictionary *style;
```
#四、实例

效果图：

![](http://upload-images.jianshu.io/upload_images/1464492-ad33589a6fa93c95.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码：
![](http://upload-images.jianshu.io/upload_images/1464492-7f0a872923be2519.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




