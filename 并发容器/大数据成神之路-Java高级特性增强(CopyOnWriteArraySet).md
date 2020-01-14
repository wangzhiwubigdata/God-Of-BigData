### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

#### 概要
内容包括：
* CopyOnWriteArraySet介绍
* CopyOnWriteArraySet原理和数据结构
* CopyOnWriteArraySet函数列表
* CopyOnWriteArraySet源码
* CopyOnWriteArraySet示例


#### CopyOnWriteArraySet介绍
它是线程安全的无序的集合，可以将它理解成线程安全的HashSet。有意思的是，CopyOnWriteArraySet和HashSet虽然都继承于共同的父类AbstractSet；但是，HashSet是通过“散列表(HashMap)”实现的，而CopyOnWriteArraySet则是通过“动态数组(CopyOnWriteArrayList)”实现的，并不是散列表。
和CopyOnWriteArrayList类似，CopyOnWriteArraySet具有以下特性：
1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

建议：在学习CopyOnWriteArraySet之前,先对HashSet进行了解。

#### CopyOnWriteArraySet原理和数据结构

CopyOnWriteArraySet的数据结构，如下图所示：![126a88369f7024046d46da2fb20bba03](大数据成神之路-Java高级特性增强(CopyOnWriteArraySet).resources/F023A78F-7010-495C-B34F-FBCDE1801AF8.jpg)
说明：  
1. CopyOnWriteArraySet继承于AbstractSet，这就意味着它是一个集合。  
2. CopyOnWriteArraySet包含CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。而CopyOnWriteArrayList本质是个动态数组队列，所以CopyOnWriteArraySet相当于通过通过动态数组实现的“集合”！ CopyOnWriteArrayList中允许有重复的元素；但是,CopyOnWriteArraySet是一个集合,所以它不能有重复集合。因此,CopyOnWriteArrayList额外提供了addIfAbsent()和addAllAbsent()这两个添加元素的API,通过这些API来添加元素时，只有当元素不存在时才执行添加操作！   
至于CopyOnWriteArraySet的"线程安全"机制,和CopyOnWriteArrayList一样,是通过volatile和互斥锁来实现的。这个在前一章节介绍CopyOnWriteArrayList时数据结构时,已经进行了说明，这里就不再重复叙述了。

#### CopyOnWriteArraySet函数列表
```
// 创建一个空 set。
CopyOnWriteArraySet()
// 创建一个包含指定 collection 所有元素的 set。
CopyOnWriteArraySet(Collection<? extends E> c)

// 如果指定元素并不存在于此 set 中，则添加它。
boolean add(E e)
// 如果此 set 中没有指定 collection 中的所有元素，则将它们都添加到此 set 中。
boolean addAll(Collection<? extends E> c)
// 移除此 set 中的所有元素。
void clear()
// 如果此 set 包含指定元素，则返回 true。
boolean contains(Object o)
// 如果此 set 包含指定 collection 的所有元素，则返回 true。
boolean containsAll(Collection<?> c)
// 比较指定对象与此 set 的相等性。
boolean equals(Object o)
// 如果此 set 不包含任何元素，则返回 true。
boolean isEmpty()
// 返回按照元素添加顺序在此 set 中包含的元素上进行迭代的迭代器。
Iterator<E> iterator()
// 如果指定元素存在于此 set 中，则将其移除。
boolean remove(Object o)
// 移除此 set 中包含在指定 collection 中的所有元素。
boolean removeAll(Collection<?> c)
// 仅保留此 set 中那些包含在指定 collection 中的元素。
boolean retainAll(Collection<?> c)
// 返回此 set 中的元素数目。
int size()
// 返回一个包含此 set 所有元素的数组。
Object[] toArray()
// 返回一个包含此 set 所有元素的数组；返回数组的运行时类型是指定数组的类型。
<T> T[] toArray(T[] a)
```
#### CopyOnWriteArraySet示例
CopyOnWriteArraySet是通过CopyOnWriteArrayList实现的，它的API基本上都是通过调用CopyOnWriteArrayList的API来实现的。相信对CopyOnWriteArrayList了解的话，对CopyOnWriteArraySet的了解是水到渠成的事。
下面，我们通过一个例子去对比HashSet和CopyOnWriteArraySet。
```
import java.util.*;
import java.util.concurrent.*;

/*
 *   CopyOnWriteArraySet是“线程安全”的集合，而HashSet是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历集合set”的示例
 *   (01) 当set是CopyOnWriteArraySet对象时，程序能正常运行。
 *   (02) 当set是HashSet对象时，程序会产生ConcurrentModificationException异常。
 *
 */
public class CopyOnWriteArraySetTest1 {

    // TODO: set是HashSet对象时，程序会出错。
    //private static Set<String> set = new HashSet<String>();
    private static Set<String> set = new CopyOnWriteArraySet<String>();
    public static void main(String[] args) {
    
        // 同时启动两个线程对set进行操作！
        new MyThread("ta").start();
        new MyThread("tb").start();
    }

    private static void printAll() {
        String value = null;
        Iterator iter = set.iterator();
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
            while (i++ < 10) {
                // “线程名” + "-" + "序号"
                String val = Thread.currentThread().getName() + "-" + (i%6);
                set.add(val);
                // 通过“Iterator”遍历set。
                printAll();
            }
        }
    }
}
```
其中一次运行结果：
```
ta-1, tb-1, ta-1, 
tb-1, ta-1, 
tb-1, ta-1, ta-2, 
tb-1, ta-1, ta-2, tb-1, tb-2, 
ta-2, ta-1, tb-2, tb-1, ta-3, 
ta-2, ta-1, tb-2, tb-1, ta-3, ta-2, tb-3, 
tb-2, ta-1, ta-3, tb-1, tb-3, ta-2, ta-4, 
tb-2, ta-1, ta-3, tb-1, tb-3, ta-2, ta-4, tb-2, tb-4, 
ta-3, ta-1, tb-3, tb-1, ta-4, ta-2, tb-4, tb-2, ta-5, 
ta-3, ta-1, tb-3, tb-1, ta-4, ta-2, tb-4, tb-2, ta-5, ta-3, tb-5, 
tb-3, ta-1, ta-4, tb-1, tb-4, ta-2, ta-5, tb-2, tb-5, ta-3, ta-0, 
tb-3, ta-1, ta-4, tb-1, tb-4, ta-2, ta-5, tb-2, tb-5, ta-3, ta-0, tb-3, tb-0, 
ta-4, ta-1, tb-4, tb-1, ta-5, ta-2, tb-5, tb-2, ta-0, ta-3, tb-0, 
tb-3, ta-1, ta-4, tb-1, tb-4, ta-2, ta-5, tb-5, ta-0, tb-0, 
ta-1, tb-2, tb-1, ta-3, ta-2, tb-3, tb-2, ta-4, ta-3, tb-4, tb-3, ta-5, ta-4, tb-5, tb-4, ta-0, ta-5, tb-0, 
tb-5, ta-1, ta-0, tb-1, tb-0, 
ta-2, ta-1, tb-2, tb-1, ta-3, ta-2, tb-3, tb-2, ta-4, ta-3, tb-4, tb-3, ta-5, tb-5, ta-0, tb-0, 
ta-4, ta-1, tb-4, tb-1, ta-5, ta-2, tb-5, tb-2, ta-0, ta-3, tb-0, 
tb-3, ta-1, ta-4, tb-1, tb-4, ta-2, ta-5, tb-2, tb-5, ta-3, ta-0, tb-3, tb-0, 
ta-4, tb-4, ta-5, tb-5, ta-0, tb-0,
```
结果说明：
由于set是集合对象,因此它不会包含重复的元素。
如果将源码中的set改成HashSet对象时,程序会产生ConcurrentModificationException异常。