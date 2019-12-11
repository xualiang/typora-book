### 15|深入解析pod对象（二）：使用进阶

**Projected Volume**：投射数据卷

作用：

1. 不是为了存放容器里的数据
2. 不是用来进行容器和宿主机之间的数据交换
3. 为容器提供预先定义好的数据

这些Volume里的信息仿佛是被k8s”**投射进**“容器当中的。四种Projected Volume：

1. **Secret**
2. **ConfigMap**
3. **Downward API**
4. **ServiceAccountToken**

#### Secret：

作用：将pod想要访问的加密数据保存在Etcd中，通过在pod的容器里挂载Volume的方式访问到这些数据，例如：存放数据库的用户名和密码等Credential信息。

在pod定义中使用Secret：

```yaml
volumes:
- name: mysql-cred
  projected:
    sources:
    - secret:
        name: user
    - secret:
        name: pass
```

user和pass是两个secret对象，可以通过k8s命令生成，如下：

```sh
kubectl create secret generic user --from-file=./username.txt
kubectl create secret generic pass --from-file=./password.txt
```

username.txt和password.txt文件中存放的是用户名和密码，通过**kubectl get secrets**可以查询到创建的secret对象。

也可以通过yaml方式创建secret对象，如下：

```yaml
apiVersion: v1
kind: Secret
metedata:
  name: mysecret
type: Opaque
data:
  user: YWRtaW4=
  pass: MWYyZDF1MmU2n2RM
```

通过yaml方式只创建了**一个secret对象**，有user和pass两个key。Secret对象要求数据必须经过Base64转码，生成方法：

```sh
echo -n "admin" | base64
echo -n "pasw0rd" | base64
```

如上，创建pod后，就能看到容器里已经有用户名和密码文件了，文件名字就是key的名字。

而且，Etcd中保存的数据更新后，这些Volume里的文件内容也会更新，**这是kebelet组件在定时维护这些Volume**，但是**更新有一定延时**，所以应用程序中连接数据库的代码最好具有**失败重试和超时处理**逻辑。

#### ConfigMap

与Secret非常相似，区别是保存不需要加密的数据，例如应用所需的配置信息。

用法几乎与secret完全相同，可以通过kubectl create configmap从文件或目录创建，也可以使用yaml创建。

**示例：**

ui.properties：

```properties
color.good=green
color.bad=yellow
allow.textmode=true
```

创建ConfigMap：

```sh
kubectl create configmap ui-config --from-file=ui.properties
```

查看创建的ConfigMap：

```sh
$ kubectl get configmap ui-config -o yaml
apiVersion: v1
data:
  ui.properties:
    color.good=green
    color.bad=yellow
    allow.textmode=true
kind: ConfigMap
metadata:
  name: ui-config
  ...
```

#### Downward API

作用：让pod里的容器能够直接获取到Pod API对象本身的信息。

定义：

```yaml
volumes:
- name: podinfo
  projected:
    sources:
    - downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels  
```

如上，声明暴露pod的metadata.labels信息给容器，当前Pod的labels字段的值会被k8s自动挂载为容器中的/etc/podinfo/labels文件。

Downward API支持暴露的字段已经非常丰富了，具体请查询官方文档，但是仅能支持容器进程**启动前就能确定下来的信息**，启动后才能确定的信息应该考虑使用**sidecar**容器实现。

以上三种projected Volume定义的信息，大多也可以通过**环境变量**的方式出现在容器里，但是**不具备自动更新能力**。



