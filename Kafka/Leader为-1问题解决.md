```tex
stderr:   /var/lib/ambari-agent/data/errors-7716.txt
Traceback (most recent call last):
  File "/var/lib/ambari-agent/cache/common-services/KAFKA/0.8.1/package/scripts/service_check.py", line 81, in <module>
    ServiceCheck().execute()
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/script.py", line 352, in execute
    method(env)
  File "/var/lib/ambari-agent/cache/common-services/KAFKA/0.8.1/package/scripts/service_check.py", line 64, in service_check
    raise Fail("Under replicated partitions found: {0}".format(under_rep_cmd_out))
resource_management.core.exceptions.Fail: Under replicated partitions found: 	Topic: sjqxBeforeTopic	Partition: 0	Leader: -1	Replicas: 1	Isr:
stdout:   /var/lib/ambari-agent/data/output-7716.txt
2019-01-22 19:51:51,390 - Stack Feature Version Info: Cluster Stack=2.6, Command Stack=None, Command Version=2.6.5.0-292 -> 2.6.5.0-292
2019-01-22 19:51:51,482 - Using hadoop conf dir: /usr/hdp/2.6.5.0-292/hadoop/conf
2019-01-22 19:51:51,486 - call['source /usr/hdp/current/kafka-broker/config/kafka-env.sh ; /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper telchina193.com:2181,telchina191.com:2181,telchina192.com:2181 --topic ambari_kafka_service_check --list'] {'logoutput': True, 'quiet': False, 'user': 'kafka'}
ambari_kafka_service_check
2019-01-22 19:51:54,295 - call returned (0, 'ambari_kafka_service_check')
2019-01-22 19:51:54,296 - call['/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --describe --zookeeper telchina193.com:2181,telchina191.com:2181,telchina192.com:2181 --under-replicated-partitions'] {'logoutput': True, 'quiet': False, 'user': 'kafka'}
	Topic: sjqxBeforeTopic	Partition: 0	Leader: -1	Replicas: 1	Isr: 
2019-01-22 19:51:57,404 - call returned (0, '\tTopic: sjqxBeforeTopic\tPartition: 0\tLeader: -1\tReplicas: 1\tIsr: ')
Command failed after 1 tries


解决方法：
[zk: localhost:2181(CONNECTED) 4] get /brokers/topics/sjqxBeforeTopic/partitions/0/state  
{"controller_epoch":86,"leader":-1,"version":1,"leader_epoch":56,"isr":[]}
cZxid = 0x2c00086d9d
ctime = Mon Jul 09 10:27:44 CST 2018
mZxid = 0x4700000838
mtime = Mon Jan 21 16:12:12 CST 2019
pZxid = 0x2c00086d9d
cversion = 0
dataVersion = 56
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 74
numChildren = 0
[zk: localhost:2181(CONNECTED) 5] set /brokers/topics/sjqxBeforeTopic/partitions/0/state {"controller_epoch":86,"leader":1,"version":1,"leader_epoch":56,"isr":[1]} 
cZxid = 0x2c00086d9d
ctime = Mon Jul 09 10:27:44 CST 2018
mZxid = 0x4d00000013
mtime = Tue Jan 22 19:54:42 CST 2019
pZxid = 0x2c00086d9d
cversion = 0
dataVersion = 57
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 74
numChildren = 0

```

