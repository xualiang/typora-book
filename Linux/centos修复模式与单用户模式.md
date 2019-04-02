今天修改LVM逻辑卷的名称时候，忘记更改fstab配置文件了，导致机器重启后找不到盘，进不了系统！立即用光盘进入修复模式进行修复！

 ### 1.修复模式操作方法：

 用光盘进入Linux修复模式，插入centos光盘选择第三项：Rescue installed system

然后修改fstab

注意进入修复模式后fstab路径为

**vi /mnt/sysimage/etc/fstab**

### 2.单用户模式操作方法

进入Linux单用户模式

执行 root# mount -o remount,rw /

然后/etc/fstab就可以修改了