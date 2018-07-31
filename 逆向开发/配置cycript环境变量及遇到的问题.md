

> 包已经解压了，但是只能输入绝对路径或者相对路径才能执行，想要直接输入命令执行需要配置环境变量

1.  启动终端Terminal
2.  进入当前用户的home目录

```
$ cd ~

```

3.  创建.bash_profile（如果有该文件跳过此步骤）

```
$ touch .bash_profile

```

4.  编辑.bash_profile文件

```
$ open -e .bash_profile
或者喜欢vim的同学
$ vim .bash_profile

```

增加相对应的绝对路径，例如：（可能每个人电脑配置不一样）

```
export cycript_src="你的cycript绝对路径"
PATH=$PATH:$cycript_src

```

5.  保存文件，关闭.bash_profile
6.  更新刚配置的环境变量

```
$ source .bash_profile

```

7.  验证配置是否成功

```
$ cycript

```

8.因为是iterm2+oh my zsh组合，需要在.zshrc配置文件中导入

```
$ open -e .zshrc
或者
$ vim .zshrc

```

在里面加入`source .bash_profile`这行命令，这样每次打开就不用手动执行了

搞定！

* * *

## 执行后遇到了这个问题：

```
dyld: Library not loaded: /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/libruby.2.0.0.dylib
  Referenced from: /Users/devzkn/Downloads/cycript_0.9.594/Cycript.lib/cycript-apl
  Reason: image not found

```

这个错误是因为电脑的ruby版本太高导致

#### 1.首先查看电脑ruby版本

```
$ ruby -v

```

我电脑上的版本是2.3

> 感谢楼里的兄弟提醒，有可能你的电脑安装了rvm，用`ruby -v`命令查看的是指定的ruby版本，正确做法应该是cd到/System/Library/Frameworks/Ruby.framework/Versions/ 目录下查看具体版本

具体命令：

```
$ cd /System/Library/Frameworks/Ruby.framework/Versions/
$ ls

```

#### 2.关闭系统的SIP

> 在 OS X El Capitan 中有一个跟安全相关的模式叫 SIP（System Integrity Protection ），它禁止让软件以 root 身份来在 Mac 上运行，在升级到 OS X 10.11 中或许你就会看到部分应用程序被禁用了，这些或许是你通过终端或者[第三方软件](https://link.jianshu.com?t=https%3A%2F%2Fwww.baidu.com%2Fs%3Fwd%3D%25E7%25AC%25AC%25E4%25B8%2589%25E6%2596%25B9%25E8%25BD%25AF%25E4%25BB%25B6%26from%3D1012015a%26fenlei%3Dmv6quAkxTZn0IZRqIHckPjm4nH00T1dBm1F-PAc3P1-hmymsuWmd0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnW0kn1mznjm3n1nsn1n3PW6d)源安装。对于大多数用户来说，这种安全设置很方便，但是也有些开发者或者高级 Mac 用户不需要这样的设置。

*   电脑重启按住command+R，进入恢复模式
*   打开终端，输入`csrutil disable`，重启
*   如果想打开SIP，重复上两步，命令改为`csrutil enable`

#### 3.直接把2.3的复制一份，改为2.0即可

运行如下命令：

```
sudo mkdir -p /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/
sudo ln -s /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/libruby.2.3.0.dylib /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/libruby.2.0.0.dylib

```

> 注：根据每个人ruby版本不同，将上面第二条命令的`/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/libruby.2.3.0.dylib`中的2.3改成本机的ruby版本。
> 这里不是降级ruby，只是复制一份2.0的ruby的dylib，让cycript运行起来。
