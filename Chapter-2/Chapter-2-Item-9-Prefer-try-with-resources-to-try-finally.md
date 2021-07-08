## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 9: Prefer try-with-resources to try-finally（使用 try-with-resources 优于 try-finally）

The Java libraries include many resources that must be closed manually（adv.手动地） by invoking a close method. Examples include InputStream,OutputStream, and java.sql.Connection. Closing resources is often overlooked（n.被忽视的；v.忽视） by clients, with predictably（adv.可以预见的是） dire（可怕的） performance consequences. While many of these resources use finalizers as a safety net, finalizers don’t work very well (Item 8).

**finalizers 并不靠谱，还是有很多必须通过close方法手动关闭的资源。**

Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return:

**从历史上看，try-finally 语句是确保正确关闭资源的最佳方法，即使在出现异常或返回时也是如此：**

```
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

This may not look bad, but it gets worse when you add a second resource:

**这可能看起来不坏，但添加第二个资源时，情况会变得更糟：**

```
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
    try {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    } finally {
        out.close();
        }
    }
    finally {
        in.close();
    }
}
```

It may be hard to believe, but even good programmers got this wrong most of the time. For starters, I got it wrong on page 88 of Java Puzzlers [Bloch05], and no one noticed for years. In fact, two-thirds of the uses of the close method in the Java libraries were wrong in 2007.

**这可能难以置信。在大多数情况下，即使是优秀的程序员也会犯这种错误。首先，我在 Java Puzzlers [Bloch05]的 88 页上做错了，多年来没有人注意到。事实上，2007 年发布的 Java 库中三分之二的 close 方法使用都是错误的。**

**译注：《Java Puzzlers》的中文译本为《Java 解惑》**

Even the correct code for closing resources with try-finally statements,as illustrated（v.阐明） in the previous two code examples, has a subtle deficiency（微妙的不足）. 

The code in both the try block and the finally block is capable of throwing exceptions. For example, in the firstLineOfFile method, the call to readLine could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. 

Under these circumstances, the second exception completely obliterates（vt.抹去） the first one. 

**第二个异常将完全覆盖第一个异常**

There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you want to see in order to diagnose the problem. While it is possible to write code to suppress the second exception in favor of the first, virtually no one did because it’s just too verbose.



All of these problems were solved in one fell swoop when Java 7 introduced the try-with-resources statement [JLS, 14.20.3]. 

**当 Java 7 引入 try-with-resources 语句 ，所有这些问题都一次性解决了。**

To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method. 

**必须实现 AutoCloseable 接口**

Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.

Here’s how our first example looks using try-with-resources:

下面是使用 try-with-resources 的第一个示例：

```
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

And here’s how our second example looks using try-with-resources:

下面是使用 try-with-resources 的第二个示例：

```
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

originals, but they provide far better diagnostics. Consider the firstLineOfFile method. If exceptions are thrown by both the readLine call and the (invisible) close, the latter exception is suppressed in favor of the former. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see. 

These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also access them programmatically with the getSuppressed method, which was added to Throwable in Java 7.

**try-with-resources真的牛皮，可以抑制多个异常；并且被抑制的异常不会被抛弃，而被打印到stack trace（可以通过getSuppressed 方法访问到他们）。**

You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without sullying（玷污） your code with another layer of nesting. As a slightly（顺手） contrived（人为的） example（可以翻译为信手拈来）, here’s a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can’t open the file or read from it:

**还可以加catch，下面是信手拈来的例子：**

```
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

The lesson is clear: Always use try-with-resources in preference to tryfinally when working with resources that must be closed. 

The resulting code is shorter and clearer, and the exceptions that it generates are more useful. 

The try-with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using try finally.

**使用 try-finally 几乎是不可能写出正确的代码，推荐使用try-with-resources。**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 8: Avoid finalizers and cleaners（避免使用终结器和清除器）](../Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)**
- **Next Item（下一条目）：[Chapter 3 Introduction（章节介绍）](../Chapter-3/Chapter-3-Introduction.md)**
