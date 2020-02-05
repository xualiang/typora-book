DATANODE安装失败，报错如下：

resource_management.core.exceptions.Fail: Execution of '/usr/bin/yum -d 0 -e 0 -y install snappy-devel' returned 1. Error: Package: snappy-devel-1.0.5-1.el6.x86_64 (HDP-UTILS-1.1.0.20)

​      Requires: snappy(x86-64) = 1.0.5-1.el6

​      Installed: snappy-1.1.0-3.el7.x86_64 (@anaconda)

​        snappy(x86-64) = 1.1.0-3.el7

​      Available: snappy-1.0.5-1.el6.x86_64 (HDP-UTILS-1.1.0.20)

​        snappy(x86-64) = 1.0.5-1.el6

 You could try using --skip-broken to work around the problem

 You could try running: rpm -Va --nofiles --nodigest

 

根据以上提示，缺少snappy-devel，所以安装：

[root@hdp1 ~]# yum install snappy-devel.x86_64   

Loaded plugins: fastestmirror, langpacks

Loading mirror speeds from cached hostfile

Resolving Dependencies

--> Running transaction check

---> Package snappy-devel.x86_64 0:1.0.5-1.el6 will be installed

--> Processing Dependency: snappy(x86-64) = 1.0.5-1.el6 for package: snappy-devel-1.0.5-1.el6.x86_64

--> Finished Dependency Resolution

Error: Package: snappy-devel-1.0.5-1.el6.x86_64 (HDP-UTILS-1.1.0.20)

​      Requires: snappy(x86-64) = 1.0.5-1.el6

​      Installed: snappy-1.1.0-3.el7.x86_64 (@anaconda)

​        snappy(x86-64) = 1.1.0-3.el7

​      Available: snappy-1.0.5-1.el6.x86_64 (HDP-UTILS-1.1.0.20)

​        snappy(x86-64) = 1.0.5-1.el6

 You could try using --skip-broken to work around the problem

 You could try running: rpm -Va --nofiles --nodigest

 

安装报错，需要卸载snappy，然后重新安装snappy和snappy-devel