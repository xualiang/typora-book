## Ubuntu apt iso镜像 本地源配置

## 1.1 操作系统

```
## 本文操作系统 Ubuntu 16.04 amd64，用以下命令查看版本代号
# lsb_release -a
```

# 2、apt 本地镜像源配置

## 2.1 镜像源挂载

```
## 创建挂载目录，挂载镜像源
# mkdir /mnt/cdrom
# mount /mnt/ubuntu-16.04.2-server-amd64.iso /mnt/cdrom/
```

## 2.2 源备份

```
## 备份原有的源配置文件
# cd /etc/apt
# mv -v source.list{,.bak}
```

## 2.3 设置本地源

```
## 设置本地源，baseurl中file路径对应挂载的路径
# cat /etc/apt/source.list
deb file:///mnt/cdrom xenial main
```

# 3、apt 源操作

## 3.1 清除原有记录

```
# apt-get clean all
```

## 3.2 更新apt源

```
# apt-get update
```

## 3.3 获取安装列表

```
# apt-cache dump 
```