![ac7b60f2d0c6bba23165c6e218902a41](Apache-Flink漫谈系列(1)-概述.resources/49D66A77-779B-468F-9BCB-6846609484DA.png)

摘要：Apache Flink 的命脉 "命脉" 即生命与血脉，常喻极为重要的事物。系列的首篇，首篇的首段不聊Apache Flink的历史，不聊Apache Flink的架构，不聊Apache Flink的功能特性，我们用一句话聊聊什么是 Apache Flink 的命脉？我的答案是：Apache Flink 是以"批是流的特例"的认知进行系统设计的。         

                     

"命脉" 即生命与血脉，常喻极为重要的事物。系列的首篇，首篇的首段不聊Apache Flink的历史，不聊Apache Flink的架构，不聊Apache Flink的功能特性，我们用一句话聊聊什么是 Apache Flink 的命脉？我的答案是：Apache Flink 是以"批是流的特例"的认知进行系统设计的。




我们经常听说 "天下武功，唯快不破"，大概意思是说 "任何一种武功的招数都是有拆招的，唯有速度快，快到对手根本来不及反应，你就将对手KO了，对手没有机会拆招，所以唯快不破"。 那么这与Apache Flink有什么关系呢？Apache Flink是Native Streaming(纯流式)计算引擎，在实时计算场景最关心的就是"快",也就是 "低延时"。



就目前最热的两种流计算引擎Apache Spark和Apache Flink而言，谁最终会成为No1呢？单从 "低延时" 的角度看，Spark是Micro Batching(微批式)模式，最低延迟Spark能达到0.5~2秒左右，Flink是Native Streaming(纯流式)模式，最低延时能达到微秒。很显然是相对较晚出道的 Apache Flink 后来者居上。 那么为什么Apache Flink能做到如此之 "快"呢？根本原因是Apache Flink 设计之初就认为 "批是流的特例"，整个系统是Native Streaming设计，每来一条数据都能够触发计算。相对于需要靠时间来积攒数据Micro Batching模式来说，在架构上就已经占据了绝对优势。



那么为什么关于流计算会有两种计算模式呢？归其根本是因为对流计算的认知不同，是"流是批的特例" 和 "批是流的特例" 两种不同认知产物。



Micro Batching 模式


Micro-Batching 计算模式认为 "流是批的特例"， 流计算就是将连续不断的批进行持续计算，如果批足够小那么就有足够小的延时，在一定程度上满足了99%的实时计算场景。那么那1%为啥做不到呢？这就是架构的魅力，在Micro-Batching模式的架构实现上就有一个自然流数据流入系统进行攒批的过程，这在一定程度上就增加了延时。具体如下示意图：

![cbac823e35fa901338428e1b2c490bf9](Apache-Flink漫谈系列(1)-概述.resources/3BF00033-D856-49C2-A301-E6DB65B22EAE.png)

很显然Micro-Batching模式有其天生的低延时瓶颈，但任何事物的存在都有两面性，在大数据计算的发展历史上，最初Hadoop上的MapReduce就是优秀的批模式计算框架，Micro-Batching在设计和实现上可以借鉴很多成熟实践。



Native Streaming 模式


Native Streaming 计算模式认为 ""批是流的特", 这个认知更贴切流的概念，比如一些监控类的消息流，数据库操作的binlog，实时的支付交易信息等等自然流数据都是一条，一条的流入。Native Streaming 计算模式每条数据的到来都进行计算，这种计算模式显得更自然，并且延时性能达到更低。具体如下示意图：

![73b0b324304bfc075c8025bd5d09848f](Apache-Flink漫谈系列(1)-概述.resources/68B36353-346D-4E32-9AD8-8AE91F3FE461.png)

很明显Native Streaming模式占据了流计算领域 "低延时" 的核心竞争力，当然Native Streaming模式的实现框架是一个历史先河，第一个实现
Native Streaming模式的流计算框架是第一个吃螃蟹的人，需要面临更多的挑战，后续章节我们会慢慢介绍。当然Native Streaming模式的框架实现上面很容易实现Micro-Batching和Batching模式的计算，Apache Flink就是Native Streaming计算模式的流批统一的计算引擎。



Apache Flink 按不同的需求支持Local，Cluster，Cloud三种部署模式，同时Apache Flink在部署上能够与其他成熟的生态产品进行完美集成，如 Cluster模式下可以利用YARN(Yet Another Resource Negotiator）/Mesos集成进行资源管理，在Cloud部署模式下可以与GCE(Google Compute Engine), EC2(Elastic Compute Cloud)进行集成。



Local 模式


该模式下Apache Flink 整体运行在Single JVM中，在开发学习中使用，同时也可以安装到很多端类设备上。参考



Cluster模式


该模式是典型的投产的集群模式，Apache Flink 既可以Standalone的方式进行部署，也可以与其他资源管理系统进行集成部署，比如与YARN进行集成。Standalone Cluster 参考 YARN Cluster 参考
这种部署模式是典型的Master/Slave模式，我们以Standalone Cluster模式为例示意如下：

![2d187cc7509ddf0c8c3f937821e708ea](Apache-Flink漫谈系列(1)-概述.resources/79ACC2B0-BF5E-4DC4-99DE-9C4DF6007A3F.png)

其中JM(JobManager)是Master，TM(TaskManager)是Slave，这种Master/Slave模式有一个典型的问题就是SPOF(single point of failure), SPOF如何解决呢？Apache Flink 又提供了HA(High Availability)方案，也就是提供多个Master，在任何时候总有一个JM服役，N(N>=1)个JM候选,进而解决SPOF问题，示意如下：


![b4efa7e407cd155208b16ab325ac06bf](Apache-Flink漫谈系列(1)-概述.resources/5F99E16C-1172-4E31-989B-DA7C0800D476.png)

在实际的生产环境我们都会配置HA方案，目前Alibaba内部使用的也是基于YARN Cluster的HA方案。



Cloud 模式


该模式主要是与成熟的云产品进行集成，Apache Flink官网介绍了Google的GCE 参考，Amazon的EC2 参考，在Alibaba我们也可以将Apache Flink部署到Alibaba的ECS(Elastic Compute Service)。



什么是容错


容错(Fault Tolerance) 是指容忍故障，在故障发生时能够自动检测出来并使系统能够自动回复正常运行。当出现某些指定的网络故障、硬件故障、软件错误时，系统仍能执行规定的一组程序，或者说程序不会因系统中的故障而中止，并且执行结果也不会因系统故障而引起计算差错。



容错的处理模式


在一个分布式系统中由于单个进程或者节点宕机都有可能导致整个Job失败，那么容错机制除了要保证在遇到非预期情况系统能够"运行"外，还要求能"正确运行",也就是数据能按预期的处理方式进行处理，保证计算结果的正确性。计算结果的正确性取决于系统对每一条计算数据处理机制，一般有如下三种处理机制：



At Most Once：最多消费一次，这种处理机制会存在数据丢失的可能。

At Least Once：最少消费一次，这种处理机制数据不会丢失，但是有可能重复消费。

Exactly Once：精确一次，无论何种情况下，数据都只会消费一次，这种机制是对数据准确性的最高要求，在金融支付，银行账务等领域必须采用这种模式。



Apache Flink的容错机制


Apache Flink的Job会涉及到3个部分，外部数据源(External Input), Flink内部数据处理(Flink Data Flow)和外部输出(External Output)。如下示意图:

![7614e8dee7b01008061f1d3622a4d18f](Apache-Flink漫谈系列(1)-概述.resources/C810C9BC-45F5-4ADA-8408-7FF57C0C31DB.png)

目前Apache Flink 支持两种数据容错机制：



* At Least Once
* Exactly Once



其中 Exactly Once 是最严格的容错机制，该模式要求每条数据必须处理且仅处理一次。那么对于这种严格容错机制，一个完整的Flink Job容错要做到 End-to-End 的 容错必须结合三个部分进行联合处理，根据上图我们考虑三个场景：



系统内部容错


Apache Flink利用Checkpointing机制来处理容错，Checkpointing的理论基础 Stephan 在 Lightweight Asynchronous Snapshots for Distributed Dataflows 进行了细节描述，该机制源于有K. MANI CHANDY和LESLIE LAMPORT 发表的 Determining-Global-States-of-a-Distributed-System Paper。Apache Flink 基于Checkpointing机制对Flink Data Flow实现了At Least Once 和 Exactly Once 两种容错处理模式。



Apache Flink Checkpointing的内部实现会利用 Barriers，StateBackend等后续章节会详细介绍的技术来将数据的处理进行Marker。Apache Flink会利用Barrier将整个流进行标记切分，如下示意图：

![728f489db662f1d46e7050a526d9e19e](Apache-Flink漫谈系列(1)-概述.resources/72919AF2-F467-4306-BE2B-042FA56E4DD4.png)

这样Apache Flink的每个Operator都会记录当前成功处理的Checkpoint，如果发生错误，就会从上一个成功的Checkpoint开始继续处理后续数据。比如 Soruce Operator会将读取外部数据源的Position实时的记录到Checkpoint中，失败时候会从Checkpoint中读取成功的position继续精准的消费数据。每个算子会在Checkpoint中记录自己恢复时候必须的数据，比如流的原始数据和中间计算结果等信息，在恢复的时候从Checkpoint中读取并持续处理流数据。



外部Source容错


Apache Flink 要做到 End-to-End 的 Exactly Once 需要外部Source的支持，比如上面我们说过 Apache Flink的Checkpointing机制会在Source节点记录读取的Position，那就需要外部数据提供读取的Position和支持根据Position进行数据读取。



外部Sink容错


Apache Flink 要做到 End-to-End 的 Exactly Once 相对比较困难，如上场景三所述，当Sink Operator节点宕机，重新恢复时候根据Apache Flink 内部系统容错 exactly once的保证,系统会回滚到上次成功的Checkpoin继续写入，但是上次成功Checkpoint之后当前Checkpoint未完成之前已经把一部分新数据写入到kafka了. Apache Flink自上次成功的Checkpoint继续写入kafka，就造成了kafka再次接收到一份同样的来自Sink Operator的数据,进而破坏了End-to-End 的 Exactly Once 语义(重复写入就变成了At Least Once了)，如果要解决这一问题，Apache Flink 利用Two phase commit(两阶段提交)的方式来进行处理。本质上是Sink Operator 需要感知整体Checkpoint的完成，并在整体Checkpoint完成时候将计算结果写入Kafka。



批与流是两种不同的数据处理模式，如Apache Storm只支持流模式的数据处理，Apache Spark只支持批(Micro Batching)模式的数据处理。那么Apache Flink 是如何做到既支持流处理模式也支持批处理模式呢？



统一的数据传输层


开篇我们就介绍Apache Flink 的 "命脉"是以"批是流的特例"为导向来进行引擎的设计的，系统设计成为 "Native Streaming"的模式进行数据处理。那么Apache FLink将批模式执行的任务看做是流式处理任务的特殊情况，只是在数据上批是有界的(有限数量的元素)。



Apache Flink 在网络传输层面有两种数据传输模式：


* PIPELINED模式 - 即一条数据被处理完成以后，立刻传输到下一个节点进行处理。
* BATCH 模式 - 即一条数据被处理完成后，并不会立刻传输到下一个节点进行处理，而是写入到缓存区，如果缓存写满就持久化到本地硬盘上，最后当所有数据都被处理完成后，才将数据传输到下一个节点进行处理。



对于批任务而言同样可以利用PIPELINED模式，比如我要做count统计，利用PIPELINED模式能拿到更好的执行性能。只有在特殊情况，比如SortMergeJoin，这时候我们需要全局数据排序，才需要BATCH模式。大部分情况流与批可用统一的传输策略，只有特殊情况，才将批看做是流的一个特例继续特殊处理。



统一任务调度层


Apache Flink 在任务调度上流与批共享统一的资源和任务调度机制（后续章节会详细介绍）。



统一的用户API层


Apache Flink 在DataStremAPI和DataSetAPI基础上，为用户提供了流批统一的上层TableAPI和SQL，在语法和语义上流批进行高度统一。(其中DataStremAPI和DataSetAPI对流和批进行了分别抽象，这一点并不优雅，在Alibaba内部对其进行了统一抽象）。



求同存异


Apache Flink 是流批统一的计算引擎，并不意味着流与批的任务都走统一的code path，在对底层的具体算子的实现也是有各自的处理的，在具体功能上面会根据不同的特性区别处理。比如 批没有Checkpoint机制，流上不能做SortMergeJoin。



组件栈


我们上面内容已经介绍了很多Apache Flink的各种组件，下面我们整体概览一下全貌，如下：

![d919b72d93c2dcfd7b6e452b9f3e8a42](Apache-Flink漫谈系列(1)-概述.resources/624AD4C0-351D-42C0-ACEA-E30851223B5F.png)



TableAPI和SQL都建立在DataSetAPI和DataStreamAPI的基础之上，那么TableAPI和SQL是如何转换为DataStream和DataSet的呢？



TableAPI&SQL到DataStrem&DataSet的架构


TableAPI&SQL最终会经过Calcite优化之后转换为DataStream和DataSet，具体转换示意如下：

![b491eca0a60b6bd617e2a48124795e6d](Apache-Flink漫谈系列(1)-概述.resources/DC074C98-3D57-4997-AAF6-896BEF272F84.png)

对于流任务最终会转换成DataStream，对于批任务最终会转换成DataSet。



ANSI-SQL的支持


Apache Flink 之所以利用ANSI-SQL作为用户统一的开发语言，是因为SQL有着非常明显的优点，如下：

![e2f124f35ef0ba1c14ae9cc7b995b7ef](Apache-Flink漫谈系列(1)-概述.resources/7C067E43-57A2-4A83-BE9C-3C769DDFF6C6.png)


Declarative - 用户只需要表达我想要什么，不用关心如何计算。

Optimized - 查询优化器可以为用户的 SQL 生成最优的执行计划，获取最好的查询性能。

Understandable - SQL语言被不同领域的人所熟知，用SQL 作为跨团队的开发语言可以很大地提高效率。

Stable - SQL 是一个拥有几十年历史的语言，是一个非常稳定的语言，很少有变动。

Unify - Apache Flink在引擎上对流与批进行统一，同时又利用ANSI-SQL在语法和语义层面进行统一。



无限扩展的优化机制


Apache Flink 利用Apache Calcite对SQL进行解析和优化，Apache Calcite采用Calcite是开源的一套查询引擎，实现了两套Planner：



HepPlanner - 是RBO(Rule Base Optimize)模式，基于规则的优化。

VolcanoPlanner - 是CBO(Cost Base Optimize)模式，基于成本的优化。



Flink SQL会利用Calcite解析优化之后，最终转换为底层的DataStrem和Dataset。上图中 Batch rules和Stream rules可以根据优化需要无限添加优化规则。



Apache Flink 优秀的架构就像一座摩天大厦的地基一样为Apache Flink 持久的生命力打下了良好的基础，为打造Apache Flink丰富的功能生态留下无限的空间。



类库


CEP - 复杂事件处理类库，核心是一个状态机，广泛应用于事件驱动的监控预警类业务场景。

ML - 机器学习类库，机器学习主要是识别数据中的关系、趋势和模式，一般应用在预测类业务场景。

GELLY - 图计算类库，图计算更多的是考虑边和点的概念，一般被用来解决网状关系的业务场景。



算子


Apache Flink 提供了丰富的功能算子，对于数据流的处理来讲，可以分为单流处理(一个数据源)和多流处理(多个数据源)。



多流操作


如上通过UION和JOIN我们可以将多流最终变成单流，Apache Flink 在单流上提供了更多的操作算子。



单流操作


将多流变成单流之后，我们按数据输入输出的不同归类如下：



类型	输入	输出	Table/SQL算子	DataStream/DataSet算子
Scalar Function	1	1	Built-in & UDF,	Map
Table Function	1	N(N>=0)	Built-in & UDTF	FlatMap
Aggregate Function	N(N>=0)	1	Built-in & UDAF	Reduce


如上表格对单流上面操作做简单归类，除此之外还可以做 过滤，排序，窗口等操作，我们后续章节会逐一介绍。



存在的问题


Apache Flink 目前的架构还存在很大的优化空间，比如前面提到的DataStreamAPI和DataSetAPI其实是流与批在API层面不统一的体现，同时看具体实现会发现DataStreamAPI会生成Transformation tree然后生成StreamGraph，最后生成JobGraph，底层对应StreamTask，但DataSetAPI会形成Operator tree，flink-optimize模块会对Batch Plan进行优化，形成Optimized Plan 后形成JobGraph,最后形成BatchTask。具体示意如下：

![bb52accfabda8668b66b21ddc0e6380c](Apache-Flink漫谈系列(1)-概述.resources/1DEA43A8-381D-451C-9B99-9106B2B058B5.png)

这种情况其实 DataStreamAPI到Runtime 和 DataSetAPI到Runtime的实现上并没有得到最大程度的统一和复用。在这一点上面Aalibab 企业版的Flink在架构和实现上都进行了进一步优化。



组件栈


Alibaba 对Apache Flink进行了大量的架构优化，如下架构是一直努力的方向，大部分功能还在持续开发中，具体如下：


![41be7de757381783db0856e2d9ca0832](Apache-Flink漫谈系列(1)-概述.resources/928F452D-B175-4740-A139-8186EEDAC99C.png)

如上架构我们发现较大的变化是：



QP/QE/QO - 我们增加了QP/QE/QO层，在这一层进行统一的流和批的查询优化和底层算子的转换。

DAG API - 我们在Runtime层面统一抽象API接口，在API层对流与批进行统一。



TableAPI&SQL到Runtime的架构


Apache Flink执行层是流批统一的设计，在API和算子设计上面我们尽量达到流批的共享，在TableAPI和SQL层无论是流任务还是批任务最终都转换为统一的底层实现。这个层面最核心的变化是批最终也会生成StreamGraph，执行层运行Stream Task，如下：

![153476e3b50765b18f6516827c2a28f5](Apache-Flink漫谈系列(1)-概述.resources/61F69F73-CEE1-43F4-BB88-177A115FC62E.png)

本篇概要的介绍了"批是流的特例"这一设计观点是Apache Flink的"命脉"，它决定了Apache Flink的运行模式是纯流式的，这在实时计算场景的"低延迟"需求上，相对于Micro Batching模式占据了架构的绝对优势，同时概要的向大家介绍了Apache Flink的部署模式，容错处理，引擎的统一性和Apache Flink的架构，最后和大家分享了Apache Flink的优化架构。

本篇没有对具体技术进行详细展开，大家只要对Apache Flink有初步感知，头脑中知道Alibaba对Apache Flink进行了架构优化，增加了众多功能就可以了，至于Apache Flink的具体技术细节和实现原理，以及Alibaba对Apache Flink做了哪些架构优化和增加了哪些功能后续章节会展开介绍！