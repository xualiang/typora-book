### 删除原配置文件

1. 进入zookeeper的安装目录：

cd /usr/hdp/2.3.0.0-2557/zookeeper/bin

2. 连接上zookeeper

  ./zkCli.sh -server zkhost:2181

3. 查看文件

ls /configs/vehiCompQuery_c

4. delete /configs/vehiCompQuery_c/schema.xml
5. quit 

### 上传新配置文件

1. 进入solr安装目录

Cd /opt/solr/solr/server/scripts/cloud-scripts

2、./zkcli.sh –zkhost node1:2181,node2:2181,node3:2181 –cmd putfile /configs/vehicompQuery_c/solrconfig.xml /software/solr_install/vehicle_solr_conf2/solrconfig.xml

说明；pufile后第一个参数是zookeeper上的路径，第二个参数是本地路径

3、重启solr所有节点