# 安装Docker Client

在个人Mac上安装docker client：

```sh
brew install docker
```

docker服务端配置：

```sh
cat /etc/docker/daemon.json
{
  "hosts":["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 10,
  "registry-mirrors":["https://8ktmlv98.mirror.aliyuncs.com"],
  "insecure-registries":["10.10.50.204:5000"]
}
```

添加如上第一行，然后重启docker服务，使用systemctl重启报错，需要修改如下：

```sh
cat /usr/lib/systemd/system/docker.service
...
[Service]
Type=notify
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock
```

去掉-H fd://即可。

测试：

```sh
docker -H tcp://10.10.50.204:2375 ps -a
```

或者：

```sh
export DOCKER_HOST="tcp://10.10.50.204:2375"
docker ps -a
```

# Docker Version Manager

https://howtowhale.github.io/dvm/

# 安装kubernetes-cli

## 参考

https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/

## 方法一

```bash
brew install kubernetes-cli
```

由于众所周知的网络原因，安装失败。

## 方法二

下载kubectl，地址如下：

```sh
https://storage.googleapis.com/kubernetes-release/release/${VERSION}/bin/darwin/amd64/kubectl
# 请将${VERSION}替换为具体版本号
```

添加可执行权限：

```
chmod +x ./kubectl
```

放置到PATH目录下：

```
mv ./kubectl /usr/local/bin/kubectl
```

连接远程Kubernetest服务配置：

```sh
mkdir -p ~/.kube
vim ~/.kube/config  ## 将服务器端的config文件拷贝过来
```

测试：

```sh
kubectl get nodes
```

# 安装helm-cli

```sh
brew install helm
```

