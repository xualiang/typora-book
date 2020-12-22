# UOS Arm Desktop

netbootaa64.efi的位置与dhcp配置文件中对应；grub.cfg的位置必须是tftp根目录下的/boot/grub/目录下



最好不用netbootaa64.efi，而使用UOS镜像中的grubaa64.efi，grub.cfg放到与grubaa64.efi同一目录下即可



将tftpboot目录及子目录设置为755：find /var/lib/tftpboot -type d -exec chmod 755 {} \;

文件权限设置为644：find /var/lib/tftpboot -type f -exec chmod 644 {} \;



利用html目录，作为tftp nfs



oem

settings.ini

default_settings.ini

filesystem.squashfs

unsquashfs

mksquashfs

squashfs-root/usr/share/deepin-installer