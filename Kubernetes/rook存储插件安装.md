### 为什么需要部署rook

虽然可以将宿主机目录挂载进容器，但是位于其他宿主机的容器却无法共享。

存储插件会在容器里挂载一个基于网络或其他机制的远程数据卷，使得在容器里创建的文件，实际上是保存在远程服务器上，或者以分布式的方式保存在多个节点上，而与当前宿主机没有任何绑定关系。这样无论哪个宿主机上的容器都可以挂载持久化存储卷，从而访问到数据卷中保存的内容。

### Rook简介

https://rook.io/

https://github.com/rook/rook

基于Ceph的kubernetes存储插件（后期加入了对更多存储实现的支持），不是对Ceph的简单封装，而是加入了水平扩展、迁移、灾备、监控等大量企业级功能。rook是“云原生”软件，依赖了k8s提供的编排能力，利用了Operator、CRD等重要的扩展特性。

### 下载yaml文件

三个yaml文件位于github上rook项目的目录：cluster/examples/kubernetes/ceph/

修改yaml文件中的镜像地址，改为本地镜像库。**需提前下载所需的镜像，并打tag后上传到本地镜像仓库**

### 安装

```sh
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml
```

### 检查

kubectl get pods -A

kubectl describe pod XXX -n rook-ceph

### 删除重建方法

直接删除pod后，会自动重启，所以应首先删除deployment

```sh
kubectl get deployments -A

kubectl get rs -A

kubectl delete deployment XXX -n rook-ceph
```

这时再次检查pod，发现已经自动删除。

然后，再次修改yaml文件后，执行kubectl apply即可。



