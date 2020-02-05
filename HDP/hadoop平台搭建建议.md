https://www.douban.com/note/609636293/

一、硬件环境

1.CPU不是核数越高越好，性价比才是关键。

经常遇到很多的企业级客户，他们机器配置非常高，CPU有128 VCore，256G内存，但是只挂载了1块8T的SATA硬盘，千兆网卡。

这样的机器配置比较适合计算密集型的业务，但是如果是IO密集型的业务的话，就会发现磁盘成为瓶颈，会发现磁盘利用率100%，网络利用率100%，但是CPU只用了不到5%。存在巨大的资源浪费。

这种问题在Hadoop系统中尤为突出，如果是这样的配置的话，很可能一个MapReduce程序就会导致全部的磁盘与网络都是使用率100%，这样所有的心跳都发送不出来，而本身Hadoop又没有很好的网络限速机制，就会导致DataNode与TaskManager陆续的因为心跳超时而挂掉。

2.SAS、SATA与SSD 磁盘的选择与对比

IOPS (Input/Output Per Second)即每秒的输入输出量(或读写次数)，是衡量磁盘性能的主要指标之一。IOPS是指单位时间内系统能处理的I/O请求数量，I/O请求通常为读或写数据操作请求。对于随机读写频繁的应用，如OLTP(Online Transaction Processing)，IOPS是关键衡量指标。

吞吐量(Throughput)，指单位时间内可以成功传输的数据数量。对于大量顺序读写的应用，如VOD(Video On Demand)，则更关注吞吐量指标。

一般SSD的**IOPS**是普通磁盘的千倍以上**，但是**吞吐量只是普通磁盘的**2**倍左右。所以如果我们的业务是顺序读写偏多的则建议选用普通SAS盘（如存储形业务，以及Hive的文本数据分析），但是如果我们的业务是随机读写偏多，那么选择SSD 更划算（如采用列存储的系统，以及YDB的索引系统）

3.SSD的颗粒请不要选择TLC

TLC的寿命太短，虽然便宜，但是用不了几个月就基本报废，一般个人电脑使用。不适合企业级使用，性价比较好的建议选用MLC颗粒。

**SSD****颗粒目前主要分三种：****SLC****、****MLC****和****TLC**

SLC=Single-LevelCell，即1bit/cell，速度快寿命长，价格超贵（约MLC3倍以上的价格），约10万次擦写寿命 MLC=Multi-LevelCell，即2bit/cell，速度一般寿命一般，价格一般，约3000---10000次擦写寿命 TLC=Trinary-LevelCell，即3bit/cell，也有Fl[ash厂家](http://xiazai.zol.com.cn/heji/4390/)叫8LC，速度相对慢寿命相对短，价格便宜，约500次擦写寿命，目前还没有厂家能做到1000次擦写。

简单地说SLC的性能最优，价格超高。一般用作企业级或高端发烧友。**MLC****性能够用，价格适中为消费级**[**SSD**](http://detail.zol.com.cn/solid_state_drive/)**应用主流**，TLC综合性能最低，价格最便宜。但可以通过高性能主控、主控算法来弥补、提高TLC闪存的性能。

4.延云YDB建议的硬件配置

**一、延云****YDB****最低配置**

1.内存：32G

2.磁盘：

离线模式：至少2块独立的物理硬盘分别用于HDFS数据盘、系统盘。

实时模式：至少3块独立的物理磁盘分别用于Kafka数据盘,、HDFS数据盘、系统盘

3.CPU:至少8线程（1颗,4核,8线程）

**二、如下场景，延云将不再提供安装技术支持**

1.低于最低配置要求的用户。

2.32位系统的用户：这类系统最大只有4G内存。

**三、延云****YDB****高性能配置** **(****毫秒响应****)**

1.机器内存：128G

2.磁盘：企业级SSD，600~800G*12个磁盘

3.CPU：32线程（2颗,16核,32线程）

4.万兆网卡

**三、延云****YDB****常规配置 （秒级响应）**

1.机器内存：128G

2.磁盘：2T*12的磁盘

3.CPU：24线程（2颗,12核,24线程）

4.千兆网卡

二、磁盘如何挂载？

1.逻辑卷的问题

一般很多Linux的默认安装，会将磁盘直接以逻辑卷的方式挂载，逻辑卷的优点是后期的扩容以及调整磁盘非常的方便，看着比RAID好用多了，但是默认的逻辑卷配置方式是只有一块盘在工作 ，其他几块盘都闲着，发挥不出来多块盘的性能，也就是说如果在逻辑卷里面挂了10块盘，那么默认的逻辑卷的配置，只能发挥出一块盘的性能。**所以对于****YDB****系统来说，大家不要使用逻辑卷。**

2.关于RAID

有些客户比较担心数据丢失，将磁盘做了RAID10或者RAID5，其实这样是没有必要的，因为本身默认配置Hadoop是有三份副本的，并不怕磁盘损坏。RAID10与RAID5会导致磁盘容量只有原先的一半，由于需要双写，磁盘整体吞吐量降低了一倍。而且RAID5一旦损坏了一块磁盘，就需要通过奇偶校验还原数据，读的吞吐量直接降低到原先了五分之一，而且更换新盘后，通过校验要还原原先盘的数据的时候，经常会发生雪崩现象，IO瞬间增大，导致其他盘陆续的跟着挂掉。**所以对于****YDB****系统来说，不推荐使用****RAID 10****或****RAID5.**还有一些客户，会将所有的盘都做成一个完整的RAID0，RAID0的缺点就是一块盘损坏，整个系统就坏掉，但是RAID0确实会比单块磁盘速度好，所以如果能做raid0**我更推荐****2****个盘组成一起做一个****RAID0,****而不是整体所有磁盘都做成一个****RAID0.**

3.关于系统盘与数据盘

好多客户，在挂盘的时候，为了节省磁盘空间，更充分的利用资源，会将一个8T的物理磁盘划分成两个逻辑分区，一个逻辑分区作为系统盘，另一个逻辑分区作为数据盘。但是数据盘一般会比较繁忙的，但是由于他们底层都共用的是同一块物理磁盘，就会导致系统盘实际上也会特别繁忙，系统盘繁忙会导致整个系统会变的非常的慢，执行任何Linux命令都很慢，Socket连接建立也缓慢，很多系统会因此而超时断线，所以延云YDB建议**操作系统要独立一块磁盘**,数据盘不要与操作系统共用同一块盘，否则数据盘很慢的时候，运行在操作系统上的软件都跟着慢，ZooKeeper之类的服务也很容易挂掉。

另外还有一部分客户，可能因某种习惯，默认会给系统盘的跟目录预留的存储空间特别小，比如说只预留了10~30个G的空间，这样其实对大数据系统来说风险较大，以Ambari为例，他的log默认是记录在/var/log下的，这30G的空间会很快的被LOG记满，大家都知道一旦操作系统根目录满了意味着什么？ 将是所有服务不可用，这样隐患太大了。所以**延云建议系统跟目录尽量留大一点的磁盘空间，如****200G,****默认****CentOS****给分配****50G****空间也太小**，如果Hadoop等日志没有及时清理掉，将来隐患较大

4.关于磁盘阵列与云

有相当一部分的客户使用云服务器，将机器虚拟化后确实节省了很多的资源，提高了硬件的利用率。目前的云服务器有相当一部分的解决方案是采用外挂存储的方式将磁盘统一的挂载到远程的一个磁盘阵列上去。这个时候磁盘阵列是单点，一旦发生断电或者磁盘阵列出现问题，因为Hadoop的三分副本都存储在这一个磁盘阵列上，一但丢失就会导致整个Hadoop集群不可用。如果有条件，我更**建议做多个磁盘阵列而不是一个磁盘阵列单点**，这样通过Hadoop的机架策略，可以将 Hadoop 的三份副本分别存储在不同的磁盘阵列上，NameNode以及SNameNode也分别存储在不同的磁盘阵列上，这样即使其中一个磁盘阵列出现了故障，我们的Hadoop也能够恢复服务，而且不丢数据。

另外由于虚拟化以后，一个真实的物理机上面可能会开多个虚拟机，如果这个物理机硬件发生损坏，这个物理机上的虚拟机也有异常，三个副本都存储在这台机器上的文件的数据会丢失，延云建议虚拟机厂商与Hadoop厂商协同，采用Hadoop机架技术，将**位于同一物理机上的虚拟机标记在同一个机架上**，以免造成数据丢失。

虚拟化后也存在系统盘与数据盘的问题，虽然在虚拟机里看到了系统盘与数据盘确实分离了，但是在物理机上有可能是在虚拟机A里面的系统盘，又作为了虚拟机B的数据盘，这样当虚拟机B的数据盘特别繁忙的时候，会造成虚拟机A的响应非常慢。针对这种情况延云YDB建议，**将物理机的磁盘分类，一些磁盘专门用于挂系统盘，一些磁盘专门用于挂数据盘，不允许交叉使用，即不允许一个物理盘即挂数据盘又被挂成系统盘**

5.将大磁盘空间的硬盘与小磁盘空间的硬盘混合挂载

可能是处于历史原因，部分客户的系统上出现了大小盘混合挂载的情况，比如说10块磁盘，有的是300G，有的是8T的磁盘，他们混搭在一起。但是目前的hadoop对这样的盘支持的并不友好，会出现300G的硬盘已经满了，8T的硬盘还没使用到原先的十分之一的情况，针对这种情况，延云**建议数据盘尽量大小一样，别出现有的盘很大，有的盘很小的情况**。那种300G的磁盘还是留作操作系统盘为好。

三、操作系统如何选择

**1.****延云推荐使用****CentOS 6.6,6.5****的系统（请不要使用****CentOS7****）**

**2.****尽量选择安装英文语言环境，中文版****Ambari****有时会有问题，。**

**3.****安装桌面版，请别安装最简版。**

**4.****配置系统的****yum****源，如果部署****Ambari****会用到。**

开源世界确实好，选择很多，但是意味着也坑很多。

对于YDB来说，是不挑操作系统版本的，只要您的系统能安装上Hadoop,那么YDB一般都能运行起来。甚至有些同学还在MAC上调试YDB。但是如果您是要运行在生产系统上，操作系统的选择就尤为重要了。

CentOS7笔者在其中一个客户下踩了一个巨坑，一个内核的BUG导致系统不断重启，所以对比较新的内核版本还是比较畏惧，所以笔者不是特别推荐大家使用比较新的系统，建议大家选用经过较多生产系统上验证过的稳定版本。如果非要推荐一个版本，那么延云推荐使用Centos 6.6的系统，因为延云的日常开发与测试均在这个版本上进行。

CentOS7我们当时踩的坑叫Transparent Huge Pages (THP)的BUG，在负载高的时候会造成机器的反复重启，并且从HDP官方也证实了这个BUG，http://www.cloudera.com/documentation/enterprise/latest/topics/cdh_admin_performance.html,但是我们按照上面的方法进行设置后，机器不重启了，但是依然发生偶尔断网的情况。

四、操作系统设置

1.Ulimit配置

操作系统默认只能打开1024个文件，打开的文件超过这个数发现程序会有“too many open files”的错误，1024对于大数据系统来说显然是不够的，如果不设置，基本上整个大数据系统是“不可用的”，根本不能用于生产环境。

配置方法如下：

echo "* soft nofile 128000" >> /etc/security/limits.conf

echo "* hard nofile 128000" >> /etc/security/limits.conf

echo "* soft nproc 128000" >> /etc/security/limits.conf

echo "* hard nproc 128000" >> /etc/security/limits.conf

cat /[etc](http://cpro.baidu.com/cpro/ui/uijs.php?adclass=0&app_id=0&c=news&cf=1001&ch=0&di=128&fv=17&is_app=0&jk=1c8e62053d4667a6&k=etc&k0=etc&kdi0=0&luki=4&n=10&p=baidu&q=csai_cpr&rb=0&rs=1&seller_id=1&sid=a667463d5628e1c&ssp2=1&stid=0&t=tpclicked3_hc&td=1730417&tu=u1730417&u=http:%2Fwww.shangxueba.com%2Fjingyan%2F121578.html&urlid=0)/security/limits.conf

sed -i 's/1024/unlimited/' /etc/security/limits.d/90-nproc.conf

cat/etc/security/limits.d/90-nproc.conf

ulimit -SHn128000

ulimit -SHu 128000

2.Swap的问题

在10~20年前一台服务器的内存非常有限，64m~128m，所以通过swap可以将磁盘的一部分空间用于内存。但是现今我们的服务器内存普遍达到了64G以上，内存已经不再那么稀缺，但是内存的读取速度与磁盘的读取相差倍数太大，如果我们某段程序使用的内存映射到了磁盘上，将会对程序的性能照成非常严重的影响，甚至导致整个服务的瘫痪。对于YDB系统来说，要求**一定要禁止使用****Swap.**

禁用方法如下，让操作系统尽量不使用Swap：

echo "vm.swappiness=1" >> /etc/sysctl.conf

sysctl -p

sysctl -a|grep swappiness

3.网络配置优化

echo " net.core.somaxconn = 32768 " >> /etc/sysctl.conf

sysctl -p

sysctl -a|grepsomaxconn

4.SSH无密码登录

安装 Hadoop与Ambari均需要无密码登录

设置方法请参考如下命令

ssh-keygen

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

chmod 700 ~/.ssh

chmod 600 ~/.ssh/authorized_keys

ssh-copy-id root@ydbslave01

ssh-copy-id root@ydbslave02

…..

5.关闭防火墙

iptables -P INPUT ACCEPT

iptables -P FORWARD ACCEPT

iptables -P OUTPUT ACCEPT

chkconfig iptables off

/etc/init.d/iptables stop

service iptables stop

iptables -F

6.配置机器名,以及hosts域名解析

hostname ydbmaster

vi /etc/sysconfig/network

vi /etc/hosts

切记 hosts文件中 不要将localhost给注释掉，并且配置完毕后，执行下 hostname -f 看下 是否能识别出域名

7.setenforce与Umask配置

•setenforce

setenforce 0

sed -i 's/enabled=1/enabled=0/' /etc/yum/pluginconf.d/refresh-packagekit.conf

cat /etc/yum/pluginconf.d/refresh-packagekit.conf

•Umask

umask 0022

echo umask 0022 >> /etc/profile

8.检查/proc/sys/vm/overcommit_memory的配置值

如果为2，建议修改为0，否则有可能会出现，明明机器可用物理内存很多，但JVM确申请不了内存的情况。

9.语言环境配置

先修改机器的语言环境

\#vi /etc/sysconfig/i18n LANG="en_US.UTF-8" SUPPORTED="zh_CN.GB18030:zh_CN:zh:en_US.UTF-8:en_US:en" SYSFONT="latarcyrheb-sun16"

然后配置环境变量为utf8

echo "export LANG=en_US.UTF-8 " >> ~/.bashrc

source ~/.bashrc

export|grep LANG

10.配置时间同步

Hadoop，YDB等均要求机器时钟同步，否则机器时间相差较大，整个集群服务就会不正常，所以一定要配置。建议配置NTP服务。

**集群时间必须同步，不然会有严重问题**

参考资料如下：http://www.linuxidc.com/Linux/2009-02/18313.htm

11.JDK安装部署

YDB支持JDK1.7,JDK1.8，为了便于管理和使用，建议使用YDB随机提供的JDK1.8

建议统一安装到**/opt/ydbsoftware**路径下。

12.环境变量

**请大家千万不要在公共的环境变量配置****HIVE****、****SPARK****、****LUCENE****、****HADOOP****等环境变量，以免相互冲突。**

13.请检查盘符，不要含有中文

尤其是Ambari，有些时候，使用U盘或移动硬盘复制软件，如果这个移动硬盘挂载点是中文路径，这时在安装Ambari的时候会出现问题，一定要注意这个问题。

14.检查磁盘空间，使用率不得超过90%

默认Yarn会为每台机器保留10%的空间，如果剩余空间较少，Yarn就会停掉这些机器上的进程，并出现Container released on a *lost* node错误。

15.关键日志，定时清理，以免时间久了磁盘满了

如可以编辑crontab -e 每小时，清理一次日志，尤其是hadoop日志，特别占磁盘空间

0 */1 * * * find /var/log/hadoop/hdfs -type f -mmin +1440 |grep -E "\.log\." |xargs rm -rf

第五章非HDP版Hadoop基础服务配置要点

一、Hadoop服务-注意事项

1)**NameNode**:是HDFS的主节点，是Hadoop**最至关重要**的服务，一旦出问题，整个集群都不可用，NameNode的editlog与image目录，一定要配置多盘，设置冗余，如果有必要，配置RAID 10。

2)**SNameNode**：是Namenode的备份节点，一旦NameNode机器损坏，可以通过SNameNode恢复数据，**故****YDB****要求一定要启动****SNameNode****服务，并且****SNameNode****不可与****NameNode****位于同一个物理机上** 。

**3)****双****NameNode HA:**为了高可用，有些客户会启用HA，**延云不建议启用****HA****，如果必须启用一定要确保首节点为****Active****状态**，而不是Stand by状态，否则整个集群的NameNode响应会比较慢，从而影响整个集群的响应速度。

4)一定要确保dfs.datanode.data.dir与yarn.nodemanager.local-dirs的目录配置的是所有的数据盘，而不是配给了系统盘，特别多的用户在初次安装Hadoop的时候忘记配置这个，导致默认将数据都存储在了/tmp目录。另外系统盘一定要与数据盘分离，否则磁盘特别繁忙的时候会造成操作系统很繁忙，ZooKeeper之类的容易挂掉。

5)规划好Hadoop的logs目录，尽量分给一个大点磁盘存储空间的目录，否则经常会出现导入几十亿数据后，logs目录将系统/var/log给撑满，占用率100%

6)确保将来准备分配给YDB的HDFS目录有读写权限，建议第一次新手安装，取消HDFS的权限验证，配置dfs.permissions.enabled为false，并重启集群。

7)Hadoop的logs目录要配置上定期清理，以免时间久了，硬盘被撑爆。

8)确保HDFS安装成功，一定要手工通过hadoop -put命令，上传一个文件试一试。

9)打开8088，检查Yarn是否启动成功， VCores Total \Memory Total 分配的是否正确。经常有朋友忘记更改Yarn的默认配置导致一台128G内存的机器最多只能启动2个进程，只能使用8G内存。

10)yarn.nodemanager.resource.memory-mb用于配置Yarn当前机器的可用内存，通常配置当前机器剩余可用内存的80%.

11)yarn.scheduler.minimum-allocation-mb为一个Yarn container申请内存的最小计费单位，建议调小一些，如128，让计费更精准.

12)yarn.scheduler.maximum-allocation-mb为一个Yarn container可以申请最大的内存，建议调整为32768 （不一定真用到这些）

13)不建议启用CGROUPS，进行CPU隔离，对于即席系统来说，尽量充分利用资源。

14)yarn.nodemanager.resource.cpu-vcores 当前机器可以启动的Yarn container的数量，建议配置为当前机器的cpu的线程数的80%。

15)yarn.scheduler.maximum-allocation-vcores配置的稍微大一些，以便单个container能够多启动一些线程。

16)yarn.nodemanager.pmem-check-enabled与yarn.nodemanager.vmem-check-enabled一定要都配置成false，因为1.6版本的spark有BUG，会使用较多的堆外内存，Yarn会kill掉相关container，造成服务的不稳定。

17)检查mapreduce.application.classpath 里面的值是否有配置的jar包并不存在，典型的情况下是找不到lzo的包（许多厂商的安装部署会配置该参数）。如果有的jar包找不到，建议注释掉相关依赖，否则可能会造成YDB启动失败，如默认的HDP集群就要将其中的lzo的配置给注释掉/usr/hdp/${hdp.version}/hadoop/lib/hadoop-lzo-0.6.0.${hdp.version}.jar；如果是红象云腾Power版本在最后加上这个 /usr/crh/4.1.5/hadoop/share/hadoop/tools/lib/*

18)为了便于查找问题，我们一般保留7天的Hadoop日志，可以配置Yarn日志清理yarn.nodemanager.delete.debug-delay-sec 为 604800 （7天）

19)调整dfs.datanode.max.transfer.threads的值，默认4096太小，建议调整为10240

20)调整ipc.server.listen.queue.size为32768

21)调整yarn.resourcemanager.am.max-attempts的值为10000000，默认的2次太小，客户测试过程反复的kill就会导致整个任务失败。

二、Spark 需要使用延云提供的spark版本

1)无需配置，只需要解压开放到指定目录即可，我们一般解压到/opt/ydbsoftware/spark

2)**请大家不要启动****spark****服务，**YDB本身会自己调用Spark启动服务，无须我们额外为Spark启动服务。

三、ZooKeeper服务注意事项

第一：要探测ZooKeeper的2181端口是否启动 可以通过netstat –npl|grep 2181来查看

第二：ZooKeeper的数据目录别与HDFS的数据盘放在一起，尽量独立一个磁盘，或者放在系统盘，否则数据盘特别繁忙的时候ZooKeeper本身非常容易挂掉。如果机器富余，建议将ZK单独部署一个集群，不要混搭，如果因机制资源有限，必须混搭，请将zookeeper部署在通常来说负载不很很高的Master节点。

第三：ZooKeeper的日志清理要打开，否则会出现系统运行几个月后，ZooKeeper所在的磁盘硬盘变满的情况，将zoo.cfg里的这两个配置项注释开即可：

autopurge.purgeInterval=24

autopurge.snapRetainCount=30

第四：YDB使用的ZK的版本一定要与ZK的版本一致，如果不一致请更换ya100/lib下的zookeeper相关jar包。

四、Kafka注意事项

如果kafka配置的不好， 会发生比较严重的数据倾斜，而且在压力较大的情况会导致数据丢失。所以跟Kafka有关的如下配置，请一定要认真阅读

**注意****kafka server** **的****num.partitions****一定要大于总分片数的两倍，否则有的进程消费不到数据，导致数据倾斜。****YDB****的总分片数为****YA100_EXECUTORS\*****（****spark.executor.max.usedcores****）；**

注：spark.executor.max.usedcores默认（没有配置）为2个，表示每个进程会启动2个分片。

**l**数据丢失根本问题在于磁盘与网络是否繁忙！！！！！！

如果磁盘长时间使用率100%，必出现丢数据，会出现如下异常，配置的kafka retry机制无效

l如果我们先前采用的send方法没使用callback，一旦消息发送失败，我们没有处理异常的话，这个消息就丢了。

**这个问题如何解决？**

**1)****如果有条件，****Kafka****尽量独立集群，最低要求也一定要独立磁盘，并且写入限速**

独立磁盘是解决问题的根本，磁盘很繁忙的情况下，broker出错的概率很大。

**2)send** **里面的****callback****，如果是异常一定要自己做容错处理**

发现send函数里的callback，一定要对Exception exception不是null的情况做重试处理，一定要处理，根据判断重试几次。

**3)****调整****kafka****的参数**

**a)****建议在****Producter****端增加如下参数**

props.put("compression.type", "gzip"); props.put("linger.ms", "50"); props.put("acks", "all"); props.put("retries ", 30); props.put("reconnect.backoff.ms ", 20000); props.put("retry.backoff.ms", 20000);

**b)****在****Server****端的****broker****增加如下配置**

 

 

第六章非HDP版本的YDB部署

一、安装前的准备

请参考第三章基本环境注意事项，第四章的依赖的服务注意事项，准备基础环境，**这个很重要。**

二、YDB软件下载

从http://url.cn/42R4CG8获取延云软件

1)下载延云YDB

2)延云YDB提供的Spark

注意一定要使用延云提供的Spark，不能从其他地方下载

该Spark延云修正了一些BUG，以及在SQL解析上做了处理

3)JDK1.8

 

三、特殊版本的Spark的编译

如果我们的Hadoop版本比较特殊，大家可以从延云下载Spark源码执行进行编译。

编译示例如下：

修改源码包里面的ydb.combile.sh，将里面的hadoop改成我们对应的版本。

然后直接运行 sh ./ydb.compile.sh 即可，编译时间取决于我们的网络，首次编译时间估计会非常长，可以先下载延云提供的repository.tar.gz，以减少访问国外网络的下载时间。

四、软件解压

解压到/opt/ydbsoftware目录下，最后可以看到目录结构是这样的

conf目录是YDB的所有配置文件，bin目录是YDB的执行文件

五、配置conf目录下的ya100_env.sh环境变量

1.基本环境配置

export HADOOP_CONF_DIR=/etc/hadoop/conf

export HADOOP_HOME=/usr/hdp/current/hadoop-client

export JAVA_HOME=/usr/jdk64/jdk1.8.0_60

export SPARK_HOME=/root/software/spark-1.6.1

**注意：配置过后大家一定要手工验证下，相关目录的配置文件是否真的存在**

2.配置内存与启动的并发数

\#为启动的进程数量，切记不要超过Yarn总的VCores的数量-1

\#建议每台机器配置CPU线程数的一半，如12个；

\#如果有3台机器，每台机器配置12个的话那么下面这项的值要写36，不要只写12

**export YA100_EXECUTORS=12**

\#启动的进程，每个给分配多少内存

\#YA100_EXECUTORS*YA100_MEMORY的大小建议为yarn总内存的3/5（剩下的留给操作系统）

\#关于内存控制参数的详细说明，请阅读example下的《3.大家需要了解的几个内存控制的参数.txt》说明

\#常规128G内存的机器，建议配置为6000m~7000m

export YA100_MEMORY=6000m

\#每个进程内启动的线程数，一般不需要修改

\#配置值不可超过Yarn的yarn.scheduler.maximum-allocation-vcores的值

\#建议默认配置为5~9

export YA100_CORES=5

\#ydb 的JDBC接口程序分配的内存，建议6000m以上

export YA100_DRIVER_MEMORY=6000m

六、配置conf目录下的ydb_site.yaml环境变量

该文件的配置非常容易出错，要注意如下几点：

1.文件格式必须为UTF8格式，切记切记

2.每个配置项的开头必须有个空格，而不TAB

3.配置文件中别出现TAB

4.注意每个KEY : VALUE 之间是有一个空格的，如果value是字符串类型，要用双引号括起来

配置项说明如下：

1.配置 YDB的存储路径的配置 ydb.hdfs.path

注意YDB的存储路径与ya100的存储路径不是一个，要分别配置成不同的路径，不能重复

ya100的默认存储路径在conf目录下的hive-site.xml中的hive.metastore.warehouse.dir

Ya100的每张表的存储路径也可以再创建表的时候由location来指定。

2.配置Ydb在实时导入过程中，所使用的临时目录ydb.reader.rawdata.hdfs.path

3.配置ydb http ui服务的端口 ydb.httpserver.port 默认为1210

4.配置ydb依赖的zookeeper：storm.zookeeper.servers 与 storm.zookeeper.root

七、其他ya100/conf目录下的配置文件的说明

hive-site.xml hive表的配置，如果想要更改Hive的一些配置，如将Hive的元数据写入到数据库里，可修改此文件。

spark-defaults.conf 用于配置Spark，如果需要修改Spark的默认调度规则，可以修改此配置。

init.sql 为ya100启动时候的初始化方法，如果我们的业务需要自定义UDF，可以考虑将自定义UDF语句放到这里，通过init.sh来执行

driver.log.properties为接口程序的log4j的配置，默认日志记录在logs目录下

worker.log.properties为ya100的工作进程的log4j的配置，默认记录在每台机器的Yarn的工作目录下。如果不想Yarn清理掉，可以通过改文件改变日志的存储的路径，为了日常运维调试的方便，我们都建议修改，但一定要注意每台机器目录的权限。

八、开始部署延云YDB-服务的启动与检查

进入bin目录，执行chmod a+x *.sh

第一：ydb

./restart-all.sh 或 ./start-all.sh

第二：spark 服务检查：

1.tail -f ../logs/ya100.log 看是否有报错，当出现如下的日志，表示启动成功

2.打开yarn的8088页面，看启动的container数量以及内存的时候是否正确

3.看下面是否有ya100 on spark的任务，点击对应的Application Master看是否能打开Spark的UI页面

第三：YDB服务检查

1.通过浏览器打开:1210页面，看是否能打开

2.点开“work工作进程列表”看启动的worker数量是否与在ya100_env.sh里配置的YA100_EXECUTORS数量一致

第四：服务的停止

./stop-all.sh