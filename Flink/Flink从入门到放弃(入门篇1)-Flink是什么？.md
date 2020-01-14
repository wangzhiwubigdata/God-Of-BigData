

> 本文是例行介绍，熟悉的直接跳过 - 鲁迅

> 鲁迅: ...

# 大纲

**入门篇：**
![8bf609799f0f1265e71fa5bd0d498c45](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/1.png)

**放弃篇：**
![563c79efb6518f991c732f1d95f97a62](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/A44BE2B6-FBC9-4143-9743-F097B9C0FDD6.png)




## Flink是什么

## 一句话概括
Apache Flink是一个面向分布式数据流处理和批量数据处理的开源计算平台，提供支持流处理和批处理两种类型应用的功能。

## 前身
Apache Flink 的前身是柏林理工大学一个研究性项目， 在 2014 被 Apache 孵化器所接受，然后迅速地成为了Apache Software Foundation的顶级项目之一。

## 特点

现有的开源计算方案，会把流处理和批处理作为两种不同的应用类型：流处理一般需要支持低延迟、Exactly-once保证，而批处理需要支持高吞吐、高效处理。
Flink是完全支持流处理，也就是说作为流处理看待时输入数据流是无界的；批处理被作为一种特殊的流处理，只是它的输入数据流被定义为有界的。


## Flink组件栈

![102b82e4ca65fa679cee53c017d830aa](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/6F963775-B91B-447F-959E-38B4029BE56D.png)

### Deployment层	
主要涉及了Flink的部署模式，Flink支持多种部署模式：本地、集群（Standalone/YARN）、云（GCE/EC2）

![cd3ae86f9ae0764f77af85696114d79b](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/F7406066-68CA-4BE7-9743-7FD65A0D722C.png)

### Runtime层 

Runtime层提供了支持Flink计算的全部核心实现，比如：支持分布式Stream处理、JobGraph到ExecutionGraph的映射、调度等等，为上层API层提供基础服务

### API层

API层主要实现了面向无界Stream的流处理和面向Batch的批处理API，其中面向流处理对应DataStream API，面向批处理对应DataSet API 

### Libaries层 

* 在API层之上构建的满足特定应用的实现计算框架，也分别对应于面向流处理和面向批处理两类

* 面向流处理支持：CEP（复杂事件处理）、基于SQL-like的操作（基于Table的关系操作）

* 面向批处理支持：FlinkML（机器学习库）、Gelly（图处理）


## Flink的优势

* 支持高吞吐、低延迟、高性能的流处理
* 支持高度灵活的窗口（Window）操作
* 支持有状态计算的Exactly-once语义
* 提供DataStream API和DataSet API

![d698791fc0eee2b74bfb9af430206705](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/3DE5BD22-BFE2-49C4-8DA8-C42EAD1948FB.png)

![f49a3e84af366184696c6c6800d84a50](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/0E6F6341-5EB0-40FD-9953-70C3F0904043.png)



## Flink基本编程模型

> * Flink程序的基础构建模块是流(streams) 与 转换(transformations)
> * 每一个数据流起始于一个或多个 source，并终止于一个或多个 sink


下面是一个由Flink程序映射为Streaming Dataflow的示意图:

![1cee64d1b99673231aa5315d579d5182](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/656C0986-42A7-4E76-B3CA-C0372395E451.png)

并行数据流示意图:
![b51a95236221451ab1958f8aefc5af62](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/E6A4AF88-12D9-413A-A318-06A86ABDC1AF.png)


## Flink基本架构

> * Flink是基于Master-Slave风格的架构
> * Flink集群启动时，会启动一个JobManager进程、至少一个TaskManager进程

![7c947040b492ea28cd48252a0f1427a7](Flink从入门到放弃(入门篇1)-Flink是什么？.resources/866EF50B-A9ED-461A-AC13-78BEBBDCCFC9.png)

### JobManager

* Flink系统的协调者，它负责接收Flink Job，调度组成Job的多个Task的执行

* 收集Job的状态信息，并管理Flink集群中从节点TaskManager

### TaskManager

* 实际负责执行计算的Worker，在其上执行Flink Job的一组Task
* TaskManager负责管理其所在节点上的资源信息，如内存、磁盘、网络，在启动的时候将资源的状态向JobManager汇报

### Client

* 用户提交一个Flink程序时，会首先创建一个Client，该Client首先会对用户提交的Flink程序进行预处理，并提交到Flink集群

* Client会将用户提交的Flink程序组装一个JobGraph， 并且是以JobGraph的形式提交的


## 最后

本文是例行介绍，熟悉的直接跳过。
