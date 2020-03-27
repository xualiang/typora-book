## 新增节点服务器准备

1. 主机名和IP设置
2. 本机hosts设置，并且在已有主机的hosts中添加新增服务器
3. 新增服务器与已有服务器免密码登录设置
4. 开机自动加载必须的内核模块
5. sysctl.conf配置net.bridge.bridge-nf-call-iptables=1
6. ssh -V版本7.0以上
7. 确认防火墙关闭
8. swapoff -a

## 安装docker

1. 安装docker
2. 将非root用户添加到docker用户组
3. 修改daemon.json，添加私有仓库配置
4. 重启docker服务 

## 将新增服务器添加到Kubernetes集群

1. 将上一次安装或者升级时产生的文件备份一下
2. 在rke安装目录下需要具有cluster.rkestate、cluster.yml、kube_config_cluster.yml、rke_linux-amd64等几个文件
3. 修改cluster.yml，添加新增服务器配置:

```yaml
- address: rke-worker04.com
  port: "22"
  internal_address: 10.10.71.84
  role:
  - worker
  hostname_override: ""
  user: ubuntu
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
```

4. 执行`rke_linux-amd64 up --update-only`

## 遇到的坑

1. 执行rke up --update-only命令所在的目录需要有安装时的文件：cluster.yml、kube_config_cluster.yml、cluster.rkestate
2. 如果执行失败，需要在执行rke remove，注意：修改cluster.yml中只保留要删除的节点
3. ingress需要使用80端口