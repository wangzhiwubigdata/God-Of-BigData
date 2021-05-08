## Hadoop面试题（四）——YARN  

### 1、简述hadoop1与hadoop2 的架构异同  
&emsp; 1）加入了yarn解决了资源调度的问题。  
&emsp; 2）加入了对zookeeper的支持实现比较可靠的高可用。  
    
### 2、为什么会产生 yarn,它解决了什么问题，有什么优势？  
&emsp; 1）Yarn最主要的功能就是解决运行的用户程序与yarn框架完全解耦。  
&emsp; 2）Yarn上可以运行各种类型的分布式运算程序（mapreduce只是其中的一种），比如mapreduce、storm程序，spark程序……  

### 3、HDFS的数据压缩算法?（☆☆☆☆☆）  
&emsp; Hadoop中常用的压缩算法有**bzip2、gzip、lzo、snappy**，其中lzo、snappy需要操作系统安装native库才可以支持。  
&emsp; 数据可以压缩的位置如下所示。  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/YARN-Pics/MapReduce%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9.png"/>  
<p align="center">
</p>
</p>  

&emsp; **企业开发用的比较多的是snappy**。  

### 4、Hadoop的调度器总结（☆☆☆☆☆）  
（1）默认的调度器FIFO  
&emsp; Hadoop中默认的调度器，它先按照作业的优先级高低，再按照到达时间的先后选择被执行的作业。  
（2）计算能力调度器Capacity Scheduler  
&emsp; 支持多个队列，每个队列可配置一定的资源量，每个队列采用FIFO调度策略，为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。调度时，首先按以下策略选择一个合适队列：计算每个队列中正在运行的任务数与其应该分得的计算资源之间的比值，选择一个该比值最小的队列；然后按以下策略选择该队列中一个作业：按照作业优先级和提交时间顺序选择，同时考虑用户资源量限制和内存限制。   
（3）公平调度器Fair Scheduler  
&emsp; 同计算能力调度器类似，支持多队列多用户，每个队列中的资源量可以配置，同一队列中的作业公平共享队列中所有资源。实际上，Hadoop的调度器远不止以上三种，最近，出现了很多针对新型应用的Hadoop调度器。  

### 5、MapReduce 2.0 容错性（☆☆☆☆☆）  
1）MRAppMaster容错性  
&emsp; 一旦运行失败，由YARN的ResourceManager负责重新启动，最多重启次数可由用户设置，默认是2次。一旦超过最高重启次数，则作业运行失败。   
2）Map Task/Reduce  
&emsp; Task Task周期性向MRAppMaster汇报心跳；一旦Task挂掉，则MRAppMaster将为之重新申请资源，并运行之。最多重新运行次数可由用户设置，默认4次。  

### 6、mapreduce推测执行算法及原理（☆☆☆☆☆）  
1）作业完成时间取决于最慢的任务完成时间  
&emsp; 一个作业由若干个Map 任务和Reduce 任务构成。因硬件老化、软件Bug 等，某些任务可能运行非常慢。  
&emsp; 典型案例：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？  
2）推测执行机制  
&emsp; 发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。  
3）不能启用推测执行机制情况  
&emsp; （1）任务间存在严重的负载倾斜；  
&emsp; （2）特殊任务，比如任务向数据库中写数据。  
4）算法原理  
&emsp; 假设某一时刻，任务T的执行进度为progress，则可通过一定的算法推测出该任务的最终完成时刻estimateEndTime。另一方面，如果此刻为该任务启动一个备份任务，则可推断出它可能的完成时刻estimateEndTime`,于是可得出以下几个公式：  
&emsp; &emsp; estimateEndTime=estimatedRunTime+taskStartTime  
&emsp; &emsp; estimatedRunTime=(currentTimestamp-taskStartTime)/progress  
&emsp; &emsp; estimateEndTime`= currentTimestamp+averageRunTime  
&emsp; 其中，currentTimestamp为当前时刻；taskStartTime为该任务的启动时刻；averageRunTime为已经成功运行完成的任务的平均运行时间。这样，MRv2总是选择（estimateEndTime- estimateEndTime·）差值最大的任务，并为之启动备份任务。为了防止大量任务同时启动备份任务造成的资源浪费，MRv2为每个作业设置了同时启动的备份任务数目上限。  
&emsp; 推测执行机制实际上采用了经典的算法优化方法：以空间换时间，它同时启动多个相同任务处理相同的数据，并让这些任务竞争以缩短数据处理时间。显然，这种方法需要占用更多的计算资源。在集群资源紧缺的情况下，应合理使用该机制，争取在多用少量资源的情况下，减少作业的计算时间。  




