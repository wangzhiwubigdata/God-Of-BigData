
## Channel实现概览

在Netty里，`Channel`是通讯的载体，而`ChannelHandler`负责Channel中的逻辑处理。

那么`ChannelPipeline`是什么呢？我觉得可以理解为ChannelHandler的容器：一个Channel包含一个ChannelPipeline，所有ChannelHandler都会注册到ChannelPipeline中，并按顺序组织起来。

在Netty中，`ChannelEvent`是数据或者状态的载体，例如传输的数据对应`MessageEvent`，状态的改变对应`ChannelStateEvent`。当对Channel进行操作时，会产生一个ChannelEvent，并发送到`ChannelPipeline`。ChannelPipeline会选择一个ChannelHandler进行处理。这个ChannelHandler处理之后，可能会产生新的ChannelEvent，并流转到下一个ChannelHandler。

![channel pipeline][1]


例如，一个数据最开始是一个`MessageEvent`，它附带了一个未解码的原始二进制消息`ChannelBuffer`，然后某个Handler将其解码成了一个数据对象，并生成了一个新的`MessageEvent`，并传递给下一步进行处理。

到了这里，可以看到，其实Channel的核心流程位于`ChannelPipeline`中。于是我们进入ChannelPipeline的深层梦境里，来看看它具体的实现。

## ChannelPipeline的主流程

Netty的ChannelPipeline包含两条线路：Upstream和Downstream。Upstream对应上行，接收到的消息、被动的状态改变，都属于Upstream。Downstream则对应下行，发送的消息、主动的状态改变，都属于Downstream。`ChannelPipeline`接口包含了两个重要的方法:`sendUpstream(ChannelEvent e)`和`sendDownstream(ChannelEvent e)`，就分别对应了Upstream和Downstream。

对应的，ChannelPipeline里包含的ChannelHandler也包含两类：`ChannelUpstreamHandler`和`ChannelDownstreamHandler`。每条线路的Handler是互相独立的。它们都很简单的只包含一个方法：`ChannelUpstreamHandler.handleUpstream`和`ChannelDownstreamHandler.handleDownstream`。

Netty官方的javadoc里有一张图(`ChannelPipeline`接口里)，非常形象的说明了这个机制(我对原图进行了一点修改，加上了`ChannelSink`，因为我觉得这部分对理解代码流程会有些帮助)：

![channel pipeline][2]

什么叫`ChannelSink`呢？ChannelSink包含一个重要方法`ChannelSink.eventSunk`，可以接受任意ChannelEvent。"sink"的意思是"下沉"，那么"ChannelSink"好像可以理解为"Channel下沉的地方"？实际上，它的作用确实是这样，也可以换个说法："处于末尾的万能Handler"。最初读到这里，也有些困惑，这么理解之后，就感觉简单许多。**只有Downstream包含`ChannelSink`**，这里会做一些建立连接、绑定端口等重要操作。为什么UploadStream没有ChannelSink呢？我只能认为，一方面，不符合"sink"的意义，另一方面，也没有什么处理好做的吧！

这里有个值得注意的地方：在一条“流”里，一个`ChannelEvent`并不会主动的"流"经所有的Handler，而是由**上一个Handler显式的调用`ChannelPipeline.sendUp(Down)stream`产生，并交给下一个Handler处理**。也就是说，每个Handler接收到一个ChannelEvent，并处理结束后，如果需要继续处理，那么它需要调用`sendUp(Down)stream`新发起一个事件。如果它不再发起事件，那么处理就到此结束，即使它后面仍然有Handler没有执行。这个机制可以保证最大的灵活性，当然对Handler的先后顺序也有了更严格的要求。

顺便说一句，在Netty 3.x里，这个机制会导致大量的ChannelEvent对象创建，因此Netty 4.x版本对此进行了改进。twitter的[finagle](https://github.com/twitter/finagle)框架实践中，就提到从Netty 3.x升级到Netty 4.x，可以大大降低GC开销。有兴趣的可以看看这篇文章：[https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead](https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead)

下面我们从代码层面来对这里面发生的事情进行深入分析，这部分涉及到一些细节，需要打开项目源码，对照来看，会比较有收获。

## 深入ChannelPipeline内部

### DefaultChannelPipeline的内部结构

`ChannelPipeline`的主要的实现代码在`DefaultChannelPipeline`类里。列一下DefaultChannelPipeline的主要字段：

```java
    public class DefaultChannelPipeline implements ChannelPipeline {
    
        private volatile Channel channel;
        private volatile ChannelSink sink;
        private volatile DefaultChannelHandlerContext head;
        private volatile DefaultChannelHandlerContext tail;
        private final Map<String, DefaultChannelHandlerContext> name2ctx =
            new HashMap<String, DefaultChannelHandlerContext>(4);
    }
```

这里需要介绍一下`ChannelHandlerContext`这个接口。顾名思义，ChannelHandlerContext保存了Netty与Handler相关的上下文信息。而咱们这里的`DefaultChannelHandlerContext`，则是对`ChannelHandler`的一个包装。一个`DefaultChannelHandlerContext`内部，除了包含一个`ChannelHandler`，还保存了"next"和"prev"两个指针，从而形成一个双向链表。

因此，在`DefaultChannelPipeline`中，我们看到的是对`DefaultChannelHandlerContext`的引用，而不是对`ChannelHandler`的直接引用。这里包含"head"和"tail"两个引用，分别指向链表的头和尾。而name2ctx则是一个按名字索引DefaultChannelHandlerContext用户的一个map，主要在按照名称删除或者添加ChannelHandler时使用。

### sendUpstream和sendDownstream

前面提到了，`ChannelPipeline`接口的两个重要的方法：`sendUpstream(ChannelEvent e)`和`sendDownstream(ChannelEvent e)`。**所有事件**的发起都是基于这两个方法进行的。`Channels`类有一系列`fireChannelBound`之类的`fireXXXX`方法，其实都是对这两个方法的facade包装。

下面来看一下这两个方法的实现。先看sendUpstream(对代码做了一些简化，保留主逻辑)：

```java
    public void sendUpstream(ChannelEvent e) {
        DefaultChannelHandlerContext head = getActualUpstreamContext(this.head);
        head.getHandler().handleUpstream(head, e);
    }
    
    private DefaultChannelHandlerContext getActualUpstreamContext(DefaultChannelHandlerContext ctx) {
        DefaultChannelHandlerContext realCtx = ctx;
        while (!realCtx.canHandleUpstream()) {
            realCtx = realCtx.next;
            if (realCtx == null) {
                return null;
            }
        }
        return realCtx;
    }
```

这里最终调用了`ChannelUpstreamHandler.handleUpstream`来处理这个ChannelEvent。有意思的是，这里我们看不到任何"将Handler向后移一位"的操作，但是我们总不能每次都用同一个Handler来进行处理啊？实际上，我们更为常用的是`ChannelHandlerContext.handleUpstream`方法(实现是`DefaultChannelHandlerContext.sendUpstream`方法)：

```java
	public void sendUpstream(ChannelEvent e) {
		DefaultChannelHandlerContext next = getActualUpstreamContext(this.next);
		DefaultChannelPipeline.this.sendUpstream(next, e);
	}
```

可以看到，这里最终仍然调用了`ChannelPipeline.sendUpstream`方法，但是**它会将Handler指针后移**。

我们接下来看看`DefaultChannelHandlerContext.sendDownstream`:

```java
	public void sendDownstream(ChannelEvent e) {
		DefaultChannelHandlerContext prev = getActualDownstreamContext(this.prev);
		if (prev == null) {
			try {
				getSink().eventSunk(DefaultChannelPipeline.this, e);
			} catch (Throwable t) {
				notifyHandlerException(e, t);
			}
		} else {
			DefaultChannelPipeline.this.sendDownstream(prev, e);
		}
	}
```

与sendUpstream好像不大相同哦？这里有两点：一是到达末尾时，就如梦境二所说，会调用ChannelSink进行处理；二是这里指针是**往前移**的，所以我们知道了：

**UpstreamHandler是从前往后执行的，DownstreamHandler是从后往前执行的。**在ChannelPipeline里添加时需要注意顺序了！

DefaultChannelPipeline里还有些机制，像添加/删除/替换Handler，以及`ChannelPipelineFactory`等，比较好理解，就不细说了。

## 回到现实：Pipeline解决的问题

好了，深入分析完代码，有点头晕了，我们回到最开始的地方，来想一想，Netty的Pipeline机制解决了什么问题？

我认为至少有两点：

一是提供了ChannelHandler的编程模型，基于ChannelHandler开发业务逻辑，基本不需要关心网络通讯方面的事情，专注于编码/解码/逻辑处理就可以了。Handler也是比较方便的开发模式，在很多框架中都有用到。

二是实现了所谓的"Universal Asynchronous API"。这也是Netty官方标榜的一个功能。用过OIO和NIO的都知道，这两套API风格相差极大，要从一个迁移到另一个成本是很大的。即使是NIO，异步和同步编程差距也很大。而Netty屏蔽了OIO和NIO的API差异，通过Channel提供对外接口，并通过ChannelPipeline将其连接起来，因此替换起来非常简单。

![universal API][3]

理清了ChannelPipeline的主流程，我们对Channel部分的大致结构算是弄清楚了。可是到了这里，我们依然对一个连接具体怎么处理没有什么概念，下篇文章，我们会分析一下，在Netty中，捷径如何处理连接的建立、数据的传输这些事情。


  [1]: http://static.oschina.net/uploads/space/2013/0921/174032_18rb_190591.png
  [2]: http://static.oschina.net/uploads/space/2013/1109/075339_Kjw6_190591.png
  [3]: http://static.oschina.net/uploads/space/2013/1124/001528_TBb5_190591.jpg

参考资料：

* Sink [http://en.wikipedia.org/wiki/Sink_\(computing\)](http://en.wikipedia.org/wiki/Sink_\(computing\))