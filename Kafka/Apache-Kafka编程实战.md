**Apache Kafka 编程实战您可能感性的文章:**

[Apache-Kafka简介](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000482%26idx%3D1%26sn%3D22b13749ed0352cd286eac7697f39f23%26chksm%3D7d3d44774a4acd6189d082976e90087a9a955e6ca12b21193395536643a302ac4c13c88fe212%23rd)

[Apache Kafka安装和使用](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000470%26idx%3D1%26sn%3D41ee111a073c51af4f9e87c2cdc4d584%26chksm%3D7d3d44434a4acd55b67414765a7b79152d7ef430ba00bec8af6cdddd8e8cf161777ee4a15841%23rd)

[Apache-Kafka核心概念](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000472%26idx%3D1%26sn%3D99353b901d1174c3edd4a9ebbe394975%26chksm%3D7d3d444d4a4acd5bf0017210f55ec394abda01d163674d540988ca94863a51411be951711553%23rd)


[Apache-Kafka核心组件和流程-协调器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000476%26idx%3D1%26sn%3D34b2127b1a09664087e3b2079844c2db%26chksm%3D7d3d44494a4acd5f3bc70d914ae2842409282780d19d57043d168895e55f160b3be7835e2446%23rd)

[Apache-Kafka核心组件和流程(副本管理器)](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000480%26idx%3D1%26sn%3D054cdf620eb82c4ecfaccd226d49d0e0%26chksm%3D7d3d44754a4acd638ca37afcfdaad802bb3dec01758b18cdf2c607ec494526832ee58ff43451%23rd)

[Apache-Kafka 核心组件和流程-控制器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000474%26idx%3D1%26sn%3Dc9b9d8fbb942f5299eb1d23a9363c0a4%26chksm%3D7d3d444f4a4acd597607e33ee59aad92db50084a5ab7edb84449df6f2f3ecc504e97f05977bb%23rd)

[Apache-Kafka核心组件和流程-日志管理器](http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzU3MzgwNTU2Mg%3D%3D%26mid%3D100000478%26idx%3D1%26sn%3Deeb3310214d7fa24ca86c4afad421baa%26chksm%3D7d3d444b4a4acd5d1987dc78f89d40a20833cec682b30b9f1a0735a26681f681a38853a6ff63%23rd)

....

本章通过实际例子，讲解了如何使用java进行kafka开发。

添加依赖：

```
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>2.0.0</version>
</dependency>

```

下面是创建主题的代码：

```
public class TopicProcessor {
private static final String ZK_CONNECT="localhost:2181";
private static final int SESSION_TIME_OUT=30000;
private static final int CONNECT_OUT=30000;

public static void createTopic(String topicName,int partitionNumber,int replicaNumber,Properties properties){
ZkUtils zkUtils = null;
try{
zkUtils=ZkUtils.apply(ZK_CONNECT,SESSION_TIME_OUT,CONNECT_OUT, JaasUtils.isZkSecurityEnabled());
if(!AdminUtils.topicExists(zkUtils,topicName)){
AdminUtils.createTopic(zkUtils,topicName,partitionNumber,replicaNumber,properties,AdminUtils.createTopic$default$6());
}
}catch (Exception e){
e.printStackTrace();
}finally {
zkUtils.close();
}
}

public static void main(String[] args){
createTopic("javatopic",1,1,new Properties());
}
}

```

首先定义了zookeeper相关连接信息。然后在createTopic中，先初始化ZkUtils，和zookeeper交互依赖于它。然后通过AdminUtils先判断是否存在你要创建的主题，如果不存在，则通过createTopic方法进行创建。传入参数包括主题名称，分区数量，副本数量等。

## **生产者生产消息**

生产者生产消息代码如下：

```
public class MessageProducer {
private static final String TOPIC="education-info";
private static final String BROKER_LIST="localhost:9092";
private static KafkaProducer<String,String> producer = null;

static{
Properties configs = initConfig();
producer = new KafkaProducer<String, String>(configs);
}

private static Properties initConfig(){
Properties properties = new Properties();
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,BROKER_LIST);
properties.put(ProducerConfig.ACKS_CONFIG,"all");
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
return properties;
}

public static void main(String[] args){
try{
String message = "hello world";
ProducerRecord<String,String> record = new ProducerRecord<String,String>(TOPIC,message);
producer.send(record, new Callback() {
@Override
public void onCompletion(RecordMetadata metadata, Exception exception) {
if(null==exception){
System.out.println("perfect!");
}
if(null!=metadata){
System.out.print("offset:"+metadata.offset()+";partition:"+metadata.partition());
}
}
}).get();
}catch (Exception e){
e.printStackTrace();
}finally {
producer.close();
}
}
}

```

1、首先初始化KafkaProducer对象。

```
producer = new KafkaProducer<String, String>(configs);

```

2、创建要发送的消息对象。

```
ProducerRecord<String,String> record = new ProducerRecord<String,String>(TOPIC,message);

```

3、通过producer的send方法，发送消息

4、发送消息时，可以通过回调函数，取得消息发送的结果。异常发生时，对异常进行处理。

初始化producer时候,需要注意下面属性设置：

```
properties.put(ProducerConfig.ACKS_CONFIG,"all");

```

这里有三种值可供选择：

*   0，不等服务器响应，直接返回发送成功。速度最快，但是丢了消息是无法知道的
*   1，leader副本收到消息后返回成功
*   all，所有参与的副本都复制完成后返回成功。这样最安全，但是延迟最高。

## 消费者消费消息

我们直接看代码

```
public class MessageConsumer {

private static final String TOPIC="education-info";
private static final String BROKER_LIST="localhost:9092";
private static KafkaConsumer<String,String> kafkaConsumer = null;

static {
Properties properties = initConfig();
kafkaConsumer = new KafkaConsumer<String, String>(properties);
kafkaConsumer.subscribe(Arrays.asList(TOPIC));
}

private static Properties initConfig(){
Properties properties = new Properties();
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,BROKER_LIST);
properties.put(ConsumerConfig.GROUP_ID_CONFIG,"test");
properties.put(ConsumerConfig.CLIENT_ID_CONFIG,"test");
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class.getName());
return properties;
}

public static void main(String[] args){
try{
while(true){
ConsumerRecords<String,String> records = kafkaConsumer.poll(100);
for(ConsumerRecord record:records){
try{
System.out.println(record.value());
}catch(Exception e){
e.printStackTrace();
}
}
}

}catch(Exception e){
e.printStackTrace();
}finally {
kafkaConsumer.close();
}
}
}

```

代码逻辑如下：

1、初始化消费者KafkaConsumer，并订阅主题。

```
kafkaConsumer = new KafkaConsumer<String, String>(properties);
kafkaConsumer.subscribe(Arrays.asList(TOPIC));

```

2、循环拉取消息

```
ConsumerRecords<String,String> records = kafkaConsumer.poll(100);

```

poll方法传入的参数100，是等待broker返回数据的时间，如果超过100ms没有响应，则不再等待。

3、拉取回消息后，循环处理。

```
for(ConsumerRecord record:records){
try{
System.out.println(record.value());
}catch(Exception e){
e.printStackTrace();
}
}

```

消费相关代码比较简单，不过这个版本没有处理偏移量提交。学习过第四章-协调器相关的同学应该还记得偏移量提交的问题。我曾说过最佳实践是同步和异步提交相结合，同时在特定的时间点，比如再均衡前进行手动提交。

加入偏移量提交，需要做如下修改：

1、enable.auto.commit设置为false

2、消费代码如下：

```
public static void main(String[] args){
try{
while(true){
ConsumerRecords<String,String> records =
kafkaConsumer.poll(100);
for(ConsumerRecord record:records){
try{
System.out.println(record.value());
}catch(Exception e){
e.printStackTrace();
}
}
kafkaConsumer.commitAsync();
}

}catch(Exception e){
e.printStackTrace();
}finally {
try{
kafkaConsumer.commitSync();
}finally {
kafkaConsumer.close();
}
}
}

```

3、订阅消息时，实现再均衡的回调方法，在此方法中手动提交偏移量

```
kafkaConsumer.subscribe(Arrays.asList(TOPIC), new ConsumerRebalanceListener() {
@Override
public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
//再均衡之前和消费者停止读取消息之后调用
kafkaConsumer.commitSync(currentOffsets);
}
});

```

通过以上三步，我们把自动提交偏移量改为了手动提交。正常消费时，异步提交kafkaConsumer.commitAsync()。即使偶尔失败，也会被后续成功的提交覆盖掉。而在发生异常的时候，手动提交 kafkaConsumer.commitSync()。此外在步骤3中，我们通过实现再均衡时的回调方法，手动同步提交偏移量，确保了再均衡前偏移量提交成功。

以上面的最佳实践提交偏移量，既能保证消费时较高的效率，又能够尽量避免重复消费。不过由于重复消费无法100%避免，消费逻辑需要自己处理重复消费的判断。

**你真的不关注一下嘛~**

![image](http://upload-images.jianshu.io/upload_images/16241060-0a3239c0e954c793.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
