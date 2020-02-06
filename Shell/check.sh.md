## 脚本内容：

```sh
#!/bin/sh

function gc_run(){ 
	taskid=$(date +"%Y%m%d%H%M%S%3N")
	cmd=$1
	for ip in $(seq -f "150.18.19.%g" 176 178)
	do
		{
		echo "--------- $ip ---------"
		ssh -q $ip  "$cmd"
		} > ./$ip.$taskid &
	done
	wait
	for ip in $(seq -f "150.18.19.%g" 176 178)
	do
		cat ./$ip.$taskid
		rm -f ./$ip.$taskid
	done
}

function gn_run(){
        cmd=$1
	for ip in $(seq -f "150.18.19.%g" 181 196)
        do
                {
                echo "--------- $ip ---------"
                ssh -q $ip  "$cmd"
                } > ./$ip.$taskid &
        done
        wait
        for ip in $(seq -f "150.18.19.%g" 181 196)
        do
                cat ./$ip.$taskid
                rm -f ./$ip.$taskid
        done
}

log_file=./log/check.log.$(date +"%Y%m%d%H%M%S")

# gcadmin
echo "#################### check gcadmin ####################" | tee -a $log_file
gcadmin | tee -a $log_file

# gcadmin lock
echo "#################### check gcluster lock ####################" | tee -a $log_file
gcadmin showlock | tee -a $log_file

# gcadmin event
echo "#################### check gcluster event ####################" | tee -a $log_file
(echo -n "DDL ";gcadmin showddlevent | grep count) | tee -a $log_file
(echo -n "DML ";gcadmin showdmlevent | grep count) | tee -a $log_file
(echo -n "DML Storage ";gcadmin showdmlstorageevent | grep count) | tee -a $log_file

# show processlist
echo "#################### show processlist ####################" | tee -a $log_file
python ~/liulz/show_processlist/show_processlist.py -ap c | tee -a $log_file

# gclusterd
echo "#################### check gclusterd ####################" | tee -a $log_file
gc_run "ps -ef | grep gclusterd | grep -vE 'grep|bash|init.d'" | tee -a $log_file

# gbased
echo "#################### check gbased ####################" | tee -a $log_file
gn_run "ps -ef | grep gbased | grep -vE 'grep|bash|init.d'" | tee -a $log_file

# gcluster dump or core
echo "#################### check gcluster core file (within 7 day) ####################" | tee -a $log_file
gc_run "find /opt/gcluster/userdata/gcluster -maxdepth 1 -type f ! -name express.* -mtime -7 -exec ls -l {} \;" | tee -a $log_file

# gnode dump or core
echo "#################### check gnode core file (within 7 day) ####################" | tee -a $log_file
gn_run "find /opt/gnode/userdata/gbase -maxdepth 1 -type f ! -name express.* -mtime -7 -exec ls -l {} \;" | tee -a $log_file

#cpu info
echo "#################### check cpu status ####################" | tee -a $log_file
gc_run "top -bn2 -d.1 | grep -E '^Cpu' | tail -n 1" | tee -a $log_file
gn_run "top -bn2 -d.1 | grep -E '^Cpu' | tail -n 1" | tee -a $log_file

#mem info
echo "#################### check mem status ####################" | tee -a $log_file
gc_run "free -m | grep -E 'Swap|Mem'" | tee -a $log_file
gn_run "free -m | grep -E 'Swap|Mem'" | tee -a $log_file

#disk info
echo "#################### check disk status ####################" | tee -a $log_file
gc_run "df -h | grep -E '/$|/opt'" | tee -a $log_file
gn_run "df -h | grep -E '/$|/opt'" | tee -a $log_file

# check gcluster log 
echo "#################### check gcluster log ####################" | tee -a $log_file
gc_run "du -sh /opt/gcluster/log" | tee -a $log_file
gn_run "du -sh /opt/gnode/log" | tee -a $log_file

# check nohup process
echo "#################### check nohup process ####################" | tee -a $log_file
for p in collect_audit_log.py show.sh show_full.sh gnode_show_full.sh
do
	if [ $(ps -ef | grep -E "\s$p" | grep -v grep | wc -l) -gt 0 ];then
		echo  "$p is running, ($(ps -ef | grep -E "\s$p" | grep -v grep))" | tee -a $log_file
	else
		echo "$p is stop!!!!!!!!!!!!!!!!!!!!!!!!" | tee -a $log_file
	fi
done

# check dayly table
echo "#################### check dayly table ####################" | tee -a $log_file
gccli -uroot -psm@2017Root -Ns -e "select 'audit_log today count: '||count(1) from dw_zh.audit_log where start_time>to_char(sysdate(),'yyyy-mm-dd')" | tee -a $log_file
gccli -uroot -psm@2017Root -Ns -e "select 'db_table_size today count: '||count(1) from dw_zh.db_table_size where date(start_time)= to_char(sysdate(),'yyyy-mm-dd')" | tee -a $log_file
gccli -uroot -psm@2017Root -Ns -e "select 'tables today count: '||count(1) from dw_zh.tables where start_time>= to_char(sysdate(),'yyyy-mm-dd')" | tee -a $log_file
gccli -uroot -psm@2017Root -Ns -e "select 'columns today count: '||count(1) from dw_zh.columns where start_time>= to_char(sysdate(),'yyyy-mm-dd')" | tee -a $log_file
```

## 学习记录：

```sh
taskid=$(date +"%Y%m%d%H%M%S%3N")                 ## 注意$()的作用
[gbase@redhat7 ~]$ date +"%Y-%m-%d %H:%M:%S:%3N"  ## date格式化
2017-12-18 22:33:05:556
```

```sh
[gbase@redhat7 ~]$ seq -f "150.18.19.%g" 176 178  ## seq -f
150.18.19.176
150.18.19.177
150.18.19.178
```

```sh
for i in `seq 1 3`
do
{
  echo "----------------127.0.0.1-------------"
  ssh -q 127.0.0.1 "date; sleep 2"                ## ssh -q并行远程执行
} > /tmp/test."${i}".log&
done
wait                                              ## wait的含义
```

```sh
[gbase@redhat7 ~]$ echo -n "date:";date|tee -a /tmp/date.log   ## echo -n不换行；tee -a
date:Mon Dec 18 23:25:27 CST 2017
```

```sh
find /opt/gcluster/log/ -maxdepth 1 -type f -name express.* -mtime -7 -exec grep ERROR {} \;
```

