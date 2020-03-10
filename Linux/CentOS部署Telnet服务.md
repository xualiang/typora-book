## 安装

```bash
$ yum install telnet-server
```

## 开通端口 

```bash
vim /etc/services
```

去掉23端口注释

## 设置Telnet服务状态

```bash
vim /etc/xinetd.d/telnet
```

将disable=yes改为disable=no

## 配置权限

```bash
vi /etc/pam.d/login
```

auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so注释掉这一行

## 开通Telent控制台

```bash
vi /etc/securetty 
```

文件末尾添加:

pts/1 

pts/2 

...

## 重启服务

```bash
service xinetd restart
```