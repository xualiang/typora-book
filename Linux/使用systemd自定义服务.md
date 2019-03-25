```shell
[root@test2 ~]# cat /etc/systemd/system/redisd.service 
[Unit]
Description=Redis server daemon
After=network.target
After=syslog.target

[Service]
Type=forking
PIDFile=/var/run/redis_7000.pid
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/7000/redis.conf

[Install]
WantedBy=multi-user.target


[root@hdp3 ~]# cat /etc/systemd/system/redis-sentineld.service 
[Unit]
Description=Redis Sentinel server daemon
After=network.target
After=syslog.target
After=redis.service

[Service]
Type=forking
PIDFile=/var/run/redis.pid
ExecStart=/usr/local/redis/bin/redis-sentinel /usr/local/redis/7001/redis-sentinel.conf

[Install]
WantedBy=multi-user.target

编辑以上两个文件后，执行以下命令即可启动redis服务及哨兵服务，并且设置了开机自启动
systemctl start redisd.service
systemctl enable redisd.service
systemctl start redis-sentineld.service
systemctl enable redis-sentineld.service
```

