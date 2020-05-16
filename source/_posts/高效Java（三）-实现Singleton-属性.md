---
title: 高效Java（三） 实现Singleton 属性
date: 2020-04-08 13:54:57
tags:
categories:
---

**单例是一个仅实例化一次的类。同创表示无状态对象。**

下面介绍一下实现单例化的三种方式：

- 私有构造方法只调用一次，来初始化Elvis.INSTANCE属性。缺少一个公告的或受保护的构造方法，来保证全局的唯一性。

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```

- 对所有对象都返回相同的对象引用

```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

- 声明单一元素的枚举类，类型公共属性方法，但是更简洁，无偿地提供了序列化机智，并提供了防止多个实例化的坚固保证。**单一元素枚举通常是实现单例的最佳方式**

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```