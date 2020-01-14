**Apache Kafka 编程实战您可能感性的文章:**

[Apache-Kafka简介](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000482%26idx%3D1%26sn%3D22b13749ed0352cd286eac7697f39f23%26chksm%3D7d3d44774a4acd6189d082976e90087a9a955e6ca12b21193395536643a302ac4c13c88fe212%23rd)

[Apache Kafka安装和使用](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000470%26idx%3D1%26sn%3D41ee111a073c51af4f9e87c2cdc4d584%26chksm%3D7d3d44434a4acd55b67414765a7b79152d7ef430ba00bec8af6cdddd8e8cf161777ee4a15841%23rd)

[Apache-Kafka核心概念](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000472%26idx%3D1%26sn%3D99353b901d1174c3edd4a9ebbe394975%26chksm%3D7d3d444d4a4acd5bf0017210f55ec394abda01d163674d540988ca94863a51411be951711553%23rd)


[Apache-Kafka核心组件和流程-协调器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000476%26idx%3D1%26sn%3D34b2127b1a09664087e3b2079844c2db%26chksm%3D7d3d44494a4acd5f3bc70d914ae2842409282780d19d57043d168895e55f160b3be7835e2446%23rd)

[Apache-Kafka核心组件和流程(副本管理器)](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000480%26idx%3D1%26sn%3D054cdf620eb82c4ecfaccd226d49d0e0%26chksm%3D7d3d44754a4acd638ca37afcfdaad802bb3dec01758b18cdf2c607ec494526832ee58ff43451%23rd)

[Apache-Kafka 核心组件和流程-控制器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000474%26idx%3D1%26sn%3Dc9b9d8fbb942f5299eb1d23a9363c0a4%26chksm%3D7d3d444f4a4acd597607e33ee59aad92db50084a5ab7edb84449df6f2f3ecc504e97f05977bb%23rd)

[Apache-Kafka核心组件和流程-日志管理器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000478%26idx%3D1%26sn%3Deeb3310214d7fa24ca86c4afad421baa%26chksm%3D7d3d444b4a4acd5d1987dc78f89d40a20833cec682b30b9f1a0735a26681f681a38853a6ff63%23rd)

....

本章简单介绍了副本管理器，副本管理器负责分区及其副本的管理。副本管理器具体的工作流程可以参考牟大恩所著的《Kafka入门与实践》

## 副本管理器

副本机制使得kafka整个集群中，只要有一个代理存活，就可以保证集群正常运行。这大大提高了Kafka的可靠性和稳定性。Kafka中代理的存活，需要满足以下两个条件：

*   存活的节点要维持和zookeeper的session连接，通过zookeeper的心跳机制实现
*   Follower副本要与leader副本保持同步，不能落后太多。

满足以上条件的节点在ISR中，一旦宕机，或者中断时间太长，Leader就会把同步副本从ISR中踢出。

所有节点中，leader节点负责接收客户端的读写操作，follower节点从leader复制数据。

副本管理器负责对副本管理。由于副本是分区的副本，所以对副本的管理体现在对分区的管理。

在第三章已经对分区和副本有了详细的讲解，这里再介绍两个重要的概念，LEO和HW。

*   LEO是Log End Offset缩写。表示每个分区副本的最后一条消息的位置，也就是说每个副本都有LEO。
*   HW是Hight Watermark缩写，他是一个分区所有副本中，最小的那个LEO。

看下图：

![image](http://upload-images.jianshu.io/upload_images/16241060-ee582b2718509fbe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分区test-0有三个副本，每个副本的LEO就是自己最后一条消息的offset。可以看到最小的LEO是Replica2的，等于3，也就是说HW=3。这代表offset=4的消息还没有被所有副本复制，是无法被消费的。而offset<=3的数据已经被所有副本复制，是可以被消费的。

副本管理器所承担的职责如下：

*   副本过期检查
*   追加消息
*   拉取消息
*   副本同步过程
*   副本角色转换
*   关闭副本

![image](http://upload-images.jianshu.io/upload_images/16241060-55d26b05293d757f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
