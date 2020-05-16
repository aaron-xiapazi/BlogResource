---
title: 高效java （六） 使用try-with-resources代替try-finally
date: 2020-04-11 15:03:59
tags:
categories:
---

通常情况我们在关闭资源的时候，会使用try-fianlly来关闭资源。

举个例子

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

看上去也没什么问题，很多情况下我们也都这么做了

但是往往有例外，如果在底层物理设备发生故障，我们可能在readline方法的调用发生了异常，且同样原因close也发生了异常。在这种情况下，第二个异常会冲掉第一个异常。在异常栈堆中没有第一个异常的记录，这可能使实际系统的调式非常复杂--因为原因你想要诊断的是第一个异常，虽然可以编写代码来异常这种异常，但是没有人会这么做，太冗长了。



**直到JDK7中主角登场 try-with-resources:**

实现这个构造，资源必须实现AutoCloseable接口，该接口返回为void的close组成。java类库和第三方类库中都大部分都继承或实现了AutoCloseable接口。如果你写的类必须关闭资源，请自行继承。

下面是实现的一个例子

```java
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
       return br.readLine();
    }
}
```

可以看到，这种方式比原始方式更精简，有更好的可读性，而且它们提供了更好的诊断，考虑上面我们说的问题，如果调用readline和close方法都抛出异常，则后一个异常将被抑制。为了保证你真正想看到的异常，可能会抑制多个异常。这些异常没有被抛弃，而是会打印在栈堆中跟踪，并且标注为被抑制了。



**结论**

在处理必须关闭的资源时，使用try-with-resources代替try-finally。

> 更简洁
>
> 更清晰
>
> 异常更有用
>
> 更不容易出错