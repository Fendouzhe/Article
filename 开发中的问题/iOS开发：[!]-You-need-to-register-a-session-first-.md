将我们的gitHub仓库代码配置CocoPods支持的时候

执行最后一步命令：
```
pod trunk push 工程名.podspec
```
报错如下：
```
[!] You need to register a session first.
```

解决方案命令如下：
```
pod trunk register 电子邮箱 '您的姓名' --description='macbook pro'
```

成功提示如下：
```
[!] Please verify the session by clicking the link in the verification email that has been sent to llrongvip@163.com
```

然后就可以进入邮箱确认了,成功的话验证一下：
```
pod trunk me
```
