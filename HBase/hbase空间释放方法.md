### 方法一：

```sh
truncate 'table_name'
```

执行后，空间并不会立即释放，会发现HDFS上的/apps/hbase/data/archive目录空间变大，只需等待几分钟，这个目录的空间即会释放。

注意：此方法只适用于删除全部数据

### 方法二：

```sh
alter 'table_name', NAME => 'column_family_name', TTL => 2592000  (单位是：秒)
```

这个命令会自动删除超过30天的数据

但是，执行以上命令后，空间不会释放，需要继续执行以下命令：

```sh
major_compact 'table_name'
```

执行major_compact命令后，也不会立即释放空间，与方法一同样，HDFS上的/apps/hbase/data/archive目录空间变大，只需等待几分钟，这个目录的空间即会释放。

关于major_compact命令的用法如下：

```sh
hbase(main):010:0> help 'major_compact'
          Run major compaction on passed table or pass a region row
          to major compact an individual region. To compact a single
          column family within a region specify the region name
          followed by the column family name.
          Examples:
          Compact all regions in a table:
          hbase> major_compact 't1'
          hbase> major_compact 'ns1:t1'
          Compact an entire region:
          hbase> major_compact 'r1'
          Compact a single column family within a region:
          hbase> major_compact 'r1', 'c1'
          Compact a single column family within a table:
          hbase> major_compact 't1', 'c1'
          Compact table with type "MOB"
          hbase> major_compact 't1', nil, 'MOB'
          Compact a column family using "MOB" type within a table
          hbase> major_compact 't1', 'c1', 'MOB'
```

