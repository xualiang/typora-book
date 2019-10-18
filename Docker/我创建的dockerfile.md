### 具有sshd服务的Ubuntu18.04镜像

#### dockerfile

```dockerfile
FROM ubuntu

MAINTAINER xucl(xucunliang@telchina.net)

RUN apt-get update \
&& apt-get install openssh-server -y \
&& mkdir -p /var/run/sshd \   ##没有这个目录，sshd无法启动
&& mkdir -p /root/.ssh \
&& echo "PermitRootLogin yes" >> /etc/ssh/sshd_config  ##Ubuntu默认不允许以root身份ssh登录，必须开启这个配置，才能使用root登录

ADD authorized_keys /root/.ssh/authorized_keys  ##免密登录设置，将宿主机的公钥拷贝进容器

ADD run.sh /run.sh
RUN chmod a+x /run.sh

EXPOSE 22

CMD ["/run.sh"]
```

#### run.sh

```sh
#!/bin/bash
/usr/sbin/sshd -D
```

