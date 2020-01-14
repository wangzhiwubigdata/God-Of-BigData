### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

### LinkedBlockingDeque介绍
LinkedBlockingDeque是双向链表实现的双向并发阻塞队列。该阻塞队列同时支持FIFO和FILO两种操作方式，即可以从队列的头和尾同时操作(插入/删除)；并且，该阻塞队列是支持线程安全。

此外，LinkedBlockingDeque还是可选容量的(防止过度膨胀)，即可以指定队列的容量。如果不指定，默认容量大小等于Integer.MAX_VALUE。

### LinkedBlockingDeque原理和数据结构
LinkedBlockingDeque的数据结构，如下图所示：


![6da00030f98f048da20fa09de9784f74](大数据成神之路-Java高级特性增强(LinkedBlockingDeque).resources/385B998A-3394-4B44-AE6B-B66F7775E0A4.jpg)
说明：1. LinkedBlockingDeque继承于AbstractQueue，它本质上是一个支持FIFO和FILO的双向的队列。
2. LinkedBlockingDeque实现了BlockingDeque接口，它支持多线程并发。当多线程竞争同一个资源时，某线程获取到该资源之后，其它线程需要阻塞等待。
3. LinkedBlockingDeque是通过双向链表实现的。
3.1 first是双向链表的表头。
3.2 last是双向链表的表尾。
3.3 count是LinkedBlockingDeque的实际大小，即双向链表中当前节点个数。
3.4 capacity是LinkedBlockingDeque的容量，它是在创建LinkedBlockingDeque时指定的。
3.5 lock是控制对LinkedBlockingDeque的互斥锁，当多个线程竞争同时访问LinkedBlockingDeque时，某线程获取到了互斥锁lock，其它线程则需要阻塞等待，直到该线程释放lock，其它线程才有机会获取lock从而获取cpu执行权。
3.6 notEmpty和notFull分别是“非空条件”和“未满条件”。通过它们能够更加细腻进行并发控制。
### LinkedBlockingDeque函数列表
```
// 创建一个容量为 Integer.MAX_VALUE 的 LinkedBlockingDeque。
LinkedBlockingDeque()
// 创建一个容量为 Integer.MAX_VALUE 的 LinkedBlockingDeque，最初包含给定 collection 的元素，以该 collection 迭代器的遍历顺序添加。
LinkedBlockingDeque(Collection<? extends E> c)
// 创建一个具有给定（固定）容量的 LinkedBlockingDeque。
LinkedBlockingDeque(int capacity)

// 在不违反容量限制的情况下，将指定的元素插入此双端队列的末尾。
boolean add(E e)
// 如果立即可行且不违反容量限制，则将指定的元素插入此双端队列的开头；如果当前没有空间可用，则抛出 IllegalStateException。
void addFirst(E e)
// 如果立即可行且不违反容量限制，则将指定的元素插入此双端队列的末尾；如果当前没有空间可用，则抛出 IllegalStateException。
void addLast(E e)
// 以原子方式 (atomically) 从此双端队列移除所有元素。
void clear()
// 如果此双端队列包含指定的元素，则返回 true。
boolean contains(Object o)
// 返回在此双端队列的元素上以逆向连续顺序进行迭代的迭代器。
Iterator<E> descendingIterator()
// 移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
int drainTo(Collection<? super E> c)
// 最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
int drainTo(Collection<? super E> c, int maxElements)
// 获取但不移除此双端队列表示的队列的头部。
E element()
// 获取，但不移除此双端队列的第一个元素。
E getFirst()
// 获取，但不移除此双端队列的最后一个元素。
E getLast()
// 返回在此双端队列元素上以恰当顺序进行迭代的迭代器。
Iterator<E> iterator()
// 如果立即可行且不违反容量限制，则将指定的元素插入此双端队列表示的队列中（即此双端队列的尾部），并在成功时返回 true；如果当前没有空间可用，则返回 false。
boolean offer(E e)
// 将指定的元素插入此双端队列表示的队列中（即此双端队列的尾部），必要时将在指定的等待时间内一直等待可用空间。
boolean offer(E e, long timeout, TimeUnit unit)
// 如果立即可行且不违反容量限制，则将指定的元素插入此双端队列的开头，并在成功时返回 true；如果当前没有空间可用，则返回 false。
boolean offerFirst(E e)
// 将指定的元素插入此双端队列的开头，必要时将在指定的等待时间内等待可用空间。
boolean offerFirst(E e, long timeout, TimeUnit unit)
// 如果立即可行且不违反容量限制，则将指定的元素插入此双端队列的末尾，并在成功时返回 true；如果当前没有空间可用，则返回 false。
boolean offerLast(E e)
// 将指定的元素插入此双端队列的末尾，必要时将在指定的等待时间内等待可用空间。
boolean offerLast(E e, long timeout, TimeUnit unit)
// 获取但不移除此双端队列表示的队列的头部（即此双端队列的第一个元素）；如果此双端队列为空，则返回 null。
E peek()
// 获取，但不移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
E peekFirst()
// 获取，但不移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。
E peekLast()
// 获取并移除此双端队列表示的队列的头部（即此双端队列的第一个元素）；如果此双端队列为空，则返回 null。
E poll()
// 获取并移除此双端队列表示的队列的头部（即此双端队列的第一个元素），如有必要将在指定的等待时间内等待可用元素。
E poll(long timeout, TimeUnit unit)
// 获取并移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
E pollFirst()
// 获取并移除此双端队列的第一个元素，必要时将在指定的等待时间等待可用元素。
E pollFirst(long timeout, TimeUnit unit)
// 获取并移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。
E pollLast()
// 获取并移除此双端队列的最后一个元素，必要时将在指定的等待时间内等待可用元素。
E pollLast(long timeout, TimeUnit unit)
// 从此双端队列所表示的堆栈中弹出一个元素。
E pop()
// 将元素推入此双端队列表示的栈。
void push(E e)
// 将指定的元素插入此双端队列表示的队列中（即此双端队列的尾部），必要时将一直等待可用空间。
void put(E e)
// 将指定的元素插入此双端队列的开头，必要时将一直等待可用空间。
void putFirst(E e)
// 将指定的元素插入此双端队列的末尾，必要时将一直等待可用空间。
void putLast(E e)
// 返回理想情况下（没有内存和资源约束）此双端队列可不受阻塞地接受的额外元素数。
int remainingCapacity()
// 获取并移除此双端队列表示的队列的头部。
E remove()
// 从此双端队列移除第一次出现的指定元素。
boolean remove(Object o)
// 获取并移除此双端队列第一个元素。
E removeFirst()
// 从此双端队列移除第一次出现的指定元素。
boolean removeFirstOccurrence(Object o)
// 获取并移除此双端队列的最后一个元素。
E removeLast()
// 从此双端队列移除最后一次出现的指定元素。
boolean removeLastOccurrence(Object o)
// 返回此双端队列中的元素数。
int size()
// 获取并移除此双端队列表示的队列的头部（即此双端队列的第一个元素），必要时将一直等待可用元素。
E take()
// 获取并移除此双端队列的第一个元素，必要时将一直等待可用元素。
E takeFirst()
// 获取并移除此双端队列的最后一个元素，必要时将一直等待可用元素。
E takeLast()
// 返回以恰当顺序（从第一个元素到最后一个元素）包含此双端队列所有元素的数组。
Object[] toArray()
// 返回以恰当顺序包含此双端队列所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
<T> T[] toArray(T[] a)
// 返回此 collection 的字符串表示形式。
String toString()
```
### LinkedBlockingDeque源码分析
下面从ArrayBlockingQueue的创建，添加，取出，遍历这几个方面对LinkedBlockingDeque进行分析

**1. 创建**

下面以LinkedBlockingDeque(int capacity)来进行说明。
```
public LinkedBlockingDeque(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
}
```
说明：capacity是“链式阻塞队列”的容量。

LinkedBlockingDeque中相关的数据结果定义如下：
```
// “双向队列”的表头
transient Node<E> first;
// “双向队列”的表尾
transient Node<E> last;
// 节点数量
private transient int count;
// 容量
private final int capacity;
// 互斥锁 , 互斥锁对应的“非空条件notEmpty”, 互斥锁对应的“未满条件notFull”
final ReentrantLock lock = new ReentrantLock();
private final Condition notEmpty = lock.newCondition();
private final Condition notFull = lock.newCondition();
```
说明：lock是互斥锁，用于控制多线程对LinkedBlockingDeque中元素的互斥访问；而notEmpty和notFull是与lock绑定的条件，它们用于实现对多线程更精确的控制。

双向链表的节点Node的定义如下：
```
static final class Node<E> {
    E item;       // 数据
    Node<E> prev; // 前一节点
    Node<E> next; // 后一节点

    Node(E x) { item = x; }
}
```
2. 添加

下面以offer(E e)为例，对LinkedBlockingDeque的添加方法进行说明。
```
public boolean offer(E e) {
    return offerLast(e);
}
```
offer()实际上是调用offerLast()将元素添加到队列的末尾。
offerLast()的源码如下：
```
public boolean offerLast(E e) {
    if (e == null) throw new NullPointerException();
    // 新建节点
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    // 获取锁
    lock.lock();
    try {
        // 将“新节点”添加到双向链表的末尾
        return linkLast(node);
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```
说明：offerLast()的作用，是新建节点并将该节点插入到双向链表的末尾。它在插入节点前，会获取锁；操作完毕，再释放锁。

linkLast()的源码如下：
```
private boolean linkLast(Node<E> node) {
    // 如果“双向链表的节点数量” > “容量”，则返回false，表示插入失败。
    if (count >= capacity)
        return false;
    // 将“node添加到链表末尾”，并设置node为新的尾节点
    Node<E> l = last;
    node.prev = l;
    last = node;
    if (first == null)
        first = node;
    else
        l.next = node;
    // 将“节点数量”+1
    ++count;
    // 插入节点之后，唤醒notEmpty上的等待线程。
    notEmpty.signal();
    return true;
}
```
说明：linkLast()的作用，是将节点插入到双向队列的末尾；插入节点之后，唤醒notEmpty上的等待线程。

**3. 删除**

下面以take()为例，对LinkedBlockingDeque的取出方法进行说明。
```
public E take() throws InterruptedException {
    return takeFirst();
}
```
take()实际上是调用takeFirst()队列的第一个元素。
takeFirst()的源码如下：
```
public E takeFirst() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    // 获取锁
    lock.lock();
    try {
        E x;
        // 若“队列为空”，则一直等待。否则，通过unlinkFirst()删除第一个节点。
        while ( (x = unlinkFirst()) == null)
            notEmpty.await();
        return x;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```
说明：takeFirst()的作用，是删除双向链表的第一个节点，并返回节点对应的值。它在插入节点前，会获取锁；操作完毕，再释放锁。

unlinkFirst()的源码如下：
```
private E unlinkFirst() {
    // assert lock.isHeldByCurrentThread();
    Node<E> f = first;
    if (f == null)
        return null;
    // 删除并更新“第一个节点”
    Node<E> n = f.next;
    E item = f.item;
    f.item = null;
    f.next = f; // help GC
    first = n;
    if (n == null)
        last = null;
    else
        n.prev = null;
    // 将“节点数量”-1
    --count;
    // 删除节点之后，唤醒notFull上的等待线程。
    notFull.signal();
    return item;
}
```
说明：unlinkFirst()的作用，是将双向队列的第一个节点删除；删除节点之后，唤醒notFull上的等待线程。

4. 遍历
下面对LinkedBlockingDeque的遍历方法进行说明。

```
public Iterator<E> iterator() {
    return new Itr();
}
```
iterator()实际上是返回一个Iter对象。
Itr类的定义如下：
```
private class Itr extends AbstractItr {
    // “双向队列”的表头
    Node<E> firstNode() { return first; }
    // 获取“节点n的下一个节点”
    Node<E> nextNode(Node<E> n) { return n.next; }
}
```
Itr继承于AbstractItr，而AbstractItr的定义如下：
```
private abstract class AbstractItr implements Iterator<E> {
    // next是下一次调用next()会返回的节点。
    Node<E> next;
    // nextItem是next()返回节点对应的数据。
    E nextItem;
    // 上一次next()返回的节点。
    private Node<E> lastRet;
    // 返回第一个节点
    abstract Node<E> firstNode();
    // 返回下一个节点
    abstract Node<E> nextNode(Node<E> n);

    AbstractItr() {
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        // 获取“LinkedBlockingDeque的互斥锁”
        lock.lock();
        try {
            // 获取“双向队列”的表头
            next = firstNode();
            // 获取表头对应的数据
            nextItem = (next == null) ? null : next.item;
        } finally {
            // 释放“LinkedBlockingDeque的互斥锁”
            lock.unlock();
        }
    }

    // 获取n的后继节点
    private Node<E> succ(Node<E> n) {
        // Chains of deleted nodes ending in null or self-links
        // are possible if multiple interior nodes are removed.
        for (;;) {
            Node<E> s = nextNode(n);
            if (s == null)
                return null;
            else if (s.item != null)
                return s;
            else if (s == n)
                return firstNode();
            else
                n = s;
        }
    }

    // 更新next和nextItem。
    void advance() {
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        lock.lock();
        try {
            // assert next != null;
            next = succ(next);
            nextItem = (next == null) ? null : next.item;
        } finally {
            lock.unlock();
        }
    }

    // 返回“下一个节点是否为null”
    public boolean hasNext() {
        return next != null;
    }

    // 返回下一个节点
    public E next() {
        if (next == null)
            throw new NoSuchElementException();
        lastRet = next;
        E x = nextItem;
        advance();
        return x;
    }

    // 删除下一个节点
    public void remove() {
        Node<E> n = lastRet;
        if (n == null)
            throw new IllegalStateException();
        lastRet = null;
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        lock.lock();
        try {
            if (n.item != null)
                unlink(n);
        } finally {
            lock.unlock();
        }
    }
}
```
### LinkedBlockingDeque示例

```
import java.util.*;
import java.util.concurrent.*;

/*
 *   LinkedBlockingDeque是“线程安全”的队列，而LinkedList是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历queue”的示例
 *   (01) 当queue是LinkedBlockingDeque对象时，程序能正常运行。
 *   (02) 当queue是LinkedList对象时，程序会产生ConcurrentModificationException异常。
 *
 */
public class LinkedBlockingDequeDemo1 {

    // TODO: queue是LinkedList对象时，程序会出错。
    //private static Queue<String> queue = new LinkedList<String>();
    private static Queue<String> queue = new LinkedBlockingDeque<String>();
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
ta1, ta1, tb1, tb1,

ta1, ta1, tb1, tb1, tb2, tb2, ta2, 
ta2, 
ta1, ta1, tb1, tb1, tb2, tb2, ta2, ta2, tb3, tb3, ta3, 
ta3, ta1, 
tb1, ta1, tb2, tb1, ta2, tb2, tb3, ta2, ta3, tb3, tb4, ta3, ta4, 
tb4, ta1, ta4, tb1, tb5, 
tb2, ta1, ta2, tb1, tb3, tb2, ta3, ta2, tb4, tb3, ta4, ta3, tb5, tb4, ta5, 
ta4, ta1, tb5, tb1, ta5, tb2, tb6, 
ta2, ta1, tb3, tb1, ta3, tb2, tb4, ta2, ta4, tb3, tb5, ta3, ta5, tb4, tb6, ta4, ta6, 
tb5, ta5, tb6, ta6,
```
结果说明：示例程序中，启动两个线程(线程ta和线程tb)分别对LinkedBlockingDeque进行操作。以线程ta而言，它会先获取“线程名”+“序号”，然后将该字符串添加到LinkedBlockingDeque中；接着，遍历并输出LinkedBlockingDeque中的全部元素。 线程tb的操作和线程ta一样，只不过线程tb的名字和线程ta的名字不同。
当queue是LinkedBlockingDeque对象时，程序能正常运行。如果将queue改为LinkedList时，程序会产生ConcurrentModificationException异常。