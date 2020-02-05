**Ambari-server相关目录****：**

Ambari-server配置文件目录如下：

<img src="../images/image-20200205190354135.png" alt="image-20200205190354135" style="zoom:50%;" />

Ambari-server安装目录：

<img src="../images/image-20200205190416020.png" alt="image-20200205190416020" style="zoom:50%;" />

Ambari-server resources目录：

<img src="../images/image-20200205190451365.png" alt="image-20200205190451365" style="zoom:50%;" />

Ambari-server/resources/common-services目录：

<img src="../images/image-20200205190526718.png" alt="image-20200205190526718" style="zoom:50%;" />

Ambari-server/resources/common-services/FLUME目录：

<img src="../images/image-20200205190606306.png" alt="image-20200205190606306" style="zoom:50%;" />

/var/lib/ambari-server/resources/stacks/目录：

<img src="../images/image-20200205190644284.png" alt="image-20200205190644284" style="zoom:50%;" />

**Ambari-agent相关目录****：**

Ambari-agent配置文件目录如下：

<img src="../images/image-20200205190719186.png" alt="image-20200205190719186" style="zoom:50%;" />

Ambari-Agent安装目录：

<img src="../images/image-20200205190735721.png" alt="image-20200205190735721" style="zoom:50%;" />

**HDP相关目录****：**

配置文件目录位于/etc下，但其实都是链接文件，指向了/usr/hdp/current/../conf,，如下所示：

<img src="../images/image-20200205190823174.png" alt="image-20200205190823174" style="zoom:50%;" />

/usr/hdp/current/目录下也都是软链接文件，指向了/usr/hdp/2.3.0.0/目录下的服务，如下所示：

<img src="../images/image-20200205190902997.png" alt="image-20200205190902997" style="zoom:50%;" />

hdp的日志文件基本都位于/var/log/目录下，如下所示：

<img src="../images/image-20200205190925912.png" alt="image-20200205190925912" style="zoom:50%;" />