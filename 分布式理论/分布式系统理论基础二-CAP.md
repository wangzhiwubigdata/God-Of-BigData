## 引言

CAP是分布式系统、特别是分布式存储领域中被讨论最多的理论，“什么是CAP定理？”在Quora 分布式系统分类下排名 FAQ 的 No.1。CAP在程序员中也有较广的普及，它不仅仅是“C、A、P不能同时满足，最多只能3选2”，以下尝试综合各方观点，从发展历史、工程实践等角度讲述CAP理论。希望大家透过本文对CAP理论有更多地了解和认识。
 
## CAP定理

CAP由Eric Brewer在2000年PODC会议上提出[1][2]，是Eric Brewer在Inktomi[3]期间研发搜索引擎、分布式web缓存时得出的关于数据一致性(consistency)、服务可用性(availability)、分区容错性(partition-tolerance)的猜想：
It is impossible for a web service to provide the three following guarantees : Consistency, Availability and Partition-tolerance.
 
该猜想在提出两年后被证明成立[4]，成为我们熟知的CAP定理：

* 数据一致性(consistency)：如果系统对一个写操作返回成功，那么之后的读请求都必须读到这个新数据；如果返回失败，那么所有读操作都不能读到这个数据，对调用者而言数据具有强一致性(strong consistency) (又叫原子性 atomic、线性一致性 linearizable consistency)
* 服务可用性(availability)：所有读写请求在一定时间内得到响应，可终止、不会一直等待
* 分区容错性(partition-tolerance)：在网络分区的情况下，被分隔的节点仍能正常对外服务 

在某时刻如果满足AP，分隔的节点同时对外服务但不能相互通信，将导致状态不一致，即不能满足C；如果满足CP，网络分区的情况下为达成C，请求只能一直等待，即不满足A；如果要满足CA，在一定时间内要达到节点状态一致，要求不能出现网络分区，则不能满足P。
 
C、A、P三者最多只能满足其中两个，和FLP定理一样，CAP定理也指示了一个不可达的结果(impossibility result)。

![91ac5357657dbc883f5b40725f7b4140](分布式系统理论基础二-CAP.resources/F2B0C729-3117-4256-BBED-AC3105092464.png)

## CAP的工程启示
CAP理论提出7、8年后，NoSql圈将CAP理论当作对抗传统关系型数据库的依据、阐明自己放宽对数据一致性(consistency)要求的正确性[6]，随后引起了大范围关于CAP理论的讨论。
 
CAP理论看似给我们出了一道3选2的选择题，但在工程实践中存在很多现实限制条件，需要我们做更多地考量与权衡，避免进入CAP认识误区[7]。
 
### 1、关于 P 的理解
Partition字面意思是网络分区，即因网络因素将系统分隔为多个单独的部分，有人可能会说，网络分区的情况发生概率非常小啊，是不是不用考虑P，保证CA就好。要理解P，我们看回CAP证明中P的定义：
In order to model partition tolerance, the network will be allowed to lose arbitrarily many messages sent from one node to another.
 
网络分区的情况符合该定义，网络丢包的情况也符合以上定义，另外节点宕机，其他节点发往宕机节点的包也将丢失，这种情况同样符合定义。现实情况下我们面对的是一个不可靠的网络、有一定概率宕机的设备，这两个因素都会导致Partition，因而分布式系统实现中 P 是一个必须项，而不是可选项。
 
对于分布式系统工程实践，CAP理论更合适的描述是：在满足分区容错的前提下，没有算法能同时满足数据一致性和服务可用性[11]：
In a network subject to communication failures, it is impossible for any web service to implement an atomic read/write shared memory that guarantees a response to every request.
 
### 2、CA非0/1的选择
CAP定理证明中的一致性指强一致性，强一致性要求多节点组成的被调要能像单节点一样运作、操作具备原子性，数据在时间、时序上都有要求。如果放宽这些要求，还有其他一致性类型：

* 序列一致性(sequential consistency)：不要求时序一致，A操作先于B操作，在B操作后如果所有调用端读操作得到A操作的结果，满足序列一致性

* 最终一致性(eventual consistency)：放宽对时间的要求，在被调完成操作响应后的某个时间点，被调多个节点的数据最终达成一致

 
可用性在CAP定理里指所有读写操作必须要能终止，实际应用中从主调、被调两个不同的视角，可用性具有不同的含义。当P(网络分区)出现时，主调可以只支持读操作，通过牺牲部分可用性达成数据一致。

工程实践中，较常见的做法是通过异步拷贝副本(asynchronous replication)、quorum/NRW，实现在调用端看来数据强一致、被调端最终一致，在调用端看来服务可用、被调端允许部分节点不可用(或被网络分隔)的效果。
 
### 3、跳出CAP
CAP理论对实现分布式系统具有指导意义，但CAP理论并没有涵盖分布式工程实践中的所有重要因素。
 
例如延时(latency)，它是衡量系统可用性、与用户体验直接相关的一项重要指标。CAP理论中的可用性要求操作能终止、不无休止地进行，除此之外，我们还关心到底需要多长时间能结束操作，这就是延时，它值得我们设计、实现分布式系统时单列出来考虑。
 
延时与数据一致性也是一对“冤家”，如果要达到强一致性、多个副本数据一致，必然增加延时。加上延时的考量，我们得到一个CAP理论的修改版本PACELC：如果出现P(网络分区)，如何在A(服务可用性)、C(数据一致性)之间选择；否则，如何在L(延时)、C(数据一致性)之间选择。
 
小结以上介绍了CAP理论的源起和发展，介绍了CAP理论给分布式系统工程实践带来的启示。
 
CAP理论对分布式系统实现有非常重大的影响，我们可以根据自身的业务特点，在数据一致性和服务可用性之间作出倾向性地选择。通过放松约束条件，我们可以实现在不同时间点满足CAP(此CAP非CAP定理中的CAP，如C替换为最终一致性)。
 
有非常非常多文章讨论和研究CAP理论，希望这篇对你认识和了解CAP理论有帮助。


 
[1] Harvest, Yield, and Scalable Tolerant Systems, Armando Fox , Eric Brewer, 1999

[2] Towards Robust Distributed Systems, Eric Brewer, 2000

[3] Inktomi's wild ride - A personal view of the Internet bubble, Eric Brewer, 2004

[4] Brewer’s Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web, Seth Gilbert, Nancy Lynch, 2002

[5] Linearizability: A Correctness Condition for Concurrent Objects, Maurice P. Herlihy,Jeannette M. Wing, 1990

[6] Brewer's CAP Theorem - The kool aid Amazon and Ebay have been drinking, Julian Browne, 2009

[7] CAP Theorem between Claims and Misunderstandings: What is to be Sacrificed?, Balla Wade Diack,Samba Ndiaye,Yahya Slimani, 2013

[8] Errors in Database Systems, Eventual Consistency, and the CAP Theorem, Michael Stonebraker, 2010

[9] CAP Confusion: Problems with 'partition tolerance', Henry Robinson, 2010

[10] You Can’t Sacrifice Partition Tolerance, Coda Hale, 2010

[11] Perspectives on the CAP Theorem, Seth Gilbert, Nancy Lynch, 2012

[12] CAP Twelve Years Later: How the "Rules" Have Changed, Eric Brewer, 2012

[13] How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs, Lamport Leslie, 1979

[14] Eventual Consistent Databases: State of the Art, Mawahib Elbushra , Jan Lindström, 2014

[15] Eventually Consistent, Werner Vogels, 2008

[16] Speed Matters for Google Web Search, Jake Brutlag, 2009

[17] Consistency Tradeoffs in Modern Distributed Database System Design, Daniel J. Abadi, 2012

[18] A CAP Solution (Proving Brewer Wrong), Guy's blog, 2008

[19] How to beat the CAP theorem, nathanmarz , 2011

[20] The CAP FAQ, Henry Robinson
