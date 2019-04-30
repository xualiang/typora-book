ZooKeeper提供了Java和C的binding. 本文关注Java相关的API.

# 1. 准备工作

拷贝ZooKeeper安装目录下的zookeeper.x.x.x.jar文件到项目的classpath路径下.

# 2. 创建连接和回调接口

首先需要创建ZooKeeper对象, 后续的一切操作都是基于该对象进行的.

```
ZooKeeper(String connectString, int sessionTimeout, Watcher watcher) throws IOException
```

**以下为各个参数的详细说明：**

1. `connectString`：zookeeper server列表, 以逗号隔开. ZooKeeper对象初始化后, `将从server列表中选择一个server, 并尝试与其建立连接`. `如果连接建立失败, 则会从列表的剩余项中选择一个server, 并再次尝试建立连接`.
2. `sessionTimeout`：指定连接的超时时间.
3. `watcher`：事件回调接口.

注意, 创建ZooKeeper对象时, 只要对象完成初始化便立刻返回. `建立连接是以异步的形式进行的`, 当连接成功建立后, 会回调watcher的process方法. `如果想要同步建立与server的连接`, 需要自己进一步封装.

```
public class ZKConnection {
    /**
     * server列表, 以逗号分割
     */
    protected String hosts = "localhost:4180,localhost:4181,localhost:4182";
    /**
     * 连接的超时时间, 毫秒
     */
    private static final int SESSION_TIMEOUT = 5000;
    private CountDownLatch connectedSignal = new CountDownLatch(1);
    protected ZooKeeper zk;

    /**
     * 连接zookeeper server
     */
    public void connect() throws Exception {
        zk = new ZooKeeper(hosts, SESSION_TIMEOUT, new ConnWatcher());
        // 等待连接完成
        connectedSignal.await();
    }

    public class ConnWatcher implements Watcher {
        public void process(WatchedEvent event) {
            // 连接建立, 回调process接口时, 其event.getState()为KeeperState.SyncConnected
            if (event.getState() == KeeperState.SyncConnected) {
                // 放开闸门, wait在connect方法上的线程将被唤醒
                connectedSignal.countDown();
            }
        }
    }
}
```

# 3. 创建znode

ZooKeeper对象的create方法用于创建znode.

```
String create(String path, byte[] data, List acl, CreateMode createMode);
```

**以下为各个参数的详细说明：**

1. `path`：znode的路径.
2. `data`：与znode关联的数据.
3. `acl`：指定权限信息, 如果不想指定权限, 可以传入Ids.OPEN_ACL_UNSAFE.
4. `createMode`：指定znode类型. CreateMode是一个枚举类, 从中选择一个成员传入即可；

```
/**
 * 创建临时节点
 */
public void create(String nodePath, byte[] data) throws Exception {
    zk.create(nodePath, data, Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
}
```

# 4 获取子node列表

ZooKeeper对象的getChildren方法用于获取子node列表.

```
List getChildren(String path, boolean watch);
```

watch参数用于`指定是否监听path node的子node的增加和删除事件`, 以及`path node本身的删除事件`.

# 5 判断znode是否存在

ZooKeeper对象的exists方法用于判断指定znode是否存在.

```
Stat exists(String path, boolean watch);
```

watch参数用于`指定是否监听path node的创建, 删除事件, 以及数据更新事件`. 如果该node存在, 则返回该node的状态信息, 否则返回null.

# 6 获取node中关联的数据

ZooKeeper对象的getData方法用于获取node关联的数据.

```
byte[] getData(String path, boolean watch, Stat stat);
```

1. watch参数用于指定`是否监听path node的删除事件, 以及数据更新事件`, `注意, 不监听path node的创建事件`, 因为如果path node不存在, 该方法将抛出KeeperException.NoNodeException异常.
2. stat参数是个传出参数, `getData方法会将path node的状态信息设置到该参数中`.

# 7 更新node中关联的数据

ZooKeeper对象的setData方法用于更新node关联的数据.

```
Stat setData(final String path, byte data[], int version);
```

1. data为待更新的数据.
2. version参数指定要更新的数据的版本, `如果version和真实的版本不同, 更新操作将失败. 指定version为-1则忽略版本检查`.
3. 返回path node的状态信息.

# 8 删除znode

ZooKeeper对象的delete方法用于删除znode.

```
void delete(final String path, int version);
```

# 9 其余接口

请查看ZooKeeper对象的API文档.

#10 代码实例

```
public class JavaApiSample implements Watcher {

	private static final int SESSION_TIMEOUT = 10000;
	//    private static final String CONNECTION_STRING = "test.zookeeper.connection_string:2181";
	private static final String CONNECTION_STRING = "127.0.0.1:2180,127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183";
	private static final String ZK_PATH = "/nileader";
	private ZooKeeper zk = null;

	private CountDownLatch connectedSemaphore = new CountDownLatch(1);

	/**
	 * 创建ZK连接
	 *
	 * @param connectString  ZK服务器地址列表
	 * @param sessionTimeout Session超时时间
	 */
	public void createConnection(String connectString, int sessionTimeout) {
		this.releaseConnection();
		try {
			zk = new ZooKeeper(connectString, sessionTimeout, this);
			connectedSemaphore.await();
		} catch (InterruptedException e) {
			System.out.println("连接创建失败，发生 InterruptedException");
			e.printStackTrace();
		} catch (IOException e) {
			System.out.println("连接创建失败，发生 IOException");
			e.printStackTrace();
		}
	}

	/**
	 * 关闭ZK连接
	 */
	public void releaseConnection() {
		if (null != this.zk) {
			try {
				this.zk.close();
			} catch (InterruptedException e) {
				// ignore
				e.printStackTrace();
			}
		}
	}

	/**
	 * 创建节点
	 *
	 * @param path 节点path
	 * @param data 初始数据内容
	 * @return
	 */
	public boolean createPath(String path, String data) {
		try {
			System.out.println("节点创建成功, Path: "
					+ this.zk.create(path, //
					data.getBytes(), //
					ZooDefs.Ids.OPEN_ACL_UNSAFE, //
					CreateMode.EPHEMERAL)
					+ ", content: " + data);
		} catch (KeeperException e) {
			System.out.println("节点创建失败，发生KeeperException");
			e.printStackTrace();
		} catch (InterruptedException e) {
			System.out.println("节点创建失败，发生 InterruptedException");
			e.printStackTrace();
		}
		return true;
	}

	/**
	 * 读取指定节点数据内容
	 *
	 * @param path 节点path
	 * @return
	 */
	public String readData(String path) {
		try {
			System.out.println("获取数据成功，path：" + path);
			return new String(this.zk.getData(path, false, null));
		} catch (KeeperException e) {
			System.out.println("读取数据失败，发生KeeperException，path: " + path);
			e.printStackTrace();
			return "";
		} catch (InterruptedException e) {
			System.out.println("读取数据失败，发生 InterruptedException，path: " + path);
			e.printStackTrace();
			return "";
		}
	}

	/**
	 * 更新指定节点数据内容
	 *
	 * @param path 节点path
	 * @param data 数据内容
	 * @return
	 */
	public boolean writeData(String path, String data) {
		try {
			System.out.println("更新数据成功，path：" + path + ", stat: " +
					this.zk.setData(path, data.getBytes(), -1));
		} catch (KeeperException e) {
			System.out.println("更新数据失败，发生KeeperException，path: " + path);
			e.printStackTrace();
		} catch (InterruptedException e) {
			System.out.println("更新数据失败，发生 InterruptedException，path: " + path);
			e.printStackTrace();
		}
		return false;
	}

	/**
	 * 删除指定节点
	 *
	 * @param path 节点path
	 */
	public void deleteNode(String path) {
		try {
			this.zk.delete(path, -1);
			System.out.println("删除节点成功，path：" + path);
		} catch (KeeperException e) {
			System.out.println("删除节点失败，发生KeeperException，path: " + path);
			e.printStackTrace();
		} catch (InterruptedException e) {
			System.out.println("删除节点失败，发生 InterruptedException，path: " + path);
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {

		JavaApiSample sample = new JavaApiSample();
		sample.createConnection(CONNECTION_STRING, SESSION_TIMEOUT);
		if (sample.createPath(ZK_PATH, "我是节点初始内容")) {
			System.out.println();
			System.out.println("数据内容: " + sample.readData(ZK_PATH) + "\n");
			sample.writeData(ZK_PATH, "更新后的数据");
			System.out.println("数据内容: " + sample.readData(ZK_PATH) + "\n");
			sample.deleteNode(ZK_PATH);
		}

		sample.releaseConnection();
	}

	/**
	 * 收到来自Server的Watcher通知后的处理。
	 */
	@Override
	public void process(WatchedEvent event) {
		System.out.println("收到事件通知：" + event.getState() + "\n");
		if (Event.KeeperState.SyncConnected == event.getState()) {
			connectedSemaphore.countDown();
		}

	}

}
```

#11 需要注意的几个地方

1. znode中关联的数据`不能超过1M`. `zookeeper的使命是分布式协作, 而不是数据存储`.
2. getChildren, getData, exists方法可指定`是否监听相应的事件`. 而create, delete, setData方法则会`触发相应的事件的发生`.
3. 以上介绍的几个方法`大多存在其异步的重载方法`, 具体请查看API说明.