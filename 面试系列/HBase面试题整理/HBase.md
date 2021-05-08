## HBase面试题整理（一）  

### 1、 HBase的特点是什么？   
&emsp; 1）大：一个表可以有数十亿行，上百万列；  
&emsp; 2）无模式：每行都有一个可排序的主键和任意多的列，列可以根据需要动态的增加，同一张表中不同的行可以有截然不同的列；  
&emsp; 3）面向列：面向列（族）的存储和权限控制，列（族）独立检索；  
&emsp; 4）稀疏：空（null）列并不占用存储空间，表可以设计的非常稀疏；  
&emsp; 5）数据多版本：每个单元中的数据可以有多个版本，默认情况下版本号自动分配，是单元格插入时的时间戳；  
&emsp; 6）数据类型单一：Hbase中的数据都是字符串，没有类型。  

### 2、HBase和Hive的区别？  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/HBase%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HBase%E5%92%8CHive%E5%8C%BA%E5%88%AB.png"/>  
<p align="center">
</p>
</p>  

 &emsp; Hive和Hbase是两种基于Hadoop的不同技术--Hive是一种类SQL的引擎，并且运行MapReduce任务，Hbase是一种在Hadoop之上的NoSQL 的Key/vale数据库。
 当然，这两种工具是可以同时使用的。就像用Google来搜索，用FaceBook进行社交一样，Hive可以用来进行统计查询，HBase可以用来进行实时查询，
 数据也可以从Hive写到Hbase，设置再从Hbase写回Hive。  

### 3、HBase适用于怎样的情景？  
&emsp; ① 半结构化或非结构化数据 
&emsp; 对于数据结构字段不够确定或杂乱无章很难按一个概念去进行抽取的数据适合用HBase。以上面的例子为例，当业务发展需要存储author的email，phone，
address信息时RDBMS需要停机维护，而HBase支持动态增加。  
&emsp; ② 记录非常稀疏  
&emsp; RDBMS的行有多少列是固定的，为null的列浪费了存储空间。而如上文提到的，HBase为null的Column不会被存储，这样既节省了空间又提高了读性能。  
&emsp; ③ 多版本数据  
&emsp; 如上文提到的根据Row key和Column key定位到的Value可以有任意数量的版本值，因此对于需要存储变动历史记录的数据，用HBase就非常方便了。
比如上例中的author的Address是会变动的，业务上一般只需要最新的值，但有时可能需要查询到历史值。  
&emsp; ④ 超大数据量  
&emsp; 当数据量越来越大，RDBMS数据库撑不住了，就出现了读写分离策略，通过一个Master专门负责写操作，多个Slave负责读操作，服务器成本倍增。
随着压力增加，Master撑不住了，这时就要分库了，把关联不大的数据分开部署，一些join查询不能用了，需要借助中间层。随着数据量的进一步增加，
一个表的记录越来越大，查询就变得很慢，于是又得搞分表，比如按ID取模分成多个表以减少单个表的记录数。经历过这些事的人都知道过程是多么的折腾。
采用HBase就简单了，只需要加机器即可，HBase会自动水平切分扩展，跟Hadoop的无缝集成保障了其数据可靠性（HDFS）和海量数据分析的高性能（MapReduce）。  

### 4、描述HBase的rowKey的设计原则？（☆☆☆☆☆）  
（1）Rowkey长度原则  
&emsp; Rowkey 是一个二进制码流，Rowkey 的长度被很多开发者建议说设计在10~100 个字节，不过建议是越短越好，不要超过16 个字节。  
&emsp; 原因如下：  
&emsp; ① 数据的持久化文件HFile 中是按照KeyValue 存储的，如果Rowkey 过长比如100 个字节，1000 万列数据光Rowkey 就要占用100*1000 万=10 亿个字节，
将近1G 数据，这会极大影响HFile 的存储效率；  
&emsp; ② MemStore 将缓存部分数据到内存，如果Rowkey 字段过长内存的有效利用率会降低，系统将无法缓存更多的数据，这会降低检索效率。
因此Rowkey 的字节长度越短越好。  
&emsp; ③ 目前操作系统是都是64 位系统，内存8 字节对齐。控制在16 个字节，8 字节的整数倍利用操作系统的最佳特性。  
（2）Rowkey散列原则  
&emsp; 如果Rowkey是按时间戳的方式递增，不要将时间放在二进制码的前面，建议将Rowkey的高位作为散列字段，由程序循环生成，低位放时间字段，
这样将提高数据均衡分布在每个Regionserver 实现负载均衡的几率。如果没有散列字段，首字段直接是时间信息将产生所有新数据都在一个 RegionServer 上堆积的
热点现象，这样在做数据检索的时候负载将会集中在个别RegionServer，降低查询效率。  
（3）Rowkey唯一原则  
&emsp; 必须在设计上保证其唯一性。  

### 5、描述HBase中scan和get的功能以及实现的异同？（☆☆☆☆☆）  
HBase的查询实现只提供两种方式：  
&emsp; 1）按指定RowKey 获取唯一一条记录，get方法（org.apache.hadoop.hbase.client.Get） Get 的方法处理分两种 : 设置了ClosestRowBefore 和
没有设置ClosestRowBefore的rowlock。主要是用来保证行的事务性，即每个get 是以一个row 来标记的。一个row中可以有很多family 和column。  
&emsp; 2）按指定的条件获取一批记录，scan方法(org.apache.Hadoop.hbase.client.Scan）实现条件查询功能使用的就是scan 方式。  
&emsp; &emsp; （1）scan 可以通过setCaching 与setBatch 方法提高速度(以空间换时间)；  
&emsp; &emsp; （2）scan 可以通过setStartRow 与setEndRow 来限定范围([start，end)start 是闭区间，end 是开区间)。范围越小，性能越高。  
&emsp; &emsp; （3）scan 可以通过setFilter 方法添加过滤器，这也是分页、多条件查询的基础。  

### 6、请描述HBase中scan对象的setCache和setBatch方法的使用？（☆☆☆☆☆）  
&emsp; setCache用于设置缓存，即设置一次RPC请求可以获取多行数据。对于缓存操作，如果行的数据量非常大，多行数据有可能超过客户端进程的内存容量，
由此引入批量处理这一解决方案。  
&emsp; setBatch 用于设置批量处理，批量可以让用户选择每一次ResultScanner实例的next操作要取回多少列，例如，在扫描中设置setBatch(5)，
则一次next()返回的Result实例会包括5列。如果一行包括的列数超过了批量中设置的值，则可以将这一行分片，每次next操作返回一片，当一行的列数不能被批量中
设置的值整除时，最后一次返回的Result实例会包含比较少的列，如，一行17列，batch设置为5，则一共返回4个Result实例，这4个实例中包括的列数分别
为5、5、5、2。  
&emsp; 组合使用扫描器缓存和批量大小，可以让用户方便地控制扫描一个范围内的行键所需要的RPC调用次数。Cache设置了服务器一次返回的行数，
而Batch设置了服务器一次返回的列数。  
&emsp; 假如我们建立了一张有两个列族的表，添加了10行数据，每个行的每个列族下有10列，这意味着整个表一共有200列（或单元格，因为每个列只有一个版本），
其中每行有20列。  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/HBase%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HBase%E4%B8%ADscan%E5%AF%B9%E8%B1%A1%E7%9A%84setCache%E5%92%8CsetBatch%E6%96%B9%E6%B3%95.png"/>  
<p align="center">
</p>
</p>  

&emsp; ① Batch参数决定了一行数据分为几个Result，它只针对一行数据，Batch再大，也只能将一行的数据放入一个Result中。所以当一行数据有10列，
而Batch为100时，也只能将一行的所有列都放入一个Result，不会混合其他行；  
&emsp; ② 缓存值决定一次RPC返回几个Result，根据Batch划分的Result个数除以缓存个数可以得到RPC消息个数（之前定义缓存值决定一次返回的行数，
这是不准确的，准确来说是决定一次RPC返回的Result个数，由于在引入Batch之前，一行封装为一个Result，因此定义缓存值决定一次返回的行数，但引入Batch后，
更准确的说法是缓存值决定了一次RPC返回的Result个数）；  
&emsp; &emsp; RPC请求次数 = （行数 * 每行列数） / Min（每行的列数，批量大小） / 扫描器缓存  
&emsp; 下图展示了缓存和批量两个参数如何联动，下图中有一个包含9行数据的表，每行都包含一些列。使用了一个缓存为6、批量大小为3的扫描器，
需要三次RPC请求来传送数据：  
<p align="center">
<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/blob/master/%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97/pics/HBase%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HBase%20Table.png"/>  
<p align="center">
</p>
</p>  

### 7、请详细描述HBase中一个cell的结构？  
&emsp; HBase中通过row和columns确定的为一个存贮单元称为cell。  
&emsp; Cell：由{row key, column(=<family> + <label>), version}唯一确定的单元。cell 中的数据是没有类型的，全部是字节码形式存贮。  

### 8、简述HBase中compact用途是什么，什么时候触发，分为哪两种，有什么区别，有哪些相关配置参数？（☆☆☆☆☆）  
&emsp; 在hbase中每当有memstore数据flush到磁盘之后，就形成一个storefile，当storeFile的数量达到一定程度后，就需要将 storefile 文件来
进行 compaction 操作。  
&emsp; Compact 的作用：  
&emsp; ① 合并文件  
&emsp; ② 清除过期，多余版本的数据  
&emsp; ③ 提高读写数据的效率  
&emsp; HBase 中实现了两种 compaction 的方式：minor and major. 这两种 compaction 方式的区别是：  
&emsp; 1）Minor 操作只用来做部分文件的合并操作以及包括 minVersion=0 并且设置 ttl 的过期版本清理，不做任何删除数据、多版本数据的清理工作。   
&emsp; 2）Major 操作是对 Region 下的HStore下的所有StoreFile执行合并操作，最终的结果是整理合并出一个文件。  

### 9、每天百亿数据存入HBase，如何保证数据的存储正确和在规定的时间里全部录入完毕，不残留数据？（☆☆☆☆☆）  
需求分析：  
&emsp; 1）百亿数据：证明数据量非常大；  
&emsp; 2）存入HBase：证明是跟HBase的写入数据有关；  
&emsp; 3）保证数据的正确：要设计正确的数据结构保证正确性；  
&emsp; 4）在规定时间内完成：对存入速度是有要求的。  
解决思路：  
&emsp; 1）数据量百亿条，什么概念呢？假设一整天60x60x24 = 86400秒都在写入数据，那么每秒的写入条数高达100万条，HBase当然是支持不了每秒百万条数据的，
所以这百亿条数据可能不是通过实时地写入，而是批量地导入。批量导入推荐使用BulkLoad方式（推荐阅读：Spark之读写HBase），性能是普通写入方式几倍以上；  
&emsp; 2）存入HBase：普通写入是用JavaAPI put来实现，批量导入推荐使用BulkLoad；  
&emsp; 3）保证数据的正确：这里需要考虑RowKey的设计、预建分区和列族设计等问题；  
&emsp; 4）在规定时间内完成也就是存入速度不能过慢，并且当然是越快越好，使用BulkLoad。  

### 10、请列举几个HBase优化方法？（☆☆☆☆☆）  
1）减少调整  
&emsp; 减少调整这个如何理解呢？HBase中有几个内容会动态调整，如region（分区）、HFile，所以通过一些方法来减少这些会带来I/O开销的调整。  
&emsp; ① Region  
&emsp; &emsp; 如果没有预建分区的话，那么随着region中条数的增加，region会进行分裂，这将增加I/O开销，所以解决方法就是根据你的RowKey设计来进行预建分区，
减少region的动态分裂。  
&emsp; ② HFile  
&emsp; &emsp; HFile是数据底层存储文件，在每个memstore进行刷新时会生成一个HFile，当HFile增加到一定程度时，会将属于一个region的HFile进行合并，
这个步骤会带来开销但不可避免，但是合并后HFile大小如果大于设定的值，那么HFile会重新分裂。为了减少这样的无谓的I/O开销，建议估计项目数据量大小，
给HFile设定一个合适的值。  
2）减少启停  
&emsp; 数据库事务机制就是为了更好地实现批量写入，较少数据库的开启关闭带来的开销，那么HBase中也存在频繁开启关闭带来的问题。  
&emsp; ① 关闭Compaction，在闲时进行手动Compaction。  
&emsp; &emsp; 因为HBase中存在Minor Compaction和Major Compaction，也就是对HFile进行合并，所谓合并就是I/O读写，大量的HFile进行肯定会带来I/O开销，
甚至是I/O风暴，所以为了避免这种不受控制的意外发生，建议关闭自动Compaction，在闲时进行compaction。  
&emsp; ② 批量数据写入时采用BulkLoad。  
&emsp; 如果通过HBase-Shell或者JavaAPI的put来实现大量数据的写入，那么性能差是肯定并且还可能带来一些意想不到的问题，所以当需要写入大量离线数据时
建议使用BulkLoad。  
3）减少数据量  
&emsp; 虽然我们是在进行大数据开发，但是如果可以通过某些方式在保证数据准确性同时减少数据量，何乐而不为呢？  
&emsp; ① 开启过滤，提高查询速度  
&emsp; &emsp; 开启BloomFilter，BloomFilter是列族级别的过滤，在生成一个StoreFile同时会生成一个MetaBlock，用于查询时过滤数据  
&emsp; ② 使用压缩  
&emsp; &emsp; 一般推荐使用Snappy和LZO压缩  
4）合理设计  
&emsp; 在一张HBase表格中RowKey和ColumnFamily的设计是非常重要，好的设计能够提高性能和保证数据的准确性  
&emsp; ① RowKey设计：应该具备以下几个属性  
&emsp; &emsp; 散列性：散列性能够保证相同相似的rowkey聚合，相异的rowkey分散，有利于查询。  
&emsp; &emsp; 简短性：rowkey作为key的一部分存储在HFile中，如果为了可读性将rowKey设计得过长，那么将会增加存储压力。     
&emsp; &emsp; 唯一性：rowKey必须具备明显的区别性。  
&emsp; &emsp; 业务性：举例来说：  
&emsp; &emsp; 假如我的查询条件比较多，而且不是针对列的条件，那么rowKey的设计就应该支持多条件查询。   
&emsp; &emsp; 如果我的查询要求是最近插入的数据优先，那么rowKey则可以采用叫上Long.Max-时间戳的方式，这样rowKey就是递减排列。  
&emsp; ② 列族的设计：列族的设计需要看应用场景     
&emsp; &emsp; 优势：HBase中数据时按列进行存储的，那么查询某一列族的某一列时就不需要全盘扫描，只需要扫描某一列族，减少了读I/O；
其实多列族设计对减少的作用不是很明显，适用于读多写少的场景  
&emsp; &emsp; 劣势：降低了写的I/O性能。原因如下：数据写到store以后是先缓存在memstore中，同一个region中存在多个列族则存在多个store，
每个store都一个memstore，当其实memstore进行flush时，属于同一个region的store中的memstore都会进行flush，增加I/O开销。  

### 11、Region如何预建分区？  
&emsp; 预分区的目的主要是在创建表的时候指定分区数，提前规划表有多个分区，以及每个分区的区间范围，这样在存储的时候rowkey按照分区的区间存储，
可以避免region热点问题。  
&emsp; 通常有两种方案：  
&emsp; 方案1：shell 方法  
&emsp; &emsp; create 'tb_splits', {NAME => 'cf',VERSIONS=> 3},{SPLITS => ['10','20','30']}  
&emsp; 方案2：JAVA程序控制  
&emsp; &emsp; ① 取样，先随机生成一定数量的rowkey,将取样数据按升序排序放到一个集合里；  
&emsp; &emsp; ② 根据预分区的region个数，对整个集合平均分割，即是相关的splitKeys；  
&emsp; &emsp; ③ HBaseAdmin.createTable(HTableDescriptor tableDescriptor,byte[][]splitkeys)可以指定预分区的splitKey，
即是指定region间的rowkey临界值。  

### 12、HRegionServer宕机如何处理？（☆☆☆☆☆）  
1）ZooKeeper会监控HRegionServer的上下线情况，当ZK发现某个HRegionServer宕机之后会通知HMaster进行失效备援；  
2）该HRegionServer会停止对外提供服务，就是它所负责的region暂时停止对外提供服务；  
3）HMaster会将该HRegionServer所负责的region转移到其他HRegionServer上，并且会对HRegionServer上存在memstore中还未持久化到磁盘中的数据进行恢复；   
4）这个恢复的工作是由WAL重播来完成，这个过程如下：  
&emsp; ① wal实际上就是一个文件，存在/hbase/WAL/对应RegionServer路径下。  
&emsp; ② 宕机发生时，读取该RegionServer所对应的路径下的wal文件，然后根据不同的region切分成不同的临时文件recover.edits。     
&emsp; ③ 当region被分配到新的RegionServer中，RegionServer读取region时会进行是否存在recover.edits，如果有则进行恢复。  

### 13、HBase读写流程？（☆☆☆☆☆）  
&emsp; **读**：  
&emsp; ① HRegionServer保存着meta表以及表数据，要访问表数据，首先Client先去访问zookeeper，从zookeeper里面获取meta表所在的位置信息，
即找到这个meta表在哪个HRegionServer上保存着。  
&emsp; ② 接着Client通过刚才获取到的HRegionServer的IP来访问Meta表所在的HRegionServer，从而读取到Meta，进而获取到Meta表中存放的元数据。  
&emsp; ③ Client通过元数据中存储的信息，访问对应的HRegionServer，然后扫描所在HRegionServer的Memstore和Storefile来查询数据。  
&emsp; ④ 最后HRegionServer把查询到的数据响应给Client。  
&emsp; **写**：  
&emsp; ① Client先访问zookeeper，找到Meta表，并获取Meta表元数据。  
&emsp; ② 确定当前将要写入的数据所对应的HRegion和HRegionServer服务器。  
&emsp; ③ Client向该HRegionServer服务器发起写入数据请求，然后HRegionServer收到请求并响应。  
&emsp; ④ Client先把数据写入到HLog，以防止数据丢失。  
&emsp; ⑤ 然后将数据写入到Memstore。  
&emsp; ⑥ 如果HLog和Memstore均写入成功，则这条数据写入成功。  
&emsp; ⑦ 如果Memstore达到阈值，会把Memstore中的数据flush到Storefile中。  
&emsp; ⑧ 当Storefile越来越多，会触发Compact合并操作，把过多的Storefile合并成一个大的Storefile。  
&emsp; ⑨ 当Storefile越来越大，Region也会越来越大，达到阈值后，会触发Split操作，将Region一分为二。  

### 14、HBase内部机制是什么？  
&emsp; Hbase是一个能适应联机业务的数据库系统  
&emsp; 物理存储：hbase的持久化数据是将数据存储在HDFS上。  
&emsp; 存储管理：一个表是划分为很多region的，这些region分布式地存放在很多regionserver上Region内部还可以划分为store，
store内部有memstore和storefile。  
&emsp; 版本管理：hbase中的数据更新本质上是不断追加新的版本，通过compact操作来做版本间的文件合并Region的split。  
&emsp; 集群管理：ZooKeeper + HMaster + HRegionServer。  

### 15、Hbase中的memstore是用来做什么的？  
&emsp; hbase为了保证随机读取的性能，所以hfile里面的rowkey是有序的。当客户端的请求在到达regionserver之后，为了保证写入rowkey的有序性，
所以不能将数据立刻写入到hfile中，而是将每个变更操作保存在内存中，也就是memstore中。memstore能够很方便的支持操作的随机插入，
并保证所有的操作在内存中是有序的。当memstore达到一定的量之后，会将memstore里面的数据flush到hfile中，这样能充分利用hadoop写入大文件的性能优势，
提高写入性能。  
&emsp; 由于memstore是存放在内存中，如果regionserver因为某种原因死了，会导致内存中数据丢失。所有为了保证数据不丢失，
hbase将更新操作在写入memstore之前会写入到一个write ahead log(WAL)中。WAL文件是追加、顺序写入的，WAL每个regionserver只有一个，
同一个regionserver上所有region写入同一个的WAL文件。这样当某个regionserver失败时，可以通过WAL文件，将所有的操作顺序重新加载到memstore中。  

### 16、HBase在进行模型设计时重点在什么地方？一张表中定义多少个Column Family最合适？为什么？（☆☆☆☆☆）  
&emsp; Column Family的个数具体看表的数据，一般来说划分标准是根据数据访问频度，如一张表里有些列访问相对频繁，而另一些列访问很少，
这时可以把这张表划分成两个列族，分开存储，提高访问效率。  

### 17、如何提高HBase客户端的读写性能？请举例说明（☆☆☆☆☆）  
&emsp; ① 开启bloomfilter过滤器，开启bloomfilter比没开启要快3、4倍  
&emsp; ② Hbase对于内存有特别的需求，在硬件允许的情况下配足够多的内存给它  
&emsp; ③ 通过修改hbase-env.sh中的 export HBASE_HEAPSIZE=3000  #这里默认为1000m  
&emsp; ④ 增大RPC数量  
&emsp; &emsp; 通过修改hbase-site.xml中的hbase.regionserver.handler.count属性，可以适当的放大RPC数量，默认值为10有点小。  

### 18、HBase集群安装注意事项？  
&emsp; ① HBase需要HDFS的支持，因此安装HBase前确保Hadoop集群安装完成；  
&emsp; ② HBase需要ZooKeeper集群的支持，因此安装HBase前确保ZooKeeper集群安装完成；  
&emsp; ③ 注意HBase与Hadoop的版本兼容性；  
&emsp; ④ 注意hbase-env.sh配置文件和hbase-site.xml配置文件的正确配置；  
&emsp; ⑤ 注意regionservers配置文件的修改；  
&emsp; ⑥ 注意集群中的各个节点的时间必须同步，否则启动HBase集群将会报错。  

### 19、直接将时间戳作为行健，在写入单个region 时候会发生热点问题，为什么呢？（☆☆☆☆☆）  
&emsp; region中的rowkey是有序存储，若时间比较集中。就会存储到一个region中，这样一个region的数据变多，其它的region数据很少，加载数据就会很慢，
直到region分裂，此问题才会得到缓解。  

### 20、请描述如何解决HBase中region太小和region太大带来的冲突？（☆☆☆☆☆）  
&emsp; Region过大会发生多次compaction，将数据读一遍并重写一遍到hdfs 上，占用io，region过小会造成多次split，region 会下线，影响访问服务，
最佳的解决方法是调整hbase.hregion. max.filesize 为256m。  








