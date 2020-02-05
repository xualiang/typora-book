# SonarQube

## 简介

- Continuous Inspection
- Detect Tricky Issues
- Centralize Quality
- DevOps Integration

## 下载

SonarQube官网：  
[https://www.sonarqube.org/downloads/](https://www.sonarqube.org/downloads/)  
目前最新版本是6.4： **sonarqube-6.4.zip**


## 环境要求

新版本的环境要求比较高，例如强制要求 **JRE8+** 和 **MySQL5.6+** ，请提前配置好，不要抱有侥幸心理。。

JRE下载地址：
[http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)  
下载后设置好PATH及JAVA_HOME环境变量即可  

MySQL下载地址：
[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)  
可以下载Linux-Generic二进制版本，手工安装  


## 主机准备

134.32.51.31

## 建用户

useradd -d/home/sonar sonar
passwd sonar

## 安装及配置

解压即可，仅需配置数据库连接参数

```
vi conf/sonar.properties

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:mysql://localhost:3316/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

## 启动

```
cd sonarqube-6.4/bin/linux-x86-64
./sonar.sh start
```

## WEB界面

http://134.32.51.31:9000  
默认用户admin/admin

## 设置代理(可选)

vi conf/sonar.properties
socksProxyHost=134.32.32.13
socksProxyPort=31080

## 安装Sonar插件

Administration  ->  System  ->  Update Center

## 使用Maven插件

https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven  

```
<profile>
    <id>sonar</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <sonar.host.url>
          http://134.32.51.31:9000
        </sonar.host.url>
    </properties>
</profile>
```

```
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.3.0.603</version>
</plugin>
```

```
mvn clean verify sonar:sonar
```