**通过Jenkins执行远程脚本后台运行，不报错，不启动**

原始命令： nohup java -jar app.jar & 

解决方法： 

(方法1) 使用setsid命令 

source /etc/profile

setsid java -jar app.jar >> out.log 2>&1 & 

注意：source /etc/profile 如果没有，会报错哦 

setsid: failed to execute java: No such file or directory 

 

(方法2) 

source /etc/profile

\#BUILD_ID=dontKillMe

nohup java -jar app.jar > nohup.out & 2>&1 & 

\#sleep 3 && echo finished

\#exit 0

注意：source /etc/profile 如果没有，没有错误也不会有java执行的进程 

增加 source /etc/profile 后不能直接使用nohup java -jar app.jar & 这时候Jenkins会打印app启动信息，不能很快完毕，甚至就不能完毕，以错误的信息结束，其实，server已经运行了