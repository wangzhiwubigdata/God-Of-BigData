## Hive面试题整理（一）

### 1、Hive表关联查询，如何解决数据倾斜的问题？（☆☆☆☆☆）
&emsp; 1）倾斜原因：map输出数据按key Hash的分配到reduce中，由于key分布不均匀、业务数据本身的特、建表时考虑不周、等原因造成的reduce 上的数据量差异过大。  
&emsp; （1）key分布不均匀;  
&emsp; （2）业务数据本身的特性;  
&emsp; （3）建表时考虑不周;  
&emsp; （4）某些SQL语句本身就有数据倾斜;  
&emsp; 如何避免：对于key为空产生的数据倾斜，可以对其赋予一个随机值。  
&emsp; 2）解决方案  
&emsp; （1）参数调节：  
&emsp; &emsp; hive.map.aggr = true  
&emsp; &emsp; hive.groupby.skewindata=true  
&emsp; 有数据倾斜的时候进行负载均衡，当选项设定位true,生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个Reduce中），最后完成最终的聚合操作。  
&emsp; （2）SQL 语句调节：  
&emsp; ① 选用join key分布最均匀的表作为驱动表。做好列裁剪和filter操作，以达到两表做join 的时候，数据量相对变小的效果。  
&emsp; ② 大小表Join：  
&emsp; &emsp; 使用map join让小的维度表（1000 条以下的记录条数）先进内存。在map端完成reduce。  
&emsp; ③ 大表Join大表：  
&emsp; &emsp; 把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null 值关联不上，处理后并不影响最终结果。  
&emsp; ④ count distinct大量相同特殊值:  
&emsp; &emsp; count distinct 时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在最后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union。  

### 2、Hive的HSQL转换为MapReduce的过程？（☆☆☆☆☆）
&emsp; HiveSQL ->AST(抽象语法树) -> QB(查询块) ->OperatorTree（操作树）->优化后的操作树->mapreduce任务树->优化后的mapreduce任务树  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hive%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HSQL%E8%BD%ACMR%EF%BC%881%EF%BC%89.png"/>  
<p align="center">
</p>
</p>  

<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hive%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HSQL%E8%BD%ACMR%EF%BC%882%EF%BC%89.png"/>  
<p align="center">
</p>
</p>  

&emsp; 过程描述如下：  
&emsp; &emsp; SQL Parser：Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree；  
&emsp; &emsp; Semantic Analyzer：遍历AST Tree，抽象出查询的基本组成单元QueryBlock；  
&emsp; &emsp; Logical plan：遍历QueryBlock，翻译为执行操作树OperatorTree；  
&emsp; &emsp; Logical plan optimizer: 逻辑层优化器进行OperatorTree变换，合并不必要的ReduceSinkOperator，减少shuffle数据量；  
&emsp; &emsp; Physical plan：遍历OperatorTree，翻译为MapReduce任务；  
&emsp; &emsp; Logical plan optimizer：物理层优化器进行MapReduce任务的变换，生成最终的执行计划。  

### 3、Hive底层与数据库交互原理？（☆☆☆☆☆）
&emsp; 由于Hive的元数据可能要面临不断地更新、修改和读取操作，所以它显然不适合使用Hadoop文件系统进行存储。目前Hive将元数据存储在RDBMS中，比如存储在MySQL、Derby中。元数据信息包括：存在的表、表的列、权限和更多的其他信息。  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hive%E9%9D%A2%E8%AF%95%E9%A2%98Pics/Hive%E5%BA%95%E5%B1%82%E4%B8%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%A4%E4%BA%92%E5%8E%9F%E7%90%86.png"/>  
<p align="center">
</p>
</p>  

### 4、Hive的两张表关联，使用MapReduce怎么实现？（☆☆☆☆☆）
&emsp; 如果其中有一张表为小表，直接使用map端join的方式（map端加载小表）进行聚合。  
&emsp; 如果两张都是大表，那么采用联合key，联合key的第一个组成部分是join on中的公共字段，第二部分是一个flag，0代表表A，1代表表B，由此让Reduce区分客户信息和订单信息；在Mapper中同时处理两张表的信息，将join on公共字段相同的数据划分到同一个分区中，进而传递到一个Reduce中，然后在Reduce中实现聚合。  

### 5、请谈一下Hive的特点，Hive和RDBMS有什么异同？
&emsp; hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供完整的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析，但是Hive不支持实时查询。  
&emsp; Hive与关系型数据库的区别：  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/Hive%E9%9D%A2%E8%AF%95%E9%A2%98Pics/Hive%E5%92%8CRDBMS%E5%BC%82%E5%90%8C.png"/>  
<p align="center">
</p>
</p>  

### 6、请说明hive中 Sort By，Order By，Cluster By，Distrbute By各代表什么意思？
&emsp; order by：会对输入做全局排序，因此只有一个reducer（多个reducer无法保证全局有序）。只有一个reducer，会导致当输入规模较大时，需要较长的计算时间。  
&emsp; sort by：不是全局排序，其在数据进入reducer前完成排序。  
&emsp; distribute by：按照指定的字段对数据进行划分输出到不同的reduce中。  
&emsp; cluster by：除了具有 distribute by 的功能外还兼具 sort by 的功能。  

### 7、写出hive中split、coalesce及collect_list函数的用法（可举例）？
&emsp; split将字符串转化为数组，即：split('a,b,c,d' , ',') ==> ["a","b","c","d"]。  
&emsp; coalesce(T v1, T v2, …) 返回参数中的第一个非空值；如果所有值都为 NULL，那么返回NULL。  
&emsp; collect_list列出该字段所有的值，不去重 => select collect_list(id) from table。  

### 8、Hive有哪些方式保存元数据，各有哪些特点？
&emsp; Hive支持三种不同的元存储服务器，分别为：内嵌式元存储服务器、本地元存储服务器、远程元存储服务器，每种存储方式使用不同的配置参数。   
&emsp; 内嵌式元存储主要用于单元测试，在该模式下每次只有一个进程可以连接到元存储，Derby是内嵌式元存储的默认数据库。  
&emsp; 在本地模式下，每个Hive客户端都会打开到数据存储的连接并在该连接上请求SQL查询。  
&emsp; 在远程模式下，所有的Hive客户端都将打开一个到元数据服务器的连接，该服务器依次查询元数据，元数据服务器和客户端之间使用Thrift协议通信。  

### 9、Hive内部表和外部表的区别？
&emsp; 创建表时：创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。  
&emsp; 删除表时：在删除表的时候，内部表的元数据和数据会被一起删除， 而外部表只删除元数据，不删除数据。这样外部表相对来说更加安全些，数据组织也更加灵活，方便共享源数据。  

### 10、Hive 中的压缩格式TextFile、SequenceFile、RCfile 、ORCfile各有什么区别？
&emsp; **1、TextFile**  
&emsp; 默认格式，**存储方式为行存储，数据不做压缩，磁盘开销大，数据解析开销大**。可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但使用这种方式，压缩后的文件不支持split，Hive不会对数据进行切分，从而无法对数据进行并行操作。并且在反序列化过程中，必须逐个字符判断是不是分隔符和行结束符，因此反序列化开销会比SequenceFile高几十倍。  
&emsp; **2、SequenceFile**  
&emsp; SequenceFile是Hadoop API提供的一种二进制文件支持，**存储方式为行存储，其具有使用方便、可分割、可压缩的特点**。  
&emsp; SequenceFile支持三种压缩选择：NONE，RECORD，BLOCK。Record压缩率低，**一般建议使用BLOCK压缩**。  
&emsp; 优势是文件和hadoop api中的MapFile是相互兼容的  
&emsp; **3、RCFile**  
&emsp; 存储方式：**数据按行分块，每块按列存储**。结合了行存储和列存储的优点：  
&emsp; &emsp; 首先，RCFile 保证同一行的数据位于同一节点，因此元组重构的开销很低；  
&emsp; &emsp; 其次，像列存储一样，RCFile 能够利用列维度的数据压缩，并且能跳过不必要的列读取；  
&emsp; **4、ORCFile**  
&emsp; 存储方式：数据按行分块 每块按照列存储。  
&emsp; 压缩快、快速列存取。  
&emsp; 效率比rcfile高，是rcfile的改良版本。  
&emsp; 总结：**相比TEXTFILE和SEQUENCEFILE，RCFILE由于列式存储方式，数据加载时性能消耗较大，但是具有较好的压缩比和查询响应**。  
&emsp; **数据仓库的特点是一次写入、多次读取，因此，整体来看，RCFILE相比其余两种格式具有较明显的优势**。  

### 11、所有的Hive任务都会有MapReduce的执行吗？
&emsp; 不是，从Hive0.10.0版本开始，对于简单的不需要聚合的类似SELECT <col> from <table> LIMIT n语句，不需要起MapReduce job，直接通过Fetch task获取数据。  

### 12、Hive的函数：UDF、UDAF、UDTF的区别？
&emsp; UDF：单行进入，单行输出  
&emsp; UDAF：多行进入，单行输出  
&emsp; UDTF：单行输入，多行输出  

### 13、说说对Hive桶表的理解？
&emsp; 桶表是对数据进行哈希取值，然后放到不同文件中存储。  
&emsp; 数据加载到桶表时，会对字段取hash值，然后与桶的数量取模。把数据放到对应的文件中。物理上，每个桶就是表(或分区）目录里的一个文件，一个作业产生的桶(输出文件)和reduce任务个数相同。  
&emsp; 桶表专门用于抽样查询，是很专业性的，不是日常用来存储数据的表，需要抽样查询时，才创建和使用桶表。  












