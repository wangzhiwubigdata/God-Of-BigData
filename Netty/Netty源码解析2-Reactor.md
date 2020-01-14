
## 一：Netty、NIO、多线程？

理清NIO与Netty的关系之前，我们必须先要来看看Reactor模式。Netty是一个典型的多线程的Reactor模式的使用，理解了这部分，在宏观上理解Netty的NIO及多线程部分就不会有什么困难了。

## 二：Reactor

### 1、Reactor的由来

Reactor是一种广泛应用在服务器端开发的设计模式。Reactor中文大多译为“反应堆”，我当初接触这个概念的时候，就感觉很厉害，是不是它的原理就跟“核反应”差不多？后来才知道其实没有什么关系，从Reactor的兄弟“Proactor”（多译为前摄器）就能看得出来，这两个词的中文翻译其实都不是太好，不够形象。实际上，Reactor模式又有别名“Dispatcher”或者“Notifier”，我觉得这两个都更加能表明它的本质。


那么，Reactor模式究竟是个什么东西呢？这要从事件驱动的开发方式说起。我们知道，对于应用服务器，一个主要规律就是，CPU的处理速度是要远远快于IO速度的，如果CPU为了IO操作（例如从Socket读取一段数据）而阻塞显然是不划算的。好一点的方法是分为多进程或者线程去进行处理，但是这样会带来一些进程切换的开销，试想一个进程一个数据读了500ms，期间进程切换到它3次，但是CPU却什么都不能干，就这么切换走了，是不是也不划算？

这时先驱们找到了事件驱动，或者叫回调的方式，来完成这件事情。这种方式就是，应用业务向一个中间人注册一个回调（event handler），当IO就绪后，就这个中间人产生一个事件，并通知此handler进行处理。*这种回调的方式，也体现了“好莱坞原则”（Hollywood principle）-“Don't call us, we'll call you”，在我们熟悉的IoC中也有用到。看来软件开发真是互通的！*

好了，我们现在来看Reactor模式。在前面事件驱动的例子里有个问题：我们如何知道IO就绪这个事件，谁来充当这个中间人？Reactor模式的答案是：由一个不断等待和循环的单独进程（线程）来做这件事，它接受所有handler的注册，并负责先操作系统查询IO是否就绪，在就绪后就调用指定handler进行处理，这个角色的名字就叫做Reactor。

### 2、Reactor与NIO

Java中的NIO可以很好的和Reactor模式结合。关于NIO中的Reactor模式，我想没有什么资料能比Doug Lea大神（不知道Doug Lea？看看JDK集合包和并发包的作者吧）在[《Scalable IO in Java》](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)解释的更简洁和全面了。NIO中Reactor的核心是`Selector`，我写了一个简单的Reactor示例，这里我贴一个核心的Reactor的循环（这种循环结构又叫做`EventLoop`），剩余代码在[learning-src](learning-src/src/main/java/us/codecraft/netty/reactor)目录下。

```java
	public void run() {
		try {
			while (!Thread.interrupted()) {
				selector.select();
				Set selected = selector.selectedKeys();
				Iterator it = selected.iterator();
				while (it.hasNext())
					dispatch((SelectionKey) (it.next()));
				selected.clear();
			}
		} catch (IOException ex) { /* ... */
		}
	}
```

### 3、与Reactor相关的其他概念

前面提到了Proactor模式，这又是什么呢？简单来说，Reactor模式里，操作系统只负责通知IO就绪，具体的IO操作（例如读写）仍然是要在业务进程里阻塞的去做的，而Proactor模式则更进一步，由操作系统将IO操作执行好（例如读取，会将数据直接读到内存buffer中），而handler只负责处理自己的逻辑，真正做到了IO与程序处理异步执行。所以我们一般又说Reactor是同步IO，Proactor是异步IO。

关于阻塞和非阻塞、异步和非异步，以及UNIX底层的机制，大家可以看看这篇文章[IO - 同步，异步，阻塞，非阻塞 （亡羊补牢篇）](http://blog.csdn.net/historyasamirror/article/details/5778378)，以及陶辉（《深入理解nginx》的作者）[《高性能网络编程》](http://blog.csdn.net/russell_tao/article/details/17452997)的系列。

## 三：由Reactor出发来理解Netty

### 1、多线程下的Reactor

讲了一堆Reactor，我们回到Netty。在《Scalable IO in Java》中讲到了一种多线程下的Reactor模式。在这个模式里，mainReactor只有一个，负责响应client的连接请求，并建立连接，它使用一个NIO Selector；subReactor可以有一个或者多个，每个subReactor都会在一个独立线程中执行，并且维护一个独立的NIO Selector。

这样的好处很明显，因为subReactor也会执行一些比较耗时的IO操作，例如消息的读写，使用多个线程去执行，则更加有利于发挥CPU的运算能力，减少IO等待时间。

![Multiple Reactors][2]

### 2、Netty中的Reactor与NIO

好了，了解了多线程下的Reactor模式，我们来看看Netty吧（以下部分主要针对NIO，OIO部分更加简单一点，不重复介绍了）。Netty里对应mainReactor的角色叫做“Boss”，而对应subReactor的角色叫做"Worker"。Boss负责分配请求，Worker负责执行，好像也很贴切！以TCP的Server端为例，这两个对应的实现类分别为`NioServerBoss`和`NioWorker`（Server和Client的Worker没有区别，因为建立连接之后，双方就是对等的进行传输了）。

Netty 3.7中Reactor的EventLoop在`AbstractNioSelector.run()`中，它实现了`Runnable`接口。这个类是Netty NIO部分的核心。它的逻辑非常复杂，其中还包括一些对JDK Bug的处理（例如`rebuildSelector`），刚开始读的时候不需要深入那么细节。我精简了大部分代码，保留主干如下：

```java
abstract class AbstractNioSelector implements NioSelector {

    
    //NIO Selector
    protected volatile Selector selector;

    //内部任务队列
    private final Queue<Runnable> taskQueue = new ConcurrentLinkedQueue<Runnable>();

    //selector循环
    public void run() {
        for (;;) {
            try {
                //处理内部任务队列
                processTaskQueue();
                //处理selector事件对应逻辑
                process(selector);
            } catch (Throwable t) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // Ignore.
                }
            }
        }
    }

    private void processTaskQueue() {
        for (;;) {
            final Runnable task = taskQueue.poll();
            if (task == null) {
                break;
            }
            task.run();
        }
    }

    protected abstract void process(Selector selector) throws IOException;

}
```

其中process是主要的处理事件的逻辑，例如在`AbstractNioWorker`中，处理逻辑如下：

```java
    protected void process(Selector selector) throws IOException {
        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        if (selectedKeys.isEmpty()) {
            return;
        }
        for (Iterator<SelectionKey> i = selectedKeys.iterator(); i.hasNext();) {
            SelectionKey k = i.next();
            i.remove();
            try {
                int readyOps = k.readyOps();
                if ((readyOps & SelectionKey.OP_READ) != 0 || readyOps == 0) {
                    if (!read(k)) {
                        // Connection already closed - no need to handle write.
                        continue;
                    }
                }
                if ((readyOps & SelectionKey.OP_WRITE) != 0) {
                    writeFromSelectorLoop(k);
                }
            } catch (CancelledKeyException e) {
                close(k);
            }

            if (cleanUpCancelledKeys()) {
                break; // break the loop to avoid ConcurrentModificationException
            }
        }
    }
```

这不就是第二部分提到的selector经典用法了么？

在Netty 4.0之后，作者觉得`NioSelector`这个叫法，以及区分`NioBoss`和`NioWorker`的做法稍微繁琐了点，干脆就将这些合并成了`NioEventLoop`，从此这两个角色就不做区分了。我倒是觉得新版本的会更优雅一点。

### 3、Netty中的多线程

下面我们来看Netty的多线程部分。一旦对应的Boss或者Worker启动，就会分配给它们一个线程去一直执行。对应的概念为`BossPool`和`WorkerPool`。对于每个`NioServerSocketChannel`，Boss的Reactor有一个线程，而Worker的线程数由Worker线程池大小决定，但是默认最大不会超过CPU核数*2，当然，这个参数可以通过`NioServerSocketChannelFactory`构造函数的参数来设置。

```java
    public NioServerSocketChannelFactory(
            Executor bossExecutor, Executor workerExecutor,
            int workerCount) {
        this(bossExecutor, 1, workerExecutor, workerCount);
    }
```

最后我们比较关心一个问题，我们之前`ChannlePipeline`中的ChannleHandler是在哪个线程执行的呢？答案是在Worker线程里执行的，并且会阻塞Worker的EventLoop。例如，在`NioWorker`中，读取消息完毕之后，会触发`MessageReceived`事件，这会使得Pipeline中的handler都得到执行。

```java
    protected boolean read(SelectionKey k) {
        ....

        if (readBytes > 0) {
            // Fire the event.
            fireMessageReceived(channel, buffer);
        }

        return true;
    }
```

可以看到，对于处理事件较长的业务，并不太适合直接放到ChannelHandler中执行。那么怎么处理呢？我们在Handler部分会进行介绍。


参考资料：

* Scalable IO in Java [http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)
* Netty5.0架构剖析和源码解读 [http://vdisk.weibo.com/s/C9LV9iVqH13rW/1391437855](http://vdisk.weibo.com/s/C9LV9iVqH13rW/1391437855)
* Reactor pattern [http://en.wikipedia.org/wiki/Reactor_pattern](http://en.wikipedia.org/wiki/Reactor_pattern)
* Reactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handles for Synchronous Events [http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf](http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf)
* 高性能网络编程6--reactor反应堆与定时器管理 [http://blog.csdn.net/russell_tao/article/details/17452997](http://blog.csdn.net/russell_tao/article/details/17452997)
* IO - 同步，异步，阻塞，非阻塞 （亡羊补牢篇）[http://blog.csdn.net/historyasamirror/article/details/5778378](http://blog.csdn.net/historyasamirror/article/details/5778378)

题图来自：[http://www.worldindustrialreporter.com/france-gives-green-light-to-tokamak-fusion-reactor/](http://www.worldindustrialreporter.com/france-gives-green-light-to-tokamak-fusion-reactor/)

  [1]: http://static.oschina.net/uploads/space/2014/0208/164000_EQQb_190591.jpg
  [2]: http://static.oschina.net/uploads/space/2013/1125/130828_uKWD_190591.jpeg