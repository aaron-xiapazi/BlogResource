---
title: JAVA编程基础 （四） switch字符串
date: 2020-04-25 23:30:07
tags:
categories:
---

**switch** 有时也被划归为一种选择语句。根据整数表达式的值，**switch** 语句可以从一系列代码中选出一段去执行。它的格式如下：

```java
switch(integral-selector) {
    case integral-value1 : statement; break;
    case integral-value2 : statement;    break;
    case integral-value3 : statement;    break;
    case integral-value4 : statement;    break;
    case integral-value5 : statement;    break;
    // ...
    default: statement;
}
```

integral-selector是一个能够产生整数值的表达式，switch能够将整个整个表达式的结果与每个integral-value相比较，若发现相符的，就执行对应的语句，没有发现就执行default语句

## switch字符串

Java7 加上可字符串switch的用法，下面展示一组传统方法及新方法。

```java
// control/StringSwitch.java

public class StringSwitch {
  public static void main(String[] args) {
    String color = "red";
    // 老的方式: 使用 if-then 判断
    if("red".equals(color)) {
      System.out.println("RED");
    } else if("green".equals(color)) {
      System.out.println("GREEN");
    } else if("blue".equals(color)) {
      System.out.println("BLUE");
    } else if("yellow".equals(color)) {
      System.out.println("YELLOW");
    } else {
      System.out.println("Unknown");
    }
    // 新的方法: 字符串搭配 switch
    switch(color) {
      case "red":
        System.out.println("RED");
        break;
      case "green":
        System.out.println("GREEN");
        break;
      case "blue":
        System.out.println("BLUE");
        break;
      case "yellow":
        System.out.println("YELLOW");
        break;
      default:
        System.out.println("Unknown");
        break;
    }
  }
}
```

一旦理解了 **switch**，你会明白这其实就是一个逻辑扩展的语法糖。新的编码方式能使得结果更清晰，更易于理解和维护。

作为 **switch** 字符串的第二个例子，我们重新访问 `Math.random()`。 它是否产生从 0 到 1 的值，包括还是不包括值 1 呢？在数学术语中，它属于 (0,1)、 [0,1)、(0,1] 、[0,1] 中的哪种呢？（方括号表示“包括”，而括号表示“不包括”）

下面是一个可能提供答案的测试程序。 所有命令行参数都作为 **String** 对象传递，因此我们可以 **switch** 参数来决定要做什么。 那么问题来了：如果用户不提供参数 ，索引到 `args` 的数组就会导致程序失败。 解决这个问题，我们需要预先检查数组的长度，若长度为 0，则使用**空字符串** `""` 替代；否则，选择 `args` 数组中的第一个元素：

```java
// control/RandomBounds.java

// Math.random() 会产生 0.0 和 1.0 吗？
// {java RandomBounds lower}
import onjava.*;

public class RandomBounds {
  public static void main(String[] args) {
    new TimedAbort(3);
    switch(args.length == 0 ? "" : args[0]) {
      case "lower":
        while(Math.random() != 0.0)
          ; // 保持重试
        System.out.println("Produced 0.0!");
        break;
      case "upper":
        while(Math.random() != 1.0)
          ; // 保持重试
        System.out.println("Produced 1.0!");
        break;
      default:
        System.out.println("Usage:");
        System.out.println("\tRandomBounds lower");
        System.out.println("\tRandomBounds upper");
        System.exit(1);
    }
  }
}
```

要运行该程序，请键入以下任一命令：

```java
java RandomBounds lower 
// 或者
java RandomBounds upper复制ErrorOK!
```

使用 `onjava` 包中的 **TimedAbort** 类可使程序在三秒后中止。从结果来看，似乎 `Math.random()` 产生的随机值里不包含 0.0 或 1.0。 这就是该测试容易混淆的地方：若要考虑 0 至 1 之间所有不同 **double** 数值的可能性，那么这个测试的耗费的时间可能超出一个人的寿命了。 这里我们直接给出正确的结果：`Math.random()` 的结果集范围包含 0.0 ，不包含 1.0。 在数学术语中，可用 [0,1）来表示。由此可知，我们必须小心分析实验并了解它们的局限性。