### 检查所有的启动选单，如下：

```shell
[root@telchina194 ~]# cat /boot/grub2/grub.cfg | grep -A 15 "menuentry '"
#menuentry 'CentOS Linux (3.10.0-327.28.3.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-0fdc8f74-1073-423b-913c-944d2236f2c7' {
#       load_video
#       set gfxpayload=keep
#       insmod gzio
。。。
#       linux16 /vmlinuz-3.10.0-327.28.3.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=zh_CN.UTF-8
#       initrd16 /initramfs-3.10.0-327.28.3.el7.x86_64.img
#}
menuentry 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-0fdc8f74-1073-423b-913c-944d2236f2c7' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
。。。
        linux16 /vmlinuz-3.10.0-327.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=zh_CN.UTF-8
        initrd16 /initramfs-3.10.0-327.el7.x86_64.img
}
menuentry 'CentOS Linux (0-rescue-49036e7c4218428285296ea8c62c4766) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-49036e7c4218428285296ea8c62c4766-advanced-0fdc8f74-1073-423b-913c-944d2236f2c7' {
        load_video
        insmod gzio
。。。
        linux16 /vmlinuz-0-rescue-49036e7c4218428285296ea8c62c4766 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet
        initrd16 /initramfs-0-rescue-49036e7c4218428285296ea8c62c4766.img
}

### END /etc/grub.d/10_linux ###
```

### 检查默认的启动选单：

```sh
[root@telchina194 ~]# cat /etc/default/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

### 设置默认的启动选单：

```shell
[root@telchina194 ~]# grub2-set-default 0
[root@telchina194 ~]# grub2-editenv list
saved_entry=0
```

