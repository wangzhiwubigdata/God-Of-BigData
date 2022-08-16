## Hadoop面试题总结（五）——优化问题

### 1、MapReduce跑得慢的原因？（**☆☆☆☆☆**）
Mapreduce 程序效率的瓶颈在于两点：  
1）计算机性能  
&emsp; CPU、内存、磁盘健康、网络  
2）I/O 操作优化  
&emsp; （1）数据倾斜  
&emsp; （2）map和reduce数设置不合理  
&emsp; （3）reduce等待过久  
&emsp; （4）小文件过多   
&emsp; （5）大量的不可分块的超大文件   
&emsp; （6）spill次数过多  
&emsp; （7）merge次数过多等  

### 2、MapReduce优化方法（☆☆☆☆☆）  
1）数据输入  
&emsp; （1）合并小文件：在执行mr任务前将小文件进行合并，大量的小文件会产生大量的map任务，增大map任务装载次数，而任务的装载比较耗时，从而导致mr运行较慢。   
&emsp; （2）采用ConbinFileInputFormat来作为输入，解决输入端大量小文件场景。  
2）map阶段  
&emsp; （1）减少spill次数：通过调整io.sort.mb及sort.spill.percent参数值，增大触发spill的内存上限，减少spill次数，从而减少磁盘 IO。   
&emsp; （2）减少merge次数：通过调整io.sort.factor参数，增大merge的文件数目，减少merge的次数，从而缩短mr处理时间。    
&emsp; （3）在 map 之后先进行combine处理，减少I/O。  
3）reduce阶段  
&emsp; （1）合理设置map和reduce数：两个都不能设置太少，也不能设置太多。太少，会导致task等待，延长处理时间；太多，会导致 map、reduce任务间竞争资源，造成处理超时等错误。   
&emsp; （2）设置map、reduce共存：调整slowstart.completedmaps参数，使map运行到一定程度后，reduce也开始运行，减少reduce的等待时间。  
&emsp; （3）规避使用reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。  
&emsp; （4）合理设置reduce端的buffer，默认情况下，数据达到一个阈值的时候，buffer中的数据就会写入磁盘，然后reduce会从磁盘中获得所有的数据。也就是说，buffer和reduce是没有直接关联的，中间多个一个写磁盘->读磁盘的过程，既然有这个弊端，那么就可以通过参数来配置，使得buffer中的一部分数据可以直接输送到reduce，从而减少IO开销：mapred.job.reduce.input.buffer.percent，默认为0.0。当值大于0的时候，会保留指定比例的内存读buffer中的数据直接拿给reduce使用。这样一来，设置buffer需要内存，读取数据需要内存，reduce计算也要内存，所以要根据作业的运行情况进行调整。  
4）IO传输  
&emsp; （1）采用数据压缩的方式，减少网络IO的时间。安装Snappy和LZOP压缩编码器。  
&emsp; （2）使用SequenceFile二进制文件  
5）数据倾斜问题  
&emsp; （1）数据倾斜现象 
&emsp; &emsp; 数据频率倾斜——某一个区域的数据量要远远大于其他区域。  
&emsp; &emsp; 数据大小倾斜——部分记录的大小远远大于平均值。  
&emsp; （2）如何收集倾斜数据  
&emsp; &emsp; 在reduce方法中加入记录map输出键的详细情况的功能。
```java
public static final String MAX_VALUES = "skew.maxvalues";
private int maxValueThreshold;

@Override
public void configure(JobConf job) {
     maxValueThreshold = job.getInt(MAX_VALUES, 100);
}

@Override
public void reduce(Text key, Iterator<Text> values,
                     OutputCollector<Text, Text> output,
                     Reporter reporter) throws IOException {
     int i = 0;
     while (values.hasNext()) {
         values.next();
         i++;
     }
     if (++i > maxValueThreshold) {
         log.info("Received " + i + " values for key " + key);
     }
}
```
&emsp; （3）减少数据倾斜的方法  
&emsp; &emsp; 方法1：抽样和范围分区  
&emsp; &emsp; &emsp; 可以通过对原始数据进行抽样得到的结果集来预设分区边界值。  
&emsp; &emsp; 方法2：自定义分区   
&emsp; &emsp; &emsp; 另一个抽样和范围分区的替代方案是基于输出键的背景知识进行自定义分区。例如，如果map输出键的单词来源于一本书。其中大部分必然是省略词（stopword）。那么就可以将自定义分区将这部分省略词发送给固定的一部分reduce实例。而将其他的都发送给剩余的reduce实例。  
&emsp; &emsp; 方法3：Combine  
&emsp; &emsp; &emsp; 使用Combine可以大量地减小数据频率倾斜和数据大小倾斜。在可能的情况下，combine的目的就是聚合并精简数据。  

### 3、HDFS小文件优化方法（☆☆☆☆☆）  
1）HDFS小文件弊端：  
&emsp; HDFS上每个文件都要在namenode上建立一个索引，这个索引的大小约为150byte，这样当小文件比较多的时候，就会产生很多的索引文件，一方面会大量占用namenode的内存空间，另一方面就是索引文件过大是的索引速度变慢。   
2）解决的方式：   
&emsp; （1）Hadoop本身提供了一些文件压缩的方案。 
&emsp; （2）从系统层面改变现有HDFS存在的问题，其实主要还是小文件的合并，然后建立比较快速的索引。  
3）Hadoop自带小文件解决方案  
&emsp; （1）Hadoop Archive：  
&emsp; &emsp; 是一个高效地将小文件放入HDFS块中的文件存档工具，它能够将多个小文件打包成一个HAR文件，这样在减少namenode内存使用的同时。   
&emsp; （2）Sequence file：  
&emsp; &emsp; sequence file由一系列的二进制key/value组成，如果为key小文件名，value为文件内容，则可以将大批小文件合并成一个大文件。   
&emsp; （3）CombineFileInputFormat：  
&emsp; &emsp; CombineFileInputFormat是一种新的inputformat，用于将多个文件合并成一个单独的split，另外，它会考虑数据的存储位置。  












