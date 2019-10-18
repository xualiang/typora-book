## redis高可用原理

redis中为了实现高可用（High Availability，简称HA），采用了如下两个方式：

- 主从复制数据。
- 采用哨兵监控数据节点的运行情况，一旦主节点出现问题由从节点顶上继续进行服务。

redis中主从节点复制数据有全量复制和部分复制之分。

## 旧版本全量复制功能的实现

全量复制使用snyc命令来实现，其流程是：

- 从服务器向主服务器发送sync命令。
- 主服务器在收到sync命令之后，调用bgsave命令生成最新的rdb文件，将这个文件同步给从服务器，这样从服务器载入这个rdb文件之后，状态就会和主服务器执行bgsave命令时候的一致。
- 主服务器将保存在命令缓冲区中的写命令同步给从服务器，从服务器执行这些命令，这样从服务器的状态就跟主服务器当前状态一致了。

[![sync](../images/sync.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/sync.png)

旧版本全量复制功能，其最大的问题是从服务器断线重连时，即便在从服务器上已经有一部分数据了，也需要进行全量复制，这样做的效率很低，于是新版本的redis在这部分做了改进。

## 新版本全量复制功能的实现

新版本redis使用psync命令来代替sync命令，该命令既可以实现完整全同步也可以实现部分同步。

### 复制偏移量

执行复制的双方，主从服务器，分别会维护一个复制偏移量：

- 主服务器每次向从服务器同步了N字节数据之后，将修改自己的复制偏移量+N。
- 从服务器每次从主服务器同步了N字节数据之后，将修改自己的复制偏移量+N。

## 复制积压缓冲区

主服务器内部维护了一个固定长度的先进先出队列做为复制积压缓冲区，其默认大小为1MB。

在主服务器进行命令传播时，不仅会将写命令同步到从服务器，还会将写命令写入复制积压缓冲区。

[![replication-backlog](../images/replication-backlog.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/replication-backlog.png)

## 服务器运行ID

每个redis服务器，都有其运行ID，运行ID由服务器在启动时自动生成，主服务器会将自己的运行ID发送给从服务器，而从服务器会将主服务器的运行ID保存起来。

从服务器redis断线重连之后进行同步时，就是根据运行ID来判断同步的进度：

- 如果从服务器上面保存的主服务器运行ID与当前主服务器运行ID一致，则认为这一次断线重连连接的是之前复制的主服务器，主服务器可以继续尝试部分同步操作。
- 否则，如果前后两次主服务器运行ID不相同，则认为是完成全同步流程。

## psync命令流程

有了前面的准备，下面开始分析psync命令的流程：

- 如果从服务器之前没有复制过任何主服务器，或者之前执行过slaveof no one命令，那么从服务器就会向主服务器发送psync ? -1命令，请求主服务器进行数据的全量同步。
- 否则，如果前面从服务器已经同步过部分数据，那么从服务器向主服务器发送psync <runid> <offset>命令，其中runid是上一次主服务器的运行id，offset是当前从服务器的复制偏移量。

前面两种情况主服务器收到psync命令之后，会出现以下三种可能：

- 主服务器返回+fullresync <runid> <offset>回复，表示主服务器要求与从服务器进行完整的数据全量同步操作。其中，runid是当前主服务器运行id，而offset是当前主服务器的复制偏移量。
- 如果主服务器应答+continue，那么表示主服务器与从服务器进行部分数据同步操作，将从服务器缺失的数据同步过来即可。
- 如果主服务器应答-err，那么表示主服务器版本低于2.8，识别不了psync命令，此时从服务器将向主服务器发送sync命令，执行完整的全量数据同步。

[![psync](../images/psync.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/psync.png)

# 哨兵机制概述

redis使用哨兵机制来实现高可用(HA)，其大概工作原理是：

- redis使用一组哨兵（sentinel）节点来监控主从redis服务的可用性。
- 一旦发现redis主节点失效，将选举出一个哨兵节点作为领导者（leader）。
- 哨兵领导者再从剩余的从redis节点中选出一个redis节点作为新的主redis节点对外服务。

以上将redis节点分为两类：

- 哨兵节点（sentinel）：负责监控节点的运行情况。
- 数据节点：即正常服务客户端请求的redis节点，有主从之分。

以上是大体的流程，这个流程需要解决以下几个问题：

- 如何对redis数据节点进行监控？
- 如何确定一个redis数据节点失效？
- 如何选择出一个哨兵领导者节点？
- 哨兵节点选择新的主redis节点的依据是什么？

以下来逐个回答这些问题。

# 三个监控任务

哨兵节点通过三个定时监控任务监控redis数据节点的服务可用性。

## info命令

每隔10秒，每个哨兵节点都会向主、从redis数据节点发送info命令，获取新的拓扑结构信息。

redis拓扑结构信息包括了：

- 本节点角色：主或从。
- 主从节点的地址、端口信息。

这样，哨兵节点就能从info命令中自动获取到从节点信息，因此那些后续才加入的从节点信息不需要显式配置就能自动感知。

[![redis-info](../images/redis-info.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/redis-info.png)

## 向__sentinel__:hello频道同步信息

每隔2秒，每个哨兵节点将会向redis数据节点的__sentinel__:hello频道同步自身得到的主节点信息以及当前哨兵节点的信息，由于其他哨兵节点也订阅了这个频道，因此实际上这个操作可以交换哨兵节点之间关于主节点以及哨兵节点的信息。

这一操作实际上完成了两件事情： * 发现新的哨兵节点：如果有新的哨兵节点加入，此时保存下来这个新哨兵节点的信息，后续与该哨兵节点建立连接。 * 交换主节点的状态信息，作为后续客观判断主节点下线的依据。

[![hello](../images/hello.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/hello.png)

## 向数据节点做心跳探测

每隔1秒，每个哨兵节点向主、从数据节点以及其他sentinel节点发送ping命令做心跳探测，这个心跳探测是后续主观判断数据节点下线的依据。

[![ping](../images/ping.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/ping.png)

# 主观下线和客观下线

## 主观下线

上面三个监控任务中的第三个探测心跳任务，如果在配置的down-after-milliseconds之后没有收到有效回复，那么就认为该数据节点“主观下线（sdown）”。

[![sdown](../images/sdown.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/sdown.png)

为什么称为“主观下线”？因为在一个分布式系统中，有多个机器在一起联动工作，网络可能出现各种状况，仅凭一个节点的判断还不足以认为一个数据节点下线了，这就需要后面的“客观下线”。

## 客观下线

当一个哨兵节点认为主节点主观下线时，该哨兵节点需要通过”sentinel is-master-down-by addr”命令向其他哨兵节点咨询该主节点是否下线了，如果有超过半数的哨兵节点都回答了下线，此时认为主节点“客观下线”。

[![odown](../images/odown.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/odown.png)

# 选举哨兵领导者

当主节点客观下线时，需要选举出一个哨兵节点做为哨兵领导者，以完成后续选出新的主节点的工作。

这个选举的大体思路是：

- 每个哨兵节点通过向其他哨兵节点发送”sentinel is-master-down-by addr”命令来申请成为哨兵领导者。
- 而每个哨兵节点在收到一个”sentinel is-master-down-by addr”命令时，只允许给第一个节点投票，其他节点的该命令都会被拒绝。
- 如果一个哨兵节点收到了半数以上的同意票，则成为哨兵领导者。
- 如果前面三步在一定时间内都没有选出一个哨兵领导者，将重新开始下一次选举。

可以看到，这个选举领导者的流程很像raft中选举leader的流程。

[![sentinel-leader](../images/sentinel-leader.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/sentinel-leader.png)

# 选出新的主节点

在剩下的redis从节点中，按照以下顺序来选择新的主节点：

- 过滤掉“不健康”的数据节点：比如主观下线、断线的从节点、五秒内没有回复过哨兵节点ping命令的节点、与主节点失联的从节点。
- 选择slave-priority（从节点优先级）最高的从节点，如果存在则返回不存在则继续后面的流程。
- 选择复制偏移量最大的从节点，这意味着这个从节点上面的数据最完整，如果存在则返回不存在则继续后面的流程。
- 到了这里，所有剩余从节点的状态都是一样的，选择runid最小的从节点。

[![new-leader](../images/new-leader.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/new-leader.png)

# 提升新的主节点

选择了新的主节点之后，还需要最后的流程让该节点成为新的主节点：

- 哨兵领导者向上一步选出的从节点发出“slaveof no one”命令，让该节点成为主节点。
- 哨兵领导者向剩余的从节点发送命令，让它们成为新主节点的从节点。
- 哨兵节点集合会将原来的主节点更新为从节点，当其恢复之后命令它去复制新的主节点的数据。

[![new-master](../images/new-master.png)](https://www.codedump.info/media/imgs/20190409-redis-sentinel/new-master.png)