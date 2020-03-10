2019年9月28日，为了迎接建国70周年国庆节，昌乐县公安局对服务器进行了安全扫描，发信我们的服务器的openSSH有很多漏洞，需要升级版本。过程如下：

### 检查当前的版本

[root@vbds ~]# ssh -V
OpenSSH_6.6.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013

[root@vbds ~]# openssl version
OpenSSL 1.0.1e-fips 11 Feb 2013

[root@vbds ~]# uname -a
Linux vbds.xucl.telchina 3.10.0-327.13.1.el7.x86_64 #1 SMP Thu Mar 31 16:04:38 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux

### 安装telnet服务

yum install telnet-server.x86_64

systemctl start telnet.socket

### 允许root用户telnet

\#vi  /etc/pam.d/remote

如图注释这一行

auth   required     pam_securetty.so

然后重启telnet.socket

### 升级

#### yum方式升级

如果最新的CentOS发行版中已经解决了高危BUG，可以通过yum update或upgrade直接升级，很简单。不过一定注意update和upgrade的区别！！！！

否则，通过最新源码安装，如下：

#### 源码方式升级

1. 备份：`mv /etc/ssh/ ./ssh.bak`
2. 安装编译所需的依赖：`yum -y install gcc zlib-devel openssl-devel`
3. 下载源码、编译

下载地址：https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/

```sh
$ tar -zxf openssh-7.4p1.tar.gz
$ cd openssh-7.4p1/
$ ./configure --prefix=/usr --sysconfdir=/etc/ssh
$ make
```

4. 卸载旧版：

```sh
$ rpm -e --nodeps `rpm -qa | grep openssh`
```

5. 安装并检查：`make install`， `ssh -V`
6. 配置开机启动：

```sh
$ cp contrib/redhat/sshd.init /etc/init.d/sshd
$	chkconfig --add sshd
```

7. 设置root访问权限

```sh
$ sed -i '/#PermitRootLogin prohibit-password/c'"PermitRootLogin yes" /etc/ssh/sshd_config
```

8. 重启sshd服务
9. ssh登录成功后，关闭telnet服务

