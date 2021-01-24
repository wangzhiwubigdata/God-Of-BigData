## Hive面试题整理（二）  

### 1、Fetch抓取  
&emsp; Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。   
&emsp; 在hive-default.xml.template文件中hive.fetch.task.conversion默认是more，老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。  

### 2、本地模式  
&emsp; 大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时Hive的输入数据量是非常小的。在这种情况下，为查询触发执行任务时消耗可能会比实际job的执行时间要多的多。对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。  
&emsp; 用户可以通过设置hive.exec.mode.local.auto的值为true，来让Hive在适当的时候自动启动这个优化。  

### **表的优化**  
### 3、小表、大表Join  
&emsp; 将key相对分散，并且数据量小的表放在join的左边，这样可以有效减少内存溢出错误发生的几率；再进一步，可以使用Group让小的维度表（1000条以下的记录条数）先进内存。在map端完成reduce。  
&emsp; 实际测试发现：新版的hive已经对小表JOIN大表和大表JOIN小表进行了优化。小表放在左边和右边已经没有明显区别。  

### 4、大表Join大表  
1）空KEY过滤  
&emsp; 有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。例如key对应的字段为空。  
2）空key转换  
&emsp; 有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。  

### 5、Group By  
&emsp; 默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。  
&emsp; 并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。  
1）开启Map端聚合参数设置  
&emsp; &emsp; （1）是否在Map端进行聚合，默认为True  
&emsp; &emsp; &emsp; hive.map.aggr = true  
&emsp; &emsp; （2）在Map端进行聚合操作的条目数目     
&emsp; &emsp; &emsp; hive.groupby.mapaggr.checkinterval = 100000  
&emsp; &emsp; （3）有数据倾斜的时候进行负载均衡（默认是false）     
&emsp; &emsp; &emsp; hive.groupby.skewindata = true  
&emsp; **当选项设定为 true，生成的查询计划会有两个MR Job**。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce中**，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。  

### 6、Count(Distinct) 去重统计  
&emsp; 数据量小的时候无所谓，数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换  

### 7、笛卡尔积  
&emsp; 尽量避免笛卡尔积，join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积  

### 8、行列过滤  
&emsp; 列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。  
&emsp; 行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤。  

### **数据倾斜**  
### 9、 Map数  
1）通常情况下，作业会通过input的目录产生一个或者多个map任务。  
&emsp; 主要的决定因素有：input的文件总个数，input的文件大小，集群设置的文件块大小。  
2）是不是map数越多越好？  
&emsp; 答案是否定的。如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。  
3）是不是保证每个map处理接近128m的文件块，就高枕无忧了？  
&emsp; 答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时。  
&emsp; 针对上面的问题2和3，我们需要采取两种方式来解决：即减少map数和增加map数；  

### 10、小文件进行合并  
&emsp; 在map执行前合并小文件，减少map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。  
&emsp; set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;  

### 11、复杂文件增加Map数  
&emsp; 当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。  
&emsp; 增加map的方法为：根据computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M公式，调整maxSize最大值。让maxSize最大值低于blocksize就可以增加map的个数。  

### 12、Reduce数  
1）调整reduce个数方法一   
&emsp; （1）每个Reduce处理的数据量默认是256MB  
&emsp; &emsp; hive.exec.reducers.bytes.per.reducer=256000000  
&emsp; （2）每个任务最大的reduce数，默认为1009  
&emsp; &emsp; hive.exec.reducers.max=1009  
&emsp; （3）计算reducer数的公式  
&emsp; &emsp; N=min(参数2，总输入数据量/参数1)  
2）调整reduce个数方法二  
&emsp; 在hadoop的mapred-default.xml文件中修改  
&emsp; 设置每个job的Reduce个数  
&emsp; set mapreduce.job.reduces = 15;  
3）reduce个数并不是越多越好  
&emsp; （1）过多的启动和初始化reduce也会消耗时间和资源；  
&emsp; （2）另外，有多少个reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题；   
&emsp; 在设置reduce个数的时候也需要考虑这两个原则：处理大数据量利用合适的reduce数；使单个reduce任务处理数据量大小要合适。  

**==========================**  

### 13、并行执行  
&emsp; Hive会将一个查询转化成一个或者多个阶段。这样的阶段可以是MapReduce阶段、抽样阶段、合并阶段、limit阶段。或者Hive执行过程中可能需要的其他阶段。默认情况下，Hive一次只会执行一个阶段。不过，某个特定的job可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个job的执行时间缩短。不过，如果有更多的阶段可以并行执行，那么job可能就越快完成。  
&emsp; 通过设置参数hive.exec.parallel值为true，就可以开启并发执行。不过，在共享集群中，需要注意下，如果job中并行阶段增多，那么集群利用率就会增加。  





