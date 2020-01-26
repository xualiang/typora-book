### PV和PVC的用法有没有过度设计嫌疑？

> 比如：公司的运维管理人员像以往一样维护一套NFS或Ceph服务器，根本不必学习Kubernetes。而开发人员靠复制粘贴的方式，在Pod的yaml中填上Volumes字段，而不需要去使用PVC。

如果只是为了职责划分，PV、PVC 体系确实不见得比直接在 Pod 里声明 Volumes 字段有什么优势。

不过，如果[Kubernetes内置的 20 种持久化数据卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)实现，都没办法满足你的容器存储需求时，该怎么办?

### 本地持久化卷需求

> 用户呼声最高的定制化需求，莫过于支持**“本地”持久化存储**了。也就是说，直接使用**宿主机上的本地磁盘目录**，而**不依赖于远程存储**服务，来提供**持久化的容器 Volume**。由于这个Volume直接使用的是**本地磁盘**，尤其是 **SSD**盘，它的读写**性能**相比于大多数远程存储来说，要**好得多**。这个需求对**本地物理服务器部署的私有 Kubernetes 集群**来说，非常常见。

Kubernetes 在 v1.10 之后，逐渐依靠PV、PVC体系实现了这个特性。这个特性的名字叫作：**Local Persistent Volume**。**适用范围：**

1. 高优先级的系统应用，需要在多个不同节点上存储数据，并且对 I/O 较为敏感。典型的应用包括:分布式数据存储比如 MongoDB、Cassandra 等；
2. 分布式文件系统比如 GlusterFS、Ceph等；
3. 需要在本地磁盘上进行大量数据缓存的分布式应用。

相比于远程共享存储方式的PV，Local Persistent Volume所在的一些**节点宕机且不能恢复时， 数据就可能丢失**。这就要求使用 Local Persistent Volume 的应用必须具备**数据备份和恢复**的能力，允许你把这些数据定时备份在其他位置。

### Local Persistent Volume设计的难点

1. 如何把本地磁盘抽象成PV。

> 你可能会想：在Pod中声明使用一个hostPath类型的Volume，这个**hostPath对应的目录已经在A节点上事先创建好**了，那么我只需要再给这个Pod加上一个**nodeAffinity=nodeA**，不就可以使用这个Volume了吗？

一个 Local Persistent Volume 对应的存储介质，一定是一块额外挂载在宿主机的磁盘或者块设备(它不应该是宿主机根目录所使用的主硬盘)。否则，磁盘随时都可能被应用写满，甚至造成整个宿主机宕机。这个原则，我们可以称为**一个 PV 一块盘**。

2. 调度器如何保证 Pod 始终能被正确地调度到它所请求的 Local Persistent Volume 所在的节点上呢?

> 常规PV不存在这个问题，因为：Kubernetes 都是先调度 Pod 到某个节点上，然后，再为这个节点实现持久化Volume（通过Attach和Mount两阶段处理），进而完成 Volume 目录与容器的绑定挂载。
>
> 可是，对于 Local PV 来说，节点上可供使用的磁盘(或块设备)，必须是运维人员提前准备好的。它们在不同节点上的挂载情况可以完全不同，甚至有的节点可以没这种磁盘。这时候，调度器就必须能够知道所有Node上Local PV的具体情况，然后根据这个信息来调度 Pod。这个原则，我们可以称为**“在调度的时候考虑 Volume 分布”**。有一个叫作 **VolumeBindingChecker 的过滤条件**专门负责这个事情。在 Kubernetes v1.11 中，这个过滤条件已经默认开启了。

### 动手实践

1. 为Node-1挂载一块硬盘或RAM Disk（内存盘）

```sh
# 在 node-1 上执行
$ mkdir /mnt/disks
$ for vol in vol1 vol2 vol3; 
  do
    mkdir /mnt/disks/$vol
    mount -t tmpfs $vol /mnt/disks/$vol 
  done
```

2. 为挂载的硬盘定义PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
    - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: local-storage 
    local:
      path: /mnt/disks/vol1
    nodeAffinity:
      required: 
        nodeSelectorTerms: 
        - matchExpressions:
          - key: kubernetes.io/hostname 
            operator: In
            values:
            - node-1
```

