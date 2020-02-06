- ## 配置java环境

- ```sh
  export JAVA_HOME=/usr/java/jdk1.8.0_151
  export JRE_HOME=/usr/java/jdk1.8.0_151/jre
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
  export PATH=$JAVA_HOME/bin:$PATH
  ```

- ## 安装flume

- 安装包下载官网的,不要下载cdh的,否则坑太多了!

- 解压并配置环境变量:

- ```sh
  export FLUME_HOME=/home/app/flume-1.6.0
  export PATH=$FLUME_HOME/bin:$PATH
  ```

- ## 配置flume-env.sh文件

- ```sh
  $ vim flume-env.sh
  export JAVA_HOME=/home/hadoop/app/jdk1.7.0_79
  export JAVA_OPTS="-Xms1024m -Xmx2048m -Dcom.sun.management.jmxremote"
  ```

- ## 版本验证

- ```sh
  $ flume-ng version
  ```

- ## 启动agent

- 1. 编辑conf配置文件

- 2. 添加自定义jar包到flume lib目录下

- 3. 修改log4j配置
  4. 启动agent

- ```sh
  $ nohup flume-ng agent -c /usr/local/flume1.6/conf -f /usr/local/flume1.6/conf/kafka-hdfs.conf -n agent > /dev/null 2>&1 &
  ```

- 

- 