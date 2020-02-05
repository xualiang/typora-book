rpm -ivh openssl-libs-1.0.2k-12.el7.x86_64.rpm --replacefiles

rpm -ivh openssl-1.0.2k-12.el7.x86_64.rpm --replacefiles 

rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm

rpm -ivh wxBase-2.8.12-20.el7.x86_64.rpm

rpm -ivh erlang-20.3-1.el7.centos.x86_64.rpm

rpm -ivh rabbitmq-server-3.7.7-1.el7.noarch.rpm

 

 

rabbitmq-plugins enable rabbitmq_management

 

vim /etc/rabbitmq/rabbitmq.config

[

{rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["asdf"]}]}

].

 

 

systemctl start rabbitmq-server.service

 

http://10.10.51.43:15672/  guest guest