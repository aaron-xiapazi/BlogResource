---
title: 高效java （八）谨慎地重写clone方法
date: 2020-04-14 14:47:09
tags:
categories:
---



**Cloneable接口不包含任何方法。**

你可能想说 WTF！那它用来做什么？



​	它决定了Obejct的受保护的clone方法实现的行为：如果一个类实现了Cloneable接口，那么Object的clone方法将返回该对象的逐个属性拷贝；否则会抛出`CloneNotSupportedException`异常。这是一个非常反常的接口使用，不应该被效仿。通常情况下，实现一个接口用来表示可以为客户做什么。但是对于Cloneable接口，它会修改父类上受保护的方法的行为。

​	虽然规范并没有说明，但是在实践中，实现Cloneable接口的类希望提供一个正常的公共clone方法。为了实现这一目标，该类以及所有父类必须遵循一个复杂的，不可执行的，稀疏的文档协议。由此产生的机制是脆弱的、危险的和不受语言影响的：它创建对象而不需要调用构造方法。



以下是从Object规范中截取的部分：

- 对于任何对象，x.clone() != x == true
- x.clone().getClass() =  x.getClass() == true
- x.clone().equals(x) == true
- 以上要求为通常情况下的，并不是绝对的



​	假如你希望在一个类中实现Cloneable接口，它的父类提供了一个行为料号的clone方法。首先调用super.clone。得到的对象僵尸院士的完全功能的复制品。在你的类中声明的任何属性将具有与原始属性相同的值。如果每个属性包含原始值或对不可变对象的引用，则返回的对象可能正是你所需要的，在这种情况下，不需要进一步处理。但请注意，不可变类永远不应该提供clone方法，因为只实惠浪费复制。

```java
// Clone method for class with no references to mutable state
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();  // Can't happen
    }
}
```

​	为了使这个方法起作用，`PhoneNumber` 的类声明必须被修改，以表明它实现了 Cloneable 接口。 虽然 Object 类的 clone 方法返回 Object 类，但是这个 clone 方法返回 `PhoneNumber` 类。 这样做是合法和可取的，因为 Java 支持协变返回类型。 换句话说，重写方法的返回类型可以是重写方法的返回类型的子类。 这消除了在客户端转换的需要。 在返回之前，我们必须将 Object 的 super.clone 的结果强制转换为 `PhoneNumber`，但保证强制转换成功。

　　super.clone 的调用包含在一个 try-catch 块中。 这是因为 Object 声明了它的 clone 方法来抛出 `CloneNotSupportedException` 异常，这是一个检查时异常。 由于 `PhoneNumber` 实现了 Cloneable 接口，所以我们知道调用 super.clone 会成功。 这里引用的需要表明 `CloneNotSupportedException` 应该是未被检查的



　　**考虑到与 Cloneable 接口相关的所有问题，新的接口不应该继承它，新的可扩展类不应该实现它。 虽然实现 Cloneable 接口对于 final 类没有什么危害，但应该将其视为性能优化的角度，仅在极少数情况下才是合理的。 通常，复制功能最好由构造方法或工厂提供。 这个规则的一个明显的例外是数组，它最好用 clone 方法复制。**