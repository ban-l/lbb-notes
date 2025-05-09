# 迭代器遍历问题

## 快速失败（fail—fast）

迭代器遍历一个集合时，如果遍历过程中对集合对象的内容进行了修改(增加、删除、修改)，则会抛出`Concurrent Modification Exception`

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

原理：迭代器遍历集合时，在遍历过程中使用一个 `modCount` 变量。

- 集合在被遍历期间如果内容发生变化，就会改变`modCount`的值。
- 每当迭代器使用`hashNext`()/`next()`遍历下一个元素之前，都会检测`modCount`变量是否为 `expectedmodCount`值
  - 相等继续遍历。
  - 否则抛出异常，终止遍历。

注意：异常抛出条件是检测到`modCount！=expectedmodCount` 这个条件

- 如果集合发生变化时修改了`modCount`值，但存在并发线程又把它设置为了`expectedmodCount`值，则异常不会抛出。
- 因此不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的`bug`。

场景：`java.util`包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。

## 安全失败（fail-safe）

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

原理：迭代时是对原集合的拷贝进行遍历

- 所以在遍历过程中对原集合所做的修改并不能被迭代器检测到，所以不会触发`ConcurrentModificationException`。

优点：避免了`ConcurrentModificationException`

缺点：迭代器不能访问到修改后的内容，在遍历期间迭代器不知道原集合发生的修改。

场景：`java.util.concurrent`包下的容器都是安全失败，可以在多线程下并发使用。