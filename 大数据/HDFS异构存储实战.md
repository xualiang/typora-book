## HDFS异构存储实战

最近在做HBase跨机房的数据迁移，正好用到HDFS的异构存储，我们使用的场景是将WAL日志保存到SSD中，其他的数据则存储在普通的SATA盘中。既充分利用了本地SSD盘的空间，又达到了提升系统性能的目的。本文是对HDFS异构存储学习和使用的总结，以及对使用HDFS异构存储过程中遇到问题的总结，希望对广大技术网友有帮助。

### 一、异构存储是什么

所谓的异构存储就是将不同需求或者冷热的数据存储到不通的介质中去，实现既能兼顾性能又能兼顾成本。对于存储到HDFS的数据大致可以分下图的4个等级。

![img](../images/webp-20190228214420832)

从上图可以看出，大部分的数据都是冷数据或者极冷数据，对于这部分数据，读请求很少，写请求也非常少，对访问延迟不敏感。如果将这部分数据存储通过高压缩比，并且存储到普通的SATA大容量盘中去，能极大地节约成本。

对于热数据和实时数据，写请求比较高，读请求也很高，但是数据量很小。这个时候为了实现高并发低延迟，我们可以将这部分数据保存到SSD中。

Hadoop从2.6.0版本开始支持异构存储，HBase也从1.1.0开始支持将WAL的异构存储策略。

备注：这里面的难点是要对业务访问模式有足够的了解，提前确认好各个目录下的数据访问热度，以便规划好数据的存储策略。

### 二、HDFS异构存储类型和策略

#### 存储类型

HDFS异构存储支持如下4种类型，分别是：

**1、RAM_DISK**

**2、SSD**

**3、DISK**

**4、ARCHIVE**

这里前面3种都很好理解，单独解释一下ARCHIVE，这里ARCHIVE并不是指某种存储介质，而是一种高密度的存储方式，用于存储极冷数据。一般用得比较多的SSD和DISK两类。如果配置的时候没有指定存储类型的话，默认就是DISK存储。比如如下配置：

/data1/hbase/hdfs,/data2/hbase/hdfs,/data3/hbase/hdfs,/data4/hbase/hdfs,/data5/hbase/hdfs,/data6/hbase/hdfs,/data7/hbase/hdfs,/data8/hbase/hdfs,/data9/hbase/hdfs,/data10/hbase/hdfs,/data11/hbase/hdfs,/data12/hbase/hdfs,[SSD]/wal_data

这里前面12个盘都没有指定存储类型，则默认是DISK存储，而第13快盘指定了SSD存储类型。

4中存储类型，按照RAM_DISK->SSD->DISK->ARCHIVE，速度由快到慢，单位存储成本由高到低。

#### 存储策略

HDFS存储策略设置如下表：

![img](../images/webp-20190228214421002)

由上图，我们可以看出HDFS总共支持Lazy_Persist、All_SSD、One_SSD、Hot、Warm和Cold等6种存储策略，默认策略为Hot。

上图中的第三列是表示存储策略对应的存储类型，具体如下：

Lazy_Persist ： 1份数据存储在[RAM_DISK]即内存中，其他副本存储在DISK中

All_SSD：全部数据都存储在SSD中

One_SSD：一份数据存储在SSD中，其他副本存储在DISK中

Hot：全部数据存储在DISK中

Warm：一份数据存储在DISK中，其他数据存储方式为ARCHIVE

Cold：全部数据以ARCHIVE的方式保存

上图中的第4、5列表示创建和写副本的时候，如果该存储策略对应的资源不足，比如磁盘不可用或者空间写满，则创建文件和同步副本的时候选择第4和第5列对应的存储类型，你可以理解为降级机制，这里不再赘述。

### 三、HDFS异构存储原理

​    对于HDFS异构存储的原理大致概括如下图所示：

![img](../images/webp-20190228214420885)

这里的原理简单概括如下：

1、在hdfs的配置文件hdfs-site.xml中配置对应的异构存储（后面配置部分有详细介绍）

2、DataNode启动的时候从配置文件中读取对应的存储类型，以及容量情况，并通过心跳的形式不断的上报给ＮameNode。

3、NameNode收到DataNode发送的关于存储类型、容量等内容的心跳包后，会进行处理，更新存储的相关内容。

4、写请求发到NameNode后，NameNode根据写请求具体的目录对应的存储策略选择对应的存储类型的DataNode进行写入操作。

备注：上面是根据自己的理解简单概括的大致调用过程，如果需要了解更详细的调用关系，可以阅读这篇文章，写得很详细：<https://blog.csdn.net/androidlushangderen/article/details/51105876>

### 四、HDFS异构存储的配置和策略设置

#### HDFS异构存的配置

HDFS异构存的配置比较简单，只需要将对应的类型添加到dfs.datanode.data.dir的配置项中即可，

备注：也需要配置dfs.storage.policy.enabled为true，因为默认就是true，所以这里忽略。

配置的时候需要申明存储类型和对应的目录，存储类型需要用中括号括起来，存储类型有[SSD]/[DISK]/[ARCHIVE]/[RAM_DISK]，如果不指定存储类型，则默认就是DISK。

比如我的机器中只配置了DISK和SSD的类型，范例如下：

![img](../images/webp-20190228214420874)

通过上面的例子，前面12个盘，我没有设置存储类型，因为都是DISK，最后一个盘使用了SSD类型。

#### HDFS异构存储策略设置

HDFS提供了专门的命令来设置对应的策略，命令使用方法如下：

查看策略的帮助信息

> hdfs storagepolicies -help

列出当前版本支持的存储策略：

> hdfs storagepolicies -listPolicies

设置对应路径的策略

> hdfs storagepolicies -setStoragePolicy -path -policy

范例：

设置/hbase/data/default为Hot的策略

> hdfs storagepolicies -setStoragePolicy -path /hbase/data/default -policy Hot

取消策略

> hdfs storagepolicies -unsetStoragePolicy -path

获取对应路径的策略

> hdfs storagepolicies -getStoragePolicy -path

### 五、HDFS异构存储的管理

对于HDFS异构存储的管理，主要包含如下两个方面：

1、统计线上数据的访问频率，确认冷热数据所在目录，灰度进行调整

2、使用hdfs storagepolicies相关命令进行策略的调整

3、修改存储策略以后，使用mover工具进行数据的迁移，mover的使用方法如下：

> hdfs mover [-p files/dirs | -f localfile ]

可以使用-p指定要迁移的目录，也可以将要迁移的文件列表写入文件中，用-f参数指定对应的文件或者目录进行迁移。

### 六、HDFS异构存储遇到的问题

**1、设置dfs.datanode.du.reserved参数的时候要注意盘的大小**

​    我们在生产环境使用的时候，由于SSD盘专门用来存储HBase的WAL，因此SSD只有100多G，而我们设置dfs.datanode.du.reserved参数的时候设置为了200G，导致，SSD盘没有写入任何数据。原因是因为dfs.datanode.du.reserved参数是全局参数。目前官方的版本貌似没有单独对某一个磁盘做单独的配置，腾讯使用的版本有专门修改过，支持对单个盘的设置。因此，如果SSD盘很小，则需要将dfs.datanode.du.reserved参数相应的调小。

**2、使用mover迁移数据的时候，发现mover不生效**

​    因为我们之前版本有bug，导致部分非WAL日志的数据也写入到了SSD，导致SSD空间不足。因此发起数据迁移，但是迁移的时候发现数据并不会对数据做迁移，原因是没有对要迁移的目录显式地指定存储策略，因此迁移之前必须提前使用hdfs storagepolicies设置好存储策略。

**3、使用mover迁移数据的时候，会导致datanode出现dead的情况，从而影响写入**

目前mover迁移数据的时候会导致datanode出现dead的问题是必现的问题，临时采用重启datanode的方式规避，目前原因还在进一步分析中。异常报错如下：

![img](../images/webp-20190228214421092)

![img](../images/webp-20190228214421020)

### 七、参考资料

<http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/ArchivalStorage.html>

<https://blog.csdn.net/androidlushangderen/article/details/51105876>

- 
- 