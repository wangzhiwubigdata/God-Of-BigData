## TimeoutHandler

在开发TCP服务时，一个常见的需求便是使用心跳保活客户端。而Netty自带的三个超时处理器IdleStateHandler，ReadTimeoutHandler和WriteTimeoutHandler可完美满足此需求。其中IdleStateHandler可处理读超时（客户端长时间没有发送数据给服务端）、写超时（服务端长时间没有发送数据到客户端）和读写超时（客户端与服务端长时间无数据交互）三种情况。这三种情况的枚举为：

```
public enum IdleState {
        READER_IDLE,    // 读超时
        WRITER_IDLE,    // 写超时
        ALL_IDLE    // 数据交互超时
    }
```

以IdleStateHandler的读超时事件为例进行分析，首先看类签名：

```
 public class IdleStateHandler extends ChannelDuplexHandler
```

注意到此Handler没有Sharable注解，这是因为每个连接的超时时间是特有的即每个连接有独立的状态，所以不能标注Sharable注解。继承自ChannelDuplexHandler是因为既要处理读超时又要处理写超时。
该类的一个典型构造方法如下：


```
    public IdleStateHandler(int readerIdleTimeSeconds, int writerIdleTimeSeconds, 
                int allIdleTimeSeconds) {
        this(readerIdleTimeSeconds, writerIdleTimeSeconds,  
                allIdleTimeSeconds, TimeUnit.SECONDS);
    }
```

分别设定各个超时事件的时间阈值。以读超时事件为例，有以下相关的字段：

```
// 用户配置的读超时时间
    private final long readerIdleTimeNanos;
    // 判定超时的调度任务Future
    private ScheduledFuture<?> readerIdleTimeout;
    // 最近一次读取数据的时间
    private long lastReadTime;
    // 是否第一次读超时事件
    private boolean firstReaderIdleEvent = true;
    // 状态，0 - 无关， 1 - 初始化完成 2 - 已被销毁
    private byte state; 
    // 是否正在读取
    private boolean reading;
```

首先看初始化方法initialize()：

```
    private void initialize(ChannelHandlerContext ctx) {
        switch (state) {
        case 1: // 初始化进行中或者已完成
        case 2: // 销毁进行中或者已完成
            return;
        }
        
        state = 1;
        lastReadTime = ticksInNanos();
        if (readerIdleTimeNanos > 0) {
            readerIdleTimeout = schedule(ctx, new ReaderIdleTimeoutTask(ctx),
                    readerIdleTimeNanos, TimeUnit.NANOSECONDS);
        }
```

初始化的工作较为简单，设定最近一次读取时间lastReadTime为当前系统时间，然后在用户设置的读超时时间readerIdleTimeNanos截止时，执行一个ReaderIdleTimeoutTask进行检测。其中使用的方法很简洁，如下：

```
     long ticksInNanos() {
        return System.nanoTime();
    }
    
    ScheduledFuture<?> schedule(ChannelHandlerContext ctx, Runnable task, 
              long delay, TimeUnit unit) {
        return ctx.executor().schedule(task, delay, unit);
    }
```

然后，分析销毁方法destroy()：

```
private void destroy() {
        state = 2;  // 这里结合initialize对比理解
        if (readerIdleTimeout != null) {
            // 取消调度任务，并置null
            readerIdleTimeout.cancel(false);
            readerIdleTimeout = null;
        }
    }
```

可知销毁的处理也很简单，分析完初始化和销毁，再看这两个方法被调用的地方，initialize()在三个方法中被调用：

```
public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        if (ctx.channel().isActive() &&
                ctx.channel().isRegistered()) {
            initialize(ctx);
        } 
    }
    
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        if (ctx.channel().isActive()) {
            initialize(ctx);
        }
        super.channelRegistered(ctx);
    }
    
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        initialize(ctx);
        super.channelActive(ctx);
    }
```

当客户端与服务端成功建立连接后，Channel被激活，此时channelActive的初始化被调用；如果Channel被激活后，动态添加此Handler，则handlerAdded的初始化被调用；如果Channel被激活，用户主动切换Channel的执行线程Executor，则channelRegistered的初始化被调用。这一部分较难理解，请仔细体会。destroy()则有两处调用：


```
 public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        destroy();
        super.channelInactive(ctx);
    }
    
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        destroy();
    }
```

即该Handler被动态删除时，handlerRemoved的销毁被执行；Channel失效时，channelInactive的销毁被执行。
分析完这些，在分析核心的调度任务ReaderIdleTimeoutTask：

```
private final class ReaderIdleTimeoutTask implements Runnable {
        
        private final ChannelHandlerContext ctx;
        
        ReaderIdleTimeoutTask(ChannelHandlerContext ctx) {
            this.ctx = ctx;
        }

        @Override
        protected void run() {
            if (!ctx.channel().isOpen()) {
                // Channel不再有效
                return;
            }
            
            long nextDelay = readerIdleTimeNanos;
            if (!reading) {
                // nextDelay<=0 说明在设置的超时时间内没有读取数据
                nextDelay -= ticksInNanos() - lastReadTime;
            }
            // 隐含正在读取时，nextDelay = readerIdleTimeNanos > 0

            if (nextDelay <= 0) {
                // 超时时间已到，则再次调度该任务本身
                readerIdleTimeout = schedule(ctx, this, readerIdleTimeNanos, 
                    TimeUnit.NANOSECONDS);

                boolean first = firstReaderIdleEvent;
                firstReaderIdleEvent = false;

                try {
                    IdleStateEvent event =
                        newIdleStateEvent(IdleState.READER_IDLE, first);
                    channelIdle(ctx, event); // 模板方法处理
                } catch (Throwable t) {
                    ctx.fireExceptionCaught(t);
                }
            } else {
                // 注意此处的nextDelay值，会跟随lastReadTime刷新
                readerIdleTimeout = schedule(ctx, this, nextDelay, TimeUnit.NANOSECONDS);
            }
        }
    }
```
这个读超时检测任务执行的过程中又递归调用了它本身进行下一次调度，请仔细品味该种使用方法。再列出channelIdle()的代码：

```
 protected void channelIdle(ChannelHandlerContext ctx, IdleStateEvent evt) 
                  throws Exception {
        ctx.fireUserEventTriggered(evt);
    }
```

本例中，该方法将写超时事件作为用户事件传播到下一个Handler，用户需要在某个Handler中拦截该事件进行处理。该方法标记为protect说明子类通常可覆盖，ReadTimeoutHandler子类即定义了自己的处理：

```
@Override
    protected final void channelIdle(ChannelHandlerContext ctx, IdleStateEvent evt)
                   throws Exception {
        assert evt.state() == IdleState.READER_IDLE;
        readTimedOut(ctx);
    }

    protected void readTimedOut(ChannelHandlerContext ctx) throws Exception {
        if (!closed) {
            ctx.fireExceptionCaught(ReadTimeoutException.INSTANCE);
            ctx.close();
            closed = true;
        }
    }
```

可知在ReadTimeoutHandler中，如果发生读超时事件，将会关闭该Channel。当进行心跳处理时，使用IdleStateHandler较为麻烦，一个简便的方法是：直接继承ReadTimeoutHandler然后覆盖readTimedOut()进行用户所需的超时处理。