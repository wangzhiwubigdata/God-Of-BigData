编解码处理器作为Netty编程时必备的ChannelHandler，每个应用都必不可少。Netty作为网络应用框架，在网络上的各个应用之间不断进行数据交互。而网络数据交换的基本单位是字节，所以需要将本应用的POJO对象编码为字节数据发送到其他应用，或者将收到的其他应用的字节数据解码为本应用可使用的POJO对象。这一部分，又和JAVA中的序列化和反序列化对应。幸运的是，有很多其他的开源工具（protobuf，thrift，json，xml等等）可方便的处理POJO对象的序列化，可参见这个链接。
在互联网中，Netty使用TCP/UDP协议传输数据。由于Netty基于异步事件处理以及TCP的一些特性，使得TCP数据包会发生粘包现象。想象这样的情况，客户端与服务端建立连接后，连接发送了两条消息：

    +------+   +------+
    | MSG1 |   | MSG2 |
    +------+   +------+
在互联网上传输数据时，连续发送的两条消息，在服务端极有可能被合并为一条：

     +------------+
    | MSG1  MSG2 |
    +------------+

这还不是最坏的情况，由于路由器的拆包和重组，可能收到这样的两个数据包：

     +----+     +---------+         +-------+    +-----+ 
        | MS |     |  G1MSG2 |  或者  | MSG1M |    | SG2 | 
        +----+     +---------+        +-------+    +-----+



而服务端要正确的识别出这样的两条消息，就需要编码器的正确工作。为了正确的识别出消息，业界有以下几种做法：

使用定界符分割消息，一个特例是使用换行符分隔每条消息。
使用定长的消息。
在消息的某些字段指明消息长度。

明白了这些，进入正题，分析Netty的编码框架ByteToMessageDecoder。

## ByteToMessageDecoder

在分析之前，需要说明一点：ByteToMessage容易引起误解，解码结果Message会被认为是JAVA对象POJO，但实际解码结果是消息帧。也就是说该解码器处理TCP的粘包现象，将网络发送的字节流解码为具有确定含义的消息帧，之后的解码器再将消息帧解码为实际的POJO对象。
明白了这点，再次回顾两条消息发送的最坏情况，可知要正确取得两条消息，需要一个内存区域存储消息，当收到MS时继续等待第二个包G1MSG2到达再进行解码操作。在ByteToMessageDecoder中，这个内存区域被抽象为Cumulator，直译累积器，可自动扩容累积字节数据，Netty将其定义为一个接口：

```
    public interface Cumulator {
        ByteBuf cumulate(ByteBufAllocator alloc, ByteBuf cumulation, ByteBuf in);
    }
```

其中，两个ByteBuf参数cumulation指已经累积的字节数据，in表示该次channelRead()读取到的新数据。返回ByteBuf为累积数据后的新累积区（必要时候自动扩容）。自动扩容的代码如下：

```
 static ByteBuf expandCumulation(ByteBufAllocator alloc, ByteBuf cumulation, 
                                       int newReadBytes) {
        ByteBuf oldCumulation = cumulation;
        // 扩容后新的缓冲区
        cumulation = alloc.buffer(oldCumulation.readableBytes() + readable);
        cumulation.writeBytes(oldCumulation);
        // 旧的缓冲区释放
        oldCumulation.release();
        return cumulation;
    }
```

自动扩容的方法简单粗暴，直接使用大容量的Bytebuf替换旧的ByteBuf。Netty定义了两个累积器，一个为MERGE_CUMULATOR：

```
public static final Cumulator MERGE_CUMULATOR = new Cumulator() {
        @Override
        public ByteBuf cumulate(ByteBufAllocator alloc, ByteBuf cumulation, ByteBuf in) {
            ByteBuf buffer;
            // 1.累积区容量不够容纳数据
            // 2.用户使用了slice().retain()或duplicate().retain()使refCnt增加
            if (cumulation.writerIndex() > cumulation.maxCapacity() - in.readableBytes()
                    || cumulation.refCnt() > 1) {
                buffer = expandCumulation(alloc, cumulation, in.readableBytes());
            } else {
                buffer = cumulation;
            }
            buffer.writeBytes(in);
            in.release();
            return buffer;
        }
    };
```
可知，两种情况下会扩容：

1. 累积区容量不够容纳新读入的数据
2. 用户使用了slice().retain()或duplicate().retain()使refCnt增加并且大于1，此时扩容返回一个新的累积区ByteBuf，方便用户对老的累积区ByteBuf进行后续处理。

另一个累积器为COMPOSITE_CUMULATOR：

```
public static final Cumulator COMPOSITE_CUMULATOR = new Cumulator() {
        @Override
        public ByteBuf cumulate(ByteBufAllocator alloc, ByteBuf cumulation, ByteBuf in) {
            ByteBuf buffer;
            if (cumulation.refCnt() > 1) {
                buffer = expandCumulation(alloc, cumulation, in.readableBytes());
                buffer.writeBytes(in);
                in.release();
            } else {
                CompositeByteBuf composite;
                if (cumulation instanceof CompositeByteBuf) {
                    composite = (CompositeByteBuf) cumulation;
                } else {
                    composite = alloc.compositeBuffer(Integer.MAX_VALUE);
                    composite.addComponent(true, cumulation);
                }
                composite.addComponent(true, in);
                buffer = composite;
            }
            return buffer;
        }
    };
```

这个累积器只在第二种情况refCnt>1时扩容，除此之外处理和MERGE_CUMULATOR一致，不同的是当cumulation不是CompositeByteBuf时会创建新的同类CompositeByteBuf，这样最后返回的ByteBuf必定是CompositeByteBuf。使用这个累积器后，当容量不够时并不会进行内存复制，只会讲新读入的in加到CompositeByteBuf中。需要注意的是：此种情况下虽然不需内存复制，却要求用户维护复杂的索引，在某些使用中可能慢于MERGE_CUMULATOR。故Netty默认使用MERGE_CUMULATOR累积器。
累积器分析完毕，步入正题ByteToMessageDecoder，首先看类签名：

```
public abstract class ByteToMessageDecoder extends
                                ChannelInboundHandlerAdapter
```

该类是一个抽象类，其中的抽象方法只有一个decode()：

```
protected abstract void decode(ChannelHandlerContext ctx, ByteBuf in, 
List<Object> out) throws Exception;
```

用户使用了该解码框架后，只需实现该方法就可定义自己的解码器。参数in表示累积器已累积的数据，out表示本次可从累积数据解码出的结果列表，结果可为POJO对象或者ByteBuf等等Object。
关注一下成员变量，以便更好的分析：

```
    ByteBuf cumulation; // 累积区
    private Cumulator cumulator = MERGE_CUMULATOR; // 累积器
    // 设置为true后每个channelRead事件只解码出一个结果
    private boolean singleDecode;   // 某些特殊协议使用
    private boolean decodeWasNull;  // 解码结果为空
    private boolean first;  // 是否首个消息
    // 累积区不丢弃字节的最大次数，16次后开始丢弃
    private int discardAfterReads = 16;
    private int numReads;   // 累积区不丢弃字节的channelRead次数
```
下面，直接进入channelRead()事件处理：

```
 public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 只对ByteBuf处理即只对字节数据进行处理
        if (msg instanceof ByteBuf) {
            // 解码结果列表
            CodecOutputList out = CodecOutputList.newInstance();
            try {
                ByteBuf data = (ByteBuf) msg;
                first = cumulation == null; // 累积区为空表示首次解码
                if (first) {
                    // 首次解码直接使用读入的ByteBuf作为累积区
                    cumulation = data;
                } else {
                    // 非首次需要进行字节数据累积
                    cumulation = cumulator.cumulate(ctx.alloc(), cumulation, data);
                }
                callDecode(ctx, cumulation, out); // 解码操作
            } catch (DecoderException e) {
                throw e;
            } catch (Throwable t) {
                throw new DecoderException(t);
            } finally {
                if (cumulation != null && !cumulation.isReadable()) {
                    // 此时累积区不再有字节数据，已被处理完毕
                    numReads = 0;
                    cumulation.release();
                    cumulation = null;
                } else if (++ numReads >= discardAfterReads) {
                    // 连续discardAfterReads次后
                    // 累积区还有字节数据，此时丢弃一部分数据
                    numReads = 0;
                    discardSomeReadBytes(); // 丢弃一些已读字节
                }

                int size = out.size();
                // 本次没有解码出数据，此时size=0
                decodeWasNull = !out.insertSinceRecycled();
                fireChannelRead(ctx, out, size); // 触发事件
                out.recycle();  // 回收解码结果
            }
        } else {
            ctx.fireChannelRead(msg);
        }
    }
```

解码结果列表CodecOutputList是Netty定制的一个特殊列表，该列表在线程中被缓存，可循环使用来存储解码结果，减少不必要的列表实例创建，从而提升性能。由于解码结果需要频繁存储，普通的ArrayList难以满足该需求，故定制化了一个特殊列表，由此可见Netty对优化的极致追求。
注意finally块的第一个if情况满足时，即累积区的数据已被读取完毕，请考虑释放累积区的必要性。想象这样的情况，当一条消息被解码完毕后，如果客户端长时间不发送消息，那么，服务端保存该条消息的累积区将一直占据服务端内存浪费资源，所以必须释放该累积区。
第二个if情况满足时，即累积区的数据一直在channelRead读取数据进行累积和解码，直到达到了discardAfterReads次（默认16），此时累积区依然还有数据。在这样的情况下，Netty主动丢弃一些字节，这是为了防止该累积区占用大量内存甚至耗尽内存引发OOM。
处理完这些情况后，最后统一触发ChannelRead事件，将解码出的数据传递给下一个处理器。注意：当out=0时，统一到一起被处理了。
再看细节的discardSomeReadBytes()和fireChannelRead()：


```
 protected final void discardSomeReadBytes() {
        if (cumulation != null && !first && cumulation.refCnt() == 1) {
            cumulation.discardSomeReadBytes();
        }
    }
    
    static void fireChannelRead(ChannelHandlerContext ctx, CodecOutputList msgs, 
                        int numElements) {
        for (int i = 0; i < numElements; i ++) {
            ctx.fireChannelRead(msgs.getUnsafe(i));
        }
    }
```

代码比较简单，只需注意discardSomeReadBytes中，累积区的refCnt() == 1时才丢弃数据是因为：如果用户使用了slice().retain()和duplicate().retain()使refCnt>1，表明该累积区还在被用户使用，丢弃数据可能导致用户的困惑，所以须确定用户不再使用该累积区的已读数据，此时才丢弃。
下面分析解码核心方法callDecode()：

```
 protected void callDecode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        try {
            while (in.isReadable()) {
                int outSize = out.size();

                if (outSize > 0) {
                    // 解码出消息就立即处理，防止消息等待
                    fireChannelRead(ctx, out, outSize);
                    out.clear();
                    
                    // 用户主动删除该Handler，继续操作in是不安全的
                    if (ctx.isRemoved()) {
                        break;
                    }
                    outSize = 0;
                }

                int oldInputLength = in.readableBytes();
                decode(ctx, in, out);   // 子类需要实现的具体解码步骤

                // 用户主动删除该Handler，继续操作in是不安全的
                if (ctx.isRemoved()) {
                    break; 
                }
                
                // 此时outSize都==0（这的代码容易产生误解 应该直接使用0）
                if (outSize == out.size()) {
                    if (oldInputLength == in.readableBytes()) {
                        // 没有解码出消息，且没读取任何in数据
                        break;
                    } else {
                        // 读取了一部份数据但没有解码出消息
                        // 说明需要更多的数据，故继续
                        continue;
                    }
                }

                // 运行到这里outSize>0 说明已经解码出消息
                if (oldInputLength == in.readableBytes()) {
                    // 解码出消息但是in的读索引不变，用户的decode方法有Bug
                    throw new DecoderException(
                            "did not read anything but decoded a message.");
                }
                
                // 用户设定一个channelRead事件只解码一次
                if (isSingleDecode()) {
                    break; 
                }
            }
        } catch (DecoderException e) {
            throw e;
        } catch (Throwable cause) {
            throw new DecoderException(cause);
        }
    }
```
循环中的第一个if分支，检查解码结果，如果已经解码出消息则立即将消息传播到下一个处理器进行处理，这样可使消息得到及时处理。在调用decode()方法的前后，都检查该Handler是否被用户从ChannelPipeline中删除，如果删除则跳出解码步骤不对输入缓冲区in进行操作，因为继续操作in已经不安全。解码完成后，对in解码前后的读索引进行了检查，防止用户的错误使用，如果用户错误使用将抛出异常。
至此，核心的解码框架已经分析完毕，再看最后的一些边角处理。首先是channelReadComplete()读事件完成后的处理：

```
public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        numReads = 0;   // 连续读次数置0
        discardSomeReadBytes(); // 丢弃已读数据，节约内存
        if (decodeWasNull) {
            // 没有解码出结果，则期待更多数据读入
            decodeWasNull = false;
            if (!ctx.channel().config().isAutoRead()) {
                ctx.read();
            }
        }
        ctx.fireChannelReadComplete();
    }

```

如果channelRead()中没有解码出消息，极有可能是数据不够，由此调用ctx.read()期待读入更多的数据。如果设置了自动读取，将会在HeadHandler中调用ctx.read()；没有设置自动读取，则需要此处显式调用。
最后再看Handler从ChannelPipelien中移除的处理handlerRemoved():

```
 public final void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        ByteBuf buf = cumulation;
        if (buf != null) {
            cumulation = null;  // 释放累积区，GC回收

            int readable = buf.readableBytes();
            if (readable > 0) {
                ByteBuf bytes = buf.readBytes(readable);
                buf.release();
                // 解码器已被删除故不再解码，只将数据传播到下一个Handler
                ctx.fireChannelRead(bytes);
            } else {
                buf.release();
            }

            numReads = 0;   // 置0，有可能被再次添加
            ctx.fireChannelReadComplete();
        }
        handlerRemoved0(ctx);   // 用户可进行的自定义处理
    }
```

当解码器被删除时，如果还有没被解码的数据，则将数据传播到下一个处理器处理，防止丢失数据。此外，当连接不再有效触发channelInactive事件或者触发ChannelInputShutdownEvent时，则会调用callDecode()解码，如果解码出消息，传播到下一个处理器。这部分的代码不再列出。
至此，ByteToMessageDecoder解码框架已分析完毕，下面，我们选用具体的实例进行分析。

### LineBasedFrameDecoder

基于行分隔的解码器LineBasedFrameDecoder是一个特殊的分隔符解码器，该解码器使用的分隔符为：windows的\r\n和类linux的\n。
首先看该类定义的成员变量：
```
    // 最大帧长度，超过此长度将抛出异常TooLongFrameException
    private final int maxLength;
    // 是否快速失败，true-检测到帧长度过长立即抛出异常不在读取整个帧
    // false-检测到帧长度过长依然读完整个帧再抛出异常
    private final boolean failFast;
    // 是否略过分隔符，true-解码结果不含分隔符
    private final boolean stripDelimiter;

    // 超过最大帧长度是否丢弃字节
    private boolean discarding;
    private int discardedBytes; // 丢弃的字节数
```
其中，前三个变量可由用户根据实际情况配置，后两个变量解码时使用。
该子类覆盖的解码方法如下：

```
protected final void decode(ChannelHandlerContext ctx, ByteBuf in, 
                   List<Object> out) throws Exception {
        Object decoded = decode(ctx, in);
        if (decoded != null) {
            out.add(decoded);
        }
    }
```
其中又定义了decode(ctx, in)解码出单个消息帧，事实上这也是其他编码子类使用的方法。decode(ctx, in)方法处理很绕弯，只给出伪代码：

```
protected Object decode(ChannelHandlerContext ctx, ByteBuf buffer) throws Exception {
        final int eol = findEndOfLine(buffer);
        if (!discarding) {
            if (eol >= 0) {
                // 此时已找到换行符
                if(!checkMaxLength()) {
                    return getFrame().retain();
                } 
                // 超过最大长度抛出异常
            } else {
                if (checkMaxLength()) {
                    // 设置true表示下一次解码需要丢弃字节
                    discarding = true;  
                    if (failFast) {
                        // 抛出异常
                    }
                } 
            }
        } else {
            if (eol >= 0) {
                // 丢弃换行符以及之前的字节
                buffer.readerIndex(eol + delimLength);
            } else {
                // 丢弃收到的所有字节
                buffer.readerIndex(buffer.writerIndex());
            }
        }
    }
```

该方法需要结合解码框架的while循环反复理解，每个if情况都是一次while循环，而变量discarding就成为控制每次解码流程的状态量，注意其中的状态转移。(想法：使用状态机实现，则流程更清晰)

### DelimiterBasedFrameDecoder

该解码器是更通用的分隔符解码器，可支持多个分隔符，每个分隔符可为一个或多个字符。如果定义了多个分隔符，并且可解码出多个消息帧，则选择产生最小帧长的结果。例如，使用行分隔符\r\n和\n分隔：

        +--------------+
        | ABC\nDEF\r\n |
        +--------------+
        
可有两种结果：

    +-----+-----+              +----------+   
    | ABC | DEF |  (√)   和    | ABC\nDEF |  (×)
    +-----+-----+              +----------+
    
    
该编码器可配置的变量与LineBasedFrameDecoder类似，只是多了一个ByteBuf[] delimiters用于配置具体的分隔符。
Netty在Delimiters类中定义了两种默认的分隔符，分别是NULL分隔符和行分隔符：

```
  public static ByteBuf[] nulDelimiter() {
        return new ByteBuf[] {
                Unpooled.wrappedBuffer(new byte[] { 0 }) };
    }
    
    public static ByteBuf[] lineDelimiter() {
        return new ByteBuf[] {
                Unpooled.wrappedBuffer(new byte[] { '\r', '\n' }),
                Unpooled.wrappedBuffer(new byte[] { '\n' }),
        };
    }
```

### FixedLengthFrameDecoder

该解码器十分简单，按照固定长度frameLength解码出消息帧。如下的数据帧解码为固定长度3的消息帧示例如下：

    +---+----+------+----+      +-----+-----+-----+
    | A | BC | DEFG | HI |  ->  | ABC | DEF | GHI |
    +---+----+------+----+      +-----+-----+-----+
    

其中的解码方法也十分简单：
    
```
 protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        if (in.readableBytes() < frameLength) {
            return null;
        } else {
            return in.readSlice(frameLength).retain();
        }
    }
```

### LengthFieldBasedFrameDecoder

基于长度字段的消息帧解码器，该解码器可根据数据包中的长度字段动态的解码出消息帧。一个推荐的二进制传输协议可设计为如下格式：

    +----------+------+----------+------+
    |  头部长度 |  头部 |  数据长度 | 数据 |
    +----------+------+----------+------+
    

这样的协议可满足大多数场景使用，但不幸的是：很多情况下并不可以设计新的协议，往往要在老旧的协议上传输数据。由此，Netty将该解码器设计的十分通用，只要有类似的长度字段便能正确解码出消息帧。当然前提是：正确使用解码器。
没有什么是完美的，由于该解码器十分通用，所以有大量的配置变量：

```
    private final ByteOrder byteOrder;
    private final int maxFrameLength;
    private final boolean failFast;
    private final int lengthFieldOffset;
    private final int lengthFieldLength;
    private final int lengthAdjustment;
    private final int initialBytesToStrip;
```

变量byteOrder表示长度字段的字节序：大端或小端，默认为大端。如果对字节序有疑问，请查阅其他资料，不再赘述。maxFrameLength和failFast与其他解码器相同，控制最大帧长度和快速失败抛异常，注意：该解码器failFast默认为true。
接下来将重点介绍其它四个变量：


* lengthFieldOffset表示长度字段偏移量即在一个数据包中长度字段的具体下标位置。标准情况，该长度字段为数据部分长度。

* lengthFieldLength表示长度字段的具体字节数，如一个int占4字节。该解码器支持的字节数有：1，2，3，4和8，其他则会抛出异常。另外，还需要注意的是：长度字段的结果为无符号数。

* lengthAdjustment是一个长度调节量，当数据包的长度字段不是数据部分长度而是总长度时，可将此值设定为头部长度，便能正确解码出包含整个数据包的结果消息帧。注意：某些情况下，该值可设定为负数。

* initialBytesToStrip表示需要略过的字节数，如果我们只关心数据部分而不关心头部，可将此值设定为头部长度从而丢弃头部。
下面我们使用具体的例子来说明：



* 需求1：如下待解码数据包，正确解码为消息帧，其中长度字段在最前面的2字节，数据部分为12字节的字符串"HELLO, WORLD"，长度字段0x000C=12 表示数据部分长度，数据包总长度则为14字节。


         解码前(14 bytes)                 解码后(14 bytes)
         +--------+----------------+      +--------+----------------+
         | Length | Actual Content |----->| Length | Actual Content |
         | 0x000C | "HELLO, WORLD" |      | 0x000C | "HELLO, WORLD" |
         +--------+----------------+      +--------+----------------+
         
正确配置（只列出四个值中不为0的值）：

```
 lengthFieldLength = 2;
```

* 需求2：需求1的数据包不变，消息帧中去除长度字段。

        解码前(14 bytes)                 解码后(12 bytes)
        +--------+----------------+      +----------------+
        | Length | Actual Content |----->| Actual Content |
        | 0x000C | "HELLO, WORLD" |      | "HELLO, WORLD" |
        +--------+----------------+      +----------------+
        
正确配置：
```
 lengthFieldLength   = 2;
    initialBytesToStrip = 2;
```

需求3：需求1数据包中长度字段表示数据包总长度。

    解码前(14 bytes)                 解码后(14 bytes)
        +--------+----------------+      +--------+----------------+
        | Length | Actual Content |----->| Length | Actual Content |
        | 0x000E | "HELLO, WORLD" |      | 0x000E | "HELLO, WORLD" |
        +--------+----------------+      +--------+----------------+
        

正确配置：

```
    lengthFieldLength =  2;
    lengthAdjustment  = -2;  // 调整长度字段的2字节
```

需求4：综合难度，数据包有两个头部HDR1和HDR2，长度字段以及数据部分组成，其中长度字段值表示数据包总长度。结果消息帧需要第二个头部HDR2和数据部分。请先给出答案再与标准答案比较，结果正确说明你已完全掌握了该解码器的使用。

    解码前 (16 bytes)                               解码后 (13 bytes)
    +------+--------+------+----------------+      +------+----------------+
    | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
    | 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
    +------+--------+------+----------------+      +------+----------------+
    
    

正确配置：

```
 lengthFieldOffset   =  1;
    lengthFieldLength   =  2;
    lengthAdjustment    = -3;
    initialBytesToStrip =  3;
```

本解码器的解码过程总体上较为复杂，由于解码的代码是在while循环里面，decode方法return或者抛出异常时可看做一次循环结束，直到in中数据被解析完或者in的readerIndex读索引不再增加才会从while循环跳出。使用状态的思路理解，每个return或者抛出异常看为一个状态：


状态1：丢弃过长帧状态，可能是用户设置了错误的帧长度或者实际帧过长。

```
 if (discardingTooLongFrame) {
        long bytesToDiscard = this.bytesToDiscard;
        int localBytesToDiscard = (int) Math.min(bytesToDiscard, in.readableBytes());
        in.skipBytes(localBytesToDiscard); // 丢弃实际的字节数
        
        bytesToDiscard -= localBytesToDiscard;
        this.bytesToDiscard = bytesToDiscard;
        failIfNecessary(false);
    }
```

变量localBytesToDiscard取得实际需要丢弃的字节数，由于过长帧有两种情况：a.用户设置了错误的长度字段，此时in中并没有如此多的字节；b.in中确实有如此长度的帧，这个帧确实超过了设定的最大长度。bytesToDiscard的计算是为了failIfNecessary()确定异常的抛出，其值为0表示当次丢弃状态已经丢弃了in中的所有数据，可以对新读入in的数据进行处理；否则，还处于异常状态。

```
private void failIfNecessary(boolean firstDetectionOfTooLongFrame) {
        if (bytesToDiscard == 0) {
            long tooLongFrameLength = this.tooLongFrameLength;
            this.tooLongFrameLength = 0;
            // 由于已经丢弃所有数据，关闭丢弃模式
            discardingTooLongFrame = false;
            // 已经丢弃了所有字节，当非快速失败模式抛异常
            if (!failFast || firstDetectionOfTooLongFrame) {
                fail(tooLongFrameLength);
            }
        } else {
            if (failFast && firstDetectionOfTooLongFrame) {
                // 帧长度异常，快速失败模式检测到即抛异常
                fail(tooLongFrameLength);
            }
        }
    }

```

可见，首次检测到帧长度是一种特殊情况，在之后的一个状态进行分析。请注意该状态并不是都抛异常，还有可能进入状态2。

状态2：in中数据不足够组成消息帧，此时直接返回null等待更多数据到达。

```
    if (in.readableBytes() < lengthFieldEndOffset) {
        return null;
    }
```

状态3：帧长度错误检测，检测长度字段为负值得帧以及加入调整长度后总长小于长度字段的帧，均抛出异常。

```
 int actualLengthFieldOffset = in.readerIndex() + lengthFieldOffset;
    // 该方法取出长度字段的值，不再深入分析
    long frameLength = getUnadjustedFrameLength(in, actualLengthFieldOffset, 
                             lengthFieldLength, byteOrder);
    if (frameLength < 0) {
        in.skipBytes(lengthFieldEndOffset);
        throw new CorruptedFrameException("...");
    }

    frameLength += lengthAdjustment + lengthFieldEndOffset;
    if (frameLength < lengthFieldEndOffset) {
        in.skipBytes(lengthFieldEndOffset);
        throw new CorruptedFrameException("...");
```

状态4：帧过长，由前述可知：可能是用户设置了错误的帧长度或者实际帧过长

```
    if (frameLength > maxFrameLength) {
            long discard = frameLength - in.readableBytes();
            tooLongFrameLength = frameLength;

            if (discard < 0) {
                in.skipBytes((int) frameLength);
            } else {
                discardingTooLongFrame = true;
                bytesToDiscard = discard;
                in.skipBytes(in.readableBytes());
            }
            failIfNecessary(true);
            return null;
        }
```

变量discard<0表示当前收到的数据足以确定是实际的帧过长，所以直接丢弃过长的帧长度；>0表示当前in中的数据并不足以确定是用户设置了错误的帧长度，还是正确帧的后续数据字节还没有到达，但无论何种情况，将丢弃状态discardingTooLongFrame标记设置为true，之后后续数据字节进入状态1处理。==0时，在failIfNecessary(true)无论如何都将抛出异常，><0时，只有设置快速失败才会抛出异常。还需注意一点：failIfNecessary()的参数firstDetectionOfTooLongFrame的首次是指正确解析数据后发生的第一次发生的帧过长，可知会有很多首次。


状态5：正确解码出消息帧。

```
 int frameLengthInt = (int) frameLength;
    if (in.readableBytes() < frameLengthInt) {
        return null;    // 到达的数据还达不到帧长
    }

    if (initialBytesToStrip > frameLengthInt) {
        in.skipBytes(frameLengthInt);   // 跳过字节数错误
        throw new CorruptedFrameException("...");
    }
    in.skipBytes(initialBytesToStrip);

    // 正确解码出数据帧
    int readerIndex = in.readerIndex();
    int actualFrameLength = frameLengthInt - initialBytesToStrip;
    ByteBuf frame = in.slice(readerIndex, actualFrameLength).retain();
    in.readerIndex(readerIndex + actualFrameLength);
    return frame;
```

代码中混合了两个简单状态，到达的数据还达不到帧长和用户设置的忽略字节数错误。由于较为简单，故合并到一起。
至此解码框架分析完毕。可见，要正确的写出基于长度字段的解码器还是较为复杂的，如果开发时确有需求，特别要注意状态的转移。下面介绍较为简单的编码框架。

