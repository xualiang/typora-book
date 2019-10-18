```sh
无法上网的机器是Ubuntu15.04,我在联网的虚拟机(搭建虚拟机太慢，可以利用docker)中安装了相同的系统，然后制作离线安装包。 
一、下载deb安装包

$ sudo apt-get clean all  ###先清理一下缓存，即清理/var/cache/apt文件夹下的内容
$ sudo apt-get -d install openssh-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  libck-connector0 ncurses-term openssh-client openssh-sftp-server ssh-import-id
Suggested packages:
  libpam-ssh keychain monkeysphere rssh molly-guard
The following NEW packages will be installed:
  libck-connector0 ncurses-term openssh-server openssh-sftp-server ssh-import-id
The following packages will be upgraded:
  openssh-client
1 upgraded, 5 newly installed, 0 to remove and 328 not upgraded.

二、新建openssh文件夹，将上述下载的deb包(连同archives文件夹)拷入。
mkdir openssh
cp /var/cache/apt/archives openssh
sudo chmod 777 -R openssh

三、生成依赖关系
$ sudo dpkg-scanpackages /openssh/ /dev/null |gzip >/openssh/Packages.gz
# 您可能需要提前安装dpkg-dev这个软件包，才能使用dpkg-scanpackages命令
# 注意请不要修改Packages.gz这个命名，然后将其拷到openssh/archives下。
$ mv openssh/Packages.gz openssh/archives/

四、离线机器上安装 
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

