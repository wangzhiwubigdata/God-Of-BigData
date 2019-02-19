
## Zookeeper 服务

ZooKeeper 是一个高可用的高性能调度服务。这一节我们将讲述他的模型、操作和接口。

## 数据模型 Data Model

ZooKeeper包含一个树形的数据模型，我们叫做znode。一个znode中包含了存储的数据和ACL（Access Control List）。ZooKeeper的设计适合存储少量的数据，并不适合存储大量数据，所以znode的存储限制最大不超过1M。

数据的访问被定义成原子性的。什么是原子性呢？一个客户端访问一个znode时，不会只得到一部分数据；客户端访问数据要么获得全部数据，要么读取失败，什么也得不到。相似的，写操作时，要么写入全部数据，要么写入失败，什么也写不进去。ZooKeeper能够保证写操作只有两个结果，成功和失败。绝对不会出现只写入了一部分数据的情况。与HDFS不同，ZooKeeper不支持字符的append（连接）操作。原因是HDFS是被设计成支持数据流访问（streaming data access）的大数据存储，而ZooKeeper则不是。

我们可以通过path来定位znode，就像Unix系统定位文件一样，使用斜杠来表示路径。但是，znode的路径只能使用绝对路径，而不能想Unix系统一样使用相对路径，即Zookeeper不能识别../和./这样的路径。

节点的名称是由Unicode字符组成的，除了zookeeper这个字符串，我们可以任意命名节点。为什么不能使用zookeeper命名节点呢？因为ZooKeeper已经默认使用zookeeper来命名了一个根节点，用来存储一些管理数据。

请注意，这里的path并不是URIs，在Java API中是一个String类型的变量。


### Ephemeral znodes

我们已经知道，znode有两种类型：ephemeral和persistent。在创建znode时，我们指定znode的类型，并且在之后不会再被修改。当创建znode的客户端的session结束后，ephemeral类型的znode将被删除。persistent类型的znode在创建以后，就与客户端没什么联系了，除非主动去删除它，否则他会一直存在。Ephemeral znode没有任何子节点。

虽然Ephemeral znode绑定了客户端session，但是对任何其他客户端都是可见的，当然是在他们的ACL策略下允许访问的情况下。
当我们在创建分布式系统时，需要知道分布式资源是否可用。Ephemeral znode就是为这种场景应运而生的。正如我们之前讲述的例子中，使用Ephemeral znode来实现一个成员关系管理，任何一个客户端进程任何时候都可以知道其他成员是否可用。
Znode的序号

如果在创建znode时，我们使用排序标志的话，ZooKeeper会在我们指定的znode名字后面增加一个数字。我们继续加入相同名字的znode时，这个数字会不断增加。这个序号的计数器是由这些排序znode的父节点来维护的。

如果我们请求创建一个znode，指定命名为/a/b-，那么ZooKeeper会为我们创建一个名字为/a/b-3的znode。我们再请求创建一个名字为/a/b-的znode，ZooKeeper会为我们创建一个名字/a/b-5的znode。ZooKeeper给我们指定的序号是不断增长的。Java API中的create()的返回结果就是znode的实际名字。

那么序号用来干什么呢？当然是用来排序用的！后面《A Lock Service》中我们将讲述如何使用znode的序号来构建一个share lock。

### 观察模式 Watches

观察模式可以使客户端在某一个znode发生变化时得到通知。观察模式有ZooKeeper服务的某些操作启动，并由其他的一些操作来触发。例如，一个客户端对一个znode进行了exists操作，来判断目标znode是否存在，同时在znode上开启了观察模式。如果znode不存在，这exists将返回false。如果稍后，另外一个客户端创建了这个znode，观察模式将被触发，将znode的创建事件通知之前开启观察模式的客户端。我们将在以后详细介绍其他的操作和触发。

观察模式只能被触发一次。如果要一直获得znode的创建和删除的通知，那么就需要不断的在znode上开启观察模式。在上面的例子中，如果客户端还继续需要获得znode被删除的通知，那么在获得创建通知后，客户端还需要继续对这个znode进行exists操作，再开启一次观察模式。

在《A Configuration Service》中，有一个例子将讲述如何使用观察模式在集群中更新配置。


## 操作 Operations

下面的表格中列出了9种ZooKeeper的操作。

操作	说明
create	 Creates a znode (the parent znode must already exist)
delete	 Deletes a znode (the znode must not have any children)
exists	 Tests whether a znode exists and retrieves its metadata
getACL, setACL	 Gets/sets the ACL for a znode
getChildren	 Gets a list of the children of a znode
getData,setData	 Gets/sets the data associated with a znode
sync	Synchronizes a client’s view of a znode with ZooKeeper


调用delete和setData操作时，我们必须指定一个znode版本号（version number），即我们必须指定我们要删除或者更新znode数据的哪个版本。如果版本号不匹配，操作将会失败。失败的原因可能是在我们提交之前，该znode已经被修改过了，版本号发生了增量变化。那么我们该怎么办呢？我可以考虑重试，或者调用其他的操作。例如，我们提交更新失败后，可以重新获取znode当前的数据，看看当前的版本号是什么，再做更新操作。

ZooKeeper虽然可以被看作是一个文件系统，但是由于ZooKeeper文件很小，所以没有提供像一般文件系统所提供的open、close或者seek操作。

> 注意
> 这里的sync操作与POSIX文件系统的fsync()操作是不同的。就像我们早前讲过的，ZooKeeper的写操作是原子性的，一个成功的写操作只保证数据被持久化到大多数ZooKeeper的服务器存储上。所以读操作可能会读取不到最新状态的数据，sync操作用来让client强制所访问的ZooKeeper服务器上的数据状态更新到最新状态。我们会在《一致性 Consistentcy》一节中详细介绍。

### 批量更新 Multiupdate
ZooKeeper支持将一些原始的操作组合成一个操作单元，然后执行这些操作。那么这种批量操作也是具有原子性的，只可能有两种执行结果，成功和失败。批量操作单元中的操作，不会出现一些操作执行成功，一些操作执行失败的情况，即要么都成功，要么都失败。

Multiupdate对于绑定一些结构化的全局变量很有用处。例如绑定一个无向图（undirected graph）。无向图的顶点（vertex）由znode来表示。添加和删除边（edge）的操作，由修改边的两个关联znode来实现。如果我们使用ZooKeeper的原始的操作来实现对边（edge）的操作，那么就有可能产生两个znode修改不一致的情况（一个修改成功，一个修改失败）。那么我们将修改两个znode的操作放入到一个Multi修改单元中，就能够保证两个znode，要么都修改成功，要么都修改失败。这样就能够避免修改无向图的边时产生修改不一致的现象。

### APIs
ZooKeeper客户端使用的核心编程语言有JAVA和C；同时也支持Perl、Python和REST。执行操作的方式呢，分为同步执行和异步执行。我们之前已经见识过了同步的Java API中的exists。

```
public Stat exists(String path, Watcher watcher) throws KeeperException,
 InterruptedException
 ```
 
下面代码则是异步方式的exists:

```
public void exists(String path, Watcher watcher, StatCallback cb, Object ctx)
```
Java API中，异步的方法的返回类型都是void，而操作的返回的结果将传递到回调对象的回调函数中。回调对象将实现StatCallback接口中的一个回调函数，来接收操作返回的结果。函数接口如下：

```
public void processResult(int rc, String path, Object ctx, Stat stat);
```

参数rc表示返回码，请参考KeeperException中的定义。在stat参数为null的情况下，非0的值表示一种异常。参数path和ctx与客户端调用的exists方法中的参数相等，这两个参数通常用来确定回调中获得的响应是来至于哪个请求的。参数ctx可以是任意对象，只有当path参数不能消灭请求的歧义时才会用到。如果不需要参数ctx，可以设置为null。

> 应该使用同步API还是异步API呢?
> 两种API提供了相同的功能，需要使用哪种API取决于你程序的模式。例如，你设计的程序模式是一个事件驱动模式的程序，那么你最好使用异步API。异步API也可以被用在追求一个比较好的数据吞吐量的场景。想象一下，如果你需要得去大量的znode数据，并且依靠独立的进程来处理他们。如果使用同步API,每次读取操作都会被阻塞住，直到返回结果。不如使用异步API，读取操作可以不必等待返回结果，继续执行。而使用另外的线程来处理返回结果。

### 观察模式触发器 Watch triggers

读操作，例如：exists、getChildren、getData会在znode上开启观察模式，并且写操作会触发观察模式事件，例如：create、delete和setData。ACL(Access Control List)操作不会启动观察模式。观察模式被触发时，会生成一个事件，这个事件的类型取决于触发他的操作：
 1, exists启动的观察模式，由创建znode，删除znode和更新znode操作来触发。
 2,getData启动的观察模式，由删除znode和更新znode操作触发。创建znode不会触发，是因为getData操作成功的前提是znode必须已经存在。
 3,getChildren启动的观察模式，由子节点创建和删除，或者本节点被删除时才会被触发。我们可以通过事件的类型来判断是本节点被删除还是子节点被删除：NodeChildrenChanged表示子节点被删除，而NodeDeleted表示本节点删除。

事件包含了触发事件的znode的path，所以我们通过NodeCreated和NodeDeleted事件就可以知道哪个znode被创建了或者删除了。如果我们需要在NodeChildrenChanged事件发生后知道哪个子节点被改变了，我们就需要再调用一次getChildren来获得一个新的子节点列表。与之类似，在NodeDataChanged事件发生后，我们需要调用getData来获得新的数据。我们在编写程序时，会在接收到事件通知后改变znode的状态，所以我们一定要清楚的记住znode的状态变化。

### ACLs 访问控制操作
znode的创建时，我们会给他一个ACL（Access Control List），来决定谁可以对znode做哪些操作。
ZooKeeper通过鉴权来获得客户端的身份，然后通过ACL来控制客户端的访问。鉴权方式有如下几种：
 * digest
使用用户名和密码方式

 * sasl
使用Kerberos鉴权

 * ip
使用客户端的IP来鉴权

客户端可以在与ZooKeeper建立会话连接后，自己给自己授权。授权是并不是必须的，虽然znode的ACL要求客户端必须是身份合法的，在这种情况下，客户端可以自己授权来访问znode。下面的例子，客户端使用用户名和密码为自己授权：

```
 zk.addAuthInfo("digest", "tom:secret".getBytes());
 ```
 
ACL是由鉴权方式、鉴权方式的ID和一个许可（permession）的集合组成。例如，我们想通过一个ip地址为10.0.0.1的客户端访问一个znode。那么，我们需要为znode设置一个ACL，鉴权方式使用IP鉴权方式，鉴权方式的ID为10.0.0.1，只允许读权限。使用JAVA我们将像如下方式创建一个ACL对象：

```
new ACL(Perms.READ,new Id("ip", "10.0.0.1"));
```

所有的许可权限将在下表中列出。请注意，exists操作不受ACL的控制，所以任何一个客户端都可以通过exists操作来获得任何znode的状态，从而得知znode是否真的存在。

在ZooDefs.Ids类中，有一些ACL的预定义变量，包括OPEN_ACL_UNSAFE，这个设置表示将赋予所有的许可给客户端（除了ADMIN的许可）。

另外，我们可以使用ZooKeeper鉴权的插件机制，来整合第三方的鉴权系统。


## 实现 Implementation

ZooKeeper服务可以在两种模式下运行。在standalone模式下，我们可以运行一个单独的ZooKeeper服务器，我们可以在这种模式下进行基本功能的简单测试，但是这种模式没有办法体现ZooKeeper的高可用特性和快速恢复特性。在生产环境中，我们一般采用replicated（复制）模式安装在多台服务器上，组建一个叫做ensemble的集群。ZooKeeper在他的副本之间实现高可用性，并且只要ensemble集群中能够推举出主服务器，ZooKeeper的服务就可以一直不终断。例如，在一个5个节点的ensemble中，容忍有2个节点脱离集群，服务还是可用的。因为剩下的3个节点投票，可以产生超过集群半数的投票，来推选一台主服务器。而6个节点的ensemble中，也只能容忍2个节点的服务器死机。因为如果3个节点脱离集群，那么剩下的3个节点无论如何不能产生超过集群半数的投票来推选一个主服务器。所以，一般情况下ensemble中的服务器数量都是奇数。

从概念上来看，ZooKeeper其实是很简单的。他所做的一切就是保证每一次对znode树的修改，都能够复制到ensemble的大多数服务器上。如果非主服务器脱离集群，那么至少有一台服务器上的副本保存了最新状态。剩下的其他的服务器上的副本，会很快更新这个最新的状态。

为了实现这个简单而不平凡的设计思路，ZooKeeper使用了一个叫做Zab的协议。这个协议分为两阶段，并且不断的运行在ZooKeeper上：

阶段 1：领导选举（Leader election）
Ensemble中的成员通过一个程序来选举出一个首领成员，我们叫做leader。其他的成员就叫做follower。在大多数（quorum）follower完成与leader状态同步时，这个阶段才结束。

阶段 2： 原子广播（Atomic broadcast）
所有的写入请求都会发送给leader，leader在广播给follower。当大多数的follower已经完成了数据改变，leader才会将更新提交，客户端就会随之得到leader更新成功的消息。协议中的设计也是具有原子性的，所以写入操作只有成功和失败两个结果。

如果leader脱离了集群，剩下的节点将选举一个新的leader。如果之前的leader回到了集群中，那么将被视作一个follower。leader的选举很快，大概200ms就能够产生结果，所以不会影响执行效率。
Ensemble中的所有节点都会在更新内存中的znode树的副本之前，先将更新数据写入到硬盘上。读操作可以请求任何一台ZooKeeper服务器，而且读取速度很快，因为读取是内存中的数据副本。

## 数据一致性 Consistency

理解了ZooKeeper的实现原理，有助于理解ZooKeeper如何保证数据的一致性。就像字面上理解的“leader”和“follower”的意思一样，在ensemble中follower的update操作会滞后于leader的update完成。事实的结果使我们在提交更新数据之前，不必在每一台ZooKeeper服务器上执行持久化变更数据，而是仅需在主服务器上执行持久化变更数据。ZooKeeper客户端的最佳实践是全部链接到follower上。然而客户端是有可能连接到leader上的，并且客户端控制不了这个选择，甚至客户端并不知道连接到了follower还是leader。下图所示，读操作向follower请求即可，而写操作由leader来提交。

每一个对znode树的更新操作，都会被赋予一个全局唯一的ID，我们称之为zxid（ZooKeeper Transaction ID）。更新操作的ID按照发生的时间顺序升序排序。例如，例如z1小于z2，那么z1的操作就早于z2的操作。

ZooKeeper在数据一致性上实现了如下几个方面：

顺序一致性
从客户端提交的更新操作是按照先后循序排序的。例如，如果一个客户端将一个znode z赋值为a，然后又将z的值改变成b，那么在这个过程中不会有客户端在z的值变为b后，取到的值是a。

原子性
更新操作的结果不是失败就是成功。即，如果更新操作失败，其他的客户端是不会知道的。

系统视图唯一性
无论客户端连接到哪个服务器，都将看见唯一的系统视图。如果客户端在同一个会话中去连接一个新的服务器，那么他所看见的视图的状态不会比之前服务器上看见的更旧。当ensemble中的一个服务器宕机，客户端去尝试连接另外一台服务器时，如果这台服务器的状态旧于之前宕机的服务器，那么服务器将不会接受客户端的连接请求，直到服务器的状态赶上之前宕机的服务器为止。

持久性
一旦更新操作成功，数据将被持久化到服务器上，并且不能撤销。所以服务器宕机重启，也不会影响数据。
时效性

系统视图的状态更新的延迟时间是有一个上限的，最多不过几十秒。如果服务器的状态落后于其他服务器太多，ZooKeeper会宁可关闭这个服务器上的服务，强制客户端去连接一个状态更新的服务器。

从执行效率上考虑，读操作的目标是内存中的缓存数据，并且读操作不会参与到写操作的全局排序中。这就会引起客户端在读取ZooKeeper的状态时产生不一致。例如，A客户端将znode z的值由a改变成a1，然后通知客户端B去读取z的值，但是B读取到的值是a，而不是修改后的a1，为了阻止这种情况出现，B在读取z的值之前，需要调用sync方法。sync方法会强制B连接的服务器状态与leader的状态同步，这样B在读取z的值就是A重新更改过的值了。

> sync操作只在异步调用时才可用，原因是你不需要等待操作结束再去执行其他的操作。因此，ZooKeeper保证所有的子操作都会在sync结束后再执行，甚至在sync操作之前发出的操作请求也不例外。


## 会话 Sessions

ZooKeeper的客户端中，配置了一个ensemble服务器列表。当启动时，首先去尝试连接其中一个服务器。如果尝试连接失败，那么会继续尝试连接下一个服务器，直到连接成功或者全部尝试连接失败。

一旦连接成功，服务器就会为客户端创建一个会话（session）。session的过期时间由创建会话的客户端应用来设定，如果在这个时间期间，服务器没有收到客户端的任何请求，那么session将被视为过期，并且这个session不能被重新创建，而创建的ephemeral znode将随着session过期被删除掉。在会话长期存在的情况下，session的过期事件是比较少见的，但是应用程序如何处理好这个事件是很重要的。（我们将在《The Resilient ZooKeeper Application》中详细介绍）
在长时间的空闲情况下，客户端会不断的发送ping请求来保持session。（ZooKeeper的客户端开发工具的liberay实现了自动发送ping请求，所以我们不必去考虑如何维持session）ping请求的间隔被设置成足够短，以便能够及时发现服务器失败（由读操作的超时时长来设置），并且能够及时的在session过期前连接到其他服务器上。
容错连接到其他服务器上，是由ZooKeeper客户端自动完成的。重要的是在连接到其他服务器上后，之前的session以及epemeral节点还保持可用状态。
在容错的过程中，应用将收到与服务断开连接和连接的通知。Watch模式的通知在断开链接时，是不会发送断开连接事件给客户端的，断开连接事件是在重新连接成功后发送给客户端的。如果在重新连接到其他节点时，应用尝试一个操作，这个操作是一定会失败的。对于这一点的处理，是一个ZooKeeper应用的重点。

## 时间 Time

在ZooKeeper中有一些时间的参数。tick是ZooKeeper的基础时间单位，用来定义ensemble中服务器上运行的程序的时间表。其他时间相关的配置都是以tick为单位的，或者以tick的值为最大值或者最小值。例如，session的过期时间在2 ticks到20 ticks之间，那么你再设置时选择的session过期时间必须在2和20之间的一个数。

通常情况1 tick等于2秒。那么就是说session的过期时间的设置范围在4秒到40秒之间。在session过期时间的设置上有一些考虑。过期时间太短会造成加快物理失败的监测频率。在组成员关系的例子中，session的过期时间与从组中移除失败的成员花费的时间相等。如果设置过低的session过期时间，那么网络延迟就有可能造成非预期的session过期。这种情况下，就会出现在短时间内一台机器不断的离开组，然后又从新加入组中。

如果应用需要创建比较复杂的临时状态，那么就需要较长的session过期时间，因为重构花费的时间比较长。有一些情况下，需要在session的生命周期内重启，而且要保证重启完后session不过期（例如，应用维护和升级的情况）。服务器会给每一个session一个ID和密码，如果在连接创建时，ZooKeeper验证通过，那么session将被恢复使用（只要session没过期就行）。所以应用程序可以实现一个优雅的关机动作，在重启之前，将session的ID和密码存储在一个稳定的地方。重启之后，通过ID和密码恢复session。

这仅仅是在一些特殊的情况下，我们需要使用这个特性来使用比较长的session过期时间。大多数情况下，我们还是要考虑当出现非预期的异常失败时，如何处理session过期，或者仅需要优雅的关闭应用，在session过期前不用重启应用。

通常情况也越大规模的ensemble，就需要越长的session过期时间。Connetction Timeout、Read Timeout和Ping Periods都由一个以服务器数量为参数的函数计算得到，当ensemble的规模扩大，这些值需要逐渐减小。如果为了解决经常失去连接而需要增加timeout的时长，建议你先监控一下ZooKeeper的metrics，再去调整。


## 状态 States

ZooKeeper对象在他的生命周期内会有不同的状态，我们通过getState()来获得当前的状态。

```
public States getState()
```

状态是一个枚举类型的数据。新构建的ZooKeeper对象在尝试连接ZooKeeper服务时的状态是CONNECTING，一旦与服务建立了连接那么状态就变成了CONNECTED。

客户端可以通过注册一个观察者对象来接收ZooKeeper对象状态的迁移。当通过CONNECTED状态后，观察者将接收到一个WatchedEvent事件，他的属性KeeperState的值是SyncConnected。

> 观察者有两个职能：一是接收ZooKeeper的状态改变通知；二是接收znode的改变通知。ZooKeeper对象构造时传递进去的watcher对象，默认是用来接收状态改变通知的，但是znode的改变通知也可能会共享使用默认的watcher对象，或者使用一个专用的watcher。我们可以通过一个Boolean变量来指定是否使用共享默认watcher。

ZooKeeper实例会与服务连接断开或者重新连接，状态会在CONNECTING和CONNECTED之间转换。如果连接断开，watcher会收到一个断开连接事件。请注意，这两个状态都是ZooKeeper实例自己初始化的，并且在断开连接后会自动进行重连接。

如果调用了close()或者session过期，ZooKeeper实例会转换为第三个状态CLOSED，此时在接受事件的KeeperState属性值为Expired。一旦ZooKeeper的状态变为CLOSED，说明实例已经不可用（可以通过isAlive()来判断），并且不能再被使用。如果要重新建立连接，就需要重新构建一个ZooKeeper实例。








