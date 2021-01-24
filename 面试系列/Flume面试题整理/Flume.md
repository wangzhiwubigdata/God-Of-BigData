## Flume面试题整理（一）  

### 1、Flume使用场景（☆☆☆☆☆）  
&emsp; 线上数据一般主要是落地（存储到磁盘）或者通过socket传输给另外一个系统，这种情况下，你很难推动线上应用或服务去修改接口，实现直接向kafka里写数据，这时候你可能就需要flume这样的系统帮你去做传输。  

### 2、Flume丢包问题（☆☆☆☆☆）  
&emsp; 单机upd的flume source的配置，100+M/s数据量，10w qps flume就开始大量丢包，因此很多公司在搭建系统时，抛弃了Flume，自己研发传输系统，但是往往会参考Flume的Source-Channel-Sink模式。  
&emsp; 一些公司在Flume工作过程中，会对业务日志进行监控，例如Flume agent中有多少条日志，Flume到Kafka后有多少条日志等等，如果数据丢失保持在1%左右是没有问题的，当数据丢失达到5%左右时就必须采取相应措施。  

### 3、Flume与Kafka的选取  
&emsp; 采集层主要可以使用Flume、Kafka两种技术。  
&emsp; Flume：Flume 是管道流方式，提供了很多的默认实现，让用户通过参数部署，及扩展API。  
&emsp; Kafka：Kafka是一个可持久化的分布式的消息队列。  
&emsp; Kafka 是一个非常通用的系统。你可以有许多生产者和很多的消费者共享多个主题Topics。相比之下，Flume是一个专用工具被设计为旨在往HDFS，HBase发送数据。它对HDFS有特殊的优化，并且集成了Hadoop的安全特性。所以，Cloudera 建议如果数据被多个系统消费的话，使用kafka；如果数据被设计给Hadoop使用，使用Flume。  
&emsp; 正如你们所知Flume内置很多的source和sink组件。然而，Kafka明显有一个更小的生产消费者生态系统，并且Kafka的社区支持不好。希望将来这种情况会得到改善，但是目前：使用Kafka意味着你准备好了编写你自己的生产者和消费者代码。如果已经存在的Flume Sources和Sinks满足你的需求，并且你更喜欢不需要任何开发的系统，请使用Flume。  
&emsp; Flume可以使用拦截器实时处理数据。这些对数据屏蔽或者过量是很有用的。Kafka需要外部的流处理系统才能做到。  
&emsp; Kafka和Flume都是可靠的系统，通过适当的配置能保证零数据丢失。然而，Flume不支持副本事件。于是，如果Flume代理的一个节点奔溃了，即使使用了可靠的文件管道方式，你也将丢失这些事件直到你恢复这些磁盘。如果你需要一个高可靠性的管道，那么使用Kafka是个更好的选择。  
&emsp; Flume和Kafka可以很好地结合起来使用。如果你的设计需要从Kafka到Hadoop的流数据，使用Flume代理并配置Kafka的Source读取数据也是可行的：你没有必要实现自己的消费者。你可以直接利用Flume与HDFS及HBase的结合的所有好处。你可以使用Cloudera Manager对消费者的监控，并且你甚至可以添加拦截器进行一些流处理。  

### 4、数据怎么采集到Kafka，实现方式  
&emsp; 使用官方提供的flumeKafka插件，插件的实现方式是自定义了flume的sink，将数据从channle中取出，通过kafka的producer写入到kafka中，可以自定义分区等。  

### 5、flume管道内存，flume宕机了数据丢失怎么解决  
&emsp; 1）Flume的channel分为很多种，可以将数据写入到文件。  
&emsp; 2）防止非首个agent宕机的方法数可以做集群或者主备。  

### 6、flume配置方式，flume集群（详细讲解下）  
&emsp; Flume的配置围绕着source、channel、sink叙述，flume的集群是做在agent上的，而非机器上。  

### 7、flume不采集Nginx日志，通过Logger4j采集日志，优缺点是什么？  
&emsp; 优点：Nginx的日志格式是固定的，但是缺少sessionid，通过logger4j采集的日志是带有sessionid的，而session可以通过redis共享，保证了集群日志中的同一session落到不同的tomcat时，sessionId还是一样的，而且logger4j的方式比较稳定，不会宕机。  
&emsp; 缺点：不够灵活，logger4j的方式和项目结合过于紧密，而flume的方式比较灵活，拔插式比较好，不会影响项目性能。  

### 8、flume和kafka采集日志区别，采集日志时中间停了，怎么记录之前的日志？  
&emsp; Flume采集日志是通过流的方式直接将日志收集到存储层，而kafka是将缓存在kafka集群，待后期可以采集到存储层。  
&emsp; Flume采集中间停了，可以采用文件的方式记录之前的日志，而kafka是采用offset的方式记录之前的日志。  

### 9、flume有哪些组件，flume的source、channel、sink具体是做什么的（☆☆☆☆☆）  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Flume%E9%9D%A2%E8%AF%95%E9%A2%98Pics/flume%E7%9A%84source%E3%80%81channel%E3%80%81sink.png"/>  
<p align="center">
</p>
</p>  

1）source：用于采集数据，Source是产生数据流的地方，同时Source会将产生的数据流传输到Channel，这个有点类似于Java IO部分的Channel。  
2）channel：用于桥接Sources和Sinks，类似于一个队列。  
3）sink：从Channel收集数据，将数据写到目标源(可以是下一个Source，也可以是HDFS或者HBase)。  
**注意：要熟悉source、channel、sink的类型**  











