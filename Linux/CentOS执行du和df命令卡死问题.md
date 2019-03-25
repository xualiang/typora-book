- **1.  在CentOS上执行以下命令查看硬盘使用状态：**

- df -h

- 发现命令卡住不动，Ctrl+c也无法终结该命令。

-  

- **2.  安装strace:**

- yum install  strace

- strace是一个非常简单的工具，它可以跟踪系统调用的执行。最简单的方式，它可以从头到尾跟踪binary的执行，然后以一行文本输出系统调用的名字，参数和返回值。

-  

- **3.  通过strace跟踪df命令执行情况：**

- strace df -h

-  

- 1. ………………

- 1. stat("/sys/fs/cgroup/hugetlb", {st_mode=S_IFDIR|0755, st_size=0, ...}) = 0
  2. stat("/sys/kernel/config", {st_mode=S_IFDIR|0755, st_size=0, ...}) = 0
  3. stat("/", {st_mode=S_IFDIR|0555, st_size=4096, ...}) = 0
  4. stat("/proc/sys/fs/binfmt_misc", ^C

- 该命令在stat("/proc/sys/fs/binfmt_misc"处卡住，我们到该目录看一下情况。

- 执行：

- cd /proc/sys/fs

- ls

- ls命令同样卡住，问题就出在该目录了。

-  

- **4  .解决办法**

- 网上查了一下，是挂载方面的问题，执行以下命令可以解决：

- systemctl restart  proc-sys-fs-binfmt_misc.automount