停止服务：systemctl stop jenkins

备份旧war包，位置ps -ef|grep jenkins，默认是：/usr/lib/jenkins

备份jenkins数据，位置：/var/lib/jenkins

下载新版本war包：wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

重启服务：systemctl stop jenkins

登陆jenkins，升级所有插件并重启jenkins

