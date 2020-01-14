## Netty中的Handler简介
`Handler`在Netty中，占据着非常重要的地位。`Handler`与Servlet中的filter很像，通过Handler可以完成通讯报文的解码编码、拦截指定的报文、

统一对日志错误进行处理、统一对请求进行计数、控制Handler执行与否。一句话，没有它做不到的只有你想不到的

　　Netty中的所有handler都实现自ChannelHandler接口。按照输入输出来分，分为`ChannelInboundHandler`、`ChannelOutboundHandler`两大类

`ChannelInboundHandler`对从客户端发往服务器的报文进行处理，一般用来执行解码、读取客户端数据、进行业务处理等；`ChannelOutboundHandler`

对从服务器发往客户端的报文进行处理，一般用来进行编码、发送报文到客户端


　　Netty中可以注册多个handler。`ChannelInboundHandler`按照注册的先后顺序执行；`ChannelOutboundHandler`按照注册的先后顺序逆序执行。
  
  
ChannelPipeline中的事件不会自动流动，而我们一般需求事件自动流动，Netty提供了两个Adapter：ChannelInboundHandlerAdapter和ChannelOutboundHandlerAdapter来满足这种需求。其中的实现类似如下：

```
    // inboud事件默认处理过程
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelRegistered();    // 事件传播到下一个Handler
    }
    
    // outboud事件默认处理过程
    public void bind(ChannelHandlerContext ctx, SocketAddress localAddress,
            ChannelPromise promise) throws Exception {
        ctx.bind(localAddress, promise);  // 事件传播到下一个Handler
    }
   
```

在Adapter中，事件默认自动传播到下一个Handler，这样带来的另一个好处是：用户的Handler类可以继承Adapter且覆盖自己感兴趣的事件实现，其他事件使用默认实现，不用再实现ChannelIn/outboudHandler接口中所有方法，提高效率。
我们常常遇到这样的需求：在一个业务逻辑处理器中，需要写数据库、进行网络连接等耗时业务。Netty的原则是不阻塞I/O线程，所以需指定Handler执行的线程池，可使用如下代码：

```
 static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);
    ...
    ChannelPipeline pipeline = ch.pipeline();
    // 简单非阻塞业务，可以使用I/O线程执行
    pipeline.addLast("decoder", new MyProtocolDecoder());
    pipeline.addLast("encoder", new MyProtocolEncoder());
    // 复杂耗时业务，使用新的线程池
    pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```
ChannelHandler中有一个Sharable注解，使用该注解后多个ChannelPipeline中的Handler对象实例只有一个，从而减少Handler对象实例的创建。代码示例如下：

```
    public class DataServerInitializer extends ChannelInitializer<Channel> {
       private static final DataServerHandler SHARED = new DataServerHandler();
  
       @Override
       public void initChannel(Channel channel) {
           channel.pipeline().addLast("handler", SHARED);
       }
   }
```

Sharable注解的使用是有限制的，多个ChannelPipeline只有一个实例，所以该Handler要求无状态。上述示例中，DataServerHandler的事件处理方法中，不能使用或改变本身的私有变量，因为ChannelHandler是非线程安全的，使用私有变量会造成线程竞争而产生错误结果。

##  ChannelHandlerContext

Context指上下文关系，ChannelHandler的Context指的是ChannleHandler之间的关系以及ChannelHandler与ChannelPipeline之间的关系。ChannelPipeline中的事件传播主要依赖于ChannelHandlerContext实现，由于ChannelHandlerContext中有ChannelHandler之间的关系，所以能得到ChannelHandler的后继节点，从而将事件传播到下一个ChannelHandler。

ChannelHandlerContext继承自AttributeMap，所以提供了attr()方法设置和删除一些状态属性值，用户可将业务逻辑中所需使用的状态属性值存入到Context中。此外，Channel也继承自AttributeMap，也有attr()方法，在Netty4.0中，这两个attr()方法并不等效，这会给用户程序员带来困惑并且增加内存开销，所以Netty4.1中将channel.attr()==ctx.attr()。在使用Netty4.0时，建议只使用channel.attr()防止引起不必要的困惑。

一个Channel对应一个ChannelPipeline，一个ChannelHandlerContext对应一个ChannelHandler，但一个ChannelHandler可以对应多个ChannelHandlerContext。当一个ChannelHandler使用Sharable注解修饰且添加同一个实例对象到不用的Channel时，只有一个ChannelHandler实例对象，但每个Channel中都有一个ChannelHandlerContext对象实例与之对应。


