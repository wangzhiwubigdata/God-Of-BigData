## 简介

Apache Flink具有两个关系API - 表API和SQL - 用于统一流和批处理。Table API是Scala和Java的语言集成查询API，允许以非常直观的方式组合来自关系运算符的查询，Table API和SQL接口彼此紧密集成，以及Flink的DataStream和DataSet API。您可以轻松地在基于API构建的所有API和库之间切换。例如，您可以使用CEP库从DataStream中提取模式，然后使用Table API分析模式，或者可以在预处理上运行Gelly图算法之前使用SQL查询扫描，过滤和聚合批处理表数据。

## Flink SQL的编程模型


### 创建一个TableEnvironment
TableEnvironment是Table API和SQL集成的核心概念，它主要负责:
　　1、在内部目录中注册一个Table
　　2、注册一个外部目录
　　3、执行SQL查询
　　4、注册一个用户自定义函数(标量、表及聚合)
　　5、将DataStream或者DataSet转换成Table
　　6、持有ExecutionEnvironment或者StreamExecutionEnvironment的引用
一个Table总是会绑定到一个指定的TableEnvironment中，相同的查询不同的TableEnvironment是无法通过join、union合并在一起。
TableEnvironment有一个在内部通过表名组织起来的表目录，Table API或者SQL查询可以访问注册在目录中的表，并通过名称来引用它们。

### 在目录中注册表
TableEnvironment允许通过各种源来注册一个表:

　　1、一个已存在的Table对象，通常是Table API或者SQL查询的结果
         Table projTable = tableEnv.scan("X").select(...);

　　2、TableSource，可以访问外部数据如文件、数据库或者消息系统
         TableSource csvSource = new CsvTableSource("/path/to/file", ...);

　　3、DataStream或者DataSet程序中的DataStream或者DataSet
         //将DataSet转换为Table
         Table table= tableEnv.fromDataSet(tableset);

### 注册TableSink	

注册TableSink可用于将 Table API或SQL查询的结果发送到外部存储系统，例如数据库，键值存储，消息队列或文件系统（在不同的编码中，例如，CSV，Apache [Parquet] ，Avro，ORC]，......）:
　　
```
TableSink csvSink = new CsvTableSink("/path/to/file", ...); 
　　
```
```
　　2、 String[] fieldNames = {"a", "b", "c"}; 
                TypeInformation[] fieldTypes = {Types.INT, Types.STRING, Types.LONG}; 
                tableEnv.registerTableSink("CsvSinkTable", fieldNames, fieldTypes, csvSink);
```

## 实战案例一

基于Flink SQL的WordCount:

```
public class WordCountSQL {

    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment tEnv = TableEnvironment.getTableEnvironment(env);

        List list  =  new ArrayList();
        String wordsStr = "Hello Flink Hello TOM";
        String[] words = wordsStr.split("\\W+");
        for(String word : words){
            WC wc = new WC(word, 1);
            list.add(wc);
        }
        DataSet<WC> input = env.fromCollection(list);
        tEnv.registerDataSet("WordCount", input, "word, frequency");
        Table table = tEnv.sqlQuery(
                "SELECT word, SUM(frequency) as frequency FROM WordCount GROUP BY word");
        DataSet<WC> result = tEnv.toDataSet(table, WC.class);
        result.print();
    }//main

    public static class WC {
        public String word;//hello
        public long frequency;//1

        // public constructor to make it a Flink POJO
        public WC() {}

        public WC(String word, long frequency) {
            this.word = word;
            this.frequency = frequency;
        }

        @Override
        public String toString() {
            return "WC " + word + " " + frequency;
        }
    }

}
```
输出如下：

```
WC TOM 1
WC Hello 2
WC Flink 1
```

## 实战案例二

本例稍微复杂，首先读取一个文件中的内容进行统计，并写入到另外一个文件中：

```
public class SQLTest {

	public static void main(String[] args) throws Exception{

		ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
		BatchTableEnvironment tableEnv = BatchTableEnvironment.getTableEnvironment(env);
		env.setParallelism(1);

		DataSource<String> input = env.readTextFile("test.txt");
		input.print();
		//转换成dataset
		DataSet<Orders> topInput = input.map(new MapFunction<String, Orders>() {
			@Override
			public Orders map(String s) throws Exception {
				String[] splits = s.split(" ");
				return new Orders(Integer.valueOf(splits[0]), String.valueOf(splits[1]),String.valueOf(splits[2]), Double.valueOf(splits[3]));
			}
		});
		//将DataSet转换为Table
		Table order = tableEnv.fromDataSet(topInput);
		//orders表名
		tableEnv.registerTable("Orders",order);

		Table tapiResult = tableEnv.scan("Orders").select("name");
		tapiResult.printSchema();

		Table sqlQuery = tableEnv.sqlQuery("select name, sum(price) as total from Orders group by name order by total desc");

		//转换回dataset
		DataSet<Result> result = tableEnv.toDataSet(sqlQuery, Result.class);

		//将dataset map成tuple输出
		/*result.map(new MapFunction<Result, Tuple2<String,Double>>() {
			@Override
			public Tuple2<String, Double> map(Result result) throws Exception {
				String name = result.name;
				Double total = result.total;
				return Tuple2.of(name,total);
			}
		}).print();*/


		TableSink sink = new CsvTableSink("SQLTEST.txt", "|");
		//writeToSink

		/*sqlQuery.writeToSink(sink);
		env.execute();*/

		String[] fieldNames = {"name", "total"};
		TypeInformation[] fieldTypes = {Types.STRING, Types.DOUBLE};
		tableEnv.registerTableSink("SQLTEST", fieldNames, fieldTypes, sink);
		sqlQuery.insertInto("SQLTEST");
		env.execute();
	}

	/**
	 * 源数据的映射类
	 */
	public static class Orders {
		/**
		 * 序号，姓名，书名，价格
		 */
		public Integer id;
		public String name;
		public String book;
		public Double price;

		public Orders() {
			super();
		}
		public Orders(Integer id, String name, String book, Double price) {
			this.id = id;
			this.name = name;
			this.book = book;
			this.price = price;
		}
	}
	/**
	 * 统计结果对应的类
	 */
	public static class Result {
		public String name;
		public Double total;

		public Result() {}
	}
	}//
```

以上所有代码，大家在公众号回复`Flink`即可下载，可以直接本地运行，方便大家调试