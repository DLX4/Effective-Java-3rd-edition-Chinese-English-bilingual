## Chapter 4. Classes and Interfaces（类和接口）

### Item 17: Minimize mutability（减少可变性）

An immutable class is simply a class whose instances cannot be modified. All of the information contained in each instance is fixed for the lifetime of the object,so no changes can ever be observed. The Java platform libraries contain many immutable classes, including String, the boxed primitive classes, and BigInteger and BigDecimal. There are many good reasons for this:Immutable classes are easier to design, implement, and use than mutable classes.They are less prone to error and are more secure.

**不可变类就是一个实例不能被修改的类。每个实例中包含的所有信息在对象的生命周期内都是固定的，因此永远不会观察到任何更改。Java 库包含许多不可变的类，包括 String、基本类型的包装类、BigInteger 和 BigDecimal。这么做有很好的理由：不可变类比可变类更容易设计、实现和使用。它们不太容易出错，而且更安全。**

To make a class immutable, follow these five rules:

要使类不可变，请遵循以下 5 条规则：

1. **Don’t provide methods that modify the object’s state** (known as mutators).

2. **Ensure that the class can’t be extended.** This prevents careless or malicious（恶意的） subclasses from compromising（妥协） the immutable behavior of the class by behaving as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.

3. **Make all fields final.** This clearly expresses your intent in a manner that is enforced by the system. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the memory model [JLS, 17.5;Goetz06, 16].

4. **Make all fields private.** This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly.While it is technically permissible （允许的）for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes（排除） changing the internal representation in a later release (Items 15 and 16).

5. **Ensure exclusive access to any mutable components.** If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client provided object reference or return the field from an accessor. Make defensive copies (Item 50) in constructors, accessors, and readObject methods (Item 88).

   **创建防御性副本**

Many of the example classes in previous items are immutable. One such class is PhoneNumber in Item 11, which has accessors for each attribute but no corresponding mutators. Here is a slightly more complex example:

**一个复杂的例子**

```
// Immutable complex number class
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart() { return re; }
    public double imaginaryPart() { return im; }
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // See page 47 to find out why we use compare instead of ==
        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

This class represents a complex number (a number with both real and imaginary parts). In addition to the standard Object methods, it provides accessors for the real and imaginary parts and provides the four basic arithmetic operations: addition, subtraction, multiplication, and division. 

Notice how the arithmetic operations create and return a new Complex instance rather than modifying this instance. 

**This pattern is known as the functional approach because methods return the result of applying a function to their operand, without modifying it**. 

**函数式**

Contrast it to the procedural or imperative approach in which methods apply a procedure to their operand, causing its state to change.

Note that the method names are prepositions (such as plus) rather than verbs (such as add). This emphasizes the fact that methods don’t change the values of the objects. The BigInteger and BigDecimal classes did not obey this naming convention, and it led to many usage errors.

**这里例子的方法名还有讲究，方法名是介词而不是动词，强调了这个对象的值不会改变。**

The functional approach（n. 方法；途径；接近） may appear unnatural if you’re not familiar（adj. 熟悉的；常见的；亲近的） with it,but it enables immutability, which has many advantages. 

**Immutable objects are simple.** An immutable object can be in exactly one state, the state in which it was created. If you make sure that all constructors establish class invariants, then it is guaranteed that these invariants will remain true for all time, with no further effort on your part or on the part of the programmer who uses the class. Mutable objects, on the other hand, can have arbitrarily complex state spaces. If the documentation does not provide a precise description of the state transitions performed by mutator methods, it can be difficult or impossible to use a mutable class reliably.

**可变对象和不可变对象。**

**Immutable objects are inherently thread-safe; they require no synchronization.** They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety.Since no thread can ever observe any effect of another thread on an immutable object, **immutable objects can be shared freely.** 

**不可变对象是实现线程安全的最简单方法。**

Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to provide public static final constants for commonly used values. For example, the Complex class might provide these constants:

**不可变对象鼓励复用。**

```
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

This approach can be taken one step further. An immutable class can provide static factories (Item 1) that cache frequently requested instances to avoid creating new instances when existing ones would do. All the boxed primitive classes and BigInteger do this. 

**缓存频繁使用的对象，所有包装类和BigInteger 就是这么干的。**

Using such static factories causes clients to share instances instead of creating new ones, reducing memory footprint and garbage collection costs. Opting for static factories in place of public constructors when designing a new class gives you the flexibility to add caching later, without modifying clients.

**使用静态工厂方便添加缓存。**

A consequence of the fact that immutable objects can be shared freely is that you never have to make defensive copies of them (Item 50). In fact, you never have to make any copies at all because the copies would be forever equivalent to the originals. Therefore, you need not and should not provide a clone method or copy constructor (Item 13) on an immutable class. This was not well understood in the early days of the Java platform, so the String class does have a copy constructor, but it should rarely, if ever, be used (Item 6).

**不可变对象不需要防御性的拷贝。**

**这在 Java 平台的早期并没有得到很好的理解，因此 String 类确实有一个复制构造函数，最好不要用他。**

**Not only can you share immutable objects, but they can share their internals.** For example, the BigInteger class uses a sign-magnitude representation internally. 

The sign is represented by an int, and the magnitude is represented by an int array. 

**符号由一个int表示，数值大小由一个int数组表示。**

The negate method produces a new BigInteger of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created BigInteger points to the same internal array as the original.

**你不仅可以共享不可变对象，而且可以共享它们的内部实现。 例如，BigInteger 类在内部使用符号+数值大小来表示。符号由 int 表示，数值大小由 int 数组表示。negate 方法产生一个新的 BigInteger，大小相同，符号相反。即使数组是可变的，也不需要复制；新创建的 BigInteger 指向与原始数组相同的内部数组。**

**Immutable objects make great building blocks for other objects,** whether mutable or immutable. It’s much easier to maintain the invariants of a complex object if you know that its component objects will not change underneath it. 

A special case of this principle is that immutable objects make great map keys and set elements: you don’t have to worry about their values changing once they’re in the map or set, which would destroy the map or set’s invariants.

**不可变对象非常适合作为其他对象的构成部分。比如map的key和集合的元素。**

**Immutable objects provide failure atomicity for free** (Item 76). Their state never changes, so there is no possibility of a temporary inconsistency.

**不可变对象自带提供故障原子性（[Item-76](../Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)）。他们的状态从未改变，所以不可能出现暂时的不一致。**

**The major disadvantage of immutable classes is that they require a separate object for each distinct value.** Creating these objects can be costly,especially if they are large. For example, suppose that you have a million-bit BigInteger and you want to change its low-order bit:

**不可变类的主要缺点是每个不同的值都需要一个单独的对象。** 

**创建这些对象的成本可能很高，尤其是如果对象很大的话。例如，假设你有一个百万位的 BigInteger，你想改变它的低阶位：**

```
BigInteger moby = ...;
moby = moby.flipBit(0);
```

The flipBit method creates a new BigInteger instance, also a million bits long, that differs from the original in only one bit. The operation requires time and space proportional to the size of the BigInteger. Contrast this to java.util.BitSet. Like BigInteger, BitSet represents an arbitrarily long sequence of bits, but unlike BigInteger, BitSet is mutable. The BitSet class provides a method that allows you to change the state of a single bit of a million-bit instance in constant time:

flipBit 方法创建了一个新的 BigInteger 实例，也有百万位长，只在一个比特上与原始的不同。该操作需要与 BigInteger 的大小成比例的时间和空间。与 `java.util.BitSet` 形成对比。与 BigInteger 一样，BitSet 表示任意长的位序列，但与 BigInteger 不同，BitSet 是可变的。BitSet 类提供了一种方法，可以让你在固定的时间内改变百万位实例的单个位的状态：

```
BitSet moby = ...;
moby.flip(0);
```

The performance problem is magnified(放大的) if you perform a multistep operation that generates a new object at every step, eventually discarding all objects except the final result. There are two approaches to coping（应对） with this problem. The first is to guess which multistep operations will be commonly required and to provide them as primitives. If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step. Internally,the immutable class can be arbitrarily clever. For example, BigInteger has a package-private mutable “companion class” that it uses to speed up multistep operations such as modular exponentiation（幂运算（取模））. It is much harder to use the mutable companion class than to use BigInteger, for all of the reasons outlined earlier. Luckily, you don’t have to use it: the implementors of BigInteger did the hard work for you.

**对多步操作 比如幂运算（取模）的优化：package-private mutable companion class**

The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a public mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder (and its obsolete predecessor, StringBuffer).

**如果你能够准确地预测客户端希望在不可变类上执行哪些复杂操作，那么包私有可变伴随类方法就可以很好地工作。如果不是，那么你最好的选择就是提供一个公共可变伴随类。这种方法在 Java 库中的主要示例是 String 类，它的可变伴随类是 StringBuilder（及其过时的前身 StringBuffer)。**

Now that you know how to make an immutable class and you understand the pros and cons（优缺点） of immutability, let’s discuss a few design alternatives. Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors (Item 1). To make this concrete, here’s how Complex would look if you took this approach:

**为了保证不变性，类不允许自己被子类化。除了把类变成final，但是还有另外一个更灵活的选择。你可以将其所有构造方法变为私有或包-私有，并在公共构造方法的位置添加公共静态工厂（[Item-1](../Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）。**

```
// Immutable class with static factories instead of constructors
public class Complex {
    private final double re;
    private final double im;
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ... // Remainder unchanged
}
```

This approach is often the best alternative. It is the most flexible because it allows the use of multiple package-private implementation classes. To its clients that reside outside its package, the immutable class is effectively final because it is impossible to extend a class that comes from another package and that lacks a public or protected constructor. Besides allowing the flexibility of multiple implementation classes, this approach makes it possible to tune the performance of the class in subsequent releases by improving the object-caching capabilities of the static factories.

**上面这样做的好处：allowing the flexibility of multiple implementation classes便于灵活扩展；可以在static factories里面实现对象缓存。**

It was not widely understood that immutable classes had to be effectively final when BigInteger and BigDecimal were written, so all of their methods may be overridden. Unfortunately, this could not be corrected after the fact while preserving backward compatibility. If you write a class whose security depends on the immutability of a BigInteger or BigDecimal argument from an untrusted client, you must check to see that the argument is a “real” BigInteger or BigDecimal, rather than an instance of an untrusted subclass. If it is the latter, you must defensively copy it under the assumption that it might be mutable (Item 50):

**BigInteger and BigDecimal居然不是final的。因为历史原因。并且因为向后兼容一直没有纠正。**

```
public static BigInteger safeInstance(BigInteger val) {
return val.getClass() == BigInteger.class ?
val : new BigInteger(val.toByteArray());
}
```

The list of rules for immutable classes at the beginning of this item says that no methods may modify the object and that all its fields must be final. In fact these rules are a bit stronger than necessary and can be relaxed to improve performance. 

In truth, no method may produce an externally visible change in the object’s state. 

**不可变对象只需要保证对外部可见的状态不变更，内部某些状态可以用来实现缓存。**

However, some immutable classes have one or more nonfinal fields in which they cache the results of expensive computations the first time they are needed. If the same value is requested again, the cached value is returned, saving the cost of recalculation. This trick works precisely because the object is immutable, which guarantees that the computation would yield the same result if it were repeated.

For example, PhoneNumber’s hashCode method (Item 11, page 53) computes the hash code the first time it’s invoked and caches it in case it’s invoked again. This technique, an example of lazy initialization (Item 83), is also used by String.

One caveat should be added concerning serializability. If you choose to have your immutable class implement Serializable and it contains one or more fields that refer to mutable objects, you must provide an explicit readObject or readResolve method, or use the ObjectOutputStream.writeUnshared and ObjectInputStream.readUnshared methods, even if the default serialized form is acceptable. Otherwise an attacker could create a mutable instance of your class. This topic is covered in detail in Item 88.

**关于序列化的情况，先略过。**

To summarize, resist the urge to write a setter for every getter. **Classes should be immutable unless there’s a very good reason to make them mutable.** Immutable classes provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances. You should always make small value objects, such as PhoneNumber and Complex, immutable. (There are several classes in the Java platform libraries, such as java.util.Date and java.awt.Point, that should have been immutable but aren’t.) 

**小对象最好搞成不可变的。**

You should seriously consider making larger value objects, such as String and BigInteger, immutable as well. You should provide a public mutable companion class for your immutable class only once you’ve confirmed that it’s necessary to achieve satisfactory performance (Item 67).
**大对象也搞成不可变的，然后可以使用一个public的可变伴随类来优化性能。**

There are some classes for which immutability is impractical. **If a class cannot be made immutable, limit its mutability as much as possible.** Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, make every field final unless there is a compelling reason to make it nonfinal. Combining the advice of this item with that of Item 15, your natural inclination should be to **declare every field private final unless there’s a good reason to do otherwise.**

**Constructors should create fully initialized objects with all of their invariants established.** Don’t provide a public initialization method separate from the constructor or static factory unless there is a compelling reason to do so. Similarly, don’t provide a “reinitialize” method that enables an object to be reused as if it had been constructed with a different initial state. Such methods generally provide little if any performance benefit at the expense of increased complexity.

**构造函数应该创建完全初始化的对象，并建立所有的不变量。 不要提供public的初始化方法。**

The CountDownLatch class exemplifies these principles. It is mutable, but its state space is kept intentionally（故意的） small. You create an instance, use it once, and it’s done: once the countdown latch’s count has reached zero, you may not reuse it.

A final note should be added concerning the Complex class in this item. This example was meant only to illustrate immutability. It is not an industrial-strength complex number implementation. It uses the standard formulas for complex multiplication and division, which are not correctly rounded and provide poor semantics for complex NaNs and infinities [Kahan91, Smith62, Thomas94].

**例子的Complex 类不是一个工业强度的复数实现。**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-4/Chapter-4-Introduction.md)**
- **Previous Item（上一条目）：[Item 16: In public classes use accessor methods not public fields（在公共类中，使用访问器方法，而不是公共字段）](../Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)**
- **Next Item（下一条目）：[Item 18: Favor composition over inheritance（优先选择复合而不是继承）](../Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md)**
