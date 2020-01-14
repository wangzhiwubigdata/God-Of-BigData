### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

* * *
## Java并发容器大纲
我将JUC包中的集合类划分为3部分来进行说明。在简单的了解JUC包中集合类的框架之后，后面的章节再逐步对各个类进行介绍。

#### List和Set

JUC(java.util.concurrent)集合包中的List和Set实现类包括: 
**CopyOnWriteArrayList
CopyOnWriteArraySet
ConcurrentSkipListSet**
ConcurrentSkipListSet稍后在说明Map时再说明,
CopyOnWriteArrayList和CopyOnWriteArraySet的框架如下图所示：
![1a78b3149b2e4a75eecdc8659514d771](大数据成神之路-Java高级特性增强(并发容器大纲).resources/9AB2EEAE-5F35-4C72-9953-E24DB769027D.jpg)

CopyOnWriteArrayList相当于线程安全的ArrayList,它实现了List接口。CopyOnWriteArrayList是支持高并发的。
CopyOnWriteArraySet相当于线程安全的HashSet,它继承于AbstractSet类。CopyOnWriteArraySet内部包含一个CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。

#### Map
JUC集合包中Map的实现类包括: ConcurrentHashMap和ConcurrentSkipListMap。它们的框架如下图所示：
![4187acf99165786a1cf403125c521dd4](大数据成神之路-Java高级特性增强(并发容器大纲).resources/95BA2E89-44BB-461D-A8FA-9C62C4E65FE3.jpg)

* ConcurrentHashMap是线程安全的哈希表(相当于线程安全的HashMap)；它继承于AbstractMap类，并且实现ConcurrentMap接口。ConcurrentHashMap是通过“锁分段”来实现的，它支持并发。

* ConcurrentSkipListMap是线程安全的有序的哈希表(相当于线程安全的TreeMap); 它继承于AbstractMap类，并且实现ConcurrentNavigableMap接口。ConcurrentSkipListMap是通过“跳表”来实现的，它支持并发。

* ConcurrentSkipListSet是线程安全的有序的集合(相当于线程安全的TreeSet)；它继承于AbstractSet，并实现了NavigableSet接口。ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，它也支持并发。

#### Queue

JUC集合包中Queue的实现类包括: ArrayBlockingQueue, LinkedBlockingQueue, LinkedBlockingDeque, ConcurrentLinkedQueue和ConcurrentLinkedDeque。它们的框架如下图所示：
![9e1832eddfe3346bc85588364e25fcf2](大数据成神之路-Java高级特性增强(并发容器大纲).resources/E30FE17C-4587-43C9-9DF3-7C0891408761.jpg)

* ArrayBlockingQueue是数组实现的线程安全的有界的阻塞队列。

* LinkedBlockingQueue是单向链表实现的(指定大小)阻塞队列，该队列按 FIFO（先进先出）排序元素。

* LinkedBlockingDeque是双向链表实现的(指定大小)双向并发阻塞队列，该阻塞队列同时支持FIFO和FILO两种操作方式。

* ConcurrentLinkedQueue是单向链表实现的无界队列，该队列按 FIFO（先进先出）排序元素。

* ConcurrentLinkedDeque是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式。

接下来
Here We Go ~