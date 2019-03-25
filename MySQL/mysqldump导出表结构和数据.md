mysqldump是mysql用于转存储数据库的实用程序。它主要产生一个SQL脚本，其中包含从头重新创建数据库所必需的命令CREATE TABLE INSERT等。

下面我们详细介绍一下mysqldump导出的各种实例：

 

1 导出一个数据库的结构

mysqldump -d dbname -uroot -p > dbname.sql

2 导出多个数据库的结构

mysqldump -d -B dbname1 dbname2 -uroot -p > dbname.sql

3 导出一个数据库中数据（不包含结构）

mysqldump -t dbname -uroot -p > dbname.sql

4 导出多个数据库中数据（不包含结构）

mysqldump -t -B dbname1 dbname2 -uroot -p > dbname.sql

5 导出一个数据库的结构以及数据

mysqldump dbname -uroot -p > dbname.sql

6 导出多个数据库的结构以及数据

mysqldump -B dbname1 dbname2 -uroot -p > dbname.sql

7 导出一个数据库中一个表的结构

mysqldump -d dbname1 tablename -uroot -p > tablename.sql

8 导出一个数据库中多个表的结构

mysqldump -d -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql

9 导出一个数据库中一个表的数据（不包含结构）

mysqldump -t dbname1 tablename -uroot -p > tablename.sql

10 导出一个数据库中多个表的数据（不包含结构）

mysqldump -t -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql

11 导出一个数据库中一个表的结构以及数据

mysqldump dbname1 tablename -uroot -p > tablename.sql

12 导出一个数据库中多个表的结构以及数据

mysqldump -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql

 

13 导出一张表的部分数据，命令格式如下：

mysqldump -u用户名 -p密码 数据库名 表名 --where="筛选条件" > 导出文件路径

 

例子：

从meteo数据库的sdata表中导出sensorid=11 且 fieldid=0的数据到 /home/xyx/Temp.sql 这个文件中

mysqldump -uroot -p123456 meteo sdata --where=" sensorid=11 and fieldid=0" > /home/xyx/Temp.sql

mysqldump -uroot -p123456 meteo sdata --where=" sensorid=11" > /home/xyx/Temp.sql

mysqldump -uroot -p123456 meteo sdata --where=" sensorid in (1,2,3) " > /home/xyx/Temp.sql

 

 

存储过程&函数操作

1 只导出存储过程和函数(不导出结构和数据，要同时导出结构的话，需要同时使用-d)

mysqldump -R -ndt dbname -u root -p > dbname.sql

2 只导出事件

mysqldump -E -ndt dbname -u root -p > dbname.sql

3 不导出触发器（触发器是默认导出的–triggers，使用–skip-triggers屏蔽导出触发器）

mysqldump --skip-triggers dbname1 -u root -p > dbname.sql

 

把导出的数据导入到数据库

mysql -u root -p

use dbname;

source dbname.sql

 

总结一下：

-d 结构(--no-data:不导出任何数据，只导出数据库表结构)

 

-t 数据(--no-create-info:只导出数据，而不添加CREATE TABLE 语句)

 

-n (--no-create-db:只导出数据，而不添加CREATE DATABASE 语句）

 

-R (--routines:导出存储过程以及自定义函数)

 

-E (--events:导出事件)

 

--triggers (默认导出触发器，使用--skip-triggers屏蔽导出)

 

-B (--databases:导出数据库列表，单个库时可省略）

 

--tables 表列表（单个表时可省略）

①同时导出结构以及数据时可同时省略-d和-t

②同时 不 导出结构和数据可使用-ntd

③只导出存储过程和函数可使用-R -ntd

④导出所有(结构&数据&存储过程&函数&事件&触发器)使用-R -E(相当于①，省略了-d -t;触发器默认导出)

⑤只导出结构&函数&事件&触发器使用 -R -E -d