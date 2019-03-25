### 拉取私服镜像registry并启动私服容器：

```bash
docker pull registry

docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
```

 

 ### 确定私服容器的IP地址

```bash
docker inspect a75e603ff996      //查看私服IP

ping 172.17.0.2
```

 

### 为将要上传私服的镜像打标签后上传私服

```bash
docker tag jenkinsci/blueocean 172.17.0.2:5000/jenkins

docker push 172.17.0.2:5000/jenkins
```

 

### 修改默认仓库地址

cd /etc/docker/

vim daemon.json          //修改为：{ "insecure-registries":["172.17.0.2:5000"] }

 

systemctl restart docker.service 

docker run -d -p 5000:5000 -v /opt/data/registry/:/tmp/registry registry



docker push 172.17.0.2:5000/jenkins

 

 ### 访问私服

curl <http://172.17.0.2:5000/v2/_catalog>