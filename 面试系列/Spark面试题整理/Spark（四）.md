## Spark面试题（四）  

### 1、Spark中的HashShufle的有哪些不足？  
&emsp; 1）shuffle产生海量的小文件在磁盘上，此时会产生大量耗时的、低效的IO操作；  
&emsp; 2）容易导致内存不够用，由于内存需要保存海量的文件操作句柄和临时缓存信息，如果数据处理规模比较大的话，容易出现OOM；  
&emsp; 3）容易出现数据倾斜，导致OOM。  

### 2、 conslidate是如何优化Hash shuffle时在map端产生的小文件？  
&emsp; 1）conslidate为了解决Hash Shuffle同时打开过多文件导致Writer handler内存使用过大以及产生过多文件导致大量的随机读写带来的低效磁盘IO；  
&emsp; 2）conslidate根据CPU的个数来决定每个task shuffle map端产生多少个文件，假设原来有10个task，100个reduce，每个CPU有10个CPU，那么
使用hash shuffle会产生10*100=1000个文件，conslidate产生10*10=100个文件  
&emsp; 注意：conslidate部分减少了文件和文件句柄，并行读很高的情况下（task很多时）还是会很多文件。  

### 3、spark.default.parallelism这个参数有什么意义，实际生产中如何设置？  
&emsp; 1）参数用于设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的Spark作业性能；  
&emsp; 2）很多人都不会设置这个参数，会使得集群非常低效，你的cpu，内存再多，如果task始终为1，那也是浪费，
spark官网建议task个数为CPU的核数*executor的个数的2~3倍。  

### 4、spark.shuffle.memoryFraction参数的含义，以及优化经验？  
&emsp; 1）spark.shuffle.memoryFraction是shuffle调优中 重要参数，shuffle从上一个task拉去数据过来，要在Executor进行聚合操作，
聚合操作时使用Executor内存的比例由该参数决定，默认是20%如果聚合时数据超过了该大小，那么就会spill到磁盘，极大降低性能；  
&emsp; 2）如果Spark作业中的RDD持久化操作较少，shuffle操作较多时，建议降低持久化操作的内存占比，提高shuffle操作的内存占比比例，
避免shuffle过程中数据过多时内存不够用，必须溢写到磁盘上，降低了性能。此外，如果发现作业由于频繁的gc导致运行缓慢，意味着task执行用户代码的内存不够用，
那么同样建议调低这个参数的值。  

### 5、Spark中standalone模式特点，有哪些优点和缺点？  
1）特点：  
&emsp; （1）standalone是master/slave架构，集群由Master与Worker节点组成，程序通过与Master节点交互申请资源，Worker节点启动Executor运行；  
&emsp; （2）standalone调度模式使用FIFO调度方式；  
&emsp; （3）无依赖任何其他资源管理系统，Master负责管理集群资源。   
2）优点：  
&emsp; （1）部署简单；  
&emsp; （2）不依赖其他资源管理系统。  
3）缺点：  
&emsp; （1）默认每个应用程序会独占所有可用节点的资源，当然可以通过spark.cores.max来决定一个应用可以申请的CPU cores个数；  
&emsp; （2）可能有单点故障，需要自己配置master HA。  

### 6、FIFO调度模式的基本原理、优点和缺点？  
&emsp; 基本原理：按照先后顺序决定资源的使用，资源优先满足最先来的job。第一个job优先获取所有可用的资源，接下来第二个job再获取剩余资源。  
以此类推，如果第一个job没有占用所有的资源，那么第二个job还可以继续获取剩余资源，这样多个job可以并行运行，如果第一个job很大，占用所有资源，
则第二job就需要等待，等到第一个job释放所有资源。  
&emsp; 优点和缺点：  
&emsp; &emsp; 1）适合长作业，不适合短作业；  
&emsp; &emsp; 2）适合CPU繁忙型作业（计算时间长，相当于长作业），不利于IO繁忙型作业（计算时间短，相当于短作业）。  

### 7、FAIR调度模式的优点和缺点？  
&emsp; 所有的任务拥有大致相当的优先级来共享集群资源，spark多以轮训的方式为任务分配资源，不管长任务还是端任务都可以获得资源，并且获得不错的响应时间，
对于短任务，不会像FIFO那样等待较长时间了，通过参数spark.scheduler.mode 为FAIR指定。  

### 8、CAPCACITY调度模式的优点和缺点？  
1）原理：  
&emsp; 计算能力调度器支持多个队列，每个队列可配置一定的资源量，每个队列采用 FIFO 调度策略，为了防止同一个用户的作业独占队列中的资源，该调度器会对
同一用户提交的作业所占资源量进行限定。调度时，首先按以下策略选择一个合适队列：计算每个队列中正在运行的任务数与其应该分得的计算资源之间的
比值(即比较空闲的队列)，选择一个该比值最小的队列；然后按以下策略选择该队列中一个作业：按照作业优先级和提交时间顺序选择，
同时考虑用户资源量限制和内存限制  
2）优点：  
&emsp; （1）计算能力保证。支持多个队列，某个作业可被提交到某一个队列中。每个队列会配置一定比例的计算资源，且所有提交到队列中的作业
共享该队列中的资源；   
&emsp; （2）灵活性。空闲资源会被分配给那些未达到资源使用上限的队列，当某个未达到资源的队列需要资源时，一旦出现空闲资源资源，便会分配给他们；  
&emsp; （3）支持优先级。队列支持作业优先级调度（默认是FIFO）；  
&emsp; （4）多重租赁。综合考虑多种约束防止单个作业、用户或者队列独占队列或者集群中的资源；  
&emsp; （5）基于资源的调度。 支持资源密集型作业，允许作业使用的资源量高于默认值，进而可容纳不同资源需求的作业。不过，当前仅支持内存资源的调度。  

### 9、常见的数压缩方式，你们生产集群采用了什么压缩方式，提升了多少效率？  
1）数据压缩，大片连续区域进行数据存储并且存储区域中数据重复性高的状况下，可以使用适当的压缩算法。数组，对象序列化后都可以使用压缩，数更紧凑，
减少空间开销。常见的压缩方式有snappy，LZO，gz等  
2）Hadoop生产环境常用的是snappy压缩方式（使用压缩，实际上是CPU换IO吞吐量和磁盘空间，所以如果CPU利用率不高，不忙的情况下，
可以大大提升集群处理效率）。snappy压缩比一般20%~30%之间，并且压缩和解压缩效率也非常高（参考数据如下）：  
&emsp; （1）GZIP的压缩率最高，但是其实CPU密集型的，对CPU的消耗比其他算法要多，压缩和解压速度也慢；  
&emsp; （2）LZO的压缩率居中，比GZIP要低一些，但是压缩和解压速度明显要比GZIP快很多，其中解压速度快的更多；  
&emsp; （3）Zippy/Snappy的压缩率最低，而压缩和解压速度要稍微比LZO要快一些。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E9%9D%A2%E8%AF%95%E9%A2%98Pics/%E5%8E%8B%E7%BC%A9%E6%96%B9%E5%BC%8F.png"/>  
<p align="center">
</p>
</p>  

&emsp; 提升了多少效率可以从2方面回答：1）数据存储节约多少存储，2）任务执行消耗时间节约了多少，可以举个实际例子展开描述。  

### 10、使用scala代码实现WordCount？  
&emsp; val conf = new SparkConf()   
&emsp; val sc = new SparkContext(conf)   
&emsp; val line = sc.textFile("xxxx.txt") line.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_). collect().foreach(println) sc.stop()  

### 11、Spark RDD 和 MapReduce2的区别？  
&emsp; 1）mr2只有2个阶段，数据需要大量访问磁盘，数据来源相对单一 ,spark RDD ,可以无数个阶段进行迭代计算，数据来源非常丰富，数据落地介质也
非常丰富spark计算基于内存；  
&emsp; 2）MapReduce2需要频繁操作磁盘IO，需要大家明确的是如果是SparkRDD的话，你要知道每一种数据来源对应的是什么，RDD从数据源加载数据，
将数据放到不同的partition针对这些partition中的数据进行迭代式计算计算完成之后，落地到不同的介质当中。  

### 12、spark和Mapreduce快？ 为什么快呢？ 快在哪里呢？  
&emsp; Spark更加快的主要原因有几点：  
&emsp; 1）基于内存计算，减少低效的磁盘交互；  
&emsp; 2）高效的调度算法，基于DAG；  
&emsp; 3）容错机制Lingage，主要是DAG和Lianage，即使spark不使用内存技术，也大大快于mapreduce。  

### 13、Spark sql为什么比hive快呢？  
&emsp; 计算引擎不一样，一个是spark计算模型，一个是mapreudce计算模型。  

### 14、RDD的数据结构是怎么样的？  
一个RDD对象，包含如下5个核心属性。  
&emsp; 1）一个分区列表，每个分区里是RDD的部分数据（或称数据块）。  
&emsp; 2）一个依赖列表，存储依赖的其他RDD。  
&emsp; 3）一个名为compute的计算函数，用于计算RDD各分区的值。  
&emsp; 4）分区器（可选），用于键/值类型的RDD，比如某个RDD是按散列来分区。  
&emsp; 5）计算各分区时优先的位置列表（可选），比如从HDFS上的文件生成RDD时，RDD分区的位置优先选择数据所在的节点，这样可以避免数据移动带来的开销。  

### 15、RDD算子里操作一个外部map，比如往里面put数据，然后算子外再遍历map，会有什么问题吗？  
&emsp; 频繁创建额外对象，容易oom。  

### 16、 说说你对Hadoop生态的认识。  
hadoop生态主要分为三大类型，1）分布式文件系统，2）分布式计算引擎，3）周边工具   
&emsp; 1）分布式系统：HDFS，hbase  
&emsp; 2）分布式计算引擎：Spark，MapReduce  
&emsp; 3）周边工具：如zookeeper，pig，hive，oozie，sqoop，ranger，kafka等  

### 17、hbase region多大会分区，spark读取hbase数据是如何划分partition的？  
&emsp; region超过了hbase.hregion.max.filesize这个参数配置的大小就会自动裂分，默认值是1G。  
&emsp; 默认情况下，hbase有多少个region，Spark读取时就会有多少个partition  


















