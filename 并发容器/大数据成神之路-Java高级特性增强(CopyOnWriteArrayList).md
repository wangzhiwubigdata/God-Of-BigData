### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

* * *
## 概要
本章是"并发容器"的CopyOnWriteArrayList篇。接下来，会先对CopyOnWriteArrayList进行基本介绍，然后再说明它的原理，接着通过代码去分析，最后通过示例更进一步的了解CopyOnWriteArrayList。内容包括：

* CopyOnWriteArrayList介绍
* CopyOnWriteArrayList原理和数据结构
* CopyOnWriteArrayList函数列表
* CopyOnWriteArrayList源码分析(JDK1.7.0_40版本)
* CopyOnWriteArrayList示例


## CopyOnWriteArrayList介绍
它相当于线程安全的ArrayList。和ArrayList一样，它是个可变数组；但是和ArrayList不同的时，它具有以下特性：
1. 它最适合于具有以下特征的应用程序：List 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作。
5. 使用迭代器进行遍历的速度很快,并且不会与其他线程发生冲突。在构造迭代器时,迭代器依赖于不变的数组快照。
建议：在学习CopyOnWriteArraySet之前,先对ArrayList进行了解！ 

#### CopyOnWriteArrayList原理和数据结构
CopyOnWriteArrayList的数据结构，如下图所示：![8c9a554d0e197a027a35fe2f0d483b81](大数据成神之路-Java高级特性增强(CopyOnWriteArrayList).resources/D89A47F9-4C4F-4E57-BAE1-A314892BFFD8.jpg)
说明：
1.CopyOnWriteArrayList实现了List接口,因此它是一个队列。
2.CopyOnWriteArrayList包含了成员lock。每一个CopyOnWriteArrayList都和一个互斥锁lock绑定,通过lock，实现了对CopyOnWriteArrayList的互斥访问。
3. CopyOnWriteArrayList包含了成员array数组,这说明CopyOnWriteArrayList本质上通过数组实现的。
下面从“动态数组”和“线程安全”两个方面进一步对CopyOnWriteArrayList的原理进行说明。
1. **CopyOnWriteArrayList的“动态数组”机制** -- 它内部有个“volatile数组”(array)来保持数据。在“添加/修改/删除”数据时，都会新建一个数组，并将更新后的数据拷贝到新建的数组中，最后再将该数组赋值给“volatile数组”。这就是它叫做CopyOnWriteArrayList的原因！CopyOnWriteArrayList就是通过这种方式实现的动态数组；不过正由于它在“添加/修改/删除”数据时，都会新建数组，所以涉及到修改数据的操作，CopyOnWriteArrayList效率很低；但是单单只是进行遍历查找的话，效率比较高。
2. **CopyOnWriteArrayList的“线程安全”机制** -- 是通过volatile和互斥锁来实现的。(01) CopyOnWriteArrayList是通过"volatile数组"来保存数据的。一个线程读取volatile数组时，总能看到其它线程对该volatile变量最后的写入;就这样，通过volatile提供了"读取到的数据总是最新的"这个机制的保证。(02) CopyOnWriteArrayList通过互斥锁来保护数据。在"添加/修改/删除"数据时，会先"获取互斥锁",再修改完毕之后，先将数据更新到“volatile数组”中，然后再"释放互斥锁",这样,就达到了保护数据的目的。 

#### CopyOnWriteArrayList函数列表
```
// 创建一个空列表。
CopyOnWriteArrayList()
// 创建一个按 collection 的迭代器返回元素的顺序包含指定 collection 元素的列表。
CopyOnWriteArrayList(Collection&lt;? extends E&gt; c)
// CopyOnWriteArrayList(E[] toCopyIn)
创建一个保存给定数组的副本的列表。
// 将指定元素添加到此列表的尾部。
boolean add(E e)
// 在此列表的指定位置上插入指定元素。
void add(int index, E element)
// 按照指定 collection 的迭代器返回元素的顺序，将指定 collection 中的所有元素添加此列表的尾部。
boolean addAll(Collection&lt;? extends E&gt; c)
// 从指定位置开始，将指定 collection 的所有元素插入此列表。
boolean addAll(int index, Collection&lt;? extends E&gt; c)
// 按照指定 collection 的迭代器返回元素的顺序，将指定 collection 中尚未包含在此列表中的所有元素添加列表的尾部。
int addAllAbsent(Collection&lt;? extends E&gt; c)
// 添加元素（如果不存在）。
boolean addIfAbsent(E e)
// 从此列表移除所有元素。
void clear()
// 返回此列表的浅表副本。
Object clone()
// 如果此列表包含指定的元素，则返回 true。
boolean contains(Object o)
// 如果此列表包含指定 collection 的所有元素，则返回 true。
boolean containsAll(Collection&lt;?&gt; c)
// 比较指定对象与此列表的相等性。
boolean equals(Object o)
// 返回列表中指定位置的元素。
E get(int index)
// 返回此列表的哈希码值。
int hashCode()
// 返回第一次出现的指定元素在此列表中的索引，从 index 开始向前搜索，如果没有找到该元素，则返回 -1。
int indexOf(E e, int index)
// 返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。
int indexOf(Object o)
// 如果此列表不包含任何元素，则返回 true。
boolean isEmpty()
// 返回以恰当顺序在此列表元素上进行迭代的迭代器。
Iterator&lt;E&gt; iterator()
// 返回最后一次出现的指定元素在此列表中的索引，从 index 开始向后搜索，如果没有找到该元素，则返回 -1。
int lastIndexOf(E e, int index)
// 返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。
int lastIndexOf(Object o)
// 返回此列表元素的列表迭代器（按适当顺序）。
ListIterator&lt;E&gt; listIterator()
// 返回列表中元素的列表迭代器（按适当顺序），从列表的指定位置开始。
ListIterator&lt;E&gt; listIterator(int index)
// 移除此列表指定位置上的元素。
E remove(int index)
// 从此列表移除第一次出现的指定元素（如果存在）。
boolean remove(Object o)
// 从此列表移除所有包含在指定 collection 中的元素。
boolean removeAll(Collection&lt;?&gt; c)
// 只保留此列表中包含在指定 collection 中的元素。
boolean retainAll(Collection&lt;?&gt; c)
// 用指定的元素替代此列表指定位置上的元素。
E set(int index, E element)
// 返回此列表中的元素数。
int size()
// 返回此列表中 fromIndex（包括）和 toIndex（不包括）之间部分的视图。
List&lt;E&gt; subList(int fromIndex, int toIndex)
// 返回一个按恰当顺序（从第一个元素到最后一个元素）包含此列表中所有元素的数组。
Object[] toArray()
// 返回以恰当顺序（从第一个元素到最后一个元素）包含列表所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
&lt;T&gt; T[] toArray(T[] a)
// 返回此列表的字符串表示形式。
String toString()
```
#### CopyOnWriteArrayList源码分析
下面我们从"创建,添加,删除,获取,遍历"这5个方面去分析CopyOnWriteArrayList的原理。
**1. 创建**
CopyOnWriteArrayList共3个构造函数。它们的源码如下：
```
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}

public CopyOnWriteArrayList(Collection&lt;? extends E&gt; c) {
    Object[] elements = c.toArray();
    if (elements.getClass() != Object[].class)
        elements = Arrays.copyOf(elements, elements.length, Object[].class);
    setArray(elements);
}

public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```
说明：这3个构造函数都调用了setArray()，setArray()的源码如下：
```
private volatile transient Object[] array;
final Object[] getArray() {
    return array;
}
final void setArray(Object[] a) {
    array = a;
}
```
说明：setArray()的作用是给array赋值；其中，array是volatile transient Object[]类型，即array是“volatile数组”。关于volatile关键字，我们知道“volatile能让变量变得可见”，即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。正在由于这种特性，每次更新了“volatile数组”之后，其它线程都能看到对它所做的更新。关于transient关键字，它是在序列化中才起作用，transient变量不会被自动序列化。
**2. 添加**
以add(E e)为例，来对"CopyOnWriteArrayList"的添加操作进行说明。下面是add(E e)的代码:
```
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 获取“锁”
    lock.lock();
    try {
        // 获取原始”volatile数组“中的数据和数据长度。
        Object[] elements = getArray();
        int len = elements.length;
        // 新建一个数组newElements，并将原始数据拷贝到newElements中；
        // newElements数组的长度=“原始数组的长度”+1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 将“新增加的元素”保存到newElements中。
        newElements[len] = e;
        // 将newElements赋值给”volatile数组“。
        setArray(newElements);
        return true;
    } finally {
        // 释放“锁”
        lock.unlock();
    }
}
```
**说明:** add(E e)的作用就是将数据e添加到”volatile数组“中。它的实现方式是,新建一个数组,接着将原始的”volatile数组“的数据拷贝到新数组中，然后将新增数据也添加到新数组中:最后,将新数组赋值给”volatile数组“。在add(E e)中有两点需要关注。
第一,在”添加操作“开始前，获取独占锁(lock)，若此时有需要线程要获取锁，则必须等待；在操作完毕后，释放独占锁(lock)，此时其它线程才能获取锁。通过独占锁，来防止多线程同时修改数据！lock的定义如下：
```
transient final ReentrantLock lock = 
new ReentrantLock();  
```
第二,操作完毕时，会通过setArray()来更新”volatile数组“。而且，前面我们提过”即对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入“；这样，每次添加元素之后，其它线程都能看到新添加的元素。 
**3. 获取**
以get(int index)为例，来对“CopyOnWriteArrayList的删除操作”进行说明。下面是get(int index)的代码：![51e409b11aa51c150090697429a953ed](大数据成神之路-Java高级特性增强(CopyOnWriteArrayList).resources/465A838B-6367-4535-96B8-C9693E5D0B61.gif)
public E get(int index) {
    return get(getArray(), index);
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
![51e409b11aa51c150090697429a953ed](大数据成神之路-Java高级特性增强(CopyOnWriteArrayList).resources/465A838B-6367-4535-96B8-C9693E5D0B61.gif)
说明：get(int index)的实现很简单，就是返回"volatile数组"中的第index个元素。 
**4. 删除**
以remove(int index)为例，来对“CopyOnWriteArrayList的删除操作”进行说明。下面是remove(int index)的代码：
```
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    // 获取“锁”
    lock.lock();
    try {
        // 获取原始”volatile数组“中的数据和数据长度。
        Object[] elements = getArray();
        int len = elements.length;
        // 获取elements数组中的第index个数据。
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        // 如果被删除的是最后一个元素，则直接通过Arrays.copyOf()进行处理，而不需要新建数组。
        // 否则，新建数组，然后将”volatile数组中被删除元素之外的其它元素“拷贝到新数组中；最后，将新数组赋值给”volatile数组“。
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        // 释放“锁”
        lock.unlock();
    }
}
```
**说明：**remove(int index)的作用就是将”volatile数组“中第index个元素删除。它的实现方式是，如果被删除的是最后一个元素，则直接通过Arrays.copyOf()进行处理，而不需要新建数组。否则，新建数组，然后将"volatile数组中被删除元素之外的其它元素"拷贝到新数组中;最后,将新数组赋值给"volatile数组"。和add(E e)一样，remove(int index)也是”在操作之前，获取独占锁；操作完成之后，释放独占是“;并且"在操作完成时，会通过将数据更新到volatile数组中"。 
**5. 遍历**
以iterator()为例，来对CopyOnWriteArrayList的遍历操作进行说明。
下面是iterator()的代码：
```
public Iterator<E> iterator() {
    return new COWIterator<E>(getArray(), 0);
}
```
说明：iterator()会返回COWIterator对象。COWIterator实现额ListIterator接口，它的源码如下：
```
private static class COWIterator&lt;E&gt; implements ListIterator&lt;E&gt; {
    private final Object[] snapshot;
    private int cursor;

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    public boolean hasNext() {
        return cursor &lt; snapshot.length;
    }

    public boolean hasPrevious() {
        return cursor &gt; 0;
    }

    // 获取下一个元素
    @SuppressWarnings("unchecked")
    public E next() {
        if (! hasNext())
            throw new NoSuchElementException();
        return (E) snapshot[cursor++];
    }

    // 获取上一个元素
    @SuppressWarnings("unchecked")
    public E previous() {
        if (! hasPrevious())
            throw new NoSuchElementException();
        return (E) snapshot[--cursor];
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void remove() {
        throw new UnsupportedOperationException();
    }

    public void set(E e) {
        throw new UnsupportedOperationException();
    }

    public void add(E e) {
        throw new UnsupportedOperationException();
    }
}
```
说明：COWIterator不支持修改元素的操作。例如，对于remove(),set(),add()等操作,COWIterator都会抛出异常！另外，需要提到的一点是，CopyOnWriteArrayList返回迭代器不会抛出ConcurrentModificationException异常，即它不是fail-fast机制的！

#### CopyOnWriteArrayList示例
下面，我们通过一个例子去对比ArrayList和CopyOnWriteArrayList。
```
import java.util.*;
import java.util.concurrent.*;

/*
 *   CopyOnWriteArrayList是“线程安全”的动态数组，而ArrayList是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历list”的示例
 *   (01) 当list是CopyOnWriteArrayList对象时，程序能正常运行。
 *   (02) 当list是ArrayList对象时，程序会产生ConcurrentModificationException异常。
 *
 */
public class CopyOnWriteArrayListTest1 {

    // TODO: list是ArrayList对象时，程序会出错。
    //private static List<String> list = new ArrayList<String>();
    private static List<String> list = new CopyOnWriteArrayList<String>();
    public static void main(String[] args) {
    
        // 同时启动两个线程对list进行操作！
        new MyThread("ta").start();
        new MyThread("tb").start();
    }

    private static void printAll() {
        String value = null;
        Iterator iter = list.iterator();
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
                String val = Thread.currentThread().getName()+"-"+i;
                list.add(val);
                // 通过“Iterator”遍历List。
                printAll();
            }
        }
    }
}
```
其中一次运行结果：
```
ta-1, tb-1, ta-1, 
tb-1, 
ta-1, ta-1, tb-1, tb-1, tb-2, 
tb-2, ta-1, ta-2, 
tb-1, ta-1, tb-2, tb-1, ta-2, tb-2, tb-3, 
ta-2, ta-1, tb-3, tb-1, ta-3, 
tb-2, ta-1, ta-2, tb-1, tb-3, tb-2, ta-3, ta-2, tb-4, 
tb-3, ta-1, ta-3, tb-1, tb-4, tb-2, ta-4, 
ta-2, ta-1, tb-3, tb-1, ta-3, tb-2, tb-4, ta-2, ta-4, tb-3, tb-5, 
ta-3, ta-1, tb-4, tb-1, ta-4, tb-2, tb-5, ta-2, ta-5, 
tb-3, ta-1, ta-3, tb-1, tb-4, tb-2, ta-4, ta-2, tb-5, tb-3, ta-5, ta-3, tb-6, 
tb-4, ta-4, tb-5, ta-5, tb-6, ta-6,
```
结果说明：如果将源码中的list改成ArrayList对象时，程序会产生ConcurrentModificationException异常。