## Linux内核日志级别

级别从0至7，严重程度逐级变轻。例如：loglevel=5时，console中将会打印出包含0-4级别的日志，5、6、7级别的日志不会打印到console中。

- 0 (KERN_EMERG)
  The system is unusable. 

  系统不可用

- 1 (KERN_ALERT)
  Actions that must be taken care of immediately. 

  必须立即采取行动

- 2 (KERN_CRIT)
  Critical conditions. 

  关键错误（硬件错误或软件错误）

- 3 (KERN_ERR)
  Noncritical error conditions.

  非关键性错误（例如设备识别失败或有问题，或者更一般的与驱动程序相关的问题）

- 4 (KERN_WARNING)
  Warning conditions that should be taken care of.

  大多数 linux 发行版的**默认级别**，警告信息

- 5 (KERN_NOTICE)
  Normal, but significant events.

  一般消息，值得关注的事件

- 6 (KERN_INFO)
  Informational messages that require no action.

  关于内核执行的操作的信息性消息

- 7 (KERN_DEBUG)
  Kernel debugging messages, output by the kernel if the developer enabled debugging at compile time.

  打印所有内核信息，一般用于调试

KERN_ERR, KERN_DEBUG等是一些宏定义，在$Linux_SRC/include/linux/printk.h中可以查看到。

## 检查当前内核日志级别

查看方法：

```sh
$ cat /proc/sys/kernel/printk  ## /proc是一个虚拟文件系统，其中包含的文件并不在硬盘上，而是由内核在内存中创建并维护的，用于直观的展示系统状态.
4	4	1	7
```

- 第一个值是当前console_loglevel。 值4意味着只有严重性级别高于此级别的消息才会显示在控制台上。
- 第二个值表示default_message_loglevel。 此值自动用于没有特定日志级别的消息：如果消息与日志级别没有关联，则将使用该值。
- 第三个值报告了minimum_console_loglevel状态。 它指示可用于console_loglevel的最低日志级别。 这里使用的级别是1，最高。
- 最后一个值表示default_console_loglevel，它是引导时用于console_loglevel的默认日志级别。

也可以使用sysctl命令来查看相同的信息：

```sh
$ sysctl kernel.printk
kernel.printk = 7	4	1	7
```

## 修改内核日志级别

最直接的方法：

```sh
$ echo "3" > /proc/sys/kernel/printk
$ cat /proc/sys/kernel/printk
3       4       1       7
```

也可以使用命令：

```sh
$ sysctl -w kernel.printk=3
```

或者

```sh
$ dmesg -n 3  # 注意：demsg -n level 改变的是console上的loglevel，dmesg命令仍然会打印出所有级别的系统信息。
```

>dmesg是从kernel的ring buffer(环缓冲区)中读取信息的.
>man dmesg得到如下信息：
>`*dmesg is used to examine or control the kernel ring buffer.
>The program helps users to print out their bootup messages. Instead of copying the messages by hand, the user need only: dmesg > dmesg.log*`
>
>那什么是ring buffer呢?
>在LINUX中,所有的系统信息(包内核信息)都会传送到ring buffer中。而内核产生的信息由printk()打印出来。系统启动时所看到的信息都是由该函数打印到屏幕中。printk()打出的信息往往以 <0>...<2>... 这样的数字表明消息的重要级别。高于一定的优先级别（当前的console loglevel）就会打印到console上，否则只会保留在系统的缓冲区中(ring buffer)。
>至于dmesg具体是如何从ring buffer中读取的,大家可以看dmesg.c源代码。$Linux-SRC/arch/m68k/tools/amiga/dmesg.c，很短，比较容易读懂。

还可以使用syslog系统调用在C程序中临时修改当前内核日志级别，如下：

```c
#include <unistd.h>
#include <sys/syscall.h>
 
static inline int syslog(int type, char *bufp, int len)
{
	return syscall(SYS_syslog, type, bufp, len);
}
 
int main()
{
	/* Set the console loglevel to 4 */
	syslog(8, 0, 4);
 
	return 0;
}
```

但是以上几种方法只是**临时生效**，计算机重启后，不会使用新设置的值。

要以持久方式更改默认日志级别，必须修改**/etc/default/grub文件**，并在启动时将loglevel参数传递给内核命令行：

```shell
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="loglevel=3 resume=UUID=df5a0685-43f8-433a-8611-57335a10ca8d"    ## 添加loglevel=3
GRUB_DISABLE_RECOVERY="true"
```

修改文件并保存后，我们必须重新加载grub，以便在下次重新启动时应用新配置，执行此操作的命令取决于我们正在运行的发行版。 通常，该命令是：

```sh
$ grub2-mkconfig -o /boot/grub2/grub.cfg  ## 执行完这个命令后，查看/boot/grub2/grub.cfg文件，可以发现kernel日志级别已改变。
# 其实，简单起见，直接修改grub.cfg文件，也是可以的。注意：不同的发行版，grub.cfg文件位置和名称可能不同。
```

debian系中命令为：

```sh
$ update-grub
```

grub配置将被更新，并且在下次重新引导时，将采用指定的日志级别作为默认级别。

## 参考

https://linuxconfig.org/introduction-to-the-linux-kernel-log-levels