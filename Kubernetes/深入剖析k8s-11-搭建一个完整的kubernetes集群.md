### 安装过程：

1. 所有节点安装docker
2. 所有节点安装kubeadm
3. 部署master节点：kubeadm init
4. 环境变量配置
5. 部署网络插件
6. 加入worker节点
7. master节点污点处理（可选）
8. 部署Dashboard可视化插件（可选）
9. 部署容器存储插件Rook

### 问题总结：

1. 使用离线源安装docker-ce时，报错：与系统**libseccomp2**版本冲突，在阿里镜像站下载高版本的deb包解决。
2. kubeadm init时，config文件对k8s版本非常敏感，需要查询官网文档，进行编写。
3. kubeadm init时，通过在config文件中指定**imageRepository**，或在命令中指定**--image-repository**选项修改镜像地址，解决无法在线拉取docker镜像的问题。
4. kubeadm init时，提示主机名不合法，**主机名中不能有下划线**。
5. kubeadm init时（kubeadm join时也有同样问题），提示必须关闭swap，**swapoff -a**
6. kubeadm init时（kubeadm join时也有同样问题），警告：检测到"cgroupfs"作为Docker cgroup驱动，应该修改为**systemd**。
7. Kubeadm init时（kubeadm join时也有同样问题），拉取镜像报错，提示：server gave HTTP response to HTTPS client，解决方法如下：

```sh
root@k8s_master:~# cat /etc/docker/daemon.json              
{
"exec-opts": ["native.cgroupdriver=systemd"],
"insecure-registries":["10.10.50.204:5000"] 
}
root@k8s_master:~# systemctl restart docker.service
```

8. 如果使用Flannel作为网络插件，kubeadm init时需要通过--pod-network-cidr指定网络地址与**flannel.yaml**中保持一致，默认是：10.244.0.0/16

### 一些好用的知识：

1. 使用**hostnamectl**命令修改主机名：

```sh
linuxtechi@localhost:~$ sudo hostnamectl set-hostname "k8s-master"
linuxtechi@localhost:~$ exec bash
linuxtechi@k8s-master:~$
```

2. 通过**apt-add-repository和apt-key**命令配置apt源：

```sh
$ apt-add-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"
$ curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add
```

3. **$HOME**代表当前用户主目录
4. **$(id -u)**代表当前用户id，**$(id -g)**代表当前用户组id。

