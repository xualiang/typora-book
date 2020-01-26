### index升级命令：

cd /opt/solr7/solr/server/solr-webapp/webapp/WEB-INF/lib/

java -cp lucene-core-7.5.0.jar:lucene-backward-codecs-7.5.0.jar org.apache.lucene.index.IndexUpgrader -delete-prior-commits /software/solr_data/data/vehiCompQuery_c_shard1_replica2/data/index/

### IndexUpgrader工具介绍

Lucene发行版包括[一个工具](https://lucene.apache.org/core/7_5_0/core/org/apache/lucene/index/IndexUpgrader.html)，可将索引从以前的Lucene版本升级到当前文件格式。

该工具可以在命令行中使用，也可以在Java中实例化和执行。

> 索引**只能**从以前的主要版本升级到当前主要版本。  这意味着，例如，任何Solr  7.x版本中的IndexUpgrader Tool只能使用6.x版本的索引，但不能使用Solr 5.x或更早版本的索引。  如果您当前正在使用早期版本（如5.x）并希望提前移动多个主要版本，则需要先将索引升级到下一个主要版本（6.x），然后再将其升级到主要版本之后（7.x）等   

在Solr发行版中，Lucene文件位于./server/solr-webapp/webapp/WEB-INF/lib。运行该工具时，您需要在类路径中包含lucene-core-<version>.jar和lucene-backwards-codecs-<version>.jar。

java -cp lucene-core-7.5.0.jar:lucene-backward-codecs-7.5.0.jar org.apache.lucene.index.IndexUpgrader [-delete-prior-commits] [-verbose] /path/to/index

此工具仅保留索引中的最后一次提交。因此，如果传入索引具有多个提交，则默认情况下该工具拒绝运行。指定-delete-prior-commits覆盖它，允许工具删除除最后一次提交之外的所有提交。

升级大型索引可能需要很长时间。根据经验，升级过程大约每分钟1 GB。

> 如果在执行之前索引已部分升级（例如，添加了文档），则此工具可能会重新排序文档。如果您的应用程序依赖于文档ID的单调性（即保留文档添加到索引的顺序），请执行完全优化。  

