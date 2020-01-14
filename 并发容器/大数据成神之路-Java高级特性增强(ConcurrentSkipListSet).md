### **Java高级特性增强-并发容器**
本部分网络上有大量的资源可以参考，在这里做了部分整理并做了大量勘误，感谢前辈的付出，每节文章末尾有引用列表~
####**多线程**
###**集合框架**
###**NIO**
###**Java并发容器**

#### ConcurrentSkipListSet介绍
ConcurrentSkipListSet是线程安全的有序的集合，适用于高并发的场景。
ConcurrentSkipListSet和TreeSet，它们虽然都是有序的集合。但是，
第一，它们的线程安全机制不同，TreeSet是非线程安全的，而ConcurrentSkipListSet是线程安全的。
第二，ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，而TreeSet是通过TreeMap实现的。

#### ConcurrentSkipListSet原理和数据结构
ConcurrentSkipListSet的数据结构，如下图所示：![ceaa5eae278d00721ece58625b3b45a0](大数据成神之路-Java高级特性增强(ConcurrentSkipListSet).resources/1378EC09-5D7F-4544-AA4C-F43796D9A609.jpg)
说明：

(01) ConcurrentSkipListSet继承于AbstractSet。因此，它本质上是一个集合。
(02) ConcurrentSkipListSet实现了NavigableSet接口。因此，ConcurrentSkipListSet是一个有序的集合。
(03) ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的。它包含一个ConcurrentNavigableMap对象m，而m对象实际上是ConcurrentNavigableMap的实现类ConcurrentSkipListMap的实例。ConcurrentSkipListMap中的元素是key-value键值对；而ConcurrentSkipListSet是集合，它只用到了ConcurrentSkipListMap中的key！
#### ConcurrentSkipListSet函数列表
```
// 构造一个新的空 set，该 set 按照元素的自然顺序对其进行排序。
ConcurrentSkipListSet()
// 构造一个包含指定 collection 中元素的新 set，这个新 set 按照元素的自然顺序对其进行排序。
ConcurrentSkipListSet(Collection<? extends E> c)
// 构造一个新的空 set，该 set 按照指定的比较器对其元素进行排序。
ConcurrentSkipListSet(Comparator<? super E> comparator)
// 构造一个新 set，该 set 所包含的元素与指定的有序 set 包含的元素相同，使用的顺序也相同。
ConcurrentSkipListSet(SortedSet<E> s)

// 如果此 set 中不包含指定元素，则添加指定元素。
boolean add(E e)
// 返回此 set 中大于等于给定元素的最小元素；如果不存在这样的元素，则返回 null。
E ceiling(E e)
// 从此 set 中移除所有元素。
void clear()
// 返回此 ConcurrentSkipListSet 实例的浅表副本。
ConcurrentSkipListSet<E> clone()
// 返回对此 set 中的元素进行排序的比较器；如果此 set 使用其元素的自然顺序，则返回 null。
Comparator<? super E> comparator()
// 如果此 set 包含指定的元素，则返回 true。
boolean contains(Object o)
// 返回在此 set 的元素上以降序进行迭代的迭代器。
Iterator<E> descendingIterator()
// 返回此 set 中所包含元素的逆序视图。
NavigableSet<E> descendingSet()
// 比较指定对象与此 set 的相等性。
boolean equals(Object o)
// 返回此 set 中当前第一个（最低）元素。
E first()
// 返回此 set 中小于等于给定元素的最大元素；如果不存在这样的元素，则返回 null。
E floor(E e)
// 返回此 set 的部分视图，其元素严格小于 toElement。
NavigableSet<E> headSet(E toElement)
// 返回此 set 的部分视图，其元素小于（或等于，如果 inclusive 为 true）toElement。
NavigableSet<E> headSet(E toElement, boolean inclusive)
// 返回此 set 中严格大于给定元素的最小元素；如果不存在这样的元素，则返回 null。
E higher(E e)
// 如果此 set 不包含任何元素，则返回 true。
boolean isEmpty()
// 返回在此 set 的元素上以升序进行迭代的迭代器。
Iterator<E> iterator()
// 返回此 set 中当前最后一个（最高）元素。
E last()
// 返回此 set 中严格小于给定元素的最大元素；如果不存在这样的元素，则返回 null。
E lower(E e)
// 获取并移除第一个（最低）元素；如果此 set 为空，则返回 null。
E pollFirst()
// 获取并移除最后一个（最高）元素；如果此 set 为空，则返回 null。
E pollLast()
// 如果此 set 中存在指定的元素，则将其移除。
boolean remove(Object o)
// 从此 set 中移除包含在指定 collection 中的所有元素。
boolean removeAll(Collection<?> c)
// 返回此 set 中的元素数目。
int size()
// 返回此 set 的部分视图，其元素范围从 fromElement 到 toElement。
NavigableSet<E> subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive)
// 返回此 set 的部分视图，其元素从 fromElement（包括）到 toElement（不包括）。
NavigableSet<E> subSet(E fromElement, E toElement)
// 返回此 set 的部分视图，其元素大于等于 fromElement。
NavigableSet<E> tailSet(E fromElement)
// 返回此 set 的部分视图，其元素大于（或等于，如果 inclusive 为 true）fromElement。
NavigableSet<E> tailSet(E fromElement, boolean inclusive)
```
#### ConcurrentSkipListSet示例
ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，它的接口基本上都是通过调用ConcurrentSkipListMap接口来实现的
```
import java.util.*;
import java.util.concurrent.*;

/*
 *   ConcurrentSkipListSet是“线程安全”的集合，而TreeSet是非线程安全的。
 *
 *   下面是“多个线程同时操作并且遍历集合set”的示例
 *   (01) 当set是ConcurrentSkipListSet对象时，程序能正常运行。
 *   (02) 当set是TreeSet对象时，程序会产生ConcurrentModificationException异常。
 *
 * @author skywang
 */
public class ConcurrentSkipListSetDemo1 {

    // TODO: set是TreeSet对象时，程序会出错。
    //private static Set<String> set = new TreeSet<String>();
    private static Set<String> set = new ConcurrentSkipListSet<String>();
    public static void main(String[] args) {
    
        // 同时启动两个线程对set进行操作！
        new MyThread("a").start();
        new MyThread("b").start();
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
                // “线程名” + "序号"
                String val = Thread.currentThread().getName() + (i%6);
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
a1, b1, 
a1, a1, a2, b1, 
b1, a1, a2, a3, b1,

a1, a2, a3, a1, a4, b1, b2, 
a2, a1, a2, a3, a4, a5, b1, b2, 
a3, a0, a4, a5, a1, b1, a2, b2, 
a3, a0, a4, a1, a5, a2, b1, a3, b2, a4, b3, 
a5, a0, b1, a1, b2, a2, b3, 
a3, a0, a4, a1, a5, a2, b1, a3, b2, a4, b3, a5, b4, 
b1, a0, b2, a1, b3, a2, b4, 
a3, a0, a4, a1, a5, a2, b1, a3, b2, a4, b3, a5, b4, b1, b5, 
b2, a0, a1, a2, a3, a4, a5, b3, b1, b4, b2, b5, 
b3, a0, b4, a1, b5, 
a2, a0, a3, a1, a4, a2, a5, a3, b0, a4, b1, a5, b2, b0, b3, b1, b4, b2, b5, b3, 
b4, a0, b5, 
a1, a2, a3, a4, a5, b0, b1, b2, b3, b4, b5, 
a0, a1, a2, a3, a4, a5, b0, b1, b2, b3, b4, b5, 
a0, a1, a2, a3, a4, a5, b0, b1, b2, b3, b4, b5, 
a0, a1, a2, a3, a4, a5, b0, b1, b2, b3, b4, b5,
```
结果说明：
示例程序中，启动两个线程(线程a和线程b)分别对ConcurrentSkipListSet进行操作。以线程a而言，它会先获取“线程名”+“序号”，然后将该字符串添加到ConcurrentSkipListSet集合中；接着，遍历并输出集合中的全部元素。 线程b的操作和线程a一样，只不过线程b的名字和线程a的名字不同。
当set是ConcurrentSkipListSet对象时，程序能正常运行。如果将set改为TreeSet时，程序会产生ConcurrentModificationException异常。
