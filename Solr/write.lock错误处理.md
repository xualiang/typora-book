### /data/index/write.lock错误处理

**网上一个解决方法：**

很多情况自己不小心重启电脑而此时solr正在索引时，将可能启动报错LockObtainFailedException

index locked for write for core。到data/index下面看应该是有一个write.lock文件，删掉就行了。

可能报文件找不到的错，就到example/solr/collection1/conf下面拷贝过来就行了。

但按照这种方法，先停solr服务，再将write.log文件mv到/tmp目录下，重启solr，**问题没有解决**。

 

**以下才是问题根本原因：collection的数据目录权限变成了root**

Fixing solr error: “/var/solr/data/core_name/data/index/write.lock”

If you deploy a new core in Solr from a backup, you can get this cryptic error:

core_name: 
 org.apache.solr.common.SolrException:org.apache.solr.common.SolrException: 
 /var/solr/data/core_name/data/index/write.lock

This means Solr can’t write the file it uses to monitor locks. Most likely this is due to files having the wrong ownership.

1. Navigate to the path on the machine listed, and do this – you’ll see the correct user information on other files.

**ls** -al

2. Then you can run this (my solr user is “solr”)

**chown** -R solr:solr core_name

3.  And restart solr:

**/**etc**/**init.d**/**solr restart

