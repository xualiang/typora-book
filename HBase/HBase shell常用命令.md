进入命令：hbase shell

常用命令：



Help

Status

Version



Create 'table_name','family1','family2'

List

Describe 'table_name'

Alter 'table_name',{NAME=>'family',METHOD=>'delete'}

Disable 'table_name'

Enable 'table_name'

Drop 'table_name'

Exists 'table_name'

Is_enabled 'table_name'

Is_disabled 'table_name'

 

Put 'table_name','key','family:column','value'

Get 'table_name','key'

Get 'table_name','key','family'

Get 'table_name','key','fammily:column'

Get 'table_name','key',{COLUMN=>'family:col',TIMESTAMP=>123456789}

 

Scan 'table_name'

 

Delete 'table_name','rowkey','family:col_name'

Deleteall 'table_name','rowkey'

 

Count 'table_name'

Truncate 'table_name'