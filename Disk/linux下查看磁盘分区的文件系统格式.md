```sh
$ df -T ## 只可以查看已经挂载的分区和文件系统类型。
Filesystem Type 1K-blocks Used Available Use% Mounted on
/dev/sda1 ext4 20642428 3698868 15894984 19% /
tmpfs tmpfs 32947160 0 32947160 0% /dev/shm

$ fdisk -l ## 可以显示出所有挂载和未挂载的分区，但不显示文件系统类型。
Disk /dev/sda: 299.4 GB, 299439751168 bytes
255 heads, 63 sectors/track, 36404 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000576df
Device Boot Start End Blocks Id System
/dev/sda1 * 1 2611 20971520 83 Linux
/dev/sda2 2611 3134 4194304 82 Linux swap / Solaris
/dev/sda3 3134 36404 267248282 83 Linux

$ parted -l ## 可以查看未挂载的文件系统类型，以及哪些分区尚未格式化。
Model: LSI MR9240-8i (scsi)
Disk /dev/sda: 299GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Number Start End Size Type File system Flags
1 1049kB 21.5GB 21.5GB primary ext4 boot
2 21.5GB 25.8GB 4295MB primary linux-swap(v1)
3 25.8GB 299GB 274GB primary ext4

$ lsblk -f ## 也可以查看未挂载的文件系统类型。
NAME FSTYPE LABEL UUID MOUNTPOINT
sda 
|-sda1 ext4 c4f338b7-13b4-48d2-9a09-8c12194a3e95 /
|-sda2 swap 21ead8d0-411f-4c23-bdca-642643aa234b [SWAP]
`-sda3 ext4 2872b14e-45va-461e-8667-43a6f04b7bc9
 
$ file -s /dev/sda3
/dev/sda3: Linux rev 1.0 ext4 filesystem data (needs journal recovery) (extents) (large files) (huge files)

$ fsck -N /dev/sda1
fsck from util-linux 2.23.2
[/sbin/fsck.xfs (1) -- /boot] fsck.xfs /dev/sda1

```

