---
title: 高效java （七） 重写 equals 方法
date: 2020-04-12 11:24:02
tags:
categories:
---

虽说Obejct类是一个具体的类，但它主要是为了继承而设计的。

它的所有非final方法都有清晰的约定，如果你不这么做，将会阻止其他依赖于约定的类，与此类一起工作。

**什么情况下需要重写equals方法呢？**

例：当你需要一个逻辑相等的概念，例如Integer或者String，程序员使用equals方法比较值对象的引用，期望发现它们在逻辑上相等。

当你重写equals方法，Object的规范如下：

- **自反性**：对于任何非空引用 x，x.equals(x) 必须返回true
- **对称性**：对于任何非空引用x、y，如果且仅当 y.equals(x)返回true时，x.equals(y)必须返回true
- **传递性**：对于任何非空引用x、y、z，如果x.equals(y)返回true，y.equals(z) 返回true，则x.equals(z)必须返回true
- **一致性**：对于任何非空引用x、y，如果在equals比较中使用的信息没有修改，则x.equals(y)的多次调用必须返回true
- **非空性**（这个名字不是官方定义的）：对于任何非空引用，x.equals(null)必须返回false



编写和测试 equals（和 hashCode）方法很繁琐，生的代码也很普通。替代手动编写和测试这些方法的优雅的手段是，使用谷歌 AutoValue 开源框架，该框架自动为你生成这些方法，只需在类上添加一个注解即可。在大多数情况下，AutoValue 框架生成的方法与你自己编写的方法本质上是相同的。

　　很多 IDE（例如 Eclipse，NetBeans，IntelliJ IDEA 等）也有生成 equals 和 hashCode 方法的功能，但是生成的源代码比使用 AutoValue 框架的代码更冗长、可读性更差，不会自动跟踪类中的更改，因此需要进行测试。这就是说，使用 IDE 工具生成 equals(和 hashCode) 方法通常比手动编写它们更可取，因为 IDE 工具不会犯粗心大意的错误，而人类则会。

**除非必须：在很多情况下，不要重写 equals 方法，从 Object 继承的实现完全是你想要的。 如果你确实重写了 equals 方法，那么一定要比较这个类的所有重要属性，并且以保护前面 equals 约定里五个规定的方式去比较。**



---



## 重写equals方法同时也要重写hashcode方法

**在每个类中，在重写equals时，一定要重写hashcode**。如果不这么做，你的类就违反了hashcode的约定，这会阻止它在HashMap和HashSet这样的集合中正常工作。Object这样规范：

- 如果没有修改equals方法中用以比较的信息，在应用程序的一次执行过程中对一个对象重复调用hashcode方法时，它必须始终返回相同的值。在应用程序多次执行过程中，每个执行过程在该对象上获取的结果可以不相同。
- 如果像个对象根据equals(Obejct)方法比较是相等的，那么在两个对象上调用hashcode就必须产生的结果是相同的整数
- 如果两个对象根据equals(Object)方法比较并不相等，则要求在每个对象上调用hashcode都必须产生不同的结果。但是，程序员应该意识到，为不相等的对象生成不同的结果可能会提高散列表的性能

#### 当无法重写hashcode时，所违反的第二个关键条款：相同的对象必须具有相等的哈希码。

根据类的equals方法，两个不同的示例可能在逻辑上是相同的，但是对于Object类的hashcode方法，它们知识两个没有什么共同之处的对象。因此，Object类的hashcode方法返回两个看似灰机的数字，而不是按约定要求的两个相等的数字。

举个例子

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

如果没重写hashcode，你本期望`m.get(new PhoneNumber(707, 867, 5309))` 方法返回 `Jenny` 字符串，但是只会返回null。

注意，这里涉及到两个 `PhoneNumber` 实例：一个实例插入到 HashMap 中，另一个作为判断相等的实例用来检索。`PhoneNumber` 类没有重写 hashCode 方法导致两个相等的实例返回了不同的哈希码，违反了 hashCode 约定。put 方法把 `PhoneNumber` 实例保存在了一个哈希桶（ hash bucket）中，但 get 方法却是从不同的哈希桶中去查找，即使恰好两个实例放在同一个哈希桶中，get 方法几乎肯定也会返回 null。因为 HashMap 做了优化，缓存了与每一项（entry）相关的哈希码，如果哈希码不匹配，则不会检查对象是否相等了。



 以下是一个简单的配方：

1. 声明一个 int 类型的变量 result，并将其初始化为对象中第一个重要属性 `c` 的哈希码，如下面步骤 2.a 中所计算的那样。（回顾条目 10，重要的属性是影响比较相等的领域。）

2. 对于对象中剩余的重要属性 `f`，请执行以下操作：
   a. 比较属性 `f` 与属性 `c` 的 int 类型的哈希码：

     -- i. 如果这个属性是基本类型的，使用 `Type.hashCode(f)` 方法计算，其中 `Type` 类是对应属性 `f` 基本类型的包装类。
     -- ii. 如果该属性是一个对象引用，并且该类的 equals 方法通过递归调用 equals 来比较该属性，并递归地调用 hashCode 方法。 如果需要更复杂的比较，则计算此字段的“范式（“canonical representation）”，并在范式上调用 hashCode。 如果该字段的值为空，则使用 0（也可以使用其他常数，但通常来使用 0 表示）。
      -- iii. 如果属性 `f` 是一个数组，把它看作每个重要的元素都是一个独立的属性。 也就是说，通过递归地应用这些规则计算每个重要元素的哈希码，并且将每个步骤 2.b 的值合并。 如果数组没有重要的元素，则使用一个常量，最好不要为 0。如果所有元素都很重要，则使用 `Arrays.hashCode` 方法。

   b. 将步骤 2.a 中属性 c 计算出的哈希码合并为如下结果：`result = 31 * result + c;`

3. 返回 result 值。

　　当你写完 hashCode 方法后，问自己是否相等的实例有相同的哈希码。 编写单元测试来验证你的直觉（除非你使用 AutoValue 框架来生成你的 equals 和 hashCode 方法，在这种情况下，你可以放心地忽略这些测试）。 如果相同的实例有不相等的哈希码，找出原因并解决问题。

　　可以从哈希码计算中排除派生属性（derived fields）。换句话说，如果一个属性的值可以根据参与计算的其他属性值计算出来，那么可以忽略这样的属性。您必须排除在 equals 比较中没有使用的任何属性，否则可能会违反 hashCode 约定的第二条。

　　步骤 2.b 中的乘法计算结果取决于属性的顺序，如果类中具有多个相似属性，则产生更好的散列函数。 例如，如果乘法计算从一个 String 散列函数中被省略，则所有的字符将具有相同的散列码。 之所以选择 31，因为它是一个奇数的素数。 如果它是偶数，并且乘法溢出，信息将会丢失，因为乘以 2 相当于移位。 使用素数的好处不太明显，但习惯上都是这么做的。 31 的一个很好的特性，是在一些体系结构中乘法可以被替换为移位和减法以获得更好的性能：`31 * i ==（i << 5） - i`。 现代 JVM 可以自动进行这种优化。

　　让我们把上述办法应用到 `PhoneNumber` 类中：

```java
// Typical hashCode method
@Override 
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

　　因为这个方法返回一个简单的确定性计算的结果，它的唯一的输入是 `PhoneNumber` 实例中的三个重要的属性，所以显然相等的 `PhoneNumber` 实例具有相同的哈希码。 实际上，这个方法是 `PhoneNumber` 的一个非常好的 hashCode 实现，与 Java 平台类库中的实现一样。 它很简单，速度相当快，并且合理地将不相同的电话号码分散到不同的哈希桶中。





---

总之 ，每次重写equals都必须重写hashcode，否则程序将无法正常运行！