### PV和PVC的定义

**PV** 的全称是：**PersistentVolume（持久化卷）**，是对底层的共享存储的一种抽象，PV 由**管理员**进行**事先**创建和配置，它和具体的底层的共享存储技术的实现方式有关，比如**远程文件存储 Ceph、GlusterFS、NFS** 等，**远程块存储（公有云提供的远程磁盘）**，Kubernetes通过**插件机制**完成与这些共享存储的对接。

**PVC** 的全称是：**PersistentVolumeClaim（持久化卷声明）**，PVC 是用户存储的一种声明，PVC 和 Pod 比较类似，**Pod 消耗的是节点，PVC 消耗的是 PV 资源，Pod 可以请求 CPU 和内存，而 PVC 可以请求特定的存储空间和访问模式**。对于真正使用存储的**用户不需要关心底层的存储实现细节，只需要直接使用 PVC 即可**。

PVC 和 PV 的设计，其实跟“**面向对象**”的思想完全一致。PVC是**接口**，PV是具体**实现**。

### PV和PVC的绑定

用户创建的 PVC 要真正被容器使用起来，就必须先和某个符合条件的 PV 进行**绑定**。需要检查两部分条件：

1. PV 和 PVC 的 spec 字段。比如，PV 的存储(storage)大小，必须满足 PVC 的要求。
2. PV 和 PVC 的 storageClassName 字段必须一样。

**一个棘手的情况：**创建 Pod 的时候，系统里**没有合适的 PV** 跟它定义的 PVC 绑定，也就是说此时容器想要使用的 Volume 不存在。这时候，**Pod 的启动就会报错**。过了一会，管理员发现了这个情况，赶紧**补建了一个PV**，我们期望Kubernetes 能够**再次完成 PVC 和 PV 的绑定操作**，从而启动 Pod。

**PersistentVolumeController**扮演的就是撮合 PV 和 PVC 的“**红娘**”的角色。它不断地查看当前每一个 PVC，是不是已经处于 Bound状态。如果不是，那它就会遍历所有的、可用的 PV，并尝试将其与这个“单身”的 PVC 进行绑定。

PV 与 PVC 绑定，其实就是将这个 PV 对象的名字，填在了 PVC 对象的 **spec.volumeName** 字段上。

### PV对象如何成为容器的Volume

> 所谓**容器的 Volume**，其实就是**将一个宿主机上的目录，跟一个容器里的目录绑定挂载在了一起**。而所谓的**持久化 Volume**，指的就是这宿主机上的目录，具备**持久性**。即：这个目录里面的内容，既不会因为容器的删除而被清理掉，也不会跟当前的宿主机绑定。这样，当**容器被重启或者在其他节点上重建出来之后，它仍然能够通过挂载这个 Volume**， 访问到这些内容。hostPath 和 emptyDir 类型的 Volume 并不具备这个特征。

**这个准备持久化宿主机目录的过程，我们可以形象地称为两阶段处理：**

> 一个 Pod 调度到一个节点上之后，kubelet 就要负责为这个 Pod 创建它的 Volume 目录，/var/lib/kubelet/pods/<Pod的ID>/volumes/kubernetes.io~<Volume 类型 >/<Volume名 >

1. 如果是**远程块存储**，比如Google Cloud的Persistent Disk(GCE提供的远程磁盘服务)，kubelet先调用 Goolge Cloud 的API，将它所提供的Persistent Disk挂载到 Pod 所在的宿主机上。**这个阶段称为Attach。**
2. 格式化这个磁盘设备，然后将它挂载到宿主机指定的挂载点上。**这个阶段称为Mount。**

这个两阶段处理流程，是靠独立于kubelet主控制循环(**Kubelet Sync Loop**)之外的两个控制循环来实现的：

1. Attach(以及Dettach)操作，是由 **Volume Controller** 负责维护的，这个控制循环是**AttachDetachController**，它不断检查每一个 Pod 对应的 PV，和这个 Pod 所在宿主机之间挂载情况。从而决定，是否需要对这个 PV 进行Attach(或者Dettach)操作。Volume Controller是 **kube- controller-manager **的一部分。所以，AttachDetachController 也一定是运行在 **Master** 节点上的。Attach 操作只需要**调用公有云或者具体存储项目的 API**，并不需要在具体的宿主机上执行操作。
2. Mount(以及Unmount)操作，必须发生在 **Pod 对应的宿主机上**，所以它必须是**kubelet**组件的一部分。这个控制循环的名字，叫作: **VolumeManagerReconciler**，它运行起来之后，是一个**独立于 kubelet 主循环的 Goroutine**。

这样将 Volume 的处理同 kubelet 的主循环解耦，Kubernetes 就避免了这些耗时的 远程挂载操作拖慢 kubelet 的主控制循环，进而导致 Pod 的创建效率大幅下降的问题。 实际上，**kubelet 的一个主要设计原则，就是它的主控制循环绝对不可以被 block**。

### StorageClass

> 大规模生产环境存在的问题：有成千上万的PVC，需要创建成千上万的PV，而且随时有新的PVC创建。