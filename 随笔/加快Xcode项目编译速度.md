## 1\. 增加XCode执行的线程数

可以根据自己`Mac`的性能，更改线程数设置`5`：`defaults write com.apple.Xcode PBXNumberOfParallelBuildSubtasks 5`

另外也有一个设置可以开启：`defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES`

XCode默认使用与CPU核数相同的线程来进行编译，但由于编译过程中的IO操作往往比CPU运算要多，因此适当的提升线程数可以在一定程度上加快编译速度。

![](http://upload-images.jianshu.io/upload_images/1464492-5cab79a7842f486d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "1479433675625943.png")

## 2.将Debug Information Format改为DWARF

在工程对应`Target`的`Build Settings`中，找到`Debug Information Format`这一项，将Debug时的`DWARF with dSYM file`改为`DWARF`。

如图：

![](http://upload-images.jianshu.io/upload_images/1464492-560522ca6227c3f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "1479433689587594.png")

**这一项设置的是是否将调试信息加入到可执行文件中，改为DWARF后，如果程序崩溃，将无法输出崩溃位置对应的函数堆栈，但由于Debug模式下可以在XCode中查看调试信息，所以改为DWARF影响并不大。这一项更改完之后，可以大幅提升编译速度。**

比如在目前本人负责的项目中，由于依赖了多个`Target`，所以需要在每个`Target`的`Debug Information Format`设置为`DWARF`。**顺便提一下，如果通过`Cocoapod`引入第三方则`Debug Information Format`默认就是设置为`DWARF`的。**

*   `SDWebImage`通过`Cocoapod``Debug Information Format`的默认设置

    ![](http://upload-images.jianshu.io/upload_images/1464492-7b9d334d45712e71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "1479433819885872.png")

> 注意：将`Debug Information Format`改为`DWARF`之后，会导致在`Debug`窗口无法查看相关类类型的成员变量的值。当需要查看这些值时，可以将`Debug Information Format`改回`DWARF with dSYM file`，`clean（必须）`之后重新编译即可。

## 3.将Build Active Architecture Only改为Yes

在工程对应`Target`的`Build Settings`中，找到`Build Active Architecture Only`这一项，将`Debug`时的`NO`改为`Yes`。

![](http://upload-images.jianshu.io/upload_images/1464492-ba19749e6ae7256e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "1479433840422261.png")

这一项设置的是是否仅编译当前架构的版本，如果为`NO`，会编译所有架构的版本。**需要注意的是，此选项在`Release`模式下必须为NO`，否则发布的ipa在部分设备上将不能运行。这一项更改完之后，可以显著提高编译速度。**

## 4.Run Script
如果添加了Run Script的在Build Phases勾选Run script only when installing

![](http://upload-images.jianshu.io/upload_images/1464492-b97b2e022b5ebd01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1464492-d815aa32ee93b808.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 4.设计编译优化等级

不要再项目中或者静态库中使用`-O4`，因为这会让`Clang`链接`Link Time Optimizations (LTO)`使得编译更慢，通常使用`-O3`。

![](http://upload-images.jianshu.io/upload_images/1464492-7a678be52f8f3e01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "1479433852851608.png")

**注意：在设置编译优化之后，XCode断点和调试信息会不正常，所以一般静态库或者其他`Target`这样设置。**

## 5.资源整合

### 5.1 将常用的代码及文件打包成静态库

### 5.2 添加预编译文件，把常用的头文件放到预编译文件里面

### 5.3 能用`@class`就用`@class`
