## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 14: Consider implementing Comparable（考虑实现 Comparable 接口）

Unlike the other methods discussed in this chapter, the compareTo method is not declared in Object. Rather, it is the sole method in the Comparable interface. It is similar in character to Object’s equals method, except that it permits order comparisons in addition to simple equality comparisons, and it is generic. 

**除了比较相等之外还能比较大小。**

By implementing Comparable, a class indicates that its instances have a natural ordering. Sorting an array of objects that implement Comparable is as simple as this:

```
Arrays.sort(a);
```

It is similarly easy to search, compute extreme values, and maintain automatically sorted collections of Comparable objects. For example, the following program, which relies on the fact that String implements Comparable, prints an alphabetized list of its command-line arguments with duplicates eliminated:

**因为String实现了comparable接口，下面的例子是把命令行参数按照字母顺序打印出来，并去重。**

```
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

By implementing Comparable, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. 

**很多算法、集合的实现类是基于Comparable接口的。**

You gain a tremendous amount of power for a small amount of effort. Virtually（实际上） all of the value classes in the Java platform libraries, as well as all enum types (Item 34), implement Comparable. If you are writing a value class with an obvious natural ordering, such as alphabetical order, numerical order, or chronological order, you should implement the Comparable interface:

**java平台库中的所有的value classes以及枚举类型都实现了Comparable接口。**

```
public interface Comparable<T> {
    int compareTo(T t);
}
```

The general contract of the compareTo method is similar to that of equals:

**compareTo 方法的契约类似于 equals 方法**

Compares this object with the specified object for order. Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws ClassCastException if the specified object’s type prevents it from being compared to this object.

**小于：返回负整数。大于：返回正整数。相等：返回0。无法比较：抛出异常。**

In the following description, the notation sgn(expression) designates the mathematical signum function, which is defined to return -1, 0, or 1,according to whether the value of expression is negative, zero, or positive.

在下面的描述中，`sgn(expression)` 表示数学中的符号函数，它被定义为：根据传入表达式的值是负数、零或正数，对应返回 -1、0 或 1。

- The implementor must ensure that sgn(x.compareTo(y)) == -sgn(y. compareTo(x)) for all x and y. (This implies that x.compareTo(y) must throw an exception if and only if y.compareTo(x) throws an exception.)

- The implementor must also ensure that the relation is transitive: (x.compareTo(y) > 0 && y.compareTo(z) > 0) implies x.compareTo(z) > 0.

- Finally, the implementor must ensure that x.compareTo(y) == 0 implies that sgn(x.compareTo(z)) == sgn(y.compareTo(z)),for all z.

- It is strongly recommended, but not required, that (x.compareTo(y)== 0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”

Don’t be put off by the mathematical nature of this contract. Like the equals contract (Item 10), this contract isn’t as complicated as it looks. Unlike the equals method, which imposes a global equivalence relation on all objects,compareTo doesn’t have to work across objects of different types:

**equals方法是针对所有的对象，而compareTo只关注同类的对象之间比较。**

 when confronted with objects of different types, compareTo is permitted to throw ClassCastException. Usually, that is exactly what it does. The contract does permit intertype comparisons, which are typically defined in an interface implemented by the objects being compared.

Just as a class that violates the hashCode contract can break other classes that depend on hashing, a class that violates the compareTo contract can break other classes that depend on comparison. Classes that depend on comparison include the sorted collections TreeSet and TreeMap and the utility classes Collections and Arrays, which contain searching and sorting algorithms.

**有序集合 TreeSet 和 TreeMap，以及实用工具类 Collections 和 Arrays的搜索和排序算法都是基于compareTo的约定的。**

Let’s go over the provisions of the compareTo contract. The first provision says that if you reverse the direction of a comparison between two object references, the expected thing happens: if the first object is less than the second,then the second must be greater than the first; if the first object is equal to the second, then the second must be equal to the first; and if the first object is greater than the second, then the second must be less than the first. The second provision says that if one object is greater than a second and the second is greater than a third, then the first must be greater than the third. The final provision says that all objects that compare as equal must yield the same results when compared to any other object.

One consequence of these three provisions is that the equality test imposed by a compareTo method must obey the same restrictions imposed by the equals con-tract: reflexivity, symmetry, and transitivity. Therefore, the same caveat applies: there is no way to extend an instantiable class with a new value component while preserving the compareTo contract, unless you are willing to forgo the benefits of object-oriented abstraction (Item 10). The same workaround applies, too. If you want to add a value component to a class that implements Comparable, don’t extend it; write an unrelated class containing an instance of the first class. Then provide a “view” method that returns the contained instance. This frees you to implement whatever compareTo method you like on the containing class, while allowing its client to view an instance of the containing class as an instance of the contained class when needed.

The final paragraph of the compareTo contract, which is a strong suggestion rather than a true requirement, simply states that the equality test imposed by the compareTo method should generally return the same results as the equals method. If this provision is obeyed, the ordering imposed by the compareTo method is said to be consistent with equals. If it’s violated, the ordering is said to be inconsistent with equals. A class whose compareTo method imposes an order that is inconsistent with equals will still work, but sorted collections containing elements of the class may not obey the general contract of the appropriate collection interfaces (Collection, Set, or Map). This is because the general contracts for these interfaces are defined in terms of the equals method, but sorted collections use the equality test imposed by compareTo in place of equals. It is not a catastrophe if this happens, but it’s something to be aware of.

**自反，对称，传递，兼容equals**

For example, consider the BigDecimal class, whose compareTo method is inconsistent with equals. If you create an empty HashSet instance and then add new BigDecimal("1.0") and new BigDecimal("1.00"),the set will contain two elements because the two BigDecimal instances added to the set are unequal when compared using the equals method. If,however, you perform the same procedure using a TreeSet instead of a HashSet, the set will contain only one element because the two BigDecimal instances are equal when compared using the compareTo method. (See the BigDecimal documentation for details.)

**例如，考虑 BigDecimal 类，它的 compareTo 方法与 equals 不一致。如果你创建一个空的 HashSet 实例，然后添加 `new BigDecimal("1.0")` 和 `new BigDecimal("1.00")`，那么该 HashSet 将包含两个元素，因为添加到该集合的两个 BigDecimal 实例在使用 equals 方法进行比较时结果是不相等的。但是，如果你使用 TreeSet 而不是 HashSet 执行相同的过程，那么该集合将只包含一个元素，因为使用 compareTo 方法比较两个 BigDecimal 实例时结果是相等的。（有关详细信息，请参阅 BigDecimal 文档。）**

Writing a compareTo method is similar to writing an equals method, but there are a few key differences. Because the Comparable interface is parameterized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointerException, and it will, as soon as the method attempts to access its members.

In a compareTo method, fields are compared for order rather than equality.To compare object reference fields, invoke the compareTo method recursively. If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead. You can write your own comparator or use an existing one, as in this compareTo method for CaseInsensitiveString in Item 10:

**CaseInsensitiveString 的 compareTo 方法：**

```
// Single-field Comparable with object reference field
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    } ... // Remainder omitted
}
```

Note that CaseInsensitiveString implements `Comparable<CaseInsensitiveString>`. This means that a CaseInsensitiveString reference can be compared only to another CaseInsensitiveString reference. This is the normal pattern to follow when declaring a class to implement Comparable.

Prior editions of this book recommended that compareTo methods compare integral primitive fields using the relational operators < and >, and floating point primitive fields using the static methods Double.compare and Float.compare. In Java 7, static compare methods were added to all of Java’s boxed primitive classes. **Use of the relational operators < and > in compareTo methods is verbose and error-prone and no longer recommended.**

**在 Java 7 中，静态比较方法被添加到所有 Java 的包装类中。在 compareTo 方法中使用关系运算符 < 和 > 冗长且容易出错，因此不再推荐使用。**

If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down. If a comparison results in anything other than zero (which represents equality),you’re done; just return the result. If the most significant field is equal, compare the next-most-significant field, and so on, until you find an unequal field or compare the least significant field. Here is a compareTo method for the PhoneNumber class in Item 11 demonstrating this technique:

**PhoneNumber的例子：**

```
// Multiple-field Comparable with primitive fields
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

In Java 8, the Comparator interface was outfitted（配备） with a set of comparator construction methods, which enable fluent （流畅）construction of comparators. These comparators can then be used to implement a compareTo method, as required by the Comparable interface. Many programmers prefer the conciseness（简明） of this approach, though it does come at a modest performance cost: sorting arrays of PhoneNumber instances is about 10% slower on my machine. When using this approach, consider using Java’s static import facility so you can refer to static comparator construction methods by their simple names for clarity and brevity. Here’s how the compareTo method for PhoneNumber looks using this approach:

**在 Java 8 中，java.util.Comparator 接口里面提供了comparingInt这样的静态方法。**

```
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
    .thenComparingInt(pn -> pn.prefix)
    .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

**译注 1：示例代码默认使用了静态导入：`import static java.util.Comparator.comparingInt;`**

**译注 2：comparingInt 及 thenComparingInt 的文档描述**

```
static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor)

Accepts a function that extracts an int sort key from a type T, and returns a Comparator<T> that compares by that sort key.
The returned comparator is serializable if the specified function is also serializable.

接受从类型 T 中提取 int 排序 key 的函数，并返回与该排序 key 进行比较的 Comparator<T>。
如果指定的函数是可序列化的，则返回的比较器也是可序列化的。

Type Parameters:
    T - the type of element to be compared
Parameters:
    keyExtractor - the function used to extract the integer sort key
Returns:
    a comparator that compares by an extracted key
Throws:
    NullPointerException - if the argument is null
Since:
    1.8
```

```
default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor)

Returns a lexicographic-order comparator with a function that extracts a int sort key.
Implementation Requirements:
This default implementation behaves as if thenComparing(comparingInt(keyExtractor)).

返回具有提取 int 排序 key 的函数的字典顺序比较器。
实现要求：
此默认实现的行为类似于 thenComparing(comparingInt(keyExtractor))。

Parameters:
    keyExtractor - the function used to extract the integer sort key
Returns:
    a lexicographic-order comparator composed of this and then the int sort key
Throws:
    NullPointerException - if the argument is null.
Since:
    1.8
```

This implementation builds a comparator at class initialization time, using two comparator construction methods. The first is comparingInt. It is a static method that takes a key extractor function that maps an object reference to a key of type int and returns a comparator that orders instances according to that key.In the previous example, comparingInt takes a lambda () that extracts the area code from a PhoneNumber and returns a `Comparator<PhoneNumber>` that orders phone numbers according to their area codes. Note that the lambda explicitly specifies the type of its input parameter (PhoneNumber pn). It turns out that in this situation, Java’s type inference isn’t powerful enough to figure the type out for itself, so we’re forced to help it in order to make the program compile.

If two phone numbers have the same area code, we need to further refine the comparison, and that’s exactly what the second comparator construction method,thenComparingInt, does. It is an instance method on Comparator that takes an int key extractor function, and returns a comparator that first applies the original comparator and then uses the extracted key to break ties. You can stack up as many calls to thenComparingInt as you like, resulting in a lexicographic ordering. In the example above, we stack up two calls to thenComparingInt, resulting in an ordering whose secondary key is the prefix and whose tertiary key is the line number. Note that we did not have to specify the parameter type of the key extractor function passed to either of the calls to thenComparingInt: Java’s type inference was smart enough to figure this one out for itself.

The Comparator class has a full complement of construction methods.There are analogues to comparingInt and thenComparingInt for the primitive types long and double. The int versions can also be used for narrower integral types, such as short, as in our PhoneNumber example. The double versions can also be used for float. This provides coverage of all of Java’s numerical primitive types.

There are also comparator construction methods for object reference types.The static method, named comparing, has two overloadings. One takes a key extractor and uses the keys’ natural order. The second takes both a key extractor and a comparator to be used on the extracted keys. There are three overloadings of the instance method, which is named thenComparing. One overloading takes only a comparator and uses it to provide a secondary order. A second overloading takes only a key extractor and uses the key’s natural order as a secondary order. The final overloading takes both a key extractor and a comparator to be used on the extracted keys.

Occasionally you may see compareTo or compare methods that rely on the fact that the difference between two values is negative if the first value is less than the second, zero if the two values are equal, and positive if the first value is greater. Here is an example:

```
// BROKEN difference-based comparator - violates transitivity!
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

Do not use this technique. It is fraught with danger from integer overflow and IEEE 754 floating point arithmetic artifacts [JLS 15.20.1, 15.21.1]. Furthermore,the resulting methods are unlikely to be significantly faster than those written using the techniques described in this item. Use either a static compare method:

**不要使用这种技术。它充满了来自整数溢出和 IEEE 754 浮点运算构件的危险 [JLS 15.20.1, 15.21.1]。此外，生成的方法不太可能比使用本项目中描述的技术编写的方法快得多。应使用静态比较方法：**

```
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

or a comparator construction method:

**或比较器构造方法：**

```
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder = Comparator
    .comparingInt(o -> o.hashCode());
```

In summary, whenever you implement a value class that has a sensible ordering, you should have the class implement the Comparable interface so that its instances can be easily sorted, searched, and used in comparison-based collections. When comparing field values in the implementations of the compareTo methods, avoid the use of the < and > operators. Instead, use the static compare methods in the boxed primitive classes or the comparator construction methods in the Comparator interface.

**总之，无论何时实现具有排序性质的值类，都应该让类实现 Comparable 接口，这样就可以轻松地对实例进行排序、搜索，并与依赖于此接口的集合实现进行互操作。在 compareTo 方法的实现中比较字段值时，避免使用 < 和 > 操作符，应使用包装类中的静态比较方法或 Comparator 接口中的 comparator 构造方法。**

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Introduction.md)**
- **Previous Item（上一条目）：[Item 13: Override clone judiciously（明智地覆盖 clone 方法）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3/Chapter-3-Item-13-Override-clone-judiciously.md)**
- **Next Item（下一条目）：[Chapter 4 Introduction（章节介绍）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Introduction.md)**
