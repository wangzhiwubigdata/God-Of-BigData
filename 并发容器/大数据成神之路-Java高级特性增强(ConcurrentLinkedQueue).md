### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

### ConcurrentLinkedQueue介绍
ConcurrentLinkedQueue是线程安全的队列，它适用于“高并发”的场景。
它是一个基于链接节点的无界线程安全队列，按照 FIFO（先进先出）原则对元素进行排序。队列元素中不可以放置null元素（内部实现的特殊节点除外）。

### ConcurrentLinkedQueue原理和数据结构
ConcurrentLinkedQueue的数据结构，如下图所示：![2c9f34f0d8819f5a0c03ecbe99b7ca82](大数据成神之路-Java高级特性增强(ConcurrentLinkedQueue).resources/2C447958-48AF-4B02-A30E-52AA0038497C.jpg)
说明：

1. ConcurrentLinkedQueue继承于AbstractQueue。
2. ConcurrentLinkedQueue内部是通过链表来实现的。它同时包含链表的头节点head和尾节点tail。ConcurrentLinkedQueue按照FIFO（先进先出）原则对元素进行排序。元素都是从尾部插入到链表，从头部开始返回。
3. ConcurrentLinkedQueue的链表Node中的next的类型是volatile，而且链表数据item的类型也是volatile。关于volatile，我们知道它的语义包含："即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入"。ConcurrentLinkedQueue就是通过volatile来实现多线程对竞争资源的互斥访问的.

### ConcurrentLinkedQueue函数列表

```
// 创建一个最初为空的 ConcurrentLinkedQueue。
ConcurrentLinkedQueue()
// 创建一个最初包含给定 collection 元素的 ConcurrentLinkedQueue，按照此 collection 迭代器的遍历顺序来添加元素。
ConcurrentLinkedQueue(Collection<? extends E> c)

// 将指定元素插入此队列的尾部。
boolean add(E e)
// 如果此队列包含指定元素，则返回 true。
boolean contains(Object o)
// 如果此队列不包含任何元素，则返回 true。
boolean isEmpty()
// 返回在此队列元素上以恰当顺序进行迭代的迭代器。
Iterator<E> iterator()
// 将指定元素插入此队列的尾部。
boolean offer(E e)
// 获取但不移除此队列的头；如果此队列为空，则返回 null。
E peek()
// 获取并移除此队列的头，如果此队列为空，则返回 null。
E poll()
// 从队列中移除指定元素的单个实例（如果存在）。
boolean remove(Object o)
// 返回此队列中的元素数量。
int size()
// 返回以恰当顺序包含此队列所有元素的数组。
Object[] toArray()
// 返回以恰当顺序包含此队列所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
<T> T[] toArray(T[] a)
```
### ConcurrentLinkedQueue源码分析

下面从ConcurrentLinkedQueue的创建，添加，删除这几个方面对它进行分析。
**1 创建**
下面以ConcurrentLinkedQueue()来进行说明。
```
public ConcurrentLinkedQueue() {
    head = tail = new Node<E>(null);
}
```
说明：在构造函数中，新建了一个“内容为null的节点”，并设置表头head和表尾tail的值为新节点。

head和tail的定义如下：
```
private transient volatile Node<E> head;
private transient volatile Node<E> tail;
```
head和tail都是volatile类型，他们具有volatile赋予的含义：“即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入”。
Node的声明如下：
```
private static class Node<E> {
    volatile E item;
    volatile Node<E> next;

    Node(E item) {
        UNSAFE.putObject(this, itemOffset, item);
    }

    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```
说明：
Node是个单向链表节点，next用于指向下一个Node，item用于存储数据。Node中操作节点数据的API，都是通过Unsafe机制的CAS函数实现的；例如casNext()是通过CAS函数“比较并设置节点的下一个节点”。


**2. 添加**

下面以add(E e)为例对ConcurrentLinkedQueue中的添加进行说明。

```
public boolean add(E e) {
    return offer(e);
}
```
说明：add()实际上是调用的offer()来完成添加操作的。

offer()的源码如下：
```
public boolean offer(E e) {
    // 检查e是不是null，是的话抛出NullPointerException异常。
    checkNotNull(e);
    // 创建新的节点
    final Node<E> newNode = new Node<E>(e);

    // 将“新的节点”添加到链表的末尾。
    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        // 情况1：q为空
        if (q == null) {
            // CAS操作：如果“p的下一个节点为null”(即p为尾节点)，则设置p的下一个节点为newNode。
            // 如果该CAS操作成功的话，则比较“p和t”(若p不等于t，则设置newNode为新的尾节点)，然后返回true。
            // 如果该CAS操作失败，这意味着“其它线程对尾节点进行了修改”，则重新循环。
            if (p.casNext(null, newNode)) {
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
        }
        // 情况2：p和q相等
        else if (p == q)
            p = (t != (t = tail)) ? t : head;
        // 情况3：其它
        else
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```
说明：offer(E e)的作用就是将元素e添加到链表的末尾。offer()比较的地方是理解for循环，下面区分3种情况对for进行分析。

情况1 -- q为空。这意味着q是尾节点的下一个节点。此时，通过p.casNext(null, newNode)将“p的下一个节点设为newNode”，若设置成功的话，则比较“p和t”(若p不等于t，则设置newNode为新的尾节点)，然后返回true。否则的话(意味着“其它线程对尾节点进行了修改”)，什么也不做，继续进行for循环。
p.casNext(null, newNode)，是调用CAS对p进行操作。若“p的下一个节点等于null”，则设置“p的下一个节点等于newNode”；设置成功的话，返回true，失败的话返回false。

情况2 -- p和q相等。这种情况什么时候会发生呢？通过“情况3”，我们知道，经过“情况3”的处理后，p的值可能等于q。
此时，若尾节点没有发生变化的话，那么，应该是头节点发生了变化，则设置p为头节点，然后重新遍历链表；否则(尾节点变化的话)，则设置p为尾节点。

情况3 -- 其它。
我们将p = (p != t && t != (t = tail)) ? t : q;转换成如下代码。
```
if (p==t) {
    p = q;
} else {
    Node<E> tmp=t;
    t = tail;
    if (tmp==t) {
        p=q;
    } else {
        p=t;
    }
}
```
如果p和t相等，则设置p为q。否则的话，判断“尾节点是否发生变化”，没有变化的话，则设置p为q；否则，设置p为尾节点。

checkNotNull()的源码如下：

```
private static void checkNotNull(Object v) {
    if (v == null)
        throw new NullPointerException();
}
```
**3. 删除**

下面以poll()为例对ConcurrentLinkedQueue中的删除进行说明。

```
public E poll() {
    // 设置“标记”
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            // 情况1
            // 表头的数据不为null，并且“设置表头的数据为null”这个操作成功的话;
            // 则比较“p和h”(若p!=h，即表头发生了变化，则更新表头，即设置表头为p)，然后返回原表头的item值。
            if (item != null && p.casItem(item, null)) {
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            // 情况2
            // 表头的下一个节点为null，即链表只有一个“内容为null的表头节点”。则更新表头为p，并返回null。
            else if ((q = p.next) == null) {
                updateHead(h, p);
                return null;
            }
            // 情况3
            // 这可能到由于“情况4”的发生导致p=q，在该情况下跳转到restartFromHead标记重新操作。
            else if (p == q)
                continue restartFromHead;
            // 情况4
            // 设置p为q
            else
                p = q;
        }
    }
}
```
说明：poll()的作用就是删除链表的表头节点，并返回被删节点对应的值。poll()的实现原理和offer()比较类似，下面根将or循环划分为4种情况进行分析。

情况1：“表头节点的数据”不为null，并且“设置表头节点的数据为null”这个操作成功。
p.casItem(item, null) -- 调用CAS函数，比较“节点p的数据值”与item是否相等，是的话，设置节点p的数据值为null。
在情况1发生时，先比较“p和h”，若p!=h，即表头发生了变化，则调用updateHead()更新表头；然后返回删除节点的item值。
updateHead()的源码如下：
```
final void updateHead(Node<E> h, Node<E> p) {
    if (h != p && casHead(h, p))
        h.lazySetNext(h);
}
```
说明：updateHead()的最终目的是更新表头为p，并设置h的下一个节点为h本身。
casHead(h,p)是通过CAS函数设置表头，若表头等于h的话，则设置表头为p。
lazySetNext()的源码如下：
```
void lazySetNext(Node<E> val) {
    UNSAFE.putOrderedObject(this, nextOffset, val);
}
```
putOrderedObject()函数，我们在前面一章“TODO”中介绍过。h.lazySetNext(h)的作用是通过CAS函数设置h的下一个节点为h自身，该设置可能会延迟执行。

情况2：如果表头的下一个节点为null，即链表只有一个“内容为null的表头节点”。
则调用updateHead(h, p)，将表头更新p；然后返回null。

情况3：p=q
在“情况4”的发生后，会导致p=q；此时，“情况3”就会发生。当“情况3”发生后，它会跳转到restartFromHead标记重新操作。

情况4：其它情况。
设置p=q。

### ConcurrentLinkedQueue示例
```
import java.util.*;
import java.util.concurrent.*;

/*
 *   ConcurrentLinkedQueue是“线程安全”的队列，而LinkedList是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历queue”的示例
 *   (01) 当queue是ConcurrentLinkedQueue对象时，程序能正常运行。
 *   (02) 当queue是LinkedList对象时，程序会产生ConcurrentModificationException异常。
 *
 * @author skywang
 */
public class ConcurrentLinkedQueueDemo1 {

    // TODO: queue是LinkedList对象时，程序会出错。
    //private static Queue<String> queue = new LinkedList<String>();
    private static Queue<String> queue = new ConcurrentLinkedQueue<String>();
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

ta1, ta1, tb1, tb1, ta2, ta2, tb2, 
tb2, 
ta1, ta1, tb1, tb1, ta2, ta2, tb2, tb2, ta3, tb3, 
ta3, ta1, tb3, tb1, ta4, 
ta2, ta1, tb2, tb1, ta3, ta2, tb3, tb2, ta4, ta3, tb4, 
tb3, ta1, ta4, tb1, tb4, ta2, ta5, 
tb2, ta1, ta3, tb1, tb3, ta2, ta4, tb2, tb4, ta3, ta5, tb3, tb5, 
ta4, ta1, tb4, tb1, ta5, ta2, tb5, tb2, ta6, 
ta3, ta1, tb3, tb1, ta4, ta2, tb4, tb2, ta5, ta3, tb5, tb3, ta6, ta4, tb6, 
tb4, ta5, tb5, ta6, tb6,
```
结果说明：如果将源码中的queue改成LinkedList对象时，程序会产生ConcurrentModificationException异常。