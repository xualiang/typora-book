### 拉取私服镜像registry并启动私服容器：

```bash
docker pull registry:2

docker run -d -p 5000:5000 --restart=always --name registry -v /docker-registry-data:/var/lib/registry registry:2
```

 ### 部署docker-registry-frontend网站

```sh
docker run \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=10.10.50.204 \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -p 8080:80 \
  --name docker-registry-frontend \
  konradkleine/docker-registry-frontend:v2
```

### 部署

```sh
docker run -it -p 18080:8080 --name registry-web --link registry -e REGISTRY_URL=http://10.10.50.204:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web 
```

### 为将要上传私服的镜像打标签后上传私服

```bash
docker tag jenkinsci/blueocean 10.10.50.204:5000/jenkins

docker push 10.10.50.204:5000/jenkins
```

### 使用私服：修改默认仓库地址

cd /etc/docker/

vim daemon.json          //修改为：{ "insecure-registries":["10.10.50.204:5000"] }

 systemctl restart docker.service 

 ### 访问私服

curl -X GET "http://10.10.50.204:5000/v2/_catalog"

curl 'http://10.10.50.204:5000/v2/alpine/tags/list'

