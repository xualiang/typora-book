### 版本兼容性检查

检查操作系统、MySQL、中间件、应用之间的兼容性，确定升级的版本。



### 停止所有访问MySQL的应用

停止应用访问，防止数据继续写入。



### 备份所有数据库

升级MySQL通常不会丢失数据，但保险起见，我们需要做这一步。输入命令： 

mkdir /software  建立一个文件夹存放备份文件 

mysqldump -u root  -p  --all-databases  --default-character-set=utf8  >  /software/databases.sql



### 停止MySQL

service mysqld stop



###  

下载并安装最新的rpm文件

 地址： <http://repo.mysql.com/yum>

使用rpm -Uvh --nodeps安装

 

 启动MySQL，输入命令：

service mysqld start

 

启动完成后，输入命令查看MySQL版本号：

mysql -V



执行以下命令升级系统表：

mysql_upgrade -uroot -p

执行完后，记得重启mysqld

