---
title: 高效java （九） 考虑实现Compable接口
date: 2020-04-15 21:34:15
tags:
categories:
---

CompareTo方法并没有在`Obejct`类中声明。

它是`Comparable`接口中唯一的方法。

它与Object类的equals方法在性质上是相似的，除了它允许在简单的相等比较之外的顺序比较，它是泛型的。通过实现Comparable接口，一个类表明它的实例有一个自然顺序。对实现Comparable接口的对象数组排序非常简单。

```
Arrays.sort(a)
```

CompareTo方法的通用约定：

　　将此对象与指定的对象按照排序进行比较。 返回值可能为负整数，零或正整数，因为此对象对应小于，等于或大于指定的对象。 如果指定对象的类型与此对象不能进行比较，则引发 `ClassCastException` 异常。

　　下面的描述中，符号 sgn(expression) 表示数学中的 signum 函数，它根据表达式的值为负数、零、正数，对应返回-1、0 和 1。

- 实现类必须确保所有 `x` 和 `y` 都满足 `sgn(x.compareTo(y)) == -sgn(y. compareTo(x))`。 （这意味着当且仅当 `y.compareTo(x)` 抛出异常时，`x.compareTo(y)` 必须抛出异常。）
- 实现类还必须确保该关系是可传递的：`(x. compareTo(y) > 0 && y.compareTo(z) > 0)` 意味着 `x.compareTo(z) > 0`。
- 最后，对于所有的 z，实现类必须确保 `x.compareTo(y) == 0` 意味着 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`。
- 强烈推荐 `(x.compareTo(y) == 0) == (x.equals(y))`，但不是必需的。 一般来说，任何实现了 `Comparable` 接口的类违反了这个条件都应该清楚地说明这个事实。 推荐的语言是「注意：这个类有一个自然顺序，与 `equals` 不一致」。



---

　在 Java 8 中 `Comparator` 接口提供了一系列比较器方法，可以使比较器流畅地构建。 这些比较器可以用来实现 `compareTo` 方法，就像 `Comparable` 接口所要求的那样。 许多程序员更喜欢这种方法的简洁性，尽管它的性能并不出众：在我的机器上排序 PhoneNumber 实例的数组速度慢了大约 10％。 在使用这种方法时，考虑使用 Java 的静态导入，以便可以通过其简单名称来引用比较器静态方法，以使其清晰简洁。 以下是 PhoneNumber 的 `compareTo` 方法的使用方法：

```java
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
          .thenComparingInt(pn -> pn.prefix)
          .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}复制ErrorOK!
```

　　此实现在类初始化时构建比较器，使用两个比较器构建方法。第一个是 `comparingInt` 方法。它是一个静态方法，它使用一个键提取器函数式接口（key extractor function）作为参数，将对象引用映射为 int 类型的键，并返回一个根据该键排序的实例的比较器。在前面的示例中，`comparingInt` 方法使用 lambda 表达式，它从 `PhoneNumber` 中提取区域代码，并返回一个 `Comparator`，根据它们的区域代码来排序电话号码。注意，lambda 表达式显式指定了其输入参数的类型 (PhoneNumber pn)。事实证明，在这种情况下，Java 的类型推断功能不够强大，无法自行判断类型，因此我们不得不帮助它以使程序编译。

　　如果两个电话号码实例具有相同的区号，则需要进一步细化比较，这正是第二个比较器构建方法，即 `thenComparingInt` 方法做的。 它是 `Comparator` 上的一个实例方法，接受一个 int 类型键提取器函数式接口（key extractor function）作为参数，并返回一个比较器，该比较器首先应用原始比较器，然后使用提取的键来打破连接。 你可以按照喜欢的方式多次调用 `thenComparingInt` 方法，从而产生一个字典顺序。 在上面的例子中，我们将两个调用叠加到 `thenComparingInt`，产生一个排序，它的二级键是 prefix，而其三级键是 lineNum。 请注意，我们不必指定传递给 `thenComparingInt` 的任何一个调用的键提取器函数式接口的参数类型：Java 的类型推断足够聪明，可以自己推断出参数的类型。

　　`Comparator` 类具有完整的构建方法。对于 long 和 double 基本类型，也有对应的类似于 comparingInt 和 `thenComparingInt` 的方法，int 版本的方法也可以应用于取值范围小于 int 的类型上，如 short 类型，如 PhoneNumber 实例中所示。对于 double 版本的方法也可以用在 float 类型上。这提供了所有 Java 的基本数字类型的覆盖。

　　也有对象引用类型的比较器构建方法。静态方法 `comparing` 有两个重载方式。第一个方法使用键提取器函数式接口并按键的自然顺序。第二种方法是键提取器函数式接口和比较器，用于键的排序。`thenComparing` 方法有三种重载。第一个重载只需要一个比较器，并使用它来提供一个二级排序。第二次重载只需要一个键提取器函数式接口，并使用键的自然顺序作为二级排序。最后的重载方法同时使用一个键提取器函数式接口和一个比较器来用在提取的键上。

　　有时，你可能会看到 `compareTo` 或 `compare` 方法依赖于两个值之间的差值，如果第一个值小于第二个值，则为负；如果两个值相等则为零，如果第一个值大于，则为正值。这是一个例子：

```java
// BROKEN difference-based comparator - violates transitivity!

static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};复制ErrorOK!
```

　　不要使用这种技术！它可能会导致整数最大长度溢出和 IEEE 754 浮点运算失真的危险[JLS 15.20.1,15.21.1]。 此外，由此产生的方法不可能比使用上述技术编写的方法快得多。 使用静态 `compare` 方法：

```java
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};复制ErrorOK!
```

　　或者使用 `Comparator` 的构建方法：

```java
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder =
        Comparator.comparingInt(o -> o.hashCode());复制ErrorOK!
```

　　总而言之，无论何时实现具有合理排序的值类，你都应该让该类实现 `Comparable` 接口，以便在基于比较的集合中轻松对其实例进行排序，搜索和使用。 比较 `compareTo` 方法的实现中的字段值时，请避免使用「<」和「>」运算符。 相反，使用包装类中的静态 `compare` 方法或 `Comparator` 接口中的构建方法。