显示进程准确运行时间：

ps -A -opid,stime,etime,args

 

显示进程具体启动时间

ps -eo pid,lstart,cmd | grep corosync

