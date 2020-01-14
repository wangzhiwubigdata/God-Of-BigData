本文是由code4craft发表在博客上的，原文基于Netty3.7的版本，源码部分对buffer、Pipeline、Reactor模式等进行了部分讲解，个人又继续新增了后续的几个核心组件的源码解读，新增了具体的案例。
Netty的源码非常好，质量极高，是Java中质量最高的开源项目之一，(比Spring系列源码高几层楼...)
我十分建议大家花上一周时间自习读一读。

## 概述

### Netty是什么

大概用Netty的，无论新手还是老手，都知道它是一个“网络通讯框架”。所谓框架，基本上都是一个作用：基于底层API，提供更便捷的编程模型。那么"通讯框架"到底做了什么事情呢？回答这个问题并不太容易，我们不妨反过来看看，不使用netty，直接基于NIO编写网络程序，你需要做什么(以Server端TCP连接为例，这里我们使用Reactor模型)：


1. 监听端口，建立Socket连接
2. 建立线程，处理内容
	1. 读取Socket内容，并对协议进行解析
	2. 进行逻辑处理
	3. 回写响应内容
	4. 如果是多次交互的应用(SMTP、FTP)，则需要保持连接多进行几次交互
3. 关闭连接

建立线程是一个比较耗时的操作，同时维护线程本身也有一些开销，所以我们会需要多线程机制，幸好JDK已经有很方便的多线程框架了，这里我们不需要花很多心思。
	
此外，因为TCP连接的特性，我们还要使用连接池来进行管理：

1. 建立TCP连接是比较耗时的操作，对于频繁的通讯，保持连接效果更好
2. 对于并发请求，可能需要建立多个连接
3. 维护多个连接后，每次通讯，需要选择某一可用连接
4. 连接超时和关闭机制

想想就觉得很复杂了！实际上，基于NIO直接实现这部分东西，即使是老手也容易出现错误，而使用Netty之后，你只需要关注逻辑处理部分就可以了。


### 体验Netty

这里我们引用Netty的example包里的一个例子，一个简单的EchoServer，它接受客户端输入，并将输入原样返回。其主要代码如下：

```java
    public void run() {
        // Configure the server.
        ServerBootstrap bootstrap = new ServerBootstrap(
                new NioServerSocketChannelFactory(
                        Executors.newCachedThreadPool(),
                        Executors.newCachedThreadPool()));

        // Set up the pipeline factory.
        bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
            public ChannelPipeline getPipeline() throws Exception {
                return Channels.pipeline(new EchoServerHandler());
            }
        });

        // Bind and start to accept incoming connections.
        bootstrap.bind(new InetSocketAddress(port));
    }
```

这里`EchoServerHandler`是其业务逻辑的实现者，大致代码如下：

```java
	public class EchoServerHandler extends SimpleChannelUpstreamHandler {

	    @Override
	    public void messageReceived(
	            ChannelHandlerContext ctx, MessageEvent e) {
	        // Send back the received message to the remote peer.
	        e.getChannel().write(e.getMessage());
	    }
	}
```
	
还是挺简单的，不是吗？

### Netty背后的事件驱动机制

完成了以上一段代码，我们算是与Netty进行了第一次亲密接触。如果想深入学习呢？

阅读源码是了解一个开源工具非常好的手段，但是Java世界的框架大多追求大而全，功能完备，如果逐个阅读，难免迷失方向，Netty也并不例外。相反，抓住几个重点对象，理解其领域概念及设计思想，从而理清其脉络，相当于打通了任督二脉，以后的阅读就不再困难了。

理解Netty的关键点在哪呢？我觉得，除了NIO的相关知识，另一个就是事件驱动的设计思想。什么叫事件驱动？我们回头看看`EchoServerHandler`的代码，其中的参数：`public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)`，MessageEvent就是一个事件。这个事件携带了一些信息，例如这里`e.getMessage()`就是消息的内容，而`EchoServerHandler`则描述了处理这种事件的方式。一旦某个事件触发，相应的Handler则会被调用，并进行处理。这种事件机制在UI编程里广泛应用，而Netty则将其应用到了网络编程领域。

在Netty里，所有事件都来自`ChannelEvent`接口，这些事件涵盖监听端口、建立连接、读写数据等网络通讯的各个阶段。而事件的处理者就是`ChannelHandler`，这样，不但是业务逻辑，连网络通讯流程中底层的处理，都可以通过实现`ChannelHandler`来完成了。事实上，Netty内部的连接处理、协议编解码、超时等机制，都是通过handler完成的。当博主弄明白其中的奥妙时，不得不佩服这种设计！

下图描述了Netty进行事件处理的流程。`Channel`是连接的通道，是ChannelEvent的产生者，而`ChannelPipeline`可以理解为ChannelHandler的集合。

![event driven in Netty][1]


### 开启Netty源码之门

理解了Netty的事件驱动机制，我们现在可以来研究Netty的各个模块了。Netty的包结构如下：

	org
	└── jboss
	    └── netty
			├── bootstrap 配置并启动服务的类
			├── buffer 缓冲相关类，对NIO Buffer做了一些封装
			├── channel 核心部分，处理连接
			├── container 连接其他容器的代码
			├── example 使用示例
			├── handler 基于handler的扩展部分，实现协议编解码等附加功能
			├── logging 日志
			└── util 工具类

在这里面，`channel`和`handler`两部分比较复杂。我们不妨与Netty官方的结构图对照一下，来了解其功能。

![components in Netty][2]

具体的解释可以看这里：[http://netty.io/3.7/guide/#architecture](http://netty.io/3.7/guide/#architecture)。图中可以看到，除了之前说到的事件驱动机制之外，Netty的核心功能还包括两部分：

* Zero-Copy-Capable Rich Byte Buffer

	零拷贝的Buffer。为什么叫零拷贝？因为在数据传输时，最终处理的数据会需要对单个传输层的报文，进行组合或者拆分。NIO原生的ByteBuffer无法做到这件事，而Netty通过提供Composite(组合)和Slice(切分)两种Buffer来实现零拷贝。这部分代码在`org.jboss.netty.buffer`包中。
	这里需要额外注意，不要和操作系统级别的Zero-Copy混淆了, 操作系统中的零拷贝主要是用户空间和内核空间之间的数据拷贝, NIO中通过DirectBuffer做了实现.

* Universal Communication API
	
	统一的通讯API。这个是针对Java的Old I/O和New I/O，使用了不同的API而言。Netty则提供了统一的API(`org.jboss.netty.channel.Channel`)来封装这两种I/O模型。这部分代码在`org.jboss.netty.channel`包中。
	
此外，Protocol Support功能通过handler机制实现。

接下来的文章，我们会根据模块，详细的对Netty源码进行分析。


### 参考资料：

* Netty 3.7 User Guide [http://netty.io/3.7/guide/](http://netty.io/3.7/guide/)

* What is Netty? [http://ayedo.github.io/netty/2013/06/19/what-is-netty.html](http://ayedo.github.io/netty/2013/06/19/what-is-netty.html)

  [1]: http://static.oschina.net/uploads/space/2013/0921/174032_18rb_190591.png
  [2]: http://static.oschina.net/uploads/space/2013/0921/225721_R0w2_190591.png
