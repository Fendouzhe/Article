这段时间有好多网友问到同一个问题：为什么pod search搜索不到MJRefresh或者MJExtension？原因是这样的：pod search只会搜索你本地缓存的框架，如果你想搜索到最新的第三方框架或者某个框架的最新版本，必须先使用pod repo update（推荐）或者pod setup将远程仓库的框架信息更新到本地。其实，从pod search的响应速度飞快，也可以猜出它并没有连接服务器，仅仅是搜索了本地的框架信息
    此外，如果你的框架更新比较慢，可以尝试执行下面2条指令更换镜像服务器
pod repo remove master
pod repo add master http://git.oschina.net/akuandev/Specs.git
    更换镜像完毕后，以后执行pod repo update的速度就会快很多

原文：http://blog.sina.com.cn/s/blog_800cdf9c0102ve7e.html
