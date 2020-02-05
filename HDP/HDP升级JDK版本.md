### 升级JDK过程

1. 在所有节点安装新版本jdk，并设置环境变量，且生效
2. Ambari-server     setup，设置JAVA_HOME
3. 重启Ambari-server和所有节点agent
4. 重启所有hdp服务

 如果出现问题，需要考虑是否是jce的问题，如果是，进行jce的替换。

- 解决方法：安装JCE

- 1. 在Ambari      Server上，获取适用于群集中JDK版本的JCE策略文件。

- - 对于Oracle JDK      1.8：
                http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html
  - 对于Oracle JDK      1.7：
                http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html

- 1. 将策略文件存档保存在临时位置。

  2. 在Ambari      Server上以及群集中的每台主机上，添加无限制的安全策略JCE jar $JAVA_HOME/jre/lib/security/.
                 例如，运行以下命令将策略jar解压缩到主机上安装的JDK中：
                 unzip -o -j -q jce_policy-8.zip -d      /usr/jdk64/jdk1.8.0_40/jre/lib/security/

  3. 重新启动Ambari Server。

     

以下是网上提供的另外一种办法，但是经验证无效！！！

```tex
针对jdk1.8.44以上版本，请将jre/lib/security/java.security文件中的

将 #crypto.policy=unlimited

改为 crypto.policy=unlimited

其他不变，也不需要其他权限jar

针对jdk1.8.44以下版本，请将jre/lib/security/ 下 的 local_policy.jar和US_export_policy.jar替换为官方网站提供了JCE无限制权限策略文件
```





 