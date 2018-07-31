熟悉几个命令
```
查看描述文件信息:$security cms -D -i 描述文件路径

查看APP的签名信息
$codesign -vv -d APP路径

查看本机所有证书
$security find-identity -v -p codesigning

查看可执行文件的加密信息!
$otool -l WeChat | grep crypt

签名
$codesign -fs "证书" 需要签名的文件
```


#重签名(以微信WeChat为例子)

1. 通过zip方式打开微信ipa（ipa其实就是一个zip包）,进入Payload，找到WeChat app
![](https://upload-images.jianshu.io/upload_images/1464492-e08314705126917b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 创建XcodeSign 工程，运行项目，将XcodeSign app替换成Wechat app 然后重命名为XcodeSign
![打开app位置](https://upload-images.jianshu.io/upload_images/1464492-a2ccc170643f4845.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![拷贝修改](https://upload-images.jianshu.io/upload_images/1464492-1f5cad9e2911c45f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![最终结果](https://upload-images.jianshu.io/upload_images/1464492-f675fb60a97a760b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 将XcodeSign app显示包内容将里面info.plist 的Bundle identifier修改为XcodeSign 工程里面的Bundle identifier
![image.png](https://upload-images.jianshu.io/upload_images/1464492-5437fd406ac1c3d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


4. 如果有Frameworks文件的话要对对 Frameworks 中的所有framework进行签名：`$codesign -fs "证书" 需要签名的文件`
```
codesign -fs "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" WCDB.framework
成功返回：WCDB.framework: replacing existing signature
```
![](https://upload-images.jianshu.io/upload_images/1464492-f4dea164830e5646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![查找iPhone Developer LuRong Lei (4Y7DZEPUB8).png](https://upload-images.jianshu.io/upload_images/1464492-474895ffff18311e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 给可执行文件执行权限! `chmod +x WeChat` 执行命令后文件由白变黑色

![执行命令后文件由白变黑色](https://upload-images.jianshu.io/upload_images/1464492-0b213fdca6c12155.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. 干掉XcodeSign app中插件Plugins文件夹(免费账号不能签名插件)! 和Watch文件夹 直接干掉! 运行Xcode,微信就安装到手机上了.
![Plugins文件夹](https://upload-images.jianshu.io/upload_images/1464492-41be931a40f1194f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Watch文件夹](https://upload-images.jianshu.io/upload_images/1464492-d533e29da1d3fef9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#遇见的问题
进行第4步对framework进行签名的时候报错：
```
执行：codesign -fs "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" MMCommon.framework
报错：iPhone Developer: LuRong Lei (4Y7DZEPUB8): ambiguous (matches "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" and "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" in /Users/416862549qq.com/Library/Keychains/login.keychain-db)
```
原因是钥匙串中有多个iPhone Developer: LuRong Lei (4Y7DZEPUB8)证书，删除掉多余的，剩下一个就可以。
![image.png](https://upload-images.jianshu.io/upload_images/1464492-73af4011aa83f429.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
