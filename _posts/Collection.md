---
　　layout: default
　　title: 集合类
---
<FONT style="FONT-FAMILY: 微软雅黑;line-height:25px;" >
##**集合**
***

<font color=red>**1. ArrayList、LinkedList、Vector的底层实现和区别**</font>
* ArrayList和Vector：底层是用数组实现的，支持快速随机访问。默认容量都为10。扩容操作ArrayList扩容至原来的1.5倍，Vector扩容至原来的2倍
**1. Vector：**线程安全的，其实现是在所有的方法上加上了`synchronized`关键字，效率很低。
**2. ArrayList：** 线程不安全，不是同步的，在Iterator或ListIterator上调用容器的add或remove方法进行修改，会抛出`ConcurrentModificationException`并发修改异常。
**3. ArrayList线程安全的实现方式：**
1. **CopyOnWriteArrayList:**在线程对集合进行写操作时会拷贝一个新的数组，完成后再将引用移到新的列表上，旧的列表如果在使用。仍然有效。不会抛出`ConcurrentModificationException`。适合“<font color='blue'>读多写少</font>”环境下使用。存在的问题：内存占用问题；数据一致性问题。
2. **Collections.synchronizedList:**建立了List的包装类实现线程安全，`synchronizedList`继承自`SynchronizedCollection`，是<font color=blue>在synchronized同步块内使用信号量</font>来实现线程安全的，但是iterator()操作不是线程安全的，扔会抛出`ConcurrentModificationException`。读操作性能不如`CopyOnWriteArrayList`好。

* **LinkedList：**双向循环链表实现，适合于**没有大量随机访问操作，删除和增加频繁**
<br>**ArrayList&LinkedList：**
**时间消耗： **在列表末尾添加和删除一个元素所花的开销都是固定的，在中间添加或删除元素，ArrayList列表剩余的元素都需要被移动，而LinkedList花销固定
**空间消耗： **ArrayList的空间浪费主要体现在在列表结尾预留一定的容量空间，而LinkedList空间浪费主要体现在它的每一个Node节点都需要有一个pre和next指针。

<font color=red>**2. HashMap 和 HashTable 的底层实现和区别，两者和 ConcurrentHashMap 的区别**</font>
* **HashMap：**底层使用 **数组+单链表** 实现，Table数组默认容量为16，装载因子为0.75。存放Entry对象，每一个Entry对象有key、value还有next属性，next属性指向下一个Entry，用于解决哈希冲突。使用put()和set()方法向HashMap中增加和更新entry，首先计算key的哈希值，然后找到对应的桶，使用equals()判断桶内是否有对应的entry。发生冲突时，将新增加的entry链入桶内链表的头部。当entry数大于初始容量*装载因子后，会通过rehash()进行扩容——初始容量翻倍。提供三种集合视图，分别是`KeySet, Values, EntrySet`。线程不安全：使用三种集合视图上的Iterator进行add、remove修改会抛异常（与ArrayList一样的）。允许插入null-null的键值对。
<br>
*  **HashTable：**线程安全，通过在方法上加Synchronized关键字实现，效率特别低。一次锁住整个Hash表。不允许插入null-null的键值对。
<br>
* **HashMap线程安全的其他实现方式：**
1. **`java.util.ConcurrentHashMap`：**<font color=blue>Node中的value和next属性都加了volatile关键字，使用分段锁，一次锁一个桶。</font>
2. 使用**`Map m = Collections.synchronizedMap(new HashMap(...))`**，对HashMap做了一次封装，在synchronized代码块中使用信号量机制来实现线程安全。

<font color=red>**3. HashMap 的 hashcode 的作用？什么时候需要重写？如何解决哈希冲突？查找的时候流程是如何？**
* **Hashcode()的作用：**如果没有hashcode，那么比较hashmap中是否存在某一个entry则需要使用equals操作逐个entry进行比较，这样效率就很低。在hashmap中先调用一个hash()计算hashcode，然后判断对应的桶中是否存在entry，存在的话在逐个进行equals()比较。
* **什么时候需要重写：**Object中的`hashcode()`方法返回的是对象实例的内存地址。如果要改变判断对象间是否相等的依据，就需要重写`equals()和hashcode()`。Java对象相同是两个对象通过`equals()`判断的结果返回true。<font color=blue>相等的对象必须具有相同的哈希值，如果两个对象哈希值相同它们并不一定相等</font>
* **如何解决哈希冲突：**采用链表解决哈希冲突，HashMap中存放的是entry对象，每一个entry都有一个next指针指向下一个entry对象，如果put一个对象时，该桶内已经存在了entry，那么把新对象链入链表的头部。
* **查找的时候流程如何：**先计算待查找的key的哈希值，然后找到table数组中对应的桶，如果桶内没有entry，则查找失败，如果有entry的话，再逐一进行equals比较。

<font color=red>**4. TreeMap、HashMap、LinkedHashMap的底层实现区别**
* **LinkedHashMap底层实现：**继承自HashMap，<font color=blue>可以保留插入的顺序</font>。
1. LinkedHashMap中的Entry继承HashMap中的Entry，除了有key、value、next指针外，还有before和after两个指针用来记录上一个元素和下一个元素，用来维护一个双向链表，保存插入的顺序。
2. 在LinkedHashMap中有一个<font color=blue>boolean型的成员变量accessOrder用来定义排序模式</font>，默认为false——插入顺序，也可以设置为true——访问顺序。
3. LinkedHashMap中有一个<font color=blue>head和tail首尾的entry节点</font>，没有放到table数组中，是独立出来的。
4. 设置`accessOrder=true，removeEldestEntry=true`时，可以实现LRUCache问题，它是按照近期访问最少到近期访问最多的顺序进行排序的。
[LinkedHashMap源码分析](http://uule.iteye.com/blog/1522291)
* **TreeMap底层实现：**排好序的map，底层是<font color=blue>红黑树实现</font>，Entry实现了Map.Entry，每一个Entry对象都有key、value、left、right、parent属性。可以指定比较器Comparator，或者key实现Comparable接口。key不能为null。
[TreeMap源码分析](http://www.cnblogs.com/hzmark/archive/2013/01/02/TreeMap-Base.html)
> **红黑树**
1. 每个节点或为红色、或为黑色
2. 根节点是黑色的
3. 如果一个节点是红色，那么它的两个孩子必须是黑色的
4. 根节点到每一个哨兵节点的路径上必须包含相同数量的黑节点
**红黑树并不是严格的平衡二叉树，它并不能严格的保证左右子树的高度差不超过1，但红黑树高度依然是平均log(n) ，且最坏高度不会超过2log(n)，所以它算是平衡树。**
```
public interface Comparable<T> {
	public int compareTo(T o);
}
public interface Comparator<T>{
	int compare(T o1, T o2);
	boolean equals(Object obj);
}
```
> **Comparable接口与Comparator接口比较**
 实现Comparable接口的类表示该类是可以与其他类进行比较的类，而实现Comparator的类表示该类是一个比较器。
 
 <font color=red>**4. Collection 包结构的组成， Map 、 Set 等内部接口的特点与用法。**
![Alt text](./集合框架结构.JPG)






