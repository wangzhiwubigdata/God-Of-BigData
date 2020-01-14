**您可能感兴趣的文章:**

[Apache-Kafka简介](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000482%26idx%3D1%26sn%3D22b13749ed0352cd286eac7697f39f23%26chksm%3D7d3d44774a4acd6189d082976e90087a9a955e6ca12b21193395536643a302ac4c13c88fe212%23rd)

[Apache Kafka安装和使用](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000470%26idx%3D1%26sn%3D41ee111a073c51af4f9e87c2cdc4d584%26chksm%3D7d3d44434a4acd55b67414765a7b79152d7ef430ba00bec8af6cdddd8e8cf161777ee4a15841%23rd)

[Apache-Kafka核心概念](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000472%26idx%3D1%26sn%3D99353b901d1174c3edd4a9ebbe394975%26chksm%3D7d3d444d4a4acd5bf0017210f55ec394abda01d163674d540988ca94863a51411be951711553%23rd)


[Apache-Kafka核心组件和流程-协调器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000476%26idx%3D1%26sn%3D34b2127b1a09664087e3b2079844c2db%26chksm%3D7d3d44494a4acd5f3bc70d914ae2842409282780d19d57043d168895e55f160b3be7835e2446%23rd)

[Apache-Kafka核心组件和流程(副本管理器)](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000480%26idx%3D1%26sn%3D054cdf620eb82c4ecfaccd226d49d0e0%26chksm%3D7d3d44754a4acd638ca37afcfdaad802bb3dec01758b18cdf2c607ec494526832ee58ff43451%23rd)

[Apache-Kafka 核心组件和流程-控制器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000474%26idx%3D1%26sn%3Dc9b9d8fbb942f5299eb1d23a9363c0a4%26chksm%3D7d3d444f4a4acd597607e33ee59aad92db50084a5ab7edb84449df6f2f3ecc504e97f05977bb%23rd)

[Apache-Kafka核心组件和流程-日志管理器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000478%26idx%3D1%26sn%3Deeb3310214d7fa24ca86c4afad421baa%26chksm%3D7d3d444b4a4acd5d1987dc78f89d40a20833cec682b30b9f1a0735a26681f681a38853a6ff63%23rd)

....

**kafka的定位**

提到kafka，不太熟悉或者稍有接触的开发人员，第一想法可能会觉得它是一个消息系统。其实Kafka的定位并不止于此。

Kafka官方文档介绍说，Apache Kafka是一个分布式流平台，并给出了如下解释：

流平台有三个关键的能力：

*   发布订阅记录流，和消息队列或者企业新消息系统类似。
*   以可容错、持久的方式保存记录流
*   当记录流产生时就进行处理

Kafka通常用于应用中的两种广播类型：

*   在系统和应用间建立实时的数据管道，能够可信赖的获取数据。
*   建立实时的流应用，可以处理或者响应数据流。

由此可见，kafka给自身的定位并不只是一个消息系统，而是通过发布订阅消息这种机制实现了流平台。

其实不管kafka给自己的定位如何，他都逃脱不了发布订阅消息的底层机制。本文讲解的重点，也是kafka发布订阅消息的特性。

Kafka和大多数消息系统一样，搭建好kafka集群后，生产者向特定的topic生产消息，而消费者通过订阅topic，能够准实时的拉取到该topic新消息，进行消费。如下图：

![image](http://upload-images.jianshu.io/upload_images/16241060-eabf90da50c94506.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Kafka特性**

kafka和有以下主要的特性：

*   消息持久化
*   高吞吐量
*   可扩展性

尤其是高吞吐量，是他的最大卖点。kafka之所以能够实现高吞吐量，是基于他自身优良的设计，及集群的可扩展性。后面章节会展开来分析。

**Kafka应用场景**

*   消息系统
*   日志系统
*   流处理
