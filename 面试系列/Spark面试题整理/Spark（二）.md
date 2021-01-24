## Spark面试题（二）

### 1、Spark有哪两种算子？  
&emsp; Transformation（转化）算子和Action（执行）算子。  

### 2、Spark有哪些聚合类的算子,我们应该尽量避免什么类型的算子？  
&emsp; 在我们的开发过程中，能避免则尽可能避免使用reduceByKey、join、distinct、repartition等会进行shuffle的算子，尽量使用map类的非shuffle算子。
这样的话，没有shuffle操作或者仅有较少shuffle操作的Spark作业，可以大大减少性能开销。  

### 3、如何从Kafka中获取数据？
&emsp; 1）基于Receiver的方式  
&emsp; 这种方式使用Receiver来获取数据。Receiver是使用Kafka的高层次Consumer API来实现的。receiver从Kafka中获取的数据都是存储在Spark Executor的内存
中的，然后Spark Streaming启动的job会去处理那些数据。  
&emsp; 2）基于Direct的方式  
&emsp; 这种新的不基于Receiver的直接方式，是在Spark 1.3中引入的，从而能够确保更加健壮的机制。替代掉使用Receiver来接收数据后，这种方式会周期性地
查询Kafka，来获得每个topic+partition的最新的offset，从而定义每个batch的offset的范围。当处理数据的job启动时，就会使用Kafka的简单consumer api来
获取Kafka指定offset范围的数据。  

### 4、RDD创建有哪几种方式？  
&emsp; 1）使用程序中的集合创建rdd  
&emsp; 2）使用本地文件系统创建rdd  
&emsp; 3）使用hdfs创建rdd  
&emsp; 4）基于数据库db创建rdd  
&emsp; 5）基于Nosql创建rdd，如hbase  
&emsp; 6）基于s3创建rdd  
&emsp; 7）基于数据流，如socket创建rdd  

### 5、Spark并行度怎么设置比较合适？  
&emsp; spark并行度，每个core承载2~4个partition,如，32个core，那么64~128之间的并行度，也就是设置64~128个partion，并行读和数据规模无关，
只和内存使用量和cpu使用时间有关。  

### 6、Spark如何处理不能被序列化的对象？  
&emsp; 将不能序列化的内容封装成object。  

### 7、collect功能是什么，其底层是怎么实现的？  
&emsp; driver通过collect把集群中各个节点的内容收集过来汇总成结果，collect返回结果是Array类型的，collect把各个节点上的数据抓过来，
抓过来数据是Array型，collect对Array抓过来的结果进行合并，合并后Array中只有一个元素，是tuple类型（KV类型的）的。  

### 8、为什么Spark Application在没有获得足够的资源，job就开始执行了，可能会导致什么什么问题发生？  
&emsp; 会导致执行该job时候集群资源不足，导致执行job结束也没有分配足够的资源，分配了部分Executor，该job就开始执行task，应该是task的调度线程
和Executor资源申请是异步的；如果想等待申请完所有的资源再执行job的：  
&emsp; 需要将  
&emsp; spark.scheduler.maxRegisteredResourcesWaitingTime设置的很大；  
&emsp; spark.scheduler.minRegisteredResourcesRatio 设置为1，但是应该结合实际考虑  
&emsp; 否则很容易出现长时间分配不到资源，job一直不能运行的情况。  

### 9、map与flatMap的区别？  
&emsp; map：对RDD每个元素转换，文件中的每一行数据返回一个数组对象。   
&emsp; flatMap：对RDD每个元素转换，然后再扁平化。   
&emsp; 将所有的对象合并为一个对象，文件中的所有行数据仅返回一个数组对象，会抛弃值为null的值。  

### 10、Spark on Mesos中，什么是的粗粒度分配，什么是细粒度分配，各自的优点和缺点是什么？  
&emsp; 1）粗粒度：启动时就分配好资源， 程序启动，后续具体使用就使用分配好的资源，不需要再分配资源；优点：作业特别多时，资源复用率高，适合粗粒度；
缺点：容易资源浪费，假如一个job有1000个task，完成了999个，还有一个没完成，那么使用粗粒度，999个资源就会闲置在那里，资源浪费。  
&emsp; 2）细粒度分配：用资源的时候分配，用完了就立即回收资源，启动会麻烦一点，启动一次分配一次，会比较麻烦。  

### 11、driver的功能是什么？  
&emsp; 1）一个Spark作业运行时包括一个Driver进程，也是作业的主进程，具有main函数，并且有SparkContext的实例，是程序的入口点；  
&emsp; 2）功能：负责向集群申请资源，向master注册信息，负责了作业的调度，负责作业的解析、生成Stage并调度Task到Executor上。包括DAGScheduler，
TaskScheduler。  

### 12、Spark技术栈有哪些组件，每个组件都有什么功能，适合什么应用场景？  
&emsp; 可以画一个这样的技术栈图先，然后分别解释下每个组件的功能和场景  
&emsp; 1）Spark core：是其它组件的基础，spark的内核，主要包含：有向循环图、RDD、Lingage、Cache、broadcast等，并封装了底层通讯框架，
是Spark的基础。  
&emsp; 2）SparkStreaming是一个对实时数据流进行高通量、容错处理的流式处理系统，可以对多种数据源（如Kafka、Flume、Twitter、Zero和TCP 套接字）
进行类似Map、Reduce和Join等复杂操作，将流式计算分解成一系列短小的批处理作业。  
&emsp; 3）Spark sql：Shark是SparkSQL的前身，Spark SQL的一个重要特点是其能够统一处理关系表和RDD，使得开发人员可以轻松地使用SQL命令进行外部查询，
同时进行更复杂的数据分析。  
&emsp; 4）BlinkDB ：是一个用于在海量数据上运行交互式 SQL 查询的大规模并行查询引擎，它允许用户通过权衡数据精度来提升查询响应时间，其数据的精度
被控制在允许的误差范围内。  
&emsp; 5）MLBase是Spark生态圈的一部分专注于机器学习，让机器学习的门槛更低，让一些可能并不了解机器学习的用户也能方便地使用MLbase。
MLBase分为四部分：MLlib、MLI、ML Optimizer和MLRuntime。  
&emsp; 6）GraphX是Spark中用于图和图并行计算。  

### 13、Spark中Worker的主要工作是什么？  
&emsp; 主要功能：管理当前节点内存，CPU的使用状况，接收master分配过来的资源指令，通过ExecutorRunner启动程序分配任务，worker就类似于包工头，
管理分配新进程，做计算的服务，相当于process服务。  
&emsp; 需要注意的是：  
&emsp; 1）worker会不会汇报当前信息给master，worker心跳给master主要只有workid，它不会发送资源信息以心跳的方式给mater，master分配的时候就知道work，
只有出现故障的时候才会发送资源。   
&emsp; 2）worker不会运行代码，具体运行的是Executor是可以运行具体appliaction写的业务逻辑代码，操作代码的节点，它不会运行程序的代码的。  

### 14、Mapreduce和Spark的都是并行计算，那么他们有什么相同和区别？  
&emsp; 两者都是用mr模型来进行并行计算:  
&emsp; 1）hadoop的一个作业称为job，job里面分为map task和reduce task，每个task都是在自己的进程中运行的，当task结束时，进程也会结束。  
&emsp; 2）spark用户提交的任务成为application，一个application对应一个SparkContext，app中存在多个job，每触发一次action操作就会产生一个job。  
这些job可以并行或串行执行，每个job中有多个stage，stage是shuffle过程中DAGSchaduler通过RDD之间的依赖关系划分job而来的，每个stage里面有多个task，
组成taskset有TaskSchaduler分发到各个executor中执行，executor的生命周期是和app一样的，即使没有job运行也是存在的，所以task可以快速启动读取内存
进行计算。  
&emsp; 3）hadoop的job只有map和reduce操作，表达能力比较欠缺而且在mr过程中会重复的读写hdfs，造成大量的io操作，多个job需要自己管理关系。  
&emsp; 4）spark的迭代计算都是在内存中进行的，API中提供了大量的RDD操作如join，groupby等，而且通过DAG图可以实现良好的容错。  

### 15、RDD机制？  
&emsp; rdd分布式弹性数据集，简单的理解成一种数据结构，是spark框架上的通用货币。  所有算子都是基于rdd来执行的，不同的场景会有不同的rdd实现类，
但是都可以进行互相转换。rdd执行过程中会形成dag图，然后形成lineage保证容错性等。 从物理的角度来看rdd存储的是block和node之间的映射。  

### 16、什么是RDD宽依赖和窄依赖？  
&emsp; RDD和它依赖的parent RDD(s)的关系有两种不同的类型，即窄依赖（narrow dependency）和宽依赖（wide dependency）  
&emsp; 1）窄依赖指的是每一个parent RDD的Partition最多被子RDD的一个Partition使用  
&emsp; 2）宽依赖指的是多个子RDD的Partition会依赖同一个parent RDD的Partition  

### 17、cache和pesist的区别？  
&emsp; cache和persist都是用于将一个RDD进行缓存的，这样在之后使用的过程中就不需要重新计算了，可以大大节省程序运行时间  
&emsp; 1） cache只有一个默认的缓存级别MEMORY_ONLY ，cache调用了persist，而persist可以根据情况设置其它的缓存级别；  
&emsp; 2）executor执行的时候，默认60%做cache，40%做task操作，persist是最根本的函数，最底层的函数。  

### 18、 cache后面能不能接其他算子,它是不是action操作？  
&emsp; cache可以接其他算子，但是接了算子之后，起不到缓存应有的效果，因为会重新触发cache。  
&emsp; cache不是action操作。  

### 19、reduceByKey是不是action？  
&emsp; 不是，很多人都会以为是action，reduce rdd是action  

### 20、 RDD通过Linage（记录数据更新）的方式为何很高效？  
&emsp; 1）lazy记录了数据的来源，RDD是不可变的，且是lazy级别的，且RDD之间构成了链条，lazy是弹性的基石。由于RDD不可变，所以每次操作就产生新的rdd，
不存在全局修改的问题，控制难度下降，所有有计算链条将复杂计算链条存储下来，计算的时候从后往前回溯 900步是上一个stage的结束，要么就checkpoint。  
&emsp; 2）记录原数据，是每次修改都记录，代价很大如果修改一个集合，代价就很小，官方说rdd是粗粒度的操作，是为了效率，为了简化，每次都是操作数据集合，
写或者修改操作，都是基于集合的rdd的写操作是粗粒度的，rdd的读操作既可以是粗粒度的也可以是细粒度，读可以读其中的一条条的记录。  
&emsp; 3）简化复杂度，是高效率的一方面，写的粗粒度限制了使用场景如网络爬虫，现实世界中，大多数写是粗粒度的场景。  














