# Map接口

`Map`是双列集合，存储⼀组键值对，它独立于`Collection`接口。常⽤的实现类有：

- `HashMap`
- `TreeMap`
- `HashTable`
- `LinkedHashMap`

`Map`是顶层接口

- `sortedMap`接口提供排序选项

所有实现类都派⽣于`AbstractMap`抽象类

迭代处理`Map`的键、值

- `map.entrySet`方法
  - `Set<Map.Entry<Integer, Integer>> entry = map.entrySet();`
- `forEach`方法
  - `map.forEach((k, v) -> System.out.println(k + ":" + v));`

`Map.Entry<K, V>` 是 `Map` 的⼀个接口，接口的内部接口默认是`public static` 

- `Map`的实现类内部会实现`Map.Entry<K, V>` 接口，存储键值对

## HashTable

- 和 `HashMap` 类似，但它是线程安全的，即同一时刻多个线程同时写入 `HashTable` 不会导致数据不一致。
- 遗留类，不应该使用，而是使用 `ConcurrentHashMap` 来支持线程安全。
- `ConcurrentHashMap`的效率会更高，因为 `ConcurrentHashMap` 引入了分段锁、CAS和锁。

## LinkedHashMap

使用**双向链表**来维护元素的顺序，顺序为：

- 插入顺序
- 最近最少使用顺序（LRU）

## TreeMap

![image-20250407204039251](assets/image-20250407204039251.png)

⽆序，不可重复，但是可排序。

⽀持范围查找，查找最近的元素。

`TreeMap`底层基于**红黑树**实现。

- `TreeMap`内部⽆扩容概念，因为使⽤的是树的链式存储结构。
- 可以保证在`log(n)`内完成`containsKey`、`get`、`put`、`remove`操作。
- 迭代器采⽤的是**中序遍历**。

红黑树相⽐于AVL树，牺牲了部分平衡性，以换取删除/插⼊操作时少量的旋转次数，整体来说，性能优于AVL树。

- AVL树为了维护严苛的平衡条件，在破坏了平衡之后（插⼊、删除），需要执⾏旋转操作。
- 共分为四种：左单旋、先左后右旋、右单旋、先右后左旋。

## 排序

内部是按照`key`进行排序的，所以不⽀持`key`为`null`。

可以⾃动对 `String` 类型或8⼤基本类型的包装类型进⾏排序 。

⾃定义类型进⾏排序：

- 存放在集合中的⾃定义类型实现 `java.lang.Comparable` 接口，并重写 `compareTo` ⽅法。
- 使用 `TreeSet/TreeMap` 带⽐较器参数（`Comparator`）的构造器 ，并重写⽐较器中的 `compare` ⽅法。

## HashMap

### 概述

![image-20250407220037233](assets/image-20250407220037233.png)

基于**数组+链表+红黑树**实现

- 数组：`transient Node<K,V>[] table`

- 链表：`Node<K, V>`
- 红黑树：`Node<K, V>`

`HashMap`从宏观来看是基于`Hash`表实现的，通过拉链法解决哈希冲突。

引⼊红黑树，避免⼤量冲突导致某个位置链表长度过长，使得某些元素的查询效率变低。

链表是**尾插法**，保证了插⼊顺序的同时，也避免了并发操作下的⼀些异常。

- 头插法在扩容时有可能导致环形链表的出现，形成死循环。

### 存储结构

```java
package java.util;

public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
  
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16，默认初始化容量
    static final int MAXIMUM_CAPACITY = 1 << 30;				// 最大容量
    static final float DEFAULT_LOAD_FACTOR = 0.75f;			// 默认装填因子
    static final int TREEIFY_THRESHOLD = 8;							// 单链表元素超过8个，则变为红⿊树
    static final int UNTREEIFY_THRESHOLD = 6;						// 红⿊树节点数量⼩于6，则变为单链表
    static final int MIN_TREEIFY_CAPACITY = 64;					// 单链表和红⿊树相互转换前提，table数组长度(桶数量)>64

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; // 哈希值，通过key的hashcode计算出hash
        final K key; 		// final修饰
        V value;
        Node<K,V> next; // 下一个节点地址
    }
   
    transient Node<K,V>[] table; // 哈希表（链表数组）

    ......
}
```

**`HashMap` 是数组和单链表的结合体（链表数组）**

- 数组查询效率⾼，但是增删元素效率较低。
- 单链表在随机增删元素⽅⾯效率较⾼，但是查询效率较低。
- `HashMap` 将⼆者结合起来，充分它们各⾃的优点。

内部采⽤`Node`数组来表⽰`Hash`表，数组长度为2的幂次。

- `transient Node<K,V>[] table`：链表数组）
- `Node`长度为2的幂次：主要为了能通过**位运算**获取`key`的索引位置，提升计算的效率。

单链表的节点每个节点是 `Node<K, V>` 类型

- 从 `next` 字段可以看出 `Node`是一个链表。
- 数组中的每个位置被当成一个桶，一个桶存放一个链表。

前提`Node`数组容量>64时：

- 如果单链表元素超过8个，则将单链表转变为红⿊树。
- 如果红⿊树节点数量⼩于6，则将红⿊树重新变为单链表。

### 拉链法(解决冲突)

1. 取`key`的 `hashCode` 值。

2. 根据 `hashcode` 计算出`hash`值。

   1. 采⽤的是**高16位 异或  低16位**求`hash`值，从`hash`分布的情况来看，这样离散性更好。

   2. ```java
      static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }
      ```

3. 根据`hash`值，通过**位运算(取模)**：`(n - 1) & hash` 计算**桶下标**（数组下标）。

   1. 同⼀单链表中所有 `Node` 的 `hash`值不⼀定⼀样（不同`hash`值映射到同一位置），但是对应的数组下标⼀定⼀样（同一位置）。

   2. 位运算替代取模前提：`n`为2的次幂(`n-1`，二级制全是1)。

   3. 桶下标计算：`i = (n - 1) & hash` ，位运算。

   4. `(n - 1) & hash`  等同于 `hash mod n`对`n` ，位运算和取模结果一样，但是位运算性能更高。

   5. ```java
      final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {
              Node<K,V> e; K k;
            	......
      }
      ```

4. 插入链表
   - JDK1.7是头插法，扩容时可能会导致死循环。
   - JDK1.8是尾插法，避免并发插入异常。
   - 找到数组下标后，`key`和链表元素比较(`equals`)：相等则覆盖；都不相等，插入尾部。

### 扩容(Node数组容量)

装填因子默认为`0.75`，超过扩容。

`loadFactor`为`0.75`是对空间和时间效率的一个平衡选择，一般不修改，除非在时间和空间比较特殊的情况下 ：

- 如果内存空间很多，而对时间效率要求很高，可以降低`loadFactor`。
- 如果内存空间紧张，而对时间效率要求不高，可以增加`loadFactor`，这个值可以大于1。

**动态扩容**，默认是扩容为原数组长度的2倍。

- 内部没有`capacity`变量，`Node`数组⼤⼩是在扩容时确定的。
- 初始化没有指定`Node`容量⼤⼩，直接从`0`扩容为`16`（默认16）。
- 初始化时指定了`Node`容量⼤⼩，则整个`Hash`表的容量⼤⼩为最接近指定容量且⼤于指定容量的2的幂次（2的下一个幂值）。

1.8之前

- 将原有`Entry`数组的元素拷贝到新的`Entry`数组里，对所有元素`rehash`，映射到对应的位置。
- 会重新计算`hash`值。
- 会重新计算每个元素在数组中的位置。

1.8及之后

- 根据`hash & oldCap`的相与结果，来判断是否在新数组里面位移。
- 元素的位置要么是在原位置，要么是在原位置再移动2次幂位置（`oldCap` ）。

- 不需要重新计算`hash`值。

- 将`key`的`hash`值和`oldCap`相与，重新计算桶下标。

  - `oldCap` 二进制是`1000….`

- 如果为`0`，则保持原位。

- 如果为`1`，则放⼊到`原位+oldCap`的位置。

  