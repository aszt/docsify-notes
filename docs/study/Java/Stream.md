# Stream流

> 链式编程流式计算

# 1. 概述
- what

  流（Steam）到底是什么呢？

  是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列

  “集合讲的是数据，流讲的是计算”

- why

  （1）Stream 自己不会存储元素

  （2）Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。

  （3）Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

- how

  （1）创建一个Stream：一个数据源（数组、集合）

  （2）中间操作：一个中间操作，处理数据源数据

  （3）终止操作：一个终止操作，执行中间操作链，产生结果

  

# 2. 四大函数式接口
## 2.1 函数型接口
```java
interface MyInterface {
    /**
     * Function<T,R>（函数型接口）
     * 参数类型T、返回类型R
     * 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。
     * 包含方法：R apply(T t);
     */
    public Integer myString(String str);
}
```
```java
// 函数式接口,有一个输入参数，一个输出参数
private static void fc() {
    Function<String, Integer> function = new Function<String, Integer>() {
        @Override
        public Integer apply(String s) {
            return 1024;
        }
    };
    System.out.println(function.apply("abc"));

    // lambda 表达式结合方法引用
    // 简写，一个参数类型可省略，括号可省略
    Function<String, Integer> function1 = s -> {
        return s.length();
    };
    System.out.println(function1.apply("abc"));
}
```

## 2.2 断定型接口
```java
interface MyInterface {
	/**
     * Predicate<T>（断定型接口）
     * 参数类型T、返回类型boolean
     * 确定类型为T的对象是否满足某约束，并返回boolean值
     * 包含方法：boolean test(T t);
     */
    public boolean isOk(String str);
}
```
```java
private static void pc() {
    Predicate<String> predicate = new Predicate<String>() {
        @Override
        public boolean test(String s) {
            return s.isEmpty();
        }
    };
    System.out.println(predicate.test(""));

    Predicate<String> predicate1 = s -> {
        return s.isEmpty();
    };
    System.out.println(predicate1.test("abc"));
}
```

## 2.3 消费型接口
```java
interface MyInterface {
	/**
     * Consumer<T>（消费型接口）
     * 参数类型T、返回类型void
     * 对类型为T的对象应用操作
     * 包含方法：void accept(T t);
     */
    public void pay(String str);
}
```
```java
private static void cs() {
    Consumer<String> consumer = new Consumer<String>() {
        @Override
        public void accept(String s) {
            System.out.println(s);
        }
    };
    consumer.accept("abc");

    Consumer<String> consumer1 = s -> {
        System.out.println(s);
    };
    consumer1.accept("abc");
}
```

## 2.1 供给型接口
```java
interface MyInterface {
	/**
     * Supplier<T>（供给型接口）
     * 参数无、返回类型为T的对象
     * 包含方法：T get();
     */
    public String getInfo();
}
```
```java
private static void sl() {
    Supplier<String> supplier = new Supplier<String>() {
        @Override
        public String get() {
            return "abc";
        }
    };
    System.out.println(supplier.get());

    Supplier<String> supplier1 = () -> {
        return "abc";
    };
    System.out.println(supplier1.get());
}
```
# 3. 流式计算
Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

```java
/**
 * 找出偶数ID且年龄大于24的用户
 * 用户名转大写按用户名字母倒排序
 * 只输出一个用户名字
 */
public static void main(String[] args) {
    User u1 = new User(11, "a", 23);
    User u2 = new User(12, "b", 24);
    User u3 = new User(13, "c", 22);
    User u4 = new User(14, "d", 28);
    User u5 = new User(16, "e", 26);

    List<User> users = asList(u1, u2, u3, u4, u5);

    users.stream().filter(u -> { return u.getId() % 2 == 0; })
            .filter(t->{return t.getAge()>24;})
            .map(m->{return m.getUserName().toUpperCase();})
            .sorted((o1,o2)->{ return o2.compareTo(o1); })
            .limit(1)
            .forEach(System.out::println);


    // 简写，return可去除（花括号去掉）
    users.stream().filter(u -> { return u.getId() % 2 == 0; })
            .filter(t-> t.getAge()>24)
            .map(m-> m.getUserName().toUpperCase())
            .sorted((o1,o2)-> o2.compareTo(o1))
            .limit(1)
            .forEach(System.out::println);
    }
```