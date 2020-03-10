不能将一台服务器的配置推送到其他服务器，会造成无法启动。

 恢复方法：

1. 进入单用户模式或救援模式

2. ls /dev/disk/by-uuid 或者 blkid /dev/sdb1

3. vim /etc/fstab

4. reboot

可能造成服务器无法启动的配置文件：

* /etc/fstab
* /etc/rc.d/rc.local
* /etc/sysctl.conf 及 /etc/sysctl.d目录
* /etc/security/limits.conf 及 /etc/security/limits.d目录