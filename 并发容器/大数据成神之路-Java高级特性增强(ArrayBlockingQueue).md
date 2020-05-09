### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~

####**多线程**

###**集合框架**

###**NIO**

###**Java并发容器**

### ArrayBlockingQueue介绍
ArrayBlockingQueue是数组实现的线程安全的有界的阻塞队列。
线程安全是指，ArrayBlockingQueue内部通过“互斥锁”保护竞争资源，实现了多线程对竞争资源的互斥访问。而有界，则是指ArrayBlockingQueue对应的数组是有界限的。 阻塞队列，是指多线程访问竞争资源时，当竞争资源已被某线程获取时，其它要获取该资源的线程需要阻塞等待；而且，ArrayBlockingQueue是按 FIFO（先进先出）原则对元素进行排序，元素都是从尾部插入到队列，从头部开始返回。


注意:ArrayBlockingQueue不同于ConcurrentLinkedQueue，ArrayBlockingQueue是数组实现的，并且是有界限的;而ConcurrentLinkedQueue是链表实现的，是无界限的.

### ArrayBlockingQueue原理和数据结构
ArrayBlockingQueue的数据结构，如下图所示：![7adae419f9a6f74c644642c6bda2673b](大数据成神之路-Java高级特性增强(ArrayBlockingQueue).resources/0BA99FFA-FAC7-470F-AB00-4523EDCCFF80.jpg)
说明：    
1. ArrayBlockingQueue继承于AbstractQueue，并且它实现了BlockingQueue接口。    
2. ArrayBlockingQueue内部是通过Object[]数组保存数据的，也就是说ArrayBlockingQueue本质上是通过数组实现的。ArrayBlockingQueue的大小，即数组的容量是创建ArrayBlockingQueue时指定的。    
3. ArrayBlockingQueue与ReentrantLock是组合关系，ArrayBlockingQueue中包含一个ReentrantLock对象(lock)。ReentrantLock是可重入的互斥锁，ArrayBlockingQueue就是根据该互斥锁实现“多线程对竞争资源的互斥访问”。而且，ReentrantLock分为公平锁和非公平锁，关于具体使用公平锁还是非公平锁，在创建ArrayBlockingQueue时可以指定；而且，ArrayBlockingQueue默认会使用非公平锁。    
4. ArrayBlockingQueue与Condition是组合关系，ArrayBlockingQueue中包含两个Condition对象(notEmpty和notFull)。而且，Condition又依赖于ArrayBlockingQueue而存在，通过Condition可以实现对ArrayBlockingQueue的更精确的访问 -- (01)若某线程(线程A)要取数据时，数组正好为空，则该线程会执行notEmpty.await()进行等待；当其它某个线程(线程B)向数组中插入了数据之后，会调用notEmpty.signal()唤醒“notEmpty上的等待线程”。此时，线程A会被唤醒从而得以继续运行。(02)若某线程(线程H)要插入数据时，数组已满，则该线程会它执行notFull.await()进行等待；当其它某个线程(线程I)取出数据之后，会调用notFull.signal()唤醒“notFull上的等待线程”。此时，线程H就会被唤醒从而得以继续运行。

### ArrayBlockingQueue函数列表
```
// 创建一个带有给定的（固定）容量和默认访问策略的 ArrayBlockingQueue。
ArrayBlockingQueue(int capacity)
// 创建一个具有给定的（固定）容量和指定访问策略的 ArrayBlockingQueue。
ArrayBlockingQueue(int capacity, boolean fair)
// 创建一个具有给定的（固定）容量和指定访问策略的 ArrayBlockingQueue，它最初包含给定 collection 的元素，并以 collection 迭代器的遍历顺序添加元素。
ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c)

// 将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则抛出 IllegalStateException。
boolean add(E e)
// 自动移除此队列中的所有元素。
void clear()
// 如果此队列包含指定的元素，则返回 true。
boolean contains(Object o)
// 移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
int drainTo(Collection<? super E> c)
// 最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
int drainTo(Collection<? super E> c, int maxElements)
// 返回在此队列中的元素上按适当顺序进行迭代的迭代器。
Iterator<E> iterator()
// 将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则返回 false。
boolean offer(E e)
// 将指定的元素插入此队列的尾部，如果该队列已满，则在到达指定的等待时间之前等待可用的空间。
boolean offer(E e, long timeout, TimeUnit unit)
// 获取但不移除此队列的头；如果此队列为空，则返回 null。
E peek()
// 获取并移除此队列的头，如果此队列为空，则返回 null。
E poll()
// 获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
E poll(long timeout, TimeUnit unit)
// 将指定的元素插入此队列的尾部，如果该队列已满，则等待可用的空间。
void put(E e)
// 返回在无阻塞的理想情况下（不存在内存或资源约束）此队列能接受的其他元素数量。
int remainingCapacity()
// 从此队列中移除指定元素的单个实例（如果存在）。
boolean remove(Object o)
// 返回此队列中元素的数量。
int size()
// 获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
E take()
// 返回一个按适当顺序包含此队列中所有元素的数组。
Object[] toArray()
// 返回一个按适当顺序包含此队列中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
<T> T[] toArray(T[] a)
// 返回此 collection 的字符串表示形式。
String toString()
```
### ArrayBlockingQueue源码分析
下面从ArrayBlockingQueue的创建，添加，取出，遍历这几个方面对ArrayBlockingQueue进行分析。
**1. 创建**
下面以ArrayBlockingQueue(int capacity, boolean fair)来进行说明。
```
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```
说明：
(01) items是保存“阻塞队列”数据的数组。它的定义如下：
```
final Object[] items;
```
(02) fair是“可重入的独占锁(ReentrantLock)”的类型。fair为true，表示是公平锁；fair为false，表示是非公平锁。
notEmpty和notFull是锁的两个Condition条件。它们的定义如下：
```
final ReentrantLock lock;
private final Condition notEmpty;
private final Condition notFull;
```
Lock的作用是提供独占锁机制，来保护竞争资源；而Condition是为了更加精细的对锁进行控制，它依赖于Lock，通过某个条件对多线程进行控制。
notEmpty表示“锁的非空条件”。当某线程想从队列中取数据时，而此时又没有数据，则该线程通过notEmpty.await()进行等待；当其它线程向队列中插入了元素之后，就调用notEmpty.signal()唤醒“之前通过notEmpty.await()进入等待状态的线程”。
同理，notFull表示“锁的满条件”。当某线程想向队列中插入元素，而此时队列已满时，该线程等待；当其它线程从队列中取出元素之后，就唤醒该等待的线程。

**2. 添加**

下面以offer(E e)为例，对ArrayBlockingQueue的添加方法进行说明。
```
public boolean offer(E e) {
    // 创建插入的元素是否为null，是的话抛出NullPointerException异常
    checkNotNull(e);
    // 获取“该阻塞队列的独占锁”
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 如果队列已满，则返回false。
        if (count == items.length)
            return false;
        else {
        // 如果队列未满，则插入e，并返回true。
            insert(e);
            return true;
        }
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```
说明：offer(E e)的作用是将e插入阻塞队列的尾部。如果队列已满，则返回false，表示插入失败；否则，插入元素，并返回true。(01) count表示”队列中的元素个数“。除此之外，队列中还有另外两个遍历takeIndex和putIndex。takeIndex表示下一个被取出元素的索引，putIndex表示下一个被添加元素的索引。它们的定义如下：
```
// 下一个被添加元素的索引
int takeIndex;
// 下一个被取出元素的索引
int putIndex;
// 队列中的元素个数
int count;
```
(02) insert()的源码如下：
```
private void insert(E x) {
    // 将x添加到”队列“中
    items[putIndex] = x;
    // 设置”下一个被取出元素的索引“
    putIndex = inc(putIndex);
    // 将”队列中的元素个数”+1
    ++count;
    // 唤醒notEmpty上的等待线程
    notEmpty.signal();
}
```
insert()在插入元素之后，会唤醒notEmpty上面的等待线程。inc()的源码如下：
```
final int inc(int i) {
    return (++i == items.length) ? 0 : i;
}
```
若i+1的值等于“队列的长度”，即添加元素之后，队列满；则设置“下一个被添加元素的索引”为0。
**3. 取出**

下面以take()为例，对ArrayBlockingQueue的取出方法进行说明。
```
public E take() throws InterruptedException {
    // 获取“队列的独占锁”
    final ReentrantLock lock = this.lock;
    // 获取“锁”，若当前线程是中断状态，则抛出InterruptedException异常
    lock.lockInterruptibly();
    try {
        // 若“队列为空”，则一直等待。
        while (count == 0)
            notEmpty.await();
        // 取出元素
        return extract();
    } finally {
        // 释放“锁”
        lock.unlock();
    }
}
```
说明：take()的作用是取出并返回队列的头。若队列为空，则一直等待。
extract()的源码如下：
```
private E extract() {
    final Object[] items = this.items;
    // 强制将元素转换为“泛型E”
    E x = this.<E>cast(items[takeIndex]);
    // 将第takeIndex元素设为null，即删除。同时，帮助GC回收。
    items[takeIndex] = null;
    // 设置“下一个被取出元素的索引”
    takeIndex = inc(takeIndex);
    // 将“队列中元素数量”-1
    --count;
    // 唤醒notFull上的等待线程。
    notFull.signal();
    return x;
}
```
说明：extract()在删除元素之后，会唤醒notFull上的等待线程。

**4. 遍历**
下面对ArrayBlockingQueue的遍历方法进行说明。
```
public Iterator<E> iterator() {
    return new Itr();
}
```
Itr是实现了Iterator接口的类，它的源码如下：
```
private class Itr implements Iterator<E> {
    // 队列中剩余元素的个数
    private int remaining; // Number of elements yet to be returned
    // 下一次调用next()返回的元素的索引
    private int nextIndex; // Index of element to be returned by next
    // 下一次调用next()返回的元素
    private E nextItem;    // Element to be returned by next call to next
    // 上一次调用next()返回的元素
    private E lastItem;    // Element returned by last call to next
    // 上一次调用next()返回的元素的索引
    private int lastRet;   // Index of last element returned, or -1 if none

    Itr() {
        // 获取“阻塞队列”的锁
        final ReentrantLock lock = ArrayBlockingQueue.this.lock;
        lock.lock();
        try {
            lastRet = -1;
            if ((remaining = count) > 0)
                nextItem = itemAt(nextIndex = takeIndex);
        } finally {
            // 释放“锁”
            lock.unlock();
        }
    }

    public boolean hasNext() {
        return remaining > 0;
    }

    public E next() {
        // 获取“阻塞队列”的锁
        final ReentrantLock lock = ArrayBlockingQueue.this.lock;
        lock.lock();
        try {
            // 若“剩余元素<=0”，则抛出异常。
            if (remaining <= 0)
                throw new NoSuchElementException();
            lastRet = nextIndex;
            // 获取第nextIndex位置的元素
            E x = itemAt(nextIndex);  // check for fresher value
            if (x == null) {
                x = nextItem;         // we are forced to report old value
                lastItem = null;      // but ensure remove fails
            }
            else
                lastItem = x;
            while (--remaining > 0 && // skip over nulls
                   (nextItem = itemAt(nextIndex = inc(nextIndex))) == null)
                ;
            return x;
        } finally {
            lock.unlock();
        }
    }

    public void remove() {
        final ReentrantLock lock = ArrayBlockingQueue.this.lock;
        lock.lock();
        try {
            int i = lastRet;
            if (i == -1)
                throw new IllegalStateException();
            lastRet = -1;
            E x = lastItem;
            lastItem = null;
            // only remove if item still at index
            if (x != null && x == items[i]) {
                boolean removingHead = (i == takeIndex);
                removeAt(i);
                if (!removingHead)
                    nextIndex = dec(nextIndex);
            }
        } finally {
            lock.unlock();
        }
    }
}
```
### ArrayBlockingQueue示例
```
import java.util.*;
import java.util.concurrent.*;
/*
 *   ArrayBlockingQueue是“线程安全”的队列，而LinkedList是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历queue”的示例
 *   (01) 当queue是ArrayBlockingQueue对象时，程序能正常运行。
 *   (02) 当queue是LinkedList对象时，程序会产生ConcurrentModificationException异常。
 */
public class ArrayBlockingQueueDemo1{
    // TODO: queue是LinkedList对象时，程序会出错。
    //private static Queue<String> queue = new LinkedList<String>();
    private static Queue<String> queue = new ArrayBlockingQueue<String>(20);
    public static void main(String[] args) {
    
        // 同时启动两个线程对queue进行操作！
        new MyThread("ta").start();
        new MyThread("tb").start();
    }

    private static void printAll() {
        String value;
        Iterator iter = queue.iterator();
        while(iter.hasNext()) {
            value = (String)iter.next();
            System.out.print(value+", ");
        }
        System.out.println();
    }

    private static class MyThread extends Thread {
        MyThread(String name) {
            super(name);
        }
        @Override
        public void run() {
                int i = 0;
            while (i++ < 6) {
                // “线程名” + "-" + "序号"
                String val = Thread.currentThread().getName()+i;
                queue.add(val);
                // 通过“Iterator”遍历queue。
                printAll();
            }
        }
    }
}
```
其中一次运行结果：
```
ta1, ta1, 
tb1, ta1, 
tb1, ta1, ta2, 
tb1, ta1, ta2, tb1, tb2, 
ta2, ta1, tb2, tb1, ta3, 
ta2, ta1, tb2, tb1, ta3, ta2, tb3, 
tb2, ta1, ta3, tb1, tb3, ta2, ta4, 
tb2, ta1, ta3, tb1, tb3, ta2, ta4, tb2, tb4, 
ta3, ta1, tb3, tb1, ta4, ta2, tb4, tb2, ta5, 
ta3, ta1, tb3, tb1, ta4, ta2, tb4, tb2, ta5, ta3, tb5, 
tb3, ta1, ta4, tb1, tb4, ta2, ta5, tb2, tb5, ta3, ta6, 
tb3, ta4, tb4, ta5, tb5, ta6, tb6,
```
结果说明：如果将源码中的queue改成LinkedList对象时，程序会产生ConcurrentModificationException异常。
