使用yum安装非常简单：

```bash
yum install subversion
```

## 2.1. 创建仓库

我们这里在`/home`下建立一个名为`svn`的仓库（repository），以后所有代码都放在这个下面，创建成功后在svn下面多了几个文件夹。

```bash
[root@localhost /]# cd /home
[root@localhost home]# mkdir svn
[root@localhost home]# svnadmin create /home/svn
[root@localhost home]# ls svn
conf  db  format  hooks  locks  README.txt
```

我们这里特别关注一下`conf`文件夹，这个是存放配置文件的

```bash
[root@localhost home]# cd svn/conf
[root@localhost conf]# ls
authz  passwd  svnserve.conf
```

其中：

- authz 是权限控制文件
- passwd 是帐号密码文件
- svnserve.conf 是SVN服务配置文件

接下来我们依次修改这3个文件。

## 2.2. 配置passwd

```
[root@localhost conf]# vi passwd 
[users]
test1=123456
test2=123456
```

上面的例子中我们创建了2个用户，一个test1，一个test2

## 2.3. 配置authz

```
[root@localhost conf]# vi authz 
[/]
liuxianan=rw
test1=r
test2=r
*=
```

上面配置的含义是，`liuxianan`对`/home/svn/`下所有文件具有可读可写权限，`test`只有只读权限，除此之外，其它用户均无任何权限，最后一行`*=`很重要不能少。

### 2.3.1. 拓展：使用用户分组

```
[root@localhost conf]# vi authz
[groups]
group1 = liuxianan
group2 = test1,test2
[/]
@group1 = rw
@group2 = r
* =
```

上面配置中创建了2个分组，分组1的用户可读可写，分组2的用户只读。

格式说明：

> 版本库目录格式：
> [<版本库>:/项目/目录]
> @<用户组名> = <权限>
> <用户名> = <权限>

## 2.4. 配置svnserve.conf

```
[root@localhost conf]# vi svnserve.conf 
打开下面的5个注释
anon-access = read #匿名用户可读
auth-access = write #授权用户可写
password-db = passwd #使用哪个文件作为账号文件
authz-db = authz #使用哪个文件作为权限文件
realm = /home/svn # 认证空间名，版本库所在目录
```

2点注意：

- 最后一行的realm记得改成你的svn目录
- 打开注释时切记前面**不要留有空格**，否则可能有问题

# 启动与停止

```
[root@localhost conf]# svnserve -d -r /home/svn（启动）
[root@localhost conf]#killall svnserve（停止）
```

上述启动命令中，`-d`表示守护进程， `-r` 表示在后台执行。停止还可以采用杀死进程的方式：