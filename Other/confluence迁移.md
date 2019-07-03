1、迁移安装目录，默认是/opt/atlassian/confluence/

2、迁移数据目录，默认是/var/atlassian/application-data/confluence，具体可以查看./confluence/atlassian-confluence-4.1.5/confluence/WEB-INF/classes/confluence-init.properties

3、创建操作系统用户confluence，将安装目录和数据目录（/var/atlassian/app-data/confluence）的属主改为confluence

4、破解过程，其实是生成confluence.cfg.xml的过程，所以要保证数据目录/var/atlassian/application-data/confluence下没有这个文件，并且confluence用户有写入目录的权限

5、创建confluence数据库、用户和权限，注册过程中会自动建表，注册完成后，把所有77张表清空

6、将原系统的mysql库导出，然后导入到新库