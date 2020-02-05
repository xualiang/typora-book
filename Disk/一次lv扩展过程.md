```sh
$ fdisk -l

Disk /dev/sda: 1099.5 GB, 1099511627776 bytes, 2147483648 sectors
...
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   167772159    83373056   8e  Linux LVM

Disk /dev/mapper/centos-root: 51.6 GB, 51636076544 bytes, 100851712 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 8455 MB, 8455716864 bytes, 16515072 sectors
...
Disk /dev/mapper/centos-home: 25.2 GB, 25211961344 bytes, 49242112 sectors
...

$ fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
Partition number (3,4, default 3): 
First sector (167772160-2147483647, default 167772160): 
Using default value 167772160
Last sector, +sectors or +size{K,M,G} (167772160-2147483647, default 2147483647): 
Using default value 2147483647
Partition 3 of type Linux and of size 944 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

$ fdisk -l

Disk /dev/sda: 1099.5 GB, 1099511627776 bytes, 2147483648 sectors
...
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   167772159    83373056   8e  Linux LVM
/dev/sda3       167772160  2147483647   989855744   83  Linux

Disk /dev/mapper/centos-root: 51.6 GB, 51636076544 bytes, 100851712 sectors
...
Disk /dev/mapper/centos-swap: 8455 MB, 8455716864 bytes, 16515072 sectors
...
Disk /dev/mapper/centos-home: 25.2 GB, 25211961344 bytes, 49242112 sectors
...

$ vgextend centos /dev/sda3
  Device /dev/sda3 not found (or ignored by filtering).
  Unable to add physical volume '/dev/sda3' to volume group 'centos'.
报错：lsblk检查未发现sda3
$ lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk 
sda               8:0    0    1T  0 disk 
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 79.5G  0 part 
  ├─centos-root 253:0    0 48.1G  0 lvm  /
  ├─centos-swap 253:1    0  7.9G  0 lvm  [SWAP]
  └─centos-home 253:2    0 23.5G  0 lvm  /home
sr0              11:0    1 1024M  0 rom  
执行partprobe重新发现分区表，问题解决，如下：
$ partprobe 
$ lsblk     
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk 
sda               8:0    0    1T  0 disk 
├─sda1            8:1    0  500M  0 part /boot
├─sda2            8:2    0 79.5G  0 part 
│ ├─centos-root 253:0    0 48.1G  0 lvm  /
│ ├─centos-swap 253:1    0  7.9G  0 lvm  [SWAP]
│ └─centos-home 253:2    0 23.5G  0 lvm  /home
└─sda3            8:3    0  944G  0 part 
sr0              11:0    1 1024M  0 rom 
$ vgextend centos /dev/sda3
  Physical volume "/dev/sda3" successfully created
  Volume group "centos" successfully extended
$ lvextend /dev/mapper/centos-root /dev/sda3 
  Size of logical volume centos/root changed from 48.09 GiB (12311 extents) to 992.09 GiB (253974 extents).
  Logical volume root successfully resized.                                                
$ xfs_growfs /dev/mapper/centos-root 
meta-data=/dev/mapper/centos-root isize=256    agcount=4, agsize=3151616 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=12606464, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=6155, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 12606464 to 260069376
```

