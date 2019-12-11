### token过期问题确认

Kubeadm join命令中加入--v=5，显示更详细的日志信息如下：

```sh
root@k8s-worker02:~# kubeadm join 10.10.51.64:6443 --token uhcqg1.0livp8gwlnyltxbo --discovery-token-ca-cert-hash sha256:a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7 --v=5

I1112 08:34:01.221385   15665 join.go:433] [preflight] Discovering cluster-info
I1112 08:34:01.221451   15665 token.go:199] [discovery] Trying to connect to API Server "10.10.51.64:6443"
I1112 08:34:01.221938   15665 token.go:74] [discovery] Created cluster-info discovery client, requesting info from "https://10.10.51.64:6443"
I1112 08:34:01.229803   15665 token.go:202] [discovery] Failed to connect to API Server "10.10.51.64:6443": token id "uhcqg1" is invalid for this cluster or it has expired. Use "kubeadm token create" on the control-plane node to create a new valid token
```

提示需要重新创建token（token的默认有效期是24小时）

### 创建新token

```sh
root@k8s-master:~/.kube/cache/discovery/10.10.51.64_6443# kubeadm token create
wq7ukn.u32xsiobtfyumv9q
```

### 获取ca证书sha256编码hash值

```sh
root@k8s-master:~/.kube/cache/discovery/10.10.51.64_6443# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex 
(stdin)= a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7
```

### 重新执行kubeadm join

```sh
root@k8s-worker02:~# kubeadm join 10.10.51.64:6443 --token wq7ukn.u32xsiobtfyumv9q --discovery-token-ca-cert-hash sha256:a8957919563d9f8bc62d4902e2a316c26f01780125a44294062256f0632c5cf7 --v=5
```

