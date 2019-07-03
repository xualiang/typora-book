# etherwake 远程唤醒

 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/a924068818/article/details/85095701

本文章的前提是，你的设备已经设置好允许局域网内唤醒，至于怎么设置，本文不做讲解。

1. 设备的网卡（必须是有线网卡）必须设置了允许唤醒设备，允许魔术包唤醒。
2. BIOS中开启了wake-on-lan/wol/等等，

那么，唤醒的咒语就是 （xx:xx:xx:xx:xx:xx 替换为你要唤醒设备的网卡的mac地址 ）

```
etherwake -b xx:xx:xx:xx:xx:xx 
1
```

可能出现的错误：

```
SIOCGIFHWADDR on eth0 failed: No such device
1
```

出现这个错误，说明你的路由器或者唤醒设备默认的网卡不是eth0，这时你需要加上-i参数，值为网卡的名称，修改如下(当然，eth2换成你自己的网卡名称)

```
etherwake -b xx:xx:xx:xx:xx:xx  -i eth2
```



# 网络唤醒-ether-wakeup

网络唤醒功能就是实现在局域网中计算机能够接受特定的数据包实现开机的操作

原理：计算机在关机状态时，其实电源还提供给+5V的电压给主板的部分芯片使用， 比如网卡，局域网中的电脑可以发送一个特定的数据包让关机但依然通电的计算机网卡接收，然后进行开机操作

一般这种数据包时已广播形式发送的，IP是255.255.255.255（局域网内广播地址）而这种数据包就是wol magic packet

使用网络唤醒， 被唤醒机器需要满足以下条件

1.使用ATX电源

2.主板提供网络唤醒的硬件和软件支持

3.网卡支持wol

如果是集成网卡，只需要主板支持就行了 ， 如果是pci网卡， 在主板和网卡会有三针的wol跳线跳线，需要将其连接好

网络唤醒工具

linux下 ether-wake

ether-wake在net-tools中集成

ether-wake 是一个linux下生成和发wake-on-lan(WOL)“Magic Packet”的工具，能够唤醒处于soft-powered-down(ACPI D3-warm 状态)下的设备，ether-wake会生成一个标准的AMD“Magic Packet"

Options

-b 发送网络唤醒包到广播地址

-D  增加Debug

-i ifname 使用的网卡名称

-p passwd 添加一个4byte或6byte的密码到包中

-V 显示软件版本信息

返回状态：

如果发送成功会返回 0 ，如果是权限问题不能发送会返回2 ，无法识别或者是非法的参数会返回3 ，没有找到网卡信息或者没有找到发送网卡去发送数据包会返回1

例：ether-wake -i eth0 -b00:22:44:66:88:aa