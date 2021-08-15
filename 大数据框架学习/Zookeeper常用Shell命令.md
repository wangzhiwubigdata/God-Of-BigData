# Zookeeper常用Shell命令

<nav>
<a href="#一节点增删改查">一、节点增删改查</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11-启动服务和连接服务">1.1 启动服务和连接服务</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-help命令">1.2 help命令</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#13-查看节点列表">1.3 查看节点列表</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#14-新增节点">1.4 新增节点</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#15-查看节点">1.5 查看节点</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#16-更新节点">1.6 更新节点</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#17-删除节点">1.7 删除节点</a><br/>
<a href="#二监听器">二、监听器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#21-get-path-[watch]">2.1 get path [watch]</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#22-stat-path-[watch]">2.2 stat path [watch]</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#23-lsls2-path--[watch]">2.3 ls\ls2 path  [watch]</a><br/>
<a href="#三-zookeeper-四字命令">三、 zookeeper 四字命令</a><br/>
</nav>


## 一、节点增删改查

### 1.1 启动服务和连接服务

```shell
# 启动服务
bin/zkServer.sh start

#连接服务 不指定服务地址则默认连接到localhost:2181
zkCli.sh -server hadoop001:2181
```

### 1.2 help命令

使用 `help` 可以查看所有命令及格式。

```
ZooKeeper -server host:port -client-configuration properties-file cmd args
        addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
        addauth scheme auth
        close
        config [-c] [-w] [-s]
        connect host:port
        create [-s] [-e] [-c] [-t ttl] path [data] [acl]
        delete [-v version] path
        deleteall path [-b batch size]
        delquota [-n|-b] path
        get [-s] [-w] path
        getAcl [-s] path
        getAllChildrenNumber path
        getEphemerals path
        history
        listquota path
        ls [-s] [-w] [-R] path
        printwatches on|off
        quit
        reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
        redo cmdno
        removewatches path [-c|-d|-a] [-l]
        set [-s] [-v version] path data
        setAcl [-s] [-v version] [-R] path acl
        setquota -n|-b val path
        stat [-w] path
        sync path
        version
```


### 1.3 查看节点列表

查看节点列表有 `ls path` 和 `ls -s path` 两个命令，后者是前者的增强，不仅可以查看指定路径下的所有节点，还可以查看当前节点的信息。

```shell
[zk: localhost:2181(CONNECTED) 0] ls /
[zk: localhost:2181(CONNECTED) 1] ls -s /
[a0000000001, b0000000002, c0000000003, hadoop, zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x300000011
cversion = 9
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 7
```

### 1.4 新增节点

```shell
create [-s] [-e] [-c] [-t ttl] path [data] [acl]   #其中-s 为有序节点，-e 临时节点
```

创建节点并写入数据：

```shell
create /hadoop 123456
```

创建有序节点，此时创建的节点名为指定节点名 + 自增序号：

```shell
[zk: localhost:2181(CONNECTED) 23] create -s /a  "aaa"
Created /a0000000022
[zk: localhost:2181(CONNECTED) 24] create -s /b  "bbb"
Created /b0000000023
[zk: localhost:2181(CONNECTED) 25] create -s /c  "ccc"
Created /c0000000024
```

创建临时节点，临时节点会在会话过期后被删除：

```shell
[zk: localhost:2181(CONNECTED) 26] create -e /tmp  "tmp"
Created /tmp
```

### 1.5 查看节点

#### 1. 获取节点数据

```shell
# 格式
# get [-s] [-w] path
[zk: localhost:2181(CONNECTED) 24] get -s -w /hadoop
123456
cZxid = 0x200000002
ctime = Sat Nov 12 17:08:09 CST 2020
mZxid = 0x200000002
mtime = Sat Nov 12 17:08:09 CST 2020
pZxid = 0x200000002
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0

[zk: localhost:2181(CONNECTED) 25] get -s -w /tmp
tmp
cZxid = 0x200000006
ctime = Sat Nov 12 17:10:45 CST 2021
mZxid = 0x200000006
mtime = Sat Nov 12 17:10:45 CST 2021
pZxid = 0x200000006
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x1000251b9bb0000
dataLength = 3
numChildren = 0
```

节点各个属性如下表。其中一个重要的概念是 Zxid(ZooKeeper Transaction  Id)，ZooKeeper 节点的每一次更改都具有唯一的 Zxid，如果 Zxid1 小于 Zxid2，则 Zxid1 的更改发生在 Zxid2 更改之前。

| **状态属性**   | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 数据节点创建时的事务 ID                                       |
| ctime          | 数据节点创建时的时间                                         |
| mZxid          | 数据节点最后一次更新时的事务 ID                               |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pZxid          | 数据节点的子节点最后一次被修改时的事务 ID                     |
| cversion       | 子节点的更改次数                                             |
| dataVersion    | 节点数据的更改次数                                           |
| aclVersion     | 节点的 ACL 的更改次数                                          |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的 SessionID；如果节点是持久节点，则该属性值为 0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 数据节点当前的子节点个数                                     |

#### 2. 查看节点状态

可以使用 `stat` 命令查看节点状态，它的返回值和 `get` 命令类似，但不会返回节点数据。

```shell
[zk: localhost:2181(CONNECTED) 32] stat /hadoop
cZxid = 0x14b
ctime = Fri May 24 17:03:06 CST 2019
mZxid = 0x14b
mtime = Fri May 24 17:03:06 CST 2019
pZxid = 0x14b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

### 1.6 更新节点

更新节点的命令是 `set`，可以直接进行修改，如下：

```shell
[zk: localhost:2181(CONNECTED) 33] set /hadoop 345
cZxid = 0x14b
ctime = Fri May 24 17:03:06 CST 2019
mZxid = 0x14c
mtime = Fri May 24 17:13:05 CST 2019
pZxid = 0x14b
cversion = 0
dataVersion = 1  # 注意更改后此时版本号为 1，默认创建时为 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

也可以基于版本号进行更改，此时类似于乐观锁机制，当你传入的数据版本号 (dataVersion) 和当前节点的数据版本号不符合时，zookeeper 会拒绝本次修改：

```shell
[zk: localhost:2181(CONNECTED) 34] set  -v 0 /hadoop 678
version No is not valid : /hadoop  #无效的版本号
```

### 1.7 删除节点

删除节点的语法如下：

```shell
delete path -v [version]
```

和更新节点数据一样，也可以传入版本号，当你传入的数据版本号 (dataVersion) 和当前节点的数据版本号不符合时，zookeeper 不会执行删除操作。

```
[zk: localhost:2181(CONNECTED) 35] delete /hadoop -v 0
version No is not valid : /hadoop
[zk: localhost:2181(CONNECTED) 36] delete /hadoop
```

要想删除某个节点及其所有后代节点，可以使用递归删除，命令为 `rmr path`。

## 二、监听器

### 2.1 get [-w] path

使用 `get [-w] path` 注册的监听器能够在节点内容发生改变的时候，向客户端发出通知。需要注意的是 zookeeper 的触发器是一次性的 (One-time trigger)，即触发一次后就会立即失效。

```shell
[zk: localhost:2181(CONNECTED) 4] get -w /hadoop
[zk: localhost:2181(CONNECTED) 5] set /hadoop 456

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop
```

### 2.2 stat [-w] path 

使用 `stat [-w] path` 注册的监听器能够在节点状态发生改变的时候，向客户端发出通知。

```shell
[zk: localhost:2181(CONNECTED) 7] stat -w /hadoop
[zk: localhost:2181(CONNECTED) 8] set /hadoop 112233
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop
```

### 2.3 ls [-s] [-w] [-R] path

使用 `ls [-s] [-w] -R path` 注册的监听器能够监听该节点下所有**子节点**的增加和删除操作。

```shell
[zk: localhost:2181(CONNECTED) 9] ls -w -R /hadoop
[]
[zk: localhost:2181(CONNECTED) 10] create  /hadoop/yarn "aaa"
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/hadoop
```



## 三、 zookeeper 四字命令

| 命令 | 功能描述|
| ---- | ------------------------------------------------------------ |
| conf | 打印有关服务配置的详细信息。|
| cons | 列出连接到此服务器的所有客户端的完整连接/会话详细信息。包括有关接收/发送的数据包数量、会话 ID、操作延迟、上次执行的操作等信息...|
| crst | 重置所有连接的连接/会话统计信息。|
| dump | 列出未完成的会话和临时节点。|
| envi | 打印有关服务环境的详细信息|
| ruok | 测试服务器是否在非错误状态下运行。如果服务器正在运行，它将以 imok 响应。否则它根本不会响应。“imok”响应并不一定表示服务器已加入仲裁，只是服务器进程处于活动状态并绑定到指定的客户端端口。使用“stat”获取有关状态wrt quorum和客户端连接信息的详细信息。|
| srst | 重置服务器统计信息。|
| srvr | 列出服务器的完整详细信息。|
| stat | 列出服务器和连接客户端的简要详细信息。|
| wchs | 列出有关服务器监视的简要信息。|
| wchc | 按会话列出有关服务器监视的详细信息。这将输出具有关联手表（路径）的会话（连接）列表。请注意，根据观察次数，此操作可能会很昂贵（即影响服务器性能），请谨慎使用。|
| dirs | 以字节为单位显示快照和日志文件的总大小|
| wchp | 按路径列出有关服务器监视的详细信息。这将输出具有关联会话的路径（znode）列表。请注意，根据观察次数，此操作可能会很昂贵（即影响服务器性能），请谨慎使用。|
| mntr | 输出可用于监控集群健康状况的变量列表。|
| isro | 测试服务器是否以只读模式运行。如果处于只读模式，服务器将响应“ro”，如果不是只读模式，则响应“rw”。
| hash | 返回与 zxid 关联的树摘要的最新历史记录。
| gtmk | 以十进制格式的 64 位有符号长值形式获取当前跟踪掩码。有关stmk可能值的说明，请参见。
| stmk | 设置当前跟踪掩码。跟踪掩码是 64 位，其中每一位启用或禁用服务器上特定类别的跟踪日志记录。Log4J 必须首先配置为启用TRACE级别才能查看跟踪日志消息。跟踪掩码的位对应于以下跟踪记录类别。


> 更多四字命令可以参阅官方文档：https://zookeeper.apache.org/doc/current/zookeeperAdmin.html

使用前需要使用 `sudo yum install nc` 安装 nc 命令，使用示例如下：

```shell
[root@hadoop001 bin]# echo stat | nc localhost 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, 
built on 06/29/2018 04:05 GMT
Clients:
 /0:0:0:0:0:0:0:1:50584[1](queued=0,recved=371,sent=371)
 /0:0:0:0:0:0:0:1:50656[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/19
Received: 372
Sent: 371
Connections: 2
Outstanding: 0
Zxid: 0x150
Mode: standalone
Node count: 167
```


```shell
[hadoop@node02 ~]$ echo stat | nc localhost 2181
stat is not executed because it is not in the whitelist.
```
- 若如上提示 指令不在白名单中，则需要修改`$ZK_HOME/conf/zoo.cfg`


```conf
vim $ZK_HOME/conf/zoo.cfg

# 将需要的命令添加到白名单中
4lw.commands.whitelist=stat, ruok, conf, isro

# 将所有命令添加到白名单中
4lw.commands.whitelist=*
```

- node01修改后将文件同步至其他节点

```shell
cd $ZK_HOME/conf
scp zoo.cfg node02:$PWD
scp zoo.cfg node03:$PWD
```

- 重启zookeeper