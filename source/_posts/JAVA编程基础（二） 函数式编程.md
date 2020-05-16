---
title: JAVA编程基础（二）函数式编程
date: 2020-02-18 22:34:24
tags: JAVA
categories:笔记
---

# JAVA函数式编程

函数式接口：他指的是有且只有一个未实现的方法接口，一般通过Functionallnterface这个注解来表明某个接口是一个函数式接口。

函数式编程在Java8引入

如果全篇对你来说无法完全理解，请记住一句话

**函数式编程让java拥有了更简单的行为参数话的能力**    

 ---我从别人那听来的。

本文起源于一段代码

```
Arrays.sort(intervals, Comparator.comparingInt(o -> o[1]));
```

我不知道什么意思，希望看完文章的你也能明白。

---

## 1 语法

使用Consumer作为示例，它是一个函数式接口，包含一个抽象方法accept，这个方法只有输入而无输出。

- 在定义一个传统的Consumer对象，是这样定义的

```Java
Consumer c = new Consumer(){
  @Override
  public void accept (Object o){
    System.out.println(0);
  }
}
```

- 而函数式编程

```
Consumer c = (o) -> {
  System.out.println(o);
}
```

或者：

```Java
Consumer c = (o) -> System.out.print(o);
```

甚至：

```Java
Consumer c = System.out::println;
```

**惊不惊喜，意不意外？**

下面开始函数式接口分析

## 2 Java函数式接口

### 2.1 Consumer

Java提供的Consumer(也就是消费类)，包含了一个有输入无输出的accept方法，和一个andThen方法

``` 
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```

下面是使用示例：

``` 
public static void consumerTest(){
        Consumer f = System.out::println;
        Consumer f2 = n -> System.out.println(n + "F2");

        f.andThen(f2).accept("test");

        f.andThen(f).andThen(f2).andThen(f).accept("test1");
    }
```

如果你执行代码，你将会看到这样的结果

test
testF2
test1
test1
test1F2
test1

请根据如上代码自行理解，或在IDE中debug。

### 2.2 Function

Function也是一个函数式编程的接口，有apply,compose,andthen,indentity 4个方法

直接上代码

``` 
public static void functionTest() {
        Function<Integer, Integer> f = s -> ++s;
        Function<Integer, Integer> g = s -> s * 2;

        /**
         * 下面表示在执行F时，先执行G，并且执行F时使用G的输出当作输入。
         * 相当于以下代码：
         * Integer a = g.apply(1);
         * System.out.println(f.apply(a));
         */
        System.out.println(f.compose(g).apply(1));
        System.out.println(f.apply(1));

        /**
         * 表示执行F的Apply后使用其返回的值当作输入再执行G的Apply；
         * 相当于以下代码
         * Integer a = f.apply(1);
         * System.out.println(g.apply(a));
         */
        System.out.println(f.andThen(g).apply(1));

        /**
         * identity方法会返回一个不进行任何处理的Function，即输出与输入值相等；
         */
        System.out.println(Function.identity().apply("a"));
    }
```

结果：

3
2
4
a

注释的很详细，此处不过多赘述。这是一个风格，需要在认知里对函数式编程有概念，才会合理的运用它们。

### 2.3 Predicate

```
/**
 * Predicate测试
 */
private static void predicateTest() {
    Predicate<String> p = o -> o.equals("test");
    Predicate<String> g = o -> o.startsWith("t");

    /**
     * negate: 用于对原来的Predicate做取反处理；
     * 如当调用p.test("test")为True时，调用p.negate().test("test")就会是False；
     */
    Assert.assertFalse(p.negate().test("test"));

    /**
     * and: 针对同一输入值，多个Predicate均返回True时返回True，否则返回False；
     */
    Assert.assertTrue(p.and(g).test("test"));

    /**
     * or: 针对同一输入值，多个Predicate只要有一个返回True则返回True，否则返回False
     */
    Assert.assertTrue(p.or(g).test("ta"));
}
```







# Comparator接口

对集合或数组排序

```
Collections.sort(stus,new Comparator<Student>(){
  @Override
  public int compare(Student s1,Student s2){
    //升序
    return s1.getAge()-s2.getAge();
    //return s1.getAge.compareTo(s2.getAge());
  }
});
```

替换成lambda的话

```
Collections.sort(stus,(s1,s2) -> (s1.getAge()-s2.getAge()));
```

或者！

```
Collections.sort(stus, Comparator.comparingInt(Student::getAge));
```



lambda的缺点：

- 难以维护（有些人难以接受，存在学习成本）
- 效率低（此点没有亲自考究，但是确实有LeetCode上lambda不通过改成匿名内部类通过的情况）



---



所以，我们解决了这段代码的含义。

Arrays.sort(intervals, Comparator.comparingInt(o -> o[1]));