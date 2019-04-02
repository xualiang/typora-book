**简介**

查阅了网上相关资料，介绍zookeeper客户端命令并不是非常全面，大多数都是简单介绍ls、get、set、delete、stat这几个简单命令的，下面我把help中的所有命令简单介绍一下以供参考。

bin/zkServer.shstart 启动服务端

bin/zkCli.sh –server ip:port 启动客户端

**help命令**

显示客户所支持的所有命令，如：

ZooKeeper -server host:port cmd args

​       connecthost:port

​       getpath [watch]

​       lspath [watch]

​       setpath data [version]

​       rmrpath

​       delquota[-n|-b] path

​       quit

​       printwatcheson|off

​       create[-s] [-e] path data acl

​       statpath [watch]

​       close

​       ls2path [watch]

​       history

​       listquotapath

​       setAclpath acl

​       getAclpath

​       syncpath

​       redocmdno

​       addauthscheme auth

​       deletepath [version]

​       setquota-n|-b val path

**connect命令**

连接zk服务端，与close命令配合使用可以连接或者断开zk服务端。

如connect 127.0.0.1:2181

**get命令**

获取节点信息，注意节点的路径皆为绝对路径，也就是说必要要从/（根路径）开始。

如get /

hello world

cZxid = 0x0

ctime = Thu Jan 01 08:00:00 CST 1970

mZxid = 0x5

mtime = Thu Apr 27 15:09:00 CST 2017

pZxid = 0xc

cversion = 1

dataVersion = 2

aclVersion = 0

ephemeralOwner = 0x0

dataLength = 11

numChildren = 1

详解：

hello world为节点数据信息

cZxid节点创建时的zxid

ctime节点创建时间

mZxid节点最近一次更新时的zxid

mtime节点最近一次更新的时间

cversion子节点数据更新次数

dataVersion本节点数据更新次数

aclVersion节点ACL(授权信息)的更新次数

ephemeralOwner如果该节点为临时节点,ephemeralOwner值表示与该节点绑定的session id. 如果该节点不是临时节点,ephemeralOwner值为0

dataLength节点数据长度，本例中为hello world的长度

numChildren子节点个数

**set命令**

设置节点的数据。

如set /zookeeper "hello world"

**rmr命令**

删除节点命令，此命令与delete命令不同的是delete不可删除有子节点的节点，但是rmr命令可以删除，注意路径为绝对路径。

如rmr /zookeeper/znode

**delquota命令**

删除配额，-n为子节点个数，-b为节点数据长度。

如delquota –n 2，请参见listquota和setquota命令。

**printwatches命令**

设置和显示监视状态，on或者off。

如printwatches on

**create命令**

创建节点，其中-s为顺序充点，-e临时节点。

如create /zookeeper/node1"test_create" world:anyone:cdrwa

其中acl处，请参见getAcl和setAcl命令。

**stat命令**

查看节点状态信息。如stat /

cZxid = 0x0

ctime = Thu Jan 01 08:00:00 CST 1970

mZxid = 0x1f

mtime = Thu Apr 27 16:05:14 CST 2017

pZxid = 0xc

cversion = 1

dataVersion = 3

aclVersion = 0

ephemeralOwner = 0x0

dataLength = 5

numChildren = 1

与get命令大体相同，请参见get命令。

**close命令**

断开客户端与服务端的连接。

**ls2命令**

ls2为ls命令的扩展，比ls命令多输出本节点信息。

如 ls /zookeeper

**history命令**

列出最近的历史命令。

如history

0 - ls /

1 - ls /

2 - ls2 /

3 - history

4 - listquota /zookeeper

5 – history

基本格式为：命令ID-命令，可以与redo命令配合使用。

 

**listquota命令**

显示配额。

如listquota /zookeeper

absolute path is/zookeeper/quota/zookeeper/zookeeper_limits

Output quota for /zookeepercount=2,bytes=-1

解释：

/zookeeper节点个数限额为2，长度无限额。

 

**setAcl命令**

设置节点Acl。

此处重点说一下acl，acl由大部分组成：1为scheme，2为user，3为permission，一般情况下表示为：scheme:id：permissions。

其中scheme和id是相关的，下面将scheme和id一起说明。

 

**scheme和id**

**world**: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的

**auth**: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)

**digest**: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication

**ip**: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段

**super**: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

**permissions**

**CREATE**(c): 创建权限，可以在在当前node下创建child node

**DELETE**(d): 删除权限，可以删除当前的node

**READ**(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes

**WRITE**(w): 写权限，可以向当前node写数据

**ADMIN**(a): 管理权限，可以设置当前node的permission

综上，一个简单使用setAcl命令，则可以为：

setAcl /zookeeper/node1 world:anyone:cdrw

 **getAcl命令**

获取节点Acl。

如getAcl /zookeeper/node1

'world,'anyone

: cdrwa

注：可参见setAcl命令。

**sync命令**

强制同步。

如sync /zookeeper

由于请求在半数以上的zk server上生效就表示此请求生效，那么就会有一些zk server上的数据是旧的。sync命令就是强制同步所有的更新操作。

**redo命令**

再次执行某命令。

如redo 10

其中10为命令ID，需与history配合使用。

**addauth命令**

节点认证。

如addauth digest username:password，可参见setAcl命令**digest处。**

使用方法：

一、通过setAcl设置用户名和密码

setAcl pathdigest:username:base64(sha1(password)):crwda

二、认证

addauth digest username:password



**setquota命令**

设置子节点个数和数据长度配额。

如setquota –n 4 /zookeeper/node 设置/zookeeper/node子节点个数最大为4

setquota –b 100 /zookeeper/node 设置/zookeeper/node节点长度最大为100