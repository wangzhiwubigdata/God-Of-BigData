### **Java高级特性增强-锁**
本部分网络上有大量的资源可以参考，在这里做了部分整理，感谢前辈的付出，每节文章末尾有引用列表，源码推荐看JDK1.8以后的版本，注意甄别~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**


* * *
## 锁
#### Java中的锁分类
在读很多并发文章中，会提及各种各样锁如公平锁，乐观锁等等，这篇文章介绍就是各种锁。介绍的内容如下：
公平锁/非公平锁
可重入锁
独享锁/共享锁
互斥锁/读写锁
乐观锁/悲观锁
分段锁
偏向锁/轻量级锁/重量级锁
自旋锁
上面是很多锁的名词，这些分类并不是全是指锁的状态，有的指锁的特性，有的指锁的设计，下面总结的内容是对每个锁的名词进行一定的解释。
**公平锁/非公平锁**
公平锁是指多个线程按照申请锁的顺序来获取锁。
非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
对于Java ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

**可重入锁**
可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。说的有点抽象，下面会有一个代码的示例。
对于Java ReentrantLock而言, 他的名字就可以看出是一个可重入锁，其名字是Re entrant Lock重新进入锁。
对于Synchronized而言,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。
```
synchronized void setA() throws Exception{
    Thread.sleep(1000);
    setB();
}

synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
```
上面的代码就是一个可重入锁的一个特点，如果不是可重入锁的话，setB可能不会被当前线程执行，可能造成死锁。

**独享锁/共享锁**
独享锁是指该锁一次只能被一个线程所持有。
共享锁是指该锁可被多个线程所持有。

对于Java ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。
读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。
对于Synchronized而言，当然是独享锁。

**互斥锁/读写锁**
上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
互斥锁在Java中的具体实现就是ReentrantLock
读写锁在Java中的具体实现就是ReadWriteLock

**乐观锁/悲观锁**
乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。

从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。
悲观锁在Java中的使用，就是利用各种锁。
乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

**分段锁**
分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（JDK7与JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock)。
当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在那一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。
分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

**偏向锁/轻量级锁/重量级锁**
这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

**自旋锁**
在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

#### Lock接口

在Lock接口出现之前，Java程序是靠synchronized关键字实现锁功能的。JDK1.5之后并发包中新增了Lock接口以及相关实现类来实现锁功能。

虽然synchronized方法和语句的范围机制使得使用监视器锁更容易编程，并且有助于避免涉及锁的许多常见编程错误，但是有时您需要以更灵活的方式处理锁。例如，用于遍历并发访问的数据结构的一些算法需要使用“手动”或“链锁定”：您获取节点A的锁定，然后获取节点B，然后释放A并获取C，然后释放B并获得D等。在这种场景中synchronized关键字就不那么容易实现了，使用Lock接口容易很多。

Lock接口的实现类： 
ReentrantLock ， ReentrantReadWriteLock.ReadLock ， ReentrantReadWriteLock.WriteLock

##### AbstractQueuedSynchronizer
当你查看源码时你会惊讶的发现ReentrantLock并没有多少代码，另外有一个很明显的特点是：基本上所有的方法的实现实际上都是调用了其静态内存类Sync中的方法，而Sync类继承了AbstractQueuedSynchronizer（AQS）。可以看出要想理解ReentrantLock关键核心在于对队列同步器AbstractQueuedSynchronizer（简称同步器）的理解。

在同步组件的实现中，AQS是核心部分，同步组件的实现者通过使用AQS提供的模板方法实现同步组件语义，AQS则实现了对同步状态的管理，以及对阻塞线程进行排队，等待通知等等一些底层的实现处理。AQS的核心也包括了这些方面:同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取这些特性的实现，而这些实际上则是AQS提供出来的模板方法，归纳整理如下：
**独占式锁：**
```
void acquire(int arg):
独占式获取同步状态，如果获取失败则插入同步队列进行等待；
void acquireInterruptibly(int arg):
与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
boolean tryAcquireNanos(int arg, long nanosTimeout):
在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
boolean release(int arg):
释放同步状态，该方法会唤醒在同步队列中的下一个节点
```

**共享式锁：**
```
void acquireShared(int arg):
共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态
void acquireSharedInterruptibly(int arg):
在acquireShared方法基础上增加了能响应中断的功能
boolean tryAcquireSharedNanos(int arg, long nanosTimeout):
在acquireSharedInterruptibly基础上增加了超时等待的功能
boolean releaseShared(int arg):共享式释放同步状态
```

##### ReentrantLock
ReentrantLock重入锁，是实现Lock接口的一个类，也是在实际编程中使用频率很高的一个锁，支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞。在java关键字synchronized隐式支持重入性,synchronized通过获取自增，释放自减的方式实现重入。与此同时，ReentrantLock还支持公平锁和非公平锁两种方式。那么，要想完完全全的弄懂ReentrantLock的话，主要也就是ReentrantLock同步语义的学习：1. 重入性的实现原理；2. 公平锁和非公平锁。

##### 重入性的实现原理
要想支持重入性，就要解决两个问题：1. 在线程获取锁的时候，如果已经获取锁的线程是当前线程的话则直接再次获取成功；2. 由于锁会被获取n次，那么只有锁在被释放同样的n次之后，该锁才算是完全释放成功。通过这篇文章，我们知道，同步组件主要是通过重写AQS的几个protected方法来表达自己的同步语义。针对第一个问题，我们来看看ReentrantLock是怎样实现的，以非公平锁为例，判断当前线程能否获得锁为例，核心方法为nonfairTryAcquire：

```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //1. 如果该锁未被任何线程占有，该锁能被当前线程获取
	if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	//2.若被占有，检查占有线程是否是当前线程
    else if (current == getExclusiveOwnerThread()) {
		// 3. 再次获取，计数加一
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

```
这段代码的逻辑也很简单，具体请看注释。为了支持重入性，在第二步增加了处理逻辑，如果该锁已经被线程所占有了，会继续检查占有线程是否为当前线程，如果是的话，同步状态加1返回true，表示可以再次获取成功。每次重新获取都会对同步状态进行加一的操作，那么释放的时候处理思路是怎样的了？（依然还是以非公平锁为例）核心方法为tryRelease：
```
protected final boolean tryRelease(int releases) {
	//1. 同步状态减1
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
		//2. 只有当同步状态为0时，锁成功被释放，返回true
        free = true;
        setExclusiveOwnerThread(null);
    }
	// 3. 锁未被完全释放，返回false
    setState(c);
    return free;
}

```
代码的逻辑请看注释，需要注意的是，重入锁的释放必须得等到同步状态为0时锁才算成功释放，否则锁仍未释放。如果锁被获取n次，释放了n-1次，该锁未完全释放返回false，只有被释放n次才算成功释放，返回true。到现在我们可以理清ReentrantLock重入性的实现了，也就是理解了同步语义的第一条.

##### 公平锁与非公平锁
ReentrantLock支持两种锁：公平锁和非公平锁。何谓公平性，是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO。ReentrantLock的构造方法无参时是构造非公平锁，源码为：
```
public ReentrantLock() {
    sync = new NonfairSync();
}
```
另外还提供了另外一种方式，可传入一个boolean值，true时为公平锁，false时为非公平锁，源码为：
```
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
在上面非公平锁获取时（nonfairTryAcquire方法）只是简单的获取了一下当前状态做了一些逻辑处理，并没有考虑到当前同步队列中线程等待的情况。我们来看看公平锁的处理逻辑是怎样的，核心方法为：

```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
  }
}
```
这段代码的逻辑与nonfairTryAcquire基本上一直，唯一的不同在于增加了hasQueuedPredecessors的逻辑判断，方法名就可知道该方法用来判断当前节点在同步队列中是否有前驱节点的判断，如果有前驱节点说明有线程比当前线程更早的请求资源，根据公平性，当前线程请求资源失败。如果当前节点没有前驱节点的话，再才有做后面的逻辑判断的必要性。公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程能再次获取到锁。

**公平锁 VS 非公平锁**
> 公平锁每次获取到锁为同步队列中的第一个节点，保证请求资源时间上的绝对顺序，而非公平锁有可能刚释放锁的线程下次继续获取该锁，则有可能导致其他线程永远无法获取到锁，造成“饥饿”现象。
> 
>公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换，而非公平锁会降低一定的上下文切换，降低性能开销。因此，ReentrantLock默认选择的是非公平锁，则是为了减少一部分上下文切换，保证了系统更大的吞吐量。

##### ReentrantReadWriteLock
在并发场景中用于解决线程安全的问题，我们几乎会高频率的使用到独占式锁，通常使用java提供的关键字synchronized或者concurrents包中实现了Lock接口的ReentrantLock。它们都是独占式获取锁，也就是在同一时刻只有一个线程能够获取锁。而在一些业务场景中，大部分只是读数据，写数据很少，如果仅仅是读数据的话并不会影响数据正确性（出现脏读），而如果在这种业务场景下，依然使用独占锁的话，很显然这将是出现性能瓶颈的地方。针对这种读多写少的情况，java还提供了另外一个实现Lock接口的ReentrantReadWriteLock(读写锁)。读写所允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞。在分析WirteLock和ReadLock的互斥性时可以按照WriteLock与WriteLock之间，WriteLock与ReadLock之间以及ReadLock与ReadLock之间进行分析。这里做一个归纳总结：

公平性选择：支持非公平性（默认）和公平的锁获取方式，吞吐量还是非公平优于公平；
重入性：支持重入，读锁获取后能再次获取，写锁获取之后能够再次获取写锁，同时也能够获取读锁；
锁降级：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁

要想能够彻底的理解读写锁必须能够理解这样几个问题：1. 读写锁是怎样实现分别记录读写状态的？2. 写锁是怎样获取和释放的？3.读锁是怎样获取和释放的？我们带着这样的三个问题，再去了解下读写锁。

###### 写锁详解

**写锁的获取**

同步组件的实现聚合了同步器（AQS），并通过重写重写同步器（AQS）中的方法实现同步组件的同步语义。因此，写锁的实现依然也是采用这种方式。在同一时刻写锁是不能被多个线程所获取，很显然写锁是独占式锁，而实现写锁的同步语义是通过重写AQS中的tryAcquire方法实现的。源码为:
```
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
	// 1. 获取写锁当前的同步状态
    int c = getState();
	// 2. 获取写锁获取的次数
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
		// 3.1 当读锁已被读线程获取或者当前线程不是已经获取写锁的线程的话
		// 当前线程获取写锁失败
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
		// 3.2 当前线程获取写锁，支持可重复加锁
        setState(c + acquires);
        return true;
    }
	// 3.3 写锁未被任何线程获取，当前线程可获取写锁
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
这段代码的逻辑请看注释，这里有一个地方需要重点关注，exclusiveCount(c)方法，该方法源码为：
```
static int exclusiveCount(int c) { 
       return c & EXCLUSIVE_MASK; 
 }
```
其中EXCLUSIVE_MASK为:  static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;      
EXCLUSIVE_MASK为1左移16位然后减1，即为0x0000FFFF。而exclusiveCount方法是将同步状态（state为int类型）与0x0000FFFF相与，即取同步状态的低16位。那么低16位代表什么呢？根据exclusiveCount方法的注释为独占式获取的次数即写锁被获取的次数，现在就可以得出来一个结论同步状态的低16位用来表示写锁的获取次数。同时还有一个方法值得我们注意：
```
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
```
该方法是获取读锁被获取的次数，是将同步状态（int c）右移16次，即取同步状态的高16位，现在我们可以得出另外一个结论同步状态的高16位用来表示读锁被获取的次数。现在还记得我们开篇说的需要弄懂的第一个问题吗？读写锁是怎样实现分别记录读锁和写锁的状态的，现在这个问题的答案就已经被我们弄清楚了，其示意图如下图所示：
![f4bab9ebca5a35df042a681e4c91c7eb](大数据成神之路-Java高级特性增强(锁).resources/46AF9BE6-A1C5-418C-9836-CDE377311CC0.png)
现在我们回过头来看写锁获取方法tryAcquire，其主要逻辑为：当读锁已经被读线程获取或者写锁已经被其他写线程获取，则写锁获取失败；否则，获取成功并支持重入，增加写状态。

**写锁的释放**
写锁释放通过重写AQS的tryRelease方法，源码为：
```
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
	//1. 同步状态减去写状态
    int nextc = getState() - releases;
	//2. 当前写状态是否为0，为0则释放写锁
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
	//3. 不为0则更新同步状态
    setState(nextc);
    return free;
}

```
源码的实现逻辑请看注释，不难理解与ReentrantLock基本一致，这里需要注意的是，减少写状态int nextc = getState() - releases;只需要用当前同步状态直接减去写状态的原因正是我们刚才所说的写状态是由同步状态的低16位表示的。

###### 读锁详解

**读锁的获取**
看完了写锁，现在来看看读锁，读锁不是独占式锁，即同一时刻该锁可以被多个读线程获取也就是一种共享式锁。按照之前对AQS介绍，实现共享式同步组件的同步语义需要通过重写AQS的tryAcquireShared方法和tryReleaseShared方法。读锁的获取实现方法为：

```
protected final int tryAcquireShared(int unused) {
    /*
     * Walkthrough:
     * 1. If write lock held by another thread, fail.
     * 2. Otherwise, this thread is eligible for
     *    lock wrt state, so ask if it should block
     *    because of queue policy. If not, try
     *    to grant by CASing state and updating count.
     *    Note that step does not check for reentrant
     *    acquires, which is postponed to full version
     *    to avoid having to check hold count in
     *    the more typical non-reentrant case.
     * 3. If step 2 fails either because thread
     *    apparently not eligible or CAS fails or count
     *    saturated, chain to version with full retry loop.
     */
    Thread current = Thread.currentThread();
    int c = getState();
	//1. 如果写锁已经被获取并且获取写锁的线程不是当前线程的话，当前
	// 线程获取读锁失败返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
		//2. 当前线程获取读锁
        compareAndSetState(c, c + SHARED_UNIT)) {
		//3. 下面的代码主要是新增的一些功能，比如getReadHoldCount()方法
		//返回当前获取读锁的次数
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
	//4. 处理在第二步中CAS操作失败的自旋已经实现重入性
    return fullTryAcquireShared(current);
}

```
代码的逻辑请看注释，需要注意的是  当写锁被其他线程获取后，读锁获取失败，否则获取成功利用CAS更新同步状态。另外，当前同步状态需要加上SHARED_UNIT（(1 << SHARED_SHIFT)即0x00010000）的原因这是我们在上面所说的同步状态的高16位用来表示读锁被获取的次数。如果CAS失败或者已经获取读锁的线程再次获取读锁时，是靠fullTryAcquireShared方法实现的，有兴趣可以看看。

**读锁的释放**
读锁释放的实现主要通过方法tryReleaseShared，源码如下，主要逻辑请看注释：
```
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
	// 前面还是为了实现getReadHoldCount等新功能
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
		// 读锁释放 将同步状态减去读状态即可
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}

```
###### 锁降级
读写锁支持锁降级，遵循按照获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁，不支持锁升级，关于锁降级下面的示例代码摘自ReentrantWriteReadLock源码中：
```
void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // Must release read lock before acquiring write lock
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // Recheck state because another thread might have
                // acquired write lock and changed state before we did.
                if (!cacheValid) {
                    data = ...
            cacheValid = true;
          }
          // Downgrade by acquiring read lock before releasing write lock
          rwl.readLock().lock();
        } finally {
          rwl.writeLock().unlock(); // Unlock write, still hold read
        }
      }
 
      try {
        use(data);
      } finally {
        rwl.readLock().unlock();
      }
    }
}
```

-----------
**参考文章和书籍：**

《Java并发编程的艺术》
《实战Java高并发程序设计》
https://blog.csdn.net/qq_34337272/article/details/79680771
https://www.jianshu.com/p/a5f99f25329a
https://www.jianshu.com/p/506c1e38a922