###1. 创建脚本文件XcodeApp.sh
![1](https://upload-images.jianshu.io/upload_images/1464492-bc940d692a9649ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###2. 给XcodeApp.sh添加脚本
```

# ${SRCROOT} 为工程文件所在的目录
TEMP_PATH="${SRCROOT}/Temp"
#资源文件夹,放三方APP的
ASSETS_PATH="${SRCROOT}/APP"
#ipa包路径
TARGET_IPA_PATH="${ASSETS_PATH}/*.ipa"



#新建Temp文件夹
rm -rf "$TEMP_PATH"
mkdir -p "$TEMP_PATH"

# --------------------------------------
# 1. 解压IPA 到Temp下
unzip -oqq "$TARGET_IPA_PATH" -d "$TEMP_PATH"
# 拿到解压的临时APP的路径
TEMP_APP_PATH=$(set -- "$TEMP_PATH/Payload/"*.app;echo "$1")
# 这里显示打印一下 TEMP_APP_PATH变量
echo "TEMP_APP_PATH: $TEMP_APP_PATH"

# -------------------------------------
# 2. 把解压出来的.app拷贝进去
#BUILT_PRODUCTS_DIR 工程生成的APP包路径
#TARGET_NAME target名称
TARGET_APP_PATH="$BUILT_PRODUCTS_DIR/$TARGET_NAME.app"
echo "TARGET_APP_PATH: $TARGET_APP_PATH"

rm -rf "$TARGET_APP_PATH"
mkdir -p "$TARGET_APP_PATH"
cp -rf "$TEMP_APP_PATH/" "$TARGET_APP_PATH/"

# -------------------------------------
# 3. 为了是重签过程简化，移走extension和watchAPP. 此外个人免费的证书没办法签extension

echo "Removing AppExtensions"
rm -rf "$TARGET_APP_PATH/PlugIns"
rm -rf "$TARGET_APP_PATH/Watch"

# -------------------------------------
# 4. 更新 Info.plist 里的BundleId
#  设置 "Set :KEY Value" "目标文件路径.plist"
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier $PRODUCT_BUNDLE_IDENTIFIER" "$TARGET_APP_PATH/Info.plist"

# 5.给可执行文件上权限
#添加ipa二进制的执行权限,否则xcode会告知无法运行
#这个操作是要找到第三方app包里的可执行文件名称，因为info.plist的 'Executable file' key对应的是可执行文件的名称
#我们grep 一下,然后取最后一行, 然后以cut 命令分割，取出想要的关键信息。存到APP_BINARY变量里
APP_BINARY=`plutil -convert xml1 -o - $TARGET_APP_PATH/Info.plist|grep -A1 Exec|tail -n1|cut -f2 -d\>|cut -f1 -d\<`


#这个为二进制文件加上可执行权限 +X
chmod +x "$TARGET_APP_PATH/$APP_BINARY"



# -------------------------------------
# 6. 重签第三方app Frameworks下已存在的动态库
TARGET_APP_FRAMEWORKS_PATH="$TARGET_APP_PATH/Frameworks"
if [ -d "$TARGET_APP_FRAMEWORKS_PATH" ];
then
#遍历出所有动态库的路径
for FRAMEWORK in "$TARGET_APP_FRAMEWORKS_PATH/"*
do
echo "🍺🍺🍺🍺🍺🍺FRAMEWORK : $FRAMEWORK"
#签名
/usr/bin/codesign --force --sign "$EXPANDED_CODE_SIGN_IDENTITY" "$FRAMEWORK"
done
fi

# ---------------------------------------------------
# 7. 注入我们编写的动态库
echo "开始注入"
# 需要注入的动态库的路径  这个路径我就写死了!
#INJECT_FRAMEWORK_RELATIVE_PATH="Frameworks/libHankHook.dylib"
#
## 通过工具实现注入
#yololib "$TARGET_APP_PATH/$APP_BINARY" "$INJECT_FRAMEWORK_RELATIVE_PATH"
echo "注入完成"

```
###3. 创建autoSign工程，在Build Phases添加Run Script,将脚本文件拖入
![拖入](https://upload-images.jianshu.io/upload_images/1464492-cfa29af0a6a108a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###4.运行项目，报错，原因是没有给脚本文件添加权限：
![image.png](https://upload-images.jianshu.io/upload_images/1464492-43c54480b904cd3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###5. 添加XcodeApp.sh的权限`chmod +x XcodeApp.sh`,再运行即可。
![image.png](https://upload-images.jianshu.io/upload_images/1464492-e317dd46df437450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###6. 新建个target选择Cocoa Touch Framework
![](https://upload-images.jianshu.io/upload_images/1464492-da37bd4c278243e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###7.给Target 添加framework的依赖关系
![1.创建Copy Files Phase](https://upload-images.jianshu.io/upload_images/1464492-39d740171f98793f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.选择Frameworks](https://upload-images.jianshu.io/upload_images/1464492-cc4107e222b0335c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.添加创建的framework](https://upload-images.jianshu.io/upload_images/1464492-8c9af3bf83cc16d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###8. 给framework创建LurongHook类
![](https://upload-images.jianshu.io/upload_images/1464492-12d8f92c2da1ac03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
运行项目Show in finder 打开AutoSign.app，显示包内容进入Frameworks文件夹就有了LurongHookFrameWork.framework
![1.png](https://upload-images.jianshu.io/upload_images/1464492-dcad498a3e8731ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2.png](https://upload-images.jianshu.io/upload_images/1464492-cb6ab2d720833da7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![3.png](https://upload-images.jianshu.io/upload_images/1464492-3d6d9de4edc8b3e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###9. 将yololib放到/usr/local/bin中方法调用
![image.png](https://upload-images.jianshu.io/upload_images/1464492-0cd14632b52bbc0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###10. 向Macho文件Wechat注入库的路径

#####1.先拷贝Macho文件Wechat出来
![1.png](https://upload-images.jianshu.io/upload_images/1464492-ec609c68f7b0f225.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2.png](https://upload-images.jianshu.io/upload_images/1464492-3f24359e9d040c7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.用yololib向Macho文件Wechat注入LurongHookFrameWork库的路径
```
yololib WeChat Frameworks/LurongHookFrameWork.framework/LurongHookFrameWork
```
![1.png](https://upload-images.jianshu.io/upload_images/1464492-e6ecc7e4c0a9d0ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.注入.png](https://upload-images.jianshu.io/upload_images/1464492-56bc068c70058932.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![3.注入成功.png](https://upload-images.jianshu.io/upload_images/1464492-9f9262ac21d9b43f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###11. 进入项目APP文件解压微信ipa包将包里面的Macho文件Wechat替换成上面的注入库路径的Wechat
!![1.png](https://upload-images.jianshu.io/upload_images/1464492-4875644f38cd1623.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.替换.png](https://upload-images.jianshu.io/upload_images/1464492-e683eac0f41a752f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###12. 重新将Wechat打包成ipa包`zip -ry WeChat.ipa Payload`然后删掉Payload文件重新运行，就可以看见LurongHook类里面的信息打印出来了。

![1.png](https://upload-images.jianshu.io/upload_images/1464492-b0cb497eefa0a606.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2.png](https://upload-images.jianshu.io/upload_images/1464492-79883db1fda8fa1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![3.png](https://upload-images.jianshu.io/upload_images/1464492-f165af97e2dea98b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




