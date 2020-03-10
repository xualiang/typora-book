## 相关资料

github项目地址：https://github.com/kubernetes/dashboard/tree/master/docs

Kubernetes中文官方文档：https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard

## 安装

kubectl apply -f recommended.yaml

https://github.com/kubernetes/dashboard/blob/master/aio/deploy/recommended.yaml

## 创建证书

参考：https://blog.csdn.net/chenleiking/article/details/81488028

### 创建自签名CA

```sh
--- 生成私钥 ---
$ openssl genrsa -out ca.key 2048

--- 生成自签名证书 ---
$ openssl req -new -x509 -key ca.key -out ca.crt -days 3650 -subj "/C=CN/ST=HB/L=WH/O=DM/OU=YPT/CN=CA"

--- 查看CA内容 ---
$ openssl x509 -in ca.crt -noout -text
```

### 签发Dashboard证书

```sh
--- 生成私钥 ---
$ openssl genrsa -out dashboard.key 2048

--- 申请签名请求 ---
$ openssl req -new -sha256 -key dashboard.key -out dashboard.csr -subj "/C=CN/ST=HB/L=WH/O=DM/OU=YPT/CN=10.10.71.80"

--- 配置文件 ---
$ cat dashboard.cnf 
extensions = san
[san]
keyUsage = digitalSignature
extendedKeyUsage = clientAuth,serverAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
subjectAltName = IP:10.10.71.80,IP:127.0.0.1,DNS:10.10.71.80,DNS:localhost

--- 签发证书 ---
$ openssl x509 -req -sha256 -days 3650 -in dashboard.csr -out dashboard.crt -CA ca.crt -CAkey ca.key -CAcreateserial -extfile dashboard.cnf

--- 查看证书 ---
$ openssl x509 -in dashboard.crt -noout -text
```

### 挂载证书到Dashboard

```sh
--- 删除已经部署的dashboard ---
$ kubectl delete -f kubernetes-dashboard.yml 

--- 创建 secret "kubernetes-dashboard-certs" ---
$ kubectl create secret generic kubernetes-dashboard-certs --from-file="tls/dashboard.crt,tls/dashboard.key" -n kubernetes-dashboard

--- 查看secret内容 ---
$ kubectl get secret kubernetes-dashboard-certs -n kubernetes-dashboard -o yaml

--- 重新部署dashboard ---
$ kubectl apply -f kubernetes-dashboard.yml 
```

### 客户端导入证书

1. 浏览器访问Dashboard网站，检查证书是否有效
2. 下载ca.crt并导入Mac钥匙串，并设置为“始终信任”

## 访问Dashboard

几种方式：

- kubectl proxy
- kubectl port-forward
- NodePort
- API Server
- Ingress

### NodePort方式：

```sh
$ kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
## Change type: ClusterIP to type: NodePort
```

## 用户权限

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

获取Token：

```sh
$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```