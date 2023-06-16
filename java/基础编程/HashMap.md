在JDK 1.7和JDK 1.8中，HashMap的实现有一些重要的不同点。下面是JDK 1.7和JDK 1.8之间HashMap实现的一些主要区别：

JDK 1.7中HashMap的实现：
1. 基于数组+链表：JDK 1.7中的HashMap使用了数组和链表的组合来实现桶（bucket）。每个桶是一个Entry链表的头节点，当多个元素哈希到同一个桶时，它们会以链表形式存储在该桶中。
2. 链表：使用单向链表来解决哈希冲突，通过equals方法进行键的比较。
3. 链表转化为红黑树：当某个桶中的链表长度达到一定阈值（默认为8），JDK 1.7中的HashMap会将链表转化为红黑树，以减少查找时间复杂度。
4. 链表遍历：在JDK 1.7中，当需要查找或删除某个键值对时，需要遍历整个链表，时间复杂度为O(n)。
5. 并发性能：在并发环境下，JDK 1.7中的HashMap性能较差，因为多个线程可能同时对同一个桶进行操作，可能导致链表成环，进而导致死循环。

JDK 1.8中HashMap的实现：
1. 基于数组+链表/红黑树：JDK 1.8中的HashMap也使用了数组和链表/红黑树的组合来实现桶。当桶中的元素超过阈值（默认为8）且数组长度超过一定阈值（默认为64）时，会将链表转换为红黑树，以提高查找效率。
2. 链表/红黑树：使用双向链表来解决哈希冲突，通过equals方法进行键的比较。红黑树用于优化桶中元素的查找效率，使得查找时间复杂度降低为O(log n)。
3. 链表转化为红黑树的条件：JDK 1.8中的HashMap在链表转化为红黑树之前，会先检查桶中的元素数量是否满足转化为红黑树的条件，如果不满足，则会对链表进行扩容或者再散列。
4. 链表遍历和红黑树查找：JDK 1.8中HashMap在链表遍历和红黑树查找方面进行了优化，使用更高效的方式进行操作，提高了性能。
5. 并发性能：JDK 1.8中的HashMap通过引入了一种新的锁机制（CAS + synchronized），提高了在并发

环境下的性能和线程安全性。

总结：
JDK 1.8中的HashMap相对于JDK 1.7进行了一些优化，包括链表转化为红黑树的条件、链表遍历和红黑树查找的效率提升以及并发性能的改进。这些改进使得JDK 1.8中的HashMap在性能和线程安全性方面更加优秀。

# 头插和尾插
在HashMap的实现中，通常使用尾插（Append）方法来处理哈希冲突。头插（Prepend）方法在HashMap中并不常见，因为它可能会导致性能下降和链表顺序的逆转。但是，头插法在其他数据结构和算法中有一些应用场景。以下是关于头插和尾插的应用场景的一些例子：

头插（Prepend）的应用场景：
1. 链表操作：在单链表中，头插法可以用于高效地插入元素。通过将新元素插入链表头部，可以在常数时间内完成插入操作，而不需要遍历整个链表。

2. 逆序构建：当需要逆序构建一个数据结构时，头插法可以很方便地实现。例如，逆序构建一个字符串，可以使用头插法将每个字符依次插入到新字符串的头部。

尾插（Append）的应用场景（HashMap中的应用场景）：
1. 处理哈希冲突：在HashMap中，哈希冲突是指多个键映射到同一个桶（数组位置）。尾插法用于将具有相同哈希值的键值对以链表或红黑树的形式存储在同一个桶中。

2. 构建有序列表：当需要构建一个有序列表时，尾插法可以保持元素的相对顺序。通过将元素追加到列表的末尾，可以保持插入顺序不变。

3. 追加操作：在某些场景下，需要在数据结构的末尾添加新元素。尾插法可以提供高效的追加操作，而不需要遍历整个数据结构来找到末尾位置。

需要注意的是，头插法和尾插法的选择取决于特定的数据结构和操作需求。在HashMap中，尾插法是常见的选择，因为它具有较好的性能和数据保持有序的能力。

## Map图
![](基础编程/assets/readme-1634103158519.png)
### LinkedHashMap
HashMap 是 Java Collection Framework 的重要成员，也是Map族(如下图所示)中
我们最为常用的一种。不过遗憾的是，HashMap是无序的，也就是说，迭代HashMap所得到的元素顺序并不是它们最初放置到HashMap的顺序。HashMap的这一
缺点往往会造成诸多不便，因为在有些场景中，我们确需要用到一个可以保持插入顺序的Map。庆幸的是，JDK为我们解决了这个问题，它为HashMap提供了一个子类
—— `LinkedHashMap`。虽然LinkedHashMap增加了时间和空间上的开销，但是它通过维护一个额外的双向链表保证了迭代顺序。特别地，该迭代顺序可以是插入顺序，
也可以是访问顺序。因此，根据链表中元素的顺序可以将LinkedHashMap分为：保持插入顺序的LinkedHashMap 和 保持访问顺序的LinkedHashMap，
其中LinkedHashMap的默认实现是按插入顺序排序的。
* 更直观地，下图很好地还原了LinkedHashMap的原貌：HashMap和双向链表的密切配合和分工合作造就了LinkedHashMap。特别需要注意的是，next用于
  维护HashMap各个桶中的Entry链，before、after用于维护LinkedHashMap的双向链表，虽然它们的作用对象都是Entry，但是各自分离，是两码事儿。

![](基础编程/assets/readme-1634103344553.png)

* 其中，HashMap与LinkedHashMap的Entry结构示意图如下图所示：

![](基础编程/assets/readme-1634103366647.png)

### SortedMap
SortedMap（java.util.SortedMap）接口是Map的子接口，SortedMap中增加了元素的排序，这意味着可以给SortedMap中的元素排序。

* NavigableMap
    * TreeMap
    * ConcurrentSkipListMap

![](基础编程/assets/readme-1634103754695.png)

#### NavigableMap
SortedMap的子接口，但是 NavigableMap接口中新加了几个SortedSet接口中没有的方法，使导航存储在映射中的键和值成为可能，本文会讲解。
既然是接口，那就必须用到它的实现，java.util包中只有一个实现 `java.util.TreeMap` ，另外java.util.concurrent包中也有实现，但是本文不讲解
```
NavigableMap original = new TreeMap();
original.put("1", "1");
original.put("2", "2");
original.put("3", "3");
```
##### TreeMap
TreeMap是`线程不安全的`，它是`红黑树`算法的实现, 红黑树又称红-黑二叉树，它首先是一颗二叉树，它具体二叉树所有的特性。同时红黑树更是一颗自平衡的排序二叉树。
*  平衡二叉树必须具备如下特性：它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。也就是说该二叉树的任何一个等等子节点，其左右子树的高度都相近。
   ![](基础编程/assets/readme-1634104376550.png)
*  红黑树顾名思义就是节点是红色或者黑色的平衡二叉树，它通过颜色的约束来维持着二叉树的平衡。对于一棵有效的红黑树二叉树而言我们必须增加如下规则：
1. 每个节点都只能是红色或者黑色
2. 根节点是黑色
3. 每个叶节点（NIL节点，空节点）是黑色的。
4. 如果一个结点是红的，则它两个子节点都是黑的。也就是说在一条路径上不能出现相邻的两个红色结点。
5. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

![](基础编程/assets/readme-1634104438402.png)
* 对于红黑二叉树而言它主要包括三大基本操作：左旋、右旋、着色。
* 左旋转
  ![](基础编程/assets/20140523092135453.gif)
* 右旋转
  ![](基础编程/assets/20140523092154062.gif)
```
SortedMap<String, String> list = new TreeMap<>(); //基于红黑树的实现，在单线程性能不错.
list.put("a", "1");
list.put("b", "2");
list.put("c", "3");
list.put("d", "4");
SortedMap<String, String> tail = list.tailMap("c");//返回大于等于c的
Iterator<String> iterator = tail.values().iterator();
while (iterator.hasNext()) {
  System.out.println(iterator.next());
}
```
##### ConcurrentSkipListMap
多线程下使用，支持更高的并发，ConcurrentSkipListMap 的存取时间是log（N），和线程数几乎无关，内部是`SkipList（跳表）`结构实现。

##### 跳跃表（SkipList）
![](基础编程/assets/readme-1644463722362.png)

1. 多条链构成，是关键字升序排列的数据结构；
2. 包含多个级别，一个head引用指向最高的级别，最低（底部）的级别，包含所有的key；
3. 每一个级别都是其更低级别的子集，并且是有序的；
4. 如果关键字 key在 级别level=i中出现，则，level<=i的链表中都会包含该关键字key；
```
ConcurrentSkipListMap<String, String> concurrentSkipListMap = new ConcurrentSkipListMap<>();
String a1 = concurrentSkipListMap.put("zzl", "zhangzhanling");
String a2 = concurrentSkipListMap.put("zzl1", "zhan1");
String a3 = concurrentSkipListMap.put("zzl2", "zhan2");
String a4 = concurrentSkipListMap.put("china", "中国");
System.out.printf(concurrentSkipListMap.toString());
```
