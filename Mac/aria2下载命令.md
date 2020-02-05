# Mac下载神器aria2

## 什么是aria2

[aria2官网](https://link.jianshu.com/?t=https://aria2.github.io)介绍:

> aria2 is a lightweight multi-protocol & multi-source command-line download utility. It supports HTTP/HTTPS, FTP, SFTP, BitTorrent and Metalink. aria2 can be manipulated via built-in JSON-RPC and XML-RPC interfaces.

简单来说就是有以下特性:

- 支持多协议: HTTP / HTTPS，FTP，SFTP，BitTorrent和Metalink
- 多线程连线：aria2 会自动从多个线程下载文件，并充分利用你的带宽；
- 轻量：运行时不会占用过多资源，根据官方介绍，内存占用通常在 4MB~9MB，使用BitTorrent 协议，下行速度 2.8MB/s 时 CPU 占用率约 6%；
- 全功能 BitTorrent 客户端，可以当BT客户端使用，抛弃迅雷。
- 支持 RPC 界面远程控制

## aria2的安装与使用

### 安装

方法有二

1. 从[aria2官网](https://link.jianshu.com/?t=https://aria2.github.io)下载对应系统的安装包，双击安装。
2. 使用homebrew安装: `brew install aria2`

### 配置

aria2 提供两种方式使用，一种是直接命令行模式下载，不推荐使用这种方法，推荐使用另外一种 RPC 模式，这种方式 aria 启动之后只会安静的等待下载请求，下载完成后也只会安静的驻留后台不会自动退出。而使用RPC模式推荐做一个配置文件方便使用。
新建一个名为`aria2.conf`的配置文件，复制下面的内容到文件内。

```bash
    #用户名
    #rpc-user=user
    #密码
    #rpc-passwd=passwd
    #上面的认证方式不建议使用,建议使用下面的token方式
    #设置加密的密钥
    #rpc-secret=token
    #允许rpc
    enable-rpc=true
    #允许所有来源, web界面跨域权限需要
    rpc-allow-origin-all=true
    #允许外部访问，false的话只监听本地端口
    rpc-listen-all=true
    #RPC端口, 仅当默认端口被占用时修改
    #rpc-listen-port=6800
    #最大同时下载数(任务数), 路由建议值: 3
    max-concurrent-downloads=5
    #断点续传
    continue=true
    #同服务器连接数
    max-connection-per-server=5
    #最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
    min-split-size=10M
    #单文件最大线程数, 路由建议值: 5
    split=10
    #下载速度限制
    max-overall-download-limit=0
    #单文件速度限制
    max-download-limit=0
    #上传速度限制
    max-overall-upload-limit=0
    #单文件速度限制
    max-upload-limit=0
    #断开速度过慢的连接
    #lowest-speed-limit=0
    #验证用，需要1.16.1之后的release版本
    #referer=*
    #文件保存路径, 默认为当前启动位置
    dir=/User/xxx/Downloads
    #文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用, 需要1.16及以上版本
    #disk-cache=0
    #另一种Linux文件缓存方式, 使用前确保您使用的内核支持此选项, 需要1.15及以上版本(?)
    #enable-mmap=true
    #文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
    #所需时间 none < falloc ? trunc << prealloc, falloc和trunc需要文件系统和内核支持
    file-allocation=prealloc
```

默认下载路径的「/Users/xxx/Downloads」可以改为任何你想要的绝对路径。此处写为 Downloads 目录，xxx 请自行替换成你的 Mac 用户名，然后保存，退出编辑器。

### 启动

`aria2c --conf-path="/Users/xxx/Documents/aria2.conf" -D`

kill aria2之后按 Tab 键，aria2 会自动变成进程号，回车即可杀掉它。

### 使用

启动完成就需要开始下载了，这就需要一个管理界面了。有两个很优秀的在线工具[YAAW](https://link.jianshu.com/?t=http://binux.github.io/yaaw/demo/)和[Aria2 WebUI](https://link.jianshu.com/?t=http://ziahamza.github.io/webui-aria2/#)

**注:** YAAW需要在界面的设置里输入`http://localhost:6800/jsonrpc`，Aria2 WebUI打开可直接使用。

## 高阶使用

如果经常在百度云上下载东西，配合[BaiduExporter](https://link.jianshu.com/?t=https://github.com/acgotaku/BaiduExporter)这个扩展就会非常方便好用了。
首先下载扩展，Chrome，Firefox和Safari都有:

- [Chrome](https://link.jianshu.com/?t=https://chrome.google.com/webstore/detail/baiduexporter/mjaenbjdjmgolhoafkohbhhbaiedbkno)

- [Firefox](https://link.jianshu.com/?t=https://addons.mozilla.org/zh-CN/firefox/addon/baiduexporter)

- [Firefox（XPI)](https://link.jianshu.com/?t=https://raw.githubusercontent.com/acgotaku/BaiduExporter/master/firefox/BaiduExporter.xpi)：下载后打开 Firefox，Ctrl/Command + O 打开选择文件对话框选中 XPI 包即可安装。

- Safari

  ：下载后双击安装即可。

  安装完成后进入百度云的下载界面，会发现网页上多出一个「导出下载」按钮，点击它弹出的「ARIA2 RPC」就自动添加到你的下载队列里了。

  ![img](../images/webp-20200130032952750)

  百度云界面

然后我们可以在[Aria2 WebUI](https://link.jianshu.com/?t=http://ziahamza.github.io/webui-aria2/#)管理界面看到下载信息了。

![img](../images/webp-20200130032952926)

下载管理界面



## 感谢

这篇文章主要参考了[少数派](https://link.jianshu.com/?t=http://sspai.com)里的[Mac 上使用百度网盘很烦躁？花点时间配置 aria2 吧](https://link.jianshu.com/?t=http://sspai.com/32167)这篇文章和[BaiduExporter](https://link.jianshu.com/?t=https://github.com/acgotaku/BaiduExporter)作者的介绍。感谢他们的总结和付出。