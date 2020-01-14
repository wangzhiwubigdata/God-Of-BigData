## 引言

《分布式系统理论基础 - 一致性、2PC和3PC》一文介绍了一致性、达成一致性需要面临的各种问题以及2PC、3PC模型，Paxos协议在节点宕机恢复、消息无序或丢失、网络分化的场景下能保证决议的一致性，是被讨论最广泛的一致性协议。



Paxos协议同时又以其“艰深晦涩”著称，下面结合 Paxos Made Simple、The Part-Time Parliament 两篇论文，尝试通过Paxos推演、学习和了解Paxos协议。

 
## Basic Paxos

何为一致性问题？简单而言，一致性问题是在节点宕机、消息无序等场景可能出现的情况下，相互独立的节点之间如何达成决议的问题，作为解决一致性问题的协议，Paxos的核心是节点间如何确定并只确定一个值(value)。


也许你会疑惑只确定一个值能起什么作用，在Paxos协议里确定并只确定一个值是确定多值的基础，如何确定多值将在第二部分Multi Paxos中介绍，这部分我们聚焦在“Paxos如何确定并只确定一个值”这一问题上。

和2PC类似，Paxos先把节点分成两类，发起提议(proposal)的一方为proposer，参与决议的一方为acceptor。假如只有一个proposer发起提议，并且节点不宕机、消息不丢包，那么acceptor做到以下这点就可以确定一个值：
```
P1. 一个acceptor接受它收到的第一项提议
```
当然上面要求的前提条件有些严苛，节点不能宕机、消息不能丢包，还只能由一个proposer发起提议。我们尝试放宽条件，假设多个proposer可以同时发起提议，又怎样才能做到确定并只确定一个值呢？


首先proposer和acceptor需要满足以下两个条件：
1. proposer发起的每项提议分别用一个ID标识，提议的组成因此变为(ID, value)

2. acceptor可以接受(accept)不止一项提议，当多数(quorum) acceptor接受一项提议时该提议被确定(chosen)

(注: 注意以上“接受”和“确定”的区别）
我们约定后面发起的提议的ID比前面提议的ID大，并假设可以有多项提议被确定，为做到确定并只确定一个值acceptor要做到以下这点：

```
P2. 如果一项值为v的提议被确定，那么后续只确定值为v的提议
```
(注: 乍看这个条件不太好理解，谨记目标是“确定并只确定一个值”)

由于一项提议被确定(chosen)前必须先被多数派acceptor接受(accepted)，为实现P2，实质上acceptor需要做到：
```
P2a. 如果一项值为v的提议被确定，那么acceptor后续只接受值为v的提议
```
满足P2a则P2成立 (P2a => P2)。

 

目前在多个proposer可以同时发起提议的情况下，满足P1、P2a即能做到确定并只确定一个值。如果再加上节点宕机恢复、消息丢包的考量呢？

 

假设acceptor c 宕机一段时间后恢复，c 宕机期间其他acceptor已经确定了一项值为v的决议但c 因为宕机并不知晓；c 恢复后如果有proposer马上发起一项值不是v的提议，由于条件P1，c 会接受该提议，这与P2a矛盾。为了避免这样的情况出现，进一步地我们对proposer作约束：


```
P2b. 如果一项值为v的提议被确定，那么proposer后续只发起值为v的提议
```
满足P2b则P2a成立 (P2b => P2a => P2)。

 

P2b约束的是提议被确定(chosen)后proposer的行为，我们更关心提议被确定前proposer应该怎么做：
```
P2c. 对于提议(n,v)，acceptor的多数派S中，如果存在acceptor最近一次(即ID值最大)接受的提议的值为v'，那么要求v = v'；否则v可为任意值
```
满足P2c则P2b成立 (P2c => P2b => P2a => P2)。

 

条件P2c是Basic Paxos的核心，光看P2c的描述可能会觉得一头雾水，我们通过 The Part-Time Parliament 中的例子加深理解：

![97d42599c38593043960af7a662cd4c4](分布式系统理论进阶 - Paxos.resources/AAEEBF57-8486-4EAC-B8F7-C66AE6967BF9.png)

假设有A~E 5个acceptor，- 表示acceptor因宕机等原因缺席当次决议，x 表示acceptor不接受提议，o 表示接受提议；多数派acceptor接受提议后提议被确定，以上表格对应的决议过程如下：

1.ID为2的提议最早提出，根据P2c其提议值可为任意值，这里假设为a
2.acceptor A/B/C/E 在之前的决议中没有接受(accept)任何提议，因而ID为5的提议的值也可以为任意值，这里假设为b
3.acceptor B/D/E，其中D曾接受ID为2的提议，根据P2c，该轮ID为14的提议的值必须与ID为2的提议的值相同，为a
4.acceptor A/C/D，其中D曾接受ID为2的提议、C曾接受ID为5的提议，相比之下ID 5较ID 2大，根据P2c，该轮ID为27的提议的值必须与ID为5的提议的值相同，为b；该轮决议被多数派acceptor接受，因此该轮决议得以确定
5.acceptor B/C/D，3个acceptor之前都接受过提议，相比之下C、D曾接受的ID 27的ID号最大，该轮ID为29的提议的值必须与ID为27的提议的值相同，为b

以上提到的各项约束条件可以归纳为3点，如果proposer/acceptor满足下面3点，那么在少数节点宕机、网络分化隔离的情况下，在“确定并只确定一个值”这件事情上可以保证一致性(consistency)：
B1(ß): ß中每一轮决议都有唯一的ID标识
B2(ß): 如果决议B被acceptor多数派接受，则确定决议B
B3(ß): 对于ß中的任意提议B(n,v)，acceptor的多数派中如果存在acceptor最近一次(即ID值最大)接受的提议的值为v'，那么要求v = v'；否则v可为任意值

(注: 希腊字母ß表示多轮决议的集合，字母B表示一轮决议)

另外为保证P2c，我们对acceptor作两个要求：

1. 记录曾接受的ID最大的提议，因proposer需要问询该信息以决定提议值

2. 在回应提议ID为n的proposer自己曾接受过ID最大的提议时，acceptor同时保证(promise)不再接受ID小于n的提议

 

至此，proposer/acceptor完成一轮决议可归纳为prepare和accept两个阶段。prepare阶段proposer发起提议问询提议值、acceptor回应问询并进行promise；accept阶段完成决议，图示如下：

![9000315110586d6ebbdf455c7a03d3e0](分布式系统理论进阶 - Paxos.resources/CF8854FA-A0E7-4FEF-A9F2-ABC66CA49297.png)

还有一个问题需要考量，假如proposer A发起ID为n的提议，在提议未完成前proposer B又发起ID为n+1的提议，在n+1提议未完成前proposer C又发起ID为n+2的提议…… 如此acceptor不能完成决议、形成活锁(livelock)，虽然这不影响一致性，但我们一般不想让这样的情况发生。解决的方法是从proposer中选出一个leader，提议统一由leader发起。

 

最后我们再引入一个新的角色：learner，learner依附于acceptor，用于习得已确定的决议。以上决议过程都只要求acceptor多数派参与，而我们希望尽量所有acceptor的状态一致。如果部分acceptor因宕机等原因未知晓已确定决议，宕机恢复后可经本机learner采用pull的方式从其他acceptor习得。


## Multi Paxos

通过以上步骤分布式系统已经能确定一个值，“只确定一个值有什么用？这可解决不了我面临的问题。” 你心中可能有这样的疑问。
 
其实不断地进行“确定一个值”的过程、再为每个过程编上序号，就能得到具有全序关系(total order)的系列值，进而能应用在数据库副本存储等很多场景。我们把单次“确定一个值”的过程称为实例(instance)，它由proposer/acceptor/learner组成，下图说明了A/B/C三机上的实例：

![1824716a2ad8ce8300c01f2a876e63fb](分布式系统理论进阶 - Paxos.resources/225D3AF6-7029-4731-A45A-2A66050338BB.png)

不同序号的实例之间互相不影响，A/B/C三机输入相同、过程实质等同于执行相同序列的状态机(state machine)指令 ，因而将得到一致的结果。


proposer leader在Multi Paxos中还有助于提升性能，常态下统一由leader发起提议，可节省prepare步骤(leader不用问询acceptor曾接受过的ID最大的提议、只有leader提议也不需要acceptor进行promise)直至发生leader宕机、重新选主。

## 小结

以上介绍了Paxos的推演过程、如何在Basic Paxos的基础上通过状态机构建Multi Paxos。Paxos协议比较“艰深晦涩”，但多读几遍论文一般能理解其内涵，更难的是如何将Paxos真正应用到工程实践。

 

微信后台开发同学实现并开源了一套基于Paxos协议的多机状态拷贝类库PhxPaxos，PhxPaxos用于将单机服务扩展到多机，其经过线上系统验证并在一致性保证、性能等方面作了很多考量。