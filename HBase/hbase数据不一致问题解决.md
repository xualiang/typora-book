## HBase集群数据不一致故障解决

## 1. 故障描述

最近我们的测试HBase集群经常因为机房网络维护而崩溃（服务宕机），在集群崩溃以后重启HBase集群发现有大部分region没有online的情况，导致查询表数据的时候会出现如下异常：

```sh
org.apache.hadoop.hbase.NotServingRegionException:...
```

我们HBase集群环境大概如下所示：

192.168.27.230 HMaster HRegionServer 
192.168.27.231 HRegionServer 
192.168.27.232 HRegionServer

即3台机器 1个HMaster 3个RegionSever的形式

我们还使用了Phoenix来创建HBase表，并创建了二级索引表，加上Phoenix自带的系统表，整个HBase集群一共有8个表。

查看regionserver上面的log，发现大量如下异常信息：

```sh
2016-05-26 14:57:46,928 INFO org.apache.hadoop.hbase.client.AsyncProcess: #5, table=C_PICRECORD_IDX_COLLISION, attempt=21/350 failed=2ops, last exception: org.apache.hadoop.hbase.exceptions.RegionOpeningException: org.apache.hadoop.hbase.exceptions.RegionOpeningException: Region C_PICRECORD_IDX_COLLISION,\x05\x00\x00\x00\x00\x00,1463624374612.2fc54757a3fea6f3f32526382947e8e6. is opening on cbds2,60020,1464245619847
    at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:2788)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegion(RSRpcServices.java:887)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.multi(RSRpcServices.java:1858)
    at org.apache.hadoop.hbase.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:31451)
    at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:2035)
    at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:107)
    at org.apache.hadoop.hbase.ipc.RpcExecutor.consumerLoop(RpcExecutor.java:130)
    at org.apache.hadoop.hbase.ipc.RpcExecutor$1.run(RpcExecutor.java:107)
    at java.lang.Thread.run(Thread.java:745)
 on cbds2,60020,1464230060200, tracking started null, retrying after=20179ms, replay=2ops
2016-05-26 14:57:47,110 INFO org.apache.hadoop.hbase.client.AsyncProcess: #4, table=C_PICRECORD_IDX, attempt=21/350 failed=2ops, last exception: org.apache.hadoop.hbase.NotServingRegionException: org.apache.hadoop.hbase.NotServingRegionException: Region C_PICRECORD_IDX,\x06\x00\x00\x00\x00,1463621907697.3ae160acd7f317598908c7a778f7e8b8. is not online on cbds2,60020,1464245619847
    at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:2791)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegion(RSRpcServices.java:887)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.multi(RSRpcServices.java:1858)
    at org.apache.hadoop.hbase.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:31451)
    at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:2035)
    at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:107)
    at org.apache.hadoop.hbase.ipc.RpcExecutor.consumerLoop(RpcExecutor.java:130)
    at org.apache.hadoop.hbase.ipc.RpcExecutor$1.run(RpcExecutor.java:107)
    at java.lang.Thread.run(Thread.java:745)
 on cbds2,60020,1464230060200, tracking started null, retrying after=20131ms, replay=2ops
2016-05-26 14:57:47,233 INFO org.apache.hadoop.hbase.client.AsyncProcess: #4, table=C_PICRECORD_IDX, attempt=21/350 failed=2ops, last exception: org.apache.hadoop.hbase.exceptions.RegionOpeningException: org.apache.hadoop.hbase.exceptions.RegionOpeningException: Region C_PICRECORD_IDX,\x07\x00\x00\x00\x00,1463621907697.2ef56bd25a9b77a920a314b9b0308c01. is opening on cbds1,60020,1464245619628
    at org.apache.hadoop.hbase.regionserver.HRegionServer.getRegionByEncodedName(HRegionServer.java:2788)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.getRegion(RSRpcServices.java:887)
    at org.apache.hadoop.hbase.regionserver.RSRpcServices.multi(RSRpcServices.java:1858)
    at org.apache.hadoop.hbase.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:31451)
    at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:2035)
    at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:107)
    at org.apache.hadoop.hbase.ipc.RpcExecutor.consumerLoop(RpcExecutor.java:130)
    at org.apache.hadoop.hbase.ipc.RpcExecutor$1.run(RpcExecutor.java:107)
    at java.lang.Thread.run(Thread.java:745)
 on cbds1,60020,1464230060318, tracking started null, retrying after=20153ms, replay=2ops
```

## 2. 尝试解决

根据网上的做法使用`hbase hbck`命令检查是否存在数据不一致的情况导致region没能online。

```sh
$ sudo -u hbase hbase hbck
```

从输出信息看到状态确实为不一致(INCONSISTENT)，并且在输出日志里面还有说明出现了region空洞(region hole)

```sh
ERROR: There is a hole in the region chain between \x05\x00\x00\x00\x00\x00 and \x06\x00\x00\x00\x00\x00.  You need to create a new .regioninfo and region dir in hdfs to plug the hole.
ERROR: There is a hole in the region chain between \x07\x00\x00\x00\x00\x00 and \x09\x00\x00\x00\x00\x00.  You need to create a new .regioninfo and region dir in hdfs to plug the hole.
ERROR: There is a hole in the region chain between \x0B\x00\x00\x00\x00\x00 and \x0C\x00\x00\x00\x00\x00.  You need to create a new .regioninfo and region dir in hdfs to plug the hole.

....

Summary:
  hbase:meta is okay.
    Number of regions: 1
    Deployed on:  cbds2,60020,1463557954277
Table C_PICRECORD_IDX_COLLISION is inconsistent.
    Number of regions: 11
    Deployed on:  cbds0,60020,1463557954330 cbds1,60020,1463557953442 cbds2,60020,1463557954277
  SYSTEM.CATALOG is okay.
    Number of regions: 1
    Deployed on:  cbds2,60020,1463557954277
  C_PICRECORD is okay.
    Number of regions: 0
    Deployed on: 
  hbase:namespace is okay.
    Number of regions: 1
    Deployed on:  cbds0,60020,1463557954330
  SYSTEM.SEQUENCE is okay.
    Number of regions: 159
    Deployed on:  cbds0,60020,1463557954330 cbds1,60020,1463557953442 cbds2,60020,1463557954277
  SYSTEM.FUNCTION is okay.
    Number of regions: 1
    Deployed on:  cbds2,60020,1463557954277
Table C_PICRECORD_IDX is inconsistent.
    Number of regions: 11
    Deployed on:  cbds0,60020,1463557954330 cbds1,60020,1463557953442 cbds2,60020,1463557954277
  SYSTEM.STATS is okay.
    Number of regions: 1
    Deployed on:  cbds1,60020,1463557953442
199 inconsistencies detected.
Status: INCONSISTENT
```

### 2.1【尝试一】

尝试使用`hbase hbck -fix`等命令来进行修复，结果失败。

### 2.2【尝试二】

通过清空zookeeper上/hbase信息并重启hbase集群以后，没能解决问题，并且hbase master隔了一段时间就挂掉

报错信息如下所示：

```sh
Failed to become active master
java.io.IOException: Timedout 300000ms waiting for namespace table to be assigned
    at org.apache.hadoop.hbase.master.TableNamespaceManager.start(TableNamespaceManager.java:98)
    at org.apache.hadoop.hbase.master.HMaster.initNamespace(HMaster.java:897)
    at org.apache.hadoop.hbase.master.HMaster.finishActiveMasterInitialization(HMaster.java:739)
    at org.apache.hadoop.hbase.master.HMaster.access$500(HMaster.java:169)
    at org.apache.hadoop.hbase.master.HMaster$1.run(HMaster.java:1479)
    at java.lang.Thread.run(Thread.java:745)

....

Unhandled exception. Starting shutdown.
java.io.IOException: Timedout 300000ms waiting for namespace table to be assigned
    at org.apache.hadoop.hbase.master.TableNamespaceManager.start(TableNamespaceManager.java:98)
    at org.apache.hadoop.hbase.master.HMaster.initNamespace(HMaster.java:897)
    at org.apache.hadoop.hbase.master.HMaster.finishActiveMasterInitialization(HMaster.java:739)
    at org.apache.hadoop.hbase.master.HMaster.access$500(HMaster.java:169)
    at org.apache.hadoop.hbase.master.HMaster$1.run(HMaster.java:1479)
    at java.lang.Thread.run(Thread.java:745)
```

## 3. 解决方案

### 3.1 【解决方案一】

最终在google上找到以下资料，跟我们出现的情况类似：

[hbase-error-there-is-a-hole-in-the-region-chain](https://viewsby.wordpress.com/2015/06/01/hbase-error-there-is-a-hole-in-the-region-chain "")

[hbase-hbck-cant-fix-region-inconsistencies](https://serverfault.com/questions/510290/hbase-hbck-cant-fix-region-inconsistencies)

照着链接上面的思路进行如下步骤：

1 ) 停掉hbase集群 
2）删除hbase在hdfs目录下所有表目录下的recovered.edits

eg :

```sh
hadoop fs -rm -R /hbase/data/default/SYSTEM.CATALOG/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/SYSTEM.FUNCTION/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/SYSTEM.SEQUENCE/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/SYSTEM.STATS/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/SYSTEM.CATALOG/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/C_PICRECORD_IDX_COLLISION/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/C_PICRECORD_IDX/*/recovered.edits
hadoop fs -rm -R /hbase/data/default/C_PICRECORD/*/recovered.edits
```

3）重启hbase集群，所有的region就都online了

**注意，这种通过删除recovered.edits的方式来恢复集群，会丢失部分数据。**

### 3.2【解决方案二】

在Google上面有人遇到了在使用phoenix本地索引的时候重启HBase集群后出现了跟我们类似的情况：

[Can-phoenix-local-indexes-create-a-deadlock-after-an-HBase-full-restart](http://apache-phoenix-user-list.1124778.n5.nabble.com/Can-phoenix-local-indexes-create-a-deadlock-after-an-HBase-full-restart-td824.html)

[phoenix-local-indexes.html](https://community.hortonworks.com/questions/8757/phoenix-local-indexes.html)

根据上面的思路，我们需要在集群所有RegionServer的`hbase-site.xml`配置文件里面增加如下配置：

```xml
<property>   <name>hbase.regionserver.executor.openregion.threads</name> <value>100</value> 
</property>
```

然后重启HBase集群就可以了。

这种解决方案应该不会丢数据，推荐使用这种方式来恢复。