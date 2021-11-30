[toc]

# 在Redhat7系列OS上搭建PXE服务SOP

## 1. PXE介绍

**PXE**（Pre-boot Execution Environment）预启动执行环境，是由Intel开发的网络技术，工作于client/server的网络模式，支持工作站通过网络从远端服务器下载镜像，并由此支持通过网络启动操作系统。

PXE不是安装的方式，而是一种引导方式。

### 1.1 PXE协议工作原理

<img src="..\images\image-20200910173655476.png" alt="image-20200910173655476" style="zoom:50%;" />

​	a.)协议分为PXE server和client两端，PXE client端集成在网卡ROM中,即PXE启动需要网卡ROM的支持。

​	b.)当client端计算机启动时，BIOS把PXE client端网络启动请求调入内存执行，并显示出启动菜单。

​	c.) 经用户选择后，PXE server进行响应，将预定的启动文件传输给PXE client。

​	d.) client端将server上的启动文件经过网络下载到本地运行

### 1.2 PXE启动流程

<img src="..\images\image-20200910174522895.png" alt="image-20200910174522895" style="zoom:87%;" />

 PXE工作原理示意图说明：

1. Client向PXE Server上的DHCP发送IP地址请求消息，DHCP检测Client是否合法（主要是检测Client的网卡MAC地址），如果合法则返回Client的IP地址，同时将启动文件pxelinux.0的位置信息一并传送给Client。

2. Client向PXE Server上的TFTP发送获取pxelinux.0请求消息，TFTP接收到消息之后再向Client发送pxelinux.0大小信息，试探Client是否满意，当TFTP收到Client发回的同意大小信息之后，正式向Client发送pxelinux.0。

   `注：在传统Intel X86平台的操作系统中，bootstrap文件通常为pxelinux.0；而在Arm架构的Linux操作系统中，bootstrap文件通常名称为grubaa64.efi。`

3. Client执行接收到的pxelinux.0文件。

4. Client向TFTP发送针对本机的配置信息（记录在TFTP的pxelinux.cfg目录下），TFTP将配置文件发回Client，继而Client根据配置文件执行后续操作。

5. Client向TFTP发送Linux内核请求信息，TFTP接收到消息之后将内核文件发送给Client。

6. Client向TFTP发送根文件请求信息，TFTP接收到消息之后返回Linux根文件系统。

7. Client启动Linux内核（启动参数已经在4中的配置文件中设置好了）。

8. Client通过NFS或HTTP或FTP下载镜像文件，读取autoyast自动化安装脚本。至此，Client正式进入自动化安装模式开始安装系统直到完成。

## 2. 准备服务器

- 准备一台机器，可以是X86服务器，也可以是Arm处理器平台的服务器，还可以是X86、Arm处理器架构的PC或者带有千兆电口的笔记本。

- 安装Redhat7系列的操作系统，例如：CentOS7、Redhat7、KylinV10等。尽量选择安装“带GUI的服务器”并安装所有组件（最大化安装）。

- 关闭防火墙和selinux

  ```sh
  systemctl stop firewalld.service
  systemctl disable firewalld.service
  setenforce 0
  sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
  ```

- 设置umask为0022，防止后续创建的目录和文件的权限不足

  ```sh
  umask 0022
  echo "umask 0022" >> /etc/profile && source /etc/profile
  ```

## 3. 安装相关服务

PXE方式安装OS依赖的服务：dhcpd、tftp、nfs、httpd。其中nfs和httpd可以二选一，不过方便起见，我们可以都安装上。

### 3.1 在线安装

如果服务器能够连通互联网，可以直接使用OS官网yum源在线安装，安装方法十分简单，直接在命令行终端下执行：

```bash
yum install dhcp tftp-server nfs-utils httpd
```

### 3.2 离线安装

如果服务器不能连通互联网，您需要首先配置一个本地yum源，然后再执行以上yum install命令。本地yum源的配置方法如下：

- 将操作系统iso镜像上传至服务器/opt目录下，然后执行以下命令：

  ```sh
  # 备份在线yum源
  mkdir -p /etc/yum.repos.d/bak
  mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/
  
  # 挂载iso镜像
  mkdir -p /opt/local-iso-dir
  mount -o loop /opt/Kylin-Desktop-V10-Release-Build1-20200710-arm64.iso /opt/local-iso-dir
  
  # 创建本地yum源，在/etc/yum.repos.d目录下创建文件local.repo，文件内容如下：
  cat /etc/yum.repos.d/local.repo 
  [local]
  name=kylin-0710-local
  baseurl=file:///opt/local-iso-dir
  gpgcheck=0
  enable=1
  
  # 通过yum makecache测试yum源是否配置成功，执行命令：
  yum makecache 
  kylin-0710-local                                                           3.7 MB/s | 3.8 kB     00:00    
  Metadata cache created.
  
  #然后就可以通过yum install安装相关服务了
  yum install dhcp tftp-server nfs-utils httpd
  ```

## 4. 服务配置

DHCPD、TFTP、NFS、HTTPD服务可以配置在一台服务器上，也可以部署在不同的服务器上。方便起见，我们一般部署在同一台机器上。

### 4.1 DHCP服务配置

- 选择服务器的一个网口提供DHCPD服务，将这个网口与待安装机器相连（可以直连，也可通过交换机连接）

- 配置DHCPD服务网口的IP地址，例如配置为：192.168.17.2

- 配置DHCPD配置文件，如下：

  ```sh
  cat /etc/dhcp/dhcpd.conf 
  ddns-update-style interim;
  ignore client-updates;
  next-server 192.168.17.2;                     ## 指定tftp服务的IP地址，由于tftp与dhcpd部署在了同一台机器，所以即：当前dhcpd服务器的IP地址 
  subnet 192.168.17.0 netmask 255.255.255.0 {   ## 指定子网和子网掩码，必须与当前服务器的IP在同一个网段
      option routers 192.168.17.1;              ## 指定网关
      range dynamic-bootp 192.168.17.3 192.168.17.253;  ## 能够为客户端提供的IP地址范围
      default-lease-time 21600;
      max-lease-time 43200;
  }
  ```

  `注：以上注释内容不要写入配置文件`

- 启动服务，`systemctl start dhcpd.service`

### 4.2 TFTP服务配置

我们知道/var/www/html目录是HTTPD服务的根目录，而TFTP服务的默认根目录为/var/lib/tftpboot，为了简化后续操作，我们将TFTP服务的根目录修改为/var/www/html。方法如下：

```sh
# 修改服务配置
sed -i 's#/var/lib/tftpboot#/var/www/html#' /usr/lib/systemd/system/tftp.service

# 启动服务
systemctl start tftp.service 
# 查看服务状态
systemctl status tftp.service 
● tftp.service - Tftp Server
   Loaded: loaded (/usr/lib/systemd/system/tftp.service; indirect; vendor preset: disabled)
   Active: active (running) since Tue 2021-01-19 17:42:54 CST; 2s ago
   ......
   CGroup: /system.slice/tftp.service
           └─40994 /usr/sbin/in.tftpd -s /var/www/html           ## 检查此处服务目录是否更改为/var/www/html

1月 19 17:42:54 localhost.localdomain systemd[1]: Started Tftp Server.
```

### 4.3 NFS服务配置

为了简化后续操作，我们将/var/www/html也配置为NFS服务的根目录，方法如下：

```sh
# 在exports配置文件中增加一行
cat /etc/exports
/var/www/html *(insecure,rw,sync,no_root_squash,no_subtree_check)

# 启动服务
systemctl status nfs.service
```

## 5. 针对不同机型的配置

以上搭建步骤适用于所有机型的PXE服务部署，属于通用配置步骤。本章节开始针对**不同的待安装机型**，进行配置说明。

### 5.1 CS5260F配置

#### 5.1.1 上传镜像文件

- 将需要安装的iso镜像上传至/opt/isos目录下，并临时挂载到/mnt目录

  ```sh
  mount -o loop /opt/isos/uniontechos-server-20-enterprise-1030-arm64.iso /mnt
  ```

- 在HTTPD、DHCPD、TFTP、NFS等服务的根目录/var/www/html下，为本机型创建如下目录结构

  ```sh
  mkdir -p /var/www/html/CS5260F/Kylin/20201202    ## 用于放置20201202版本的麒麟镜像内容
  mkdir -p /var/www/html/CS5260F/UOS/1030          ## 用于放置1030版本的UOS镜像内容
  
  rsync -a /mnt /var/www/html/CS5260F/UOS/1030     ## 将1030版本UOS镜像内容拷贝至1030目录下
  ```

#### 5.1.2 准备bootstrap文件及其配置文件

CS5260F机型为ARM64处理器平台，bootstrap文件名称为grubaa64.efi，此文件用于引导，使待安装机器的屏幕上显示出系统安装菜单，通常这个文件可以在Kylin或UOS的系统镜像中找到，随便使用其中之一即可。

`注：对于某些处理器平台，Kylin和UOS系统镜像中的EFI引导文件可能无法使用，需要向CPU厂商索取，例如：龙芯。`

如下所示，将UOS镜像中的grubaa64.efi文件拷贝至机型根目录下，此处为CS5260F目录：

```sh
cp /var/www/html/CS5260F/UOS/1030/EFI/BOOT/grubaa64.efi /var/www/html/CS5260F/ 
```

bootstrap文件需要读取一个配置文件，在Linux中通常名称为grub.cfg，这个文件决定了屏幕上显示出什么样的系统安装菜单。

`注：通常情况下将grub.cfg文件与grubaa64.efi文件放置在同一个目录下即可，但也有例外，此处不再展开讨论。`

一个典型的grub.cfg配置文件内容如下：

```sh
set default="0"

function load_video {
  if [ x$feature_all_video_module = xy ]; then
    insmod all_video
  else
    insmod efi_gop
    insmod efi_uga
    insmod ieee1275_fb
    insmod vbe
    insmod vga
    insmod video_bochs
    insmod video_cirrus
  fi
}

load_video
set gfxpayload=keep
insmod gzio
insmod part_gpt
insmod ext2

set timeout=10

menuentry 'Net Install Kylin 20201202' --class red --class gnu-linux --class gnu --class os {
    set root=(tftp,192.168.17.2)
    linux /CS5260F/Kylin/20201202/images/pxeboot/vmlinuz ro inst.geoloc=0 rd.iscsi.waitnet=0 ip=dhcp inst.repo=http://192.168.17.2/CS5260F/Kylin/20201202 inst.ks=http://192.168.17.2/CS5260F/Kylin/kylin-ks.cfg rd.debug rd.udev.debug systemd.log_level=debug
    initrd /CS5260F/Kylin/20201202/images/pxeboot/initrd.img
}

menuentry "Net Install UOS 1030" {
set gfxpayload=keep
linux /CS5260F/UOS/1030/live/vmlinuz console=tty boot=live netboot=nfs nfsroot=192.168.17.2:/var/www/html/NF2180M3/UOS/1030 ip=192.168.17.2 components union=overlay locales=zh_CN.UTF-8 livecd-installer 
initrd /CS5260F/UOS/1030/live/initrd.img
}
```

grubaa64.efi读取此grub.cfg配置文件，就会在待安装机器的屏幕上显示出系统安装菜单，如下：

​	**Net Install Kylin 20201202**

​	**Net Install UOS 1030**

#### 5.1.3 在DHCP服务中配置bootstrap的入口

根据前面介绍的PXE启动流程可知：DHCP服务除了能够为待安装机器分配IP地址之外，还能够告知待安装机器bootstrap文件的所在位置，然后待安装机器才能够通过TFTP文件传输协议到指定的位置获取bootstrap文件。

在DHCP服务中配置bootstrap文件位置的方法为：

```sh
cat /etc/dhcp/dhcpd.conf 
ddns-update-style interim;
ignore client-updates;
next-server 192.168.17.2;
filename "CS5260F/grubaa64.efi";               ## 增加此行，指定bootstrap文件的位置（这个位置是一个相对路径，相对于TFTP服务的根目录）  
subnet 192.168.17.0 netmask 255.255.255.0 {
    option routers 192.168.17.1;
    range dynamic-bootp 192.168.17.3 192.168.17.253;
    default-lease-time 21600;
    max-lease-time 43200;
}
```

#### 5.1.4 Kickstart无人值守配置

对于Redhat系列的OS，一般通过kickstart配置文件实现无人值守自动化安装，文件中配置：语言、时区、安装盘符、分区、用户名和密码等。例如：针对KylinV10，一个典型的配置内容如下：

```sh
# 安装盘符
ignoredisk --only-use=sda
autopart --type=lvm
# 自动清理已存在的分区
clearpart --drives=sda --all
# 使用图形化安装界面
graphical
# 键盘布局
keyboard --vckeymap=cn --xlayouts='cn'
# 语言
lang zh_CN.UTF-8
# 网络设置
network  --bootproto=dhcp --device=enp8s0f0 --ipv6=auto --activate
network  --hostname=localhost.localdomain
# 首次启动时进行初始化配置
firstboot --enable
# X Window System configuration information
xconfig  --startxonboot
# 采用chronyd时间同步服务
services --enabled="chronyd"
# 时区设置
timezone Asia/Shanghai --utc --nontp
%packages
# 安装带GUI的服务器
@^Server with UKUI GUI
# 安装wget命令
wget
%end
# 设置用户名和密码
# Root password
rootpw inspur123@
user --name=inspur --password=inspur123@
# 从MBR启动
bootloader --location=mbr

# 密码复杂度策略
%anaconda
pwpolicy root --minlen=8 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=8 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=8 --minquality=1 --notstrict --nochanges --notempty
%end

%post
# Get kyinfo
wget http://192.168.17.56/kylin/{.kyinfo,LICENSE} --random-wait --directory-prefix /etc/
# Enable kdump
sed -i "s/ crashkernel=auto / /" /boot/efi/EFI/kylin/grub.cfg
%end
```

注：对于Ubuntu系列的OS（例如：UOS），并不使用kickstart实现无人值守自动化安装。UOS无人值守安装需要向统信申请定制iso镜像。

此外，通过U盘或光盘安装完KylinV10后，系统下存在一个/root/anaconda-ks.cfg文件，这个文件即kickstart配置文件，可以用于替换到此处。例如：您不知道如何在Kickstart配置文件中设置自定义分区，那么，您可以首先采用U盘安装OS，进行自定义分区设置，那么生成的anaconda-ks.cfg中就记录了您的自定义配置，可以直接拿来用于PXE安装。

#### 5.1.5 小结

至此，对于CS5260F这个机型，一个较为完整的配置目录结构如下：

```sh
## tree命令可以展示目录结构
tree -L 3 /var/www/html/CS5260F/
/var/www/html/CS5260F/
├── grubaa64.efi      ## bootstrap文件
├── grub.cfg          ## bootstrap读取的配置文件
├── Kylin
│   ├── 20201202
│   ├── kylin-ks.cfg  ## kickstart无人值守配置文件
└── UOS
    └── 1030
        ├── boot
        ├── boot.cat
        ├── deepin-boot-maker.exe
        ├── deepin-boot-maker.zip
        ├── dists
        ├── EFI
        ├── live
        ├── md5check.sh
        ├── md5sum.txt
        ├── oem
        └── pool
```

为其他机型配置PXE时，只需要仿照这个目录结构，放置对应机型的配置文件即可。然后再在dhcpd.conf中配置对应的bootstrap入口。

## 6. 开始PXE安装OS

- 待安装机器与PXE服务器已建立物理网络连接

- 待安装机器BIOS中已开启网络引导开关

- 启动PXE相关服务

  ```sh
  systemctl start dhcpd tftp nfs httpd
  ```

- 待安装机器开机，按键F12，从网络引导

- 等待进入安装界面

- 自动化无人值守安装或手动安装

## 7. Troubleshooting

PXE安装过程中一旦遇到问题，可以通过在服务端`tail -f /var/log/messages`命令排障，messages日志中记录了从获取IP、下载efi文件、下载内核文件一系列过程。可以很容易地定位哪一步出了问题。