---
title: 高效Java（四）避免创建不必要的对象
date: 2020-04-09 13:28:03
tags:
categories:
---

尽量在需要时重用一个对象而不要创建一个新的相同功能对象，这听上去虽然是一句废话。但是我们在实际场景中往往会忽略掉。本文举两个场景来介绍：

上例子：

```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

VS

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

**结论：方法2往往会比方法1高几倍**

---

这个实现的问题在于它依赖于String.matches方法。虽然String.matched检查字符串是否匹配是实现正则表达式最简单的方式，但是不适合用于性能临界的情况下重复使用。原理是它在内部为正则表达式创建了一个Patter示例，并且只使用它一次，之后它就有资格进行垃圾回收回收。创建Pattern是很昂贵的仓做，因此在多次调用的情况下，将它编译为一个Pattern实例是更优秀的做法。

### 自动装箱导致的不必要创建对象

例子：

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

这是一个正确的例子，因为java允许程序员混用基本类型和包装的基本类型，根据需要自动装箱和拆箱。在例子中程序构造了大学2的31方个Long的示例，因为每次往Long类型的sum变量中增加一个long类型构造的示例都要把变量的类型从Long改为long。

如果我们讲Long  改为 long 数据类型。 这将会省去大量的时间，如这个例子，在机器上运行的时间将会有6.3s减低至0.59秒。



---

说在最后：

**这个章节不应该被误解为我们应该尽量避免创建对象，相反，使用构造方法创建和回收小的对象是很廉价的，构造方法只会做很少的显示工作，尤其在JVM上，创建额外的对象以增强程序的清晰度，简单性或功能通常是件好事**

**除非池中的对象非常重量级，否则通过维护自己的对象池来避免对象创建是一个坏主意。对象池的典型例子就是数据连接。连接成本非常的搞，因此重用这些对象是有意义的。但是一般来说，维护自己的对象池会使代码混乱，增加内存占用，并损害性能。现在的JVM具有高度优化的垃圾收集器，还是挺厉害的。**