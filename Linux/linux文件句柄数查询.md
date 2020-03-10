## 查看线程占句柄数

```sh
$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 59367
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 59367
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
其中：
open files                      (-n) 1024         代表每个
```

## 查看系统打开句柄最大数量

```sh
$ cat /proc/sys/fs/file-max
```

## 查看打开句柄总数

```sh
$ lsof | awk '{print $2}' | wc -l
```

根据打开文件句柄的数量降序排列，其中第二列为进程ID：

```sh
$ lsof | awk '{print $2}' | sort | uniq -c | sort -nr | more
```

## 修改单进程最大文件连接数

```shell
$ vi /etc/security/limits.conf
*soft　　nofile　　65536
*hard　　nofile　　65536
```

修改以后保存，注销当前用户，重新登录，执行ulimit -a ,ok ,参数生效了：