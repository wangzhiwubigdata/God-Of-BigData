Flink的HA搭建并不复杂，本质来说就是配置2个jobmanager。
本文作为Flink集群部署的补充篇。
> 这篇文章来自网络，向作者尼小摩致敬，

## 概述

JobManager 协调每个 Flink 部署。它负责调度和资源管理。
默认情况下，每个 Flink 集群只有一个 JobManager 实例。 这会产生单点故障(SPOF)：如果 JobManager 崩溃，则无法提交新作业并且导致运行中的作业运行失败。
使用 JobManager 高可用性模式，可以避免这个问题，从而消除 SPOF。您可以为Standalone和 YARN 集群配置高可用性。


## Standalone集群高可用性

针对 Standalone 集群的 JobManager 高可用性的一般概念是，任何时候都有一个 主 JobManager 和多个备 JobManagers，以便在主节点失败时有备 JobManagers 来接管集群。这保证了没有单点故障，一旦备 JobManager 接管集群，作业就可以正常运行。主备  JobManager 实例之间没有明显的区别。每个 JobManager 都可以充当主备节点。
例如，请考虑以下三个 JobManager 实例的设置:

![2278c5ee57d47a73498685c4c728c432](10-Flink集群的高可用(搭建篇补充).resources/6278EDED-A65A-4539-A16D-7BCD9FE77864.png)

### 配置

要启用 JobManager 高可用性，您必须将高可用性模式设置为 zookeeper，配置 zookeeper quorum 将所有 JobManager 主机及其 web UI 端口写入配置文件。
Flink利用 ZooKeeper 在所有正在运行的 JobManager 实例之间进行分布式协调。 ZooKeeper 是独立于 Flink 的服务，通过 Leader 选举和轻量级一致状态存储提供高可靠的分布式协调。

### Masters文件 (masters服务器)

要启动HA集群，请在以下位置配置Master文件


* conf/masters:masters文件：masters文件包含启动 jobmanager 的所有主机和 web 用户界面绑定的端口。

```
 jobManagerAddress1:webUIPort1
    [...]
 jobManagerAddressX:webUIPortX
```

默认情况下，job manager选一个随机端口作为进程随机通信端口。您可以通过 high-availability.jobmanager.port 键修改此设置。此配置接受单个端口（例如50010），范围（50000-50025）或两者的组合（50010,50011,50020-50025,50050-50075）。

### 配置文件（flink-conf.yaml）

要启动HA集群，请将以下配置键添加到 conf/flink-conf.yaml:

* 高可用性模式（必需）：在 conf/flink-conf.yaml 中，必须将高可用性模式设置为zookeeper，以打开高可用模式。或者将此选项设置为工厂类的 FQN，Flink 通过创建 HighAvailabilityServices 实例使用。

```
 high-availability: zookeeper
```

* Zookeeper quorum（必需）： ZooKeeper quorum 是 ZooKeeper 服务器的复制组，它提供分布式协调服务。

```
high-availability.zookeeper.quorum:address1:2181[,...],addressX:2181
```
每个 addressX:port 都是一个 ZooKeeper 服务器的ip及其端口，Flink 可以在指定的地址和端口访问zookeeper。

* ZooKeeper root （推荐）： ZooKeeper 根节点，在该节点下放置所有集群节点。

```
  high-availability.zookeeper.path.root: /flink
```

* ZooKeeper cluster-id（推荐）： ZooKeeper的cluster-id节点，在该节点下放置集群的所有相关数据。

```
 high-availability.cluster-id: /default_ns # important: customize per cluster
```

**重要：** 在运行 YARN 或其他群集管理器中运行时，不要手动设置此值。在这些情况下，将根据应用程序 ID 自动生成 cluster-id。 手动设置 cluster-id 会覆盖 YARN 中的自动生成的 ID。反过来，使用 -z CLI 选项指定 cluster-id 会覆盖手动配置。如果在裸机上运行多个 Flink HA 集群，则必须为每个集群手动配置单独的 cluster-id。


* 存储目录（必需）： JobManager 元数据保存在文件系统 storageDir 中，在 ZooKeeper 中仅保存了指向此状态的指针。

```
high-availability.storageDir: hdfs:///flink/recovery
```

该storageDir 中保存了 JobManager 恢复状态所需的所有元数据。
配置 master 文件和 ZooKeeper quorum 之后，您可以使用提供的集群启动脚本。它们将启动 HA 群集。请注意，启动 Flink HA 集群前，必须启动 Zookeeper 集群，并确保为要启动的每个 HA 群集配置单独的 ZooKeeper 根路径。

**示例：具有2个 JobManager 的 Standalone 集群**

1. 在conf/flink-conf.yaml 中配置高可用模式和 ZooKeeper quorum：

```
high-availability: zookeeper
high-availability.zookeeper.quorum: localhost:2181
high-availability.zookeeper.path.root: /flink
high-availability.cluster-id: /cluster_one
high-availability.storageDir:    hdfs:///flink/recovery
```
2. 在 conf/master 中配置 master:

```
    localhost:8081
    localhost:8082
```
3. 在 conf/zoo.cfg 中配置 ZooKeeper 服务（目前，每台机器只能运行一个 ZooKeeper 进程）

```
    server.0=localhost:2888:3888
```

4. 启动 ZooKeeper quorum：

```
    $ bin/start-zookeeper-quorum.sh
    Starting zookeeper daemon on host localhost.
```

5. 启动 Flink HA 集群：

```
    $ bin/start-cluster.sh
    Starting HA cluster with 2 masters and 1 peers     in ZooKeeper quorum.
    Starting jobmanager daemon on host localhost.
    Starting jobmanager daemon on host localhost.
    Starting taskmanager daemon on host localhost.
```

6. 停止 Zookeeper quorum 和集群：

```
    $ bin/stop-cluster.sh
    Stopping taskmanager daemon (pid: 7647) on localhost.
    Stopping jobmanager daemon (pid: 7495) on host localhost.
    Stopping jobmanager daemon (pid: 7349) on host localhost.
    $ bin/stop-zookeeper-quorum.sh
    Stopping zookeeper daemon (pid: 7101) on host localhost.
```

## YARN 集群的高可用性

在运行高可用性 YARN 集群时，我们不会运行多个 JobManager (ApplicationMaster) 实例，而只运行一个，该JobManager实例失败时，YARN会将其重新启动。Yarn的具体行为取决于您使用的 YARN 版本。


### 配置

Application Master最大重试次数（yarn-site.xml）
在YARN 配置文件 yarn-site.xml 中，需要配置 application master 的最大重试次数：

```
<property>
  <name>yarn.resourcemanager.am.max-attempts</name>
  <value>4</value>
  <description>
    The maximum number of application master execution attempts.
  </description>
</property>
```

当前 YARN 版本的默认值是2(表示允许单个JobManager失败两次)。

Application Attempts（flink-conf.yaml）：
除了HA配置(参考上文)之外，您还必须配置最大重试次数 conf/flink-conf.yaml:

```
yarn.application-attempts: 10
```

这意味着在如果程序启动失败，YARN会再重试9次（9 次重试 + 1次启动）。如果 YARN 操作需要，如果启动10次作业还失败，yarn才会将该任务的状态置为失败。如果抢占，节点硬件故障或重启，NodeManager 重新同步等操作需要，YARN继续尝试启动应用。 这些重启不计入 yarn.application-attempts 个数中。重要的是要注意 yarn.resourcemanager.am.max-attempts 为yarn中程序重启上限。因此， Flink 中设置的程序尝试次数不能超过 YARN 的集群设置。

### 示例：高可用的YARN Session

1.配置 HA 模式和 ZooKeeper 集群在 conf/flink-conf.yaml 中：
```
    high-availability: zookeeper
    high-availability.zookeeper.quorum: localhost:2181
    high-availability.storageDir: hdfs:///flink/recovery
    high-availability.zookeeper.path.root: /flink
    yarn.application-attempts: 10

```
2. 配置 ZooKeeper 服务在 conf/zoo.cfg 中(目前每台机器只能运行一个 ZooKeeper 进程)：
```
    server.0=localhost:2888:3888

```
3. 启动 ZooKeeper 集群：
```
    $ bin/start-zookeeper-quorum.sh
    Starting zookeeper daemon on host localhost.

```
4. 启动 HA 集群：
```
    $ bin / yarn-session.sh -n 2
```
### 配置 Zookeeper 安全性

如果 ZooKeeper 使用 Kerberos 以安全模式运行，flink-conf.yaml 根据需要覆盖以下配置：
```
zookeeper.sasl.service-name: zookeeper 
# 默认设置是 “zookeeper” 。如果 ZooKeeper 集群配置了
# 不同的服务名称，那么可以在这里提供。

zookeeper.sasl.login-context-name: Client  
# 默认设置是 “Client”。该值配置需要匹配
# "security.kerberos.login.contexts"中的其中一个值。
```

有关 Kerberos 安全性的 Flink 配置的更多信息，请参阅 此处。您还可以在 此处 找到关于 Flink 内部如何设置基于 kerberos 的安全性的详细信息。


### Bootstrap ZooKeeper

如果您没有正在运行的ZooKeeper，则可以使用Flink程序附带的脚本。
这是一个 ZooKeeper 配置模板 conf/zoo.cfg。您可以为主机配置为使用 server.X 条目运行 ZooKeeper，其中 X 是每个服务器的唯一IP:
```
server.X=addressX:peerPort:leaderPort
[...]
server.Y=addressY:peerPort:leaderPort
```
该脚本 bin/start-zookeeper-quorum.sh 将在每个配置的主机上启动 ZooKeeper 服务器。 Flink wrapper 会启动 ZooKeeper 服务，该 wraper 从 conf/zoo.cfg 中读取配置，并设置一些必需的配置项。在生产设置中，建议您使用自己安装的 ZooKeeper。

