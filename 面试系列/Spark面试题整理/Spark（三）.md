## Spark面试题整理（三）  

### 1、为什么要进行序列化序列化？  
&emsp; 可以减少数据的体积，减少存储空间，高效存储和传输数据，不好的是使用的时候要反序列化，非常消耗CPU。  

### 2、Yarn中的container是由谁负责销毁的，在Hadoop Mapreduce中container可以复用么？  
&emsp; ApplicationMaster负责销毁，在Hadoop Mapreduce不可以复用，在spark on yarn程序container可以复用。  

### 3、提交任务时，如何指定Spark Application的运行模式？  
&emsp; 1）cluster模式：./spark-submit --class xx.xx.xx --master yarn --deploy-mode cluster xx.jar   
&emsp; 2）client模式：./spark-submit --class xx.xx.xx --master yarn --deploy-mode client xx.jar  

### 4、不启动Spark集群Master和work服务，可不可以运行Spark程序？  
&emsp; 可以，只要资源管理器第三方管理就可以，如由yarn管理，spark集群不启动也可以使用spark；spark集群启动的是work和master，这个其实就是资源管理框架，
yarn中的resourceManager相当于master，NodeManager相当于worker，做计算是Executor，和spark集群的work和manager可以没关系，归根接底还是JVM的运行，
只要所在的JVM上安装了spark就可以。  

### 5、spark on yarn Cluster 模式下，ApplicationMaster和driver是在同一个进程么？  
&emsp; 是，driver 位于ApplicationMaster进程中。该进程负责申请资源，还负责监控程序、资源的动态情况。  

### 6、运行在yarn中Application有几种类型的container？  
&emsp; 1）运行ApplicationMaster的Container：这是由ResourceManager（向内部的资源调度器）申请和启动的，用户提交应用程序时，
可指定唯一的ApplicationMaster所需的资源；  
&emsp; 2）运行各类任务的Container：这是由ApplicationMaster向ResourceManager申请的，并由ApplicationMaster与NodeManager通信以启动之。  

### 7、Executor启动时，资源通过哪几个参数指定？  
&emsp; 1）num-executors是executor的数量  
&emsp; 2）executor-memory 是每个executor使用的内存  
&emsp; 3）executor-cores 是每个executor分配的CPU  

### 8、为什么会产生yarn，解决了什么问题，有什么优势？  
&emsp; 1）为什么产生yarn，针对MRV1的各种缺陷提出来的资源管理框架  
&emsp; 2）解决了什么问题，有什么优势，参考这篇博文：http://www.aboutyun.com/forum.php?mod=viewthread&tid=6785  

### 9、一个task的map数量由谁来决定？  
&emsp; 一般情况下，在输入源是文件的时候，一个task的map数量由splitSize来决定的  
&emsp; 那么splitSize是由以下几个来决定的   
&emsp; &emsp; goalSize = totalSize / mapred.map.tasks  
&emsp; &emsp; inSize = max {mapred.min.split.size, minSplitSize}  
&emsp; &emsp; splitSize = max (minSize, min(goalSize, dfs.block.size))  
&emsp; 一个task的reduce数量，由partition决定。  

### 10、列出你所知道的调度器，说明其工作原理？  
&emsp; 1）FiFo schedular 默认的调度器  先进先出  
&emsp; 2）Capacity schedular  计算能力调度器  选择占用内存小  优先级高的  
&emsp; 3）Fair schedular 调度器  公平调度器  所有job 占用相同资源  

### 11、导致Executor产生FULL gc 的原因，可能导致什么问题？  
&emsp; 可能导致Executor僵死问题，海量数据的shuffle和数据倾斜等都可能导致full gc。以shuffle为例，伴随着大量的Shuffle写操作，JVM的新生代不断GC，
Eden Space写满了就往Survivor Space写，同时超过一定大小的数据会直接写到老生代，当新生代写满了之后，也会把老的数据搞到老生代，如果老生代空间不足了，
就触发FULL GC，还是空间不够，那就OOM错误了，此时线程被Blocked，导致整个Executor处理数据的进程被卡住。  

### 12、Spark累加器有哪些特点？  
&emsp; 1）累加器在全局唯一的，只增不减，记录全局集群的唯一状态；  
&emsp; 2）在exe中修改它，在driver读取；  
&emsp; 3）executor级别共享的，广播变量是task级别的共享两个application不可以共享累加器，但是同一个app不同的job可以共享。  

### 13、spark hashParitioner的弊端是什么？  
&emsp; HashPartitioner分区的原理很简单，对于给定的key，计算其hashCode，并除于分区的个数取余，如果余数小于0，则用余数+分区的个数，最后返回的值就是
这个key所属的分区ID；弊端是数据不均匀，容易导致数据倾斜，极端情况下某几个分区会拥有rdd的所有数据。  

### 14、RangePartitioner分区的原理？  
&emsp; RangePartitioner分区则尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，也就是说一个分区中的元素肯定都是比另一个分区内的元素小
或者大；但是分区内的元素是不能保证顺序的。简单的说就是将一定范围内的数映射到某一个分区内。其原理是水塘抽样。  

### 15、rangePartioner分区器特点？  
&emsp; rangePartioner尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，一个分区中的元素肯定都是比另一个分区内的元素小或者大；
但是分区内的元素是不能保证顺序的。简单的说就是将一定范围内的数映射到某一个分区内。RangePartitioner作用：将一定范围内的数映射到某一个分区内，
在实现中，分界的算法尤为重要。算法对应的函数是rangeBounds。  

### 16、如何理解Standalone模式下，Spark资源分配是粗粒度的？  
&emsp; spark默认情况下资源分配是粗粒度的，也就是说程序在提交时就分配好资源，后面执行的时候使用分配好的资源，除非资源出现了故障才会重新分配。
比如Spark shell启动，已提交，一注册，哪怕没有任务，worker都会分配资源给executor。  

### 17、union操作是产生宽依赖还是窄依赖？  
&emsp; 产生窄依赖。  

### 18、窄依赖父RDD的partition和子RDD的parition是不是都是一对一的关系？  
&emsp; 不一定，除了一对一的窄依赖，还包含一对固定个数的窄依赖（就是对父RDD的依赖的Partition的数量不会随着RDD数量规模的改变而改变），
比如join操作的每个partiion仅仅和已知的partition进行join，这个join操作是窄依赖，依赖固定数量的父rdd，因为是确定的partition关系。  

### 19、Hadoop中，Mapreduce操作的mapper和reducer阶段相当于spark中的哪几个算子？  
&emsp; 相当于spark中的map算子和reduceByKey算子，当然还是有点区别的,MR会自动进行排序的，spark要看你用的是什么partitioner。  

### 20、什么是shuffle，以及为什么需要shuffle？  
&emsp; shuffle中文翻译为洗牌，需要shuffle的原因是：某种具有共同特征的数据汇聚到一个计算节点上进行计算。  

















