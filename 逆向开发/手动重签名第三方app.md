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
1. 通过zip方式打开微信ipa（ipa其实就是一个zip包）,进入Payload，找到WeChat app，右键显示包内容

2. 干掉插件Plugins文件夹(免费账号不能签名插件)! 和Watch文件夹 直接干掉!

3. 如果有Frameworks文件的话要对对 Frameworks 中的所有framework进行签名：`codesign -fs "iPhone Developer: XXX (XXX)" 需要签名的framework全名`
```
$codesign -fs "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" WCDB.framework
成功返回：WCDB.framework: replacing existing signature
```
![](https://upload-images.jianshu.io/upload_images/1464492-f4dea164830e5646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![查找iPhone Developer LuRong Lei (4Y7DZEPUB8).png](https://upload-images.jianshu.io/upload_images/1464492-474895ffff18311e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 给可执行文件执行权限! `chmod +x WeChat`

![执行命令后变黑色](https://upload-images.jianshu.io/upload_images/1464492-0b213fdca6c12155.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 将HandleSign app里面的描述文件embedded.mobileprovision拷贝到WeChat app里面
![](https://upload-images.jianshu.io/upload_images/1464492-2993e9de3e380819.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. 将WeChat app里面info.plist 的Bundle identifier修改为HandleSign demo里面的Bundle identifier

7. WeChat app目录中输入查看描述文件命令：`security cms -D -i embedded.mobileprovision`，找到`<key>Entitlements</key> `拷贝下面这段权限字典：
```
	<dict>
		<key>keychain-access-groups</key>
		<array>
			<string>9QLCGED959.*</string>
		</array>
		<key>get-task-allow</key>
		<true/>
		<key>application-identifier</key>
		<string>9QLCGED959.*</string>
		<key>com.apple.developer.team-identifier</key>
		<string>9QLCGED959</string>
	</dict>
```
到自己创建生成比如取名en.plist权限文件中，然后将en.plist放到WeChat app同级目录Payload文件夹中
![在xcode中创建方便](https://upload-images.jianshu.io/upload_images/1464492-f152e7b366595277.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
![位置](https://upload-images.jianshu.io/upload_images/1464492-f39417ef9d95f9d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


8. 在Payload 中签名整个APP：
```
	$codesign -fs "iPhone Developer: LuRong Lei (4Y7DZEPUB8)"  --no-strict --entitlements=en.plist WeChat.app///命令根据实际app名字进行修改
```

9. 将en.plist移除，在Payload同级目录（不是Payload目录中）终端输入：`zip -ry WeChat.ipa Payload `打包成ipa，成功就会在Payload下方生成WeChat.ipa
![成功](https://upload-images.jianshu.io/upload_images/1464492-dea66a744bb0d0aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


10. 通过Xcode安装ipa到真机 window -> Devices and Simulators -> 下方 + 号添加ipa包安装
![](https://upload-images.jianshu.io/upload_images/1464492-2528070be75a3bcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#遇见的问题
进行第4步对framework进行签名的时候报错：
```
执行：codesign -fs "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" MMCommon.framework
报错：iPhone Developer: LuRong Lei (4Y7DZEPUB8): ambiguous (matches "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" and "iPhone Developer: LuRong Lei (4Y7DZEPUB8)" in /Users/416862549qq.com/Library/Keychains/login.keychain-db)
```
原因是钥匙串中有多个iPhone Developer: LuRong Lei (4Y7DZEPUB8)证书，删除掉多余的，剩下一个就可以
![image.png](https://upload-images.jianshu.io/upload_images/1464492-73af4011aa83f429.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
