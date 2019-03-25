Linux中Swap（交换分区），类似于Windows的虚拟内存，就是当内存不足的时候，系统会把一部分硬盘空间虚拟成内存使用,从而解决内存容量不足的情况。

 

　　如果内存够大，应当告诉 linux 不必太多的使用 SWAP 分区， 可以通过修改 swappiness 的数值。swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。

　　在ubuntu里面，默认设置swappiness这个值等于60。

 

1.查看当前swappiness值

　$ cat /proc/sys/vm/swappiness

 

2.修改swappiness值为10（临时修改，重启后即还原为默认值）

　$ sudo sysctl vm.swappiness=10

 

3.永久修改swappiness默认值（重启生效）

$ sudo gedit /etc/sysctl.conf

在文档的最后加上:

　　vm.swappiness=10

保存重启，搞定收工！

 