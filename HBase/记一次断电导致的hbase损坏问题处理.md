### 事故：

2019-06-13日中午，沧州公安局机房断电，来电后，重启HDP服务，启动hbase消费者(使用flume将kafka数据抽取到hbase中)，发现消费者flume日志报错如下：

```sh
[INFO] 2019-06-14 14:37:39 [org.apache.hadoop.hbase.client.AsyncRequestFutureImpl.waitUntilDone]  #12, waiting for 406  actions to finish on table: T_VSD_CSK
[INFO] 2019-06-14 14:37:39 [org.apache.hadoop.hbase.client.AsyncRequestFutureImpl.resubmit]  id=12, table=T_VSD_CSK, attempt=14/36, failureCount=12ops, last exception=org.apache.hadoop.hbase.NotServingRegionException: org.apache.hadoop.hbase.NotServingRegionException: T_VSD_CSK,\xE5\x86\x80Y6179\xE8\xAD\xA6\x002018-03-02T14:57:16Z\x0013090100000000000180-6,1521074503171.6d017ff294eeed8fe0d888623a193752. is not online on telchinacz02.com,16020,1560493343996
        at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:3273)
```

### 解决过程

#### 过程1

根据以往经验，尝试disable 'T_VSD_CSK'，但是这个命令执行很久不能完成，最终报错：

```sh
hbase(main):002:0> disable 'T_VSD_CSK'

ERROR: The procedure 9231 is still running
```

#### 过程2

使用HBase Master UI界面检查发现T_VSD_CSK表存在1个region为other状态，不是online状态，且发现有1个region一直在进行迁移任务(regions in transition)，于是尝试了以下方法进行修复未果。

**方法1**

使用hbase hbck命令检查发现如下报错：

```sh
ERROR: There is a hole in the region chain between 28492e79:\x00\x04X\xDE\xEEg and 28f5c280. You need to create a new .regioninfo and region dir in hdfs to plug the hole.
```

```sh
hbase hbck -fixMeta 
```

执行这个命令后，提示HBase2.0已经不支持这个命令了，如果要使用这个功能，需要从github拉取源码，生成jar，因为这个功能已经独立出来作为一个单独的项目。

**方法2**

```sh
1、停止hbase服务
2、hdfs dfs -rm -R /apps/hbase/data/data/default/T_VSD_CSK/*/recovered.edits
3、重启hbase
```

#### 过程3

检查regionServer日志，发现如下报错信息：

```sh
Caused by: org.apache.hadoop.hbase.io.hfile.CorruptHFileException: Problem reading HFile Trailer from file hdfs://telchinacz01.com:8020/apps/hbase/data/data/default/T_VSD_CSK/6d017ff294eeed8fe0d888623a193752/detail/3c2e87a9d67f4a369652b42f0617e8bd
	at org.apache.hadoop.hbase.io.hfile.HFile.openReader(HFile.java:545)
	at org.apache.hadoop.hbase.io.hfile.HFile.createReader(HFile.java:579)
	at org.apache.hadoop.hbase.regionserver.StoreFileReader.<init>(StoreFileReader.java:108)
	at org.apache.hadoop.hbase.regionserver.StoreFileInfo.open(StoreFileInfo.java:270)
	at org.apache.hadoop.hbase.regionserver.HStoreFile.open(HStoreFile.java:367)
	... 6 more
Caused by: org.apache.hadoop.hdfs.BlockMissingException: Could not obtain block: BP-1702746656-13.186.5.112-1513838651661:blk_1082763432_9022739 file=/apps/hbase/data/data/default/T_VSD_CSK/6d017ff294eeed8fe0d888623a193752/detail/3c2e87a9d67f4a369652b42f0617e8bd
	at org.apache.hadoop.hdfs.DFSInputStream.refetchLocations(DFSInputStream.java:870)
	at org.apache.hadoop.hdfs.DFSInputStream.chooseDataNode(DFSInputStream.java:853)
	at org.apache.hadoop.hdfs.DFSInputStream.chooseDataNode(DFSInputStream.java:832)
	at org.apache.hadoop.hdfs.DFSInputStream.blockSeekTo(DFSInputStream.java:564)
```

通过HDFS WEB UI界面发现存在corrupt blocks，执行以下命令也能确认存在corrupt blocks：

```sh
hdfs fsck /apps/hbase/data/data/default/T_VSD_CSK
```

使用以下命令删除corrupt blocks：

```sh
hdfs fsck /apps/hbase/data/data/default/T_VSD_CSK -delete
```

发现T_VSD_CSK表regions in transition task消失了，但是所有region都变为了other状态，并没有恢复为online，通过hbase shell中执行count命令也报错，但是使用disable后再enable，问题彻底解决，所有region恢复为online，执行count命令也正常，regionServer也不再报错。