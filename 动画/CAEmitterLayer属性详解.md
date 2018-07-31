在[上一篇](https://www.jianshu.com/p/287158935034)中我们大致了解了粒子系统，在这篇中我们再深入解析一下。在粒子系统中，CAEmitterLayer负责发射粒子（当然粒子也可以发射粒子），而这些所谓的粒子，就是CAEmitterCell，我们可以将CAEmitterLayer比作是CAEmitterCell的工厂，它会按照你的设置来以不同的样式不断产生粒子，也就是CAEmitterCell。

（1）CAEmitterLayer决定了粒子从什么样的几何特性上发射出来，这个几何特性包括了位置，形状，大小。另外还有一些渲染相关的特性。另外的一些属性是CAEmitterLayer和CAEmiiterCell都有的，CAEmitterLayer的这些属性会作为CAEmitterCell相同属性取值的系数，例如当CAEmitterCell的lifetime（生命周期）为1，其所属CAEmitterLayer的lifetime为2时，在其它参数选择默认值的情况下，这个CAEmitterCell的生命周期就是1*2=2秒，2秒后，CAEmitterCell就会从粒子系统中被移除。

（2）CAEmitterCell则决定了粒子自身的一些特征，例如速度，加速度，发射的范围，颜色等等。这些属性大多是以“中间值”搭配一个“范围”（A mean and a “cone”）的方式来表示的，例如velocity和velocityRange。表示CAEmitterCell的初始速度为velocity ± velocityRange。

下面我们再深入了解下CAEmitterLayer和CAEmitterCell中的几个常用的属性。

## 一、CAEmitterLayer常用属性

### 1、控制粒子发射位置和形状的属性

CAEmitterLayer并不是杂乱无章地发射粒子的，它发射粒子时的位置，发射的面积和面积对应的几何图形都是可以配置的，可以是一个点，或者一个方形、圆形等等

```
emitterPosition决定了粒子发射形状的中心点，
emitterSize则决定了粒子发射形状的大小，
emitterShape是粒子从什么形状发射出来，它并不是表示粒子自己的形状,
emitterMode 决定了粒子的发射模式。

```

###### （1）emitterShape

为什么叫做“发射形状”呢？ 看看emitterShape的枚举，你就会大概能明白了

```
NSString * const kCAEmitterLayerPoint;      // 点
NSString * const kCAEmitterLayerLine;               // 直线
NSString * const kCAEmitterLayerRectangle;      // 矩形
NSString * const kCAEmitterLayerCircle;          // 圆形
NSString * const kCAEmitterLayerCuboid;        // 3D rectangle
NSString * const kCAEmitterLayerSphere;   // 3D circle

```

我们用kCAEmitterLayerLine来说明一下。当你的CAEmitterLayer的emitterSize为CGSize（10， 10）时，你的所选择的emitterPosition为CGPoint（10,10）。那么形状为“Line”的CAEmitterLayer就会在如下图紫色的直线上产生粒子，对于“Line”来说，emitterSize的高度是被忽略的。

![](http://upload-images.jianshu.io/upload_images/1464492-f6d4c938794afdd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们可以这样理解，emitterPosition是所选emitterShape的中心点，例如对于矩形是对角线交点，对于圆形是圆心，对于直线是中点。而emitterSize则决定了矩形的大小，圆形的大小，直线的长度。这样说应该就够通俗易懂了。

另外，我们可以将emitterCell的速度相关的属性全部设置为0，你也可以直接注释掉他们，采用默认值，这样我们将会得到一些不会移动的粒子，因为它们没有速度，这样我们能看清楚CAEmitterLayer的形状是什么

kCAEmitterLayerRectangle：

![](http://upload-images.jianshu.io/upload_images/1464492-78bfac7d28aa2204.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


kCAEmitterLayerLine:

![](http://upload-images.jianshu.io/upload_images/1464492-3a2f258bf63f316d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


kCAEmitterLayerPoint:

![](http://upload-images.jianshu.io/upload_images/1464492-16668607286bd33e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


kCAEmitterLayerCircle:

![](http://upload-images.jianshu.io/upload_images/1464492-792809c0a566d82c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### （2）emitterMode

emitterMode的作用是进一步决定发射的区域是在发射形状的哪一部份，当我们看到它的枚举时，也就能大概了解了。

```
NSString * const kCAEmitterLayerPoints;  // 顶点
NSString * const kCAEmitterLayerOutline;     // 轮廓，即边上
NSString * const kCAEmitterLayerSurface;   // 表面，即图形的面积内
NSString * const kCAEmitterLayerVolume;    // 容积，即3D图形的体积内

```

当我们选择Points的时候，粒子会从发射形状的“顶点”发射出来，这里顶点只是一个简单的描述，有些图形不能用顶点来描述的,例如对于圆形来说，“顶点”就是圆心。Outline是指从形状的边界上发射，surface则是从形状的表面上发射，Voloume是相对于3D形状的“球体内”或“立方体内”发射，关于3D形状的问题，如果稍后有时间我再补吧，因为笔者现在还不知道怎么实现3D的粒子系统。

现在我们利用同样的方法看下emitterMode的样式是什么样的(我门保持发射形状为kCAEmitterLayerRectangle)：

kCAEmitterLayerPoints:

![](http://upload-images.jianshu.io/upload_images/1464492-623e8c5b48097c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


kCAEmitterLayerOutline:

![](http://upload-images.jianshu.io/upload_images/1464492-429a8c7851af16bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


kCAEmitterLayerSurface:

![](http://upload-images.jianshu.io/upload_images/1464492-2f487bd4da2b2d1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


CAEmitterLayer的上述属性，共同决定了你的粒子会在什么样的位置上被均匀地发射出来。 还有一些属性大多是跟CAEmitterCell同名的属性，它们会作为CAEmitterCell同名属性的“倍数”。

## 二、CAEmitterCell常用属性

CAEmitterLayer决定了粒子系统的基调，致于怎么让粒子系统酷炫起来，就需要CAEmitterCell的帮助。

###### (1)lifetime

粒子在系统上存在的时间，单位是秒。它可以配合lifetimeRage来让粒子生命周期均匀变化，以便可以让粒子的出现和消失显得更加离散。

###### (2)birthRate

粒子产生数量的决定参数,它表示CAEmitterLayer上每秒产生的粒子数量，birthRate是一个浮点数，大家可以灵活一点使用它，例如出于某些需要（测试需要），你想让你的粒子在屏幕上停留久一点以便可以观察它，一方面你可以让你的粒子速度相关的属性都为0，让它不会移动。另外设置lifetime让你的粒子存在时间变长，例如10秒，那么你只要将birthRate设置成0.1，就可以在10秒内观察同一个粒子，并且不会有其它的粒子来影响你的观察，10秒后，新的粒子出现，旧粒子被移除，你可以从头在观察它的变化。这样就方便很多。

**下面介绍一下CAEmitterCell关于颜色控制的属性，这部分属性让你获得了控制粒子颜色，颜色变化范围和速度的能力，你可以凭借它来完成一些渐变的效果或其它构建在它之上的酷炫效果。不过在认识CAEmitterCell的颜色属性之前，我们有必要先了解它的contents属性:**

###### (3)contents

contents其实我们在使用CALayer时就已经十分熟悉了，当我们想要一个CALayer展示一张静态图片时，就会使用到这个属性。在CAEmitterCell上它的意义也是一样的。不过由于下面4）提到的color属性的特点，我们一般都会将contents设置成一张纯色的图片，尤其是白色，看完4）的描述，你就明白了为什么这样会给我们带来方便了。

###### (4)color

color 会结合contents内容的颜色来改变我们的CAEmitterCell。它的结合算法其实是相当简单的，我们通过使用颜色来创建图像的方法来观察它跟contents的结合方式。下面的代码简单展示了如何用UIColor来创建UIImage。

```
-(UIImage*)imageWithColor:(UIColor*)color andSize:(CGSize)size
{
    UIGraphicsBeginImageContext(size);
    CGContextRef context=UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, color.CGColor);

    CGRect rect=CGRectMake(0, 0, size.width, size.height);
    UIBezierPath*bezierPath=[UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:size.width/2.0];

    CGContextAddPath(context, bezierPath.CGPath);
    CGContextFillPath(context);
    CGContextSetFillColorWithColor(context, color.CGColor);
    UIImage*theImage=UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return theImage;
}

```

假设我们的contents通过上面的接口传入 [UIColor colorWithRed:1 green:1 blue:1 alpha:1]来创建，并且我们cell的color传入[UIColor colorWithRed:0.5 green:1 blue:1 alpha:1]。那么在其它属性都为默认值的条件下，我们得到的CAEmitterCell的颜色是[UIColor colorWithRed:0.5 green:1 blue:1 alpha:1]，如果用数码测色计（在luanchPad的小工具集合里）来测色，会发现颜色值为rgb(128,255,255)。这是因为这两个值的结合算法就是两个颜色对应component的乘积。例子中的红色component的计算方式就是 0.5*1 = 0.5，换成rgb值的话，就是255*0.5*1 = 127.5，再向上取整得到128。如果你将contents的红色component从1改为0.75，那么就可以知道红色component是255*0.5*0.75 = 95.62，那么最终的颜色就会是rgb(96,255,255)。以此类推。不知道这样说得是否清楚。出于上面的原因，我们想完全通过color来控制CAEmitterCell的颜色，那么最好就选用一张颜色值为(255,255,255)，即白色的图片作为CAEmitterCell的contents。因为大家都知道在UIColor中，rgb值为255的component的值其实为1，这就是为什么我们是总是用这样的语句来表达值为rgb(x,y,z)的UIColor的原因：

```
[UIColor colorWithRed:x/255.f green:y/255.f blue:z/255.f alpha:1];

```

那么通过上面的算式我们知道，最后计算出来的结果一定跟color中的颜色是一致的，因为白色的UIColor每个component都为1，CAEmitterCell的color中任意component乘以contents中颜色对应的component都会得到color上原来的值，乘以1不改变值嘛。

###### 5、redRange，greenRange，blueRange

是粒子对应颜色的component的初始取值范围。例如redRange为0.1，那么当你的粒子的color对应的rgb为（10，255，255）。那么在其它值取默认值的前提下，这个粒子在CAEmitterLayer上发射出来的时候，它的rgb中的red component会均匀分布在 10正负0.1*255之间，即［0，35］，颜色值不能为负。

###### 6、redSpeed，greenSpeed，blueSpeed

表示对应的颜色component的变化速度，它的取值范围也是0～1。表示每秒钟的颜色变化率，这样说可能不太容易理解。我们拿一些数字来举例就很容易明白了。例如你的粒子的颜色是rgb（0，255，255），并且你的粒子的redRange也为0 ，这表示你的粒子被发射出来的时候，它的颜色值中的红色值总是为0 。你的粒子的生命周期lifetime为10，即粒子可以存在10秒，10秒后会从layer上移除。那么当你的redSpeed取值为0.1时，你将会看到你的粒子的红色值每秒增加0.1*255，这个过程是连续不断变化，外观上看就是你的粒子越来越接近白色。当粒子到达生命结束的时候也就是10秒的时候，你的粒子的颜色也恰好变成了rgb(255,255,255)。

*上述5和6这些属性共同决定了粒子的颜色变化情况。它们都是一个值在[0,1] 的浮点数。另外还有一对类似的属性，alphaRange和alphaSpeed，他们决定了颜色中的alpha通道的值，你可以通过它来让你的粒子渐隐或渐现。*

**下面我们一起来了解一下决定粒子发射角度和发散范围的若干属性。从CAEmitterLayer的介绍中，我们知道了粒子会从CAEmitterLayer设置好的位置大小和形状上发射出来，然而发射出来后，粒子往什么方向继续飞行，则是由下面的几个属性共同决定的:**

###### 7、emissionLongitude

emissionLongtitude决定了粒子飞行方向跟水平坐标轴（x轴）之间的夹角，默认是0，即沿着x轴向右飞行。顺时针方向是正向，例如emissionLongtitude为0 ，则粒子顺着x轴飞行，如果你想让你的粒子发射出来后，沿着y轴向下飞行，那么emissionLongtitude就应该设置为PI/2。即90度，正如我所说的那样，顺时针方向是正方向。如果你想让你的粒子沿着y轴向上飞行，你可以将emissionLongtitude设置为 3*PI/2也可设置为 -PI/2。下图就简单展示了emissionLongtitude的几个典型取值，图中绿色箭头所指的方向就是当emissionLongtitude为箭头对应角度时的粒子飞行方向:

![](http://upload-images.jianshu.io/upload_images/1464492-0065ac87dcd75a4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 8、emissionRange

emissionRange则决定了粒子的发散范围，同样是一个弧度值（radians），表示粒子在沿着emissionLongtitude方向所形成的顶角为2倍emissionRange的圆锥范围内发散。我们看例图：我们把emisstionLongtitude设置为-PI/2，让粒子向上飞行，并且让emissionRange为PI／4。那么按照上面的说法，我们应该能得到一个向上的，并且顶角为2 * PI／4的圆锥，结果应该如下图：

![](http://upload-images.jianshu.io/upload_images/1464492-0f5aeb96064c3937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我门可以修改代码检验一下：

```
    emitterLayer.emitterShape=kCAEmitterLayerPoint;
    emitterLayer.emitterPosition=self.view.center;
    cell0.birthRate=10000;
    cell0.velocity=10;
    cell0.lifetime=20;
    cell0.emissionLongitude=-M_PI_2;
    cell0.emissionRange=M_PI_4;

```

结果如图：

![](http://upload-images.jianshu.io/upload_images/1464492-fe7749a7662971f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 9、 xAcceleration yAcceleration zAcceleration

这3个属性分别定义了3个坐标轴上的加速度，它们代表了不同坐标轴方向上的每秒的速度增量，当xAcceleration为正数时，粒子每秒向x轴正方向加速，为负数时则向负方向即水平向左加速。当yAcceleration为正数时，粒子向y轴的负方向加速，也是就是向下加速，否则向上加速。

###### 10、spin，spinRange

粒子的自转是以弧度制来计算的，表示每秒钟粒子自转的弧度数。例如你的粒子的生命周期就是10秒，那么你想让你的粒子在10秒内刚好自转1周，我们假定spinRange为0，那么要达到你的要求，你的粒子的spin值就应该为((PI/180)*360)/10，前面是将360度转成弧度，后面是将360度对应的弧度平均分到10秒去，这样就得到了每秒需要转动的弧度数。另外，当spin为正数的时候，粒子是顺时针旋转的，为负数的话就是逆时针选转了。 spinRange就不多说了，跟其它range的意义和作用是一样的。

**每一个粒子都可以作为一个发射器来发射粒子，我们可以通过这个特点来实现一些递进的粒子效果，例如瀑布下面的水花，太阳的周围的散射的光线等**

###### 11、emitterCells

CAEmitterCell的emitterCells跟CAEmitterLayer的一样，也是一个CAEmitterCell的队列。我们基本可以按照操作CAEmitterLayer的emitterCells一样来设置我们粒子的emitterCells。只是有几点内容是需要我们留意一下的。为了方便描述，我们将从粒子上发射出来的粒子称作称作subCell。

（1）、CAEmitterCell也是服从CAMediatiming协议的，我们通过控制subCell的beginTime来控制subCell的出现时机。当你的subCell的beginTime为0时，表示你的粒子从CAEmitterLayer上发射出来后就会立即开始发射subCell，你需要适当选取你的subCell的birthRate，因为你要知道，你的每个粒子都会每秒发射birthRate个subCell。subCell的beginTime是不能大于你的粒子的lifetime的，它其实代表了你的粒子从什么时候开始发射subCell嘛，所以大于你的粒子的生命周期，它就无法发射你的subCell了，因为你的粒子已经“死了”。

(2)另外，值得一提的是，如果你想控制subCell的发射方向，那么需要考虑到父粒子的emissionLongtitude的情况。无论粒子是从什么样的形状上发射出来的，当它要发射subCell的时候，subCell总是从kCAEmitterLayerPoint形状上由父粒子的中心发射出来的。根据上面关于emissionLongtitude的描述， 当它为0时，粒子水平向右飞行。然而subCell的为0时，subCell是沿着父粒子的发射方向飞行的，也就是说，父粒子的发射方向是subCell的emissionLongtitude为0时的飞行方向。

```
 CAEmitterCell*cell0=[CAEmitterCell emitterCell];
    cell0.lifetime=5.0;
    cell0.birthRate=100.0;
    cell0.velocity=50;
    cell0.emissionLongitude=M_PI_4*1.5;
    cell0.emissionRange=M_PI_4/2;
    cell0.xAcceleration=-5;
    cell0.contents=(id)[UIImage imageNamed:@"xiaolvkuai.png"].CGImage;

    CAEmitterCell*subCell=[CAEmitterCell emitterCell];
    subCell.lifetime=8;
    subCell.birthRate=10;
    subCell.contents=(id)[UIImage imageNamed:@"xiaolvkuai.png"].CGImage;
    subCell.velocity=60;
    subCell.emissionLongitude=0;
    subCell.emissionRange=M_PI_2;
    subCell.beginTime=4.5;
    subCell.scale=0.5;
    cell0.emitterCells=@[subCell];
    emitterLayer.emitterCells=@[cell0];

```

![](http://upload-images.jianshu.io/upload_images/1464492-985f02122b65ebbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在下图中红色的箭头表示了上述例子中，subCell的emissionLongtitude的取值，大家可以看到垂直向下的红色箭头的取值为0 ，因为垂直向下就是subCell的父粒子的发射方向

![](http://upload-images.jianshu.io/upload_images/1464492-21b0a78c01aa3ace.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



