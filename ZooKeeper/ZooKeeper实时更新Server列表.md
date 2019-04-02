## 【Zookeeper系列五】ZooKeeper 实时更新server列表 - 陶邦仁的个人空间 - 开源中国

\#1 场景描述# 在分布式应用中, 我们经常同时启动多个server, 调用方(client)选择其中之一发起请求.

`分布式应用必须考虑高可用性和可扩展性`: server的应用进程可能会崩溃, 或者server本身也可能会宕机. 当server不够时, 也有可能增加server的数量. `总而言之, server列表并非一成不变, 而是一直处于动态的增减中`.

那么client如何才能实时的更新server列表呢? 解决的方案很多, 本文将讲述利用ZooKeeper的解决方案.

\#2 思路# `启动server时, 在zookeeper的某个znode(假设为/sgroup)下创建一个子节点`. 所创建的子节点的类型应该为ephemeral, 这样一来, 如果server进程崩溃, 或者server宕机, 与zookeeper连接的session就结束了, 那么其所创建的子节点会被zookeeper自动删除. 当崩溃的server恢复后, 或者新增server时, 同样需要在/sgroup节点下创建新的子节点.

`对于client, 只需注册/sgroup子节点的监听`, 当/sgroup下的子节点增加或减少时, zookeeper会通知client, 此时client更新server列表.

\#3 实现AppServer# AppServer的逻辑非常简单, 只须在启动时, 在zookeeper的"/sgroup"节点下新增一个子节点即可.

**AppServer.java源码：**

```
package com.king.server;

import org.apache.zookeeper.*;

public class AppServer {

	private String groupNode = "sgroup";

	private String subNode = "sub";

	/**
	 * 连接zookeeper
	 *
	 * @param address server的地址
	 */
	public void connectZookeeper(String address) throws Exception {
		ZooKeeper zk = new ZooKeeper("localhost:2180,localhost:2181,localhost:2182", 5000, new Watcher() {
			public void process(WatchedEvent event) {
				// 不做处理
			}
		});
		// 在"/sgroup"下创建子节点
		// 子节点的类型设置为EPHEMERAL_SEQUENTIAL, 表明这是一个临时节点, 且在子节点的名称后面加上一串数字后缀
		// 将server的地址数据关联到新创建的子节点上
		String createdPath = zk.create("/" + groupNode + "/" + subNode, address.getBytes("utf-8"),
				ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
		System.out.println("create: " + createdPath);
	}

	/**
	 * server的工作逻辑写在这个方法中
	 * 此处不做任何处理, 只让server sleep
	 */
	public void handle() throws InterruptedException {
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		// 在参数中指定server的地址
		if (args.length == 0) {
			System.err.println("The first argument must be server address");
			System.exit(1);
		}

		AppServer as = new AppServer();
		as.connectZookeeper(args[0]);

		as.handle();
	}
}
```

\#4 实现AppClient# AppClient的逻辑比AppServer稍微复杂一些, 需要监听"/sgroup"下子节点的变化事件, 当事件发生时, 需要更新server列表.

`注册监听"/sgroup"下子节点的变化事件, 可在getChildren方法中完成. 当zookeeper回调监听器的process方法时, 判断该事件是否是"/sgroup"下子节点的变化事件, 如果是, 则调用更新逻辑, 并再次注册该事件的监听`.

**AppClient.java源码：**

```
package com.king.server;

import java.util.ArrayList;
import java.util.List;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

public class AppClient {

	private String groupNode = "sgroup";

	private ZooKeeper zk;

	private Stat stat = new Stat();

	private volatile List<String> serverList;

	/**
	 * 连接zookeeper
	 */
	public void connectZookeeper() throws Exception {
		zk = new ZooKeeper("localhost:2180,localhost:2181,localhost:2182", 5000, new Watcher() {
			public void process(WatchedEvent event) {
				// 如果发生了"/sgroup"节点下的子节点变化事件, 更新server列表, 并重新注册监听
				if (event.getType() == Event.EventType.NodeChildrenChanged
						&& ("/" + groupNode).equals(event.getPath())) {
					try {
						updateServerList();
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			}
		});

		updateServerList();
	}

	/**
	 * 更新server列表
	 */
	private void updateServerList() throws Exception {
		List<String> newServerList = new ArrayList<String>();

		// 获取并监听groupNode的子节点变化
		// watch参数为true, 表示监听子节点变化事件. 
		// 每次都需要重新注册监听, 因为一次注册, 只能监听一次事件, 如果还想继续保持监听, 必须重新注册
		List<String> subList = zk.getChildren("/" + groupNode, true);
		for (String subNode : subList) {
			// 获取每个子节点下关联的server地址
			byte[] data = zk.getData("/" + groupNode + "/" + subNode, false, stat);
			newServerList.add(new String(data, "utf-8"));
		}

		// 替换server列表
		serverList = newServerList;

		System.out.println("server list updated: " + serverList);
	}

	/**
	 * client的工作逻辑写在这个方法中
	 * 此处不做任何处理, 只让client sleep
	 */
	public void handle() throws InterruptedException {
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		AppClient ac = new AppClient();
		ac.connectZookeeper();

		ac.handle();
	}
}
```