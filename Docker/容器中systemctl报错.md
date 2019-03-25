刚开始接触Docker的朋友，可能会遇到这么一个问题，使用centos7镜像创建容器后，在里面使用systemctl启动服务报错。针对这个报错，我们接下来就分析下！

```bash
docker run -itd --name centos7 centos:7
docker attach centos7
yum install vsftpd
systemctl start vsftpd
Failed to get D-Bus connection: Operation not permitted
```

不能启动服务，什么情况？

难道容器不能运行服务嘛！！！

 

答：

Docker的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到systemd，这个套件已经成为主流Linux发行版（比如CentOS7、Ubuntu14+）默认的服务管理，取代了传统的SystemV风格服务管理。systemd维护系统服务程序，它需要特权去会访问Linux内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问Linux内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器设计原则，一个容器里运行一个前台服务！

 

我就想这样运行，难道解决不了吗？

答：可以，以特权模式运行容器。

 

创建容器：

```bash
docker run -d -name centos7 --privileged=true centos:7 /usr/sbin/init
```

进入容器：

```bash
docker exec -it centos7 /bin/bash
```

这样可以使用systemctl启动服务了。