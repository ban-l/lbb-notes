# Set

`Set`接口也继承⾃`Collenction`。

`Set`是无序集合，并且不允许元素重复。

`Set`要保证没有重复，需要正确重写`equals()`和`hashCode()`。

- 通过`hashCode()`计算存储位置，通过`equals()`比较。

`Set`常⽤的实现类有：

- `HashSet`（散列表）
- `TreeSet`（红黑树）
- `LinkedHashSet`（双向链表, 记录插入顺序）

## HashSet

- 基于散列表实现。
- `HashSet` ⽆序（没下标），不可重复。
- 支持快速查找。
- `HashSet` 查找的时间复杂度为 `O(1)`，`TreeSet` 则为 `O(logN)。`

## TreeSet（树集）

- 基于红黑树实现。
- `TreeSet` ⽆序（没下标），不可重复。

- 支持有序性操作，可以排序 （元素实现`Comparable`接口、传入比较器对象）。

## LinkedHashSet

- 具有 `HashSet` 的查找效率，并且内部使用**双向链表**维护元素的插入顺序。

## 注意

`HashSet` 为 `HashMap` 的 key 部分。

`TreeSet` 为 `TreeMap` 的 key 部分。

## TreeSet排序(Comparable，Comparator)

两种方式实现

- 元素实现`Comparable`接口，覆写方法。
- 传入比较器对象。

```java
// 实现Comparable接口
public class Demo implements Comparable<Demo> {

    private int number;

    @Override
    public int compareTo(Demo o) {
        return this.number - o.number;
    }
}
```

```java
// 创建比较器
public class MyComparator implements Comparator<Demo2> {
    @Override
    public int compare(Demo2 o1, Demo2 o2) {
        return o1.getNumber() - o2.getNumber();
    }
}
```

```java
public class TreeSetDemo {
    public static void main(String[] args) {
        Set<Demo> set = new TreeSet<>();
        // 1.Demo实现Comparable接口，排序
        set.add(new Demo(10));
        set.add(new Demo(5));
        set.add(new Demo(25));
        set.add(new Demo(50));
        set.add(new Demo(20));
        set.forEach(i -> System.out.println(i.getNumber()));

        // 2.传递⼀个⽐较器对象给 TreeSet 构造器
        Set<Demo2> set2 = new TreeSet<>(new MyComparator());
        set2.add(new Demo2(10));
        set2.add(new Demo2(5));
        set2.add(new Demo2(25));
        set2.add(new Demo2(50));
        set2.add(new Demo2(20));
        set2.forEach(i -> System.out.println(i.getNumber()));

        // 3.使用匿名内部类(⽐较器对象)
        Set<Demo2> set3 = new TreeSet<>(new Comparator<Demo2>() {
            @Override
            public int compare(Demo2 o1, Demo2 o2) {
                return o1.getNumber() - o2.getNumber();
            }
        });
        set3.add(new Demo2(10));
        set3.add(new Demo2(5));
        set3.add(new Demo2(25));
        set3.add(new Demo2(50));
        set3.add(new Demo2(20));
        set3.forEach(i -> System.out.println(i.getNumber()));

        //  4.使⽤ lambda 表达式，传递⼀个⽐较器对象
        Set<Demo2> set4 = new TreeSet<>((o1, o2) -> o1.getNumber() - o2.getNumber());
        set4.add(new Demo2(10));
        set4.add(new Demo2(5));
        set4.add(new Demo2(25));
        set4.add(new Demo2(50));
        set4.add(new Demo2(20));
        set4.forEach(i -> System.out.println(i.getNumber()));
    }
}
```



