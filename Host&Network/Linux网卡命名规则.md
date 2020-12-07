## 网口命名问题的由来

> 服务器通常有多块网卡，有板载集成的，同时也有插在PCIe插槽的。Linux系统的命名规则原来是eth0,eth1这样的形式，但是这个编号往往不一定准确对应网卡接口的物理顺序。

为了解决这个问题，**DELL**开发了**biosdevname**方案，Linux的**systemd v197**版本中将DELL的方案作了进一步的一般化拓展，CentOS7既支持DELL的方案也支持systemd的方案。

DELL方案需要BIOS的特殊支持，且系统中安装biosdevname，并且grub启动参数中设置biosdevname=1，这样才能生效，通常用于DELL的服务器中，所以为了便于大家理解，本文不对这个方案进行过多讨论。

## 命名规范

先把简单的、容易理解的，说在前头，先说一下网口的命名规范，如下：

### net.ifnames的命名规范

当grub启动参数中设置net.ifnames=1时，采用这种规范，即**systemd的命名规范**，命名方式为：**设备类型 + 设备位置 + 数字**

**设备类型：**

| 类型代号 | 含义           |
| -------- | -------------- |
| en       | Ethernet       |
| wl       | WLAN           |
| ww       | 无线广域网WWAN |

**举例说明：**

| 网口名称        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| eno1            | 板载网卡，**o是onboard的意思**。                             |
| ens33           | 采用PCI-E槽位号来命名网口，**s是slot的意思**。               |
| enp6s0f0        | 采用网卡所连接的物理位置（en + p<bus>s<slot>f<function>）来命名，**p是PCIE总线的意思**。 |
| wlp3s0          | PCI无线网卡                                                  |
| enx78e7d1ea46da | 采用Mac地址来命名网口，x代表Mac Address                      |

### biosdevname的命名规范

em1         板载网卡

p3p4        pci网卡

p3p4_1    虚拟网卡

## 命名逻辑

### 情形一：当net.ifnames=0且biosdevname=0时

这种grub配置下，第一次启动：

1. 首先，采用ethX命名规则随机命名各个网口，

   ```txt
   例如四个网口被命名为：eth0、eth1、eth2、eth3
   ```

2. 然后，依据规则文件**/usr/lib/udev/rules.d/60-net.rules**，使用**/lib/udev/rename_device**这个程序，去查询/etc/sysconfig/network-scripts/下所有以**ifcfg-开头的文件**，如果在ifcfg-xx中匹配到HWADDR=xx:xx:xx:xx:xx参数的网卡接口则选取**DEVICE=yyyy**中设置的名字作为网卡名称。

   ```tex
   假如在第1步中，eth0对应的Mac为12:34:56:78:9a，而在本次启动前，有一个ifcfg-enp6s0f0文件中配置了HWADDR=12:34:56:78:9a，DEVICE=enp6s0f0，那么在这一步中，eth0将会被rename为enp6s0f0。
   所有网口最终变为：enp6s0f0、eth1、eth2、eth3
   ```

保持这种grub配置不变，无论**重启**多少遍，所有网口名称及对应的Mac地址都保持**不变**。

这种方式也是**旧版本Linux**的网口命名方式。

### 情形二：当net.ifnames=1且biosdevname=0时

新版Linux不修改grub配置，**默认**就是这种方式。这也是本文的重点。

这种grub配置下启动时：

​	前2步，与net.ifnames=0且biosdevname=0时相同，先随机命名为ethX，再匹配 ifcfg-XX 文件。但是还会多出以下步骤：

3. 依据规则文件**/lib/udev/rules.d/75-net-description.rules**，**udev**通过检查网卡信息，填写每个网卡的如下属性的值：

   - ID_NET_NAME_ONBOARD
   - ID_NET_NAME_SLOT
   - ID_NET_NAME_PATH
   - ID_NET_NAME_MAC

   系统启动后，可以使用如下命令检查某个网口的这些属性值：

   ```sh
   [root@localhost ~]# udevadm info /sys/class/net/enp6s0f0/
   P: /devices/pci0000:00/0000:00:00.0/0000:01:00.0/0000:02:09.0/0000:06:00.0/net/enp6s0f0
   L: 0
   E: DEVPATH=/devices/pci0000:00/0000:00:00.0/0000:01:00.0/0000:02:09.0/0000:06:00.0/net/enp6s0f0
   E: INTERFACE=enp6s0f0
   E: IFINDEX=2
   E: SUBSYSTEM=net
   E: USEC_INITIALIZED=19985199
   E: ID_NET_NAMING_SCHEME=v243
   E: ID_NET_NAME_MAC=enx6c92bfed709c
   E: ID_OUI_FROM_DATABASE=Inspur Electronic Information Industry Co.,Ltd.
   E: ID_NET_NAME_PATH=enp6s0f0
   E: ID_BUS=pci
   E: ID_VENDOR_ID=0x8086
   E: ID_MODEL_ID=0x1521
   E: ID_PCI_CLASS_FROM_DATABASE=Network controller
   E: ID_PCI_SUBCLASS_FROM_DATABASE=Ethernet controller
   E: ID_VENDOR_FROM_DATABASE=Intel Corporation
   E: ID_MODEL_FROM_DATABASE=I350 Gigabit Network Connection
   E: ID_MM_CANDIDATE=1
   E: ID_PATH=pci-0000:06:00.0
   E: ID_PATH_TAG=pci-0000_06_00_0
   E: ID_NET_DRIVER=igb
   E: ID_NET_LINK_FILE=/usr/lib/systemd/network/99-default.link
   E: SYSTEMD_ALIAS=/sys/subsystem/net/devices/enp6s0f0
   E: TAGS=:systemd:
   ```

   

4. 根据规则文件**80-net-setup-link.rules**（旧一点的版本可能是**80-net-name-slot.rules**），udev根据step3中网卡的各个属性值，按照指定的**scheme规则**，去给在**step2中没有命名的网卡**重命名。

   >scheme规则简单理解为：
   >
   >- 如果ID_NET_NAME_ONBOARD有值，则使用该值将网口命名为enoX，否则看ID_NET_NAME_SLOT是否有值；
   >- 如果ID_NET_NAME_SLOT有值，则命名为ensX，否则继续看ID_NET_NAME_PATH是否有值；
   >- 如果ID_NET_NAME_PATH有值，则命名为enp<bus>s<slot>f<function>
   >- ID_NET_NAME_MAC，一般不使用Mac地址来命名
   >- 如果对于一个网口，以上属性值都不存在，那么继续使用step1中随机生成的ethX
   >
   >可以简单认为命名顺序为：**enoX -> ensX -> enpXsXfX，使用先命中的**。

### 情形三：当biosdevname=1时

这种grub配置一般只适用于DELL服务器，所以我们只简单说一下：

1. step1和step2与前面完全相同

2. 只不过在step3之前，按照biosdevname的命名规范，从BIOS中取相关信息来命名step2中未重命名的网口，将其rename为emX。规则文件为/usr/lib/udev/rules.d/71-biosdevname.rules（只有安装了biosdevname后才会有）。

   > 主要是取SMBIOS中的type 9 (System Slot) 和 type 41 (Onboard Devices Extended Information)，不过要求SMBIOS的版本要高于2.6，且系统中要安装biosdevname程序。

3. 之后的过程与net.ifnames取值有关，如果是0，那么就不会执行后续步骤，如果取值为1，步骤与之前相同，继续对剩余的、未重命名的网口采用systemd的命名规则。

### 情形四：当创建了自定义规则时

以上三种情形都有一个前提，就是：不存在自定义的规则文件。

我们知道**/usr/lib/udev/rules.d/**目录下放置的都是系统的默认规则文件，而我们可以在**/etc/udev/rules.d**目录下创建自定义的规则文件，而且当存在冲突时，优先使用自定义的规则文件。

例如：

一种自定义规则方法，例如对一个网口采用自定义的名称：

```sh
[root@localhost ~]# cat /etc/systemd/network/10-xucl.link
[Match]
Path=pci-0000:03:00.0  # Path的值取自udevadm info命令中的ID_PATH的值

[Link]
Name=eth3s0f0          # 随便自定义你的网口名称
```

禁用60-net.rules + ifcfg-X规则的方法：

```sh
ln -s /dev/null /etc/udev/rules.d/60-net.rules    ## 这样即使存在ifcfg-X文件，也不会去匹配.
```

一种禁用新版systemd命名规则，延用旧规则的方法：

```sh
[root@localhost ~]# ln -s /dev/null /etc/systemd/network/99-default.link
这样就会回归eth0、eth1、eth2、eth3的命名规则，当然有一个前提：不存在ifcfg-enp6s0f0这样的文件，如果有，网口仍然会被命名为enp6s0f0，删除这样的ifcfg文件，即可回归ethX这种命名。
[root@localhost ~]# ln -s /dev/null /etc/udev/rules.d/80-net-setup-link.rules
这样也可以，具有相同的效果
```

通过大量测试发现，无论在任何场景下，只要存在默认的60-net.rules文件和ifcfg-X文件，**最终都会根据ifcfg-X文件进行对应网口的命名，也就是说60-net.rules + ifcfg-X优先级最高**，即使进行了自定义规则。可以认为ifcfg-X也是一种自定义规则，而且优先级最高。

## 附录

### 几个rule文件说明

```sh
[root@localhost rules.d]# cat 60-net.rules 
ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{type}=="1", PROGRAM="/lib/udev/rename_device", RESULT=="?*", NAME="$result"

[root@localhost rules.d]# cat 75-net-description.rules 
# do not edit this file, it will be overwritten on update

ACTION=="remove", GOTO="net_end"
SUBSYSTEM!="net", GOTO="net_end"

IMPORT{builtin}="net_id"

SUBSYSTEMS=="usb", IMPORT{builtin}="usb_id", IMPORT{builtin}="hwdb --subsystem=usb"
SUBSYSTEMS=="usb", GOTO="net_end"

SUBSYSTEMS=="pci", ENV{ID_BUS}="pci", ENV{ID_VENDOR_ID}="$attr{vendor}", ENV{ID_MODEL_ID}="$attr{device}"
SUBSYSTEMS=="pci", IMPORT{builtin}="hwdb --subsystem=pci"

LABEL="net_end"

[root@localhost rules.d]# cat 80-net-setup-link.rules 
# do not edit this file, it will be overwritten on update

SUBSYSTEM!="net", GOTO="net_setup_link_end"

IMPORT{builtin}="path_id"

ACTION!="add", GOTO="net_setup_link_end"

IMPORT{builtin}="net_setup_link"

NAME=="", ENV{ID_NET_NAME}!="", NAME="$env{ID_NET_NAME}"

LABEL="net_setup_link_end"

[root@localhost rules.d]# cat /usr/lib/systemd/network/99-default.link
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Match]
OriginalName=*

[Link]
NamePolicy=keep kernel database onboard slot path
#NamePolicy=keep kernel database slot path
MACAddressPolicy=persistent
```

