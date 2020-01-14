
## 需求
某个图书网站，希望看到双十一秒杀期间实时的热销排行榜单。我们可以将“实时热门商品”翻译成程序员更好理解的需求:每隔5秒钟输出最近一小时内点击量最多的前 N 个商品/图书.


## 需求分解

将这个需求进行分解我们大概要做这么几件事情：

* 告诉 Flink 框架基于时间做窗口，我们这里用processingTime，不用自带时间戳
* 过滤出图书点击行为数据
* 按一小时的窗口大小，每5秒钟统计一次，做滑动窗口聚合（Sliding Window）
* 聚合，输出窗口中点击量前N名的商品



## 代码实现

### 向Kafka发消息模拟购买事件

```
public class KafkaProducer {


    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> text = env.addSource(new MyNoParalleSource()).setParallelism(1);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "127.0.0.1:9092");
        //new FlinkKafkaProducer("topn",new KeyedSerializationSchemaWrapper(new SimpleStringSchema()),properties,FlinkKafkaProducer.Semantic.EXACTLY_ONCE);
	    FlinkKafkaProducer<String> producer = new FlinkKafkaProducer("topn",new SimpleStringSchema(),properties);
/*
        //event-timestamp事件的发生时间
        producer.setWriteTimestampToKafka(true);
*/
        text.addSink(producer);
        env.execute();
    }
}//
```
其中的：`MyNoParalleSource` 是作者自己实现的一个并行度为1的发送器，用来向kafka发送数据：

```
public class MyNoParalleSource implements SourceFunction<String> {//1

    //private long count = 1L;
    private boolean isRunning = true;

    /**
     * 主要的方法
     * 启动一个source
     * 大部分情况下，都需要在这个run方法中实现一个循环，这样就可以循环产生数据了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void run(SourceContext<String> ctx) throws Exception {
        while(isRunning){
            //图书的排行榜
            List<String> books = new ArrayList<>();
            books.add("Pyhton从入门到放弃");//10
            books.add("Java从入门到放弃");//8
            books.add("Php从入门到放弃");//5
            books.add("C++从入门到放弃");//3
            books.add("Scala从入门到放弃");//0-4
            int i = new Random().nextInt(5);
            ctx.collect(books.get(i));

            //每1秒产生一条数据
            Thread.sleep(1000);
        }
    }
    //取消一个cancel的时候会调用的方法
    @Override
    public void cancel() {
        isRunning = false;
    }
}

```

可见，我们每过1秒向Kafka的topn这个topic随机发送一本书的名字用来模拟购买行为。

整体实现代码如下：

```
public class TopN {

	public static void main(String[] args) throws Exception{

		/**
		 *
		 *  书1 书2 书3
		 *  （书1,1） (书2，1) （书3,1）
		 *
		 *
		 */
		//每隔5秒钟 计算过去1小时 的 Top 3 商品
		StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

		env.setParallelism(1);

		env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime); //以processtime作为时间语义


		Properties properties = new Properties();
		properties.setProperty("bootstrap.servers", "127.0.0.1:9092");
		FlinkKafkaConsumer<String> input = new FlinkKafkaConsumer<>("topn", new SimpleStringSchema(), properties);

		//从最早开始消费 位点
		input.setStartFromEarliest();


		DataStream<String> stream = env
				.addSource(input);

		DataStream<Tuple2<String, Integer>> ds = stream
				.flatMap(new LineSplitter()); //将输入语句split成一个一个单词并初始化count值为1的Tuple2<String, Integer>类型


		DataStream<Tuple2<String, Integer>> wcount = ds
				.keyBy(0)
				.window(SlidingProcessingTimeWindows.of(Time.seconds(600),Time.seconds(5)))
				//key之后的元素进入一个总时间长度为600s,每5s向后滑动一次的滑动窗口
				.sum(1);// 将相同的key的元素第二个count值相加

		wcount
				.windowAll(TumblingProcessingTimeWindows.of(Time.seconds(5)))//(shu1, xx) (shu2,xx)....
				//所有key元素进入一个5s长的窗口（选5秒是因为上游窗口每5s计算一轮数据，topN窗口一次计算只统计一个窗口时间内的变化）
				.process(new TopNAllFunction(3))
				.print();
//redis sink  redis -> 接口

		env.execute();
	}//





	private static final class LineSplitter implements
			FlatMapFunction<String, Tuple2<String, Integer>> {

		public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
			// normalize and split the line
			//String[] tokens = value.toLowerCase().split("\\W+");

			// emit the pairs
			/*for (String token : tokens) {
				if (token.length() > 0) {
					out.collect(new Tuple2<String, Integer>(token, 1));
				}
			}*/

			//（书1,1） (书2，1) （书3,1）
			out.collect(new Tuple2<String, Integer>(value, 1));
		}
	}

	private static class TopNAllFunction
			extends
			ProcessAllWindowFunction<Tuple2<String, Integer>, String, TimeWindow> {

		private int topSize = 3;

		public TopNAllFunction(int topSize) {

			this.topSize = topSize;
		}

		public void process(

				ProcessAllWindowFunction<Tuple2<String, Integer>, String, TimeWindow>.Context arg0,
				Iterable<Tuple2<String, Integer>> input,
				Collector<String> out) throws Exception {

			TreeMap<Integer, Tuple2<String, Integer>> treemap = new TreeMap<Integer, Tuple2<String, Integer>>(
					new Comparator<Integer>() {

						@Override
						public int compare(Integer y, Integer x) {
							return (x < y) ? -1 : 1;
						}

					}); //treemap按照key降序排列，相同count值不覆盖

			for (Tuple2<String, Integer> element : input) {
				treemap.put(element.f1, element);
				if (treemap.size() > topSize) { //只保留前面TopN个元素
					treemap.pollLastEntry();
				}
			}


			for (Map.Entry<Integer, Tuple2<String, Integer>> entry : treemap
					.entrySet()) {
				out.collect("=================\n热销图书列表:\n"+ new Timestamp(System.currentTimeMillis()) +  treemap.toString() + "\n===============\n");
			}

		}

	}


}//
```

查看输出：
```
=================
热销图书列表:
2019-03-05 22:32:40.004{8=(Java从入门到放弃,8), 7=(C++从入门到放弃,7), 5=(Php从入门到放弃,5)}
===============
=================
热销图书列表:
2019-03-05 22:32:45.004{8=(Java从入门到放弃,8), 7=(C++从入门到放弃,7), 5=(Php从入门到放弃,5)}
===============

```