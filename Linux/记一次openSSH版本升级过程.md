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

如果最新的CentOS发行版中已经解决了高危BUG，可以通过yum update或upgrade直接升级，很简单。不过一定注意update和upgrade的区别！！！！

否则，通过最新源码安装，如下：

#### 卸载openssl

[root@vbds ~]# rpm  -qa  |grep openssl
openssl-libs-1.0.1e-51.el7_2.4.x86_64
openssl-1.0.1e-51.el7_2.4.x86_64
openssl098e-0.9.8e-29.el7.centos.3.x86_64

#### 升级openssl





[root@vic-0 yum.repos.d]# uname -a
Linux vic-0 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux

