---
title: 高效Java-（二）-静态工厂替代构造方法
date: 2020-04-07 01:31:02
tags: java
categories: 
---

### 构造方法过多时，建议使用builder模式

常用的JavaBean缺点：

> 由于构造方法被分割成了多次调用，所以在构造过程中，JavaBean可能处于不一致的状态。
>
> 导致javabean的方法虽然简单易用，但是在不一致的状态下尝试使用对象可能导致一些错误
>
> javabean模式排除了让类不可变得可能性，所以需要增加工序以保证线程安全

---

Builder模式：**结合了可伸缩构造方法模式的安全性和JavaBean模式的可读性**

`可伸缩构造方法：为各种可选参数单独写构造方法`

客户端不需要直接构造所需的对象，而是调用一个包含所有必须参数的构造方法得到一个builder对象。然后调用builder对象的与setter相似的方法来设计可选参数。

实例:

```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val) { 
            calories = val;      
            return this;
        }

        public Builder fat(int val) { 
           fat = val;           
           return this;
        }

        public Builder sodium(int val) { 
           sodium = val;        
           return this; 
        }

        public Builder carbohydrate(int val) { 
           carbohydrate = val;  
           return this; 
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

　　`NutritionFacts` 类是不可变的，所有的参数默认值都在一个地方。builder 的 setter 方法返回 builder 本身，这样就可以进行链式调用，从而生成一个流畅的 API。下面是客户端代码的示例：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();
```





### Builder 还非常适合类层次结构

使用平行层次的 builder，每个builder嵌套在相应的类中。 抽象类有抽象的 builder；具体的类有具体的 builder。 例如，考虑代表各种比萨饼的根层次结构的抽象类：

```java
// Builder pattern for class hierarchies
import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;

public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Subclasses must override this method to return "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```

请注意，`Pizza.Builder` 是一个带有递归类型参数（ recursive type parameter）（详见第 30 条）的泛型类型。 这与抽象的 `self` 方法一起，允许方法链在子类中正常工作，而不需要强制转换。 Java 缺乏自我类型的这种变通解决方法被称为模拟自我类型（simulated self-type）。

　　这里有两个具体的 `Pizza` 的子类，其中一个代表标准的纽约风格的披萨，另一个是半圆形烤乳酪馅饼。前者有一个所需的尺寸参数，而后者则允许指定酱汁是否应该在里面或在外面：

```java
import java.util.Objects;

public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() {
            return this; 
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

请注意，每个子类 builder 中的 `build` 方法被声明为返回正确的子类：`NyPizza.Builder` 的 `build` 方法返回 `NyPizza`，而 `Calzone.Builder` 中的 `build` 方法返回 `Calzone`。 这种技术，其一个子类的方法被声明为返回在超类中声明的返回类型的子类型，称为协变返回类型（covariant return typing）。 它允许客户端使用这些 builder，而不需要强制转换。

　　这些「分层 builder（hierarchical builders）」的客户端代码基本上与简单的 `NutritionFacts` builder 的代码相同。为了简洁起见，下面显示的示例客户端代码假设枚举常量的静态导入：

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```



---



​          **总而言之，当设计类的构造方法或静态工厂的参数超过几个时，Builder 模式是一个不错的选择，特别是许多参数是可选的或相同类型的。builder模式客户端代码比使用伸缩构造方法（telescoping constructors）更容易读写，并且builder模式比 JavaBeans 更安全。**