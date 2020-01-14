### **Java高级特性增强-NIO
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了部分勘误，感谢前辈的付出，每节文章末尾有引用列表~
* * *
**写在所有文字的前面**：作者在此特别推荐Google排名第一的关于NIO的文章：
http://tutorials.jenkov.com/java-nio/index.html
虽然是英文的，但是看下来并不困难。后面如果各位看官呼声很高，作者会翻译这一系列文章。


## Java NIO之Channel（通道）

#### Buffer(缓冲区)介绍


通常来说NIO中的所有IO都是从 Channel（通道） 开始的。

* 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
* 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

数据读取和写入操作图示：
![342194a2fdfeaf96e6051e08c9951de3](Java NIO之Channel(通道).resources/2958433B-EEAF-4D8B-98A2-39941C7C1733.png)

**Java NIO Channel通道和流非常相似，主要有以下几点区别：**

通道可以读也可以写，流一般来说是单向的（只能读或者写，所以之前我们用流进行IO操作的时候需要分别创建一个输入流和一个输出流）。
通道可以异步读写。
通道总是基于缓冲区Buffer来读写。

**Java NIO中最重要的几个Channel的实现：**

* FileChannel： 用于文件的数据读写
* DatagramChannel： 用于UDP的数据读写
* SocketChannel： 用于TCP的数据读写，一般是客户端实现
* ServerSocketChannel: 允许我们监听TCP链接请求，每个请求会创建会一个SocketChannel，一般是服务器实现

**类层次结构：**
下面的UML图使用Idea生成的。
![5153431ea4cfbf8d64f746d098f8bda5](Java NIO之Channel(通道).resources/3A2E73E4-2445-4B90-93F0-0EB34EB8C82B.png)


#### FileChannel的使用
使用FileChannel读取数据到Buffer（缓冲区）以及利用Buffer（缓冲区）写入数据到FileChannel：
```
package filechannel;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class FileChannelTxt {
    public static void main(String args[]) throws IOException {
        //1.创建一个RandomAccessFile（随机访问文件）对象，
        RandomAccessFile raf=new RandomAccessFile("D:\\niodata.txt", "rw");
        //通过RandomAccessFile对象的getChannel()方法。FileChannel是抽象类。
        FileChannel inChannel=raf.getChannel();
        //2.创建一个读数据缓冲区对象
        ByteBuffer buf=ByteBuffer.allocate(48);
        //3.从通道中读取数据
        int bytesRead = inChannel.read(buf);
        //创建一个写数据缓冲区对象
        ByteBuffer buf2=ByteBuffer.allocate(48);
        //写入数据
        buf2.put("filechannel test".getBytes());
        buf2.flip();
        inChannel.write(buf);
        while (bytesRead != -1) {

            System.out.println("Read " + bytesRead);
            //Buffer有两种模式，写模式和读模式。在写模式下调用flip()之后，Buffer从写模式变成读模式。
            buf.flip();
           //如果还有未读内容
            while (buf.hasRemaining()) {
                System.out.print((char) buf.get());
            }
            //清空缓存区
            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        //关闭RandomAccessFile（随机访问文件）对象
        raf.close();
    }
}

```
运行效果：
![93e3d051206ec5c22f1997fae7e3a143](Java NIO之Channel(通道).resources/0CC9E605-79FB-455E-AF3F-1CD41832B4A6.png)
通过上述实例代码，我们可以大概总结出FileChannel的一般使用规则：
>**1. 开启FileChannel**

使用之前，FileChannel必须被打开 ，但是你无法直接打开FileChannel（FileChannel是抽象类）。需要通过 InputStream ， OutputStream 或 RandomAccessFile 获取FileChannel。
我们上面的例子是通过RandomAccessFile打开FileChannel的：
```
//1.创建一个RandomAccessFile（随机访问文件）对象，
        RandomAccessFile raf=new RandomAccessFile("D:\\niodata.txt", "rw");
        //通过RandomAccessFile对象的getChannel()方法。FileChannel是抽象类。
        FileChannel inChannel=raf.getChannel();
```
>**2. 从FileChannel读取数据/写入数据**
从FileChannel中读取数据/写入数据之前首先要创建一个Buffer（缓冲区）对象，Buffer（缓冲区）对象的使用我们在上一篇文章中已经详细说明了，如果不了解的话可以看我的上一篇关于Buffer的文章。

使用FileChannel的read()方法读取数据：
```
//2.创建一个读数据缓冲区对象
  ByteBuffer buf=ByteBuffer.allocate(48);
//3.从通道中读取数据
  int bytesRead = inChannel.read(buf);
```
使用FileChannel的write()方法写入数据：
```
 //创建一个写数据缓冲区对象
   ByteBuffer buf2=ByteBuffer.allocate(48);
 //写入数据
   buf2.put("filechannel test".getBytes());
   buf2.flip();
   inChannel.write(buf);
```
> **3. 关闭FileChannel**

完成使用后，FileChannel您必须关闭它。
```
channel.close();   
```

#### SocketChannel和ServerSocketChannel的使用
利用SocketChannel和ServerSocketChannel实现客户端与服务器端简单通信：
SocketChannel 用于创建基于tcp协议的客户端对象，因为SocketChannel中不存在accept()方法，所以，它不能成为一个服务端程序。通过 connect()方法 ，SocketChannel对象可以连接到其他tcp服务器程序。
客户端:
```
package socketchannel;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class WebClient {
    public static void main(String[] args) throws IOException {
        //1.通过SocketChannel的open()方法创建一个SocketChannel对象
        SocketChannel socketChannel = SocketChannel.open();
        //2.连接到远程服务器（连接此通道的socket）
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 3333));
        // 3.创建写数据缓存区对象
        ByteBuffer writeBuffer = ByteBuffer.allocate(128);
        writeBuffer.put("hello WebServer this is from WebClient".getBytes());
        writeBuffer.flip();
        socketChannel.write(writeBuffer);
        //创建读数据缓存区对象
        ByteBuffer readBuffer = ByteBuffer.allocate(128);
        socketChannel.read(readBuffer);
        //String 字符串常量，不可变；StringBuffer 字符串变量（线程安全），可变；StringBuilder 字符串变量（非线程安全），可变
        StringBuilder stringBuffer=new StringBuilder();
        //4.将Buffer从写模式变为可读模式
        readBuffer.flip();
        while (readBuffer.hasRemaining()) {
            stringBuffer.append((char) readBuffer.get());
        }
        System.out.println("从服务端接收到的数据："+stringBuffer);

        socketChannel.close();
    }

}
```
ServerSocketChannel 允许我们监听TCP链接请求，通过ServerSocketChannelImpl的 accept()方法 可以创建一个SocketChannel对象用户从客户端读/写数据。

服务端：
```
package socketchannel;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;

public class WebServer {
    public static void main(String args[]) throws IOException {
        try {
            //1.通过ServerSocketChannel 的open()方法创建一个ServerSocketChannel对象，open方法的作用：打开套接字通道
            ServerSocketChannel ssc = ServerSocketChannel.open();
            //2.通过ServerSocketChannel绑定ip地址和port(端口号)
            ssc.socket().bind(new InetSocketAddress("127.0.0.1", 3333));
            //通过ServerSocketChannelImpl的accept()方法创建一个SocketChannel对象用户从客户端读/写数据
            SocketChannel socketChannel = ssc.accept();
            //3.创建写数据的缓存区对象
            ByteBuffer writeBuffer = ByteBuffer.allocate(128);
            writeBuffer.put("hello WebClient this is from WebServer".getBytes());
            writeBuffer.flip();
            socketChannel.write(writeBuffer);
            //创建读数据的缓存区对象
            ByteBuffer readBuffer = ByteBuffer.allocate(128);
            //读取缓存区数据
            socketChannel.read(readBuffer);
            StringBuilder stringBuffer=new StringBuilder();
            //4.将Buffer从写模式变为可读模式
            readBuffer.flip();
            while (readBuffer.hasRemaining()) {
                stringBuffer.append((char) readBuffer.get());
            }
            System.out.println("从客户端接收到的数据："+stringBuffer);
            socketChannel.close();
            ssc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
**运行效果**
客户端：
![37ac5661df301bcc55f3bab690d6c3ea](Java NIO之Channel(通道).resources/6AF85EF7-83C7-48B6-A6AB-C70AD22A91D4.png)
服务端：
![d6b8298bd2108e3fcd6ed422cec8daa8](Java NIO之Channel(通道).resources/821A61BD-80DF-493F-99D8-4F5330211339.png)
通过上述实例代码，我们可以大概总结出SocketChannel和ServerSocketChannel的使用的一般使用规则：
考虑到篇幅问题，下面只给出大致步骤，不贴代码，可以结合上述实例理解。
**客户端**
1.通过SocketChannel连接到远程服务器
2.创建读数据/写数据缓冲区对象来读取服务端数据或向服务端发送数据
3.关闭SocketChannel
**服务端**
1.通过ServerSocketChannel 绑定ip地址和端口号
2.通过ServerSocketChannelImpl的accept()方法创建一个SocketChannel对象用户从客户端读/写数据
3.创建读数据/写数据缓冲区对象来读取客户端数据或向客户端发送数据
4. 关闭SocketChannel和ServerSocketChannel

#### DatagramChannel的使用

DataGramChannel，类似于java 网络编程的DatagramSocket类；使用UDP进行网络传输， UDP是无连接，面向数据报文段的协议，对传输的数据不保证安全与完整 ；和上面介绍的SocketChannel和ServerSocketChannel的使用方法类似，所以这里就简单介绍一下如何使用。
**1.获取DataGramChannel**
```
//1.通过DatagramChannel的open()方法创建一个DatagramChannel对象
 DatagramChannel datagramChannel = DatagramChannel.open();
  //绑定一个port（端口）
 datagramChannel.bind(new InetSocketAddress(1234));
```
上面代码表示程序可以在1234端口接收数据报。

**2.接收/发送消息**
接收消息：
先创建一个缓存区对象，然后通过receive方法接收消息，这个方法返回一个SocketAddress对象，表示发送消息方的地址：
```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```
发送消息：
由于UDP下，服务端和客户端通信并不需要建立连接，只需要知道对方地址即可发出消息，但是是否发送成功或者成功被接收到是没有保证的;发送消息通过send方法发出，改方法返回一个int值，表示成功发送的字节数：
```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put("datagramchannel".getBytes());
buf.flip();
int send = channel.send(buffer, new InetSocketAddress("localhost",1234));
```
这个例子发送一串字符：“datagramchannel”到主机名为”localhost”服务器的端口1234上。

#### Scatter/Gather
Channel 提供了一种被称为 Scatter/Gather 的新功能，也称为本地矢量 I/O。Scatter/Gather 是指在多个缓冲区上实现一个简单的 I/O 操作。正确使用 Scatter / Gather可以明显提高性能。
大多数现代操作系统都支持本地矢量I/O（native vectored I/O）操作。当您在一个通道上请求一个Scatter/Gather操作时，该请求会被翻译为适当的本地调用来直接填充或抽取缓冲区，减少或避免了缓冲区拷贝和系统调用；
Scatter/Gather应该使用直接的ByteBuffers以从本地I/O获取最大性能优势。
Scatter/Gather功能是通道(Channel)提供的  并不是Buffer。

* Scatter:  从一个Channel读取的信息分散到N个缓冲区中(Buufer).
* Gather:  将N个Buffer里面内容按照顺序发送到一个Channel.

**Scattering Reads**
"scattering read"是把数据从单个Channel写入到多个buffer,如下图所示：
![820b8ed4fd205e451772c9d18e0d629f](Java NIO之Channel(通道).resources/D2633F82-0A59-488A-AEC6-AB443A3125F4.png)
示例代码:
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = { header, body };
channel.read(bufferArray);
```
read()方法内部会负责把数据按顺序写进传入的buffer数组内。一个buffer写满后，接着写到下一个buffer中。
举个例子，假如通道中有200个字节数据，那么header会被写入128个字节数据，body会被写入72个字节数据；
注意：
无论是scatter还是gather操作，都是按照buffer在数组中的顺序来依次读取或写入的；
**Gathering Writes**
"gathering write"把多个buffer的数据写入到同一个channel中，下面是示意图
![f39ff57a4463a05cc93ae22f402e6683](Java NIO之Channel(通道).resources/19060EA5-78B2-49F1-A706-0C99F3BC51A5.png)
示例代码：
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
//write data into buffers
ByteBuffer[] bufferArray = { header, body };
channel.write(bufferArray);
```
write()方法内部会负责把数据按顺序写入到channel中。
注意：
并不是所有数据都写入到通道，写入的数据要根据position和limit的值来判断，只有position和limit之间的数据才会被写入；
举个例子，假如以上header缓冲区中有128个字节数据，但此时position=0，limit=58；那么只有下标索引为0-57的数据才会被写入到通道中.

#### 通道之间的数据传输
在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel。


* **transferFrom():** transferFrom方法把数据从通道源传输到FileChannel
* **transferTo():** transferTo方法把FileChannel数据传输到另一个channel

**参考文档：**

* 官方JDK相关文档
* 谷歌搜索排名第一的Java NIO教程
* 《Java程序员修炼之道》
* ByteBuffer常用方法详解
* JavaNIO易百教程


参考文章：
《Netty官网》
>https://www.jianshu.com/nb/18340870