
MessageToByteEncoder框架可见用户使用POJO对象编码为字节数据存储到ByteBuf。用户只需定义自己的编码方法encode()即可。
首先看类签名：
```
    public abstract class MessageToByteEncoder<I> extends 
                                ChannelOutboundHandlerAdapter
```
可知该类只处理出站事件，切确的说是write事件。

该类有两个成员变量，preferDirect表示是否使用内核的DirectedByteBuf，默认为true。TypeParameterMatcher用于检测泛型参数是否是期待的类型，比如说，如果需要编码String类的POJO对象，Matcher会确保write()传入的参数Object的实际切确类型为String。
直接分析write()的处理：
```
    public void write(ChannelHandlerContext ctx, Object msg, 
                          ChannelPromise promise) throws Exception {
        ByteBuf buf = null;
        try {
            if (acceptOutboundMessage(msg)) {
                I cast = (I) msg;
                // 分配一个输出缓冲区
                buf = allocateBuffer(ctx, cast, preferDirect);
                try {
                    encode(ctx, cast, buf); // 用户定义的编码方法
                } finally {
                    ReferenceCountUtil.release(cast);
                }

                if (buf.isReadable()) {
                    ctx.write(buf, promise); // 确实写入了数据
                } else {
                    // 没有需要写的数据，也有可能是用户编码错误
                    buf.release();  
                    ctx.write(Unpooled.EMPTY_BUFFER, promise);
                }
                buf = null;
            } else {
                ctx.write(msg, promise);
            }
        } catch (EncoderException e) {
            throw e;
        } catch (Throwable e) {
            throw new EncoderException(e);
        } finally {
            if (buf != null) {
                buf.release();
            }
        }
    }
```
编码框架简单明了，再列出allocateBuffer()方法的代码：
```
    protected ByteBuf allocateBuffer(ChannelHandlerContext ctx,  I msg,
                               boolean preferDirect) throws Exception {
        if (preferDirect) {
            return ctx.alloc().ioBuffer();  // 内核直接缓存
        } else {
            return ctx.alloc().heapBuffer(); // JAVA队缓存
        }
    }
```
总的来说，编码的复杂度大大小于解码的复杂度，这是因为编码不需考虑TCP粘包。编解码的处理还有一个常用的类MessageToMessageCodec用于POJO对象之间的转换。如果有兴趣，可下载源码查看。至此，编解码框架已分析完毕。

