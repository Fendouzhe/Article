#### 关于 UIBezierPath

`UIBezierPath`这个类在UIKit中， 是Core Graphics框架关于path的一个封装，使用此类可以定义简单的形状，比如我们常用到，矩形，圆形，椭圆，弧，或者不规则的多边形

#### UIBezierPath 基本使用方法

`UIBezierPath`对象是CGPathRef数据类型的封装。path如果是基于矢量形状的，都用直线或曲线去创建。我们一般使用`UIBezierPath`都是在重写view的`drawRect`方法这种情形。我们用直线去创建矩形或多边形，使用曲线创建弧或者圆。创建和使用path对象步骤：

1、  重写View的`drawRect`方法
2、 创建`UIBezierPath`的对象
3、 使用方法`moveToPoint:` 设置初始点
4、 根据具体要求使用`UIBezierPath`类方法绘图（比如要画线、矩形、圆、弧？等）
5、 设置`UIBezierPath`对象相关属性 （比如`lineWidth`、`lineJoinStyle`、`aPath.lineCapStyle`、`color`）
6、 使用stroke 或者 fill方法结束绘图

比如我们想要画一条线demo：

```
- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set]; //设置线条颜色

    UIBezierPath *path = [UIBezierPath bezierPath];
    [path moveToPoint:CGPointMake(10, 10)];
    [path addLineToPoint:CGPointMake(200, 80)];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound; //线条拐角
    path.lineJoinStyle = kCGLineJoinRound; //终点处理

    [path stroke];
}

```

#### 其他基本使用方法

在介绍其他使用方法之前，我们先来看一下  `path`的几个属性，以便下面我进行设置。

1. `[color set];`设置线条颜色，也就是相当于画笔颜色
2. `path.lineWidth = 5.0;`这个很好理解了，就是划线的宽度
3. `path.lineCapStyle`这个线段起点是终点的样式，这个样式有三种：
3.1 `kCGLineCapButt`该属性值指定不绘制端点， 线条结尾处直接结束。这是默认值。
3.2 `kCGLineCapRound` 该属性值指定绘制圆形端点， 线条结尾处绘制一个直径为线条宽度的半圆。
3.3 `kCGLineCapSquare` 该属性值指定绘制方形端点。 线条结尾处绘制半个边长为线条宽度的正方形。需要说明的是，这种形状的端点与“butt”形状的端点十分相似，只是采用这种形式的端点的线条略长一点而已*

4. `path.lineJoinStyle`这个属性是用来设置两条线连结点的样式，同样它也有三种样式供我们选择
4.1 `kCGLineJoinMiter` 斜接
4.2 `kCGLineJoinRound` 圆滑衔接
4.3 `kCGLineJoinBevel` 斜角连接
5. `[path stroke];`用 stroke 得到的是不被填充的 view ，`[path fill];` 用 fill 得到的内部被填充的 view，这点在下面的代码还有绘制得到的图片中有，可以体会一下这两者的不同。

##### 绘制多边形

![](http://upload-images.jianshu.io/upload_images/1464492-31488f850a07dc0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


绘制多边形，实际上就是又一些直线条连成，主要使用`moveToPoint:` 和`addLineToPoint:`方法去创建，`moveToPoint:`这个方法是设置起始点，意味着从这个点开始，我们就可以使用`addLineToPoint:`去设置我们想要创建的多边形经过的点，也就是两线相交的那个点，用``` addLineToPoint:``去创建一个形状的线段，我们可以连续创建line，每一个line的起点都是先前的终点，终点就是指定的点，将线段连接起来就是我们想要创建的多边形了。

```

#import "DrawPolygonView.h"

@implementation DrawPolygonView

- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set]; //设置线条颜色

    UIBezierPath* path = [UIBezierPath bezierPath];
    path.lineWidth = 5.0;

    path.lineCapStyle = kCGLineCapRound; //线条拐角
    path.lineJoinStyle = kCGLineJoinRound; //终点处理

    [path moveToPoint:CGPointMake(200.0, 50.0)];//起点

    // Draw the lines
    [path addLineToPoint:CGPointMake(300.0, 100.0)];
    [path addLineToPoint:CGPointMake(260, 200)];
    [path addLineToPoint:CGPointMake(100.0, 200)];
    [path addLineToPoint:CGPointMake(100, 70.0)];
    [path closePath];//第五条线通过调用closePath方法得到的

    //    [path stroke];//Draws line 根据坐标点连线
    [path fill];//颜色填充

}

```

在这里我们可以看到最后第五条线是用`[path closePath];`得到的，closePath方法不仅结束一个shape的subpath表述，它也在最后一个点和第一个点之间画一条线段，这个一个便利的方法我们不需要去画最后一条线了， 哈哈哈哈。这里我们用到的是`[path fill];//颜色填充`进行坐标连点，但是我们看见的是五边形内部被颜色填充了， 如果我们使用`[path stroke];`那我们得到的就是一个用线画的五边形。

##### 画矩形或者正方形

![](http://upload-images.jianshu.io/upload_images/1464492-be86b76ceea487ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


大家都知道正方形就是特殊的矩形咯，不多讲。只说矩形。

使用`+ (UIBezierPath *)bezierPathWithRect:(CGRect)rect`这个方法，设置好坐标 frame 就好了，就像我们创建 view 一样，好理解。

```
- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set]; //设置线条颜色

    UIBezierPath* path = [UIBezierPath bezierPathWithRect:CGRectMake(20, 20, 100, 80)];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound; //线条拐角
    path.lineJoinStyle = kCGLineJoinRound; //终点处理

    [path stroke];
}

```

##### 创建圆形或者椭圆形

![](http://upload-images.jianshu.io/upload_images/1464492-e615b7f7e32b2a92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用`+ (UIBezierPath *)bezierPathWithOvalInRect:(CGRect)rect`这个方法创建圆形或者椭圆形。

传入的rect矩形参数绘制一个内切曲线，如果我们传入的rect是矩形就得到矩形的内切椭圆，如果传入的是 正方形得到的就是正方形的内切圆。

```
- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set];

    UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(100, 100) radius:90 startAngle:0 endAngle:TO_RADIAUS(120) clockwise:YES];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound;
    path.lineJoinStyle = kCGLineJoinRound;
    [path stroke];
}

```

##### 创建一段弧线

使用`+ (UIBezierPath *)bezierPathWithArcCenter:(CGPoint)center radius:(CGFloat)radius startAngle:(CGFloat)startAngle endAngle:(CGFloat)endAngle clockwise:(BOOL)clockwis`这个方法创建一段弧线，介绍一下这个方法中的参数：
ArcCenter: 原点
radius: 半径
startAngle: 开始角度
endAngle: 结束角度
clockwise: 是否顺时针方向

弧线的参考系：

![](http://upload-images.jianshu.io/upload_images/1464492-d9beafdbc840ff34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 绘制二次贝塞尔曲线

![](http://upload-images.jianshu.io/upload_images/1464492-c3942388421d14b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用`- (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint`这个方法绘制二次贝塞尔曲线。曲线段在当前点开始，在指定的点结束，

一个控制点的切线定义。下图显示了两种曲线类型的相似，以及控制点和curve形状的关系:

![](http://upload-images.jianshu.io/upload_images/1464492-63c8af4703e43358.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set];

    UIBezierPath *path = [UIBezierPath bezierPath];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound;
    path.lineJoinStyle = kCGLineJoinRound;
    /*
     - (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint
     Parameters
     endPoint
     The end point of the curve.
     controlPoint
     The control point of the curve.
     */
    [path moveToPoint:CGPointMake(40, 150)];
    [path addQuadCurveToPoint:CGPointMake(140, 200) controlPoint:CGPointMake(20, 40)];
    [path stroke];
}

```

##### 绘制三次贝塞尔曲线

![](http://upload-images.jianshu.io/upload_images/1464492-0040c09718c76485.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用这个方法绘制三次贝塞尔曲线

```
- (void)addCurveToPoint:(CGPoint)endPoint controlPoint1:(CGPoint)controlPoint1 controlPoint2:(CGPoint)controlPoint2  
Parameters 

```

这个方法绘制三次贝塞尔曲线。曲线段在当前点开始，在指定的点结束，两个控制点的切线定义。下图显示了两种曲线类型的相似，以及控制点和curve形状的关系:

![](http://upload-images.jianshu.io/upload_images/1464492-7e2b774e04adc3e0.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```
- (void)drawRect:(CGRect)rect {
    /*
     - (void)addCurveToPoint:(CGPoint)endPoint controlPoint1:(CGPoint)controlPoint1 controlPoint2:(CGPoint)controlPoint2
     Parameters
     endPoint
     The end point of the curve.
     controlPoint1
     The first control point to use when computing the curve.
     controlPoint2
     The second control point to use when computing the curve.
     */

    UIColor *color = [UIColor redColor];
    [color set];

    UIBezierPath *path = [UIBezierPath bezierPath];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound;
    path.lineJoinStyle = kCGLineJoinRound;

    [path moveToPoint:CGPointMake(20, 200)];
    [path addCurveToPoint:CGPointMake(260, 200) controlPoint1:CGPointMake(140, 0) controlPoint2:CGPointMake(140, 400)];
    [path stroke];
}

```

##### 画带圆角的矩形

![](http://upload-images.jianshu.io/upload_images/1464492-50a51480670723cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



使用`+ (instancetype)bezierPathWithRect:(CGRect)rect;`这个方法绘制，这个方法和`bezierPathWithRect:`类似，绘制一个带内切圆的矩形。

```
- (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];
    [color set]; //设置线条颜色

    UIBezierPath* path = [UIBezierPath bezierPathWithRect:CGRectMake(20, 20, 100, 80)];

    path.lineWidth = 5.0;
    path.lineCapStyle = kCGLineCapRound; //线条拐角
    path.lineJoinStyle = kCGLineJoinRound; //终点处理

    [path stroke];
}

```

##### 指定矩形的某个角为圆角

![](http://upload-images.jianshu.io/upload_images/1464492-dc7a2609acd1fa6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


使用```+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect byRoundingCorners:(UIRectCorner)corners cornerRadii:(CGSize)cornerRadii;```
```
typedef NS_OPTIONS(NSUInteger, UIRectCorner) {
    UIRectCornerTopLeft     = 1 << 0,
    UIRectCornerTopRight    = 1 << 1,
    UIRectCornerBottomLeft  = 1 << 2,
    UIRectCornerBottomRight = 1 << 3,
    UIRectCornerAllCorners  = ~0UL
}; UIRectCorner 用来指定需要设置的角。cornerRadii 圆角的半径

```
```
-   (void)drawRect:(CGRect)rect {

    UIColor *color = [UIColor redColor];

    [color set];
    UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(0, 0, 100, 100) byRoundingCorners:UIRectCornerTopRight cornerRadii:CGSizeMake(20, 20)];
    path.lineCapStyle = kCGLineCapRound;
    path.lineJoinStyle = kCGLineJoinRound;
    path.lineWidth = 5.0;
    [path stroke];

 }

```

总结：`- (void)drawRect:(CGRect)rect `有会导致内存问题不建议使用，这里使用只是为了演示方便，实际开发中我们还要结合CAShapeLayer来生成自己想要的图形。后续有时间会更新，欢迎交流。
