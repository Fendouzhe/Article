## 第一步：创建模板库
```
$ pod lib create LRLibrary
```
报错误提示: 
```
.rvm/rubies/ruby-2.3.3/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require’:
 cannot load such file – colored2 (LoadError)
```
![错误提示.png](https://upload-images.jianshu.io/upload_images/1464492-5b978adfbe7c3281.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输入下面提示的colored2的两条gem命令即可解决问题 
```
$ sudo gem install colored2
$ sudo gem update –system
```

完成后继续执行自己的命令 
```
$ pod lib create LRLibrary
```
## 第二步：创建模板库步骤填写
```
Cloning `https://github.com/CocoaPods/pod-template.git` into `LRLibrary`.
Configuring LRLibrary template.
! Before you can create a new library we need to setup your git credentials.

 What is your email?
 > llrongvip@163.com

! Setting your email in git to llrongvip@163.com
  git config user.email "llrongvip@163.com"

------------------------------

To get you started we need to ask a few questions, this should only take a minute.

2018-06-06 15:43:58.289 defaults[5850:1346329] 
The domain/default pair of (org.cocoapods.pod-template, HasRunbefore) does not exist
If this is your first time we recommend running through with the guide: 
 - https://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and double click links to open in a browser. )

 Press return to continue.


What platform do you want to use?? [ iOS / macOS ]
 > iOS

What language do you want to use?? [ Swift / ObjC ]
 > ObjC

Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

Would you like to do view based testing? [ Yes / No ]
 > No

What is your class prefix?
 > LR

Running pod install on your new library.

Analyzing dependencies
Fetching podspec for `LRLibrary` from `../`
Downloading dependencies
Installing LRLibrary (0.1.0)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `LRLibrary.xcworkspace` for this project from now on.
Sending stats
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.

 Ace! you're ready to go!
 We will start you off by opening your project in Xcode
  open 'LRLibrary/Example/LRLibrary.xcworkspace'

To learn more about the template see `https://github.com/CocoaPods/pod-template.git`.
To learn more about creating a new pod, see `http://guides.cocoapods.org/making/making-a-cocoapod`.
```
## 第三步：模板库创建成功，工程会自动启动
然后代码写在Pods/Development/LRLibrary/Classes里面：

![1 创建类.png](https://upload-images.jianshu.io/upload_images/1464492-da342d833abfaf21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第四步：GitHub创建项目LRLibrary
![2 github创建项目.png](https://upload-images.jianshu.io/upload_images/1464492-c9dd88d58ba58760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第五步：LRLibrary.podspec
![3 编辑LRLibrary.podspec .png](https://upload-images.jianshu.io/upload_images/1464492-318e769671c6df79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第六步：上传github
```
$ git remote add origin https://github.com/Fendouzhe/LRLibrary.git
$ git push -u origin master -f
```

![4 上传github.png](https://upload-images.jianshu.io/upload_images/1464492-b52c5d53e755c4f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第七步：设置tag号，提交修改（只要LRLibrary.podspec文件发生改变，就要重新提交tag，注意版本号）
```
1. git add .
2. git commit -m "修改了podspec 文件"
3. git tag 0.1.0(对应LRLibrary.podpsec 中的版本号)   (添加tag)
4. git push --tags    （推送tag到远程）
5. git push origin master  （推送到远程代码仓库）
```

## 第八步：验证.podspec 文件是否合法
```
1. pod spec lint LRLibrary.podspec  
2. pod spec lint LRLibrary.podspec --allow-warnings  (忽略警告,推荐使用)
```
如果遇到错误，需要修改错误，否则不能提交
验证成功则出现:

```
 -> LRLibrary (0.1.0)
    - WARN  | summary: The summary is not meaningful.

Analyzed 1 podspec.

LRLibrary.podspec passed validation.
```

## 第九步：修改错误前，删除刚刚上传的tag版本， 修改错误之后重新添加tag版本（即执行第七步）
```
1.  git tag -d 0.1.0  // 删除本地tag
2.  git push origin -d tag 0.1.0  // 删除远程仓库的tag
```
## 第十步： 提交框架到cocoapods
```
1. pod trunk push LRLibrary.podspec  
2. pod trunk push LRLibrary.podspec --allow-warnings   (忽略警告，推荐使用)
```

显示如下则上传成功!
```
--------------------------------------------------------------------------------
 🎉  Congrats

 🚀  LRLibrary (0.1.0) successfully published
 📅  June 6th, 04:09
 🌎  https://cocoapods.org/pods/LRLibrary
 👍  Tell your friends!
--------------------------------------------------------------------------------
```

## 第十一步：cocoapods：pod search 无法搜索到类库的解决办法

```
[!] Unable to find a pod with name, author, summary, or description matching `LRLibrary`
```
![image.png](https://upload-images.jianshu.io/upload_images/1464492-6b4e9c214423a168.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/880)

```
1. pod setup //   成功后会生成 ~/Library/Caches/CocoaPods/search_index.json文件
2. rm  ~/Library/Caches/CocoaPods/search_index.json   // 删除该文件需要检查是否被成功删除
3. pod search LRLibrary  // 重新生成  ~/Library/Caches/CocoaPods/search_index.json 文件
```

至此已经将框架上传完毕
等待一段时间后就可在cocoapods网站上查询自己的框架（https://cocoapods.org/）

## 第十二步：引用(在以后的项目工程目录下 执行)
```
1. pod init   // 生成 Podfile 文件
2. 修改Podfile文件集成 pod  'LRLibrary', '~> 0.1.0'
```
