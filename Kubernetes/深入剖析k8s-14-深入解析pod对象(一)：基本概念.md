### 14|深入解析pod对象（一）：基本概念

pod是k8s项目中最小的编排单位，而不是容器！

容器是Pod对象中的一个属性字段。

哪些属性属于pod？哪些又属于容器呢？

可以把pod看成传统环境中的”虚拟机“，所以凡是**调度、网络、存储、以及安全相关的属性，基本上都是pod级别的**

**NodeSelector：**根据一定的规则，将pod调度到符合条件的node上

**NodeName：**越过了调度器，直接指定pod所在的node，调试时用到。

**HostAliases：**定义pod的hosts文件



**凡是跟容器的Linux Namespace相关的属性，也一定是pod级别的。如下：**

shareProcessNamespace: true    ##pod中所有容器共享PID Namespace



**凡是Pod中的容器共享宿主机的Namespace，也一定是pod级别的。如下：**

hostNetwork: true  ##直接使用宿主机的网络

hostIPC: true     ##直接与宿主机进行IPC通信

hostPID: true    ##看到宿主机里运行的进程



**Containers字段也属于pod对象，包括Init Containers。**



**Container对象包含的属性：**

Image、Command、workingDir、Ports、volumeMounts等。

ImagePullPolicy：默认是alway，即每次创建pod都重新拉取镜像。如果容器镜像类似于nginx、nginx:latest时，默认也是alway。如果定义为Never或IfNotPresent，意味着pod永远不会拉取镜像或者只在不存在时才拉取。

Lifecycle：Container Lifecycle Hooks。在容器状态发生变化时触发一系列”钩子“。**lifecycle.postStart**在容器启动后立刻执行一个指定的操作，但是并**不严格保证顺序**，有可能执行时ENTRYPOINT还没结束。**preStop**在容器被杀死之前执行一个操作，它会阻塞当前的容器杀死流程，直到这个hook定义操作完成后，容器才会被杀死，可以用来实现容器的”**优雅退出**“



**Pod对象的生命周期：**

pod生命周期体现在Pod API对象的**Status**部分。它是除了**Metadata、Spec**第三个重要的字段。其中**pod.status.phase**就是pod的当前状态，包括：

1. Pending：pod的yaml已经提交给 k8s，API对象已经被创建并保存在Etcd中，但是这个pod中有些容器因为某些原因不能被顺利创建。
2. Running：pod已经调度成功，跟一个具体的节点绑定。它包含的所有容器创建成功，并至少有一个正在运行中。
3. Succeeded：pod里所有容器都运行完毕，并且已经退出了，这种情况在运行一次性任务时最常见。
4. Failed：Pod里至少有一个容器以不正常的状态退出。需要Debug这个容器的应用，比如查看pod的Events和日志。
5. Unknown：pod的状态不能持续的被kubelet汇报给kube-apiserver，有可能主从节点之间通信出现问题。

status字段还可以细分出一组Conditions。这些细分状态的值包括：PodScheduled、Ready、Initialized、UnSchedulable。它们主要用以描述当前status的具体原因。比如：status是pending，对应的Condition是UnSchedulable，意味着它的调度出了问题。

Ready这个细分状态非常值得关注：它意味着pod不仅已经正常启动（running状态），而且已经可以对外提供服务了。