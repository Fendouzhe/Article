
参考文章：[《简单说一下Aria2GUI的配置方法》](https://www.jianshu.com/p/b58fff3fb946)

使用 Aria2GUI 的原因，用户可以在 百度网盘 网页端直接通过该工具下载，并且速度有很大提升。

简单来说有以下步骤：

1.下载 Aria2GUI 客户端。地址：[aria2gui](https://github.com/yangshun1029/aria2gui/releases)

2.下载 网盘助手 插件（文件要保存好路径，如果误删会导致插件无法使用）。地址：[BaiduExporter](https://github.com/acgotaku/BaiduExporter)

3.在浏览器上安装插件。有两种浏览器安装方式：

a.Chrome  窗口 --> 扩展程序 --> 加载已解压的扩展程序，进入 BaiduExporter-master，选择chrome文件夹即可。

b.Firefox  工具 --> 扩展 --> 点击设置图标 --> 调试附加组件 --> 临时载入附加组件，进入 BaiduExporter-master，进入chrome，选择 manifest.json ，点击OK即可。

打开 百度网盘 使用时，选择文件，点击 导出下载 ，选择 ARIA2 RPC 即可。

更新：Firefox Version 57.0.1 ，工具 --> 附加组件 --> 扩展 ......，每次使用都要重新导入一下插件，不然不会出现“导出下载”字样。

出现错误：

参考文章：[《Internal server error and Error: Unauthorized bug 反馈》](https://github.com/yangshun1029/aria2gui/issues/41)

1.打开 Aria2GUI 客户端，提示  Error:Internal server error 错误。 解决办法：将 客户端 添加到 应用程序中。

2.如果还是不行，参考上述文章。

#我的解决办法：将 客户端 添加到 应用程序中然后分别进行设置：
###Aria2GUI设置：
![http://127.0.0.1:6800/jsonrpc.png](http://upload-images.jianshu.io/upload_images/1464492-07cb54065d868ca5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###百度云设置:
![1.png](http://upload-images.jianshu.io/upload_images/1464492-ebaa856c3c61a9d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![2.png](http://upload-images.jianshu.io/upload_images/1464492-8b0454f67e4d2440.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###百度云下载:
![](http://upload-images.jianshu.io/upload_images/1464492-bba818d6f48f036f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
