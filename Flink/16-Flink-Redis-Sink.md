## 简介

流式计算中，我们经常有一些场景是消费Kafka数据，进行处理，然后存储到其他的数据库或者缓存或者重新发送回其他的消息队列中。
本文讲述一个简单的Redis作为Sink的案例。
后续，我们会补充完善，比如落入Hbase，Kafka，Mysql等。


## 关于Redis Sink

Flink提供了封装好的写入Redis的包给我们用，首先我们要新增一个依赖：
```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-redis_2.10</artifactId>
    <version>1.1.5</version>
</dependency>

```


然后我们实现一个自己的RedisSinkExample：

```
//指定Redis set
public static final class RedisSinkExample implements RedisMapper<Tuple2<String,Integer>> {
public RedisCommandDescription getCommandDescription() {
    return new RedisCommandDescription(RedisCommand.SET, null);
}

public String getKeyFromData(Tuple2<String, Integer> data) {
    return data.f0;
}

public String getValueFromData(Tuple2<String, Integer> data) {
    return data.f1.toString();
}
}
```

我们用最简单的单机Redis的SET命令进行演示。

完整的代码如下，实现一个读取Kafka的消息，然后进行WordCount，并把结果更新到redis中：

```

public class RedisSinkTest {

public static void main(String[] args) throws Exception{

StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
env.enableCheckpointing(2000);
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

//连接kafka
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "127.0.0.1:9092");

FlinkKafkaConsumer<String> consumer = new FlinkKafkaConsumer<>("test", new SimpleStringSchema(), properties);
consumer.setStartFromEarliest();
DataStream<String> stream = env.addSource(consumer);
DataStream<Tuple2<String, Integer>> counts = stream.flatMap(new LineSplitter()).keyBy(0).sum(1);

//实例化FlinkJedisPoolConfig 配置redis
FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("127.0.0.1").setPort("6379").build();
//实例化RedisSink，并通过flink的addSink的方式将flink计算的结果插入到redis

counts.addSink(new RedisSink<>(conf,new RedisSinkExample()));
env.execute("WordCount From Kafka To Redis");

}//
public static final class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {

@Override
public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
    String[] tokens = value.toLowerCase().split("\\W+");
    for (String token : tokens) {
        if (token.length() > 0) {
            out.collect(new Tuple2<String, Integer>(token, 1));
        }
    }
}
}
//指定Redis set
public static final class RedisSinkExample implements RedisMapper<Tuple2<String,Integer>> {
public RedisCommandDescription getCommandDescription() {
    return new RedisCommandDescription(RedisCommand.SET, null);
}

public String getKeyFromData(Tuple2<String, Integer> data) {
    return data.f0;
}

public String getValueFromData(Tuple2<String, Integer> data) {
    return data.f1.toString();
}
}

}//

```
预告，后续更新写入Hbase和Mysql案例代码。
