Posted on Jul 26, 2017

[社区原文](https://community.hortonworks.com/articles/97489/completely-uninstall-hdp-and-ambari.html)

简介：

在不需要重装操作系统的情况下完全卸载HDP，并准备好自动安装HDP2.6的环境。

文章：

升级HDP失败后，我被迫彻底清除HDP 2.4，Ambari 2.5并安装HDP 2.6。 我想避免重新安装操作系统，所以执行了如下的步骤。

#### 1、停止在Ambari中的所有服务或杀死他们

可以通过ambari控制台停止所有服务。在我的情况下，Ambari在降级时损坏了他的数据库，无法启动。 所以我手动杀死所有节点上的所有进程：

$ ps -u hdfs (可以看到所有服务列表)
 $ kill PID

####2、在所有集群节点上运行python脚本

$ python /usr/lib/python2.6/site-packages/ambari_agent/HostCleanup.py --silent --skip=users

#### 3、删除所有节点上的Hadoop包

yum remove hive\*
 yum remove oozie\*
 yum remove pig\*
 yum remove zookeeper\*
 yum remove tez\*
 yum remove hbase\*
 yum remove ranger\*
 yum remove knox\*
 yum remove storm\*
 yum remove accumulo\*
 yum remove falcon\*
 yum remove ambari-metrics-hadoop-sink 
 yum remove smartsense-hst
 yum remove slider_2_4_2_0_258
 yum remove ambari-metrics-monitor
 yum remove spark2_2_5_3_0_37-yarn-shuffle
 yum remove spark_2_5_3_0_37-yarn-shuffle
 yum remove ambari-infra-solr-client

#### 4、删除ambari-server（在Ambari主机上）和ambari-agent（在所有节点上）

ambari-server stop
 ambari-agent stop
 yum erase ambari-server
 yum erase ambari-agent

#### 5、删除所有节点上的存储库

rm -rf /etc/yum.repos.d/ambari.repo /etc/yum.repos.d/HDP*
 yum clean all

#### 6、删除所有节点上的日志文件夹

rm -rf /var/log/ambari-agent
 rm -rf /var/log/ambari-metrics-grafana
 rm -rf /var/log/ambari-metrics-monitor
 rm -rf /var/log/ambari-server/
 rm -rf /var/log/falcon
 rm -rf /var/log/flume
 rm -rf /var/log/hadoop
 rm -rf /var/log/hadoop-mapreduce
 rm -rf /var/log/hadoop-yarn
 rm -rf /var/log/hive
 rm -rf /var/log/hive-hcatalog
 rm -rf /var/log/hive2
 rm -rf /var/log/hst
 rm -rf /var/log/knox
 rm -rf /var/log/oozie
 rm -rf /var/log/solr
 rm -rf /var/log/zookeeper

#### 7、删除所有节点上的Hadoop文件夹，包括HDFS数据

rm -rf /hadoop/*
 rm -rf /hdfs/hadoop
 rm -rf /hdfs/lost+found
 rm -rf /hdfs/var
 rm -rf /local/opt/hadoop
 rm -rf /tmp/hadoop
 rm -rf /usr/bin/hadoop
 rm -rf /usr/hdp
 rm -rf /var/hadoop

#### 8、删除所有节点上的配置文件夹

rm -rf /etc/ambari-agent
 rm -rf /etc/ambari-metrics-grafana
 rm -rf /etc/ambari-server
 rm -rf /etc/ams-hbase
 rm -rf /etc/falcon
 rm -rf /etc/flume
 rm -rf /etc/hadoop
 rm -rf /etc/hadoop-httpfs
 rm -rf /etc/hbase
 rm -rf /etc/hive 
 rm -rf /etc/hive-hcatalog
 rm -rf /etc/hive-webhcat
 rm -rf /etc/hive2
 rm -rf /etc/hst
 rm -rf /etc/knox 
 rm -rf /etc/livy
 rm -rf /etc/mahout 
 rm -rf /etc/oozie
 rm -rf /etc/phoenix
 rm -rf /etc/pig 
 rm -rf /etc/ranger-admin
 rm -rf /etc/ranger-usersync
 rm -rf /etc/spark2
 rm -rf /etc/tez
 rm -rf /etc/tez_hive2
 rm -rf /etc/zookeeper

####9、删除所有节点上的PID

rm -rf /var/run/ambari-agent
 rm -rf /var/run/ambari-metrics-grafana
 rm -rf /var/run/ambari-server
 rm -rf /var/run/falcon
 rm -rf /var/run/flume
 rm -rf /var/run/hadoop 
 rm -rf /var/run/hadoop-mapreduce
 rm -rf /var/run/hadoop-yarn
 rm -rf /var/run/hbase
 rm -rf /var/run/hive
 rm -rf /var/run/hive-hcatalog
 rm -rf /var/run/hive2
 rm -rf /var/run/hst
 rm -rf /var/run/knox
 rm -rf /var/run/oozie 
 rm -rf /var/run/webhcat
 rm -rf /var/run/zookeeper

#### 10、删除所有节点上的库文件夹

rm -rf /usr/lib/ambari-agent
 rm -rf /usr/lib/ambari-infra-solr-client
 rm -rf /usr/lib/ambari-metrics-hadoop-sink
 rm -rf /usr/lib/ambari-metrics-kafka-sink
 rm -rf /usr/lib/ambari-server-backups
 rm -rf /usr/lib/ams-hbase
 rm -rf /usr/lib/mysql
 rm -rf /var/lib/ambari-agent
 rm -rf /var/lib/ambari-metrics-grafana
 rm -rf /var/lib/ambari-server
 rm -rf /var/lib/flume
 rm -rf /var/lib/hadoop-hdfs
 rm -rf /var/lib/hadoop-mapreduce
 rm -rf /var/lib/hadoop-yarn 
 rm -rf /var/lib/hive2
 rm -rf /var/lib/knox
 rm -rf /var/lib/smartsense
 rm -rf /var/lib/storm

####11、清除所有节点上的文件夹/var/tmp/*

rm -rf /var/tmp/*

#### 12、从cron在所有节点上删除HST

/usr/hdp/share/hst/bin/hst-scheduled-capture.sh sync
 /usr/hdp/share/hst/bin/hst-scheduled-capture.sh

####13、删除数据库。 删除MySQL和Postgres的实例，以便Ambari安装和配置新的数据库。

yum remove mysql mysql-server
 yum erase postgresql
 rm -rf /var/lib/pgsql
 rm -rf /var/lib/mysql

#### 14、删除所有节点上的符号链接。

尤其是检查文件夹/usr/sbin和/usr/lib/python2.6/site-packages

cd /usr/bin
 rm -rf accumulo
 rm -rf atlas-start
 rm -rf atlas-stop
 rm -rf beeline
 rm -rf falcon
 rm -rf flume-ng
 rm -rf hbase
 rm -rf hcat
 rm -rf hdfs
 rm -rf hive
 rm -rf hiveserver2
 rm -rf kafka
 rm -rf mahout
 rm -rf mapred
 rm -rf oozie
 rm -rf oozied.sh
 rm -rf phoenix-psql
 rm -rf phoenix-queryserver
 rm -rf phoenix-sqlline
 rm -rf phoenix-sqlline-thin
 rm -rf pig
 rm -rf python-wrap
 rm -rf ranger-admin
 rm -rf ranger-admin-start
 rm -rf ranger-admin-stop
 rm -rf ranger-kms
 rm -rf ranger-usersync
 rm -rf ranger-usersync-start
 rm -rf ranger-usersync-stop
 rm -rf slider
 rm -rf sqoop
 rm -rf sqoop-codegen
 rm -rf sqoop-create-hive-table
 rm -rf sqoop-eval
 rm -rf sqoop-export
 rm -rf sqoop-help
 rm -rf sqoop-import
 rm -rf sqoop-import-all-tables
 rm -rf sqoop-job
 rm -rf sqoop-list-databases
 rm -rf sqoop-list-tables
 rm -rf sqoop-merge
 rm -rf sqoop-metastore
 rm -rf sqoop-version
 rm -rf storm
 rm -rf storm-slider
 rm -rf worker-lanucher
 rm -rf yarn
 rm -rf zookeeper-client
 rm -rf zookeeper-server
 rm -rf zookeeper-server-cleanup

#### 15、删除所有节点上的服务用户

userdel -r accumulo
 userdel -r ambari-qa
 userdel -r ams
 userdel -r falcon
 userdel -r flume
 userdel -r hbase
 userdel -r hcat
 userdel -r hdfs
 userdel -r hive
 userdel -r kafka
 userdel -r knox
 userdel -r mapred
 userdel -r oozie
 userdel -r ranger
 userdel -r spark
 userdel -r sqoop
 userdel -r storm
 userdel -r tez
 userdel -r yarn
 userdel -r zeppelin
 userdel -r zookeeper

#### 16、在所有节点上运行find / -name **

你一定会找到更多的文件/文件夹。 删除它们

find / -name *ambari*
 find / -name *accumulo*
 find / -name *atlas*
 find / -name *beeline*
 find / -name *falcon*
 find / -name *flume*
 find / -name *hadoop*
 find / -name *hbase*
 find / -name *hcat*
 find / -name *hdfs*
 find / -name *hdp*
 find / -name *hive*
 find / -name *hiveserver2*
 find / -name *kafka*
 find / -name *mahout*
 find / -name *mapred*
 find / -name *oozie*
 find / -name *phoenix*
 find / -name *pig*
 find / -name *ranger*
 find / -name *slider*
 find / -name *sqoop*
 find / -name *storm*
 find / -name *yarn*
 find / -name *zookeeper*

#### 17、重新启动所有节点

reboot