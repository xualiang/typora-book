## 启动至grub菜单
开机至grub菜单页面
## 修改grub菜单
在第一个启动菜单项上按e，进入编辑页面，然后修改启动参数：
FROM：
```sh
linux     /boot/vmlinuz-4-4.0-22-generic root=UUID=43ad24d3-e\
c5b-44ee-a099-a88eb9520989 ro  quiet splash $vt_handoff
```

TO:
```sh
linux     /boot/vmlinuz-4-4.0-22-generic root=UUID=43ad24d3-e\
c5b-44ee-a099-a88eb9520989 rw init=/bin/bash
```
然后按ctrl+x开始启动
## 修改密码
按照如上操作后，将会进入一个bash命令行界面，而且此时根分区也被挂载为可读可写，可以通过以下命令检查：
```sh
mount | grep -w /
```
修改root密码：
```sh
passwd
```
## 重启系统
使用以下命令重启系统：
```sh
exec /sbin/init
```
## Troubleshooting
### 问题一：
```sh
Enter new UNIX password:
Retype new UNIX password:
passwd: Authentication token manipulation error
passwd: password unchanged
```
问题原因：
修改grub参数时，未修改为rw
解决方法：
```sh
mount -o remount,rw /
```
### 问题二：
```sh
[ end Kernel panic - not syncing: Attempted to kill init! exit code=0x0007f00
```
问题原因：
修改grub参数时，未去掉splash
### 问题三：
```sh
Failed to connect to bus: No such file or directory
Failed to talk to init daemon.
```
问题原因：
不能使用reboot重启，只能通过`exec /sbin/init`重启