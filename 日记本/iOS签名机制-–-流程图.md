###签名的作用：
保证安装在用户手机上的APP都是经过Apple官方也许的
####证书的利用：
![证书的利用.png](https://upload-images.jianshu.io/upload_images/1464492-103182906018ecd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####证书的注册和下载：
![证书的注册和下载.png](https://upload-images.jianshu.io/upload_images/1464492-702575d3cced829d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####签名流程图：
![签名流程图.png](https://upload-images.jianshu.io/upload_images/1464492-6369ca15e7cb9b36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###文件总结：
* .certSigningRequest􏰀􏰁 
    Mac􏰄􏰅公钥
* .cer􏰀􏰁 
    利用􏰆􏰇Apple􏰈􏰅􏰉私钥（CA），对Mac公钥生成了数字签名􏰄􏰅􏰍􏰎􏰏􏰐􏰑􏰒􏰓
* .mobileprovision
    利用􏰆􏰇Apple􏰈􏰅􏰉私钥，对【􏰆􏰇􏰈􏰅􏰋􏰌􏰔.cer􏰕􏰖证书+devices+App ID+entitlements权限】􏰗􏰘􏰙􏰐􏰑􏰒􏰓进行数字签名
