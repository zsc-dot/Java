# 一、Lambda表达式

## 1、需求分析

创建一个新的线程，指定线程需要执行的任务

```java
public static void main(String[] args) {
    // 开启一个新的线程
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("新线程中执行的代码" + Thread.currentThread().getName());
        }
    }).start();
    System.out.println("主线程中的代码：" + Thread.currentThread().getName());
}
```

代码分析：

1. Thread类需要一个Runnable接口作为参数，其中的抽象方法run方法时用来指定线程任务内容的核心
2. 为了指定run方法体，不得不需要Runnable的实现类
3. 为了省去定义一个Runnable的实现类，不得不使用匿名内部类
4. 必须覆盖重写抽象的run方法，所有的方法名称，方法参数，方法返回值不得不都重写一遍，而且不能出错
5. 而实际上，我们只在乎方法体中的代码



## 2、Lambda表达式初体验

Lambda表达式是一个匿名函数，可以理解为一段可以传递的代码

```java
new Thread(() -> {
    System.out.println("新线程Lambda表达式：" + Thread.currentThread().getName());
}).start();
```

Lambda表达式的优点：简化了匿名内部类的使用，语法更加简单



## 3、Lambda的语法规则

Lambda省去了面向对象的条条框框，Lambda的标准格式由3个部分组成：

```java
(参数类型 参数名称) -> {
    代码体;
}
```

格式说明：

- (参数类型 参数名称)：参数列表
- {代码体;}：方法体
- ->：箭头，分隔参数列表和方法体



### 3.1、Lambda练习1

练习无参无返回值的Lambda

定义一个接口

```java
public interface UserService {
    void show();
}
```

然后创建主方法使用

```java
public static void main(String[] args) {
    goShow(new UserService() {
        @Override
        public void show() {
            System.out.println("show方法执行了...");
        }
    });
    System.out.println("------------------------");
    goShow(() -> {
        System.out.println("Lambda show 方法执行了...");
    });
}

public static void goShow(UserService userService) {
    userService.show();
}
```

输出：

```
show方法执行了...
------------------------
Lambda show 方法执行了
```



### 3.2、Lambda练习2

有参有返回值的Lambda

创建一个Person对象

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private String name;
    private Integer age;
    private Integer height;
}
```

在List集合中保存多个Person对象，根据age进行排序

```java
public static void main(String[] args) {
    List<Person> list = new ArrayList<>();
    list.add(new Person("周杰伦", 33, 175));
    list.add(new Person("刘德华", 43, 185));
    list.add(new Person("周星驰", 38, 177));

    Collections.sort(list, new Comparator<Person>() {
        @Override
        public int compare(Person o1, Person o2) {
            return o1.getAge() - o2.getAge();
        }
    });
    for (Person person : list) {
        System.out.println(person);
    }
}
```

我们发现在sort方法的第二个参数是一个Comparator接口的匿名内部类，且执行的方法有参数和返回值，那么可以改写为Lambda表达式

```java
public static void main(String[] args) {
    List<Person> list = new ArrayList<>();
    list.add(new Person("周杰伦", 33, 175));
    list.add(new Person("刘德华", 43, 185));
    list.add(new Person("周星驰", 38, 177));

    Collections.sort(list, (Person o1, Person o2) -> {
        return o1.getAge() - o2.getAge();
    });
    for (Person person : list) {
        System.out.println(person);
    }
}
```



## 4、@FunctionalInterface注解

FunctionalInterface是一个标志注解，被该注解修饰的接口只能声明一个抽象方法



## 5、Lambda表达式原理

匿名内部类的本质是在编译时生成一个Class文件，xxxx$0.class。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240406172935439.png" alt="image-20240406172935439" style="zoom: 80%;" />

还可以通过反编译工具来查看生成的代码 XJad工具来查看。

Lambda表达式的原理也可以通过反编译工具来查看会报错，可以通过JDK自带的一个工具：javap对字节码进行反汇编操作。

```shell
javap -c -p 文件名.class

-c：表示对代码进行反汇编
-p：显示所有的类和成员
```

在这个反编译的源码中，可以看到静态方法lambda$main0，这个方法里面做了什么事，可以通过debug的方式查看。

方法体中的代码是在lambda$main0中执行的

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240406173150846.png" alt="image-20240406173150846" style="zoom:80%;" />

上面的效果可以理解为如下：

```java
public class Demo03Lambda {
    public static void main(String[] args) {
    	....
    }
    private static void lambda$main$0();
    	System.out.println("Lambda show 方法执行了...");
    }
}
```

为了更加直观的理解这个内容，可以在运行的时候添加 -Djdk.internal.lambda.dumpProxyClasses，加上这个参数会将内部Class码输出到一个文件中

```shell
-Djdk.internal.lambda.dumpProxyClasses 要运行的包名.类名
```

反编译后的内容：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240406173255969.png" alt="image-20240406173255969" style="zoom:80%;" />

可以看到匿名的内部类实现了UserService接口，并重写了show方法。在show方法中调用了Demo03Lambda.lambda$main0()，也就是调用了Lambda中的内容。

小结：

匿名内部类在编译的时候会产生一个class文件。

Lambda表达式在程序运行的时候，会形成一个类。

1. 在类中新增了一个方法，这个方法的方法体就是Lambda表达式中的内容
2. 还会形成一个匿名内部类，实现接口，重写抽象方法
3. 在接口中重写方法会调用新生成的方法



## 6、Lambda表达式的省略写法

在lambda表达式的标准写法基础上，可以使用省略写的规则：

1. 小括号内的参数类型可以省略
2. 如果小括号内有且仅有一个参数，则小括号可以省略
3. 如果大括号内有且仅有一个语句，可以同时省略大括号，return 关键字及语句分号

```java
public interface StudentService {
    String show(String name, Integer age);
}

public interface OrderService {
    Integer show(String name);
}

public static void main(String[] args) {
    goStudent((String name, Integer age) -> {
        return name + age + "666666 ...";
    });
    // 省略写法
    goStudent((name, age) -> name + age + "666666 ...");
    System.out.println("------------------");
    goOrder((String name) -> {
        System.out.println("------>" + name);
        return 666;
    });
    // 省略写法
    goOrder(name -> {
        System.out.println("------>" + name);
        return 666;
    });
}

public static void goStudent(StudentService studentService) {
    studentService.show("张三", 22);
}

public static void goOrder(OrderService orderService) {
    orderService.show("李四");
}
```



## 7、Lambda表达式的使用前提

Lambda表达式的语法是非常简洁的，但Lambda表达式不是随便使用的，使用时有几个条件要特别注意：

1. 方法的参数或局部变量类型必须为接口才能使用Lambda
2. 接口中有且仅有一个抽象方法(@FunctionalInterface可以限制)



## 8、Lambda和匿名内部类的对比

1. 所需类型不一样
   - 匿名内部类的类型可以是类、抽象类、接口
   - Lambda表达式的类型必须是接口
2. 抽象方法的数量不一样
   - 匿名内部类所需的接口中的抽象方法的数量是任意的
   - Lambda表达式所需的接口中只能有一个抽象方法
3. 实现原理不一样
   - 匿名内部类是在编译后形成一个class
   - Lambda表达式是在程序运行的时候动态生成class



# 二、接口中新增的方法

## 1、JDK8中接口的新增

在JDK8中针对接口有做增强，在JDK8之前

```java
interface 接口名{
    静态常量;
    抽象方法;
}
```

在JDK8之后，对接口做了增强，接口中可以有**默认方法**和**静态方法**

```java
interface 接口名{
    静态常量;
    抽象方法;
    默认方法;
    静态方法;
}
```



## 2、默认方法

### 2.1、为什么要增加默认方法

在JDK8以前，接口中只能有抽象方法和静态常量，会存在以下的问题：

如果接口中新增抽象方法，那么实现类都必须重写这个抽象方法，非常不利于接口的扩展

```java
public class Demo01 {
    public static void main(String[] args) {
        A b = new B();
        A c = new C();
    }
}

interface A{
    void test1();
    // 接口中新增抽象方法，所有实现类都需要重写这个方法，不利于接口的扩展
    void test2();
}

class B implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }
}

class C implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }
}
```



### 2.2、接口默认方法的格式

接口中默认方法的格式是

```java
interface 接口名{
    修饰符 default 返回值类型 方法名{
        方法体;
    }
}
```

```java
public class Demo01 {
    public static void main(String[] args) {
        A b = new B();
        b.test3();
        A c = new C();
        c.test3();
    }
}

interface A{
    void test1();
    // 接口中新增抽象方法，所有实现类都需要重写这个方法，不利于接口的扩展
    void test2();

    // 接口中定义的默认方法
    public default String test3() {
        System.out.println("接口中的默认方法执行了...");
        return "hello";
    }
}

class B implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }

    @Override
    public String test3() {
        System.out.println("B 实现类中重写了默认方法...");
        return "ok...";
    }
}

class C implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }
}
```



### 2.3、接口中默认方法的使用

接口中的默认方法有两种使用方式

1. 实现类直接调用接口的默认方法
2. 实现类重写接口的默认方法



## 3、静态方法

JDK8中为接口新增了静态方法，作用也是为了接口的扩展



### 3.1、语法规则

```java
interface 接口名{
    修饰符 static 返回值类型 方法名{
        方法体;
    }
}
```

```java
public class Demo01 {
    public static void main(String[] args) {
        A b = new B();
        b.test3();
        A c = new C();
        c.test3();
        A.test4();
    }
}

interface A{
    void test1();
    // 接口中新增抽象方法，所有实现类都需要重写这个方法，不利于接口的扩展
    void test2();

    // 接口中定义的默认方法
    public default String test3() {
        System.out.println("接口中的默认方法执行了...");
        return "hello";
    }

    // 接口中的静态方法
    public static String test4() {
        System.out.println("接口中的静态方法...");
        return "hello";
    }
}

class B implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }

    @Override
    public String test3() {
        System.out.println("B 实现类中重写了默认方法...");
        return "ok...";
    }
}

class C implements A{
    @Override
    public void test1() {

    }
    @Override
    public void test2() {

    }
}
```



### 3.2、接口中静态方法的使用

接口中的静态方法在实现类中是不能被重写的，调用的话只能通过接口类型来实现：接口名.静态方法名();

![image-20240406182510596](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240406182510596.png)



## 4、两者的区别

1. 默认方法通过实例调用，静态方法通过接口名调用
2. 默认方法可以被继承，实现类可以直接调用接口默认方法，也可以重写接口默认方法
3. 静态方法不能被继承，实现类不能重写接口的静态方法，只能使用接口名调用



# 三、函数式接口

## 1、函数式接口的由来

我们知道使用Lambda表达式的前提是需要有函数式接口，而Lambda表达式使用时不关心接口名、抽象方法名，只关心抽象方法的参数列表和返回值类型。因此，为了让我们使用Lambda表达式更加的方便，在JDK中提供了大量常用的函数式接口

```java
public class Demo01Fun {
    public static void main(String[] args) {
        fun1(arr -> {
            int sum = 0;
            for (int i : arr) {
                sum += i;
            }
            return sum;
        });
    }

    public static void fun1(Operator operator) {
        int[] arr = {1, 2, 3, 4};
        int sum = operator.getSum(arr);
        System.out.println("sum = " + sum);
    }
}

/**
 * 函数式接口
 */
@FunctionalInterface
interface Operator{
    int getSum(int[] arr);
}
```



## 2、函数式接口介绍

在JDK中帮我们提供的有函数式接口，主要是在java.util.function包中



### 2.1、Supplier

无参有返回值的接口，对应的Lambda表达式需要提供一个返回数据的类型

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

使用：

```java
/**
    Supplier 函数式接口的使用
 */
public class SupplierTest {
    public static void main(String[] args) {
        fun1(() -> {
            int[] arr = {22, 33, 55, 66, 44, 99, 10};
            // 计算出数组中的最大值
            Arrays.sort(arr);
            return arr[arr.length - 1];
        });
    }
    private static void fun1(Supplier<Integer> supplier) {
        // get是一个无参有返回值的抽象方法
        Integer max = supplier.get();
        System.out.println("max = " + max);
    }
}
```



### 2.2、Consumer

有参无返回值的接口，前面介绍的Supplier接口是用来生产数据的，而Consumer接口是用来消费数据的，使用的时候需要指定一个泛型来定义参数类型

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
```

使用：将输入的数据统一转换为小写输出

```java
public class ConsumerTest {
    public static void main(String[] args) {
        test((msg) -> System.out.println(msg + " ->转换为小写：" + msg.toLowerCase()));
    }

    public static void test(Consumer<String> consumer) {
        consumer.accept("HELLO WORLD");
    }
}
```

默认方法：andThen

如果一个方法的参数和返回值全部是Consumer类型，那么就可以实现效果，消费一个数据的时候，首先做一个操作，然后再做一个操作，实现组合，而这个方法就是Consumer接口中的default方法 andThen方法

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```

具体操作：

```java
public class ConsumerAndThenTest {
    public static void main(String[] args) {
        test2(msg1 -> System.out.println(msg1 + " ->转换为小写：" + msg1.toLowerCase()),
                msg2 -> System.out.println(msg2 + " ->转换为大写：" + msg2.toUpperCase()));
    }

    public static void test2(Consumer<String> c1, Consumer<String> c2) {
        String str = "Hello World";
        // c1.accept(str); // 转小写
        // c2.accept(str); // 转大写
        c1.andThen(c2).accept(str);
    }
}
```

理解：andThen方法会生成一个新的Consumer对象，编译器根据上下文推断发现调用本身accpet方法会造成递归，所以使用外部实例对象调用accpet方法

```java
public Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return new Consumer<T>() {
        @Override
        public void accept(T t) {
            Consumer.this.accept(t);  // 调用外部类对象（即当前对象）的accept方法
            after.accept(t);
        }
    };
}
```



### 2.3、Function

有参有返回值的接口，Function接口是根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。有参数有返回值

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
```

使用：传递进去一个字符串，返回一个数字

```java
public class FunctionTest {
    public static void main(String[] args) {
        test(msg -> {
            return Integer.parseInt(msg);
        });
    }

    public static void test(Function<String, Integer> function) {
        Integer apply = function.apply("666");
        System.out.println("apply的结果：" + apply);
    }
}
```



#### 2.3.1、默认方法

1、addThen，也是用来进行组合操作

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```

```java
public class FunctionAndThenTest {
    public static void main(String[] args) {
        test(msg1 -> {
            return Integer.parseInt(msg1);
        }, msg2 -> {
            return msg2 * 10;
        });
    }

    public static void test(Function<String, Integer> f1, Function<Integer, Integer> f2) {
        // Integer i1 = f1.apply("666");
        // Integer i2 = f2.apply(i1);
        Integer i2 = f1.andThen(f2).apply("666");
        System.out.println("i2 = " + i2);
    }
}
```



2、compose方法

与apply方法顺序刚好相反

```java
public class FunctionAndThenTest {
    public static void main(String[] args) {
        test(msg1 -> {
            return Integer.parseInt(msg1);
        }, msg2 -> {
            return msg2 * 10;
        });
    }

    public static void test(Function<String, Integer> f1, Function<Integer, Integer> f2) {
        // Integer i1 = f1.apply("666");
        // Integer i2 = f2.apply(i1);
        Integer i2 = f2.compose(f1).apply("666");
        System.out.println("i2 = " + i2);
    }
}
```



3、identity

传过来什么就返回什么

```java
static <T> Function<T, T> identity() {
    return t -> t;
}
```





### 2.4、Predicate

有参且返回值为boolean的接口

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```

使用：

```java
public class PredicateTest {
    public static void main(String[] args) {
        test(msg -> {
            return msg.length() > 3;
        }, "Hello World");
    }

    private static void test(Predicate<String> predicate, String msg) {
        boolean b = predicate.test(msg);
        System.out.println("b：" + b);
    }
}
```

在Predicate中的默认方法提供了逻辑关系操作：and、or、negate、isEqual

```java
public class PredicateDefaultTest {
    public static void main(String[] args) {
        test(msg1 -> {
            return msg1.contains("H");
        }, msg2 -> {
            return msg2.contains("W");
        });
    }

    private static void test(Predicate<String> p1, Predicate<String> p2) {
        // b1包含H  b2包含W
        // b1包含H 同时 b2包含W
        boolean b1 = p1.and(p2).test("Hello");
        // b1包含H 或者 b2包含W
        boolean b2 = p1.or(p2).test("Hello");
        // p1不包含H
        boolean b3 = p1.negate().test("Hello");
        System.out.println(b1); // false
        System.out.println(b2); // true
        System.out.println(b3); // false
    }
}
```



# 四、方法引用

## 1、为什么要用方法引用

### 1.1、Lambda表达式冗余

在使用Lambda表达式的时候，也会出现代码冗余的情况，比如：用Lambda表达式求一个数组的和

```java
public class FunctionRefTest01 {
    public static void main(String[] args) {
        printMax(a -> {
            // Lambda表达式中的代码和getTotal中的代码冗余了
            int sum = 0;
            for (Integer i : a) {
                sum += i;
            }
            System.out.println("数组求和：" + sum);
        });
    }

    /**
     * 求数组中的所有元素的和
     * @param a
     */
    public void getTotal(Integer[] a) {
        int sum = 0;
        for (Integer i : a) {
            sum += i;
        }
        System.out.println("数组求和：" + sum);
    }

    private static void printMax(Consumer<Integer[]> consumer) {
        Integer[] a = {10, 20, 30, 40, 50, 60};
        consumer.accept(a);
    }
}
```



### 1.2、解决方案

因为在Lambda表达式中要执行的代码和我们另一个方法中的代码是一样的，这时就没有必要重写一份逻辑了，这时我们就可以“引用”重复代码

个人理解：方法引用本质是FunctionRefTest02.getTotal()的简化写法，因为参数和返回值一致，所以jdk8支持简化

```java
public class FunctionRefTest02 {
    public static void main(String[] args) {
        // ::方法引用 也是JDK8中新的语法
        printMax(FunctionRefTest02::getTotal);
    }

    /**
     * 求数组中的所有元素的和
     * @param a
     */
    public static void getTotal(Integer[] a) {
        int sum = 0;
        for (Integer i : a) {
            sum += i;
        }
        System.out.println("数组求和：" + sum);
    }

    private static void printMax(Consumer<Integer[]> consumer) {
        Integer[] a = {10, 20, 30, 40, 50, 60};
        consumer.accept(a);
    }
}
```



## 2、方法引用的格式

符号表示：`::`

符号说明：双帽号为方法引用运算符，而它所在的表达式被称为`方法引用`

应用场景：如果Lambda表达式所要实现的方案，已经有其他方法存在相同的方案，那么则可以使用方法引用

常见的引用方式：

方法引用在JDK8中使用是相当灵活的，有以下几种形式：

1. instanceName::methodName   对象::方法名
2. ClassName::staticMethodName   类名::静态方法
3. ClassName::methodName   类名::普通方法
4. ClassName::new   类名::调用的构造器
5. TypeName[]::new   String[]::new 调用数组的构造器



### 2.1、对象名::方法名

这是最常见的一种用法。如果一个类中已经存在了一个成员方法，则可以通过对象名引用成员方法

```java
public static void main(String[] args) {
    Date now = new Date();
    Supplier<Long> supplier = () -> now.getTime();
    System.out.println(supplier.get());
    // 然后我们通过方法引用的方式来处理
    Supplier<Long> supplier1 = now::getTime;
    System.out.println(supplier1.get());
}
```

方法引用的注意事项：

1. 被引用的方法参数要和接口中的抽象方法的参数一样
2. 当接口的抽象方法有返回值时，被引用的方法也必须有返回值



### 2.2、类名::静态方法名

也是比较常用的方式。

```java
public static void main(String[] args) {
    Supplier<Long> supplier1 = () -> {
        return System.currentTimeMillis();
    };
    System.out.println(supplier1.get());
    // 通过方法引用实现
    Supplier<Long> supplier2 = System::currentTimeMillis;
    System.out.println(supplier2.get());
}
```



### 2.3、类名::普通方法名

Java面向对象中，类名只能调用静态方法，类名引用实例方法是有前提的，实际上是拿第一个参数作为方法的调用者。

```java
public static void main(String[] args) {
    Function<String, Integer> function = (s) -> {
        return s.length();
    };
    System.out.println(function.apply("hello"));
    // 通过方法引用来实现
    Function<String, Integer> function1 = String::length;
    System.out.println(function1.apply("hahahahaha"));
    BiFunction<String, Integer, String> function2 = String::substring;
    String msg = function2.apply("Hello World", 3);
    System.out.println(msg);
}
```



### 2.4、类名::构造器

由于构造器的名称和类名完全一致，所以构造器引用使用`::new`的格式使用

```java
public static void main(String[] args) {
    Supplier<Person> supplier = () -> {return new Person();};
    System.out.println(supplier.get());
    // 通过方法引用来实现
    Supplier<Person> supplier1 = Person::new;
    System.out.println(supplier1.get());
    BiFunction<String, Integer, Person> function = Person::new;
    System.out.println(function.apply("张三", 22));
}
```



### 2.5、数组::构造器

数组是怎么构造出来的呢？

```java
public static void main(String[] args) {
    Function<Integer, String[]> function1 = (len) -> {
        return new String[len];
    };
    String[] a1 = function1.apply(4);
    System.out.println("数组的长度是：" + a1.length);
    // 方法引用的方式来调用数组的构造器
    Function<Integer, String[]> function2 = String[]::new;
    String[] a2 = function2.apply(5);
    System.out.println("数组的长度是：" + a2.length);
}
```



### 2.6、小结

方法引用是对Lambda表达式符合特定情况下的一种缩写方式，它使得我们的Lambda表达式更加精简，也可以理解为Lambda表达式的缩写形式，不过要注意的是，方法引用只能引用已经存在的方法



# 五、Stream API

## 1、集合处理数据的弊端

当我们在需要对集合中的元素进行操作的时候，除了必须的添加、删除、获取外，最典型的操作就是集合遍历

```java
public static void main(String[] args) {
    // 定义一个List集合
    List<String> list = Arrays.asList("张三", "张三丰", "成龙", "周星驰");
    // 1. 获取所有姓张的信息
    List<String> list1 = new ArrayList<>();
    for (String s : list) {
        if (s.startsWith("张")) {
            list1.add(s);
        }
    }
    // 2. 获取名称长度为3的用户
    List<String> list2 = new ArrayList<>();
    for (String s : list1) {
        if (s.length() == 3) {
            list2.add(s);
        }
    }
    // 3. 输出所有用户的信息
    for (String s : list2) {
        System.out.println(s);
    }
}
```

上面的代码针对于我们不同的需求，总是一次次的循环，这时我们希望有更加高效的处理方式，这时我们就可以通过JDK8中提供的Stream API来解决这个问题。

Stream更加优雅的解决方案：

```java
public static void main(String[] args) {
    // 定义一个List集合
    List<String> list = Arrays.asList("张三", "张三丰", "成龙", "周星驰");
    // 1. 获取所有姓张的信息
    // 2. 获取名称长度为3的用户
    // 3. 输出所有用户的信息
    list.stream()
        .filter(s -> s.startsWith("张"))
        .filter(s -> s.length() == 3)
        .forEach(s -> System.out.println(s));
    System.out.println("-------------------");
    list.stream()
        .filter(s -> s.startsWith("张"))
        .filter(s -> s.length() == 3)
        .forEach(System.out::println);
}
```

上面的Stream API代码的含义：获取流，过滤张，过滤长度，逐一打印



## 2、Stream流式思想概述

注意：Stream流和IO流没有任何关系，请暂时忘记对传统IO流的固有印象！

Stream流式思想类似于工厂车间的“生产流水线”，Stream不是一种数据结构，不保存数据，而是对数据进行加工处理。

Stream流可以看作是流水线上的一个工序。在流水线上，通过多个工序让一个原材料加工成一个商品。

![image-20240407113153954](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240407113153954.png)

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240407113323717.png" alt="image-20240407113323717" style="zoom:67%;" />

Stream API能让我们快速完成许多复杂的操作，如筛选、切片、映射、查找、去除重复，统计，匹配和归约。



## 3、Stream流的获取方式

### 3.1、根据Collection获取

首先，java.util.Collection接口中加入了default方法stream，也就是说，Collection接口下的所有实现都可以通过stream方法获取Stream流对象

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.stream();
    Set<String> set = new HashSet<>();
    set.stream();
    Vector vector = new Vector();
    vector.stream();
}
```

但是Map接口没有实现Collection接口，那这时该怎么办呢？这时我们可以根据Map获取对应的key value的集合。

```java
public static void main(String[] args) {
    Map<String, Object> map = new HashMap<>();
    Stream<String> stream = map.keySet().stream(); // key
    Stream<Object> stream1 = map.values().stream(); // value
    Stream<Map.Entry<String, Object>> stream2 = map.entrySet().stream(); // entry
}
```



### 3.2、通过Stream的of方法

在实际开发中，我们不可避免的还是会操作到数组中的数据，由于数组对象不可能添加默认方法，所以Stream接口中提供了静态方法of

```java
public static void main(String[] args) {
    Stream<String> a1 = Stream.of("a1", "a2", "a3");
    String[] arr1 = {"aa", "bb", "cc"};
    Stream<String> arr11 = Stream.of(arr1);
    Integer[] arr2 = {1, 2, 3, 4};
    Stream<Integer> arr21 = Stream.of(arr2);
    arr21.forEach(System.out::println);
    // 注意：基本数据类型的数组是不行的，会打印出地址
    int[] arr3 = {1, 2, 3, 4};
    Stream.of(arr3).forEach(System.out::println);
}
```



## 4、Stream常用方法介绍

Stream常用方法

Stream流模型的操作很丰富，这里介绍一些常用的API，这些方法可以被分成两种：

| 方法名  |  方法作用  | 返回值类型 | 方法种类 |
| :-----: | :--------: | :--------: | :------: |
|  count  |  统计个数  |    long    |   终结   |
| forEach |  逐一处理  |    void    |   终结   |
| filter  |    过滤    |   Stream   | 函数拼接 |
|  limit  | 取用前几个 |   Stream   | 函数拼接 |
|  skip   | 跳过前几个 |   Stream   | 函数拼接 |
|   map   |    映射    |   Stream   | 函数拼接 |
| concat  |    组合    |   Stream   | 函数拼接 |

**终结方法：**返回值类型不再是Stream类型的方法，不再支持链式调用。本小结中，终结方法包括count和forEach方法

**非终结方法：**返回值类型仍然是Stream类型的方法，支持链式调用。（除了终结方法外，其余方法均为非终结方法）

**Stream注意事项（重要）**

1. Stream只能操作一次
2. Stream方法返回的是新的流
3. Stream不调用终结方法，中间的操作不会执行



### 4.1、forEach

forEach用来遍历流中的数据的

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个Consumer接口，会将每一个流元素交给函数处理

```java
public static void main(String[] args) {
    Stream.of("a1", "a2", "a3").forEach(System.out::println);
}
```



### 4.2、count

count用来统计其中的元素个数

```java
long count();
```

该方法返回一个long值，代表元素的个数

```java
public static void main(String[] args) {
    long count = Stream.of("a1", "a2", "a3").count();
    System.out.println(count);
}
```



### 4.3、filter

filter是用来过滤数据的，返回符合条件的数据

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240408105228201.png" alt="image-20240408105228201" style="zoom:67%;" />

可以通过filter方法将一个流转换成另一个子集流

```java
Stream<T> filter(Predicate<? super T> predicate);
```

该接口接收一个Predicate函数式接口参数，作为筛选条件

```java
public static void main(String[] args) {
    Stream.of("a1", "a2", "a3", "bb", "cc", "dd", "aa")
        .filter(s -> s.contains("a"))
        .forEach(System.out::println);
}
```

输出：

```txt
a1
a2
a3
aa
```



### 4.4、limit

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240408105930411.png" alt="image-20240408105930411" style="zoom:67%;" />

limit可以对流进行截取处理，只取前n个数据

```java
Stream<T> limit(long maxSize);
```

参数是一个long类型的数值，如果集合当前长度大于参数，就进行截取，否则不操作

```java
public static void main(String[] args) {
    Stream.of("a1", "a2", "a3", "bb", "cc", "dd", "aa")
        .limit(3)
        .forEach(System.out::println);
}
```

输出：

```txt
a1
a2
a3
```



### 4.5、skip

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410102106307.png" alt="image-20240410102106307" style="zoom: 67%;" />

如果希望跳过前面几个元素，可以使用skip方法，获取一个截取之后的新流：

```java
Stream<T> skip(long n);
```

操作：

```java
public static void main(String[] args) {
    Stream.of("a1", "a2", "a3", "bb", "cc", "dd", "aa")
        .skip(3)
        .forEach(System.out::println);
}
```

输出：

```txt
bb
cc
dd
aa
```



### 4.6、map

如果我们需要将流中的元素映射到另一个流中，可以使用map方法：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410102754533.png" alt="image-20240410102754533" style="zoom:67%;" />

该接口需要一个Function函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的数据

```java
public static void main(String[] args) {
    Stream.of("1", "2", "3", "4", "5", "6", "7")
        // .map(msg -> Integer.parseInt(msg))
        .map(Integer::parseInt)
        .forEach(System.out::println);
}
```



### 4.7、sorted

如果需要将数据排序，可以使用sorted方法：

```java
Stream<T> sorted();
```

在使用的时候可以根据自然规则排序，也可以通过比较器来指定对应的排序规则

```java
public static void main(String[] args) {
    Stream.of("1", "3", "2", "4", "0", "9", "7")
        // .map(msg -> Integer.parseInt(msg))
        .map(Integer::parseInt)
        // .sorted() // 根据数据的自然顺序排序，从小到大
        .sorted((o1, o2) -> o2 - o1) // 降序
        .forEach(System.out::println);
}
```



### 4.8、distinct

如果要去掉重复数据，可以使用distinct

```java
Stream<T> distinct();
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410103931099.png" alt="image-20240410103931099" style="zoom:67%;" />

使用：

```java
public static void main(String[] args) {
    Stream.of("1", "3", "3", "4", "0", "1", "7")
        // .map(msg -> Integer.parseInt(msg))
        .map(Integer::parseInt)
        // .sorted() // 根据数据的自然顺序排序，从小到大
        .sorted((o1, o2) -> o2 - o1) // 降序
        .distinct() // 去掉重复的记录
        .forEach(System.out::println);
    System.out.println("-----------------");
    Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 18)
    )
        .distinct()
        .forEach(System.out::println);
}
```

Stream流中的distinct方法对于基本数据类型是可以直接去重的，但是对于引用数据类型，我们需要重写equals和hashCode方法来移除重复元素



### 4.9、match

如果需要判断数据是否匹配指定的条件，可以使用match相关的方法

```java
boolean anyMatch(Predicate<? super T> predicate); // 元素是否有任意一个满足条件
boolean allMatch(Predicate<? super T> predicate); // 元素是否都满足条件
boolean noneMatch(Predicate<? super T> predicate); // 元素是否都不满足条件
```

使用：

```java
public static void main(String[] args) {
    boolean b = Stream.of("1", "3", "3", "4", "0", "1", "7")
        .map(Integer::parseInt)
        // .allMatch(s -> s > 0); // false
        // .anyMatch(s -> s > 4); // true
        .noneMatch(s -> s > 4); // false
    System.out.println(b);
}
```

注意：match是一个终结方法



### 4.10、find

如果我们需要找到某些数据，可以使用find方法实现

```java
Optional<T> findFirst(); // 获取第一个
Optional<T> findAny(); // 返回任意一个元素
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410105414549.png" alt="image-20240410105414549" style="zoom:67%;" />

使用：

```java
public static void main(String[] args) {
    Optional<String> first = Stream.of("1", "3", "3", "4", "0", "1", "7").findFirst();
    System.out.println(first.get());

    Optional<String> any = Stream.of("1", "3", "3", "4", "0", "1", "7").findAny();
    System.out.println(any.get());
}
```



### 4.11、max和min

如果我们想要获取最大值和最小值，那么可以使用max和min方法

```java
Optional<T> max(Comparator<? super T> comparator);
Optional<T> min(Comparator<? super T> comparator);
```

使用：

```java
public static void main(String[] args) {
    Optional<Integer> max = Stream.of("1", "3", "3", "4", "0", "1", "7")
        .map(Integer::parseInt)
        .max((o1, o2) -> o1 - o2);
    System.out.println(max.get());

    Optional<Integer> any = Stream.of("1", "3", "3", "4", "0", "1", "7")
        .map(Integer::parseInt)
        .min((o1, o2) -> o1 - o2);
    System.out.println(any.get());
}
```



### 4.12、reduce

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410111429559.png" alt="image-20240410111429559" style="zoom:67%;" />

如果需要将所有数据归纳得到一个数据，可以使用reduce方法

```java
T reduce(T identity, BinaryOperator<T> accumulator);
```

使用：

```java
public static void main(String[] args) {
    Integer sum = Stream.of(4, 5, 3, 9)
        // identity默认值
        // 第一次循环时会将默认值赋值给x
        // 之后每次会将上一次的操作结果赋值给x
        // y就是每次从数据中获取元素
        .reduce(0, (x, y) -> {
            System.out.println("x = " + x + ", y = " + y);
            return x + y;
        });
    System.out.println(sum);
    // 获取最大值
    Integer max = Stream.of(4, 5, 3, 9)
        .reduce(0, (x, y) -> {
            return x > y ? x : y;
        });
    System.out.println(max);
}
```



### 4.13、map和reduce的组合

在实际开发中，我们经常会将map和reduce一块使用

```java
public static void main(String[] args) {
    // 1. 求出所有年龄的总和
    Integer sumAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).map(Person::getAge) // 实现数据类型的转换
        .reduce(0, Integer::sum);
    System.out.println(sumAge);
    // 2. 求出所有年龄的最大值
    Integer maxAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).map(Person::getAge) // 实现数据类型的转换
        .reduce(0, Math::max);
    System.out.println(maxAge);
    // 3. 统计字符 a 出现的次数
    Integer count = Stream.of("a", "b", "c", "d", "a", "c", "a")
        .map(ch -> "a".equals(ch) ? 1 : 0)
        .reduce(0, Integer::sum);
    System.out.println(count);
}
```



### 4.14、mapToInt

如果需要将Stream中的Integer类型转换成int类型，可以使用mapToInt方法来实现

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410114616398.png" alt="image-20240410114616398" style="zoom:67%;" />

```java
public static void main(String[] args) {
    // Integer占用的内存要比int多很多，在Stream流操作中会自动装箱拆箱操作
    Integer[] arr = {1, 2, 3, 5, 6, 8};
    Stream.of(arr)
        .filter(i -> i > 0)
        .forEach(System.out::println);
    System.out.println("----------");
    // 为了提高程序代码的效率，我们可以先将流中的Integer数据转换为int数据，然后再操作
    IntStream intStream = Stream.of(arr)
        .mapToInt(Integer::intValue);
    intStream.filter(i -> i > 3)
        .forEach(System.out::println);
}
```



### 4.15、concat

如果有两个流希望合并成为一个流，那么可以使用Stream接口的静态方法concat

```java
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
    Objects.requireNonNull(a);
    Objects.requireNonNull(b);

    @SuppressWarnings("unchecked")
    Spliterator<T> split = new Streams.ConcatSpliterator.OfRef<>(
        (Spliterator<T>) a.spliterator(), (Spliterator<T>) b.spliterator());
    Stream<T> stream = StreamSupport.stream(split, a.isParallel() || b.isParallel());
    return stream.onClose(Streams.composedClose(a, b));
}
```

使用：

```java
public static void main(String[] args) {
    Stream<String> stream1 = Stream.of("a", "b", "c");
    Stream<String> stream2 = Stream.of("x", "y", "z");
    // 通过concat方法将两个流合并为一个新的流
    Stream.concat(stream1, stream2).forEach(System.out::println);
}
```



### 4.16、综合案例

定义两个集合，在集合中存储多个用户名称，完成如下操作：

1. 第一个队伍只保留姓名长度为3的成员
2. 第一个队伍筛选之后，只要前三个人
3. 第二个队伍只要姓张的成员
4. 第二个队伍筛选之后不要前两个人
5. 将两个队伍合并为一个队伍
6. 根据姓名创建Person对象
7. 打印整个队伍的Person信息

```java
public static void main(String[] args) {
    List<String> list1 = Arrays.asList("迪丽热巴", "宋远桥", "苏星河", "老子", "庄子", "孙子","洪七公");
    List<String> list2 = Arrays.asList("古力娜扎", "张无忌", "张三丰", "赵丽颖", "张二狗", "张天爱", "张三");
    // 1. 第一个队伍只保留姓名长度为3的成员
    // 2. 第一个队伍筛选之后，只要前三个人
    Stream<String> stream1 = list1.stream().filter(s -> s.length() == 3).limit(3);
    // 3. 第二个队伍只要姓张的成员
    // 4. 第二个队伍筛选之后不要前两个人
    Stream<String> stream2 = list2.stream().filter(s -> s.startsWith("张")).skip(2);
    // 5. 将两个队伍合并为一个队伍
    // 6. 根据姓名创建Person对象
    // 7. 打印整个队伍的Person信息
    Stream.concat(stream1, stream2).map(Person::new).forEach(System.out::println);
}
```

输出结果：

```txt
Person(name=宋远桥, age=null)
Person(name=苏星河, age=null)
Person(name=洪七公, age=null)
Person(name=张二狗, age=null)
Person(name=张天爱, age=null)
Person(name=张三, age=null)
```



## 5、Stream结果收集

### 5.1、结果收集到集合中

```java
/**
 * Stream结果收集
 *      收集到集合中
 */
@Test
public void test01() {
    // 收集到List集合中
    List<String> list = Stream.of("aa", "bb", "cc", "aa").collect(Collectors.toList());
    System.out.println(list);
    // 收集到Set集合中
    Set<String> set = Stream.of("aa", "bb", "cc", "aa").collect(Collectors.toSet());
    System.out.println(set);
    // 如果需要获取的类型为具体的实现，比如：ArrayList、HashSet
    ArrayList<String> arrayList = Stream.of("aa", "bb", "cc", "aa").collect(Collectors.toCollection(ArrayList::new));
    System.out.println(arrayList);
    HashSet<String> hashSet = Stream.of("aa", "bb", "cc", "aa").collect(Collectors.toCollection(HashSet::new));
    System.out.println(hashSet);
}
```

输出结果：

```txt
[aa, bb, cc, aa]
[aa, bb, cc]
[aa, bb, cc, aa]
[aa, bb, cc]
```



### 5.2、结果收集到数组中

Stream中提供了toArray方法来将结果放到一个数组中，返回值类型是Object[]，如果我们要指定返回的类型，那么可以使用另一个重载的toArray(IntFunction f)方法

```java
/**
 * Stream结果收集到数组中
 */
@Test
public void test02() {
    Object[] objects = Stream.of("aa", "bb", "cc", "aa").toArray(); // 返回的数组中的元素是Object类型的
    System.out.println(Arrays.toString(objects));
    // 如果需要指定返回的数组中的元素类型
    String[] strings = Stream.of("aa", "bb", "cc", "aa").toArray(String[]::new);
    System.out.println(Arrays.toString(strings));
}
```



### 5.3、对流中的数据做聚合计算

当我们使用Stream流处理数据后，可以像数据库的聚合函数一样，对某个字段进行操作，比如获得最大值、最小值、求和、平均值、统计数量等。

```java
/**
 * Stream流中数据的聚合计算
 */
@Test
public void test03() {
    // 获取年龄的最大值
    Optional<Person> maxAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).collect(Collectors.maxBy((p1, p2) -> p1.getAge() - p2.getAge()));
    System.out.println("最大年龄：" + maxAge.get());
    // 获取年龄的最小值
    Optional<Person> minAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).collect(Collectors.minBy((p1, p2) -> p1.getAge() - p2.getAge()));
    System.out.println("最小年龄：" + minAge.get());
    // 求所有人的年龄之和
    Integer sumAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    )
        // .collect(Collectors.summingInt(s -> s.getAge()));
        .collect(Collectors.summingInt(Person::getAge));
    System.out.println("年龄总和：" + sumAge);
    // 年龄的平均值
    Double avgAge = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).collect(Collectors.averagingDouble(Person::getAge));
    System.out.println("年龄的平均值：" + avgAge);
    // 统计数量
    Long count = Stream.of(
        new Person("张三", 18),
        new Person("李四", 22),
        new Person("张三", 13),
        new Person("王五", 15),
        new Person("张三", 19)
    ).filter(p -> p.getAge() > 18).collect(Collectors.counting());
    System.out.println("满足条件的记录数：" + count);
}
```



### 5.4、对流中数据做分组操作

当我们使用Stream流处理数据后，可以根据某个属性将数据分组

```java
/**
 * 分组计算
 */
@Test
public void test04() {
    // 根据账号对数据进行分组
    Map<String, List<Person>> map1 = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 18, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).collect(Collectors.groupingBy(Person::getName));
    map1.forEach((k, v) -> System.out.println("k = " + k + "\t" + "v = " + v));
    System.out.println("------------------------------------------");
    // 根据年龄分组，如果大于等于18 成年否则未成年
    Map<String, List<Person>> map2 = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 18, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).collect(Collectors.groupingBy(p -> p.getAge() >= 18 ? "成年" : "未成年"));
    map2.forEach((k, v) -> System.out.println("k = " + k + "\t" + "v = " + v));
}
```

输出结果：

```txt
k = 李四	v = [Person(name=李四, age=22, height=177), Person(name=李四, age=15, height=166)]
k = 张三	v = [Person(name=张三, age=18, height=175), Person(name=张三, age=18, height=165), Person(name=张三, age=19, height=182)]
------------------------------------------
k = 未成年	v = [Person(name=李四, age=15, height=166)]
k = 成年	v = [Person(name=张三, age=18, height=175), Person(name=李四, age=22, height=177), Person(name=张三, age=18, height=165), Person(name=张三, age=19, height=182)]
```

多级分组：先根据name分组，然后根据年龄分组

```java
/**
 * 分组计算---多级分组
 */
@Test
public void test05() {
    // 先根据name分组，然后根据age(成年和未成年)分组
    Map<String, Map<String, List<Person>>> map = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 14, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).collect(Collectors.groupingBy(
        Person::getName,
        Collectors.groupingBy(p -> p.getAge() >= 18 ? "成年" : "未成年")
    )
             );
    map.forEach((k, v) -> {
        System.out.println(k);
        v.forEach((k1, v1) -> System.out.println("\t" + k1 + "=" + v1));
    });
}
```

输出结果：

```txt
李四
	未成年=[Person(name=李四, age=15, height=166)]
	成年=[Person(name=李四, age=22, height=177)]
张三
	未成年=[Person(name=张三, age=14, height=165)]
	成年=[Person(name=张三, age=18, height=175), Person(name=张三, age=19, height=182)]
```



### 5.5、对流中数据做分区操作

Collectors.partitioningBy会根据值是否为true，把集合中的数据分割为两个列表，一个true列表，一个false列表

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240410151836172.png" alt="image-20240410151836172" style="zoom: 80%;" />

```java
/**
 * 分区操作
 */
@Test
public void test06() {
    Map<Boolean, List<Person>> map = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 14, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).collect(Collectors.partitioningBy(p -> p.getAge() > 18));
    map.forEach((k, v) -> System.out.println(k + "\t" + v));
}
```

输出结果：

```txt
false	[Person(name=张三, age=18, height=175), Person(name=张三, age=14, height=165), Person(name=李四, age=15, height=166)]
true	[Person(name=李四, age=22, height=177), Person(name=张三, age=19, height=182)]
```



### 5.6、对流中的数据做拼接

Collectors.joining会根据指定的连接符将所有的元素连接成一个字符串

```java
/**
  对流中的数据做拼接操作
 */
@Test
public void test07() {
    String s1 = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 14, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).map(Person::getName).collect(Collectors.joining());
    System.out.println(s1); // 张三李四张三李四张三
    String s2 = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 14, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).map(Person::getName).collect(Collectors.joining("_"));
    System.out.println(s2); // 张三_李四_张三_李四_张三
    String s3 = Stream.of(
        new Person("张三", 18, 175),
        new Person("李四", 22, 177),
        new Person("张三", 14, 165),
        new Person("李四", 15, 166),
        new Person("张三", 19, 182)
    ).map(Person::getName).collect(Collectors.joining("_", "###", "$$$"));
    System.out.println(s3); // ###张三_李四_张三_李四_张三$$$
}
```



## 6、并行Stream流

### 6.1、串行Stream流

我们前面使用的Stream流都是串行的，也就是在一个线程上面执行

```java
/**
 * 串行流
 */
@Test
public void test01() {
    long count = Stream.of(5, 6, 8, 3, 1, 6)
        .filter(s -> {
            System.out.println(Thread.currentThread() + "\t" + s);
            return s > 3;
        }).count();
}
```

输出：

```txt
Thread[main,5,main]	5
Thread[main,5,main]	6
Thread[main,5,main]	8
Thread[main,5,main]	3
Thread[main,5,main]	1
Thread[main,5,main]	6
```



### 6.2、并行流

parallelStream其实就是一个并行执行的流，它通过默认的ForkJoinPool，可以提高多线程任务的速度



#### 6.2.1、获取并行流

我们可以通过两种方式获取并行流。

1. 通过List接口中的parallelStream方法来获取
2. 通过已有的串行流转换为并行流parallel

实现：

```java
/**
 * 获取并行流的两种方式
 */
public void test02() {
    List<Integer> list = new ArrayList<>();
    // 通过List接口直接获取并行流
    Stream<Integer> integerStream = list.parallelStream();
    // 将已有的串行流转换为并行流
    Stream<Integer> parallel = Stream.of(1, 2, 3).parallel();
}
```



#### 6.2.2、并行流操作

```java
/**
 * 并行流操作
 */
@Test
public void test03() {
    Stream.of(1, 4, 2, 6, 1, 5, 9)
        .parallel() // 将流转换为并发流，Stream处理的时候就会通过多线程处理
        .filter(s -> {
            System.out.println(Thread.currentThread() + " s=" + s);
            return s > 2;
        }).count();
}
```

输出结果：

```txt
Thread[main,5,main] s=1
Thread[ForkJoinPool.commonPool-worker-2,5,main] s=9
Thread[main,5,main] s=6
Thread[ForkJoinPool.commonPool-worker-9,5,main] s=4
Thread[ForkJoinPool.commonPool-worker-2,5,main] s=5
Thread[ForkJoinPool.commonPool-worker-11,5,main] s=1
Thread[ForkJoinPool.commonPool-worker-13,5,main] s=2
```



### 6.3、并行流和串行流对比

我们通过for循环、串行Stream流、并行Stream流来对5亿个数字求和，来看消耗时间

```java
public class Test03 {
    private static long times = 500000000;
    private long start;

    @Before
    public void before() {
        start = System.currentTimeMillis();
    }

    @After
    public void end() {
        long end = System.currentTimeMillis();
        System.out.println("消耗时间：" + (end - start));
    }

    /**
     * 普通for循环
     *      消耗时间：140
     */
    @Test
    public void test01() {
        System.out.println("普通for循环：");
        long res = 0;
        for (long i = 0; i < times; i++) {
            res += i;
        }
    }

    /**
     * 串行流
     *      消耗时间：189
     */
    @Test
    public void test02() {
        System.out.println("串行流：serialStream");
        LongStream.rangeClosed(0, times).reduce(0, Long::sum);
    }

    /**
     * 并行流
     *      消耗时间：110
     */
    @Test
    public void test03() {
        LongStream.rangeClosed(0, times).parallel().reduce(0, Long::sum);
    }
}
```

通过案例我们可以看到parallelStream的效率是最高的。

Stream并行处理的过程会分而治之，也就是将一个大的任务切分成了多个小任务，这表示每个任务都是一个线程操作。



### 6.4、线程安全问题

在多线程的处理下，肯定会出现数据安全问题，如下：

```java
/**
 * 并行流中的数据安全问题
 */
@Test
public void test01() {
    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < 1000; i++) {
        list.add(i);
    }
    System.out.println(list.size()); // 1000
    List<Integer> listNew = new ArrayList<>();
    // 使用并行流来向集合中添加数据
    list.parallelStream().forEach(listNew::add);
    System.out.println(listNew.size()); // 845
}
```

数量不一致或者直接抛异常。

针对这个问题，我们的解决方案有哪些呢？



#### 6.4.1、加同步锁

```java
/**
 * 加同步锁
 */
@Test
public void test02() {
    List<Integer> listNew = new ArrayList<>();
    Object obj = new Object();
    IntStream.rangeClosed(1, 1000)
        .parallel()
        .forEach(s -> {
            synchronized (obj) {
                listNew.add(s);
            }
        });
    System.out.println(listNew.size()); // 1000
}
```



#### 6.4.2、使用线程安全的容器

```java
/**
 * 使用线程安全的容器
 */
@Test
public void test03() {
    List<Integer> listNew = new Vector<>();
    IntStream.rangeClosed(1, 1000)
        .parallel()
        .forEach(listNew::add);
    System.out.println(listNew.size()); // 1000
}
```



#### 6.4.3、将线程不安全的容器转换为线程安全的容器

```java
/**
 * 将线程不安全的容器转换为线程安全的容器
 */
@Test
public void test04() {
    List<Integer> listNew = new ArrayList<>();
    // 将线程不安全的容器包装为线程安全的容器
    List<Integer> synchronizedList = Collections.synchronizedList(listNew);
    IntStream.rangeClosed(1, 1000)
        .parallel()
        .forEach(synchronizedList::add);
    System.out.println(synchronizedList.size()); // 1000
}
```



#### 6.4.4、通过Stream中的toArray方法或者collector方法操作

```java
/**
 * 通过Stream中的toArray方法或者collector方法操作 就是满足线程安全的要求
 */
@Test
public void test05() {
    List<Integer> listNew = IntStream.rangeClosed(1, 1000)
        .parallel()
        .boxed() // 转换为IntStream
        .collect(Collectors.toList());
    System.out.println(listNew.size()); // 1000
}
```



## 7、Fork/Join框架

parallelStream使用的是Fork/Join框架。Fork/Join框架自JDK7引入。Fork/Join框架可以将一个大任务拆分为很多小任务来异步执行。Fork/Join框架主要包含三个模块：

1. 线程池：ForkJoinPool
2. 任务对象：ForkJoinTask
3. 执行任务的线程：ForkJoinWorkerThread

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240411104359084.png" alt="image-20240411104359084" style="zoom:80%;" />



### 7.1、Fork/Join原理-分治法

ForkJoinPool主要用来使用分治法（Divide-and-Conquer Algorithm）来解决问题。典型的应用比如快速排序法。FokJoinPool需要使用相对较少的线程来处理大量的任务。比如要对1000万个数据进行排序，那么会将这个任务分割成两个500万的排序任务和一个针对这两组500万数据的合并任务。以此类推，对于500万的数据也会做出同样的分割处理，到最后会设置一个阈值来规定当数据规模到多少时，停止这样的分割处理。比如，当元素的数量小于10时，会停止分割，转而使用插入排序对它们进行排序。那么到最后，所有的任务加起来大概会有2000000+个。问题的关键在于，对于一个任务而言，只有当它所有的子任务完成之后，它才能够被执行。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240411105739124.png" alt="image-20240411105739124" style="zoom:67%;" />



### 7.2、Fork/Join原理-工作窃取法

Fork/Join最核心的地方就是利用了现代硬件设备多核，在一个操作时会有空闲的cpu，那么如何利用好这个空闲的cpu就成了提高性能的关键，而这里我们要提到的工作窃取（work-stealing）算法就是整个Fork/Join框架的核心理念。Fork/Join工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240411110420816.png" alt="image-20240411110420816" style="zoom:80%;" />

那么为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖
的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来
执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的
任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就
去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任
务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永
远从双端队列的尾部拿任务执行。
工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，
比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。
上文中已经提到了在Java 8引入了自动并行化的概念。它能够让一部分Java代码自动地以并行的方式执行，也就是我
们使用了ForkJoinPool的ParallelStream。
对于ForkJoinPool通用线程池的线程数量，通常使用默认值就可以了，即运行时计算机的处理器数量。可以通过设置
系统属性：java.util.concurrent.ForkJoinPool.common.parallelism=N （N为线程数量），来调整ForkJoinPool的线
程数量，可以尝试调整成不同的参数来观察每次的输出结果。



### 7.3、Fork/Join案例

需求：使用Fork/Join计算1到10000的和，当一个任务的计算数量大于3000的时候，拆分任务，数量小于3000的时候，就计算

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240411113804756.png" alt="image-20240411113804756" style="zoom: 80%;" />

案例的实现：

```java
public class Test05 {
    /**
     * 使用Fork/Join计算1到10000的和，当一个任务的计算数量大于3000的时候，拆分任务，数量小于3000的时候，就计算
     * @param args
     */
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        ForkJoinPool pool = new ForkJoinPool();
        SumRecursiveTask task = new SumRecursiveTask(1, 10000L);
        Long result = pool.invoke(task);
        System.out.println("result = " + result);
        long end = System.currentTimeMillis();
        System.out.println("总耗时：" + (end - start));
    }
}

class SumRecursiveTask extends RecursiveTask<Long> {
    // 定义一个拆分的临界值
    private static final long THRESHOLD = 3000L;

    private final long start;

    private final long end;

    public SumRecursiveTask(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long length = end - start;
        if (length <= THRESHOLD) {
            // 任务不用拆分，可以计算
            long sum = 0;
            for(long i = start; i <= end; i++) {
                sum += i;
            }
            System.out.println("计算：" + start + "----->" + end + "的结果为：" + sum);
            return sum;
        }else {
            // 数量大于预定的数量，说明任务还需要继续拆分
            long middle = (start + end) / 2;
            System.out.println("拆分：左边 " + start + "----->" + middle + "，右边 " + (middle + 1) + "----->" + end);
            SumRecursiveTask left = new SumRecursiveTask(start, middle);
            left.fork();
            SumRecursiveTask right = new SumRecursiveTask(middle + 1, end);
            right.fork();
            return left.join() + right.join();
        }
    }
}
```

输出结果：

```txt
拆分：左边 1----->5000右边 5001----->10000
拆分：左边 1----->2500右边 2501----->5000
计算：1----->2500的结果为：3126250
计算：2501----->5000的结果为：9376250
拆分：左边 5001----->7500右边 7501----->10000
计算：5001----->7500的结果为：15626250
计算：7501----->10000的结果为：21876250
result = 50005000
总耗时：3
```



# 七、Optional类

这个Optional类主要是解决空指针的问题。



## 1、以前对null的处理

```java
@Test
public void test01() {
    // String userName = "张三";
    String userName = null;
    if (userName != null) {
        System.out.println("字符串的长度：" + userName.length());
    }else {
        System.out.println("字符串为空");
    }
}
```



## 2、Optional类

Optional是一个没有子类的工具类，Optional是一个可以为null的容器对象，它的主要作用就是为了避免null检查，防止NullPointerException。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240411151949960.png" alt="image-20240411151949960" style="zoom:80%;" />



## 3、Optional的基本使用

Optional对象的创建方式：

```java
/**
 * Optional对象的创建方式
 */
@Test
public void test02() {
    // 第一种方式 通过of方法 of方法不支持null，执行会报错
    Optional<String> op1 = Optional.of("zhangsan");
    // Optional<Object> op2 = Optional.of(null);
    // 第二种方式 通过ofNullable方法 支持null
    Optional<String> op3 = Optional.ofNullable("lisi");
    Optional<Object> op4 = Optional.ofNullable(null);
    // 第三种方式 通过empty方法直接创建一个空的Optional对象
    Optional<Object> op5 = Optional.empty();
}
```



## 4、Optional的常用方法

```java
/**
 *  Optional的常用方法介绍
 *      get()：如果Optional有值则返回，否则抛出NoSuchElementException异常
 *              get()通常和isPresent方法一块使用
 *      isPresent()：判断是否包含值，包含值返回true，不包含值返回false
 *      orElse(T t)：如果调用对象包含值，就返回该值，否则返回t
 *      orElseGet(Supplier s)：如果调用对象包含值，就返回该值，否则返回lambda表达式的返回值
 */
@Test
public void test03() {
    Optional<String> op1 = Optional.of("zhangsan");
    Optional<String> op2 = Optional.empty();
    // 获取Optional中的值
    if (op1.isPresent()) {
        String s1 = op1.get();
        System.out.println("用户名称：" + s1);
    }
    if (op2.isPresent()) {
        System.out.println(op2.get());
    }else {
        System.out.println("op2是一个空Optional对象");
    }
    String s3 = op1.orElse("李四");
    System.out.println(s3);
    String s4 = op2.orElse("王五");
    System.out.println(s4);
    String s5 = op2.orElseGet(() -> "Hello");
    System.out.println(s5);
}
```

```java
/**
 * 自定义一个方法，将Person对象中的name转换为大写，并返回
 */
@Test
public void test05() {
    Person person = new Person("zhangsan", 18);
    String name = getNameForOptional(Optional.of(person));
    System.out.println("name = " + name);
}

/**
 * 根据Person对象将name转换为大写，并返回
 * @param op
 * @return
 */
public String getNameForOptional(Optional<Person> op) {
    if (op.isPresent()) {
        return op.map(Person::getName)
            .map(String::toUpperCase)
            .orElse("空值");
    }else {
        return null;
    }
}

/**
 * 根据Person对象将name转换为大写，并返回
 * @param person
 * @return
 */
public String getName(Person person) {
    if (person != null) {
        String name = person.getName();
        if (name != null) {
            return name.toUpperCase();
        }else {
            return null;
        }
    }else {
        return null;
    }
}
```



# 八、新时间日期API

## 1、旧版日期时间的问题

在旧版本中，JDK对于日期和时间的设计是非常差的。

```java
/**
 * 旧版日期时间设计的问题
 */
@Test
public void test01() throws Exception {
    // 1. 设计不合理
    Date date = new Date(2021, 05, 05);
    System.out.println(date); // Sun Jun 05 00:00:00 CST 3921
    // 2. 时间格式化和解析操作是线程不安全的
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    for (int i = 0; i < 50; i++) {
        new Thread(() -> {
            System.out.println(sdf.format(date));
            try {
                System.out.println(sdf.parse("2021-05-06"));
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        }).start();
    }
}
```

1. 设计不合理，在java.util和java.sql的包中，都有日期类，java.util.Date同时包含日期和时间，而java.sql.Date仅仅包含日期，此外，用于格式化和解析的类，在java.text包下
2. 非线程安全，java.util.Date是非线程安全的，所有的日期类都是可变的，这是java日期类最大的问题之一
3. 时区处理麻烦，日期类并不提供国际化，没有时区支持



## 2、新日期时间API介绍

JDK 8中增加了一套全新的日期时间API，这套API设计合理，是线程安全的。新的日期及时间API位于java.time 包中，下面是一些关键类：

- LocalDate：表示日期，包含年月日，格式为 2019-10-16
- LocalTime：表示时间，包含时分秒，格式为 16:38:54.158549300
- LocalDateTime：表示日期时间，包含年月日，时分秒，格式为 2018-09-06T15:33:56.750
- DateTimeFormatter：日期时间格式化类
- Instant：时间戳，表示一个特定的时间瞬间
- Duration：用于计算2个时间(LocalTime，时分秒)的距离
- Period：用于计算2个日期(LocalDate，年月日)的距离
- ZonedDateTime：包含时区的时间

Java中使用的历法是ISO 8601日历系统，它是世界民用历法，也就是我们所说的公历。平年有365天，闰年是366天。此外Java 8还提供了4套其他历法，分别是：

- ThaiBuddhistDate：泰国佛教历
- MinguoDate：中华民国历
- JapaneseDate：日本历
- HijrahDate：伊斯兰历



### 2.1、日期时间的常见操作

LocalDate，LocalTime，LocalDateTime

```java
/**
 * 新版JDK8日期时间操作
 */
@Test
public void test01() {
    // 1. 创建指定的日期
    LocalDate date1 = LocalDate.of(2021, 05, 06);
    System.out.println("date1 = " + date1); // date1 = 2021-05-06
    // 2. 得到当前的日期
    LocalDate now = LocalDate.now();
    System.out.println("now = " + now); // now = 2024-04-11
    // 3. 根据LocalDate对象获取对应的日期信息
    System.out.println("年：" + now.getYear());
    System.out.println("月：" + now.getMonth().getValue());
    System.out.println("日：" + now.getDayOfMonth());
    System.out.println("周：" + now.getDayOfWeek().getValue());
}

/**
 * 时间操作
 */
@Test
public void test02() {
    // 1. 得到指定的时间
    LocalTime time = LocalTime.of(5, 26, 33, 23145);
    System.out.println(time);
    // 2. 获取当前的时间
    LocalTime now = LocalTime.now();
    System.out.println(now);
    // 3. 获取时间信息
    System.out.println(now.getHour());
    System.out.println(now.getMinute());
    System.out.println(now.getSecond());
    System.out.println(now.getNano());
}

/**
 * 日期时间类型
 */
@Test
public void test03() {
    // 1. 获取指定的日期时间
    LocalDateTime dateTime = LocalDateTime.of(2020, 06, 01, 12, 12, 33, 213);
    System.out.println(dateTime); // 2020-06-01T12:12:33.000000213
    // 2.获取当前的日期时间
    LocalDateTime now = LocalDateTime.now();
    System.out.println(now); // 2024-04-11T16:52:25.109
    // 3. 获取日期时间信息
    System.out.println(now.getYear());
    System.out.println(now.getMonth().getValue());
    System.out.println(now.getDayOfMonth());
    System.out.println(now.getDayOfWeek().getValue());
    System.out.println(now.getHour());
    System.out.println(now.getMinute());
    System.out.println(now.getSecond());
    System.out.println(now.getNano());
}
```



### 2.2、日期时间的修改和比较

```java
/**
 * 日期时间的修改
 */
@Test
public void test01() {
    LocalDateTime now = LocalDateTime.now();
    System.out.println("now = " + now);
    // 修改日期时间 对日期时间的修改，对已存在的LocalDateTime对象，创建了它的模板
    // 并不会修改原来的信息
    LocalDateTime localDateTime = now.withYear(1998);
    System.out.println("now：" + now);
    System.out.println("修改后的：" + localDateTime);
    System.out.println("月份" + now.withMonth(10));
    System.out.println("天" + now.withDayOfMonth(6));
    System.out.println("小时" + now.withHour(8));
    System.out.println("分钟" + now.withMinute(15));
    // 在当前日期的基础上，加上或者减去指定的时间
    System.out.println("两天后：" + now.plusDays(2));
    System.out.println("十年后：" + now.plusYears(10));
    System.out.println("六个月后" + now.plusMonths(6));
    System.out.println("十年前：" + now.minusYears(10));
    System.out.println("半年前：" + now.minusMonths(6));
    System.out.println("一周前" + now.minusDays(7));
}

/**
 * 日期时间的比较
 */
@Test
public void test02() {
    LocalDate now = LocalDate.now();
    LocalDate date = LocalDate.of(2020, 1, 3);
    // 在JDK8中，要实现日期的比较
    System.out.println(now.isAfter(date)); // true
    System.out.println(now.isBefore(date)); // false
    System.out.println(now.isEqual(date)); // false
}
```

注意：在进行日期时间修改的时候，原来的LocalDate对象是不会被修改的，每次操作都是返回了一个新的LocalDate对象，所以在多线程场景下是数据安全的



### 2.3、格式化和解析操作

在JDK8中，我们可以通过`java.time.format.DateTimeFormatter`类可以进行日期的解析和格式化操作

```java
/**
 * 日期格式化操作
 */
@Test
public void test01() {
    LocalDateTime now = LocalDateTime.now();
    // 指定格式  使用系统默认的格式
    DateTimeFormatter isoLocalDateTime = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
    // 将日期时间转换为字符串
    String format = now.format(isoLocalDateTime);
    System.out.println("format = " + format); // format = 2024-04-11T17:15:09.755
    // 通过ofPattern方法来指定特定的格式
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String format1 = now.format(dateTimeFormatter);
    System.out.println("format1 = " + format1); // format1 = 2024-04-11 17:15:09
    // 将字符串解析为日期时间类型
    LocalDateTime parse = LocalDateTime.parse("1997-05-06 22:45:16", dateTimeFormatter);
    System.out.println("parse = " + parse);
}
```



### 2.4、Instant类

在JDK8中给我们新增了Instant类（时间戳/时间线），内部保存了从1970年1月1日 00:00:00以来的秒和纳秒

```java
/**
 * Instant 时间戳
 *      可以用来统计时间消耗
 */
@Test
public void test01() throws Exception {
    Instant now = Instant.now();
    System.out.println("now = " + now);
    // 获取从1970年1月1日 00:00:00 到现在的纳秒
    System.out.println(now.getNano()); // 96000000
    Thread.sleep(5000);
    Instant now1 = Instant.now();
    System.out.println("耗时：" + (now1.getNano() - now.getNano())); // 41000000
}
```



### 2.5、计算日期时间差

在JDK8中，提供了两个工具类Duration和Period：计算日期时间差

1. Duration：用来计算两个时间差(LocalTime)
2. Period：用来计算两个日期差(LocalDate)

```java
/**
 * 计算日期时间差
 */
@Test
public void test01() {
    // 计算时间差
    LocalTime now = LocalTime.now();
    LocalTime time = LocalTime.of(22, 48, 59);
    System.out.println("now = " + now);
    // 通过Duration来计算时间差
    Duration duration = Duration.between(now, time);
    System.out.println(duration.toDays());
    System.out.println(duration.toHours());
    System.out.println(duration.toMinutes());
    System.out.println(duration.toMillis());

    // 计算日期差
    LocalDate nowDate = LocalDate.now();
    LocalDate date = LocalDate.of(1997, 12, 5);
    Period period = Period.between(date, nowDate);
    System.out.println(period.getYears());
    System.out.println(period.getMonths());
    System.out.println(period.getDays());
}
```



### 2.6、时间校正器

有时候我们可能需要如下调整：将日期调整到“下个月的第一天”等操作。这时我们通过时间校正器效果可能会更好。

- TemporalAdjuster：时间校正器
- TemporalAdjusters：通过该类静态方法提供了大量的常用TemporalAdjuster的实现，可以简化我们的处理操作

```java
/**
 * 时间校正器
 */
@Test
public void test02() {
    LocalDateTime now = LocalDateTime.now();
    // 将当前的日期调整到下个月的一号
    TemporalAdjuster adjuster = (temporal) -> {
        LocalDateTime dateTime = (LocalDateTime) temporal;
        LocalDateTime nextMonth = dateTime.plusMonths(1).withDayOfMonth(1);
        System.out.println("nextMonth = " + nextMonth);
        return nextMonth;
    };
    // 我们还可以通过TemporalAdjusters 来实现
    // LocalDateTime nextMonth = now.with(adjuster);
    LocalDateTime nextMonth = now.with(TemporalAdjusters.firstDayOfNextMonth());
    System.out.println("nextMonth = " + nextMonth);
}
```



### 2.7、日期时间的时区

 Java8 中加入了对时区的支持，LocalDate、LocalTime、LocalDateTime是不带时区的，带时区的日期时间类分别为：ZonedDate、ZonedTime、ZonedDateTime。

其中每个时区都对应着 ID，ID的格式为 “区域/城市” 。例如 ：Asia/Shanghai 等。

ZoneId：该类中包含了所有的时区信息

```java
/**
 * 时区操作
 */
@Test
public void test01() {
    // 1. 获取所有的时区id
    // ZoneId.getAvailableZoneIds().forEach(System.out::println);
    // 获取当前时间 中国使用的东八区的时区，比标准时间早八个小时
    LocalDateTime now = LocalDateTime.now();
    System.out.println("now = " + now); // 2024-04-11T19:17:58.791
    // 获取标准时间
    ZonedDateTime bz = ZonedDateTime.now(Clock.systemUTC());
    System.out.println("bz = " + bz); // 2024-04-11T11:17:58.791Z

    // 使用计算机默认的时区，创建日期时间
    ZonedDateTime now1 = ZonedDateTime.now();
    System.out.println("now1 = " + now1); // 2024-04-11T19:17:58.791+08:00[Asia/Shanghai]

    // 使用指定的时区创建日期时间
    ZonedDateTime now2 = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
    System.out.println("now2 = " + now2); // 2024-04-11T19:19:35.193+08:00[Asia/Shanghai]
}
```



JDK新的日期和时间API的优势：

1. 新版日期时间API中，日期和时间对象是不可变的，操作日期不会影响原来的值，而是生成一个新的实例
2. 提供了不同的两种方式，有效的区分了人和机器的操作
3. TemporalAdjuster可以更精确的操作日期，还可以自定义日期调整器
4. 线程安全



# 九、其他新特性

## 1、重复注解

自从Java 5中引入 注解 以来，注解开始变得非常流行，并在各个框架和项目中被广泛使用。不过注解有一个很大的限制是：在同一个地方不能多次使用同一个注解。JDK 8引入了重复注解的概念，允许在同一个地方多次使用同一个注解。在JDK 8中使用@Repeatable注解定义重复注解。

1.1、定义一个重复注解的容器

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotations {
    MyAnnotation[] value();
}
```



1.2、定义一个可以重复的注解

```java
@Repeatable(MyAnnotations.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value();
}
```



1.3、配置多个重复的注解

```java
@MyAnnotation("test1")
@MyAnnotation("test2")
@MyAnnotation("test3")
public class AnnoTest {
    @MyAnnotation("fun1")
    @MyAnnotation("fun2")
    public void test01() {}
}
```



1.4、解析得到指定的注解

```java
@MyAnnotation("test1")
@MyAnnotation("test2")
@MyAnnotation("test3")
public class AnnoTest {
    @MyAnnotation("fun1")
    @MyAnnotation("fun2")
    public void test01() {}

    // 解析重复注解
    public static void main(String[] args) throws Exception {
        // 获取类中标注的重复注解
        MyAnnotation[] annotationsByType = AnnoTest.class.getAnnotationsByType(MyAnnotation.class);
        for (MyAnnotation myAnnotation : annotationsByType) {
            System.out.println(myAnnotation.value());
        }
        // 获取方法上标注的重复注解
        MyAnnotation[] test01s = AnnoTest.class.getMethod("test01").getAnnotationsByType(MyAnnotation.class);
        for (MyAnnotation myAnnotation : test01s) {
            System.out.println(myAnnotation.value());
        }
    }
}
```



## 2、类型注解

JDK 8为@Target元注解新增了两种类型： TYPE_PARAMETER ， TYPE_USE。

- TYPE_PARAMETER：表示该注解能写在类型参数的声明语句中。类型参数声明如：<T>
- TYPE_USE：表示注解可以再任何用到类型的地方使用



TYPE_PARAMETER 

```java
@Target(ElementType.TYPE_PARAMETER)
public @interface TypeParam {
}
```

使用：

```java
public class TypeDemo01 <@TypeParam T>{
    public <@TypeParam K extends Object> K test01() {
        return null;
    }
}
```



TYPE_USE

```java
@Target(ElementType.TYPE_USE)
public @interface NotNull {
}
```

使用：

```java
public class TypeUserDemo01 {
    public @NotNull Integer age = 10;
    public Integer sum(@NotNull Integer a, @NotNull Integer b) {
        return a + b;
    }
}
```

