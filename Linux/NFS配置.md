# 服务端

## 安装软件

```sh
$ yum -y install nfs-utils
$ yum -y install rpcbind
```

## 配置文件

```sh
$ mkdir /nfsroot
$ cat /etc/exports
/nfsroot 192.168.1.8(rw,no_root_squash,no_all_squash,async)
192.168.1.9(rw,no_root_squash,no_all_squash,async)
```

说明：

- 开放本地存储目录/nfsroot
- 192.168.1.8和192.168.1.9有访问权限，还可以使用：192.168.1.0/24
- rw表示允许读写
- no_root_squash表示root用户具有完全的管理权限
- no_all_squash表示保留共享文件的UID和GID，此项是默认不写也可以
- async表示数据暂时在内存中，不是直接写入磁盘，可以提高性能，sync表示数据直接同步到磁盘
- 可以配置多个共享目录，添加新行即可

## 启动服务

```sh
$ systemctl start rpcbind.service
$ systemctl start nfs.service
```

## 更改配置

两种生效方法：

1. 重启服务
2. `exportfs -a`刷新配置

# 客户端

## 安装软件

同服务端，需要nfs-utils和rpcbind，并启动

## 挂载

创建挂载点：`mkdir /mnt/nfstest`

挂载：`mount -t nfs 192.168.1.3:/nfsroot /mnt/nfstest`

挂载检查：`df -h`

## 测试

服务端往/nfsroot目录下写入文件，客户端检查是否能够看到

客户端往/mnt/nfstest目录写入文件，服务端/nfsroot目录下是否看到

## 卸载

```sh
$ umount /mnt/nfstest
```

# 开机自动挂载

```sh
$ cat /etc/fstab  ## 仅客户端
192.168.1.3:/nfsroot /mnt/nfstest nfs rw,tcp,intr 0 1
$ systemctl enable nfs.service      ## 服务端和客户端都要执行
$ systemctl enable rpcbind.service  ## 服务端和客户端都要执行
```

