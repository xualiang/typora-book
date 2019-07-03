场景：公司为了防止代码泄露，要求所有开发工作在虚拟桌面内进行，公司具有统一的gitlab服务，但只能在虚拟桌面内才能访问。虚拟桌面与外部网络不通，无法从虚拟桌面中拷贝出任何文件，但是可以从外部向虚拟桌面内拷贝文件。

工作任务：将当前gitlab上所有的项目迁移到公司统一的gitlab

思路：

1. 将当前gitlab仓库clone到本地

2. 将本地仓库拷贝到虚拟桌面内

3. 在虚拟桌面内将本地仓库push到公司gitlab

过程：

一、将当前gitlab仓库clone到本地

git clone http://10.10.50.204:9080/police/videobds-java.git将远程仓库克隆到本地

cd videobds-java

git branch -a查看所有的本地分支和远程分支，会看到本地只有一个分支develop(默认分支)，而远程有多个分支

git checkout -t origin/master；git checkout -t origin/integration将所有的远程分支checkout到本地仓库，-t参数会自动创建和远程分支相同名称的本地分支

二、登录到虚拟桌面内，可以看到你个人电脑中的硬盘，将第一步中clone的本地仓库拷贝到您虚拟桌面的一个目录下即可

三、在虚拟桌面内将本地仓库push到公司gitlab

1. 登录公司的gitlab页面，创建项目并拷贝地址
2. cd videobds-java进入项目主目录内
3. git remote remove origin清除原有的远程仓库地址
4. git remote add origin http://10.10.70.200/BigData/THYF2016-007VideoBigDataPlatform/videobds-java.git重新添加远程仓库地址为公司gitlab中的项目地址
5. git push origin master；git push origin develop；git push origin integration推送所有的分支
6. git push origin --tags推送所有的tag
7. 在公司gitlab界面中检查项目的分支、tag、提交次数等