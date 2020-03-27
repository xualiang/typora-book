# Android SDK

## 下载sdk-tools并解压

```sh
cd /var/jenkins_home/android-home/
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
unzip sdk-tools-linux-4333796.zip
mv tools android-tools
```

## 环境变量

```sh
$ cat ~/.jenkins_profile
export ANDROID_HOME=/var/jenkins_home/android-home/
export PATH=$PATH:$ANDROID_HOME/android-tools:$ANDROID_HOME/android-tools/bin:$ANDROID_HOME/platform-tools
$ source ~/.jenkins_profile
```

## 测试sdk-tools

```sh
$ sdkmanager –list
```

## 安装build-tools、platform-tools、platforms

```sh
$ sdkmanager "build-tools;29.0.3"
$ sdkmanager "platform-tools"
$ sdkmanager "platforms;android-28"
```

## 检查

```sh
$ ls -l /var/jenkins_home/android-home/
total 150960
drwxr-xr-x 6 jenkins jenkins       186 Mar 17 05:35 android-tools
drwxr-xr-x 3 jenkins jenkins        20 Mar 17 05:46 build-tools
drwxr-xr-x 2 jenkins jenkins        33 Mar 17 05:44 licenses
drwxr-xr-x 5 jenkins jenkins       288 Mar 17 05:54 platform-tools
drwxr-xr-x 3 jenkins jenkins        24 Mar 17 05:54 platforms
```

# Gradle

## 下载并解压

```sh
cd /var/jenkins_home/gradle-home/
wget https://services.gradle.org/distributions/gradle-5.4.1-all.zip
unzip gradle-5.4.1-all.zip
```

## 环境变量

```sh
$ cat ~/.jenkins_profile
export ANDROID_HOME=/var/jenkins_home/android-home/
export PATH=$PATH:$ANDROID_HOME/android-tools:$ANDROID_HOME/android-tools/bin:$ANDROID_HOME/platform-tools
export GRADLE_HOME=/var/jenkins_home/gradle-home/gradle-5.4.1
export PATH=$PATH:$GRADLE_HOME/bin
$ source ~/.jenkins_profile
```

## 验证

```sh
$ gradle --version
Welcome to Gradle 5.4.1!
```

