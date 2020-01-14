

本文介绍消费Kafka的消息实时写入Mysql

1. maven新增依赖：

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.39</version>
</dependency>

```


2.重写RichSinkFunction,实现一个Mysql Sink

```
public class MysqlSink extends
    RichSinkFunction<Tuple3<Integer, String, Integer>> {
private Connection connection;
private PreparedStatement preparedStatement;
String username = "";
String password = "";
String drivername = "";   //配置改成自己的配置
String dburl = "";

@Override
public void invoke(Tuple3<Integer, String, Integer> value) throws Exception {
    Class.forName(drivername);
    connection = DriverManager.getConnection(dburl, username, password);
    String sql = "replace into table(id,num,price) values(?,?,?)"; //假设mysql 有3列 id,num,price
    preparedStatement = connection.prepareStatement(sql);
    preparedStatement.setInt(1, value.f0);
    preparedStatement.setString(2, value.f1);
    preparedStatement.setInt(3, value.f2);
    preparedStatement.executeUpdate();
    if (preparedStatement != null) {
        preparedStatement.close();
    }
    if (connection != null) {
        connection.close();
    }
}
}
```

3. Flink主类

```
public class MysqlSinkTest {

public static void main(String[] args) throws Exception {
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");

// 1,abc,100  类似这样的数据，当然也可以是很复杂的json数据，去做解析
FlinkKafkaConsumer<String> consumer = new FlinkKafkaConsumer<>("test", new SimpleStringSchema(), properties);
env.getConfig().disableSysoutLogging();  //设置此可以屏蔽掉日记打印情况
env.getConfig().setRestartStrategy(
        RestartStrategies.fixedDelayRestart(5, 5000));
env.enableCheckpointing(2000);
DataStream<String> stream = env
        .addSource(consumer);

DataStream<Tuple3<Integer, String, Integer>> sourceStream = stream.filter((FilterFunction<String>) value -> StringUtils.isNotBlank(value))
                                                                        .map((MapFunction<String, Tuple3<Integer, String, Integer>>) value -> {
    String[] args1 = value.split(",");
    return new Tuple3<Integer, String, Integer>(Integer
            .valueOf(args1[0]), args1[1],Integer
            .valueOf(args1[2]));
});

sourceStream.addSink(new MysqlSink());
env.execute("data to mysql start");
}
}

```
