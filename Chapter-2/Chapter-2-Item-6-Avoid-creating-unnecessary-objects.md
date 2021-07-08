## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）

It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish（优雅的）. An object can always be reused if it is immutable (Item 17).

As an extreme example of what not to do, consider this statement:

```
String s = new String("bikini"); // DON'T DO THIS!
```

The statement creates a new String instance each time it is executed, and none of those object creations is necessary. The argument to the String constructor ("bikini") is itself a String instance, functionally identical to all of the objects created by the constructor. If this usage occurs in a loop or in a frequently invoked method, millions of String instances can be created needlessly.

The improved version is simply the following:

```
String s = "bikini";
```

This version uses a single String instance, rather than creating a new one each time it is executed. Furthermore, it is guaranteed that the object will be reused by any other code running in the same virtual machine that happens to contain the same string literal [JLS, 3.10.5].

**相同的字符串字面量**

You can often avoid creating unnecessary objects by using static factory methods (Item 1) in preference to constructors on immutable classes that provide both. 

**静态工厂方法 > 构造方法，因为构造方法每次调用都会构造出一个新的实例出来。**

For example, the factory method Boolean.valueOf(String) is preferable to the constructor Boolean(String), which was deprecated in Java 9. 

The constructor must create a new object each time it’s called, while the factory method is never required to do so and won’t in practice. 

In addition to reusing immutable objects, you can also reuse mutable objects if you know they won’t be modified.

**不可变对象可以reuse**

Some object creations are much more expensive than others. If you’re going to need such an “expensive object” repeatedly, it may be advisable（adj.明智的，适当的） to cache it for reuse. Unfortunately, it’s not always obvious when you’re creating such an object. 

**昂贵的对象需要reuse**

Suppose you want to write a method to determine（v.下决心；vt.确定） whether a string is a valid Roman numeral. Here’s the easiest way to do this using a regular expression:

```
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

The problem with this implementation is that it relies on the String.matches method. **While String.matches is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations.** The problem is that it internally creates a Pattern instance for the regular expression and uses it only once, after which it becomes eligible（可以选） for garbage collection. Creating a Pattern instance is expensive because it requires compiling the regular expression into a finite state machine.

**将正则表达式编译成有限状态机（参考算法4）**

To improve the performance, explicitly compile the regular expression into a Pattern instance (which is immutable) as part of class initialization, cache it,and reuse the same instance for every invocation of the isRomanNumeral method:

```
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

The improved version of isRomanNumeral provides significant performance gains if invoked frequently. On my machine, the original version takes 1.1 μs on an 8-character input string, while the improved version takes 0.17 μs, which is 6.5 times faster. Not only is the performance improved, but arguably（可以说）, so is clarity（明晰）. Making a static final field for the otherwise invisible Pattern instance allows us to give it a name, which is far more readable than the regular expression itself.

If the class containing the improved version of the isRomanNumeral method is initialized but the method is never invoked, the field ROMAN will be initialized needlessly. It would be possible to eliminate the initialization by lazily initializing the field (Item 83) the first time the isRomanNumeral method is invoked, but this is not recommended. As is often the case with lazy initialization, it would complicate the implementation with no measurable performance improvement (Item 67).

**lazily initializing可以有但是没有必要**

When an object is immutable, it is obvious it can be reused safely, but there are other situations where it is far less obvious, even counterintuitive（违法直觉）. Consider the case of adapters [Gamma95], also known as views. An adapter is an object that delegates（代表） to a backing object, providing an alternative interface. Because an adapter has no state beyond that of its backing object, there’s no need to create more than one instance of a given adapter to a given object.

**适配器不需要创建多个**

For example, the keySet method of the Map interface returns a Set view of the Map object, consisting of all the keys in the map. Naively, it would seem that every call to keySet would have to create a new Set instance, but every call to keySet on a given Map object may return the same Set instance. Although the returned Set instance is typically mutable, all of the returned objects are functionally identical: when one of the returned objects changes, so do all the others, because they’re all backed by the same Map instance. While it is largely harmless to create multiple instances of the keySet view object, it is unnecessary and has no benefits.

**Map返回的keySet是共享的。**

Another way to create unnecessary objects is autoboxing, which allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. **Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types.** 

There are subtle semantic distinctions（微妙的语义区别） and not-so-subtle performance differences (Item 61). Consider the following method, which calculates the sum of all the positive int values. To do this, the program has to use long arithmetic because an int is not big enough to hold the sum of all the positive int values:

```
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

This program gets the right answer, but it is much slower than it should be,due to a one-character typographical（排印） error. The variable sum is declared as a Long instead of a long, which means that the program constructs about 231 unnecessary Long instances (roughly one for each time the long i is added to the Long sum). Changing the declaration of sum from Long to long reduces the runtime from 6.3 seconds to 0.59 seconds on my machine. The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**

**基本类型 > (优先于) 装箱类型**

This item should not be misconstrued（曲解） to imply that object creation is expensive and should be avoided. On the contrary, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. Creating additional objects to enhance the clarity,simplicity, or power of a program is generally a good thing.

**创建对象的开销并不大**

Conversely, avoiding object creation by maintaining your own object pool is a bad idea unless the objects in the pool are extremely heavyweight. 

**只有在对象本身非常heavyweight的时候才需要池化。**

The classic example of an object that does justify an object pool is a database connection.The cost of establishing the connection is sufficiently high that it makes sense to reuse these objects. Generally speaking, however, maintaining your own object pools clutters your code, increases memory footprint（脚印）, and harms performance.Modern JVM implementations have highly optimized garbage collectors that easily outperform（跑赢） such object pools on lightweight objects.

**数据库建立连接的开销足够大**

The counterpoint （对立）to this item is Item 50 on defensive copying. The present item says, “Don’t create a new object when you should reuse an existing one,”while Item 50 says, “Don’t reuse an existing object when you should create a new one.” Note that the penalty （惩罚）for reusing an object when defensive copying is called for is far greater than the penalty （惩罚）for needlessly creating a duplicate object. Failing to make defensive copies where required can lead to insidious bugs and security holes; creating objects unnecessarily merely affects style and performance.

**第50条defensive copying危害更大**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）](../Chapter-2/Chapter-2-Item-5-Prefer-dependency-injection-to-hardwiring-resources.md)**
- **Next Item（下一条目）：[Item 7: Eliminate obsolete object references（排除过时的对象引用）](../Chapter-2/Chapter-2-Item-7-Eliminate-obsolete-object-references.md)**
