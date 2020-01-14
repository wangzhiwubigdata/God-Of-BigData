**Apache Kafka 编程实战您可能感性的文章:**

[Apache-Kafka简介](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000482%26idx%3D1%26sn%3D22b13749ed0352cd286eac7697f39f23%26chksm%3D7d3d44774a4acd6189d082976e90087a9a955e6ca12b21193395536643a302ac4c13c88fe212%23rd)

[Apache Kafka安装和使用](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000470%26idx%3D1%26sn%3D41ee111a073c51af4f9e87c2cdc4d584%26chksm%3D7d3d44434a4acd55b67414765a7b79152d7ef430ba00bec8af6cdddd8e8cf161777ee4a15841%23rd)


[Apache-Kafka核心概念](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000472%26idx%3D1%26sn%3D99353b901d1174c3edd4a9ebbe394975%26chksm%3D7d3d444d4a4acd5bf0017210f55ec394abda01d163674d540988ca94863a51411be951711553%23rd)

[Apache-Kafka核心组件和流程-协调器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000476%26idx%3D1%26sn%3D34b2127b1a09664087e3b2079844c2db%26chksm%3D7d3d44494a4acd5f3bc70d914ae2842409282780d19d57043d168895e55f160b3be7835e2446%23rd)

[Apache-Kafka核心组件和流程(副本管理器)](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000480%26idx%3D1%26sn%3D054cdf620eb82c4ecfaccd226d49d0e0%26chksm%3D7d3d44754a4acd638ca37afcfdaad802bb3dec01758b18cdf2c607ec494526832ee58ff43451%23rd)

[Apache-Kafka 核心组件和流程-控制器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000474%26idx%3D1%26sn%3Dc9b9d8fbb942f5299eb1d23a9363c0a4%26chksm%3D7d3d444f4a4acd597607e33ee59aad92db50084a5ab7edb84449df6f2f3ecc504e97f05977bb%23rd)

[Apache-Kafka核心组件和流程-日志管理器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000478%26idx%3D1%26sn%3Deeb3310214d7fa24ca86c4afad421baa%26chksm%3D7d3d444b4a4acd5d1987dc78f89d40a20833cec682b30b9f1a0735a26681f681a38853a6ff63%23rd)

....

**单机环境**

官方建议使用JDK 1.8版本，因此本文使用的环境都是JDK1.8。关于JDK的安装，本文不再详述，默认Java环境已经具备。

由于Kafka依赖zookeeper，kafka通过zookeeper现实分布式系统的协调，所以我们需要先安装zookeeper。

接下来我们按照如下步骤，一步步来安装kafka：

1、下载zookeeper，解压。

下载地址：[https://zookeeper.apache.org/releases.html#download](http://link.zhihu.com/?target=https%3A//zookeeper.apache.org/releases.html%23download)

2、创建zookeeper配置文件

在zookeeper解压后的目录下找到conf文件夹，进入后，复制文件zoo_sample.cfg，并命名为zoo.cfg

zoo.cfg中一共四个配置项，可以使用默认配置。

3、启动zookeeper。

进入zookeeper根目录执行 bin/zkServer.sh start

![image](http://upload-images.jianshu.io/upload_images/16241060-45642ac6542526af.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、下载kafka，解压。

kafka 2.0版本下载地址：[https://www.apache.org/dyn/closer.cgi?path=/kafka/2.0.0/kafka_2.11-2.0.0.tgz](http://link.zhihu.com/?target=https%3A//www.apache.org/dyn/closer.cgi%3Fpath%3D/kafka/2.0.0/kafka_2.11-2.0.0.tgz)

5、修改kafka的配置文件

进入kafka根目录下的config文件夹下，打开server.properties,修改如下配置

zookeeper.connect=localhost:2181

broker.id=0

log.dirs=/tmp/kafka-logs

zookeeper.connect是zookeeper的链接信息，broker.id是当前kafka实例的id，log.dirs是kafka存储消息内容的路径。

6、启动kafka

进入kafka根目录执行 bin/kafka-server-start.sh config/server.properties

此命令告诉kaka启动时使用config/server.properties配置项

![image](http://upload-images.jianshu.io/upload_images/16241060-7daff2291823ea47.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动kafka后，如果控制台没有报错信息，那么kafka应该已经启动成功了，我们可以通过查看zookeeper中相关节点值来确认。步骤如下：

1、启动zookeeper的client

进入zookeeper根目录下，执行 bin/zkCli.sh -server 127.0.0.1:2181。启动成功后如下图

![image](http://upload-images.jianshu.io/upload_images/16241060-e93e674f8687f166.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、输入命令 ls /brokers，回车，可以看到如下信息：

![image](http://upload-images.jianshu.io/upload_images/16241060-332ed3a117128b6a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这些子节点存储的就是kafka集群管理的数据。broker是kafka的一个服务单元实例

3、我看看一下ids这个节点下的数据，输入命令 ls /brokers/ids，可以看到如下信息：

![image](http://upload-images.jianshu.io/upload_images/16241060-d562b8ccbc167ca9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还记得我们在配置单机环境时，修改的kafka配置项broker.id=0 吗？这里的0就是表示那个kafka的实例已经加入了kafka集群。

### **集群环境**

集群环境的搭建也很简单，在单机环境的基础上，让多个单机连接到同一个zookeeper即可。需要注意两点：

1、每个实例设置不同的broker.id。

2、如果多个实例部署在同一台服务器，还要注意修改log.dirs为不同目录，确保消息存储时不会有冲突。集群环境的具体搭建，在此精简教程中不再做详细讨论。

发出你的第一条kafka消息

我们通过kafka带的工具来创建一个topic，然后尝试发送和消费一个消息，直观的去感受下kafka。

1、创建topic

进入kafka根目录，执行如下命令：

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic study

```

执行成功后，创建了study这个topic，如下图所示：

![image](http://upload-images.jianshu.io/upload_images/16241060-1ee1e88312b5607e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此命令行有几个参数，分别指明了zookeeper的链接信息，分区和副本的数量等。关于分区和副本后续会仔细讲解，现在不用过多关注。

2、启动消费者

我们开启一个消费者并且订阅study这个topic，执行如下命令:

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic study --from-beginning

```

看到如下图，光标停留在最前面，没有任何信息输出，说明启动消费者成功，此时在等待新的消息。

![image](http://upload-images.jianshu.io/upload_images/16241060-4c9ec1250f2a7d52.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、开启生产者

新打开一个命令窗口，输入命令

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic study

```

启动成功后，如下图，等待你输入新的消息。

![image](http://upload-images.jianshu.io/upload_images/16241060-1dc35a39b6a6c1a6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、发送你的第一条消息

在上面生产者的窗口输入一条消息 hello kafka,点击回车，如下图：

![image](http://upload-images.jianshu.io/upload_images/16241060-aa03f34cc9fdc77d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时切换到消费者的窗口，可以看到消费者已经消费到这条消息，在窗口中打印了出来。

![image](http://upload-images.jianshu.io/upload_images/16241060-148e5147382d1712.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此我们走完了一个发送消息的流程，可以看到我们经历了创建topic、启动生产者、消费者、生产者生产消息、消费者消费消息，这几个步骤。

小结：通过本章节学习，相信你已经能够成功搭建起kafka单机环境，甚至集群环境。然后通过kafka自带的工具，直观的感受了kafka运转的整个过程。接下来的章节我们将会进入kafka的核心领域，也是本教程的重点章节，只有理解了kafka内在的设计理念和原理，才能做到活学活用。

![image](http://upload-images.jianshu.io/upload_images/16241060-2349cea8df6b9d79.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
