不能将一台服务器的配置推送到其他服务器，会造成无法启动。

 

恢复方法：

 

ls /dev/disk/by-uuid 或者 blkid /dev/sdb1

 

vim /etc/fstab

 

reboot

