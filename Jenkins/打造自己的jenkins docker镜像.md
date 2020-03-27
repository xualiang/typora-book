## 参考

官方镜像仓库地址：

https://hub.docker.com/r/jenkins/jenkins           (master镜像)

https://hub.docker.com/r/jenkins/jnlp-slave/   （slave镜像）

可以查看所有可用的tag，例如：后缀lts-alpine代表基于alpine的长期支持版本。

Jenkins Docker项目地址：https://github.com/jenkinsci/docker，有详细的使用说明。

## 基于官方镜像安装常用插件

```dockerfile
FROM jenkins/jenkins:latest

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt

ENV JENKINS_UC https://mirrors.tuna.tsinghua.edu.cn/jenkins
ENV JENKINS_UC_DOWNLOAD $JENKINS_UC

RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
```

**预装的插件如下：**

```sh
$ cat plugins.txt
bouncycastle-api:latest
git:latest
subversion:latest
deploy:latest
maven-plugin:latest
gitlab-plugin:latest
locale:latest
localization-zh-cn:latest
nodejs:latest
build-timeout:latest
ldap:latest
pam-auth:latest
workflow-aggregator:latest
ssh-slaves:latest
timestamper:latest
ws-cleanup:latest
docker-plugin:latest
docker-workflow:latest
kubernetes:latest
gradle:latest
sonar
```

## 安装常用的工具

```dockerfile
... 接上
USER root
## Maven
RUN set -eux; \
        wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz; \
        tar x apache-maven-3.6.3-bin.tar.gz; \
        chown -R jenkins:jenkins apache-maven-3.6.3; \
        mv apache-maven-3.6.3/conf/settings.xml apache-maven-3.6.3/conf/settings.xml.bak

COPY settings.xml apache-maven-3.6.3/conf/settings.xml
## Nodejs
RUN set -eux; \
        wget https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz; \
        tar xf node-v12.16.1-linux-x64.tar.xz; \
        rm -f node-v12.16.1-linux-x64.tar.xz; \
        chown -R jenkins:jenkins node-v12.16.1-linux-x64
## Android
RUN set -eux; \
        wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip; \
        unzip sdk-tools-linux-4333796.zip -d android-home; \
        rm -f sdk-tools-linux-4333796.zip; \
        chown -R jenkins:jenkins android-home; \
        mv android-home/tools android-home/android-tools
## Gradle
RUN set -eux; \
        wget https://services.gradle.org/distributions/gradle-5.4.1-all.zip; \
        unzip gradle-5.4.1-all.zip; \
        rm -f gradle-5.4.1-all.zip; \
        chown -R jenkins:jenkins gradle-5.4.1

USER jenkins

ENV MAVEN_HOME /apache-maven-3.6.3
ENV NODE_HOME /node-v12.16.1-linux-x64
ENV ANDROID_HOME /android-home
ENV GRADLE_HOME /gradle-5.4.1
ENV PATH $PATH:$MAVEN_HOME/bin
ENV PATH $PATH:$NODE_HOME/bin
ENV PATH $PATH:$ANDROID_HOME/android-tools:$ANDROID_HOME/android-tools/bin:$ANDROID_HOME/platform-tools
ENV PATH $PATH:$GRADLE_HOME/bin

RUN set -eux; \
        npm config set registry http://10.10.70.220:8081/repository/npm-group/; \
        echo y | sdkmanager "build-tools;29.0.3"; \
        echo y | sdkmanager "platform-tools"; \
        echo y | sdkmanager "platforms;android-28"; \
        mvn -v; node -v; gradle --version
```

## 使用方法

**制作镜像：**

```sh
$ docker build -t jenkins/jenkins-telchina:latest 
```

**启动容器：**

```sh
$ docker run -d --name jenkins_dev -p 8989:8080 -v /mnt/disk01/jenkins-data/_data/:/var/jenkins_home jenkins/jenkins-telchina:latest
```

**访问Jenkins：**

http://ip:8989

跳过插件安装页面即可，因为镜像中已经预装了插件，您只需要在：系统管理->全局工具配置页面中，添加Maven、Gradle、NodeJS配置即可，Name属性随意指定，路径选择镜像中各个工具的安装目录（见Dockerfile）。

然后，您就可以开始创建构建任务了...

## Jenkins及插件版本升级

如果要升级运行中的jenkins容器的版本以及插件版本，docker rm删除容器，然后使用新版本的jenkins image重新创建容器，并挂载原来的volumn用以恢复所有的构建任务即可，但是执行docker run时必须要添加两个ENV，否则容器中的插件版本不会升级到新版本image中的插件版本：

```sh
-e TRY_UPGRADE_IF_NO_MARKER=true -e PLUGINS_FORCE_UPGRADE=true
```

> By default, plugins will be upgraded if they haven't been upgraded manually and if the version from the docker image is newer than the version in the container. Versions installed by the docker image are tracked through a marker file.
>
> To force upgrades of plugins that have been manually upgraded, run the docker image with `-e PLUGINS_FORCE_UPGRADE=true`.
>
> The default behaviour when upgrading from a docker image that didn't write marker files is to leave existing plugins in place. If you want to upgrade existing plugins without marker you may run the docker image with `-e TRY_UPGRADE_IF_NO_MARKER=true`. Then plugins will be upgraded if the version provided by the docker image is newer.