Metrics安装报错如下：

resource_management.core.exceptions.Fail: Execution of '/usr/bin/yum -d 0 -e 0 -y install ambari-metrics-monitor' returned 1. Error: Package: ambari-metrics-monitor-2.1.0-1470.x86_64 (ambari)
      Requires: python-devel
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest

 

缺少python-devel，安装后retry成功