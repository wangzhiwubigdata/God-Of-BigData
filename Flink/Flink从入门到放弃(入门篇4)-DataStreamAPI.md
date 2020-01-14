
DataStream算子将一个或多个DataStream转换为新DataStream。程序可以将多个转换组合成复杂的数据流拓扑。
DataStreamAPI和DataSetAPI主要的区别在于Transformation部分。
## DataStream Transformation

### map
* DataStream→DataStream
用一个数据元生成一个数据元。一个map函数，它将输入流的值加倍：
```
DataStream<Integer> dataStream = //...
dataStream.map(new MapFunction<Integer, Integer>() {
    @Override
    public Integer map(Integer value) throws Exception {
        return 2 * value;
    }
});
```


### FlatMap

* DataStream→DataStream

采用一个数据元并生成零个，一个或多个数据元。将句子分割为单词的flatmap函数：

```
dataStream.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String value, Collector<String> out)
        throws Exception {
        for(String word: value.split(" ")){
            out.collect(word);
        }
    }
});
```

### Filter
* DataStream→DataStream	
计算每个数据元的布尔函数，并保存函数返回true的数据元。过滤掉零值的过滤器：

```
dataStream.filter(new FilterFunction<Integer>() {
    @Override
    public boolean filter(Integer value) throws Exception {
        return value != 0;
    }
});
```

### KeyBy
* DataStream→KeyedStream

逻辑上将流分区为不相交的分区。具有相同Keys的所有记录都分配给同一分区。在内部，keyBy（）是使用散列分区实现的。指定键有不同的方法。

此转换返回KeyedStream，其中包括使用被Keys化状态所需的KeyedStream。

```
dataStream.keyBy("someKey") // Key by field "someKey"
dataStream.keyBy(0) // Key by the first element of a Tuple
```

🌺注意：

如果出现以下情况，则类型不能成为key：

* 它是POJO类型但不覆盖hashCode（）方法并依赖于Object.hashCode（）实现

* 任何类型的数组

### Reduce
KeyedStream→DataStream

将当前数据元与最后一个Reduce的值组合并发出新值。 
例如：reduce函数，用于创建部分和的流：

```
keyedStream.reduce(new ReduceFunction<Integer>() {
    @Override
    public Integer reduce(Integer value1, Integer value2)
    throws Exception {
        return value1 + value2;
    }
});   
```
### Fold
KeyedStream→DataStream

具有初始值的被Keys化数据流上的“滚动”折叠。将当前数据元与最后折叠的值组合并发出新值。 

折叠函数，当应用于序列（1,2,3,4,5）时，发出序列“start-1”，“start-1-2”，“start-1-2-3”,. ..

```
DataStream<String> result =
  keyedStream.fold("start", new FoldFunction<Integer, String>() {
    @Override
    public String fold(String current, Integer value) {
        return current + "-" + value;
    }
  });
```

### 聚合
* KeyedStream→DataStream

在被Keys化数据流上滚动聚合。min和minBy之间的差异是min返回最小值，而minBy返回该字段中具有最小值的数据元(max和maxBy相同)。

```
keyedStream.sum(0);
keyedStream.sum("key");
keyedStream.min(0);
keyedStream.min("key");
keyedStream.max(0);
keyedStream.max("key");
keyedStream.minBy(0);
keyedStream.minBy("key");
keyedStream.maxBy(0);
keyedStream.maxBy("key");
```

### Window函数

关于Flink的窗口概念，我们会在后面有详细介绍。

* Window
KeyedStream→WindowedStream

可以在已经分区的KeyedStream上定义Windows。Windows根据某些特征（例如，在最后5秒内到达的数据）对每个Keys中的数据进行分组。

```
dataStream.keyBy(0)
.window(TumblingEventTimeWindows
.of(Time.seconds(5))); // Last 5 seconds of data
    
```
* Window Apply
WindowedStream→DataStream 
AllWindowedStream→DataStream

将一般函数应用于整个窗口。下面是一个手动求和窗口数据元的函数。

注意：如果您正在使用windowAll转换，则需要使用AllWindowFunction。

```
windowedStream.apply (new WindowFunction<Tuple2<String,Integer>, Integer, Tuple, Window>() {
    public void apply (Tuple tuple,
            Window window,
            Iterable<Tuple2<String, Integer>> values,
            Collector<Integer> out) throws Exception {
        int sum = 0;
        for (value t: values) {
            sum += t.f1;
        }
        out.collect (new Integer(sum));
    }
});

// applying an AllWindowFunction on non-keyed window stream
allWindowedStream.apply (new AllWindowFunction<Tuple2<String,Integer>, Integer, Window>() {
    public void apply (Window window,
            Iterable<Tuple2<String, Integer>> values,
            Collector<Integer> out) throws Exception {
        int sum = 0;
        for (value t: values) {
            sum += t.f1;
        }
        out.collect (new Integer(sum));
    }
});
    
```

* Window Reduce
WindowedStream→DataStream

将reduce函数应用于窗口并返回reduce后的值。

```
windowedStream.reduce (new ReduceFunction<Tuple2<String,Integer>>() {
    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
        return new Tuple2<String,Integer>(value1.f0, value1.f1 + value2.f1);
    }
});
```

* 提取时间戳
>关于Time我们在后面有专门的章节进行介绍


DataStream→DataStream

从记录中提取时间戳，以便使用使用事件时间语义的窗口。

```
stream.assignTimestamps (new TimeStampExtractor() {...});
```

### Partition 分区

* 自定义分区
DataStream→DataStream	
使用用户定义的分区程序为每个数据元选择目标任务。

```
dataStream.partitionCustom(partitioner, "someKey");
dataStream.partitionCustom(partitioner, 0);
```

* 随机分区
DataStream→DataStream	
根据均匀分布随机分配数据元。
```
dataStream.shuffle();
```     
* Rebalance （循环分区）
DataStream→DataStream	
分区数据元循环，每个分区创建相等的负载。在存在数据倾斜时用于性能优化。
```
dataStream.rebalance();
```

* rescale
DataStream→DataStream

如果上游 算子操作具有并行性2并且下游算子操作具有并行性6，则一个上游 算子操作将分配元件到三个下游算子操作，而另一个上游算子操作将分配到其他三个下游 算子操作。另一方面，如果下游算子操作具有并行性2而上游 算子操作具有并行性6，则三个上游 算子操作将分配到一个下游算子操作，而其他三个上游算子操作将分配到另一个下游算子操作。

在不同并行度不是彼此的倍数的情况下，一个或多个下游 算子操作将具有来自上游 算子操作的不同数量的输入。

请参阅此图以获取上例中连接模式的可视化：

![5bd63a6c99ad06ba3d96d03be3cb25ff.svg+xml](evernotecid://DF961740-2AB0-48AB-AAE7-53BB9D286C7A/appyinxiangcom/12131181/ENResource/p1410)
```
dataStream.rescale();
```

* 广播
DataStream→DataStream	
向每个分区广播数据元。
```
dataStream.broadcast();
```      