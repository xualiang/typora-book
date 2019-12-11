### 安装FastDFS

每个节点需要安装：

libfastcommon-master.zip

fastdfs-5.11.zip

### 配置tracker和storage

### 安装nginx

### Nginx配置

### 部署基于FastDFS的Drawer

### Dispose工程中配置

application.properties中：

```properties
telchina.saveFileFlag=true
telchina.drawer=http://IP:9999/  ##nginx代理总地址
```

增加fast-client.conf文件：

```properties
http.tracker_http_port=6666
tracker_server=IP1:22122
tracker_server=IP2:22122
tracker_server=IP3:22122
```

### Web工程配置

drawer地址修改为：

http://IP:9999/  ##nginx代理总地址

