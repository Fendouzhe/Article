mac下hosts文件在 /etc/hosts。所以先打开终端。然后输入如下命令,打开hosts文件。
```
$ su vim /etc/hosts
```
输入mac管理员密码，就可以使用vim打开hosts文件。将如下github的host放到hosts文件中。在 vim 编辑中，按 “i”建进入插入模式，就可以将下面的 hosts 修改粘贴进入到hosts文件中。
```
http://github.com 204.232.175.94 http://gist.github.com 107.21.116.220
http://help.github.com 207.97.227.252 http://nodeload.github.com 199.27.76.130
http://raw.github.com 107.22.3.110 http://status.github.com 204.232.175.78
http://training.github.com 207.97.227.243 http://www.github.com
```
然后按 “ESC”键,输入 “shift + ;”，将vim切换到保存模式。然后输入 “wq” 保存hosts文件。
