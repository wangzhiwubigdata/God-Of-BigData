### **Java高级特性增强-多线程**
本部分网络上有大量的资源可以参考，在这里做了部分整理，感谢前辈的付出，每节文章末尾有引用列表，源码推荐看JDK1.8以后的版本，注意甄别~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

* * *
## 多线程
![89bf0392f832b459ed62efb31af4461e](大数据成神之路-Java高级特性增强(多线程).resources/F18CB21B-41D4-4D8D-890D-4B632F69F96A.jpg)
参考资料列表：
java并发编程指南
**https://blog.csdn.net/qq_34337272/column/info/20860**
死磕系列：
**http://cmsblogs.com/?p=2611**
面试题系列：
**https://blog.csdn.net/linzhiqiang0316/article/details/80473906**
简书：
**https://www.jianshu.com/nb/4893857**
以上几个博客足够了，着重推荐一下死磕系列和简书的文章，比较深入


#### 进程和多线程简介
##### 进程和线程
进程和线程的对比这一知识点由于过于基础，所以在面试中很少碰到，但是极有可能会在笔试题中碰到。常见的提问形式是这样的：“什么是线程和进程?，请简要描述线程与进程的关系、区别及优缺点？ ”。

##### 何为进程？
进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。
或者我们可以这样说：
进程，是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如CPU时间，内存空间，文件，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系统载入内存中。
##### 何为线程？
线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

##### 何为多线程
多线程就是多个线程同时运行或交替运行。单核CPU的话是顺序执行，也就是交替运行。多核CPU的话，因为每个CPU有自己的运算器，所以在多个CPU中可以同时运行。

##### 为什么多线程是必要的
个人觉得可以用一句话概括：开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

##### 为什么提倡多线程而不是多进程
线程就是轻量级进程，是程序执行的最小单位。使用多线程而不是用多进程去进行并发程序的设计，是因为线程间的切换和调度的成本远远小于进程。

##### 线程有什么优缺点
1）好处
使用多线程可以把程序中占据时间长的任务放到后台去处理，如图片、视屏的下载。
发挥多核处理器的优势，并发执行让系统运行的更快、更流畅，用户体验更好。
2）坏处
大量的线程降低代码的可读性。
更多的线程需要更多的内存空间。
当多个线程对同一个资源出现争夺时候要注意线程安全的问题。
##### 多线程中重要的概念
**同步和异步**
同步和异步通常用来形容一次方法调用。同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者可以继续后续的操作。

关于异步目前比较经典以及常用的实现方式就是消息队列：在不使用消息队列服务器的时候，用户的请求数据直接写入数据库，在高并发的情况下数据库压力剧增，使得响应速度变慢。但是在使用消息队列之后，用户的请求数据发送给消息队列之后立即 返回，再由消息队列的消费者进程从消息队列中获取数据，异步写入数据库。由于消息队列服务器处理速度快于数据库（消息队列也比数据库有更好的伸缩性），因此响应速度得到大幅改善。

**并发(Concurrency)和并行(Parallelism)**
并发和并行是两个非常容易被混淆的概念。它们都可以表示两个或者多个任务一起执行，但是偏重点有些不同。并发偏重于多个任务交替执行，而多个任务之间有可能还是串行的。而并行是真正意义上的“同时执行”。

多线程在单核CPU的话是顺序执行，也就是交替运行（并发）。多核CPU的话，因为每个CPU有自己的运算器，所以在多个CPU中可以同时运行（并行）。

**高并发**
高并发（High Concurrency）是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计保证系统能够同时并行处理很多请求。

高并发相关常用的一些指标有响应时间（Response Time），吞吐量（Throughput），每秒查询率QPS（Query Per Second），并发用户数等。

**临界区**
临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。在并行程序中，临界区资源是保护的对象。

**阻塞和非阻塞**
非阻塞指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回，而阻塞与之相反。

##### 多线程的创建方式

**继承`Thread`类**
```java
public class MyThread extends Thread {
	@Override
	public void run() {
		super.run();
		System.out.println("MyThread");
	}
}
```
**实现`Runnable`接口**
```java
public class MyRunnable implements Runnable {
	@Override
	public void run() {
		System.out.println("MyRunnable");
	}
}
```
**实现`Callable`接口**
```java
class ImplementsCallable implements Callable<String>{

    @Override
    public String call() throws Exception {
        return UUID.randomUUID().toString().substring(0,8);
    }
}
```
```java
private static void implementsCallable() throws ExecutionException, InterruptedException {
        //A
        FutureTask futureTaskA = new FutureTask<String>(new ImplementsCallable());
        new Thread(futureTaskA,"implementsCallable-A").start();

}
```
**线程池**
> 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。 说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。
—————《阿里巴巴Java开发手册》泰山版第一章第七节并发处理第3点。

> 【强制】线程池不允许使用Executors去创建，而是通过`ThreadPoolExecutor`的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：`Executors`返回的线程池对象的弊端如下： 1） `FixedThreadPool`和`SingleThreadPool`： 允许的请求队列长度为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致OOM。 2） `CachedThreadPool`： 允许的创建线程数量为`Integer.MAX_VALUE`，可能会创建大量的线程，从而导致OOM。————《阿里巴巴Java开发手册》泰山版第一章第七节并发处理第4点。

我们在实际开发环境中，建议使用线程池的方式创建线程。

```java
public class ThreadPool
{
	private static int POOL_NUM = 10;
	
	public static void main(String[] args)
	{
        ExecutorService executorService = new ThreadPoolExecutor(
                        5,
                        5,
                        1l,
                        TimeUnit.SECONDS,
                        new LinkedBlockingQueue<>(100),
                        Executors.defaultThreadFactory(),
                        new ThreadPoolExecutor.AbortPolicy()
                );
		for(int i = 0; i<POOL_NUM; i++)
		{
		RunnableThread thread = new RunnableThread();
		executorService.execute(thread);
		}
	}
}
 
class RunnableThread implements Runnable
{
	private int THREAD_NUM = 10;
	public void run()
	{
		for(int i = 0; i<THREAD_NUM; i++)
		{
			System.out.println("线程" + Thread.currentThread() + " " + i);
		} 
	}
}
```
##### 线程的生命周期

线程一共有五个状态，分别如下：
**新建(new)：**
当创建Thread类的一个实例（对象）时，此线程进入新建状态（未被启动）。例如：Thread t1 = new Thread() 。

**可运行(runnable)：**
线程对象创建后，其他线程(比如 main 线程）调用了该对象的 start 方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取 cpu 的使用权。例如：t1.start() 。

**运行(running)：**
线程获得 CPU 资源正在执行任务（#run() 方法），此时除非此线程自动放弃 CPU 资源或者有优先级更高的线程进入，线程将一直运行到结束。
死亡(dead)：当线程执行完毕或被其它线程杀死，线程就进入死亡状态，这时线程不可能再进入就绪状态等待执行。
**自然终止：**
正常运行完 #run()方法，终止。
**异常终止：**
调用 #stop() 方法，让一个线程终止运行。
**堵塞(blocked)：**
由于某种原因导致正在运行的线程让出 CPU 并暂停自己的执行，即进入堵塞状态。直到线程进入可运行(runnable)状态，才有机会再次获得 CPU 资源，转到运行(running)状态。阻塞的情况有三种：
**正在睡眠：**
调用 #sleep(long t) 方法，可使线程进入睡眠方式。
一个睡眠着的线程在指定的时间过去可进入可运行(runnable)状态。
**正在等待：**
调用 #wait() 方法。
调用 notify() 方法，回到就绪状态。
**被另一个线程所阻塞：**
调用 #suspend() 方法。
调用 #resume() 方法，就可以恢复。

见下图：
![5eeec5f68f4fc412246efd4111d6fdec](大数据成神之路-Java高级特性增强(多线程).resources/6AC11272-0DE3-44D7-9533-2647D0A652BF.png)

##### 线程的优先级
每个线程都具有各自的优先级，线程的优先级可以在程序中表明该线程的重要性，如果有很多线程处于就绪状态，系统会根据优先级来决定首先使哪个线程进入运行状态。但这个并不意味着低。
优先级的线程得不到运行，而只是它运行的几率比较小，如垃圾回收机制线程的优先级就比较低。所以很多垃圾得不到及时的回收处理。

线程优先级具有继承特性比如A线程启动B线程，则B线程的优先级和A是一样的。

线程优先级具有随机性也就是说线程优先级高的不一定每一次都先执行完。

Thread类中包含的成员变量代表了线程的某些优先级。如Thread.MIN_PRIORITY（常数1），Thread.NORM_PRIORITY（常数5）,
Thread.MAX_PRIORITY（常数10）。其中每个线程的优先级都在Thread.MIN_PRIORITY（常数1） 到Thread.MAX_PRIORITY（常数10） 之间，在默认情况下优先级都是Thread.NORM_PRIORITY（常数5）。

学过操作系统这门课程的话，我们可以发现多线程优先级或多或少借鉴了操作系统对进程的管理

##### 线程的终止
**interrupt()方法**
注意：interrupt()方法的使用效果并不像for+break语句那样，马上就停止循环。调用interrupt方法是在当前线程中打了一个停止标志，并不是真的停止线程。
```
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
输出结果：
```
...
i=499994
i=499995
i=499996
i=499997
i=499998
i=499999
i=500000
```
**判断线程是否停止状态**
Thread.java类中提供了两种方法：

this.interrupted(): 测试当前线程是否已经中断；
this.isInterrupted(): 测试线程是否已经中断；
那么这两个方法有什么图区别呢？
我们先来看看this.interrupted()方法的解释：测试当前线程是否已经中断，当前线程是指运行this.interrupted()方法的线程。

```
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            i++;
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();

            System.out.println("stop 1->" + thread.interrupted());
            System.out.println("stop 2->" + thread.interrupted());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：
```
stop 1->false
stop 2->false
```
类Run.java中虽然是在thread对象上调用以下代码：thread.interrupt(), 后面又使用

```
System.out.println("stop 1->" + thread.interrupted());
System.out.println("stop 2->" + thread.interrupted());  
```
来判断thread对象所代表的线程是否停止，但从控制台打印的结果来看，线程并未停止，这也证明了interrupted()方法的解释，测试当前线程是否已经中断。这个当前线程是main，它从未中断过，所以打印的结果是两个false.

如何使main线程产生中断效果呢？
```
public class Run2 {
    public static void main(String args[]){
        Thread.currentThread().interrupt();
        System.out.println("stop 1->" + Thread.interrupted());
        System.out.println("stop 2->" + Thread.interrupted());

        System.out.println("End");
    }
}    
```
运行结果为：
```
stop 1->true
stop 2->false
End
```
方法interrupted()的确判断出当前线程是否是停止状态。但为什么第2个布尔值是false呢？ 官方帮助文档中对interrupted方法的解释：
测试当前线程是否已经中断。线程的中断状态由该方法清除。 换句话说，如果连续两次调用该方法，则第二次调用返回false。

下面来看一下inInterrupted()方法。
```
public class Run3 {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        thread.interrupt();
        System.out.println("stop 1->" + thread.isInterrupted());
        System.out.println("stop 2->" + thread.isInterrupted());
    }
}
```

运行结果：

```
stop 1->true
stop 2->true
```
isInterrupted()并为清除状态，所以打印了两个true。

**能停止的线程--异常法**
有了前面学习过的知识点，就可以在线程中用for语句来判断一下线程是否是停止状态，如果是停止状态，则后面的代码不再运行即可：

```
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            if(this.interrupted()) {
                System.out.println("线程已经终止， for循环不再执行");
                break;
            }
            System.out.println("i="+(i+1));
        }
    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        try {
            Thread.sleep(2000);
            thread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：
```
...
i=202053
i=202054
i=202055
i=202056
线程已经终止， for循环不再执行
```
上面的示例虽然停止了线程，但如果for语句下面还有语句，还是会继续运行的。看下面的例子：
```
public class MyThread extends Thread {
    public void run(){
        super.run();
        for(int i=0; i<500000; i++){
            if(this.interrupted()) {
                System.out.println("线程已经终止， for循环不再执行");
                break;
            }
            System.out.println("i="+(i+1));
        }

        System.out.println("这是for循环外面的语句，也会被执行");
    }
}
```
使用Run.java执行的结果是：
```
...
i=180136
i=180137
i=180138
i=180139
线程已经终止， for循环不再执行
这是for循环外面的语句，也会被执行
```
如何解决语句继续运行的问题呢？ 看一下更新后的代码：
```
public class MyThread extends Thread {
    public void run(){
        super.run();
        try {
            for(int i=0; i<500000; i++){
                if(this.interrupted()) {
                    System.out.println("线程已经终止， for循环不再执行");
                        throw new InterruptedException();
                }
                System.out.println("i="+(i+1));
            }

            System.out.println("这是for循环外面的语句，也会被执行");
        } catch (InterruptedException e) {
            System.out.println("进入MyThread.java类中的catch了。。。");
            e.printStackTrace();
        }
    }
}
```
使用Run.java运行的结果如下：
```
...
i=203798
i=203799
i=203800
线程已经终止， for循环不再执行
进入MyThread.java类中的catch了。。。
java.lang.InterruptedException
    at thread.MyThread.run(MyThread.java:13)
```
**在沉睡中停止**
如果线程在sleep()状态下停止线程，会是什么效果呢？
```
public class MyThread extends Thread {
    public void run(){
        super.run();

        try {
            System.out.println("线程开始。。。");
            Thread.sleep(200000);
            System.out.println("线程结束。");
        } catch (InterruptedException e) {
            System.out.println("在沉睡中被停止, 进入catch， 调用isInterrupted()方法的结果是：" + this.isInterrupted());
            e.printStackTrace();
        }

    }
}
```
使用Run.java运行的结果是：
```
线程开始。。。
在沉睡中被停止, 进入catch， 调用isInterrupted()方法的结果是：false
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at thread.MyThread.run(MyThread.java:12)
```
从打印的结果来看， 如果在sleep状态下停止某一线程，会进入catch语句，并且清除停止状态值，使之变为false。

前一个实验是先sleep然后再用interrupt()停止，与之相反的操作在学习过程中也要注意：
```
public class MyThread extends Thread {
    public void run(){
        super.run();
        try {
            System.out.println("线程开始。。。");
            for(int i=0; i<10000; i++){
                System.out.println("i=" + i);
            }
            Thread.sleep(200000);
            System.out.println("线程结束。");
        } catch (InterruptedException e) {
             System.out.println("先停止，再遇到sleep，进入catch异常");
            e.printStackTrace();
        }

    }
}

public class Run {
    public static void main(String args[]){
        Thread thread = new MyThread();
        thread.start();
        thread.interrupt();
    }
}
```
运行结果：
```
i=9998
i=9999
先停止，再遇到sleep，进入catch异常
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at thread.MyThread.run(MyThread.java:15)
```
**能停止的线程---暴力停止**
使用stop()方法停止线程则是非常暴力的。
```
public class MyThread extends Thread {
    private int i = 0;
    public void run(){
        super.run();
        try {
            while (true){
                System.out.println("i=" + i);
                i++;
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        Thread.sleep(2000);
        thread.stop();
    }
}
```
运行结果：
```
i=0
i=1
i=2
i=3
i=4
i=5
i=6
i=7
i=8
i=9

Process finished with exit code 0
```
**方法stop()与java.lang.ThreadDeath异常**

调用stop()方法时会抛出java.lang.ThreadDeath异常，但是通常情况下，此异常不需要显示地捕捉。
```
public class MyThread extends Thread {
    private int i = 0;
    public void run(){
        super.run();
        try {
            this.stop();
        } catch (ThreadDeath e) {
            System.out.println("进入异常catch");
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
    }
}
```
stop()方法以及作废，因为如果强制让线程停止有可能使一些清理性的工作得不到完成。另外一个情况就是对锁定的对象进行了解锁，导致数据得不到同步的处理，出现数据不一致的问题。

**释放锁的不良后果**

使用stop()释放锁将会给数据造成不一致性的结果。如果出现这样的情况，程序处理的数据就有可能遭到破坏，最终导致程序执行的流程错误，一定要特别注意：
```
public class SynchronizedObject {
    private String name = "a";
    private String password = "aa";

    public synchronized void printString(String name, String password){
        try {
            this.name = name;
            Thread.sleep(100000);
            this.password = password;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

public class MyThread extends Thread {
    private SynchronizedObject synchronizedObject;
    public MyThread(SynchronizedObject synchronizedObject){
        this.synchronizedObject = synchronizedObject;
    }

    public void run(){
        synchronizedObject.printString("b", "bb");
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        SynchronizedObject synchronizedObject = new SynchronizedObject();
        Thread thread = new MyThread(synchronizedObject);
        thread.start();
        Thread.sleep(500);
        thread.stop();
        System.out.println(synchronizedObject.getName() + "  " + synchronizedObject.getPassword());
    }
}
```
输出结果：
```
b  aa
```
由于stop()方法以及在JDK中被标明为“过期/作废”的方法，显然它在功能上具有缺陷，所以不建议在程序张使用stop()方法。

 **使用return停止线程**
 将方法interrupt()与return结合使用也能实现停止线程的效果：
```
 public class MyThread extends Thread {
    public void run(){
        while (true){
            if(this.isInterrupted()){
                System.out.println("线程被停止了！");
                return;
            }
            System.out.println("Time: " + System.currentTimeMillis());
        }
    }
}

public class Run {
    public static void main(String args[]) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        Thread.sleep(2000);
        thread.interrupt();
    }
}
```
输出结果：
```
...
Time: 1467072288503
Time: 1467072288503
Time: 1467072288503
线程被停止了！
```
笔者花了巨大篇幅介绍线程的终止，因为这是在实际开发中最容易犯的错误，千万注意哦~