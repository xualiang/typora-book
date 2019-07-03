## ubuntu下的apt内网本地源的正确搭建

**16.04的系统使用14.04的源，提示执行apt-get install -f,这条命令千万不要执行。在此记录一下**

APT本地源的搭建（可用于局域网apt-get源搭建或者本地源） 
本文档介绍使用apt-mirror软件搭建apt本地源 
需求：内网开发环境由于其特定原因不能上外网，所以需要本地环境下的内网源来方便开发人员下载安装软件 
建议：单独使用一块磁盘来存放源文件或者单独一个目录下，避免混淆 

## 服务端配置

## 1、安装apt-mirror

```sh
apt-get install apt-mirror
```

## 2、修改apt-mirror配置文件

在修改配置文件之前，我们首先要确定自己系统的版本，命令：`sudo lsb_release -a`

```bash
$sudo lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 16.04 LTS
    Release:    16.04
    Codename:   xenial
```

Codename代号的意思，16.04代号xenial，所以我们接下来的配置文件跟xenial有关，当然14.04代号是trusty，一样的操作。 
打开 [阿里云源](http://mirrors.aliyun.com/ubuntu/)(也可以使用别的源，网址可以自己百度) 
进入dists目录，在目录下找到跟系统代号相关问文件夹，一般是5个，将下面规则文本复制出来，把加粗部分替换成相应的5个文件目录名。进入这5个目录，里面有4个跟源有关的目录（by-hash除外），目录名与下面斜体部分比较，如果不一样请修改。

> deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) **xenial** *main restricted universe multiverse* 
> deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) **xenial-security** *main restricted universe multiverse* 
> deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) **xenial-updates** *main restricted universe multiverse* 
> deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) **xenial-proposed** *main restricted universe multiverse* 
> deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) **xenial-backports** *main restricted universe multiverse*

然后

```
vim /etc/apt/mirror.list
```

参考以下配置文件： 
清空原有的配置文件，修改以下配置文件相应代号部分即可，如果想添加多个版本的源，可以依次在下面增加相应的规则（就是增加对应代号的源地址）

```bash
############# config ##################
# 以下注释的内容都是默认配置，如果需要自定义，取消注释修改即可
set base_path /var/spool/apt-mirror
#
# 镜像文件下载地址
# set mirror_path $base_path/mirror
# 临时索引下载文件目录，也就是存放软件仓库的dists目录下的文件（默认即可）
# set skel_path $base_path/skel
# 配置日志（默认即可）
# set var_path $base_path/var
# clean脚本位置
# set cleanscript $var_path/clean.sh
# 架构配置，i386/amd64，默认的话会下载跟本机相同的架构的源
set defaultarch amd64
# set postmirror_script $var_path/postmirror.sh
# set run_postmirror 0
# 下载线程数
set nthreads 20
set _tilde 0
#
############# end config ##############
# Ali yun（这里没有添加deb-src的源）
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

clean http://mirrors.aliyun.com/ubuntu
```

## 3、开始同步

```sh
执行 apt-miiror
```

然后等待很长时间（该镜像差不多100G左右，具体时间看网络环境），同步的镜像文件目录为/var/spool/apt-mirror/mirror/mirrors.aliyun.com/ubuntu/，当然如果增加了其他的源，在/var/spool/apt-mirror/mirror目录下还有其他的地址为名的目录。

注意：当apt-mirror 被意外中断时，只需要重新运行即可，apt-mirror支持断点续存；另外，意外关闭，需要在/var/spool/apt-mirror/var目录下面删除 apt-mirror.lock文件【 sudo rm apt-mirror.lock 】，之后执行apt-mirror重新启动

## 4、安装apache2

```
apt-get install apache2
```

由于Apache2的默认网页文件目录位于/var/www/html，因此，可以做个软链接（这样我们就可以直接访问了，无需将其直接导入该目录）

```
ln -s /var/spool/apt-mirror/mirror/mirrors.aliyun.com/ubuntu /var/www/html/ubuntu
```

然后就可以通过如下地址访问了

```
    http://127.0.0.1/ubuntu   
#ip和port是自己本机的，其中端口默认为80
在测试时可能遇到打不开的情况，查看下iptables规则是否限制或者selinux的问题（这点相信大家在学习lanmp的时候都已经了解过了）
```



## 客户端配置：


1、编辑/etc/apt/source.list，参考以下内容（以下是64位机，ubuntu16.04），修改相应的代号,硬件架构arch，加入文件中

```
# Local Source 　　　　 #ip和port是自己本机的，其中端口默认为80
deb [arch=amd64] http://[host]:[port]/ubuntu/ xenial main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ xenial-security main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ xenial-updates main restricted universe multiverse  
deb [arch=amd64] http://[host]:[port]/ubuntu/ xenial-proposed main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ xenial-backports main restricted universe multiverse
```

2、更新apt-get源

```
sudo apt-get update　　　　#这步很重要
sudo apt-get upgrade
```