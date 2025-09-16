# this 逃逸

## 1. 核心概念：什么是 “this 逃逸”？

**“this 逃逸” 指的是在对象的构造函数尚未完全执行完毕（即对象未处于一个稳定、一致的状态）时，该对象的引用（`this`）就被其他线程或代码所见或使用。**

简单来说，就是一个“未完全建好的对象”被泄露了出去。

## 2. 为什么 “this 逃逸” 是危险的？

在 Java 中，创建一个对象（`new MyClass()`）不是一个原子操作，它大致分为三步：
1.  分配内存空间。
2.  执行构造函数（初始化字段、执行构造函数体内的逻辑）。
3.  将引用指向分配的内存地址。

JVM 为了优化性能，可能会指令重排序，导致步骤 2 和 3 的顺序不确定。这意味着，**在其他线程看来，一个对象的引用可能已经存在（非`null`），但其内部字段可能还是默认值（如 `0`, `null`），而不是构造函数中设置的初始值。**

如果在构造函数中就将 `this` 泄露出去，另一个线程可能拿到了一个“半成品”对象，并试图使用它，从而导致不可预料的错误、数据不一致或程序崩溃。

## 3. “this 逃逸” 的常见场景

### 场景一：在构造函数中启动线程

这是最经典也是最危险的例子。

```java
public class ThisEscapeExample {
    private int value;

    public ThisEscapeExample() {
        // 1. 此时 value 还是默认值 0
        this.value = 42; // 2. 设置 value 为 42

        // 错误：在构造函数中启动线程，并将 this 隐式传递给了 Runnable
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                // 这个内部类隐式持有外部类 (ThisEscapeExample) 的引用
                // 新线程可能在任何时候开始执行 run 方法
                // 它可能看到 value 是 0，而不是 42！
                System.out.println("Value is: " + value); // 可能输出 0
                doSomething();
            }
        });
        thread.start(); // 3. 启动新线程

        // ... 构造函数继续执行其他操作
    }

    private void doSomething() {
        // 使用 this 的其他方法
    }
}
```
**问题分析：**
当你执行 `new ThisEscapeExample()` 时，构造函数内部的 `new Thread(...).start()` 会立即启动一个新线程。

这个新线程的 `Runnable` 内部类捕获了外部类 `ThisEscapeExample.this` 的引用。**JVM 无法保证新线程在看到 `this` 引用时，构造函数已经执行完毕（`value` 已经被正确初始化为 42）**。

新线程完全有可能看到一个 `value` 仍为 0 的“残缺”对象。

### 场景二：在构造函数中发布内部类

```java
public class EventSource {
    public void registerListener(EventListener listener) { /* ... */ }
}

public interface EventListener {
    void onEvent(Event e);
}

public class ThisEscapeExample2 {
    private final int id;

    public ThisEscapeExample2(EventSource source) {
        this.id = 12345;

        // 错误：将内部类实例发布了出去，内部类隐含了对外部类 this 的引用
        source.registerListener(new EventListener() {
            @Override
            public void onEvent(Event e) {
                // 内部类可以访问外部类的 id
                System.out.println("ID: " + id); // 可能看到 id 是 0 而不是 12345
            }
        });
    }
}
```
**问题分析：**
在构造函数中，你将一个匿名内部类实例注册到了 `EventSource` 中。

这个内部类实例在创建时就已经持有了外部类 `ThisEscapeExample2.this` 的引用。

如果 `EventSource` 在另一个线程中回调 `onEvent` 方法，它拿到的 `this` 引用所指向的对象可能还没有完成初始化（`id` 可能还是 0）。

### 场景三：在构造函数中将 `this` 赋值给公共静态变量或共享变量

```java
public class ThisEscapeExample3 {
    public static ThisEscapeExample3 leakedRef;
    private String name;

    public ThisEscapeExample3() {
        leakedRef = this; // 危险！静态变量全局可见
        // 模拟一些耗时的初始化操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) { /* ... */ }
        this.name = "Fully Constructed";
    }

    public String getName() {
        return name;
    }
}

// 另一个线程
// 可能看到 leakedRef 不为 null，但调用 leakedRef.getName() 却返回 null
```

## 4. 如何避免 “this 逃逸”？

核心原则：**不要在构造函数中将 `this` 引用泄露给外部世界。**

**1. 工厂方法模式（最常用和推荐的方法）**

将对象的构造和“可能会泄露 this”的初始化操作分离开。先完全构造好对象，然后再去做启动线程、注册监听器等操作。

```java
public class SafeExample {
    private int value;
    // 将构造函数私有化，防止外部直接 new
    private SafeExample() {
        this.value = 42;
        // 构造函数只做简单的赋值，不做任何“危险”操作
    }

    // 提供一个公共的工厂方法来创建对象
    public static SafeExample newInstance() {
        SafeExample instance = new SafeExample(); // 1. 安全地完成构造
        instance.init(); // 2. 构造完成后，再执行初始化操作
        return instance;
    }

    // 初始化方法，用于执行那些可能泄露 this 的操作
    private void init() {
        // 现在 this 已经是一个完全构建好的对象了
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Safe Value: " + value); // 保证看到 42
            }
        });
        thread.start();
    }
}

// 使用方式：SafeExample safe = SafeExample.newInstance();
```

**2. 私有构造函数 + 静态工厂方法**

如果对象必须要在构造过程中就与某些资源绑定，可以借助一个私有的构造函数和一个静态工厂方法，确保在发布引用前对象已经完全初始化。

```java
public class SafeEventExample {
    private final int id;

    // 私有构造函数，防止外部调用
    private SafeEventExample(int id, EventSource source) {
        this.id = id;
        // 注意：这里仍然没有注册监听器
    }

    public static SafeEventExample newInstance(EventSource source) {
        SafeEventExample instance = new SafeEventExample(12345, source);
        // 对象已完全构建，现在可以安全地注册了
        source.registerListener(instance.new SafeEventListener());
        return instance;
    }

    // 使用一个命名的内部类，而不是匿名内部类，逻辑更清晰
    private class SafeEventListener implements EventListener {
        @Override
        public void onEvent(Event e) {
            // 现在访问 id 绝对是安全的
            System.out.println("Safe ID: " + id);
        }
    }
}
```

## 总结

| 特性 | “this 逃逸” |
| :--- | :--- |
| **本质** | 在对象未完成构造前，其 `this` 引用被其他代码所见 |
| **危险** | 其他线程可能看到对象处于不一致的状态（字段未初始化），导致不可预知的行为 |
| **根源** | 对象构造非原子性 + JVM 指令重排序优化 |
| **避免方法** | **不要在构造函数中泄露 `this`**。使用**工厂方法模式**，将对象的**创建**与**初始化**（启动线程、注册监听器等）分离开。 |
