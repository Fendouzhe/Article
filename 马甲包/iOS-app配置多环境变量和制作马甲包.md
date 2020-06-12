> 需求一：很多公司的app都分成debug环境和release环境(多个接口域名)，平时开发和测试都在debug环境，打包上线的时候才切换到release环境；需求二：公司要求制作马甲包，即在原来app的基础上，只修改app的名称，图标，LaunchImage，替换app中带有app名称的文本，并用另一个开发者账号发布，马甲包的数量不定。手动在代码里更改环境变量，替换图片肯定是不可取的，这里我们采用Configuration来实现这两个需求。定义好不同的Configuration后，就可以分别设置 Build、Archive、Test等操作分别使用哪一个 Configuration 进行编译，从而可以轻松地分离开各个环境变量的设置。

## 1.新建Configuration

点击Project->Info，默认有Debug和Release两个Configuration，顾名思义Debug用于调试，Release用于发布，区别是Debug默认添加了预编译宏DEBUG=1，Release不能调试程序，并且Release编译时做了优化。点击Configurations选项卡下面的加号，分别复制一个Debug和Release的Configuration，这里我取名为Debug_a和Release_a，代表马甲包a的两个Configuration

![image](//upload-images.jianshu.io/upload_images/1070332-28886043c2ea691a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

![image](//upload-images.jianshu.io/upload_images/1070332-777f2aa3d65ea910.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/703)

**注意**：新建完Configuration之后请重新执行pod install命令

## 2.新建Scheme

为上一步新建的Configuration再新建Scheme，这里我新建了两个Scheme，命名为马甲包a_release和马甲包a_debug，在Manage Schemes里面把右边的Shared选项勾选，否则在git上无法同步。

![image](//upload-images.jianshu.io/upload_images/1070332-fc817b61402791b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/508)

![image](//upload-images.jianshu.io/upload_images/1070332-6ecd8b485feaa368.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/831)

在Edit Scheme里面把Run和Archive模式改成对应新建的Build Configuration

![image](//upload-images.jianshu.io/upload_images/1070332-a80425aeef3615ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/730)

![image](//upload-images.jianshu.io/upload_images/1070332-e3a66224c8f5ff44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/729)

## 3.配置AppIcon和LaunchImage

新建一个App Icon和Launch Image文件夹，重新命名，拖入图片。

![image](//upload-images.jianshu.io/upload_images/1070332-23d61c64fe1ba6f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/495)

选择Targets->Build Settings，搜索asset，在Asset Catalog App Icon Set Name 和 Asset Catalog Launch Image Set Name 配置各个Configuration所对应的图片文件夹名称。

![image](//upload-images.jianshu.io/upload_images/1070332-0acb2b8686cfc827.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/899)

## 4.配置App名称，Bundle ID 和 证书

### 4.1 配置App名称

在Project->Build Settings 点击加号选择Add User-Defined Setting]，即增加用户自定义设置，添加一个App名称的设置，为不同的Configuration设置不同的App名字。

![image](//upload-images.jianshu.io/upload_images/1070332-d87c612021db4b2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/747)

![image](//upload-images.jianshu.io/upload_images/1070332-54bacd3a61fbc220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/520)

然后在info.plist中设置Bundle display name为我们自定义的设置，${CusomAppName}。

![image](//upload-images.jianshu.io/upload_images/1070332-ef6c9d606cfcf074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/819)

### 4.2 配置Bundle ID和证书

不同的马甲包可能是由不同的开发者账号发布的，所以需要配置对应的bundle id 和 证书。

bundle id 在Targets->Build Settings 中的Product Bundle Identifier设置。

![image](//upload-images.jianshu.io/upload_images/1070332-113ba92ea46a5b1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/889)

证书配置如下图

![image](//upload-images.jianshu.io/upload_images/1070332-6effb0a3fd10c598.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/821)

## 5.其他配置

在Targets->Build Settings->Preprocessor Macros中，可以根据Configuration配置不同的预编译宏，根据这个预编译宏的不同，在代码里面也可以有不同的配置，比如渠道号，接口域名等。

![image](//upload-images.jianshu.io/upload_images/1070332-580ffcd17ea9981f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/889)

```
#ifdef MaJiaA
#define kchannelCode @"majia_1"
#endif

#ifdef DEBUG
NSString *kServiceDomain = @"http://api.test";
#else
NSString *kServiceDomain = @"http://api.release";
#endif

```

![image](//upload-images.jianshu.io/upload_images/1070332-30ea1e00d0221d28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/365)

## 6.总结

用这种方法能比较方便地实现多环境变量的配置，[具体demo可以在github上下载](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fzhouhuanqiang%2FZhqMultiEnvironmentDemo)。

原文作者：周焕强
原文链接：https://www.jianshu.com/p/04b63de8ae23
