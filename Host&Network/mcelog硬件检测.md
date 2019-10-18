mcelog 是 x86 的 Linux 系统上用来检查硬件错误，特别是内存和CPU错误的工具。

比如服务器隔一段时间莫名的重启一次，而message和syslog又检测不到有价值的信息。

 

通常发生MCE报错的原因有如下：

1、内存报错或者ECC问题

2、处理器过热

3、系统总线错误

4、CPU或者硬件缓存错误

 

一般来说当有错误提示时，需要优先注意内存问题，但由于现在内存控制器是集成在cpu里，所以有个别情况是由CPU问题引起的

 

一、如果是联网的情况下，yum源配置可用则

yum install mcelog

然后运行

service mcelogd start

mcelog  --daemon

查看日志方式

/var/log/mcelog

 

故障重启日志如下：

MCE 0

HARDWARE ERROR. This is NOT a software problem!

Please contact your hardware vendor

CPU 1 BANK 8 TSC 1193fd60c6699 [at 2000 Mhz 1 days 18:56:49 uptime (unreliable)]

MISC 8f44960800095840 ADDR 4a9f3b1c0

MCG status:

MCi status:

Error overflow

MCi_MISC register valid

MCi_ADDR register valid

MCA: MEMORY CONTROLLER RD_CHANNELunspecified_ERR

Transaction: Memory read error

Memory read ECC error

Memory corrected error count (CORE_ERR_CNT): 18

Memory transaction Tracker ID (RTId): 40

Memory DIMM ID of error: 1

Memory channel ID of error: 0

Memory ECC syndrome: f449608

STATUS cc0004800001009f MCGSTATUS 0

 

 

Mcelog相关文件

/dev/mcelog 设备文件

/var/log/mcelog    messages日志文件

/etc/mcelog/mcelog.conf配置文件

/var/run/mcelog.pid

 

默认故障日志只记录在/var/log/mcelog，并不记录到系统日志中。

如果需要在系统日志中也体现，需修改/etc/mcelog/mcelog.conf文件，将前面#去掉，并保存。

 

Mcelog相关设置

1.mcelog的随系统启动，查看boot下的config文件，可以看到mce模块随机启动

2.配置mcelog后台运行

\#mcelog --daemon

3.查看mcelo

 

由于各厂家服务器内存和CPU槽位设计可能不同，定位可能不准