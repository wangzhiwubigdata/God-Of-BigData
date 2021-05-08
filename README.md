
<div align="center">  
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/logo.jpg" width=""/>
</br>

已经更新300+篇~ </br>
关注[公众号](#QQQ扫我关注公众号)~
</div>

## 大数据成神之路目录
- 图片打不开，点[这里](https://blog.csdn.net/u013411339/article/details/113097759)

### 大数据开发基础篇
| :ski:Java基础| :memo:NIO|:book:并发|:guitar:JVM|:dollar:分布式|:floppy_disk:Zookeeper|:punch:RPC|:art:Netty|:computer:Linux|
| :------:| :------: | :------: |:------: |:------: |:------: |:------: |:------: |:------:|
| [Java基础](#一Java基础) | [NIO](#二NIO基础) | [并发容器](#三Java并发容器) |[JVM](#四JVM深度解析和面试点) |[分布式](#五分布式理论基础和原理) |[zookeeper](#六大数据框架开发基础-zookeeper)|[RPC](#七大数据框架开发基础-RPC)|[Netty](#八大数据框架基石之网路通信-Netty)|[Linux](/九Linux基础/Linux基础和命令.md)|

<br/>

### 大数据框架学习篇

<table>
    <tr>
      <th><img width="50px" src="pictures/hadoop.jpg"></th>
      <th><img width="50px" src="pictures/hive.jpg"></th>
      <th><img width="50px" src="pictures/spark.jpg"></th>
      <th><img width="50px" src="pictures/flink.png"></th>
      <th><img width="50px" src="pictures/hbase.png"></th>
      <th><img width="50px" src="pictures/kafka.png"></th>
      <th><img width="50px" src="pictures/zookeeper.jpg"></th>
      <th><img width="50px" src="pictures/flume.png"></th>
      <th><img width="50px" src="pictures/sqoop.png"></th>
      <th><img width="50px" src="pictures/azkaban.png"></th>
    </tr>
    <tr>
      <td align="center"><a href="#一hadoop">Hadoop</a></td>
      <td align="center"><a href="#二hive">Hive</a></td>
      <td align="center"><a href="#三spark">Spark</a></td>
      <td align="center"><a href="#四flink">Flink</a></td>
      <td align="center"><a href="#五hbase">HBase</a></td>
      <td align="center"><a href="#六kafka">Kafka</a></td>
      <td align="center"><a href="#七zookeeper">Zookeeper</a></td>
      <td align="center"><a href="#八flume">Flume</a></td>
      <td align="center"><a href="#九sqoop">Sqoop</a></td>
      <td align="center"><a href="#十azkaban">Azkaban</a></td>
    </tr>
  </table>
<br/>


### 大数据开发实战进阶篇

这里的文章主要是我平时发表在公众号，博客等的文章，精心挑选，以飨读者。

<table>
    <tr>
      <th><img width="50px" src="pictures/flink.png"></th>
      <th><img width="50px" src="pictures/spark.jpg"></th>
      <th><img width="50px" src="pictures/kafka.png"></th>
      <th><img width="50px" src="pictures/olap.jpg"></th>
    </tr>
    <tr>
      <td align="center"><a href="#Flink实战合集">Flink实战进阶</a></td>
      <td align="center"><a href="#Spark实战合集">Spark实战进阶</a></td>
      <td align="center"><a href="#Kafka实战合集">Kafka实战进阶</a></td>
      <td align="center"><a href="#数据仓库实战合集">OLAP实战进阶</a></td>
    </tr>
  </table>
<br/>

### 大数据开发面试篇

<table>
    <tr>
      <th><img width="50px" src="pictures/olap.jpg"></th>
      <th><img width="50px" src="pictures/olap.jpg"></th>
    </tr>
    <tr>
      <td align="center"><a href="#面试系列合集">面试系列合集</a></td>
      <td align="center"><a href="#大数据算法">大数据算法</a></td>
    </tr>
  </table>
<br/> 


## 第一部分: 大数据开发基础篇

### 一、Java基础
 * [大数据成神之路-Java高级特性增强(多线程)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(%E5%A4%9A%E7%BA%BF%E7%A8%8B).md)
 * [大数据成神之路-Java高级特性增强(Synchronized关键字)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(Synchronized%E5%85%B3%E9%94%AE%E5%AD%97).md)
 * [大数据成神之路-Java高级特性增强(volatile关键字)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(volatile%E5%85%B3%E9%94%AE%E5%AD%97).md)
 * [大数据成神之路-Java高级特性增强(锁)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(%E9%94%81).md)
 * [大数据成神之路-Java高级特性增强(ArrayList/Vector)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6).md)
 * [大数据成神之路-Java高级特性增强(LinkedList)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA.md)
 * [大数据成神之路-Java高级特性增强(HashMap)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(HashMap).md)
 * [大数据成神之路-Java高级特性增强(HashSet)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(HashSet).md)
 * [大数据成神之路-Java高级特性增强(LinkedHashMap)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(LinkedHashMap).md)

### 二、NIO基础
 * [大数据成神之路-Java高级特性增强-NIO大纲](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA-NIO.md)
 * [NIO概览](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/NIO%E6%A6%82%E8%A7%88.md)
 * [Java NIO之Buffer(缓冲区)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/Java%20NIO%E4%B9%8BBuffer(%E7%BC%93%E5%86%B2%E5%8C%BA).md)
 * [Java NIO之Channel(通道)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/Java%20NIO%E4%B9%8BChannel(%E9%80%9A%E9%81%93).md)
 * [ava NIO之Selector(选择器)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/Java%20NIO%E4%B9%8BSelector(%E9%80%89%E6%8B%A9%E5%99%A8).md)
 * [Java NIO之拥抱Path和Files](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/Java%20NIO%E4%B9%8B%E6%8B%A5%E6%8A%B1Path%E5%92%8CFiles.md)

### 三、Java并发容器
 * [大数据成神之路-Java高级特性增强(并发容器大纲)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8%E5%A4%A7%E7%BA%B2).md)
 * [大数据成神之路-Java高级特性增强(LinkedBlockingQueue)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(LinkedBlockingQueue).md)
 * [大数据成神之路-Java高级特性增强(LinkedBlockingDeque)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(LinkedBlockingDeque).md)
 * [大数据成神之路-Java高级特性增强(CopyOnWriteArraySet)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(CopyOnWriteArraySet).md)
 * [大数据成神之路-Java高级特性增强(CopyOnWriteArrayList)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(CopyOnWriteArrayList).md)
 * [大数据成神之路-Java高级特性增强(ConcurrentSkipListSet)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(ConcurrentSkipListSet).md)
 * [大数据成神之路-Java高级特性增强(ConcurrentSkipListMap)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(ConcurrentSkipListMap).md)
 * [大数据成神之路-Java高级特性增强(ConcurrentLinkedQueue)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(ConcurrentLinkedQueue).md)
 * [大数据成神之路-Java高级特性增强(ConcurrentHashMap)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(ConcurrentHashMap).md)
 * [大数据成神之路-Java高级特性增强(ArrayBlockingQueue)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%B9%B6%E5%8F%91%E5%AE%B9%E5%99%A8/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%88%90%E7%A5%9E%E4%B9%8B%E8%B7%AF-Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA(ArrayBlockingQueue).md)

### 四、JVM深度解析和面试点
   ##### 先来10篇基础热身
   * [JVM内存结构](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/JVM%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.md)
   * [HotSpot虚拟机对象探秘](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/HotSpot%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AF%B9%E8%B1%A1%E6%8E%A2%E7%A7%98.md)
   * [垃圾收集策略与算法](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E7%AD%96%E7%95%A5%E4%B8%8E%E7%AE%97%E6%B3%95.md)
   * [HotSpot垃圾收集器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/HotSpot%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.md)
   * [内存分配与回收策略](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E4%B8%8E%E5%9B%9E%E6%94%B6%E7%AD%96%E7%95%A5.md)
   * [JVM性能调优](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/JVM%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98.md)
   * [类文件结构](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E7%B1%BB%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84.md)
   * [类加载的时机](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E7%9A%84%E6%97%B6%E6%9C%BA.md)
   * [类加载的过程](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E7%9A%84%E8%BF%87%E7%A8%8B.md)
   * [类加载器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8.md)
   ##### 再来5篇详细解说
   * [java类的加载机制](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/jvm%E7%B3%BB%E5%88%97(%E4%B8%80)java%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6.md)
   * [JVM内存结构](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/jvm%E7%B3%BB%E5%88%97(%E4%BA%8C)JVM%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.md)
   * [GC算法 垃圾收集器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/jvm%E7%B3%BB%E5%88%97(%E4%B8%89)GC%E7%AE%97%E6%B3%95%20%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.md)
   * [jvm调优-命令大全](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/jvm%E7%B3%BB%E5%88%97(%E5%9B%9B)jvm%E8%B0%83%E4%BC%98-%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8%EF%BC%88jps%20jstat%20jmap%20jhat%20jstack%20jinfo%EF%BC%89.md)
   * [Java GC 分析](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/JVM/jvm%E7%B3%BB%E5%88%97(%E4%BA%94)Java%20GC%20%E5%88%86%E6%9E%90.md)

### 五、分布式理论基础和原理
   * [分布式系统的一些基本概念](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%9A%84%E4%B8%80%E4%BA%9B%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.md)
   * [分布式系统理论基础一： 一致性、2PC和3PC](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80%E4%B8%80%EF%BC%9A%20%E4%B8%80%E8%87%B4%E6%80%A7%E3%80%812PC%E5%92%8C3PC.md)
   * [分布式系统理论基础二-CAP](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80%E4%BA%8C-CAP.md)
   * [分布式系统理论基础三-时间、时钟和事件顺序](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80%E4%B8%89-%E6%97%B6%E9%97%B4%E3%80%81%E6%97%B6%E9%92%9F%E5%92%8C%E4%BA%8B%E4%BB%B6%E9%A1%BA%E5%BA%8F.md)
   * [分布式系统理论进阶 - Paxos](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E8%BF%9B%E9%98%B6%20-%20Paxos.md)
   * [分布式系统理论进阶 - Raft、Zab](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E8%BF%9B%E9%98%B6%20-%20Raft%E3%80%81Zab.md)
   * [分布式系统理论进阶：选举、多数派和租约](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E7%90%86%E8%AE%BA%E8%BF%9B%E9%98%B6%EF%BC%9A%E9%80%89%E4%B8%BE%E3%80%81%E5%A4%9A%E6%95%B0%E6%B4%BE%E5%92%8C%E7%A7%9F%E7%BA%A6.md)
   * [分布式锁的解决方案](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.md)
   * [分布式锁的解决方案(二)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88(%E4%BA%8C).md)
   * [分布式事务的解决方案](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.md)
   * [分布式ID生成器解决方案](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA/%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E5%99%A8%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.md)

### 六、大数据框架开发基础-Zookeeper

   * [安装和运行](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/zookeeeper/zk%E5%AE%89%E8%A3%85%E5%92%8C%E8%BF%90%E8%A1%8C.md)
   * [zookeeper服务](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/zookeeeper/zk%E6%9C%8D%E5%8A%A1.md)
   * [zookeeper应用程序](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/zookeeeper/ZooKeeper%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F.md)
   * [zookeeper开发实例](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/zookeeeper/zk%E5%BC%80%E5%8F%91%E5%AE%9E%E4%BE%8B.md)
   * [zookeeper集群构建](http://www.importnew.com/23237.html)

### 七、大数据框架开发基础-RPC

   * [RPC简单介绍](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/RPC/RPC%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D.md)
   * [RPC的原理和框架](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/RPC/RPC%E7%9A%84%E5%8E%9F%E7%90%86%E5%92%8C%E6%A1%86%E6%9E%B6.md)
   * [手把手教你实现一个简单的RPC](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/RPC/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84RPC.md)

### 八、大数据框架基石之网路通信-Netty
   * [关于Netty我们都需要知道什么](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/%E5%85%B3%E4%BA%8ENetty%E6%88%91%E4%BB%AC%E9%83%BD%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E4%BB%80%E4%B9%88.md)
   * [Netty源码解析-概述篇](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90-%E6%A6%82%E8%BF%B0%E7%AF%87.md)
   * [Netty源码解析1-Buffer](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%901-Buffer.md) 
   * [Netty源码解析2-Reactor](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%902-Reactor.md)
   * [Netty源码解析3-Pipeline](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%903-Pipeline.md)
   * [Netty源码解析4-Handler综述](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%904-Handler%E7%BB%BC%E8%BF%B0.md)
   * [Netty源码解析5-ChannelHandler](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%905-ChannelHandler.md)
   * [Netty源码解析6-ChannelHandler实例之LoggingHandler](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%906-ChannelHandler%E5%AE%9E%E4%BE%8B%E4%B9%8BLoggingHandler.md)
   * [Netty源码解析7-ChannelHandler实例之TimeoutHandler](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%907-ChannelHandler%E5%AE%9E%E4%BE%8B%E4%B9%8BTimeoutHandler.md)
   * [Netty源码解析8-ChannelHandler实例之CodecHandler](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%908-ChannelHandler%E5%AE%9E%E4%BE%8B%E4%B9%8BCodecHandler.md)
   * [Netty源码解析9-ChannelHandler实例之MessageToByteEncoder](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Netty/Netty%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%909-ChannelHandler%E5%AE%9E%E4%BE%8B%E4%B9%8BMessageToByteEncoder.md)


## 第二部分:大数据框架学习篇

本部分引用作者heibaiying，大佬写的文章非常好，欢迎大家关注他的博客。我个人会持续补充更有深度和实战性的文章~

### 一、Hadoop

1. [分布式文件存储系统 —— HDFS](大数据框架学习/Hadoop-HDFS.md)
2. [分布式计算框架 —— MapReduce](大数据框架学习/Hadoop-MapReduce.md)
3. [集群资源管理器 —— YARN](大数据框架学习/Hadoop-YARN.md)
4. [Hadoop 单机伪集群环境搭建](大数据框架学习/installation/Hadoop单机环境搭建.md)
5. [Hadoop 集群环境搭建](大数据框架学习/installation/Hadoop集群环境搭建.md)
6. [HDFS 常用 Shell 命令](大数据框架学习/HDFS常用Shell命令.md)
7. [HDFS Java API 的使用](大数据框架学习/HDFS-Java-API.md)
8. [基于 Zookeeper 搭建 Hadoop 高可用集群](大数据框架学习/installation/基于Zookeeper搭建Hadoop高可用集群.md)
9. [Hadoop级简入门](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Hadoop/Hadoop%E6%9E%81%E7%AE%80%E5%85%A5%E9%97%A8.md)
10. [MapReduce编程模型和计算框架架构原理](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Hadoop/MapReduce%E7%BC%96%E7%A8%8B%E6%A8%A1%E5%9E%8B%E5%92%8C%E8%AE%A1%E7%AE%97%E6%A1%86%E6%9E%B6%E6%9E%B6%E6%9E%84%E5%8E%9F%E7%90%86.md)

### 二、Hive

1. [Hive 简介及核心概念](大数据框架学习/Hive简介及核心概念.md)
2. [Linux 环境下 Hive 的安装部署](大数据框架学习/installation/Linux环境下Hive的安装部署.md)
4. [Hive CLI 和 Beeline 命令行的基本使用](大数据框架学习/HiveCLI和Beeline命令行的基本使用.md)
6. [Hive 常用 DDL 操作](大数据框架学习/Hive常用DDL操作.md)
7. [Hive 分区表和分桶表](大数据框架学习/Hive分区表和分桶表.md)
8. [Hive 视图和索引](大数据框架学习/Hive视图和索引.md)
9. [Hive常用 DML 操作](大数据框架学习/Hive常用DML操作.md)
10. [Hive 数据查询详解](大数据框架学习/Hive数据查询详解.md)

### 三、Spark

**Spark Core :**

1. [Spark 简介](大数据框架学习/Spark简介.md)
2. [Spark 开发环境搭建](大数据框架学习/installation/Spark开发环境搭建.md)
4. [弹性式数据集 RDD](大数据框架学习/Spark_RDD.md)
5. [RDD 常用算子详解](大数据框架学习/Spark_Transformation和Action算子.md)
5. [Spark 运行模式与作业提交](大数据框架学习/Spark部署模式与作业提交.md)
6. [Spark 累加器与广播变量](大数据框架学习/Spark累加器与广播变量.md)
7. [基于 Zookeeper 搭建 Spark 高可用集群](大数据框架学习/installation/Spark集群环境搭建.md)

**Spark SQL :**

1. [DateFrame 和 DataSet ](大数据框架学习/SparkSQL_Dataset和DataFrame简介.md)
2. [Structured API 的基本使用](大数据框架学习/Spark_Structured_API的基本使用.md)
3. [Spark SQL 外部数据源](大数据框架学习/SparkSQL外部数据源.md)
4. [Spark SQL 常用聚合函数](大数据框架学习/SparkSQL常用聚合函数.md)
5. [Spark SQL JOIN 操作](大数据框架学习/SparkSQL联结操作.md)

**Spark Streaming ：**

1. [Spark Streaming 简介](大数据框架学习/Spark_Streaming与流处理.md)
2. [Spark Streaming 基本操作](大数据框架学习/Spark_Streaming基本操作.md)
3. [Spark Streaming 整合 Flume](大数据框架学习/Spark_Streaming整合Flume.md)
4. [Spark Streaming 整合 Kafka](大数据框架学习/Spark_Streaming整合Kafka.md)


## 四、Flink

1. [Flink 核心概念综述](大数据框架学习/Flink核心概念综述.md)
2. [Flink 开发环境搭建](大数据框架学习/Flink开发环境搭建.md)
3. [Flink Data Source](大数据框架学习/Flink_Data_Source.md)
4. [Flink Data Transformation](大数据框架学习/Flink_Data_Transformation.md)
4. [Flink Data Sink](大数据框架学习/Flink_Data_Sink.md)
6. [Flink 窗口模型](大数据框架学习/Flink_Windows.md)
7. [Flink 状态管理与检查点机制](大数据框架学习/Flink状态管理与检查点机制.md)
8. [Flink Standalone 集群部署](大数据框架学习/installation/Flink_Standalone_Cluster.md)

#### Flink当前最火的实时计算引擎-入门篇
   * [Flink从入门到放弃(入门篇1)-Flink是什么](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/Flink%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83(%E5%85%A5%E9%97%A8%E7%AF%871)-Flink%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md)
   * [Flink从入门到放弃(入门篇2)-本地环境搭建&构建第一个Flink应用](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/Flink%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83(%E5%85%A5%E9%97%A8%E7%AF%872)-%E6%9C%AC%E5%9C%B0%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%26%E6%9E%84%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AAFlink%E5%BA%94%E7%94%A8.md)
   * [Flink从入门到放弃(入门篇3)-DataSetAPI](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/Flink%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83(%E5%85%A5%E9%97%A8%E7%AF%873)-DataSetAPI.md)
   * [Flink从入门到放弃(入门篇4)-DataStreamAPI](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/Flink%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83(%E5%85%A5%E9%97%A8%E7%AF%874)-DataStreamAPI.md)
   * [Flink集群部署](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/Flink%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.md)
   * [Flink重启策略](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/6-Flink%E9%87%8D%E5%90%AF%E7%AD%96%E7%95%A5.md)
   * [Flink的分布式缓存](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/7-Flink%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98.md)
   * [Flink中的窗口](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/8-Flink%E4%B8%AD%E7%9A%84%E7%AA%97%E5%8F%A3.md)
   * [Flink中的Time](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/9-Flink%E4%B8%AD%E7%9A%84Time.md)
   * [Flink集群搭建的HA](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/10-Flink%E9%9B%86%E7%BE%A4%E7%9A%84%E9%AB%98%E5%8F%AF%E7%94%A8(%E6%90%AD%E5%BB%BA%E7%AF%87%E8%A1%A5%E5%85%85).md)
   * [Flink中的时间戳和水印](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/11-%E6%97%B6%E9%97%B4%E6%88%B3%E5%92%8C%E6%B0%B4%E5%8D%B0.md)
   * [Flink广播变量](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/12-Broadcast%E5%B9%BF%E6%92%AD%E5%8F%98%E9%87%8F.md)
   * [Flink-Kafka-Connector](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/13-Flink-Kafka-Connector.md)
   * [Flink-Table-&-SQL实战](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/14-Flink-Table-%26-SQL.md)
   * [15-Flink实战项目之实时热销排行](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/15-Flink%E5%AE%9E%E6%88%98%E9%A1%B9%E7%9B%AE%E4%B9%8B%E5%AE%9E%E6%97%B6%E7%83%AD%E9%94%80%E6%8E%92%E8%A1%8C.md)
   * [16-Flink-Redis-Sink](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/16-Flink-Redis-Sink.md)
   * [17-Flink消费Kafka写入Mysql](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink/17-Flink%E6%B6%88%E8%B4%B9Kafka%E5%86%99%E5%85%A5Mysql.md)

#### Flink当前最火的实时计算引擎-放弃篇

   * [Flink漫谈系列1-概述](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97/Apache-Flink%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97(1)-%E6%A6%82%E8%BF%B0.md)
   * [Flink漫谈系列2-watermark](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97/Apache-Flink-%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97(02)-Watermark.md)
   * [Flink漫谈系列3-state](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Flink%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97/Apache-Flink-%E6%BC%AB%E8%B0%88%E7%B3%BB%E5%88%97(03)-State.md)

   


## 五、HBase

1. [Hbase 简介](大数据框架学习/Hbase简介.md)
2. [HBase 系统架构及数据结构](大数据框架学习/Hbase系统架构及数据结构.md)
3. [HBase 基本环境搭建 (Standalone /pseudo-distributed mode)](大数据框架学习/installation/HBase单机环境搭建.md)
4. [HBase 集群环境搭建](大数据框架学习/installation/HBase集群环境搭建.md)
5. [HBase 常用 Shell 命令](大数据框架学习/Hbase_Shell.md)
6. [HBase Java API](大数据框架学习/Hbase_Java_API.md)
7. [Hbase 过滤器详解](大数据框架学习/Hbase过滤器详解.md)
8. [HBase 协处理器详解](大数据框架学习/Hbase协处理器详解.md)
9. [HBase 容灾与备份](大数据框架学习/Hbase容灾与备份.md)
10. [HBase的 SQL 中间层 —— Phoenix](大数据框架学习/Hbase的SQL中间层_Phoenix.md)
11. [Spring/Spring Boot 整合 Mybatis + Phoenix](大数据框架学习/Spring+Mybtais+Phoenix整合.md)

## 六、Kafka

**Kafka基本原理 ：**

1. [Kafka 简介](大数据框架学习/Kafka简介.md)
2. [基于 Zookeeper 搭建 Kafka 高可用集群](大数据框架学习/installation/基于Zookeeper搭建Kafka高可用集群.md)
3. [Kafka 生产者详解](大数据框架学习/Kafka生产者详解.md)
4. [Kafka 消费者详解](大数据框架学习/Kafka消费者详解.md)
5. [深入理解 Kafka 副本机制](大数据框架学习/Kafka深入理解分区副本机制.md)

**分布式消息队列Kafka原理及与流式计算的集成 ：**

 1. [Apache-Kafka简介](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E7%AE%80%E4%BB%8B.md)
 2. [Apache-Kafka核心概念](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.md)
 3. [Apache-Kafka安装和使用](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8.md)
 4. [Apache-Kafka编程实战](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98.md)
 5. [Apache-Kafka核心组件和流程(副本管理器)](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E5%92%8C%E6%B5%81%E7%A8%8B(%E5%89%AF%E6%9C%AC%E7%AE%A1%E7%90%86%E5%99%A8).md)
 6. [Apache-Kafka核心组件和流程-协调器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E5%92%8C%E6%B5%81%E7%A8%8B-%E5%8D%8F%E8%B0%83%E5%99%A8.md)
 7. [Apache-Kafka核心组件和流程-控制器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E5%92%8C%E6%B5%81%E7%A8%8B-%E6%8E%A7%E5%88%B6%E5%99%A8.md)
 8. [Apache-Kafka核心组件和流程-日志管理器](https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/Kafka/Apache-Kafka%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E5%92%8C%E6%B5%81%E7%A8%8B-%E6%97%A5%E5%BF%97%E7%AE%A1%E7%90%86%E5%99%A8.md)


## 七、Zookeeper

1. [Zookeeper 简介及核心概念](大数据框架学习/Zookeeper简介及核心概念.md)
2. [Zookeeper 单机环境和集群环境搭建](大数据框架学习/installation/Zookeeper单机环境和集群环境搭建.md) 
3. [Zookeeper 常用 Shell 命令](大数据框架学习/Zookeeper常用Shell命令.md)
4. [Zookeeper Java 客户端 —— Apache Curator](大数据框架学习/Zookeeper_Java客户端Curator.md)
5. [Zookeeper  ACL 权限控制](大数据框架学习/Zookeeper_ACL权限控制.md)

## 八、Flume

1. [Flume 简介及基本使用](大数据框架学习/Flume简介及基本使用.md)
2. [Linux 环境下 Flume 的安装部署](大数据框架学习/installation/Linux下Flume的安装.md)
3. [Flume 整合 Kafka](大数据框架学习/Flume整合Kafka.md)

## 九、Sqoop

1. [Sqoop 简介与安装](大数据框架学习/Sqoop简介与安装.md)
2. [Sqoop 的基本使用](大数据框架学习/Sqoop基本使用.md)

## 十、Azkaban

1. [Azkaban 简介](大数据框架学习/Azkaban简介.md)
2. [Azkaban3.x 编译及部署](大数据框架学习/installation/Azkaban_3.x_编译及部署.md)
3. [Azkaban Flow 1.0 的使用](大数据框架学习/Azkaban_Flow_1.0_的使用.md)
4. [Azkaban Flow 2.0 的使用](大数据框架学习/Azkaban_Flow_2.0_的使用.md)


## 十一、大数据算法

#### 大数据算法

[大数据算法](https://blog.csdn.net/u013411339/article/details/113429172)

## 第三部分:大数据开发实战进阶篇

### 一、Flink实战进阶文章合集

#### Flink实战合集

[点我查看Flink实战合集](实战系列文章/Flink实战.md)

1. [菜鸟供应链实时技术架构演进](https://mp.weixin.qq.com/s/fnx2GnbCWNcaptVPsSp7dw)
2. [趣头条实战-基于Flink+ClickHouse构建实时数据平台](https://mp.weixin.qq.com/s/s6YFOINMw9TKg-QVOkZT9A)
3. [ApacheFlink新场景-OLAP引擎](https://mp.weixin.qq.com/s/NLwYpjzNgkR8O5zRg7RCoQ)
4. [说说Flink DataStream的八种物理分区逻辑](https://mp.weixin.qq.com/s/d_jzHb-b7LEGNz1CN34zMg)
5. [State Processor API：如何读取，写入和修改 Flink 应用程序的状态](https://mp.weixin.qq.com/s/eHPQx3kGKnhXeZpLhUNvng)
6. [Flink滑动窗口原理与细粒度滑动窗口的性能问题](https://mp.weixin.qq.com/s/Q4k0xgPCOUQ-A2DQ-XaJgw)
7. [基于Flink快速开发实时TopN](https://mp.weixin.qq.com/s/Ppz5740WTB7lTTLHNL72Tg)
8. [使用 Apache Flink 开发实时 ETL](https://mp.weixin.qq.com/s/kLjEceslHQxDDi3mRrjR-g)
9. [Flink Source/Sink探究与实践：RocketMQ数据写入HBase](https://mp.weixin.qq.com/s/kZQRQBVjYiXKfMhKM7SSqQ)
10. [Spark/Flink广播实现作业配置动态更新](https://mp.weixin.qq.com/s/GDylIWCDnjpX9_X6T9NmMA)
11. [Flink全链路延迟的测量方式](https://mp.weixin.qq.com/s/A6CIPsGf-aCWXkB7O-toVw)
12. [Flink原理-Flink中的数据抽象及数据交换过程](https://mp.weixin.qq.com/s/5BlCzguYiEP1h48jwkos2w)
13. [Flink SQL Window源码全解析](https://mp.weixin.qq.com/s/UkpkS_JiRGR0ibZKYechbg)
14. [Flink DataStream维度表Join的简单方案](https://mp.weixin.qq.com/s/e-lyViKV4NPmOVwA5Jn6Qw)
15. [Apache Flink的内存管理](https://mp.weixin.qq.com/s/cBMrF814jGtEFdve0Lrr6g)
16. [Flink1.9整合Kafka实战](https://mp.weixin.qq.com/s/e0BQoY5Y79NHhcQ9MqltFQ)
17. [Apache Flink在小米的发展和应用](https://mp.weixin.qq.com/s/KbhmJCW80UmeFwRxM3jerg)
18. [基于Kafka+Flink+Redis的电商大屏实时计算案例](https://mp.weixin.qq.com/s/BPzOBz7oTfn2_yW8tevEEw)
19. [Flink实战-壳找房基于Flink的实时平台建设](https://mp.weixin.qq.com/s/TsU_5N0Csfw-afN9AdAihw)
20. [用Flink取代Spark Streaming！知乎实时数仓架构演进](https://mp.weixin.qq.com/s/0M8XLTgpj6jWNcokNhyxAw)
21. [Flink实时数仓-美团点评实战](https://mp.weixin.qq.com/s/Oom-TaEsT6GKGs95dJil5Q)
22. [来将可留姓名？Flink最强学习资源合集!](https://mp.weixin.qq.com/s/13w43iYT3-riIj757HPGxw)
23. [数据不撒谎，Flink-Kafka性能压测全记录!](https://mp.weixin.qq.com/s/0VXqbzLBj5rZjjf4jAc3UQ)
24. [菜鸟在物流场景中基于Flink的流计算实践](https://mp.weixin.qq.com/s/2_8uOdDJwzYxUP-NLh6VhA)
25. [基于Flink构建实时数据仓库](https://mp.weixin.qq.com/s/Rhgt33y102WzR9-Zq15iVQ)
26. [Flink/Spark 如何实现动态更新作业配置](https://mp.weixin.qq.com/s/sjRV_F9tXEfqKL_00rJc7w)

### 二、Spark实战进阶文章合集

#### Spark实战合集

[点我查看Spark实战合集](实战系列文章/Spark实战.md)

1. [如果你在准备面试，好好看看这130道题](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486986&idx=1&sn=422d1a3c11c72ff97b32cc01142839f4&chksm=fd3d489fca4ac1895242ab94b932b12c65dc57b5f3a16acc7084dc8a189e9026290245a64c4f&token=1999457569&lang=zh_CN#rd)
2. [ORC文件存储格式的深入探究](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486981&idx=1&sn=9c8fc4c127d7e6108ac4e171e750d490&chksm=fd3d4890ca4ac186614f0dda8ffb2d35693a925b03861a01769c898652b53d0d436bca05ea12&token=1999457569&lang=zh_CN#rd)
3. [基于SparkStreaming+Kafka+HBase实时点击流案例](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486938&idx=1&sn=83347b444fc721442d2b4e1a58eca0e8&chksm=fd3d4b4fca4ac2597abd4a39a21dea83220858ae5efd1ae287b471bb9970f165b4854426f4c2&token=1999457569&lang=zh_CN#rd)
4. [HyperLogLog函数在Spark中的高级应用](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486931&idx=1&sn=9c3f3d6a677ed2aa6cc508046ca6da78&chksm=fd3d4b46ca4ac25044d6033458d3b43e7d1f50f447bded4eed8b246991cf620c91919c51e35b&token=1999457569&lang=zh_CN#rd)
5. [我们常说的海量小文件的根源是什么？](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486931&idx=2&sn=1c5987f3bad7a805895484ebfd683e11&chksm=fd3d4b46ca4ac250d31502a76f0cfd02ebea29d1161b9e9faec23ea6aa8947d0dd7d4268427f&token=1999457569&lang=zh_CN#rd)
6. [Structured Streaming | Apache Spark中处理实时数据的声明式API](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486904&idx=1&sn=5f4b673a87497a9c1dc9d7ce253994f3&chksm=fd3d4b2dca4ac23b9850c54d62ebe8be5920cbd4a491a46a0325e1fbbd91f6b1aef9adf2f638&token=1999457569&lang=zh_CN#rd)
7. [Spark面对OOM问题的解决方法及优化总结](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486765&idx=1&sn=516a32e8c1e9842606a7670862ec7e97&chksm=fd3d4bb8ca4ac2ae24e315083cdb195fdf897e1c3cce69d2c94183d5c20c1af1bfbe0e5480fd&token=1999457569&lang=zh_CN#rd)
8. [Spark 动态资源分配(Dynamic Resource Allocation) 解析](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486761&idx=1&sn=959aaa5266307a64181631ba1ae46e86&chksm=fd3d4bbcca4ac2aad8a3958da40c22e565997180c635bb6c32a9b4aa7e82a74a56423320c65a&token=1999457569&lang=zh_CN#rd)
9. [Apache Spark在海致大数据平台中的优化实践](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486668&idx=1&sn=981028fdcf937ba45b3914e2d48b94db&chksm=fd3d4a59ca4ac34f089ff1dce9542a0f481a73439acec9223fa2d776ff3f2eaec498cc543d06&token=1999457569&lang=zh_CN#rd)
10. [Spark/Flink广播实现作业配置动态更新](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486644&idx=1&sn=d2637a1e918c2b1be4c9fe3d74f75a92&chksm=fd3d4a21ca4ac3377cc8836939cc041cf934bb57f73b6b618fd1de608495d86e278c1c7e4cdc&token=1999457569&lang=zh_CN#rd)
11. [Spark SQL读数据库时不支持某些数据类型的问题](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486629&idx=2&sn=54d05858d659756a46b9be36b7b63ee5&chksm=fd3d4a30ca4ac326f69ba9b7cf6a82418afd31193e96aa28346779264a7d8a1df4c7a3a31fba&token=1999457569&lang=zh_CN#rd)
12. [这个面试问题很难么 | 如何处理大数据中的数据倾斜](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486586&idx=1&sn=e35def6429adaada0910b91eed8d7b1f&chksm=fd3d4aefca4ac3f9918d57b947d4bc8f71afd9d7666ffe28e3660e5efab332c714bef0879a90&token=1999457569&lang=zh_CN#rd)
13. [Spark难点 | Join的实现原理](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486582&idx=1&sn=7b6291dedb2e6892342e1ed705bdfb2e&chksm=fd3d4ae3ca4ac3f591f297635c0ff8e63e3deb6d6aec72c8fb18e19f8f6fcdb5cd2abcd37d09&token=1999457569&lang=zh_CN#rd)
14. [面试注意点 | Spark&Flink的区别拾遗](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486577&idx=1&sn=49fd0138ad9837192b6eb78cbdc478e6&chksm=fd3d4ae4ca4ac3f2aec91f40435d1eb55a9e497fe7d8ab9ad52396838f467879ac56cb83c9c2&token=1999457569&lang=zh_CN#rd)
15. [Spark Checkpoint的运行原理和源码实现](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486514&idx=1&sn=a3c3084b8a797a0889d7583fb10d70c9&chksm=fd3d4aa7ca4ac3b12416ba8269041032a515b7a4e71bda1709e359a15a04df4f638738c6e01a&token=1999457569&lang=zh_CN#rd)
16. [阿里云Spark Shuffle的优化](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486505&idx=1&sn=91316d3aa5a99945ccd992c0a98e500d&chksm=fd3d4abcca4ac3aa26f051504244239ff1ab48eb71cf01c14f689eb4767c121b1477d93b60ea&token=1999457569&lang=zh_CN#rd)
17. [使用Kafka+Spark+Cassandra构建实时处理引擎](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486485&idx=1&sn=999b84e0ac87f2aa1be4870921279a21&chksm=fd3d4a80ca4ac39695d9307582ff58938180d0c5b12f6db84ddae933edb345b0cbe46f6284ed&token=1999457569&lang=zh_CN#rd)
18. [基于HBase和Spark构建企业级数据处理平台](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486477&idx=1&sn=21394cca8fe279bbc2032d48f65672f6&chksm=fd3d4a98ca4ac38e3bf9700cfe65131fffadbbf9493ccc658b3e7be97ddb6665ad41a70f67f9&token=1999457569&lang=zh_CN#rd)
19. [SparkSQL在字节跳动的应用实践和优化实战](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486419&idx=1&sn=3bc8af144370a602817ca87415fe525d&chksm=fd3d4d46ca4ac450ec6f1ac84b2f3162071eb85221464bdb1976178ca2363cd2acb3d63d401c&token=1999457569&lang=zh_CN#rd)
20. [SparkRDD转DataSet/DataFrame的一个深坑](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486369&idx=1&sn=9a760114ed2c7509e191ad29370eecce&chksm=fd3d4d34ca4ac4222d310c37fb67bcaafd5425e1a16c43434c4d1fea403756e266de21b83181&token=1999457569&lang=zh_CN#rd)
21. [Spark和Flink的状态管理State的区别和应用](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486332&idx=1&sn=ecbe21981c5c36f6c420755e8d63fb8f&chksm=fd3d4de9ca4ac4ff1b11093432f444c9e4f66c45a6e01155d69cface70456fb418a98163757d&token=1999457569&lang=zh_CN#rd)
22. [Kafka+Spark Streaming管理offset的几种方法](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486328&idx=1&sn=b5d53e0007032114fb277e440e5ce4bf&chksm=fd3d4dedca4ac4fbb5026338af7dbbfe1d845fdc11e7e2279035dddc8ff0b6dd24c32be129ce&token=1999457569&lang=zh_CN#rd)
23. [从 PageRank Example谈Spark应用程序调优](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247486322&idx=2&sn=00ddcd16109249e45a70233d5ef959ba&chksm=fd3d4de7ca4ac4f15f85d9a2873c5d1070af3bb929479bd66f7a0dc89ac0777b6960cbce5970&token=1999457569&lang=zh_CN#rd)
24. [Spark调优|SparkSQL参数调优](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485574&idx=1&sn=723c02562ee1f44e88c389d6ac8a2c87&chksm=fd3d4e13ca4ac705ad892ada2c68792c906946f271bb1e6afd15a93749f8f87d0ea967ba2d37&token=1999457569&lang=zh_CN#rd)
25. [Flink/Spark 如何实现动态更新作业配置](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485507&idx=1&sn=2cab7a3714ce4e16351b8edcde95e777&chksm=fd3d4ed6ca4ac7c058d820f36b2d03c3ccddc293e1035b6897e99544e55931f6cd6a2f343647&token=1999457569&lang=zh_CN#rd)
26. [Stream SQL的执行原理与Flink的实现](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485361&idx=1&sn=8348203b6f17662a64fa5d412de97296&chksm=fd3d4124ca4ac8329bac00fae705f37c08236d583e9900f268796d463be9323d82ba4b8b1554&token=1999457569&lang=zh_CN#rd)
27. [Spark将Dataframe数据写入Hive分区表的方案](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485321&idx=1&sn=13e73673fb29bd134ab79e03a369288c&chksm=fd3d411cca4ac80ac68204d18aee55b95cadde7bf18cc2f8a549fc0b6ff4ec4db570190566c3&token=1999457569&lang=zh_CN#rd)
28. [Spark中几种ShuffleWriter的区别你都知道吗？](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485316&idx=1&sn=8a7a02023f15324885de7a5c93d4dd94&chksm=fd3d4111ca4ac807f34e1fa03023494d46f3770c035290ddefe1333419453a9258f504584aee&token=1999457569&lang=zh_CN#rd)
29. [SparkSQL的3种Join实现](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485217&idx=1&sn=3ce9fa8ad179c008754873129e51fbe7&chksm=fd3d41b4ca4ac8a25957fc9541437e6546fc2926df2908bad5adbd30a6cecf9f797fef74f894&token=1999457569&lang=zh_CN#rd)
30. [周期性清除Spark Streaming流状态的方法](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485138&idx=1&sn=8f71070470c8963e7c973b5f10bf3c03&chksm=fd3d4047ca4ac951981f7f0fa08f9f6a1821270441b5008da28115b67020b01ef2936b5f1e88&token=1999457569&lang=zh_CN#rd)
31. [Structured Streaming之状态存储解析](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485119&idx=1&sn=fd172b1f9c9ef99eac2ed976a7d4459f&chksm=fd3d402aca4ac93ced929dfb9b3d785fa5e00d82939b4ea7ed31e68096b87f31a0d11c5f7414&token=1999457569&lang=zh_CN#rd)
32. [Spark SQL重点知识总结](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485105&idx=1&sn=0bea228e6845d04739937b75bd2f8d9a&chksm=fd3d4024ca4ac9322cae55569b7bdc6d8546dc9c4583f25801d3820ab3b30094c50816c7c1c8&token=1999457569&lang=zh_CN#rd)
33. [SparkSQL极简入门](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485058&idx=1&sn=4d3b5c25ca1fdf1f0fb0cd99959d2371&chksm=fd3d4017ca4ac90140442dfc6032346d5841d6705bd92441ad3d0a0366db346752a6b154b976&token=1999457569&lang=zh_CN#rd)
34. [Spark Shuffle在网易的优化](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247485051&idx=1&sn=a1a70cad450634ceae44d4e14a4fc3ef&chksm=fd3d40eeca4ac9f8901a93271683811da05c62e7a6f1d2825378c32542ef2edf9947273a601a&token=1999457569&lang=zh_CN#rd)
35. [广告点击数实时统计：Spark StructuredStreaming + Redis Streams](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484999&idx=1&sn=f9cf6eae39bc1d54faaa144357731d2f&chksm=fd3d40d2ca4ac9c4e886bd0208521c45dbab98e9cfd34826e6808d414bcd66f203f0359382e1&token=1999457569&lang=zh_CN#rd)
36. [Spark内存调优](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484972&idx=1&sn=ff9a2925c31e07b558504be17937872b&chksm=fd3d40b9ca4ac9afe2cce8ff4a6c50724a146b87d2d05d87d6dcdfb34a177f3be4d5ba79b492&token=1999457569&lang=zh_CN#rd)
37. [Structured Streaming 实现思路与实现概述](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484959&idx=1&sn=2173f71a32e16b510f047fa716549bc2&chksm=fd3d408aca4ac99cef4ec3079d9c7376c14f457e80f814e1494f5324612b553fb5487783a486&token=1999457569&lang=zh_CN#rd)
38. [Spark之数据倾斜调优](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484956&idx=1&sn=9182a40fcf1fced04acee81aa9261bfe&chksm=fd3d4089ca4ac99f23952f0d627db4600a81808d98b1a635ae40e06c939e4a229a47d666de47&token=1999457569&lang=zh_CN#rd)
39. [你不得不知道的知识-零拷贝](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484766&idx=1&sn=8d0aeaa1166a9338df9f28bb47959f4a&chksm=fd3d43cbca4acadd0dfc9e753ca4fe1cc2ca49060d886b359bdf3537809015b04f82b4be27ac&token=1999457569&lang=zh_CN#rd)
40. [Spark Streaming消费Kafka数据的两种方案](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247484751&idx=1&sn=11315f599b39eac96c17a78da2fa1258&chksm=fd3d43daca4acaccc624947f5fa84f650e7f61d8638feda67db07cacf51bc985faae74cc94c3&token=1999457569&lang=zh_CN#rd)

### 三、Kafka实战进阶文章合集

#### Kafka实战合集

[点我查看Kafka实战合集](实战系列文章/Kafka实战.md)


### 四、数据仓库实战系列

#### 数据仓库实战合集

[点我查看数据仓库实战合集](实战系列文章/数据仓库.md)

### 五、OLAP实战文章系列

### 六、面试系列合集

#### 面试系列合集
---
#### 一、Hadoop
&emsp; 1.[Hadoop面试题总结（一）](面试系列/Hadoop面试题总结/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%80%EF%BC%89.md)  
&emsp; 2.[Hadoop面试题总结（二）——HDFS](面试系列/Hadoop面试题总结/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94HDFS.md)  
&emsp; 3.[Hadoop面试题总结（三）——MapReduce](面试系列/Hadoop面试题总结/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%89%EF%BC%89%E2%80%94%E2%80%94MapReduce.md)  
&emsp; 4.[Hadoop面试题总结（四）——YARN](面试系列/Hadoop面试题总结/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93%EF%BC%88%E5%9B%9B%EF%BC%89%E2%80%94%E2%80%94YARN.md)  
&emsp; 5.[Hadoop面试题总结（五）——优化问题](面试系列/Hadoop面试题总结/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93%EF%BC%88%E4%BA%94%EF%BC%89%E2%80%94%E2%80%94%E4%BC%98%E5%8C%96%E9%97%AE%E9%A2%98.md) 


#### 二、Zookeeper
&emsp; 1.[Zookeeper面试题总结（一）](面试系列/Zookeeper面试题总结/Zookeeper.md)  

#### 三、Hive
&emsp; 1.[Hive面试题总结（一）](面试系列/Hive面试题总结/Hive%EF%BC%88%E4%B8%80%EF%BC%89.md)  
&emsp; 2.[Hive面试题总结（二）](面试系列/Hive面试题总结/Hive%EF%BC%88%E4%BA%8C%EF%BC%89.md)  

#### 四、HBase
&emsp; 1.[HBase面试题总结（一）](面试系列/Hbase面试题整理/HBase.md)  

#### 五、Flume
&emsp; 1.[Flume面试题总结（一）](面试系列/Flume面试题整理/Flume.md)  

#### 六、Kafka
&emsp; 1.[Kafka面试题总结（一）](面试系列/Kafka面试题整理/Kafka%EF%BC%88%E4%B8%80%EF%BC%89.md)  
&emsp; 2.[Kafka面试题总结（二）](面试系列/Kafka面试题整理/Kafka%EF%BC%88%E4%BA%8C%EF%BC%89.md)  

#### 七、Spark
&emsp; 1.[Spark面试题总结（一）](面试系列/Spark面试题整理/Spark%EF%BC%88%E4%B8%80%EF%BC%89.md)  
&emsp; 2.[Spark面试题总结（二）](面试系列/Spark面试题整理/Spark%EF%BC%88%E4%BA%8C%EF%BC%89.md)  
&emsp; 3.[Spark面试题总结（三）](面试系列/Spark面试题整理/Spark%EF%BC%88%E4%B8%89%EF%BC%89.md)  
&emsp; 4.[Spark面试题总结（四）](面试系列/Spark面试题整理/Spark%EF%BC%88%E5%9B%9B%EF%BC%89.md)  
  
&emsp; **Spark性能优化：**  
&emsp; 5.[Spark面试题总结（五）——几种常见的数据倾斜情况及调优方式](面试系列/Spark面试题整理/Spark调优/%E6%95%B0%E6%8D%AE%E5%80%BE%E6%96%9C.md)  
&emsp; 6.[Spark面试题总结（六）——Shuffle配置调优](面试系列/Spark面试题整理/Spark调优/Shuffle%E9%85%8D%E7%BD%AE%E8%B0%83%E4%BC%98.md)  
&emsp; 7.[Spark面试题总结（七）——程序开发调优](面试系列/Spark面试题整理/Spark调优/%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%8F%91%E8%B0%83%E4%BC%98.md)  
&emsp; 8.[Spark面试题总结（八）——运行资源调优](面试系列/Spark面试题整理/Spark调优/%E8%B5%84%E6%BA%90%E8%B0%83%E4%BC%98.md)  


本系列的大纲会根据实际情况进行调整，欢迎大家关注~


## SSS、声明
文档中参考引用了大量网络上的博客和文章，大部分给出了出处，有些没写，如果造成了侵权行为，请您联系我，立即删除~


## QQQ、扫我关注公众号

关注公众号：每天定时推送Hadoop/Spark/Flink等最新的**大数据干货**技术文章,学习资料下载等

<div align="center"> <img width="350px" src="qrcodes/wechat01.png"/> </div>

## KKK、如果对你有用，欢迎请我喝杯咖啡
备注Github，感谢您～
<div align="center"> <img width="350px" src="qrcodes/wechat02.jpeg"/> </div>

## HHH、扫我加群
 备注**来自GitHub加群**，小助手会拉你进大数据讨论组，一起学习交流，期待你的到来~

<div align="center"> <img width="350px" src="qrcodes/个人微信.jpeg"/> </div>

## 为什么有这个文档
- 以前这里只是几个txt文档
- 是我面试腾讯阿里美团等公司大数据开发工程师的过程中总结出来的大数据开发的必知必会的知识点~
- 后续更新在微信公众号更新，欢迎关注~

## 言而总之
**大数据成神之路** 该系列文章将为希望从事大数据开发或者由后端转型为大数据开发的工程师们指出需要学习的知识点和路径，本系列文章同时致敬我曾经在网络上看到无数个Java和大数据系列文章，深受启发同时也收货很多。

欢迎关注公众号‘大数据技术与架构’或者搜索import_bigdata关注~

 
