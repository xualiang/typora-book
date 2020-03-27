## 本地apt源配置

```sh
$ mount ubuntu-16.04.5-server-amd64.iso /media/cdrom/
$ vim /etc/apt/source.list
deb file:///media/cdrom/ xenial main
$ apt-get update
```

## http方式的系统源

完成本地apt源配置后，首先安装apache2服务：

```sh
$ apt-get install apache2
```

可以修改apache2的端口号：

```sh
$ vim /etc/apache2/ports.conf
ubuntu@rke-worker03:/etc/apache2$ cat /etc/apache2/ports.conf
...
Listen 10080
<IfModule ssl_module>
	Listen 10443
</IfModule>
...
```

启动apache2服务：`service apache2 start`

制作http方式的apt源：

```sh
$ cp -r /media/cdrom/dists /var/www/html/
$ cp -r /media/cdrom/pool /var/www/html/
```

客户端主机使用：

```sh
$ vim /etc/apt/source.list
deb [arch=amd64] http://10.10.71.83:10080 xenial main
```

## http方式的非系统源

有一些软件并不在ubuntu的ISO镜像中，以openssh-server为例：

首先在一台联网的虚拟机(搭建虚拟机太慢，可以利用docker)中安装相同的系统，然后制作离线安装包：

```sh
一、下载deb安装包
$ sudo apt-get clean all  ###先清理一下缓存，即清理/var/cache/apt文件夹下的内容
$ sudo apt-get -d install openssh-server  ## 下载安装包
另外一种下载命令：
$ apt install openssh-server --reinstall -d

二、新建openssh文件夹，将上述下载的deb包(连同archives文件夹)拷入。
mkdir openssh
cp /var/cache/apt/archives openssh
sudo chmod 777 -R openssh

三、生成依赖关系
$ sudo dpkg-scanpackages /openssh/ /dev/null |gzip >/openssh/Packages.gz
# 您可能需要提前安装dpkg-dev这个软件包，才能使用dpkg-scanpackages命令
# 注意请不要修改Packages.gz这个命名，然后将其拷到openssh/archives下。
$ mv openssh/Packages.gz openssh/archives/
```

本地使用方式：

```sh
1、将openssh文件夹用U盘拷到离线机器的根目录下。 
2、修改系统源source.list（注意之前要备份）

$ sudo vim /etc/apt/sources.list
将里面内容删掉，加入：
deb file:/// /openssh/archives/
# 注意： archives后面有一个斜杠，全路径前面还要有空格

3、更新系统源
$ sudo apt-get update --allow-insecure-repositories
### 或者deb [trusted=yes] file:/// /openssh/archives/，则不用使用参数：--allow-insecure-repositories
#有时可能还会在archives目录下生成一个空的Rlease文件，需要删掉

4、安装软件
$ sudo apt-get install openssh-server -f --allow-unauthenticated
# 本地的源是没有签名的，直接更新ubuntu1604下的apt会提示找不到release文件，是一种不安全的源，默认是被禁用的。如果还要安装的话需要加上 --allow-unauthenticated 选项。
```

http方式：

1. 将openssh文件夹拷贝到一台具有http服务的机器的/var/www/html/目录下
2. 客户端机器使用：

```sh
$ vim /etc/apt/sources.list
deb http://10.10.10.25/  openssh/ 
$ apt-get update
```