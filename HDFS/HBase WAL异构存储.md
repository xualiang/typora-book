HBase-12848（Utilize Flash storage for WAL）是1.1.0新推出的特性，它可以将WAL单独置于SSD上，配置方式是将如下的配置做相应修改：
hbase.wal.storage.policy
该配置的默认值是NONE，也就是wal文件和数据都存储在DISK上，不做区分，可以修改为ONE_SSD或者ALL_SDD，不同在于：
ONE_SSD：wal的一个副本置于SSD上，而其他副本仍然在默认存储；
ALL_SSD：wal文件的所有副本都存储于SSD盘上；
HBase实现异构wal存储很简单，底层依赖的就是hdfs的异构storage策略，不过是将wal文件所在的目录经反射调用dfs client的setStoragePolicy方法设置为用户指定的policy，主要代码如下：

```java
public static void setStoragePolicy(final FileSystem fs, final Configuration conf,
    final Path path, final String policyKey, final String defaultPolicy) {
  String storagePolicy = conf.get(policyKey, defaultPolicy).toUpperCase();
  if (!storagePolicy.equals(defaultPolicy) &&
      fs instanceof DistributedFileSystem) {
    DistributedFileSystem dfs = (DistributedFileSystem)fs;
    Class<? extends DistributedFileSystem> dfsClass = dfs.getClass();
    Method m = null;
    try {
      m = dfsClass.getDeclaredMethod("setStoragePolicy",
          new Class<?>[] { Path.class, String.class });
      m.setAccessible(true);
    } catch (NoSuchMethodException e) {
      LOG.info("FileSystem doesn't support"
          + " setStoragePolicy; --HDFS-7228 not available");
    } catch (SecurityException e) {
      LOG.info("Doesn't have access to setStoragePolicy on "
          + "FileSystems --HDFS-7228 not available", e);
      m = null; // could happen on setAccessible()
    }
    if (m != null) {
      try {
        m.invoke(dfs, path, storagePolicy);
        LOG.info("set " + storagePolicy + " for " + path);
      } catch (Exception e) {
        LOG.warn("Unable to set " + storagePolicy + " for " + path, e);
      }
    }
  }
```