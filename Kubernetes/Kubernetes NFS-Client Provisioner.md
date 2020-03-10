StorageClass用于为PVC动态创建PV，StorageClass的定义中必须提供Provisioner字段，Provisioner是供应PV的存储插件实现。Volume Plugin与Provisioner的对应列表：

| Volume Plugin        | Internal Provisioner | Config Example                                               |
| :------------------- | :------------------- | :----------------------------------------------------------- |
| AWSElasticBlockStore | ✓                    | [AWS EBS](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs) |
| AzureFile            | ✓                    | [Azure File](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file) |
| AzureDisk            | ✓                    | [Azure Disk](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk) |
| CephFS               | -                    | -                                                            |
| Cinder               | ✓                    | [OpenStack Cinder](https://kubernetes.io/docs/concepts/storage/storage-classes/#openstack-cinder) |
| FC                   | -                    | -                                                            |
| FlexVolume           | -                    | -                                                            |
| Flocker              | ✓                    | -                                                            |
| GCEPersistentDisk    | ✓                    | [GCE PD](https://kubernetes.io/docs/concepts/storage/storage-classes/#gce-pd) |
| Glusterfs            | ✓                    | [Glusterfs](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| iSCSI                | -                    | -                                                            |
| Quobyte              | ✓                    | [Quobyte](https://kubernetes.io/docs/concepts/storage/storage-classes/#quobyte) |
| NFS                  | -                    | -                                                            |
| RBD                  | ✓                    | [Ceph RBD](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| VsphereVolume        | ✓                    | [vSphere](https://kubernetes.io/docs/concepts/storage/storage-classes/#vsphere) |
| PortworxVolume       | ✓                    | [Portworx Volume](https://kubernetes.io/docs/concepts/storage/storage-classes/#portworx-volume) |
| ScaleIO              | ✓                    | [ScaleIO](https://kubernetes.io/docs/concepts/storage/storage-classes/#scaleio) |
| StorageOS            | ✓                    | [StorageOS](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageos) |
| Local                | -                    | [Local](https://kubernetes.io/docs/concepts/storage/storage-classes/#local) |

上表可见，官方并没有提供NFS的内置Provisioner，所以需要使用外部provisioner，[kubernetes-incubator
/external-storage](https://github.com/kubernetes-incubator/external-storage)项目就提供了很多外部Provisioner，其中就包括NFS-Client。

## 安装

### 使用Helm

遵循项目 https://github.com/helm/charts/tree/master/stable/nfs-client-provisioner 所维护的稳定版的helm chart的说明

The tl;dr is

```sh
$ helm install stable/nfs-client-provisioner --set nfs.server=x.x.x.x --set nfs.path=/exported/path
```

### 不使用Helm

- 确保你的Kubernetes集群与NFS Server网络畅通（并不需要另外部署NFS客户端，因为此Provisioner实现了客户端功能）
- 获取https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client/deploy目录下的yaml文件
- 设置授权，可以为将要部署的插件指定一个Namespace，例如：nfs-client，需要替换rbac.yaml中的Namespace，如下：

```sh
$ sed -i'' "s/namespace:.*/namespace: $NAMESPACE/g" rbac.yaml deployment.yaml
$ kubectl apply -f rbac.yaml
```

- 使用deployment.yaml部署Provisioner：
  + 修改Namespace，与rbac.yaml保持一致
  + 修改deployment.yaml配置文件，使用你的NFS Server配置替换其中的NFS_SERVER和NFS_PATH的值
  + 你也可以修改PROVISIONER_NAME的值，只要在StorageClass中使用时保持一致即可。
  + 执行`kubectl apply -f deployment.yaml`

- 创建StorageClass：`kubectl apply -f class.yaml`

## 测试

- 创建pvc和使用pvc的pod：`kubectl create -f test-claim.yaml -f test-pod.yaml`，然后检查NFS服务器的共享目录下是否多了个SUCCESS文件
- 删除pvc和pod：`kubectl delete -f deploy/test-pod.yaml -f deploy/test-claim.yaml`，然后检查文件是否被删除

## 在StatefulSet中使用

```yaml
  ...
  volumeClaimTemplates:
  - metadata:
      namespace: cale
      name: data-volume
      annotations:
        volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
    spec:
      accessModes: ["ReadWriteMany"]
      resources:
        requests:
          storage: 1Gi
```

pod启动成功后，可以进入到容器中在挂载目录下添加文件，然后检查NFS Server中是否同步了文件。

还可以测试：删除pod，pod被StatefulSet重建后，数据是否还在。

