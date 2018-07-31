###1. 创建Dylib注入项目，在.xcodeproj同级目录创建APP文件,放入微信ipa包，Build Phase添加Run Script，将脚本路径拖入Run Script.
![image.png](https://upload-images.jianshu.io/upload_images/1464492-91cffbe352fff0a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###2. 创建一个dylib库
![1.png](https://upload-images.jianshu.io/upload_images/1464492-592f75c75541170c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2.png](https://upload-images.jianshu.io/upload_images/1464492-9e8333838d313fe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###3. 选择scheme 为动态库，运行编译
![1.png](https://upload-images.jianshu.io/upload_images/1464492-53efc1e97ef5609e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2 Show in Finder 显示位置.png](https://upload-images.jianshu.io/upload_images/1464492-a13cf5e399b3ca17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###4. Build Phases 添加Copy Files
![1.png](https://upload-images.jianshu.io/upload_images/1464492-b2e660315d116451.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](https://upload-images.jianshu.io/upload_images/1464492-9e023b729d643044.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.png](https://upload-images.jianshu.io/upload_images/1464492-fa5c9ffca2424ef4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4.png](https://upload-images.jianshu.io/upload_images/1464492-86fc7b16f63ffabd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###5. 选择scheme 重新改回去,运行报错
![1.png](https://upload-images.jianshu.io/upload_images/1464492-7a352e9d731b1c94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](https://upload-images.jianshu.io/upload_images/1464492-855cd8e19e322063.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####解决方法1：将库拷贝进工程app同级目录
![1.png](https://upload-images.jianshu.io/upload_images/1464492-5c3ffb50e5c9d8f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](https://upload-images.jianshu.io/upload_images/1464492-f97e3a2f8a6cfed5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.png](https://upload-images.jianshu.io/upload_images/1464492-18f48037a75607fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####解决方法2：修改查找的设置路径（Debug和Release都改成工程app路径目录Debug-iphoneos ）

![1.修改工程加载路径.png](https://upload-images.jianshu.io/upload_images/1464492-5a2c195fd35e2f86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.修改动态库加载路径.png](https://upload-images.jianshu.io/upload_images/1464492-1d0e9ff5dedb7ba9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.重选编译.png](https://upload-images.jianshu.io/upload_images/1464492-a895fa44bd95328d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4.有了.png](https://upload-images.jianshu.io/upload_images/1464492-7be3ca37aa6b5436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![5.重选运行.png](https://upload-images.jianshu.io/upload_images/1464492-ce04f3ec2e06c147.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###6. 此时库还没有注入Macho文件，需要写脚本注入在XcodeApp.sh加入注入脚本
```
# ---------------------------------------------------
# 7. 注入我们编写的动态库
echo "开始注入"
# 需要注入的动态库的路径  这个路径我就写死了!
INJECT_FRAMEWORK_RELATIVE_PATH="Frameworks/libLurongHook.dylib"
#
## 通过工具实现注入
yololib "$TARGET_APP_PATH/$APP_BINARY" "$INJECT_FRAMEWORK_RELATIVE_PATH"
echo "注入完成"
```
![1.png](https://upload-images.jianshu.io/upload_images/1464492-30be238081ca1074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Command+B 然后 Command+9看打印信息：
![2.png](https://upload-images.jianshu.io/upload_images/1464492-bf52e65a600b6a68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用MachoView打开app里面的Macho文件Wechat,找到load Commands下方就可以看见库导进去了：
![image.png](https://upload-images.jianshu.io/upload_images/1464492-6098741274f1d613.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1464492-1f4af7b97030c65a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Command+B 运行报架构错误：
![image.png](https://upload-images.jianshu.io/upload_images/1464492-273150418964cd3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查看库的架构 `file+库名字及后缀`：
![image.png](https://upload-images.jianshu.io/upload_images/1464492-5c61792c19c2f7d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###7. 前面的都是错误演示，我们现在重新从创建dylib库开始
1.首先将库删除：
![1.png](https://upload-images.jianshu.io/upload_images/1464492-479a09cd6e21d687.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](https://upload-images.jianshu.io/upload_images/1464492-da795945d8836ece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.png](https://upload-images.jianshu.io/upload_images/1464492-24d670f7118319d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.重新创建同名的库进行前面第1步设置脚本和第4步的操作，然后设置Base SDK：

![1.png](https://upload-images.jianshu.io/upload_images/1464492-a7f54f64b343214d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.变化.png](https://upload-images.jianshu.io/upload_images/1464492-9629a069d7d096dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.设置Code Sign Identity为iPhone Developer：
![image.png](https://upload-images.jianshu.io/upload_images/1464492-a99f963019c5b2d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.编译库，看app同级目录，已经自动导入了创建的库
![image.png](https://upload-images.jianshu.io/upload_images/1464492-e57621967c6eab60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.运行项目  
