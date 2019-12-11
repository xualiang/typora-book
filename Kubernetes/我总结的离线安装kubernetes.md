### 所有节点离线安装docker-ce

通过制作离线源的方式安装，不再多说。

问题：系统自带的libseccomp2版本低	

解决方法：阿里镜像网站中搜索libseccomp2，下载libseccomp2_2.4.1-2_amd64.deb，手动安装

### 所有节点离线安装kubeadm

也是通过制作离线源的方式安装，不再多说。

### 制作本地镜像仓库

可以先执行一次kubeadm init，报错信息中提示了所需的镜像，然后使用能联网的机器，执行docker pull，docker tag，docker push几个命令，完成本地镜像仓库的创建。

### Master节点执行kubeadm init

编辑kubeadm2.yaml文件如下，使用自定义的镜像库：10.10.50.204:5000/google_containers，设置：horizontal-pod-autoscaler-use-rest-clients

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
controllerManager:
  ExtraArgs:
    horizontal-pod-autoscaler-use-rest-clients: "true"
    horizontal-pod-autoscaler-sync-period: "10s"
    node-monitor-grace-period: "10s"
apiServer:
  ExtraArgs:
    runtime-config: "api/all=true"
    kubernetesVersion: "v1.16.2"
imageRepository: "10.10.50.204:5000/google_containers"
```

执行kubeadm init，报错如下：

```verilog
root@k8s_master:~# kubeadm init --config /tmp/kubeadm2.yaml 
W1110 15:33:51.586140    7158 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: dial tcp: lookup dl.k8s.io on 127.0.0.53:53: server misbehaving
W1110 15:33:51.586182    7158 version.go:102] falling back to the local client version: v1.16.2
nodeRegistration.name: Invalid value: "k8s_master": a DNS-1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')
To see the stack trace of this error execute with --v=5 or higher
```

解决方法：hostnamectl set-hostname k8s-master

仍然报错如下：

```verilog
root@k8s_master:~# kubeadm init --config /tmp/kubeadm2.yaml 
W1110 15:36:24.695141    7641 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: dial tcp: lookup dl.k8s.io on 127.0.0.53:53: server misbehaving
W1110 15:36:24.695183    7641 version.go:102] falling back to the local client version: v1.16.2
[init] Using Kubernetes version: v1.16.2
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.4. Latest validated version: 18.09
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

解决方法：swapoff -a

解决WARNING：detected "cgroupfs" as the Docker cgroup driver，方法如下：

{
"exec-opts": ["native.cgroupdriver=systemd"]
}

仍然报错如下：

```verilog
root@k8s_master:~# kubeadm init --config /tmp/kubeadm2.yaml 
W1110 15:50:10.931667   10415 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: dial tcp: lookup dl.k8s.io on 127.0.0.53:53: server misbehaving
W1110 15:50:10.931920   10415 version.go:102] falling back to the local client version: v1.16.2
[init] Using Kubernetes version: v1.16.2
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.4. Latest validated version: 18.09
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR ImagePull]: failed to pull image 10.10.50.204:5000/google_containers/kube-apiserver:v1.16.2: output: Error response from daemon: Get https://10.10.50.204:5000/v2/: http: server gave HTTP response to HTTPS client
, error: exit status 1
        [ERROR ImagePull]: failed to pull image 10.10.50.204:5000/google_containers/kube-controller-manager:v1.16.2: output: Error response from daemon: Get https://10.10.50.204:5000/v2/: http: server gave HTTP response to HTTPS client
, error: exit status 1
```

解决方法：

```sh
root@k8s_master:~# cat /etc/docker/daemon.json              
{
"exec-opts": ["native.cgroupdriver=systemd"],
"insecure-registries":["10.10.50.204:5000"] 
}
root@k8s_master:~# systemctl restart docker.service
```

Kubeadm init执行成功如下：

```sh
root@k8s_master:~# kubeadm init --config /tmp/kubeadm2.yaml
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.10.51.64:6443 --token uhcqg1.0livp8gwlnyltxbo \
    --discovery-token-ca-cert-hash sha256:a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7
```

### kubectl授权配置

根据提示执行：

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 安装网络插件

检查node状态和pod状态：

```sh
root@k8s_master:~# kubectl get nodes
NAME         STATUS     ROLES    AGE    VERSION
k8s-master   NotReady   master   7m3s   v1.16.2
root@k8s_master:~# kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-68c6b978c4-bh77r             0/1     Pending   0          7m33s
kube-system   coredns-68c6b978c4-k9kvr             0/1     Pending   0          7m33s
kube-system   etcd-k8s-master                      1/1     Running   0          6m43s
kube-system   kube-apiserver-k8s-master            1/1     Running   0          6m30s
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          6m24s
kube-system   kube-proxy-2rp5j                     1/1     Running   0          7m33s
kube-system   kube-scheduler-k8s-master            1/1     Running   0          6m20s
```

发现node状态为NotReady，coredns pods状态为Pending，检查原因：

```sh
root@k8s_master:~# kubectl describe node k8s-master
Conditions:          
  MemoryPressure   False   Sun, 10 Nov 2019 16:03:09 +0000   Sun, 10 Nov 2019 15:55:03 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 10 Nov 2019 16:03:09 +0000   Sun, 10 Nov 2019 15:55:03 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 10 Nov 2019 16:03:09 +0000   Sun, 10 Nov 2019 15:55:03 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Sun, 10 Nov 2019 16:03:09 +0000   Sun, 10 Nov 2019 15:55:03 +0000   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```

原因为缺少网络插件，其实kubeadm init命令的执行结果中已经有提示了，提示我们参考：https://kubernetes.io/docs/concepts/cluster-administration/addons/，安装任意一种网络插件。

以weave网络插件为例，官方安装说明如下：

```sh
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

由于我们是离线服务器，所以首先通过联网电脑下载yaml文件：

```
root@k8s-master:~# kubectl version | base64 | tr -d '\n'
Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiIxNiIsIEdpdFZlcnNpb246InYxLjE2LjIiLCBHaXRDb21taXQ6ImM5N2ZlNTAzNmVmM2RmMjk2N2QwODY3MTFlNmMwYzQwNTk0MWUxNGIiLCBHaXRUcmVlU3RhdGU6ImNsZWFuIiwgQnVpbGREYXRlOiIyMDE5LTEwLTE1VDE5OjE4OjIzWiIsIEdvVmVyc2lvbjoiZ28xLjEyLjEwIiwgQ29tcGlsZXI6ImdjIiwgUGxhdGZvcm06ImxpbnV4L2FtZDY0In0KU2VydmVyIFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiIxNiIsIEdpdFZlcnNpb246InYxLjE2LjIiLCBHaXRDb21taXQ6ImM5N2ZlNTAzNmVmM2RmMjk2N2QwODY3MTFlNmMwYzQwNTk0MWUxNGIiLCBHaXRUcmVlU3RhdGU6ImNsZWFuIiwgQnVpbGREYXRlOiIyMDE5LTEwLTE1VDE5OjA5OjA4WiIsIEdvVmVyc2lvbjoiZ28xLjEyLjEwIiwgQ29tcGlsZXI6ImdjIiwgUGxhdGZvcm06ImxpbnV4L2FtZDY0In0K
```

下载地址：https://cloud.weave.works/k8s/net?k8s-version=以上版本编码

将下载的net.yaml文件上传到离线服务器，执行：

```sh
root@k8s-master:~# kubectl apply -f /tmp/net.yaml 
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
root@k8s-master:~# kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS             RESTARTS   AGE
kube-system   coredns-68c6b978c4-bh77r             0/1     Pending            0          29m
kube-system   coredns-68c6b978c4-k9kvr             0/1     Pending            0          29m
kube-system   etcd-k8s-master                      1/1     Running            0          29m
kube-system   kube-apiserver-k8s-master            1/1     Running            0          28m
kube-system   kube-controller-manager-k8s-master   1/1     Running            0          28m
kube-system   kube-proxy-2rp5j                     1/1     Running            0          29m
kube-system   kube-scheduler-k8s-master            1/1     Running            0          28m
kube-system   weave-net-x2nk4                      0/2     ImagePullBackOff   0          17s
root@k8s-master:~# kubectl describe pod weave-net-x2nk4 -n kube-system
......
  Warning  Failed     92s (x3 over 2m)     kubelet, k8s-master  Error: ImagePullBackOff
  Normal   Pulling    78s (x3 over 2m1s)   kubelet, k8s-master  Pulling image "docker.io/weaveworks/weave-kube:2.6.0"
```

解决方法：

1. 手工下载所需的镜像并push到本地镜像仓库
2. 修改net.yaml中的镜像仓库地址
3. 重新执行kubectl apply。

再次检查node和pod：

```sh
root@k8s-master:~# kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-68c6b978c4-bh77r             0/1     Running   0          36m
kube-system   coredns-68c6b978c4-k9kvr             1/1     Running   0          36m
kube-system   etcd-k8s-master                      1/1     Running   0          35m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          35m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          35m
kube-system   kube-proxy-2rp5j                     1/1     Running   0          36m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          35m
kube-system   weave-net-6tv87                      2/2     Running   0          28s
root@k8s-master:~# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   36m   v1.16.2
```

### worker节点加入集群

```sh
root@k8s-worker01:~# kubeadm join 10.10.51.64:6443 --token uhcqg1.0livp8gwlnyltxbo --discovery-token-ca-cert-hash sha256:a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.4. Latest validated version: 18.09
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

解决方法：

1. 关闭swap：swapoff -a
2. 修改docker cgroup驱动为systemd

join命令执行成功：

```sh
root@k8s-worker01:~# kubeadm join 10.10.51.64:6443 --token uhcqg1.0livp8gwlnyltxbo --discovery-token-ca-cert-hash sha256:a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.4. Latest validated version: 18.09
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

检查nodes和pod状态：

```sh
root@k8s-master:~# kubectl get nodes
NAME           STATUS     ROLES    AGE   VERSION
k8s-master     Ready      master   45m   v1.16.2
k8s-worker01   NotReady   <none>   34s   v1.16.2
root@k8s-master:~# kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS              RESTARTS   AGE 
kube-system   kube-proxy-2rp5j                     1/1     Running             0          46m
kube-system   kube-proxy-rkj28                     0/1     ContainerCreating   0          100s
kube-system   kube-scheduler-k8s-master            1/1     Running             0          45m
kube-system   weave-net-mdt52                      0/2     ContainerCreating   0          100s
root@k8s-master:~# kubectl describe pod -n kube-system kube-proxy-rkj28
Events:

  Warning  FailedCreatePodSandBox  14s (x4 over 117s)  kubelet, k8s-worker01  Failed create pod sandbox: rpc error: code = Unknown desc = failed pulling image "10.10.50.204:5000/google_containers/pause:3.1": Error response from daemon: Get https://10.10.50.204:5000/v2/: http: server gave HTTP response to HTTPS client
```

解决方法：

```sh
root@k8s-worker01:~# cat /etc/docker/daemon.json 
{
        "exec-opts": ["native.cgroupdriver=systemd"],
        "insecure-registries":["10.10.50.204:5000"]
}
root@k8s-worker01:~# systemctl restart docker.service
```

不需要重新执行join命令，直接检查node和pod状态，发现立刻恢复正常！！