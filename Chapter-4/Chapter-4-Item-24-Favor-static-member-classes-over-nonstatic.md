## Chapter 4. Classes and Interfaces（类和接口）

### Item 24: Favor static member classes over nonstatic（静态成员类优于非静态成员类）

A nested class is a class defined within another class. A nested class should exist only to serve its enclosing class. If a nested class would be useful in some other context, then it should be a top-level class. There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes. All but the first kind are known as inner classes. This item tells you when to use which kind of nested class and why.

**嵌套类是在另一个类中定义的类。嵌套类应该只为外部类服务。**

**如果嵌套类在其他环境中有用，那么它应该是顶级类。**

**有四种嵌套类：静态成员类、非静态成员类、匿名类和局部类。除了第一种，所有的类都被称为内部类。**

**A static member class** is the simplest kind of nested class. It is best thought of as an ordinary class that happens to be declared inside another class and has access to all of the enclosing class’s members, even those declared private. A static member class is a static member of its enclosing class and obeys the same accessibility rules as other static members. If it is declared private, it is accessible only within the enclosing class, and so forth.

静态成员类是最简单的嵌套类。最好把它看做是一个普通的类，只是碰巧在另一个类中声明而已，并且可以访问外部类的所有成员，甚至那些声明为 private 的成员。静态成员类是其外部类的静态成员，并且遵守与其他静态成员相同的可访问性规则。如果声明为私有，则只能在外部类中访问，等等。

One common use of a static member class is as a public helper class, useful only in conjunction（n. 结合；[语] 连接词；同时发生） with its outer class. 

For example, consider an enum describing the operations supported by a calculator (Item 34). **The Operation enum should be a public static member class of the Calculator class**. Clients of Calculator could then refer to operations using names like Calculator.Operation.PLUS and Calculator.Operation.MINUS.

**静态成员类经常用来当做辅助类。比如，operation枚举类是calculator的静态成员类。**

Syntactically（语法）, the only difference between static and nonstatic member classes is that static member classes have the modifier static in their declarations. Despite the syntactic similarity, these two kinds of nested classes are very different. Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class. **Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified this construct [JLS, 15.8.4]**. If an instance of a nested class can exist in isolation from an instance of its enclosing class, then the nested class must be a static member class: it is impossible to create an instance of a nonstatic member class without an enclosing instance.

**非静态成员类可以通过加了qualified 的this来获得外部类的引用。**

**非静态成员类new的时候需要外部类的实例。**

下面我亲自举了个例子：

```
public class Outer {

    private String name;

    public Outer(String name) {
        this.name = name;
    }

    private void fuck() {
        System.out.println("fuck" + name);
    }

    private void fuck2() {

    }

    class Inner {

        public Inner() {}

        private void fuck22() {
            Outer.this.name = "3333";
            Outer.this.fuck();
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer("111");
        Outer.Inner inner = outer.new Inner();
        inner.fuck22();// print fuck3333
    }
}
```

The association between a nonstatic member class instance and its enclosing instance is established when the member class instance is created and cannot be modified thereafter. Normally, the association is established automatically by invoking a nonstatic member class constructor from within an instance method of the enclosing class. It is possible, though rare, to establish the association manually using the expression enclosingInstance.new MemberClass(args). As you would expect, the association takes up space in the nonstatic member class instance and adds time to its construction.

**非静态成员类实例与外部实例之间的关联是在创建成员类实例时建立的，之后无法修改**。

One common use of a nonstatic member class is to define an Adapter [Gamma95] that allows an instance of the outer class to be viewed as an instance of some unrelated class. For example, implementations of the Map interface typically use nonstatic member classes to implement their collection views, which are returned by Map’s keySet, entrySet, and values methods. Similarly, implementations of the collection interfaces, such as Set and List, typically use nonstatic member classes to implement their iterators:

**例如，Map 接口的实现通常使用非静态成员类来实现它们的集合视图，这些视图由 Map 的 keySet、entrySet 和 values 方法返回。类似地，集合接口的实现，例如 Set 和 List，通常使用非静态成员类来实现它们的迭代器：**

```
// Typical use of a nonstatic member class
public class MySet<E> extends AbstractSet<E> {
    ... // Bulk of the class omitted
    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }
    private class MyIterator implements Iterator<E> {
      ...
    }
}
```

**If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration,** making it a static rather than a nonstatic member class. If you omit this modifier, each instance will have a hidden extraneous reference to its enclosing instance. As previously mentioned, storing this reference takes time and space. More seriously, it can result in the enclosing instance being retained when it would otherwise be eligible for garbage collection (Item 7). The resulting memory leak can be catastrophic. It is often difficult to detect because the reference is invisible.

**如果声明的成员类不需要访问外部的实例，那么应始终在声明中添加 static 修饰符，使其成为静态的而不是非静态的成员类。** 如果省略这个修饰符，每个实例都有一个隐藏的对其外部实例的额外引用。如前所述，存储此引用需要时间和空间。更严重的是，它可能会在满足进行垃圾收集条件时仍保留外部类的实例（[Item-7](../Chapter-2/Chapter-2-Item-7-Eliminate-obsolete-object-references.md)）。由于引用是不可见的，因此通常很难检测到。

A common use of private static member classes is to represent components of the object represented by their enclosing class. 

For example, consider a Map instance, which associates keys with values. Many Map implementations have an internal Entry object for each key-value pair in the map. 

While each entry is associated with a map, the methods on an entry (getKey, getValue, and setValue) do not need access to the map. Therefore, it would be wasteful to use a nonstatic member class to represent entries: a private static member class is best. If you accidentally omit the static modifier in the entry declaration, the map will still work, but each entry will contain a superfluous reference to the map, which wastes space and time.

**私有静态成员类的一个常见用法是做外部类的组件。比如Map的Entry。**

It is doubly important to choose correctly between a static and a nonstatic member class if the class in question is a public or protected member of an exported class. In this case, the member class is an exported API element and cannot be changed from a nonstatic to a static member class in a subsequent release without violating backward compatibility.

**非静态成员类变成静态成员类会影响API。**

As you would expect, an anonymous class has no name. It is not a member of its enclosing class. Rather than being declared along with other members, it is simultaneously declared and instantiated at the point of use. Anonymous classes are permitted at any point in the code where an expression is legal. Anonymous classes have enclosing instances if and only if they occur in a nonstatic context. But even if they occur in a static context, they cannot have any static members other than constant variables, which are final primitive or string fields initialized to constant expressions [JLS, 4.12.4].

**如你所料，匿名类没有名称。它不是外部类的成员。它不是与其他成员一起声明的，而是在使用时同时声明和实例化。匿名类在代码中任何一个表达式合法的地方都是被允许的。当且仅当它们发生在非静态环境中时，匿名类才具有外部类实例。但是，即使它们发生在静态环境中，它们也不能有除常量变量以外的任何静态成员（比如，public final static ArrayList aaa就不行）。**

There are many limitations on the applicability of anonymous classes. You can’t instantiate them except at the point they’re declared. You can’t perform instanceof tests or do anything else that requires you to name the class. 

You can’t declare an anonymous class to implement multiple interfaces or to extend a class and implement an interface at the same time. Clients of an anonymous class can’t invoke any members except those it inherits from its supertype. Because anonymous classes occur in the midst of expressions, they must be kept short—about ten lines or fewer—or readability will suffer.

**匿名类的适用性有很多限制。你不能实例化它们，除非在声明它们的时候。你不能执行 instanceof 测试，也不能执行任何其他需要命名类的操作。你不能声明一个匿名类来实现多个接口或扩展一个类并同时实现一个接口。匿名类的client只能调用从超类型继承的成员。因为匿名类发生在表达式的中间，所以它们必须保持简短——大约 10 行或几行，否则可读性会受到影响。**

Before lambdas were added to Java (Chapter 6), anonymous classes were the preferred means of creating small function objects and process objects on the fly（动态的）, but lambdas are now preferred (Item 42). Another common use of anonymous classes is in the implementation of static factory methods (see intArrayAsList in Item 20).

**在 lambda 表达式被添加到 Java（Chapter 6）之前，匿名类是动态创建小型函数对象和进程对象的首选方法，但 lambda 表达式现在是首选方法（[Item-42](../Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)）。匿名类的另一个常见用法是实现静态工厂方法（参见 [Item-20](../Chapter-4/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md) 中的 intArrayAsList 类）确实骚操作。**

Local classes are the least frequently used of the four kinds of nested classes. A local class can be declared practically anywhere a local variable can be declared and obeys the same scoping rules. Local classes have attributes in common with each of the other kinds of nested classes. Like member classes, they have names and can be used repeatedly. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members. And like anonymous classes, they should be kept short so as not to harm readability.

**局部类是四种嵌套类中最不常用的。**

**局部类几乎可以在任何能够声明局部变量的地方使用。**

**与成员类一样，它们有名称，可以重复使用。**

**与匿名类一样，它们只有在非静态环境中定义的情况下才具有外部类实例，而且它们不能包含静态成员。**

**和匿名类一样，它们应该保持简短，以免损害可读性。**

To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

简单回顾一下，有四种不同类型的嵌套类，每一种都有自己的用途。如果嵌套的类需要在单个方法之外可见，或者太长，不适合放入方法中，则使用成员类。如果成员类的每个实例都需要引用其外部类实例，则使其非静态；否则，让它保持静态。假设嵌套类属于方法内部，如果你只需要从一个位置创建实例，并且存在一个能够描述类的现有类型，那么将其设置为匿名类；否则，将其设置为局部类。

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-4/Chapter-4-Introduction.md)**
- **Previous Item（上一条目）：[Item 23: Prefer class hierarchies to tagged classes（类层次结构优于带标签的类）](../Chapter-4/Chapter-4-Item-23-Prefer-class-hierarchies-to-tagged-classes.md)**
- **Next Item（下一条目）：[Item 25: Limit source files to a single top level class（源文件仅限有单个顶层类）](../Chapter-4/Chapter-4-Item-25-Limit-source-files-to-a-single-top-level-class.md)**
