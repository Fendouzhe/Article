在使用git推送项目时候出现 “fatal: The remote end hung up unexpectedly ” 原因是推送的文件太大。
# **解决方案1：**

在克隆/创建版本库生成的 .git目录下面修改生成的config文件增加如下：
```
[http]  
postBuffer = 524288000
```

![image.png](https://upload-images.jianshu.io/upload_images/1464492-3e14ab3b132bcf93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重新推送即可。

**或者设置缓存**

```
$ git config http.postBuffer 524288000
$ git config --global http.postBuffer 524288000
$ git config --global https.postBuffer 1048576000
$ git push -f origin master
```

# 解决方案2：

使用git更新或提交中途有时出现The remote end hung up unexpectedly的异常，特别是资源库在国外的情况下。此问题可能由网络原因引起。

配置git的最低速度和最低速度时间：

```
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999  单位 秒
```

`--global`配置对当前用户生效，如果需要对所有用户生效，则用`--system`

# 解决方案3：

```
fatal: The remote end hung up unexpectedly | 7.00 KiB/s
```

这句显示 远程结束挂起 |7kb/s

```
应该是墙的原因导致网速太慢，且项目有点大上传不上
```

解决办法：`翻墙` 或者等等重新在push一遍 就没问题了
