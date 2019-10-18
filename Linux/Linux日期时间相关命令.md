- ## 一、显示日期、时间、时区等相关信息

- [root@node1 ~]# timedatectl

- ### 二、修改时间

- timedatectl set-time HH:MM:SS

- 备注：同时修改hardware clock和system  clock.

- ### 三、修改日期

- timedatectl set-time YYYY-MM-DD

-  备注：修改日期不同时修改时间，将重置现在的时间为00:00:00.

- 例子：

- [root@node1 ~]# timedatectl set-time 2016-04-18
     [root@node1 ~]# date
     2016年 04月 18日 星期一 00:00:01 CST

- ### 四、同时修改日期和时间

- timedatectl set-time 'YYYY-MM-DD HH:MM:SS'

- ### 五、修改时区

- 执行如下命令显示可用的时区：

- timedatectl list-timezones

-  

- 执行如下命令设置使用的时区：

- timedatectl set-timezone time_zone

- [root@node1 ~]# timedatectl set-timezone Asia/Shanghai
     [root@node1 ~]# ls -lrt /etc/localtime
      lrwxrwxrwx 1 root root 35 4月  17 00:08 /etc/localtime -> ../usr/share/zoneinfo/Asia/Shanghai

- ### 六、同步时间到一个远程服务器

- timedatectl命令可以用来控制是否开启NTP,开启NTP将启动chronyd或者ntpd服务，依赖于被安装的那个。

- 开启：

- timedatectl set-ntp yes

- 关闭：

- timedatectl set-ntp no

- 备注：

- - 执行set-ntp时会同时开启或关闭ntpd或者chronyd服务。但是ntpd服务和chronyd可以通过systemctl命令来单独控制，不是必须使用timedatectl来进行控制。

-  

- - 如果使用set-ntp是yes的状态（即：timedatectl命令中NTP enabled状态显示为yes，那么将不能同时使用set-time来修改时间。

- timedatectl set-ntp yes会启动chronyd服务，并且设置为开机自启动，并导致ntpd状态变为inactive，导致无法使用ntpd同步时间；timedatectl set-ntp yes并不会启动ntpd

-  

- 正确启用ntp同步的方法：

-   timedatectl set-ntp true 

-    systemctl stop chronyd.service 

-    systemctl disable chronyd.service 

-    systemctl restart ntpd

-    systemctl enable ntpd

- ### 七、将硬件时钟调整为与系统时钟一致:

- timedatectl  set-local-rtc true

- timedatectl set-local-rtc false设置为UTC时间 

- ### 八、将日期写入CMOS：

- clock –w

-  

- ### 九、查看修改时区

- 查看时区：

- date -R

-  

- 修改时区：

- ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

### 十、系统时间和硬件时间同步

　　以系统时间为基准，修改硬件时间，执行命令hwclock --systohc或者hwclock -w。

　　以硬件时间为基准，修改系统时间，执行命令hwclock --hctosys或者hwclock -s。

　　备注：系统重启后，系统将会修改为硬件时间。