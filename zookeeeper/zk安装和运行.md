

> 本文由holynull发表在了gitbook上
> 大家可以点击这里获取更好的阅读体验: https://holynull.gitbooks.io/zookeeper/content/chapter1.html

> zk目前更新到了3.x版本，官网在这里：https://zookeeper.apache.org/doc/current/zookeeperStarted.html

另外推荐文章：
http://www.importnew.com/23237.html


## 简介

Zookeeper是Hadoop分布式调度服务，用来构建分布式应用系统。构建一个分布式应用是一个很复杂的事情，主要的原因是我们需要合理有效的处理分布式集群中的部分失败的问题。例如，集群中的节点在相互通信时，A节点向B节点发送消息。A节点如果想知道消息是否发送成功，只能由B节点告诉A节点。那么如果B节点关机或者由于其他的原因脱离集群网络，问题就出现了。A节点不断的向B发送消息，并且无法获得B的响应。B也没有办法通知A节点已经离线或者关机。集群中其他的节点完全不知道B发生了什么情况，还在不断的向B发送消息。这时，你的整个集群就发生了部分失败的故障。
Zookeeper不能让部分失败的问题彻底消失，但是它提供了一些工具能够让你的分布式应用安全合理的处理部分失败的问题。

## 安装和运行Zookeeper

我们采用standalone模式，安装运行一个单独的zookeeper服务。安装前请确认您已经安装了Java运行环境。
我们去Apache ZooKeeper releases page下载zookeeper安装包，并解压到本地：

```
% tar xzf zookeeper-x.y.z.tar.gz
```
ZooKeeper提供了一些可执行程序的工具，为了方便起见，我们将这些工具的路径加入到PATH环境变量中：
```
% export ZOOKEEPER_HOME=~/sw/zookeeper-x.y.z
% export PATH=$PATH:$ZOOKEEPER_HOME/bin
```
运行ZooKeeper之前我们需要编写配置文件。配置文件一般在安装目录下的conf/zoo.cfg。我们可以把这个文件放在/etc/zookeeper下，或者放到其他目录下，并在环境变量设置ZOOCFGDIR指向这个个目录。下面是配置文件的内容：
```
tickTime=2000
dataDir=/Users/tom/zookeeper
clientPort=2181
```
tickTime是zookeeper中的基本时间单元，单位是毫秒。datadir是zookeeper持久化数据存放的目录。clientPort是zookeeper监听客户端连接的端口，默认是2181.
启动命令：
```
% zkServer.sh start
```
我们通过nc或者telnet命令访问2181端口，通过执行ruok（Are you OK?）命令来检查zookeeper是否启动成功：
```
% echo ruok | nc localhost 2181
imok
```
那么我看见zookeeper回答我们“I’m OK”。

