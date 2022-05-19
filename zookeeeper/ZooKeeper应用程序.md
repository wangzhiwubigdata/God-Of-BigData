
在对ZooKeeper有了一个深入的了解以后，我们来看一下用ZooKeeper可以实现哪些应用。

## 配置服务 Configuration Service

一个基本的ZooKeeper实现的服务就是“配置服务”，集群中的服务器可以通过ZooKeeper共享一个通用的配置数据。从表面上，ZooKeeper可以理解为一个配置数据的高可用存储服务，为应用提供检索和更新配置数据服务。我们可以使用ZooKeeper的观察模式实现一个活动的配置服务，当配置数据发生变化时，可以通知与配置相关客户端。

接下来，我们来实现一个这样的活动配置服务。首先，我们设计用znode来存储key-value对，我们在znode中存储一个String类型的数据作为value，用znode的path来表示key。然后，我们实现一个client，这个client可以在任何时候对数据进行跟新操作。那么这个设计的ZooKeeper数据模型应该是：master来更新数据，其他的worker也随之将数据更新，就像HDFS的namenode那样。

我们在一个叫做ActiveKeyValueStore的类中编写代码如下：

```
public class ActiveKeyValueStore extends ConnectionWatcher {

  private static final Charset CHARSET = Charset.forName("UTF-8");

  public void write(String path, String value) throws InterruptedException,
      KeeperException {
    Stat stat = zk.exists(path, false);
    if (stat == null) {
      zk.create(path, value.getBytes(CHARSET), Ids.OPEN_ACL_UNSAFE,
          CreateMode.PERSISTENT);
    } else {
      zk.setData(path, value.getBytes(CHARSET), -1);
    }
  }
}
```
write()方法主要实现将给定的key-value对写入到ZooKeeper中。这其中隐含了创建一个新的znode和更新一个已存在的znode的实现方法的不同。那么操作之前，我们需要根据exists()来判断znode是否存在，然后再根据情况进行相关的操作。其他值得一提的就是String类型的数据在转换成byte[]时，使用的字符集是UTF-8。

我们为了说明ActiveKeyValueStore怎么使用，我们考虑实现一个ConfigUpdater类来实现更新配置。下面代码实现了一个在一些随机时刻更新配置数据的应用。

```
public class ConfigUpdater {

  public static final String PATH = "/config";

  private ActiveKeyValueStore store;
  private Random random = new Random();

  public ConfigUpdater(String hosts) throws IOException, InterruptedException {
    store = new ActiveKeyValueStore();
    store.connect(hosts);
  }

  public void run() throws InterruptedException, KeeperException {
    while (true) {
      String value = random.nextInt(100) + "";
      store.write(PATH, value);
      System.out.printf("Set %s to %s\n", PATH, value);
      TimeUnit.SECONDS.sleep(random.nextInt(10));
    }
  }

  public static void main(String[] args) throws Exception {
    ConfigUpdater configUpdater = new ConfigUpdater(args[0]);
    configUpdater.run();
  }
}
```
上面的代码很简单。在ConfigUpdater的构造函数中，ActiveKeyValueStore对象连接到ZooKeeper服务。然后run()不断的循环运行，使用一个随机数不断的随机更新/configznode上的值。

下面我们来看一下，如何读取/config上的值。首先，我们在ActiveKeyValueStore中实现一个读方法。

```
public String read(String path, Watcher watcher) throws InterruptedException,
      KeeperException {
    byte[] data = zk.getData(path, watcher, null/*stat*/);
    return new String(data, CHARSET);
  }
```
ZooKeeper的getData()方法的参数包含：path，一个Watcher对象和一个Stat对象。Stat对象中含有从getData()返回的值，并且负责接收回调信息。这种方式下，调用者不仅可以获得数据，还能够获得znode的metadata。

做为服务的consumer，ConfigWatcher以观察者身份，创建一个ActiveKeyValueStore对象，并且在启动以后调用read()函数（在dispalayConfig()函数中）获得相关数据。

下面的代码实现了一个以观察模式获得ZooKeeper中的数据更新的应用，并将值到后台中。

```
public class ConfigWatcher implements Watcher {

  private ActiveKeyValueStore store;

  public ConfigWatcher(String hosts) throws IOException, InterruptedException {
    store = new ActiveKeyValueStore();
    store.connect(hosts);
  }

  public void displayConfig() throws InterruptedException, KeeperException {
    String value = store.read(ConfigUpdater.PATH, this);
    System.out.printf("Read %s as %s\n", ConfigUpdater.PATH, value);
  }

  @Override
  public void process(WatchedEvent event) {
    if (event.getType() == EventType.NodeDataChanged) {
      try {
        displayConfig();
      } catch (InterruptedException e) {
        System.err.println("Interrupted. Exiting.");        
        Thread.currentThread().interrupt();
      } catch (KeeperException e) {
        System.err.printf("KeeperException: %s. Exiting.\n", e);        
      }
    }
  }

  public static void main(String[] args) throws Exception {
    ConfigWatcher configWatcher = new ConfigWatcher(args[0]);
    configWatcher.displayConfig();

    // stay alive until process is killed or thread is interrupted
    Thread.sleep(Long.MAX_VALUE);
  }
}
```

当ConfigUpadater更新znode时，ZooKeeper将触发一个EventType.NodeDataChanged的事件给观察者。ConfigWatcher将在他的process()函数中获得这个时间，并将显示读取到的最新的版本的配置数据。

由于观察模式的触发是一次性的，所以每次都要调用ActiveKeyValueStore的read()方法，这样才能获得未来的更新数据。我们不能确保一定能够接受到更新通知事件，因为在接受观察事件和下一次读取之间的窗口期内，znode可能被改变了（有可能很多次），但是client可能没有注册观察模式，所以client不会接到znode改变的通知。在配置服务中这不是一个什么问题，因为client只关心配置数据的最新版本。然而，建议读者关注一下这个潜在的问题。

让我们来看一下控制台打印的ConfigUpdater运行结果：

```
% java ConfigUpdater localhost
Set /config to 79
Set /config to 14
Set /config to 78
```

然后立即在另外的控制台终端窗口中运行ConfigWatcher:

```
% java ConfigWatcher localhost
Read /config as 79
Read /config as 14
Read /config as 78
```

## 坚韧的ZooKeeper应用

分布式计算设计的第一谬误就是认为“网络是稳定的”。我们所实现的程序目前都是假设网络稳定的情况下实现的，所以当我们在一个真实的网络环境下，会有很多原因可以使程序执行失败。下面我们将阐述一些可能造成失败的场景，并且讲述如何正确的处理这些失败，让我们的程序在面对这些异常时更具韧性。
在ZooKeeper的API中，每一个ZooKeeper的操作都会声明抛出两个异常：InterruptedException和KeeperException。


### InterrupedException

当一个操作被中断时，会抛出一个InterruptedException。在JAVA中有一个标准的阻塞机制用来取消程序的执行，就是在需要阻塞的地方调用interrupt()。如果取消执行成功，会以抛出一个InterruptedException作为结果。ZooKeeper坚持了这个标准，所以我们可以用这种方式来取消client的对ZooKeeper的操作。用到ZooKeeper的类和库需要向上抛出InterruptedException，才能使我们的client实现取消操作。

InterruptedException并不意味着程序执行失败，可能是人为设计中断的，所以在上面配置应用的例子中，当向上抛出InterruptedException时，会引起应用终止。

### KeeperException

当ZooKeeper服务器出现错误信号，或者出现了通信方面的问题，就会抛出一个KeeperException。由于错误的不同原因，所以KeeperException有很多子类。例如，KeeperException.NoNodeException当操作一个znode时，而这个znode并不存在，就会抛出这个异常。

每一个之类都有一个异常码作为异常的类型。例如，KeeperException.NoNodeException的异常码就是KeeperException.Code.NONODE(一个枚举值)。
有两种方法来处理KeeperException。一种是直接捕获KeeperException，然后根据异常码进行不同类型异常处理。另一种是捕获具体的子类，然后根据不同类型的异常进行处理。

KeeperException包含了3大类异常。

### 状态异常 State Exception

当无法操作znode树造成操作失败时，会产生状态异常。通常引起状态异常的原因是有另外的程序在同时改变znode。例如，一个setData()操作时，会抛出KeeperException.BadVersionException。因为另外的一个程序已经在setData()操作之前修改了znode，造成setData()操作时版本号不匹配了。程序员必须了解，这种情况是很有可能发生的，我们必须靠编写处理这种异常的代码来解决他。

有的一些异常是编写代码时的疏忽造成的，例如KeeperException.NoChildrenForEphemeralsException。这个异常是当我们给一个enphemeral类型的znode添加子节点时抛出的。

### 重新获取异常 Recoverable Exception

重新获取异常来至于那些能够获得同一个ZooKeeper session的应用。伴随的表现是抛出KeeperException.ConnectionLossException，表示与ZooKeeper的连接丢失。ZooKeeper将会尝试重新连接，大多数情况下重新连接都会成功并且能够保证session的完整性。

然而，ZooKeeper无法通知客户端操作由于KeeperException.ConnectionLossException而失败。这就是一个部分失败的例子。只能依靠程序员编写代码来处理这个不确定性。

在这点上，幂等操作和非幂等操作的差别就会变得非常有用了。一个幂等操作是指无论运行一次还是多次结果都是一样的，例如一个读请求，或者一个不设置任何值得setData操作。这些操作可以不断的重试。

一个非幂等操作不能被不分青红皂白的不停尝试执行，就像一些操作执行一次的效率和执行多次的效率是不同。我们将在之后会讨论如何利用非幂等操作来处理Recovreable Exception。

### 不能重新获取异常 Unrecoverable exceptions

在一些情况下，ZooKeeper的session可能会变成不可用的——比如session过期，或者因为某些原因session被close掉（都会抛出KeeperException.SessionExpiredException），或者鉴权失败（KeeperException.AuthFailedException）。无论何种情况，ephemeral类型的znode上关联的session都会丢失，所以应用在重新连接到ZooKeeper之前都需要重新构建他的状态。

## 稳定的配置服务


回过头来看一下ActiveKeyValueStore中的write()方法，其中调用了exists()方法来判断znode是否存在，然后决定是创建一个znode还是调用setData来更新数据。

```
public void write(String path, String value) throws InterruptedException,
      KeeperException {
    Stat stat = zk.exists(path, false);
    if (stat == null) {
      zk.create(path, value.getBytes(CHARSET), Ids.OPEN_ACL_UNSAFE,
          CreateMode.PERSISTENT);
    } else {
      zk.setData(path, value.getBytes(CHARSET), -1);
    }
  }
  ```
  
  
从整体上来看，write()方法是一个幂等方法，所以我们可以不断的尝试执行它。我们来修改一个新版本的write()方法，实现在循环中不断的尝试write操作。我们为尝试操作设置了一个最大尝试次数参数（MAX_RETRIES）和每次尝试间隔的休眠(RETRY_PERIOD_SECONDS)时长：

```
public void write(String path, String value) throws InterruptedException,
      KeeperException {
    int retries = 0;
    while (true) {
      try {
        Stat stat = zk.exists(path, false);
        if (stat == null) {
          zk.create(path, value.getBytes(CHARSET), Ids.OPEN_ACL_UNSAFE,
              CreateMode.PERSISTENT);
        } else {
          zk.setData(path, value.getBytes(CHARSET), stat.getVersion());
        }
        return;
      } catch (KeeperException.SessionExpiredException e) {
        throw e;
      } catch (KeeperException e) {
        if (retries++ == MAX_RETRIES) {
          throw e;
        }
        // sleep then retry
        TimeUnit.SECONDS.sleep(RETRY_PERIOD_SECONDS);
      }
    }
  }
  ```
  
  
细心的读者可能会发现我们并没有在捕获KeeperException.SessionExpiredException时继续重新尝试操作，这是因为当session过期后，ZooKeeper会变为CLOSED状态，就不能再重新连接了。我们只是简单的抛出一个异常，通知调用者去创建一个新的ZooKeeper实例，所以write()方法可以不断的尝试执行。一个简单的方式来创建一个ZooKeeper实例就是重新new一个ConfigUpdater实例。

```
public static void main(String[] args) throws Exception {
    while (true) {
      try {
        ResilientConfigUpdater configUpdater =
          new ResilientConfigUpdater(args[0]);
        configUpdater.run();
      } catch (KeeperException.SessionExpiredException e) {
        // start a new session
      } catch (KeeperException e) {
        // already retried, so exit
        e.printStackTrace();
        break;
      }
    }
  }
  ```
  
  
另一个可以替代处理session过期的方法就是使用watcher来监控Expired的KeeperState，然后重新建立一个连接。这种方法下，我们只需要不断的尝试执行write()，如果我们得到了KeeperException.SessionExpiredException异常，连接最终也会被重新建立起来。那么我们抛开如何从一个过期的session中恢复问题，我们的重点是连接丢失的问题也可以这样解决，只是处理方法不同而已。

> 我们这里忽略了另外一种情况，在zookeeper实例不断的尝试连接了ensemble中的所有节点后发现都无法连接成功，就会抛出一个IOException，说明所有的集群节点都不可用。而有一些应用被设计为不断的尝试连接，直到ZooKeeper服务恢复可用为止。

## 锁服务

分布式锁用来为一组程序提供互斥机制。任意一个时刻仅有一个进程能够获得锁。分布式锁可以用来实现大型分布式系统的leader选举算法，即leader就是获取到锁的那个进程。

> 不要把ZooKeeper的原生leader选举算法和我们这里所说的通用leader选举服务搞混淆了。ZooKeeper的原生leader选举算法并不是公开的算法，并不能向我们这里所说的通用leader选举服务那样，为一个分布式系统提供主进程选举服务。

为了使用ZooKeeper实现分布式锁，我们使用可排序的znode来实现进程对锁的竞争。思路其实很简单：首先，我们需要一个表示锁的znode，获得锁的进程就表示被这把锁给锁定了（命名为，/leader）。然后，client为了获得锁，就需要在锁的znode下创建ephemeral类型的子znode。在任何时间点上，只有排序序号最小的znode的client获得锁，即被锁定。例如，如果两个client同时创建znode /leader/lock-1和/leader/lock-2，所以创建/leader/lock-1的client获得锁，因为他的排序序号最小。ZooKeeper服务被看作是排序的权威管理者，因为是由他来安排排序的序号的。
锁可能因为删除了/leader/lock-1znode而被简单的释放。另外，如果相应的客户端死掉，使用ephemeral znode的价值就在这里，znode可以被自动删除掉。创建/leader/lock-2的client就获得了锁，因为他的序号现在最小。当然客户端需要启动观察模式，在znode被删除时才能获得通知：此时他已经获得了锁。
获得锁的伪代码如下：

  1. 在lock的znode下创建名字为lock-的ephemeral类型znode，并记录下创建的znode的path（会在创建函数中返回）。
  2. 获取lock znode的子节点列表，并开启对lock的子节点的watch模式。
  3. 如果创建的子节点的序号最小，则再执行一次第2步，那么就表示已经获得锁了。退出。

等待第2步的观察模式的通知，如果获得通知，则再执行第2步。

###  羊群效应 

虽然这个算法是正确的，但是还是有一些问题。第一个问题是羊群效应。试想一下，当有成千成百的client正在试图获得锁。每一个client都对lock节点开启了观察模式，等待lock的子节点的变化通知。每次锁的释放和获取，观察模式将被触发，每个client都会得到消息。那么羊群效应就是指像这样，大量的client都会获得相同的事件通知，而只有很小的一部分client会对事件通知有响应。我们这里，只有一个client将获得锁，但是所有的client都得到了通知。那么这就像在网络公路上撒了把钉子，增加了ZooKeeper服务器的压力。

为了避免羊群效应，通知的范围需要更精准。我们通过观察发现，只有当序号排在当前znode之前一个znode离开时，才有必要通知创建当前znode的client，而不必在任意一个znode删除或者创建时都通知client。在我们的例子中，如果client1、client2和client3创建了znode/leader/lock-1、/leader/lock-2和leader/lock-3，client3仅在/leader/lock-2消失时，才获得通知。而不需要在/leader/lock-1消失时，或者新建/leader/lock-4时，获得通知。


### 重新获取异常 Recoverable Exception

这个锁算法的另一个问题是没有处理当连接中断造成的创建失败。在这种情况下，我们根本就不知道之前的创建是否成功了。创建一个可排序的znode是一个非等幂操作，所以我们不能简单重试，因为如果第一次我们创建成功了，那么第一次创建的znode就成了一个孤立的znode了，将永远不会被删除直到会话结束。

那么问题的关键在于，在重新连接以后，client不能确定是否之前创建过lock节点的子节点。我们在znode的名字中间嵌入一个client的ID，那么在重新连接后，就可以通过检查lock znode的子节点znode中是否有名字包含client ID的节点。如果有这样的节点，说明之前创建节点操作成功了，就不需要再创建了。如果没有这样的节点，那就重新创建一个。

Client的会话ID是一个长整型数据，并且在ZooKeeper中是唯一的。我们可以使用会话的ID在处理连接丢失事件过程中作为client的id。在ZooKeeper的JAVA API中，我们可以调用getSessionId()方法来获得会话的ID。

那么Ephemeral类型的可排序znode不要命名为lock-<sessionId>-，所以当加上序号后就变成了lock-<sessionId>-<sequenceNumber>。那么序号虽然针对上一级名字是唯一的，但是上一级名字本身就是唯一的，所以这个方法既可以标记znode的创建者，也可以实现创建的顺序排序。
  
### 不能恢复异常 Unrecoverable Exception

如果client的会话过期，那么他创建的ephemeral znode将被删除，client将立即失去锁（或者至少放弃获得锁的机会）。应用需要意识到他不再拥有锁，然后清理一切状态，重新创建一个锁对象，并尝试再次获得锁。注意，应用必须在得到通知的第一时间进行处理，因为应用不知道如何在znode被删除事后判断是否需要清理他的状态。


### 实现 Implementation

考虑到所有的失败模式的处理的繁琐，所以实现一个正确的分布式锁是需要做很多细微的设计工作。好在ZooKeeper为我们提供了一个 产品级质量保证的锁的实现，我们叫做WriteLock。我们可以轻松的在client中应用。

### 更多的分布式数据结构和协议

我们可以用ZooKeeper来构建很多分布式数据结构和协议，例如，barriers，queues和two-phase commit。有趣的是我们注意到这些都是同步协议，而我们却使用ZooKeeper的原生异步特征（比如通知机制）来构建他们。
在ZooKeeper官网上提供了一些数据结构和协议的伪代码。并且提供了实现这些的数据结构和协议的标准教程（包括locks、leader选举和队列）；你可以在recipes目录中找到。

Apache Curator project也提供了一些简单客户端的教程。










