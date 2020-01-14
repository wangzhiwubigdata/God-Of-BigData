
## 概述

* Flink支持不同的重启策略，以在故障发生时控制作业如何重启
* 集群在启动时会伴随一个默认的重启策略，在没有定义具体重启策略时会使用该默认策略。 
* 如果在工作提交时指定了一个重启策略，该策略会覆盖集群的默认策略默认的重启策略可以通过 Flink 的配置文件 flink-conf.yaml 指定。配置参数 restart-strategy 定义了哪个策略被使用。
* 常用的重启：

    1.策略固定间隔 (Fixed delay)
    2.失败率 (Failure rate)
    3.无重启 (No restart)

* 如果没有启用 checkpointing，则使用无重启 (no restart) 策略。如果启用了 checkpointing，但没有配置重启策略，则使用固定间隔 (fixed-delay) 策略
* 重启策略可以在flink-conf.yaml中配置，表示全局的配置。也可以在应用代码中动态指定，会覆盖全局配置


## 固定间隔

第一种：全局配置 flink-conf.yaml
```
	restart-strategy: fixed-delay 
	restart-strategy.fixed-delay.attempts: 3 
	restart-strategy.fixed-delay.delay: 10 s
```
第二种：应用代码设置：
	
    ```
    env.setRestartStrategy(RestartStrategies.fixedDelayRestart( 3,// 尝试重启的次数 
        Time.of(10, TimeUnit.SECONDS) // 间隔 ));
    ```

## 失败率

* 失败率重启策略在Job失败后会重启，但是超过失败率后，Job会最终被认定失败。在两个连续的重启尝试之间，重启策略会等待一个固定的时间

**下面配置是5分钟内若失败了3次则认为该job失败，重试间隔为10s**

第一种：全局配置 flink-conf.yaml
```
    restart-strategy: failure-rate  
	restart-strategy.failure-rate.max-failures-per-interval: 3  
	restart-strategy.failure-rate.failure-rate-interval: 5 min  
	restart-strategy.failure-rate.delay: 10 s
```
    
第二种：应用代码设置

```
   env.setRestartStrategy(RestartStrategies.failureRateRestart(  3,//一个时间段内的最大失败次数  
Time.of(5, TimeUnit.MINUTES), // 衡量失败次数的是时间段  Time.of(10, TimeUnit.SECONDS) // 间隔  ));
```

## 无重启策略

第一种：全局配置 flink-conf.yaml

```
	restart-strategy: none
```

第二种：应用代码设置
```
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment(); 	env.setRestartStrategy(RestartStrategies.noRestart());

```


## 实际代码演示

```
public class RestartTest {

    public static void main(String[] args) {
        //获取flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 每隔1000 ms进行启动一个检查点【设置checkpoint的周期】
        env.enableCheckpointing(1000);

        // 间隔10秒 重启3次
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3,Time.seconds(10)));

        //5分钟内若失败了3次则认为该job失败，重试间隔为10s
        env.setRestartStrategy(RestartStrategies.failureRateRestart(3,Time.of(5,TimeUnit.MINUTES),Time.of(10,TimeUnit.SECONDS)));

        //不重试
        env.setRestartStrategy(RestartStrategies.noRestart());
    }//

}
```

