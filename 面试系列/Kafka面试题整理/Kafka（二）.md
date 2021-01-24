## Kafka面试题整理（二）  

### 1、请说明什么是Apache Kafka？  
&emsp; Apache Kafka是由Apache开发的一种发布订阅消息系统，它是一个分布式的、分区的和重复的日志服务。  

### 2、请说明什么是传统的消息传递方法？  
&emsp; 传统的消息传递方法包括两种：  
&emsp; &emsp; 队列：在队列中，一组用户可以从服务器中读取消息，每条消息都发送给其中一个人。  
&emsp; &emsp; 发布-订阅：在这个模型中，消息被广播给所有的用户。  

### 3、请说明Kafka相对于传统的消息传递方法有什么优势？  
&emsp; 高性能：单一的Kafka代理可以处理成千上万的客户端，每秒处理数兆字节的读写操作，Kafka性能远超过传统的ActiveMQ、RabbitMQ等，而且Kafka支持Batch操作；   
&emsp; 可扩展：Kafka集群可以透明的扩展，增加新的服务器进集群；  
&emsp; 容错性： Kafka每个Partition数据会复制到几台服务器，当某个Broker失效时，Zookeeper将通知生产者和消费者从而使用其他的Broker。  

### 4、在Kafka中broker的意义是什么？  
&emsp; 在Kafka集群中，broker指Kafka服务器。  
&emsp; 术语解析：  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Kafka%E9%9D%A2%E8%AF%95%E9%A2%98Pics/Kafka%E4%B8%ADbroker%E7%9A%84%E6%84%8F%E4%B9%89.png"/>  
<p align="center">
</p>
</p>  

### 5、Kafka服务器能接收到的最大信息是多少？  
&emsp; Kafka服务器可以接收到的消息的最大大小是1000000字节。  

### 6、Kafka中的ZooKeeper是什么？Kafka是否可以脱离ZooKeeper独立运行？  
&emsp; Zookeeper是一个开放源码的、高性能的协调服务，它用于Kafka的分布式应用。  
&emsp; 不可以，不可能越过Zookeeper直接联系Kafka broker，一旦Zookeeper停止工作，它就不能服务客户端请求。  
&emsp; Zookeeper主要用于在集群中不同节点之间进行通信，在Kafka中，它被用于提交偏移量，因此如果节点在任何情况下都失败了，它都可以从之前提交的偏移量中获取，除此之外，它还执行其他活动，如: leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等。  

### 7、解释Kafka的用户如何消费信息？  
&emsp; 在Kafka中传递消息是通过使用sendfile API完成的。它支持将字节Socket转移到磁盘，通过内核空间保存副本，并在内核用户之间调用内核。  

### 8、解释如何提高远程用户的吞吐量？  
&emsp; 如果用户位于与broker不同的数据中心，则可能需要调优Socket缓冲区大小，以对长网络延迟进行摊销。  

### 9、解释一下，在数据制作过程中，你如何能从Kafka得到准确的信息？  
&emsp; 在数据中，为了精确地获得Kafka的消息，你必须遵循两件事：**在数据消耗期间避免重复，在数据生产过程中避免重复**。  
&emsp; 这里有两种方法，可以在数据生成时准确地获得一个语义:   
&emsp; 每个分区使用一个单独的写入器，每当你发现一个网络错误，检查该分区中的最后一条消息，以查看您的最后一次写入是否成功。  
&emsp; 在消息中包含一个主键(UUID或其他)，并在用户中进行反复制。  

### 10、解释如何减少ISR中的扰动？broker什么时候离开ISR？（☆☆☆☆☆）  
&emsp; ISR是一组与leaders完全同步的消息副本，也就是说ISR中包含了所有提交的消息。ISR应该总是包含所有的副本，直到出现真正的故障。如果一个副本从leader中脱离出来，将会从ISR中删除。  

### 11、Kafka为什么需要复制？  
&emsp; Kafka的信息复制确保了任何已发布的消息不会丢失，并且可以在机器错误、程序错误或更常见些的软件升级中使用。  

### 12、如果副本在ISR中停留了很长时间表明什么？  
&emsp; 如果一个副本在ISR中保留了很长一段时间，那么它就表明，跟踪器无法像在leader收集数据那样快速地获取数据。  

### 13、请说明如果首选的副本不在ISR中会发生什么？  
&emsp; 如果首选的副本不在ISR中，控制器将无法将leadership转移到首选的副本。  

### 14、Kafka有可能在生产后发生消息偏移吗？  
&emsp; 在大多数队列系统中，作为生产者的类无法做到这一点，它的作用是触发并忘记消息。broker将完成剩下的工作，比如使用id进行适当的元数据处理、偏移量等。  
&emsp; 作为消息的用户，你可以从Kafka broker中获得补偿。如果你注视SimpleConsumer类，你会注意到它会获取包括偏移量作为列表的MultiFetchResponse对象。此外，当你对Kafka消息进行迭代时，你会拥有包括偏移量和消息发送的MessageAndOffset对象。  

### 15、请说明Kafka 的消息投递保证（delivery guarantee）机制以及如何实现？（☆☆☆☆☆）  
&emsp; Kafka支持三种消息投递语义：  
&emsp; ① At most once 消息可能会丢，但绝不会重复传递  
&emsp; ② At least one 消息绝不会丢，但可能会重复传递  
&emsp; ③ Exactly once 每条消息肯定会被传输一次且仅传输一次，很多时候这是用户想要的  
&emsp; consumer在从broker读取消息后，可以选择commit，该操作会在Zookeeper中存下该consumer在该partition下读取的消息的offset，该consumer下一次再读该partition时会从下一条开始读取。如未commit，下一次读取的开始位置会跟上一次commit之后的开始位置相同。  
&emsp; 可以将consumer设置为autocommit，即consumer一旦读到数据立即自动commit。如果只讨论这一读取消息的过程，那Kafka是确保了Exactly once。但实际上实际使用中consumer并非读取完数据就结束了，而是要进行进一步处理，而数据处理与commit的顺序在很大程度上决定了消息从broker和consumer的delivery guarantee semantic。  
&emsp; 读完消息先commit再处理消息。这种模式下，如果consumer在commit后还没来得及处理消息就crash了，下次重新开始工作后就无法读到刚刚已提交而未处理的消息，这就对应于At most once。  
&emsp; 读完消息先处理再commit消费状态(保存offset)。这种模式下，如果在处理完消息之后commit之前Consumer crash了，下次重新开始工作时还会处理刚刚未commit的消息，实际上该消息已经被处理过了，这就对应于At least once。  
&emsp; 如果一定要做到Exactly once，就需要协调offset和实际操作的输出。经典的做法是引入两阶段提交，但由于许多输出系统不支持两阶段提交，更为通用的方式是将offset和操作输入存在同一个地方。比如，consumer拿到数据后可能把数据放到HDFS，如果把最新的offset和数据本身一起写到HDFS，那就可以保证数据的输出和offset的更新要么都完成，要么都不完成，间接实现Exactly once。（目前就high level API而言，offset是存于Zookeeper中的，无法存于HDFS，而low level API的offset是由自己去维护的，可以将之存于HDFS中）。  
&emsp; 总之，Kafka默认保证At least once，并且允许通过设置producer异步提交来实现At most once，而Exactly once要求与目标存储系统协作，Kafka提供的offset可以较为容易地实现这种方式。  

### 16、如何保证Kafka的消息有序（☆☆☆☆☆）  
&emsp; Kafka对于消息的重复、丢失、错误以及顺序没有严格的要求。  
&emsp; Kafka只能保证一个partition中的消息被某个consumer消费时是顺序的，事实上，从Topic角度来说，当有多个partition时，消息仍然不是全局有序的。  

### 17、kafka数据丢失问题,及如何保证？  
1）数据丢失：  
&emsp; acks=1的时候(只保证写入leader成功)，如果刚好leader挂了。数据会丢失。  
&emsp; acks=0的时候，使用异步模式的时候，该模式下kafka无法保证消息，有可能会丢。  
2）brocker如何保证不丢失：  
&emsp; acks=all : 所有副本都写入成功并确认。  
&emsp; retries = 一个合理值。  
&emsp; min.insync.replicas=2  消息至少要被写入到这么多副本才算成功。  
&emsp; unclean.leader.election.enable=false 关闭unclean leader选举，即不允许非ISR中的副本被选举为leader，以避免数据丢失。  
3）Consumer如何保证不丢失  
&emsp; 如果在消息处理完成前就提交了offset，那么就有可能造成数据的丢失。  
&emsp; enabel.auto.commit=false关闭自动提交offset  
&emsp; 处理完数据之后手动提交。  

### 18、kafka的balance是怎么做的？  
&emsp; 生产者将数据发布到他们选择的主题。生产者可以选择在主题中分配哪个分区的消息。这可以通过循环的方式来完成，只是为了平衡负载，或者可以根据一些语义分区功能（比如消息中的一些键）来完成。更多关于分区在一秒钟内的使用。  

### 19、kafka的消费者方式？  
&emsp; consumer采用pull（拉）模式从broker中读取数据。  
&emsp; push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。  
&emsp; 对于Kafka而言，pull模式更合适，它可简化broker的设计，consumer可自主控制消费消息的速率，同时consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。  
&emsp; pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直等待数据到达。为了避免这种情况，我们在我们的拉请求中有参数，允许消费者请求在等待数据到达的“长轮询”中进行阻塞。  













