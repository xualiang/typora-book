## 检查默认字符集

- ```sql
   mysql> show variables like 'char%';
    +--------------------------+---------------------------------+
   | Variable_name            | Value                           |
   +--------------------------+---------------------------------+
   | character_set_client     | utf8                            |
   | character_set_connection | utf8                            |
   | character_set_database   | utf8                            |
   | character_set_filesystem | binary                          |
   | character_set_results    | utf8                            |
   | character_set_server     | utf8                            |
   | character_set_system     | utf8                            |
   | character_sets_dir       | /usr/local/mysql/share/charsets/|
   +--------------------------+---------------------------------+
  ```

  ```sql
   mysql> show variables like 'collation%';
    +----------------------+-----------------+
   | Variable_name        | Value           |
   +----------------------+-----------------+
   | collation_connection | utf8_general_ci |
   | collation_database   | utf8_unicode_ci |
   | collation_server     | utf8_unicode_ci |
   +----------------------+-----------------+
  ```

## 修改默认字符集

在my.cnf文件中添加如下配置后重启服务：

```sql
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8


[mysqld]
default-character-set = utf8      ### 这一句很可能不需要 
collation-server = utf8_general_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

## 建议设置为utf8mb4

```sql
# 💩 𝌆
# UTF-8 should be used instead of Latin1. Obviously.
# NOTE "utf8" in MySQL is NOT full UTF-8: http://mathiasbynens.be/notes/mysql-utf8mb4

[client]
default-character-set = utf8mb4

[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

```sql
mysql> SHOW VARIABLES LIKE 'char%'; SHOW VARIABLES LIKE 'collation%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_general_ci |
| collation_database   | utf8mb4_unicode_ci |
| collation_server     | utf8mb4_unicode_ci |
+----------------------+--------------------+
3 rows in set (0.00 sec)
```

character_set_system [is always utf8](http://dev.mysql.com/doc/refman/5.0/en/server-system-variables.html#sysvar_character_set_system). 

This won't affect existing tables, it's just the default setting (used for new tables). The following [ALTER code](https://stackoverflow.com/a/6115705) can be used to convert an existing table (without the dump-restore workaround):

```sql
ALTER DATABASE databasename CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE tablename CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

