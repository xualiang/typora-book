### 下载alpine镜像

```powershell
1	[root@docker43 ~]# docker pull alpine
2	Using default tag: latest
3	Trying to pull repository docker.io/library/alpine ...
4	latest: Pulling from docker.io/library/alpine
5	4fe2ade4980c: Pull complete
6	Digest: sha256:621c2f39f8133acb8e64023a94dbdf0d5ca81896102b9e57c0dc184cadaf5528
7	Status: Downloaded newer image for docker.io/alpine:latest
8	[root@docker43 ~]# docker images
9	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
10	docker.io/alpine    latest              196d12cf6ab1        3 weeks ago         4.41 MB
```

### 编写dockerfile

```powershell
2.1.创建一个工作目录
1	[root@docker43 ~]# cd /opt/
2	[root@docker43 opt]# mkdir alpine_ssh && cd alpine_ssh && touch Dockerfile
3	 
4	[root@docker43 alpine_ssh]# ll
5	总用量 4
6	-rw-r--r-- 1 root root 654 10月  3 23:21 Dockerfile
2.2.编写Dockerfile
1	# 指定创建的基础镜像
2	FROM alpine
3	 
4	# 作者描述信息
5	MAINTAINER alpine_sshd (zhujingzhi@123.com)
6	 
7	# 替换阿里云的源
8	RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories
9	RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
10	 
11	# 同步时间
12	 
13	# 更新源、安装openssh 并修改配置文件和生成key 并且同步时间
14	RUN apk update && \
15	    apk add --no-cache openssh-server tzdata && \
16	    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
17	    sed -i "s/#PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config && \
18	    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
19	    ssh-keygen -t ecdsa -P "" -f /etc/ssh/ssh_host_ecdsa_key && \
20	    ssh-keygen -t ed25519 -P "" -f /etc/ssh/ssh_host_ed25519_key && \
21	    echo "root:admin" | chpasswd
22	 
23	# 开放22端口
24	EXPOSE 22
25	 
26	# 执行ssh启动命令
27	CMD ["/usr/sbin/sshd", "-D"]
```

### 创建镜像

```shell
2.3.创建镜像
1	# 在dockerfile所在的目录下
2	[root@docker43 alpine_ssh]# pwd
3	/opt/alpine_ssh
4	[root@docker43 alpine_ssh]# docker build -t alpine:sshd .
```

### 创建容器测试

```shell
创建容器
1	[root@docker43 alpine_ssh]# docker run -itd -p 10022:22 --name alpine_ssh_v1 alpine:sshd
2	[root@docker43 alpine_ssh]# docker ps
3	CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES
4	b353f5f3b703        alpine:sshd         "/usr/sbin/sshd -D"   17 minutes ago      Up 17 minutes       0.0.0.0:10022->22/tcp   alpine_ssh_v1
测试
1	[root@docker43 alpine_ssh]# ssh root@127.0.0.1 -p10022
2	root@127.0.0.1's password:
3	Welcome to Alpine!
4	 
5	The Alpine Wiki contains a large amount of how-to guides and general
6	information about administrating Alpine systems.
7	See <http://wiki.alpinelinux.org>.
8	 
9	You can setup the system with the command: setup-alpine
10	 
11	You may change this message by editing /etc/motd.
12	 
13	b353f5f3b703:~#
```

### 问题总结

```shell
这些都是我在手动测试的时候遇见的，已经在写Dockerfile的时候加进去了处理方法
1	1. apk add --no-cache openssh-server   # 安装openssh的问题
2	 
3	/ # apk add --no-cache openssh-server
4	fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/main/x86_64/APKINDEX.tar.gz
5	fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/community/x86_64/APKINDEX.tar.gz
6	(1/3) Installing openssh-keygen (7.7_p1-r2)
7	ERROR: openssh-keygen-7.7_p1-r2: package mentioned in index not found (try 'apk update')
8	(2/3) Installing openssh-server-common (7.7_p1-r2)
9	(3/3) Installing openssh-server (7.7_p1-r2)
10	ERROR: openssh-server-7.7_p1-r2: package mentioned in index not found (try 'apk update')
11	2 errors; 4 MiB in 14 packages
12	 
13	原因是：提示源没有这个openssh的包
14	 
15	解决方式：
16	在dockerfile中改为国内的源
17	http://mirrors.aliyun.com/alpine/latest-stable/main/
18	http://mirrors.aliyun.com/alpine/latest-stable/community/
19	 
20	创建容器文件修改
21	[root@docker43 ~]# docker run -it alpine
22	/ # vi /etc/apk/repositories
23	http://mirrors.aliyun.com/alpine/latest-stable/main/
24	http://mirrors.aliyun.com/alpine/latest-stable/community/
25	                                                     
26	#http://dl-cdn.alpinelinux.org/alpine/v3.8/main    
27	#http://dl-cdn.alpinelinux.org/alpine/v3.8/community
28	 
29	# 注释或者删除原来的默认源，添加阿里云的源，然后执行apk update，在进行安装就OK了
30	 
31	 
32	2、ssh 启动问题
33	/ # /etc/init.d/sshd start
34	/bin/sh: /etc/init.d/sshd: not found
35	 
36	这样的方式不能启动，需要安装一个alpine的管理工具
37	apk add --no-cache openrc
38	/ # /etc/init.d/sshd start
39	 * WARNING: sshd is already starting
40	 所以使用 /usr/sbin/sshd -D 方式启动。但是又出现如下错误
41	 / # /usr/sbin/sshd -D
42	Could not load host key: /etc/ssh/ssh_host_rsa_key
43	Could not load host key: /etc/ssh/ssh_host_ecdsa_key
44	Could not load host key: /etc/ssh/ssh_host_ed25519_key
45	sshd: no hostkeys available -- exiting.
46	解决方式：
47	ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key
48	ssh-keygen -t ecdsa -P "" -f /etc/ssh/ssh_host_ecdsa_key
49	ssh-keygen -t ed25519 -P "" -f /etc/ssh/ssh_host_ed25519_key
50	 
51	再次启动
52	/ # /usr/sbin/sshd -D
53	 
54	启动成功
55	 
56	 
57	3、创建容器后的网络问题
58	[root@docker43 opt]# docker run -it alpine
59	WARNING: IPv4 forwarding is disabled. Networking will not work.
60	 
61	解决方式：
62	[root@docker43 ~]# vim /etc/sysctl.conf
63	net.ipv4.ip_forward=1    # 添加这一行
64	 
65	[root@docker43 ~]# docker run -it alpine
66	/ #
```