```sh
apt-cdrom -m -d=/mnt/cdrom add  #添加源

1.apt-cache search package #搜索包

2.apt-cache show package #获取包的相关信息，如说明，大小，版本。

3.apt-cache depends package #了解使用依赖

4.apt-get rdepends package #查看该包被那些包依赖

5.sudo apt-get install package #安装包

6.sudo apt-get install package=version #安装制定版本的包

7.sudo apt-get install package --reinstall #重新安装包

8.sudo apt-get -f install #修复安装(17.10.31,之前小看这个东东了，这个是启动APT自动安装依赖关系的一个功能键，换句话说，你更新完源之后，如果APT还不能自行解决依赖关系，就可以执行一下这个命令)

9.apt-get source package #下载该包的源代码
apt-get install -d -o=dir::cache=/tmp apache2 #下载deb包，如果本地已经有了，就不会下载了，例如本地系统iso镜像中的deb是不会下载的，一般这个命令用于从互联网下载，然后制作成apt源，供离线环境下使用

10.sudo apt-get remove package #删除包

11.sudo apt-get remove package --purge #删除包,包括删除配置文件等

12.sudo apt-get update #更新apt软件源数据库

13.sudo apt-get upgrade #更新以安装的包

14.sudo apt-get dist-upgrade #升级系统

15.sudo apt-get dselect-upgrade #使用dselect升级

16.sudo apt-get build-dep package #安装相关的编译环境

17.sudo apt-get clean & sudo apt-get autoclean #清理无用的包

18.sudo apt-get check #检查是否有损坏的依赖

apt list --installed #列出所有已安装的软件包
dpkg -l

apt-get install --print-uris build-essential  #打印依赖包的下载地址


```

