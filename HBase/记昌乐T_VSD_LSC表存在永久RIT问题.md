### T_VSD_LSC表问题现象：

1. 存在region状态other
2. 存在一个Region In Transition运行几个月都未完成
3. 表上存在排它Lock，原因是存在procedure
4. disable该表，运行很长时间，最终报错：有一个正在运行的procedure



### 解决过程：

1. 删除hbase:meta内该表的元数据信息
2. 删除HDFS上该表的数据
3. 删除zookeeper中该表的所有ZNode信息
4. 删除HDFS上/apps/hbase/data/MasterProcWALs/*.log
5. 重启hbase master
6. 检查Hbase UI确认没有该表的RIT、Lock、Table Info
7. 重建hbase表