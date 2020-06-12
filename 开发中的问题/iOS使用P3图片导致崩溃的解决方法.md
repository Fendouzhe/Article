最近app刚上架,突然收到大面积投诉....一看bugly,9.0-9.3的机器无一幸免,由于项目里有些图标是我直接从阿里图库下载的,问了UI P3,16进制的图片是什么他也说不清,索性让他重新做图了,这个问题只要图片是UI做图基本就可避免

1.打包成ipa
2.把ipa的后缀改成zip,解压缩(这时候会看到一个Payload文件夹)
3.打开终端  输入  cd 
4.把 Payload  拖动到终端里(这里的拖动只是为了获取这个文件在电脑上的地址),  回车
5.在终端输入  find . -name 'Assets.car'  回车
6.在终端输入  sudo xcrun --sdk iphoneos assetutil --info ./Assets.car > /tmp/Assets.json  回车
7.在终端输入  open /tmp/Assets.json  回车
8.这时候会打开一个text 搜索 DisplayGamut  看看后面是不是P3 如果搜索到的是p3  图片格式还是不对,如果是空或者搜索到显示的不是P3,那图片就对了,根据Name去查找项目里的这张图片吧,然后将其替换.


如果搜索P3还搜到了一个AssetType:"PackedImage",name为ZZZZPackedAsset-2.0.1-gamut1的东西,这时候先不要管他,等到把P3图片全部替换了这个东西就没了(我傻兮兮的找了很久...)
下面是ZZZZPackedAsset这个东西的示意图
![image](http://upload-images.jianshu.io/upload_images/9610202-572d1936afeebab9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

--------------------- 
作者：带我逃跑吧 
原文：https://blog.csdn.net/Simona_1973/article/details/80438372 
