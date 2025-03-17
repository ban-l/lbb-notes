# 接口与lambda表达式
# 接口
接口，用来描述类应该做什么，而不指定具体如何做（细节），是对希望符合（继承）这个接口的类的一组需求。

- 接口方法自动`public abstract`
- 接口常量自动`public static final`
- **接口绝不会有实例字段** 
  - 提供实例字段和方法实现的应该由接口的实现类完成
  - 接口可以看成没有实例字段的抽象类

- `interface`，`implements `关键字，可以实现多个接口

- 接口不可以使用`new`实例化，可以声明接口的变量，引用接口实现类的对象(向上转型)
- `instanceof `检测一个对象是否实现了某个特定接口

## 接口版本差异

- Java8前，接口不会实现方法
- Java8中，接口中可以添加静态方法、默认方法
- Java9中，接口方法可以是`private`（只能在接口中使用），辅助方法

```java
public interface InterfaceTest {
    int TEST = 1; // 常量 默认 public static final

    void test(); // 方法 默认 public abstract

    default void defaultTest() { // 默认方法
        // 默认方法可以调用其它方法
        test();
        System.out.println("test2");
        staticTest();
    }

    static void staticTest() { // 静态方法
        System.out.println("test3");
    }
}

```

## 默认方法（可不实现，防止源代码兼容问题）

Java8 开始，可以为接口方法提供一个默认实现（默认方法），使用`default`修饰符标记方法（可不实现）

- 默认方法可以调用其它方法

- 接口增加方法可能导致“**源代码兼容**”问题（接口实现类没有实现），使用默认方法，不会导致这个问题
- 解决默认方法冲突
  - 超类优先，超类提供了一个具体方法，同名同参的默认方法被忽略 
  - 接口冲突，覆写方法解决冲突

# lambda表达式（代码块，**闭包**）

**lambda表达式是一个代码块 ，以及必须传入代码的变量规范。**

lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）

使用lambda表达式可以使代码变的更加简洁紧凑。

**lambda表达式形式：参数  箭头(->)  表达式**

- `(parameters) -> expression`
- `(parameters) ->{statements;}`

## lambda表达式的重要特征

1. 可选的类型声明：不需要声明参数类型，编译器可以统一识别参数值。
2. 可选的参数圆括号` ()`：
   - 一个参数无需定义圆括号 `()`
   - 多个参数需要定义圆括号` ()`
3. 可选的大括号` {}`：如果主体只有一个表达式，就不需要使用大括号
4. 可选的返回关键字：如果主体只有一个表达式，返回值则编译器会自动返回值
5. 大括号`{}`需要指定表达式返回了一个数值(`return`)。

## 函数式接口(一个抽象方法的接口，提供lambda表达式)

对于只有一个抽象方法的接口 ，需要这种接口的对象时，可以提供lambda表达式，调用对象的方法时，执行lambda表达式的体。

- lambda表达式看作一个函数 ，不是一个对象。

- lambda表达式可以转换为接口(接口变量)。
- `@FunctionalInterface` 注解：强制接口只有一个抽象方法。如果有多个话直接会报错。

```java
public class FunctionalInterface {

    /**
     * 参数为接口的对象，使用lambda表达式（作为一个函数参数）
     *
     * @param a
     * @param b
     * @param mathOperation
     */
    private int operate(int a, int b, MathOperation mathOperation) {
        // 调用对象的方法，会执行lambda表达式的体（覆写的方法）
        return mathOperation.operation(a, b);
    }

    public static void main(String args[]) {
        FunctionalInterface l = new FunctionalInterface();

        // 函数式接口，生成接口对象（接口实现类对象）

        // 声明类型参数
        MathOperation addition = (int a, int b) -> a + b;

        // 不声明参数类型
        MathOperation subtraction = (a, b) -> a - b;

        // 表示式有大括号，需有返回语句
        MathOperation multiplication = (int a, int b) -> {
            return a * b;
        };

        // 表示式无大括号，无需返回语句
        MathOperation division = (int a, int b) -> a / b;

        // 函数作为参数
        System.out.println("10 + 5 = " + l.operate(10, 5, addition));
        System.out.println("10 - 5 = " + l.operate(10, 5, subtraction));
        System.out.println("10 x 5 = " + l.operate(10, 5, multiplication));
        System.out.println("10 / 5 = " + l.operate(10, 5, division));

        // 参数无括号
        GreetingService greetService1 = message ->
                System.out.println("Hello " + message);

        // 参数使用括号
        GreetingService greetService2 = (message) ->
                System.out.println("Hello " + message);

        greetService1.sayMessage("Runoob");
        greetService2.sayMessage("Google");
    }

    // 函数式接口，只有一个抽象方法
    interface MathOperation {
        int operation(int a, int b);
    }

    // 函数式接口，只有一个抽象方法
    interface GreetingService {
        void sayMessage(String message);
    }

}
```

## 方法引用(lambda表达式涉及一个方法：参数，返回类型相同)

方法引用，lambda表达式另一种格式，把方法当作参数传递。

- 某个方法**除方法名外**，**方法参数、返回类型和接口抽象方法一致**，就可以直接**传入方法引用**，覆盖接口抽象方法，调用给定的方法。

**⽅法引⽤不能独⽴存在，会转换为函数式接口的实例**

- ⽅法引⽤会指示编译器⽣成⼀个函数式接口实例，覆盖这个接口的抽象⽅法，来调⽤给定的⽅法。
- 类似于lambda表达式，⽅法引⽤也不是⼀个对象

⽅法引⽤使⽤ `::`分隔⽅法名与对象/类名，主要有三种情况： 

1. 对象 `::` 实例方法：等价于向方法传递参数的lambda表达式
2. 类` :: `实例方法：实际调用时，方法第一个参数为实例方法隐式参数 (this)
3. 类` ::` 静态方法：所有参数传递到静态方法

```java
@ToString
public class MethodReference {

    private String value;

    public MethodReference() {
    }

    public MethodReference(String value) {
        this.value = value;
    }

    // 静态方法
    public static int cmp1(String s1, String s2) {
        return s1.compareTo(s2);
    }

    // 实例方法
    public int cmp2(String s1, String s2) {
        return s1.compareTo(s2);
    }
    
    public static void main(String[] args) {
        // 如果某个方法除了方法名外，方法参数，返回类型和接口抽象方法恰好一致，就可以直接传入方法引用，覆盖这个接口的抽象方法，来调用给定的方法。

        String[] array = new String[]{"Apple", "Orange", "Banana", "Lemon"};

        // 类::静态方法
        Arrays.sort(array, MethodReference::cmp1);

        // 对象::实例方法
        MethodReference m = new MethodReference();
        Arrays.sort(array, m::cmp2); // 对象引用，对象为null ，会抛出异常
        Arrays.sort(array, (s1, s2) -> s1.compareTo(s2)); // 在调用时才会抛出异常

        // 类::实例方法
        Arrays.sort(array, String::compareTo);

        // 类::new，构造器引用
        Supplier<String, MethodReference> supplier = MethodReference::new;
        MethodReference mr = supplier.get("方法引用");
        System.out.println(mr.toString());
    }

}
```

## lambda和⽅法引⽤区别

- 只有当lambda表达式的体，只调⽤⼀个⽅法⽽不做其他操作时，才能把lambda表达式重写成⽅法引⽤ 。
  - 如`s->s.length()  == 0`，这里有一个方法调用，但还有一个比较，不能使用方法引用。
- 对象 `::` 实例方法 与 等价的lambda表达式还有⼀个细微差别
  - 如果对象为空，⽅法引⽤会直接抛出异常，lambda表达式只有在调⽤时才会抛出异常 
- 可以在⽅法引⽤中使⽤` this` 、`super`参数

## 构造器引⽤

- 与⽅法引⽤类似，不过⽅法名为 `new`

- `Person::new`这就是`Person`构造器的⼀个引⽤，具体哪⼀个构造器呢，取决于上下⽂
- 数组的构造器引⽤`Integer[]::new`

## 变量作用域

lambda 表达式只能引用标记了 `final` 的外层变量 ，不能在 lambda 内部修改定义在域外的变量

局部变量也可以不用声明为` final`，但是必须不可修改（即隐性的具有`final` 的语义）

在lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

lambda表达式中的 this 含义与外⾯⼀致。

```java
public class ScopeOfVariable {
    private String id;

    // final 外层局部变量
    public final static String s1 = "lbb";

    public static void main(String[] args) {
        // final 外层局部变量
        final String s2 = "ban";
        // 函数式接口，生成接口对象
        TestInterface t1 = message -> System.out.println(message + "," + s1);
        TestInterface t2 = message -> System.out.println(message + "," + s2);
        t1.sayMessage("hello");
        t2.sayMessage("hello2");

        String s3 = "test";
        // s3 改变，报错
        s3 = "test2";
        TestInterface t3 = message -> System.out.println(message + "," + s3);
        // 报错,不允许声明一个与局部变量同名的参数或者局部变量
        TestInterface t4 = s3 -> System.out.println(message + "," + s3);
    }

    public void test() {
        TestInterface t = message -> {
            this.id = "123";
            System.out.println(message + "," + this.id);
        };
    }

    interface TestInterface {
        void sayMessage(String message);
    }

}
```

## 处理lambda表达式（延迟执行）

**lambda表达式可以延迟执行（懒计算 ）**

- 接受lambda表达式，选择一个函数式接口 

- `@FunctionalInterface` 注解

## 常用函数式接口

- Runnable
- ActionListener
- Comparator
- Callable
- Consumer
- Predicate
- Supplier