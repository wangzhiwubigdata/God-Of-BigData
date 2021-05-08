## Hadoop面试题（一）

### 1、集群的最主要瓶颈  
&emsp; 磁盘IO  

### 2、Hadoop运行模式  
&emsp; 单机版、伪分布式模式、完全分布式模式  

### 3、Hadoop生态圈的组件并做简要描述  
&emsp; 1）Zookeeper：是一个开源的分布式应用程序协调服务,基于zookeeper可以实现同步服务，配置维护，命名服务。  
&emsp; 2）Flume：一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。   
&emsp; 3）Hbase：是一个分布式的、面向列的开源数据库, 利用Hadoop HDFS作为其存储系统。   
&emsp; 4）Hive：基于Hadoop的一个数据仓库工具，可以将结构化的数据档映射为一张数据库表，并提供简单的sql 查询功能，可以将sql语句转换为MapReduce任务进行运行。   
&emsp; 5）Sqoop：将一个关系型数据库中的数据导进到Hadoop的 HDFS中，也可以将HDFS的数据导进到关系型数据库中。  

### 4、解释“hadoop”和“hadoop 生态系统”两个概念  
&emsp; Hadoop是指Hadoop框架本身；hadoop生态系统，不仅包含hadoop，还包括保证hadoop框架正常高效运行其他框架，比如zookeeper、Flume、Hbase、Hive、Sqoop等辅助框架。  

### 5、请列出正常工作的Hadoop集群中Hadoop都分别需要启动哪些进程，它们的作用分别是什么?  
&emsp; 1）NameNode：它是hadoop中的主服务器，管理文件系统名称空间和对集群中存储的文件的访问，保存有metadate。  
&emsp; 2）SecondaryNameNode：它不是namenode的冗余守护进程，而是提供周期检查点和清理任务。帮助NN合并editslog，减少NN启动时间。  
&emsp; 3）DataNode：它负责管理连接到节点的存储（一个集群中可以有多个节点）。每个存储数据的节点运行一个datanode守护进程。  
&emsp; 4）ResourceManager（JobTracker）：JobTracker负责调度DataNode上的工作。每个DataNode有一个TaskTracker，它们执行实际工作。  
&emsp; 5）NodeManager：（TaskTracker）执行任务。  
&emsp; 6）DFSZKFailoverController：高可用时它负责监控NN的状态，并及时的把状态信息写入ZK。它通过一个独立线程周期性的调用NN上的一个特定接口来获取NN的健康状态。FC也有选择谁作为Active NN的权利，因为最多只有两个节点，目前选择策略还比较简单（先到先得，轮换）。  
&emsp; 7）JournalNode：高可用情况下存放namenode的editlog文件。  
