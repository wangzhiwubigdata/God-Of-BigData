## Spark面试题（八）——Spark的Shuffle配置调优
#### 1、Shuffle优化配置 `-spark.shuffle.file.buffer`  
&emsp; **默认值**：32k  
&emsp; **参数说明**：该参数用于设置shuffle write task的BufferedOutputStream的buffer缓冲大小。将数据写到磁盘文件之前，会先写入buffer缓冲中，待缓冲写满之后，才会溢写到磁盘。         
&emsp; **调优建议**：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如64k），从而减少shuffle write过程中溢写磁盘文件的次数，也就可以减少磁盘IO次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。  

#### 2、Shuffle优化配置 `-spark.reducer.maxSizeInFlight`
&emsp; **默认值**：48m  
&emsp; **参数说明**：该参数用于设置shuffle read task的buffer缓冲大小，而这个buffer缓冲决定了每次能够拉取多少数据。        
&emsp; **调优建议**：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。 

#### 3、Shuffle优化配置 `-spark.shuffle.io.maxRetries`
&emsp; **默认值**：3  
&emsp; **参数说明**：shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败。        
&emsp; **调优建议**：对于那些包含了特别耗时的shuffle操作的作业，建议增加重试最大次数（比如60次），以避免由于JVM的full gc或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的shuffle过程，调节该参数可以大幅度提升稳定性。  

#### 4、Shuffle优化配置 `-spark.shuffle.io.retryWait`
&emsp; **默认值**：5s  
&emsp; **参数说明**： shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的，该参数代表了每次重试拉取数据的等待间隔，默认是5s。  
&emsp; **调优建议**：建议加大间隔时长（比如60s），以增加shuffle操作的稳定性。  

#### 5、Shuffle优化配置 `-spark.shuffle.memoryFraction`
&emsp; **默认值**：0.2  
&emsp; **参数说明**：该参数代表了Executor内存中，分配给shuffle read task进行聚合操作的内存比例，默认是20%。      
&emsp; **调优建议**：在资源参数调优中讲解过这个参数。如果内存充足，而且很少使用持久化操作，建议调高这个比例，给shuffle read的聚合操作更多内存，以避免由于内存不足导致聚合过程中频繁读写磁盘。在实践中发现，合理调节该参数可以将性能提升10%左右。  

#### 6、Shuffle优化配置 `-spark.shuffle.manager`
&emsp; **默认值**：sort  
&emsp; **参数说明**：该参数用于设置ShuffleManager的类型。Spark 1.5以后，有三个可选项：hash、sort和tungsten-sort。HashShuffleManager是Spark 1.2以前的默认选项，但是Spark 1.2以及之后的版本默认都是SortShuffleManager了。tungsten-sort与sort类似，但是使用了tungsten计划中的堆外内存管理机制，内存使用效率更高。       
&emsp; **调优建议**：由于SortShuffleManager默认会对数据进行排序，因此如果你的业务逻辑中需要该排序机制的话，则使用默认的SortShuffleManager就可以；而如果你的业务逻辑不需要对数据进行排序，那么建议参考后面的几个参数调优，通过bypass机制或优化的HashShuffleManager来避免排序操作，同时提供较好的磁盘读写性能。这里要注意的是，tungsten-sort要慎用，因为之前发现了一些相应的bug。  

#### 7、Shuffle优化配置 `-spark.shuffle.sort.bypassMergeThreshold`
&emsp; **默认值**：200   
&emsp; **参数说明**：当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。      
&emsp; **调优建议**：当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。  

#### 8、Shuffle优化配置 `-spark.shuffle.consolidateFiles`
&emsp; **默认值**：false  
&emsp; **参数说明**：如果使用HashShuffleManager，该参数有效。如果设置为true，那么就会开启consolidate机制，会大幅度合并shuffle write的输出文件，对于shuffle read task数量特别多的情况下，这种方法可以极大地减少磁盘IO开销，提升性能。       
&emsp; **调优建议**：如果的确不需要SortShuffleManager的排序机制，那么除了使用bypass机制，还可以尝试将spark.shffle.manager参数手动指定为hash，使用HashShuffleManager，同时开启consolidate机制。在实践中尝试过，发现其性能比开启了bypass机制的SortShuffleManager要高出10%~30%。  

#### 总结：
&emsp; 1、`spark.shuffle.file.buffer`：主要是设置的Shuffle过程中写文件的缓冲，默认32k，如果内存足够，可以适当调大，来减少写入磁盘的数量。  
&emsp; 2、`spark.reducer.maxSizeInFight`：主要是设置Shuffle过程中读文件的缓冲区，一次能够读取多少数据，如果内存足够，可以适当扩大，减少整个网络传输次数。  
&emsp; 3、`spark.shuffle.io.maxRetries`：主要是设置网络连接失败时，重试次数，适当调大能够增加稳定性。  
&emsp; 4、`spark.shuffle.io.retryWait`：主要设置每次重试之间的间隔时间，可以适当调大，增加程序稳定性。  
&emsp; 5、`spark.shuffle.memoryFraction`：Shuffle过程中的内存占用，如果程序中较多使用了Shuffle操作，那么可以适当调大该区域。  
&emsp; 6、`spark.shuffle.manager`：Hash和Sort方式，Sort是默认，Hash在reduce数量 比较少的时候，效率会很高。  
&emsp; 7、`spark.shuffle.sort. bypassMergeThreshold`：设置的是Sort方式中，启用Hash输出方式的临界值，如果你的程序数据不需要排序，而且reduce数量比较少，那推荐可以适当增大临界值。  
&emsp; 8、`spark. shuffle.cosolidateFiles`：如果你使用Hash shuffle方式，推荐打开该配置，实现更少的文件输出。  



















