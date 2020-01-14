## 广播变量简介

在Flink中，同一个算子可能存在若干个不同的并行实例，计算过程可能不在同一个Slot中进行，不同算子之间更是如此，因此不同算子的计算数据之间不能像Java数组之间一样互相访问，而广播变量Broadcast便是解决这种情况的。

我们可以把广播变量理解为是一个公共的共享变量，我们可以把一个dataset 数据集广播出去，然后不同的task在节点上都能够获取到，这个数据在每个节点上只会存在一份



## 用法

```
1：初始化数据
  DataSet<Integer> num = env.fromElements(1, 2, 3)
  2：广播数据
  .withBroadcastSet(toBroadcast, "num");
  3：获取数据
  Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("num");
  
  注意：
  1：广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大。因为广播出去的数据，会常驻内存，除非程序执行结束
  2：广播变量在初始化广播出去以后不支持修改，这样才能保证每个节点的数据都是一致的。

```

## 注意事项

### 使用广播状态，task 之间不会相互通信

只有广播的一边可以修改广播状态的内容。用户必须保证所有 operator 并发实例上对广播状态的     修改行为都是一致的。或者说，如果不同的并发实例拥有不同的广播状态内容，将导致不一致的结果。
### 广播状态中事件的顺序在各个并发实例中可能不尽相同

广播流的元素保证了将所有元素（最终）都发给下游所有的并发实例，但是元素的到达的顺序可能在并发实例之间并不相同。因此，对广播状态的修改不能依赖于输入数据的顺序。

### 所有operator task都会快照下他们的广播状态
在checkpoint时，所有的 task 都会 checkpoint 下他们的广播状态，随着并发度的增加，checkpoint 的大小也会随之增加
### 广播变量存在内存中

广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大，百兆左右可以接受，Gb不能接受


## 案例

```
public class BroadCastTest {

    public static void main(String[] args) throws Exception{
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //1.封装一个DataSet
        DataSet<Integer> broadcast = env.fromElements(1, 2, 3);
        DataSet<String> data = env.fromElements("a", "b");
        data.map(new RichMapFunction<String, String>() {
            private List list = new ArrayList();
            @Override
            public void open(Configuration parameters) throws Exception {
                // 3. 获取广播的DataSet数据 作为一个Collection
                Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("number");
                list.addAll(broadcastSet);
            }

            @Override
            public String map(String value) throws Exception {
                return value + ": "+ list;
            }
        }).withBroadcastSet(broadcast, "number") 
            // 2. 广播的broadcast
          .printToErr();//打印到err方便查看
    }
}
```

输出结果：

```
a: [1, 2, 3]
b: [1, 2, 3]
```