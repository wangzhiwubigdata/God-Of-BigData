## Hadoop面试题总结（三）——MapReduce  

### 1、谈谈Hadoop序列化和反序列化及自定义bean对象实现序列化?  
1）序列化和反序列化  
&emsp; （1）序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。   
&emsp; （2）反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。  
&emsp; （3）Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。  
2）自定义bean对象要想序列化传输步骤及注意事项：  
&emsp; （1）必须实现Writable接口  
&emsp; （2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造  
&emsp; （3）重写序列化方法  
&emsp; （4）重写反序列化方法  
&emsp; （5）注意反序列化的顺序和序列化的顺序完全一致  
&emsp; （6）要想把结果显示在文件中，需要重写toString()，且用"\t"分开，方便后续用  
&emsp; （7）如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序  

### 2、FileInputFormat切片机制（☆☆☆☆☆）  
job提交流程源码详解  
&emsp; waitForCompletion()  
&emsp; submit();  
&emsp; // 1、建立连接  
&emsp; &emsp; connect();   
&emsp; &emsp; &emsp; // 1）创建提交job的代理  
&emsp; &emsp; &emsp; new Cluster(getConfiguration());  
&emsp; &emsp; &emsp; &emsp; // （1）判断是本地yarn还是远程  
&emsp; &emsp; &emsp; &emsp; initialize(jobTrackAddr, conf);  
&emsp; // 2、提交job  
&emsp; submitter.submitJobInternal(Job.this, cluster)  
&emsp; &emsp; // 1）创建给集群提交数据的Stag路径  
&emsp; &emsp; Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);  
&emsp; &emsp; // 2）获取jobid ，并创建job路径  
&emsp; &emsp; JobID jobId = submitClient.getNewJobID();  
&emsp; &emsp; // 3）拷贝jar包到集群  
&emsp; &emsp; copyAndConfigureFiles(job, submitJobDir);  
&emsp; &emsp; rUploader.uploadFiles(job, jobSubmitDir);  
&emsp; &emsp; // 4）计算切片，生成切片规划文件  
&emsp; &emsp; writeSplits(job, submitJobDir);  
&emsp; &emsp; maps = writeNewSplits(job, jobSubmitDir);  
&emsp; &emsp; input.getSplits(job);  
&emsp; &emsp; // 5）向Stag路径写xml配置文件  
&emsp; &emsp; writeConf(conf, submitJobFile);  
&emsp; &emsp; conf.writeXml(out);  
&emsp; &emsp; // 6）提交job,返回提交状态  
&emsp; &emsp; status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());  

### 3、在一个运行的Hadoop 任务中，什么是InputSplit？（☆☆☆☆☆）
FileInputFormat源码解析(input.getSplits(job))  
（1）找到你数据存储的目录。  
（2）开始遍历处理（规划切片）目录下的每一个文件。  
（3）遍历第一个文件ss.txt。  
&emsp; a）获取文件大小fs.sizeOf(ss.txt);。  
&emsp; b）计算切片大小computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M。  
&emsp; c）**默认情况下，切片大小=blocksize**。  
&emsp; d）开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M（每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，**不大于1.1倍就划分一块切片**）。   
&emsp; e）将切片信息写到一个切片规划文件中。  
&emsp; f）整个切片的核心过程在getSplit()方法中完成。  
&emsp; g）数据切片只是在逻辑上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。  
&emsp; h）注意：block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。  
（4）**提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数**。  

### 4、如何判定一个job的map和reduce的数量?
1）map数量  
&emsp; splitSize=max{minSize,min{maxSize,blockSize}}  
&emsp; map数量由处理的数据分成的block数量决定default_num = total_size / split_size;  
2）reduce数量  
&emsp; reduce的数量job.setNumReduceTasks(x);x 为reduce的数量。不设置的话默认为 1。  

### 5、 Maptask的个数由什么决定？
&emsp; 一个job的map阶段MapTask并行度（个数），由客户端提交job时的切片个数决定。  

### 6、MapTask和ReduceTask工作机制（☆☆☆☆☆）（也可回答MapReduce工作原理）
**MapTask工作机制**
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/MR-Pics/MapTask%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

（1）Read阶段：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。  
（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。  
（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。  
（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。  
（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。  

**ReduceTask工作机制**
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/MR-Pics/ReduceTask%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。  
（2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。  
（3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。 由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。  
（4）Reduce阶段：reduce()函数将计算结果写到HDFS上。  

### 7、描述mapReduce有几种排序及排序发生的阶段（☆☆☆☆☆）
1）排序的分类：  
&emsp; （1）部分排序：  
&emsp; &emsp; MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。  
&emsp; （2）全排序：  
&emsp; &emsp; 如何用Hadoop产生一个全局排序的文件？最简单的方法是使用一个分区。但该方法在处理大型文件时效率极低，因为一台机器必须处理所有输出文件，从而完全丧失了MapReduce所提供的并行架构。  
&emsp; &emsp; 替代方案：首先创建一系列排好序的文件；其次，串联这些文件；最后，生成一个全局排序的文件。主要思路是使用一个分区来描述输出的全局排序。例如：可以为待分析文件创建3个分区，在第一分区中，记录的单词首字母a-g，第二分区记录单词首字母h-n, 第三分区记录单词首字母o-z。  
&emsp; （3）辅助排序：（GroupingComparator分组）  
&emsp; &emsp; Mapreduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大多数MapReduce程序会避免让reduce函数依赖于值的排序。但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。  
&emsp; （4）二次排序：  
&emsp; &emsp; 在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。  
2）自定义排序WritableComparable  
&emsp; bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序  
&emsp; &emsp; @Override  
&emsp; &emsp; public int compareTo(FlowBean o) {  
&emsp; &emsp; &emsp; // 倒序排列，从大到小  
&emsp; &emsp; &emsp; return this.sumFlow > o.getSumFlow() ? -1 : 1;  
&emsp; &emsp; }  
3）排序发生的阶段：  
&emsp; （1）一个是在map side发生在spill后partition前。  
&emsp; （2）一个是在reduce side发生在copy后 reduce前。  

### 8、描述mapReduce中shuffle阶段的工作流程，如何优化shuffle阶段（☆☆☆☆☆）
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/MR-Pics/mapReduce%E4%B8%ADshuffle%E9%98%B6%E6%AE%B5%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png"/>  

分区，排序，溢写，拷贝到对应reduce机器上，增加combiner，压缩溢写的文件。  

### 9、描述mapReduce中combiner的作用是什么，一般使用情景，哪些情况不需要，及和reduce的区别？
1）Combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量。  
2）Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟reducer的输入kv类型要对应起来。  
3）Combiner和reducer的区别在于运行的位置。  
&emsp; Combiner是在每一个maptask所在的节点运行；  
&emsp; Reducer是接收全局所有Mapper的输出结果。  

<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/MR-Pics/mapReduce%E4%B8%ADcombiner%E4%BD%9C%E7%94%A8.png"/>

### 10、如果没有定义partitioner，那数据在被送达reducer前是如何被分区的？
&emsp; 如果没有自定义的 partitioning，则默认的 partition 算法，即根据每一条数据的 key 的 hashcode 值摸运算（%）reduce 的数量，得到的数字就是“分区号“。  

### 11、MapReduce 出现单点负载多大，怎么负载平衡？ （☆☆☆☆☆）
&emsp; 通过Partitioner实现  

### 12、MapReduce 怎么实现 TopN？ （☆☆☆☆☆）
&emsp; 可以自定义groupingcomparator，对结果进行最大值排序，然后再reduce输出时，控制只输出前n个数。就达到了topn输出的目的。  

### 13、Hadoop的缓存机制（Distributedcache）（☆☆☆☆☆）
&emsp; 分布式缓存一个最重要的应用就是在进行join操作的时候，如果一个表很大，另一个表很小，我们就可以将这个小表进行广播处理，即每个计算节点上都存一份，然后进行map端的连接操作，经过我的实验验证，这种情况下处理效率大大高于一般的reduce端join，广播处理就运用到了分布式缓存的技术。  
&emsp; DistributedCache将拷贝缓存的文件到Slave节点在任何Job在节点上执行之前，文件在每个Job中只会被拷贝一次，缓存的归档文件会被在Slave节点中解压缩。将本地文件复制到HDFS中去，接着Client会通过addCacheFile() 和addCacheArchive()方法告诉DistributedCache在HDFS中的位置。当文件存放到文地时，JobClient同样获得DistributedCache来创建符号链接，其形式为文件的URI加fragment标识。当用户需要获得缓存中所有有效文件的列表时，JobConf 的方法 getLocalCacheFiles() 和getLocalArchives()都返回一个指向本地文件路径对象数组。  

### 14、如何使用mapReduce实现两个表的join?（☆☆☆☆☆）
&emsp; 1）reduce side join : 在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）,比如：tag=0 表示来自文件File1，tag=2 表示来自文件File2。  
&emsp; 2）map side join : Map side join 是针对以下场景进行的优化：两个待连接表中，有一个表非常大，而另一个表非常小，以至于小表可以直接存放到内存中。这样，我们可以将小表复制多份，让每个map task 内存中存在一份（比如存放到hash table 中），然后只扫描大表：对于大表中的每一条记录key/value，在hash table 中查找是否有相同的key 的记录，如果有，则连接后输出即可。  

### 15、什么样的计算不能用mr来提速？
&emsp; 1）数据量很小。  
&emsp; 2）繁杂的小文件。  
&emsp; 3）索引是更好的存取机制的时候。  
&emsp; 4）事务处理。  
&emsp; 5）只有一台机器的时候。  

### 16、ETL是哪三个单词的缩写
&emsp; Extraction-Transformation-Loading的缩写，中文名称为数据提取、转换和加载。  












