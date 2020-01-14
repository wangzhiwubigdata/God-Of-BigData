### **Java高级特性增强-Volatile**
本部分网络上有大量的资源可以参考，在这里做了部分整理，感谢前辈的付出，每节文章末尾有引用列表，源码推荐看JDK1.8以后的版本，注意甄别~

####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**


* * *
## volatile关键字


#### volatile特性
volatile就可以说是java虚拟机提供的最轻量级的同步机制。但它同时不容易被正确理解，也至于在并发编程中很多程序员遇到线程安全的问题就会使用synchronized。Java内存模型告诉我们，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。线程在工作内存进行操作后何时会写到主内存中？这个时机对普通变量是没有规定的，而针对volatile修饰的变量给java虚拟机特殊的约定，线程对volatile变量的修改会立刻被其他线程所感知，即不会出现数据脏读的现象，从而保证数据的“可见性”。
通俗来说就是，线程A对一个volatile变量的修改，对于其它线程来说是可见的，即线程每次获取volatile变量的值都是最新的。

#### volatile的实现原理
在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令。我们想这个Lock指令肯定有神奇的地方，那么Lock前缀的指令在多核处理器下会发现什么事情了？主要有这两个方面的影响：

将当前处理器缓存行的数据写回系统内存；
这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。因此，经过分析我们可以得出如下结论：

Lock前缀的指令会引起处理器缓存写回内存；
一个处理器的缓存回写到内存会导致其他处理器的缓存失效；
当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

这样针对volatile变量通过这样的机制就使得每个线程都能获得该变量的最新值。

#### 我们在项目中如何使用？
1、状态标记量
在高并发的场景中，通过一个boolean类型的变量isopen，控制代码是否走促销逻辑，该如何实现？
```
public class ServerHandler {
    private volatile isopen;
    public void run() {
        if (isopen) {
           //isopen=true逻辑
        } else {
          //其他逻辑
        }
    }
    public void setIsopen(boolean isopen) {
        this.isopen = isopen
    }
}
```
场景细节无需过分纠结，这里只是举个例子说明volatile的使用方法，用户的请求线程执行run方法，如果需要开启促销活动，可以通过后台设置，具体实现可以发送一个请求，调用setIsopen方法并设置isopen为true，由于isopen是volatile修饰的，所以一经修改，其他线程都可以拿到isopen的最新值，用户请求就可以执行isopen=true的逻辑。

2、double check
单例模式的一种实现方式，但很多人会忽略volatile关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是100%，说不定在未来的某个时刻，隐藏的bug就出来了。
```
class Singleton {
    private volatile static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    } 
}
```
不过在众多单例模式的实现中，我比较推荐懒加载的优雅写法Initialization on Demand Holder（IODH）。
```
public class Singleton {  
    static class SingletonHolder {  
        static Singleton instance = new Singleton();  
    }  
      
    public static Singleton getInstance(){  
        return SingletonHolder.instance;  
    }  
}  
```

#### 如何保证内存可见性

在java虚拟机的内存模型中，有主内存和工作内存的概念，每个线程对应一个工作内存，并共享主内存的数据，下面看看操作普通变量和volatile变量有什么不同：
1、对于普通变量：读操作会优先读取工作内存的数据，如果工作内存中不存在，则从主内存中拷贝一份数据到工作内存中；写操作只会修改工作内存的副本数据，这种情况下，其它线程就无法读取变量的最新值。
2、对于volatile变量，读操作时JMM会把工作内存中对应的值设为无效，要求线程从主内存中读取数据；写操作时JMM会把工作内存中对应的数据刷新到主内存中，这种情况下，其它线程就可以读取变量的最新值。
volatile变量的内存可见性是基于内存屏障(Memory Barrier)实现的，什么是内存屏障？内存屏障，又称内存栅栏，是一个CPU指令。在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM为了保证在不同的编译器和CPU上有相同的结果，通过插入特定类型的内存屏障来禁止特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和CPU：不管什么指令都不能和这条Memory Barrier指令重排序。

举例如下：
```
class Singleton {
    private volatile static Singleton instance;
    private int a;
    private int b;
    private int b;
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    a = 1;  // 1
                     b = 2;  // 2
                    instance = new Singleton();  // 3
                    c = a + b;  // 4
                }
            }
        }
        return instance;
    } 
}
```
1、如果变量instance没有volatile修饰，语句1、2、3可以随意的进行重排序执行，即指令执行过程可能是3214或1324。
2、如果是volatile修饰的变量instance，会在语句3的前后各插入一个内存屏障。
通过观察volatile变量和普通变量所生成的汇编代码可以发现，操作volatile变量会多出一个lock前缀指令：
```
Java代码：
instance = new Singleton();

汇编代码：
0x01a3de1d: movb $0x0,0x1104800(%esi);
0x01a3de24: **lock** addl $0x0,(%esp);
```
这个lock前缀指令相当于上述的内存屏障，提供了以下保证：
1、将当前CPU缓存行的数据写回到主内存；
2、这个写回内存的操作会导致在其它CPU里缓存了该内存地址的数据无效。
CPU为了提高处理性能，并不直接和内存进行通信，而是将内存的数据读取到内部缓存（L1，L2）再进行操作，但操作完并不能确定何时写回到内存，如果对volatile变量进行写操作，当CPU执行到Lock前缀指令时，会将这个变量所在缓存行的数据写回到内存，不过还是存在一个问题，就算内存的数据是最新的，其它CPU缓存的还是旧值，所以为了保证各个CPU的缓存一致性，每个CPU通过嗅探在总线上传播的数据来检查自己缓存的数据有效性，当发现自己缓存行对应的内存地址的数据被修改，就会将该缓存行设置成无效状态，当CPU读取该变量时，发现所在的缓存行被设置为无效，就会重新从内存中读取数据到缓存中。
这也是我们之前讲的原理部分的解释~


#### volatile的happens-before关系
volatile变量可以通过缓存一致性协议保证每个线程都能获得最新值，即满足数据的“可见性”。我们继续延续上一篇分析问题的方式（我一直认为思考问题的方式是属于自己，也才是最重要的，也在不断培养这方面的能力），我一直将并发分析的切入点分为两个核心，三大性质。两大核心：JMM内存模型（主内存和工作内存）以及happens-before；三条性质：原子性，可见性，有序性（关于三大性质的总结在以后得文章会和大家共同探讨）。废话不多说，先来看两个核心之一：volatile的happens-before关系。
在六条happens-before规则中有一条是：volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。下面我们结合具体的代码，我们利用这条规则推导下：
```
public class VolatileExample {
    private int a = 0;
    private volatile boolean flag = false;
    public void writer(){
        a = 1;          //1
        flag = true;   //2
    }
    public void reader(){
        if(flag){      //3
            int i = a; //4
        }
    }
}
```
上面的实例代码对应的happens-before关系如下图所示：
![ab3fc4589fa61bf75ad91d7080664a7d](大数据成神之路-Java高级特性增强(volatile关键字).resources/14BF4468-D1E0-4FBF-B503-A888E309418D.png)
加锁线程A先执行writer方法，然后线程B执行reader方法图中每一个箭头两个节点就代码一个happens-before关系，黑色的代表根据程序顺序规则推导出来，红色的是根据volatile变量的写happens-before 于任意后续对volatile变量的读，而蓝色的就是根据传递性规则推导出来的。这里的2 happen-before 3，同样根据happens-before规则定义：如果A happens-before B,则A的执行结果对B可见，并且A的执行顺序先于B的执行顺序，我们可以知道操作2执行结果对操作3来说是可见的，也就是说当线程A将volatile变量 flag更改为true后线程B就能够迅速感知。

-----------
**参考文章和书籍：**

《Java并发编程的艺术》
《实战Java高并发程序设计》

https://blog.csdn.net/qq_34337272/article/details/79680771

https://www.jianshu.com/p/a5f99f25329a

https://www.jianshu.com/p/506c1e38a922
