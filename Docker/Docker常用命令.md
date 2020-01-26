```bash
安装：
rpm -ivh docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm 
rpm -ivh docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm 

启动：
systemctl start docker.service 

版本：
docker version 

版本详细信息：
docker info

帮助：
man docker-inspect

docker search --automated -s 10 jenkins
docker search --automated jenkins
docker search jenkins
docker search registry


docker pull ubuntu:12.04

docker images 
docker tag jenkinsci/blueocean:latest jenkins
docker history  jenkinsci/blueocean

docker inspect -f {{".Architecture"}} jenkins
docker inspect jenkins

docker create -it jenkinsci/blueocean

docker run   -u root   --rm   -d   -p 8080:8080   -p 50000:50000   -v jenkins-data:/var/jenkins_home   -v /var/run/docker.sock:/var/run/docker.sock   jenkinsci/blueocean
docker run -d jenkinsci/blueocean /bin/bash -c "while true;do echo hello;sleep 5;done"
docker logs df
docker run -it ubuntu:12.04 
docker run ubuntu:12.04 echo "aaa"
docker ps 
docker ps  -a

docker start 2b8e1d94c953
docker stop df
docker restart df

docker attach df
docker exec -it 2b8e1d94c953 bash

docker rm 3bb337c37996
docker rmi jenkins:latest

docker commit -m "add file /tmp/test_commit.txt" -a "xucl" 50c2e38ae20c jenkins

docker save  -o jenkins.tar jenkinsci/blueocean
docker load --input jenkins.tar 

docker export -o test_exp_docker.tar 50
docker import test_exp_docker.tar test_exp_docker

docker login

docker cp /root/jdk-8u151-linux-x64.tar.gz 807:/root/

docker run -it -d -p 8080:8080 -p 9031:9031 -p 9036:9036 -v jenkins-data:/root/.jenkins -v /var/log/jenkins:/var/log/jenkins xucl/jenkins_vbds3

下载docker的方法：
yum install --downloadonly --downloaddir=/tmp/docker docker
```

