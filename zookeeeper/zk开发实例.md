
## ZooKeeper中的组和成员

我们可以把Zookeeper理解为一个高可用的文件系统。但是它没有文件和文件夹的概念，只有一个叫做znode的节点概念。那么znode即是数据的容器，也是其他节点的容器。（其实znode就可以理解为文件或者是文件夹）我们用父节点和子节点的关系来表示组和成员的关系。那么一个节点代表一个组，组节点下的子节点代表组内的成员.
如下图所示：

![zk](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/zookeeeper/859b65a6868f6b56eadd77a226db5e03.jpeg)

## 创建组

我们使用zookeeper的Java API来创建一个/zoo的组节点：
```
public class CreateGroup implements Watcher {
 private static final int SESSION_TIMEOUT = 5000;
 private ZooKeeper zk;
 private CountDownLatch connectedSignal = new CountDownLatch(1);
 public void connect(String hosts) throws IOException, InterruptedException {
     zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
     connectedSignal.await();
 }

 @Override
 public void process(WatchedEvent event) { // Watcher interface
     if (event.getState() == KeeperState.SyncConnected) {
     connectedSignal.countDown();
     }
 }

 public void create(String groupName) throws KeeperException,
 InterruptedException {
     String path = "/" + groupName;
     String createdPath = zk.create(path, null/*data*/, Ids.OPEN_ACL_UNSAFE,
     CreateMode.PERSISTENT);
     System.out.println("Created " + createdPath);
 }

 public void close() throws InterruptedException {
     zk.close();
 }

 public static void main(String[] args) throws Exception {
     CreateGroup createGroup = new CreateGroup();
     createGroup.connect(args[0]);
     createGroup.create(args[1]);
     createGroup.close();
     }
 }
 ```
当main()执行时，首先创建了一个CreateGroup的对象，然后调用connect()方法，通过zookeeper的API与zookeeper服务器连接。创建连接我们需要3个参数：一是服务器端主机名称以及端口号，二是客户端连接服务器session的超时时间，三是Watcher接口的一个实例。Watcher实例负责接收Zookeeper数据变化时产生的事件回调。

在连接函数中创建了zookeeper的实例，然后建立与服务器的连接。建立连接函数会立即返回，所以我们需要等待连接建立成功后再进行其他的操作。我们使用CountDownLatch来阻塞当前线程，直到zookeeper准备就绪。这时，我们就看到Watcher的作用了。我们实现了Watcher接口的一个方法：
```
public void process(WatchedEvent event);
```
当客户端连接上了zookeeper服务器，Watcher将由process()函数接收一个连接成功的事件。我们接下来调用CountDownLatch，释放之前的阻塞。

连接成功后，我们调用create()方法。我们在这个方法中调用zookeeper实例的create()方法来创建一个znode。参数包括：一是znode的path；二是znode的内容（一个二进制数组），三是一个access control list(ACL，访问控制列表，这里使用完全开放模式)，最后是znode的性质。

znode的性质分为ephemeral和persistent两种。ephemeral性质的znode在创建他的客户端的会话结束，或者客户端以其他原因断开与服务器的连接时，会被自动删除。而persistent性质的znode就不会被自动删除，除非客户端主动删除，而且不一定是创建它的客户端可以删除它，其他客户端也可以删除它。这里我们创建一个persistent的znode。
create()将返回znode的path。我们讲新建znode的path打印出来。
我们执行如上程序：
```
% export CLASSPATH=ch21-zk/target/classes/:$ZOOKEEPER_HOME/*:\
$ZOOKEEPER_HOME/lib/*:$ZOOKEEPER_HOME/conf
% java CreateGroup localhost zoo
Created /zoo
```

### 加入组
接下来我们实现如何在一个组中注册成员。我们将使用ephemeral znode来创建这些成员节点。那么当客户端程序退出时，这些成员将被删除。
我们创建一个ConnetionWatcher类，然后继承实现一个JoinGroup类：

```
public class ConnectionWatcher implements Watcher {

 private static final int SESSION_TIMEOUT = 5000;

 protected ZooKeeper zk;

 private CountDownLatch connectedSignal = new CountDownLatch(1);

 public void connect(String hosts) throws IOException, InterruptedException {
     zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
     connectedSignal.await();
 }

 @Override
 public void process(WatchedEvent event) {
     if (event.getState() == KeeperState.SyncConnected) {
         connectedSignal.countDown();
     }
 }

 public void close() throws InterruptedException {
     zk.close();
 }
}
public class JoinGroup extends ConnectionWatcher {

 public void join(String groupName, String memberName) throws KeeperException,
 InterruptedException {
     String path = "/" + groupName + "/" + memberName;
     String createdPath = zk.create(path, null/*data*/, Ids.OPEN_ACL_UNSAFE,
     CreateMode.EPHEMERAL);
     System.out.println("Created " + createdPath);
 }

 public static void main(String[] args) throws Exception {
     JoinGroup joinGroup = new JoinGroup();
     joinGroup.connect(args[0]);
     joinGroup.join(args[1], args[2]);
     // stay alive until process is killed or thread is interrupted
     Thread.sleep(Long.MAX_VALUE);
 }
}
```

加入组与创建组非常相似。我们加入了一个ephemeral znode后，让线程阻塞住。然后我们可以使用命令行查看zookeeper中我们创建的znode。当我们将阻塞的程序强行关闭后，我们会发现我们创建的znode会自动消失。

### 成员列表

下面我们实现一个程序来列出一个组中的所有成员。

```
public class ListGroup extends ConnectionWatcher {

 public void list(String groupName) throws KeeperException,
 InterruptedException {
     String path = "/" + groupName;
     try {
         List<String> children = zk.getChildren(path, false);
         if (children.isEmpty()) {
             System.out.printf("No members in group %s\n", groupName);
             System.exit(1);
         }
         for (String child : children) {
             System.out.println(child);
         }
     } catch (KeeperException.NoNodeException e) {
         System.out.printf("Group %s does not exist\n", groupName);
         System.exit(1);
     }
 }

 public static void main(String[] args) throws Exception {
     ListGroup listGroup = new ListGroup();
     listGroup.connect(args[0]);
     listGroup.list(args[1]);
     listGroup.close();
 }
}

```

我们在list()方法中通过调用getChildren()方法来获得某一个path下的子节点，然后打印出来。我们这里会试着捕获KeeperException.NoNodeException，当znode不存在时会抛出这个异常。我们运行程序，会看见如下结果，说明我们还没在zoo组中添加任何成员几点：
```
% java ListGroup localhost zoo
No members in group zoo
```
我们可以运行之前的JoinGroup来添加成员。在后台运行一些JoinGroup程序，这些程序添加节点后都处于sleep状态：
```
% java JoinGroup localhost zoo duck &
% java JoinGroup localhost zoo cow &
% java JoinGroup localhost zoo goat &
% goat_pid=$!
```
最后一行命令的作用是将最后一个启动的java程序的pid记录下来，我们好在列出zoo下面的成员后，将该进程kill掉。
下面我们将zoo下的成员打印出来：
```
% java ListGroup localhost zoo
goat
duck
cow
然后我们将kill掉最后启动的JoinGroup客户端：
% kill $goat_pid
过几秒后，我们发现goat节点不见了。因为之前我们创建的goat节点是一个ephemeral节点，而创建这个节点的客户端在ZooKeeper上的会话已经被终结了，因为这个回话在5秒后失效了（我们设置了会话的超时时间为5秒）：
% java ListGroup localhost zoo
duck
cow
```

让我们回过头来看看，我们到底都做了一些什么？我们首先创建了一个节点组，这些节点的创建者都在同一个分布式系统中。这些节点的创建者之间互相都不知情。一个创建者想使用这些节点数据进行一些工作，例如通过znode节点是否存在来判断节点的创建者是否存在。

最后一点，我们不能只依靠组成员关系来完全解决在与节点通信时的网络错误。当与一个集群组成员节点进行通信时，发生了通信失败，我们需要使用重试或者试验与组中其他的节点通信，来解决这次通信失败。

### Zookeeper的命令行工具

Zookeeper有一套命令行工具。我们可以像如下使用，来查找zoo下的成员节点：

```
% zkCli.sh -server localhost ls /zoo
[cow, duck]
```

你可以不加参数运行这个工具，来获得帮助。

## 删除分组

下面让我们来看一下如何删除一个分组？
ZooKeeper的API提供一个delete()方法来删除一个znode。我们通过输入znode的path和版本号（version number）来删除想要删除的znode。我们除了使用path来定位我们要删除的znode，还需要一个参数是版本号。只有当我们指定要删除的本版号，与znode当前的版本号一致时，ZooKeeper才允许我们将znode删除掉。这是一种optimistic locking机制，用来处理znode的读写冲突。我们也可以忽略版本号一致检查，做法就是版本号赋值为-1。
删除一个znode之前，我们需要先删除它的子节点，就下如下代码中实现的那样：


```
public class DeleteGroup extends ConnectionWatcher {

 public void delete(String groupName) throws KeeperException,
 InterruptedException {
     String path = "/" + groupName;

     try {
         List<String> children = zk.getChildren(path, false);
         for (String child : children) {
             zk.delete(path + "/" + child, -1);
         }
         zk.delete(path, -1);
     } catch (KeeperException.NoNodeException e) {
         System.out.printf("Group %s does not exist\n", groupName);
         System.exit(1);
     }
 }

 public static void main(String[] args) throws Exception {
     DeleteGroup deleteGroup = new DeleteGroup();
     deleteGroup.connect(args[0]);
     deleteGroup.delete(args[1]);
     deleteGroup.close();
 }
}
```

最后我们执行如下操作来删除zoo group：

```
% java DeleteGroup localhost zoo
% java ListGroup localhost zoo
Group zoo does not exist
```




