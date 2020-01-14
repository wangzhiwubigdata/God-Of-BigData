
## 分布式缓存

Flink提供了一个分布式缓存，类似于hadoop，可以使用户在并行函数中很方便的读取本地文件，并把它放在taskmanager节点中，防止task重复拉取。
此缓存的工作机制如下：程序注册一个文件或者目录(本地或者远程文件系统，例如hdfs或者s3)，通过ExecutionEnvironment注册缓存文件并为它起一个名称。
当程序执行，Flink自动将文件或者目录复制到所有taskmanager节点的本地文件系统，仅会执行一次。用户可以通过这个指定的名称查找文件或者目录，然后从taskmanager节点的本地文件系统访问它。


## 示例

在ExecutionEnvironment中注册一个文件：

```
//获取运行环境
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();


//1：注册一个文件,可以使用hdfs上的文件 也可以是本地文件进行测试
env.registerCachedFile("/Users/wangzhiwu/WorkSpace/quickstart/text","a.txt");

```

在用户函数中访问缓存文件或者目录(这里是一个map函数)。这个函数必须继承RichFunction,因为它需要使用RuntimeContext读取数据:

```
DataSet<String> result = data.map(new RichMapFunction<String, String>() {
            private ArrayList<String> dataList = new ArrayList<String>();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //2：使用文件
                File myFile = getRuntimeContext().getDistributedCache().getFile("a.txt");
                List<String> lines = FileUtils.readLines(myFile);
                for (String line : lines) {
                    this.dataList.add(line);
                    System.err.println("分布式缓存为:" + line);
                }
            }

            @Override
            public String map(String value) throws Exception {
                //在这里就可以使用dataList
                System.err.println("使用datalist：" + dataList + "------------" +value);
                //业务逻辑
                return dataList +"：" +  value;
            }
        });

        result.printToErr();
    }
```

完整代码如下,仔细看注释：

```

public class DisCacheTest {

    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //1：注册一个文件,可以使用hdfs上的文件 也可以是本地文件进行测试
      //text 中有4个单词:hello flink hello FLINK env.registerCachedFile("/Users/wangzhiwu/WorkSpace/quickstart/text","a.txt");

        DataSource<String> data = env.fromElements("a", "b", "c", "d");

        DataSet<String> result = data.map(new RichMapFunction<String, String>() {
            private ArrayList<String> dataList = new ArrayList<String>();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //2：使用文件
                File myFile = getRuntimeContext().getDistributedCache().getFile("a.txt");
                List<String> lines = FileUtils.readLines(myFile);
                for (String line : lines) {
                    this.dataList.add(line);
                    System.err.println("分布式缓存为:" + line);
                }
            }

            @Override
            public String map(String value) throws Exception {
                //在这里就可以使用dataList
                System.err.println("使用datalist：" + dataList + "------------" +value);
                //业务逻辑
                return dataList +"：" +  value;
            }
        });

        result.printToErr();
    }
}//

```

输出结果如下：
```
[hello, flink, hello, FLINK]：a
[hello, flink, hello, FLINK]：b
[hello, flink, hello, FLINK]：c
[hello, flink, hello, FLINK]：d
```
