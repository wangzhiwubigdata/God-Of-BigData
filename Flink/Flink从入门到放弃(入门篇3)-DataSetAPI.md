
## 编程结构

```
public class SocketTextStreamWordCount {

	public static void main(String[] args) throws Exception {
		if (args.length != 2){
System.err.println("USAGE:\nSocketTextStreamWordCount <hostname> <port>");
			return;
		}
		String hostName = args[0];
		Integer port = Integer.parseInt(args[1]);
		final StreamExecutionEnvironment env = StreamExecutionEnvironment
				.getExecutionEnvironment();
		DataStream<String> text = env.socketTextStream(hostName, port);

		DataStream<Tuple2<String, Integer>> counts 
		text.flatMap(new LineSplitter())
				.keyBy(0)
				.sum(1);
		counts.print();
		env.execute("Java WordCount from SocketTextStream Example");
	}
```
上面的`SocketTextStreamWordCount`是一个典型的Flink程序，他由一下及格部分构成：
* 获得一个execution environment，
* 加载/创建初始数据，
* 指定此数据的转换，
* 指定放置计算结果的位置，
* 触发程序执行



## DataSet API
分类：

* Source: 数据源创建初始数据集，例如来自文件或Java集合
* Transformation: 数据转换将一个或多个DataSet转换为新的DataSet
* Sink: 将计算结果存储或返回

### DataSet Sources

#### 基于文件的

* `readTextFile(path)/ TextInputFormat`- 按行读取文件并将其作为字符串返回。

* `readTextFileWithValue(path)/ TextValueInputFormat`- 按行读取文件并将它们作为StringValues返回。StringValues是可变字符串。

* `readCsvFile(path)/ CsvInputFormat`- 解析逗号（或其他字符）分隔字段的文件。返回元组或POJO的DataSet。支持基本java类型及其Value对应作为字段类型。

* `readFileOfPrimitives(path, Class)/ PrimitiveInputFormat`- 解析新行（或其他字符序列）分隔的原始数据类型（如String或）的文件Integer。

* `readFileOfPrimitives(path, delimiter, Class)/ PrimitiveInputFormat`- 解析新行（或其他字符序列）分隔的原始数据类型的文件，例如String或Integer使用给定的分隔符。

* `readSequenceFile(Key, Value, path)/ SequenceFileInputFormat`- 创建一个JobConf并从类型为SequenceFileInputFormat，Key class和Value类的指定路径中读取文件，并将它们作为Tuple2 <Key，Value>返回。

#### 基于集合

* `fromCollection(Collection)` - 从Java Java.util.Collection创建数据集。集合中的所有数据元必须属于同一类型。

* `fromCollection(Iterator, Class)` - 从迭代器创建数据集。该类指定迭代器返回的数据元的数据类型。

* `fromElements(T ...)` - 根据给定的对象序列创建数据集。所有对象必须属于同一类型。

* `fromParallelCollection(SplittableIterator, Class) `- 并行地从迭代器创建数据集。该类指定迭代器返回的数据元的数据类型。

* `generateSequence(from, to)` - 并行生成给定间隔中的数字序列。

#### 通用方法

* `readFile(inputFormat, path)/ FileInputFormat`- 接受文件输入格式。

* `createInput(inputFormat)/ InputFormat`- 接受通用输入格式。

#### 代码示例

```
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// 从本地文件系统读
DataSet<String> localLines = env.readTextFile("file:///path/to/my/textfile");

// 读取HDFS文件
DataSet<String> hdfsLines = env.readTextFile("hdfs://nnHost:nnPort/path/to/my/textfile");

// 读取CSV文件
DataSet<Tuple3<Integer, String, Double>> csvInput = env.readCsvFile("hdfs:///the/CSV/file").types(Integer.class, String.class, Double.class);

// 读取CSV文件中的部分
DataSet<Tuple2<String, Double>> csvInput = env.readCsvFile("hdfs:///the/CSV/file").includeFields("10010").types(String.class, Double.class);

// 读取CSV映射为一个java类
DataSet<Person>> csvInput = env.readCsvFile("hdfs:///the/CSV/file").pojoType(Person.class, "name", "age", "zipcode");

// 读取一个指定位置序列化好的文件
DataSet<Tuple2<IntWritable, Text>> tuples =
 env.readSequenceFile(IntWritable.class, Text.class, "hdfs://nnHost:nnPort/path/to/file");

// 从输入字符创建
DataSet<String> value = env.fromElements("Foo", "bar", "foobar", "fubar");

// 创建一个数字序列
DataSet<Long> numbers = env.generateSequence(1, 10000000);

// 从关系型数据库读取
DataSet<Tuple2<String, Integer> dbData =
env.createInput(JDBCInputFormat.buildJDBCInputFormat()                    .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")                   .setDBUrl("jdbc:derby:memory:persons")
.setQuery("select name, age from persons")
.setRowTypeInfo(new RowTypeInfo(BasicTypeInfo.STRING_TYPE_INFO, BasicTypeInfo.INT_TYPE_INFO))
.finish());
```

### DataSet Transformation

> 详细可以参考官网:https://flink.sojb.cn/dev/batch/dataset_transformations.html#filter

* Map	


采用一个数据元并生成一个数据元。
```
data.map(new MapFunction<String, Integer>() {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```
* FlatMap	

采用一个数据元并生成零个，一个或多个数据元。
```
data.flatMap(new FlatMapFunction<String, String>() {
  public void flatMap(String value, Collector<String> out) {
    for (String s : value.split(" ")) {
      out.collect(s);
    }
  }
});
```

* MapPartition


在单个函数调用中转换并行分区。该函数将分区作为Iterable流来获取，并且可以生成任意数量的结果值。每个分区中的数据元数量取决于并行度和先前的 算子操作。
```
data.mapPartition(new MapPartitionFunction<String, Long>() {
  public void mapPartition(Iterable<String> values, Collector<Long> out) {
    long c = 0;
    for (String s : values) {
      c++;
    }
    out.collect(c);
  }
});
```
* Filter

计算每个数据元的布尔函数，并保存函数返回true的数据元。
重要信息：系统假定该函数不会修改应用谓词的数据元。违反此假设可能会导致错误的结果。
```
data.filter(new FilterFunction<Integer>() {
  public boolean filter(Integer value) { return value > 1000; }
});
```
* Reduce	

通过将两个数据元重复组合成一个数据元，将一组数据元组合成一个数据元。Reduce可以应用于完整数据集或分组数据集。
```
data.reduce(new ReduceFunction<Integer> {
  public Integer reduce(Integer a, Integer b) { return a + b; }
});
```
如果将reduce应用于分组数据集，则可以通过提供CombineHintto 来指定运行时执行reduce的组合阶段的方式 setCombineHint。在大多数情况下，基于散列的策略应该更快，特别是如果不同键的数量与输入数据元的数量相比较小（例如1/10）。

* ReduceGroup	


将一组数据元组合成一个或多个数据元。ReduceGroup可以应用于完整数据集或分组数据集。

```
data.reduceGroup(new GroupReduceFunction<Integer, Integer> {
  public void reduce(Iterable<Integer> values, Collector<Integer> out) {
    int prefixSum = 0;
    for (Integer i : values) {
      prefixSum += i;
      out.collect(prefixSum);
    }
  }
});
```

* Aggregate	


将一组值聚合为单个值。聚合函数可以被认为是内置的reduce函数。聚合可以应用于完整数据集或分组数据集。

```
Dataset<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input.aggregate(SUM, 0).and(MIN, 2);
```

您还可以使用简写语法进行最小，最大和总和聚合。

```
Dataset<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input.sum(0).andMin(2);
```
* Distinct	

返回数据集的不同数据元。它相对于数据元的所有字段或字段子集从输入DataSet中删除重复条目。
```
data.distinct();
```
使用reduce函数实现Distinct。您可以通过提供CombineHintto 来指定运行时执行reduce的组合阶段的方式 setCombineHint。在大多数情况下，基于散列的策略应该更快，特别是如果不同键的数量与输入数据元的数量相比较小（例如1/10）。

* Join	

通过创建在其键上相等的所有数据元对来连接两个数据集。可选地使用JoinFunction将数据元对转换为单个数据元，或使用FlatJoinFunction将数据元对转换为任意多个（包括无）数据元。请参阅键部分以了解如何定义连接键。

```
result = input1.join(input2)
               .where(0)       // key of the first input (tuple field 0)
               .equalTo(1);    // key of the second input (tuple field 1)
```

您可以通过Join Hints指定运行时执行连接的方式。提示描述了通过分区或广播进行连接，以及它是使用基于排序还是基于散列的算法。
如果未指定提示，系统将尝试估算输入大小，并根据这些估计选择最佳策略。

```
// This executes a join by broadcasting the first data set
// using a hash table for the broadcast data
result = input1.join(input2, JoinHint.BROADCAST_HASH_FIRST)
               .where(0).equalTo(1);
```

请注意，连接转换仅适用于等连接。其他连接类型需要使用OuterJoin或CoGroup表示。

* OuterJoin	

在两个数据集上执行左，右或全外连接。外连接类似于常规（内部）连接，并创建在其键上相等的所有数据元对。此外，如果在另一侧没有找到匹配的Keys，则保存“外部”侧（左侧，右侧或两者都满）的记录。匹配数据元对（或一个数据元和null另一个输入的值）被赋予JoinFunction以将数据元对转换为单个数据元，或者转换为FlatJoinFunction以将数据元对转换为任意多个（包括无）数据元。请参阅键部分以了解如何定义连接键。

```
input1.leftOuterJoin(input2) // rightOuterJoin or fullOuterJoin for right or full outer joins
      .where(0)              // key of the first input (tuple field 0)
      .equalTo(1)            // key of the second input (tuple field 1)
      .with(new JoinFunction<String, String, String>() {
          public String join(String v1, String v2) {
             // NOTE:
             // - v2 might be null for leftOuterJoin
             // - v1 might be null for rightOuterJoin
             // - v1 OR v2 might be null for fullOuterJoin
          }
      });

```

* CoGroup	


reduce 算子操作的二维变体。将一个或多个字段上的每个输入分组，然后关联组。每对组调用转换函数。

```
data1.coGroup(data2)
     .where(0)
     .equalTo(1)
     .with(new CoGroupFunction<String, String, String>() {
         public void coGroup(Iterable<String> in1, Iterable<String> in2, Collector<String> out) {
           out.collect(...);
         }
      });
```

* Cross	


构建两个输入的笛卡尔积（交叉乘积），创建所有数据元对。可选择使用CrossFunction将数据元对转换为单个数据元

```
DataSet<Integer> data1 = // [...]
DataSet<String> data2 = // [...]
DataSet<Tuple2<Integer, String>> result = data1.cross(data2);
```

注：交叉是一个潜在的非常计算密集型 算子操作它甚至可以挑战大的计算集群！建议使用crossWithTiny（）和crossWithHuge（）来提示系统的DataSet大小。

* Union	


生成两个数据集的并集。

```
DataSet<String> data1 = // [...]
DataSet<String> data2 = // [...]
DataSet<String> result = data1.union(data2);
```

* Rebalance	


均匀地Rebalance 数据集的并行分区以消除数据偏差。只有类似Map的转换可能会遵循Rebalance 转换。

```
DataSet<String> in = // [...]
DataSet<String> result = in.rebalance()
                           .map(new Mapper());
                           
```

* Hash-Partition


散列分区给定键上的数据集。键可以指定为位置键，表达键和键选择器函数。

```
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionByHash(0)
                            .mapPartition(new PartitionMapper());
```

* Range-Partition


Range-Partition给定键上的数据集。键可以指定为位置键，表达键和键选择器函数。

```
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionByRange(0)
                            .mapPartition(new PartitionMapper());
```

* Custom Partitioning


手动指定数据分区。 
注意：此方法仅适用于单个字段键。

```
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionCustom(Partitioner<K> partitioner, key)
```

* Sort Partition	


本地按指定顺序对指定字段上的数据集的所有分区进行排序。可以将字段指定为元组位置或字段表达式。通过链接sortPartition（）调用来完成对多个字段的排序。

```
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.sortPartition(1, Order.ASCENDING)
                            .mapPartition(new PartitionMapper());
```

* First-n	


返回数据集的前n个（任意）数据元。First-n可以应用于常规数据集，分组数据集或分组排序数据集。分组键可以指定为键选择器函数或字段位置键。

```
DataSet<Tuple2<String,Integer>> in = // [...]
// regular data set
DataSet<Tuple2<String,Integer>> result1 = in.first(3);
// grouped data set
DataSet<Tuple2<String,Integer>> result2 = in.groupBy(0)                                     .first(3);
// grouped-sorted data set
DataSet<Tuple2<String,Integer>> result3 = in.groupBy(0)                                     .sortGroup(1, Order.ASCENDING)                     .first(3);

```

### DataSet Sink

数据接收器使用DataSet用于存储或返回。使用OutputFormat描述数据接收器算子操作 。Flink带有各种内置输出格式，这些格式封装在DataSet上的算子操作中：

* writeAsText()/ TextOutputFormat- 按字符串顺序写入数据元。通过调用每个数据元的toString（）方法获得字符串。
* writeAsFormattedText()/ TextOutputFormat- 按字符串顺序写数据元。通过为每个数据元调用用户定义的format（）方法来获取字符串。
* writeAsCsv(...)/ CsvOutputFormat- 将元组写为逗号分隔值文件。行和字段分隔符是可配置的。每个字段的值来自对象的toString（）方法。
* print()/ printToErr()/ print(String msg)/ printToErr(String msg)- 在标准输出/标准错误流上打印每个数据元的toString（）值。可选地，可以提供前缀（msg），其前缀为输出。这有助于区分不同的打印调用。如果并行度大于1，则输出也将与生成输出的任务的标识符一起添加。
* write()/ FileOutputFormat- 自定义文件输出的方法和基类。支持自定义对象到字节的转换。
* output()/ OutputFormat- 大多数通用输出方法，用于非基于文件的数据接收器（例如将结果存储在数据库中）。

可以将DataSet输入到多个 算子操作。程序可以编写或打印数据集，同时对它们执行其他转换。

示例：

```
// text data
DataSet<String> textData = // [...]

// write DataSet to a file on the local file system
textData.writeAsText("file:///my/result/on/localFS");

// write DataSet to a file on a HDFS with a namenode running at nnHost:nnPort
textData.writeAsText("hdfs://nnHost:nnPort/my/result/on/localFS");

// write DataSet to a file and overwrite the file if it exists
textData.writeAsText("file:///my/result/on/localFS", WriteMode.OVERWRITE);

// tuples as lines with pipe as the separator "a|b|c"
DataSet<Tuple3<String, Integer, Double>> values = // [...]
values.writeAsCsv("file:///path/to/the/result/file", "\n", "|");

// this writes tuples in the text formatting "(a, b, c)", rather than as CSV lines
values.writeAsText("file:///path/to/the/result/file");

// this writes values as strings using a user-defined TextFormatter object
values.writeAsFormattedText("file:///path/to/the/result/file",
    new TextFormatter<Tuple2<Integer, Integer>>() {
        public String format (Tuple2<Integer, Integer> value) {
            return value.f1 + " - " + value.f0;
        }
    });
```

使用自定义输出格式：

```
DataSet<Tuple3<String, Integer, Double>> myResult = [...]

// write Tuple DataSet to a relational database
myResult.output(
    // build and configure OutputFormat
    JDBCOutputFormat.buildJDBCOutputFormat()
                    .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")
                    .setDBUrl("jdbc:derby:memory:persons")
                    .setQuery("insert into persons (name, age, height) values (?,?,?)")
                    .finish()
    );
```

## 序列化器

* Flink自带了针对诸如int，long，String等标准类型的序列化器


* 针对Flink无法实现序列化的数据类型，我们可以交给Avro和Kryo


* 使用方法：ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

```
使用avro序列化：env.getConfig().enableForceAvro();
使用kryo序列化：env.getConfig().enableForceKryo();
使用自定义序列化：env.getConfig().addDefaultKryoSerializer(Class<?> type, Class<? extends Serializer<?>> serializerClass)

```

## 数据类型

* Java Tuple 和 Scala case class

* Java POJOs：java实体类

* Primitive Types
  默认支持java和scala基本数据类型

* General Class Types
  默认支持大多数java和scala class

* Hadoop Writables
  支持hadoop中实现了org.apache.hadoop.Writable的数据类型


* Special Types
  例如scala中的Either Option 和Try

