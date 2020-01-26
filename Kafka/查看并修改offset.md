offset最大值，即已经消费到的offset：

sh kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 10.10.51.31:6667 --topic telchina_vsd_topic_num1 --time -1

 

offset最小值

sh kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 10.10.51.31:6667 --topic telchina_vsd_topic_num1 --time -2

 

 

**设置consumer group的offset**

启动zookeeper client

/zookeeper/bin/zkCli.sh

通过下面命令设置consumer group:testgroup topic:test partition:0的offset为1288:

set /consumers/testgroup/offsets/test/0 1288

注意如果你的kafka设置了zookeeper root，比如为/kafka，那么命令应该改为：

set /kafka/consumers/testgroup/offsets/test/0 1288

重启相关的应用程序，就可以从设置的offset开始读数据了。