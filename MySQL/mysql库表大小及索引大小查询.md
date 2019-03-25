 

1、进去指定schema 数据库（存放了其他的数据库的信息）

​     mysql> use information_schema;

 

2、查询所有数据的大小 

​     mysql> select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES;

 

3、查看指定数据库实例的大小，比如说数据库 yoon 

​     mysql> select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES where table_schema='yoon';

 

4、查看指定数据库的表的大小，比如说数据库 yoon 中的 yoon 表 

​     mysql> select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES 

​                 where table_schema='yoon' and table_name='yoon';

 

5、指定库的索引大小：

SELECT CONCAT(ROUND(SUM(index_length)/(1024*1024), 2), ' MB') AS 'Total Index Size' FROM TABLES  WHERE table_schema = 'sakila'; 

 

6、指定库的指定表的索引大小：

SELECT CONCAT(ROUND(SUM(index_length)/(1024*1024), 2), ' MB') AS 'Total Index Size' FROM TABLES  WHERE table_schema = 'test' and table_name='sakila'; 

 

7、一个库中的使用情况：

SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', CONCAT(ROUND(table_rows/1000000,4),'M') AS 'Number of Rows', CONCAT(ROUND(data_length/(1024*1024*1024),4),'G') AS 'Data Size', CONCAT(ROUND(index_length/(1024*1024*1024),4),'G') AS 'Index Size', CONCAT(ROUND((data_length+index_length)/(1024*1024*1024),4),'G') AS'Total'FROM information_schema.TABLES WHERE table_schema LIKE 'sakila';