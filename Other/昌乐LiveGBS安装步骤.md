### 下载：

https://www.liveqing.com/docs/download/LiveGBS.html

### 上传：

上传到：37.88.99.70:/mnt/disk01/liveGBS

### 解压：

tar xf **LiveCMS-linux-2.1.0-1910181616.tar.gz**

tar xf **LiveSMS-linux-2.1.0-1910181616.tar.gz**

### 配置并启动CMS：

cd /mnt/disk01/liveGBS/LiveCMS-linux-2.1.0-1910181616

Vim livecms.ini

sh start.sh

* sip-id ：37072500002001000008
* sip-域：3707250000
* sip-host：37.88.99.70
* sip-端口：5060
* 设备统一接入密码：空（必需为空）

### 启动SMS

### 在页面上配置SMS

### 国标级联-添加上级平台：

* 名称：深瞐解析平台（上级）
* sip服务国标编码：37072500002000000010
* sip服务国标域：3707250000
* sip服务ip：37.88.99.69
* sip服务端口：5060
* sip认证用户：37072500002001000008
* sip认证密码：12345678
* 信令传输：UDP
* 字符集：GB2312

### 可能需要重启seemmo服务

### 测试：

登录liveGBS服务器37.88.99.70，执行：

```sh
tcpdump -i em1 -A -vvv host 37.88.99.69 and port 5060
```

