## Chapter 4. Classes and Interfaces（类和接口）

### Item 20: Prefer interfaces to abstract classes（接口优于抽象类）

**Java has two mechanisms to define a type that permits multiple implementations: interfaces and abstract classes.** 

**java有两种机制用来define type：接口和抽象类。**

Since the introduction of default methods for interfaces in Java 8 [JLS 9.4.3], both mechanisms allow you to provide implementations for some instance methods. 

**java 8已经允许在接口中包含默认方法实现。**

A major difference is that to implement the type defined by an abstract class, a class must be a subclass of the abstract class. Because Java permits only single inheritance, this restriction on abstract classes severely constrains their use as type definitions.

**以抽象类定义的type，一个class要去implement它必须要成为它的子类。**

Any class that defines all the required methods and obeys the general contract is permitted to implement an interface, regardless of where the class resides in the class hierarchy.

**任何class要去implement一个接口只需要定义所有的方法以及遵守所有的契约。**

**Existing classes can easily be retrofitted to implement a new interface.** All you have to do is to add the required methods, if they don’t yet exist, and to add an implements clause to the class declaration. For example, many existing classes were retrofitted to implement the Comparable, Iterable, and Autocloseable interfaces when they were added to the platform. 

**已有的class很容易就能被改造去实现一个新的接口。**

Existing classes cannot, in general, be retrofitted to extend a new abstract class. If you want to have two classes extend the same abstract class, you have to place it high up in the type hierarchy where it is an ancestor of both classes. Unfortunately,this can cause great collateral （附带）damage to the type hierarchy, forcing all descendants of the new abstract class to subclass it, whether or not it is appropriate.

**extend一个新的抽象类就很麻烦，需要强行成为它的儿子。**

**Interfaces are ideal for defining mixins.** Loosely speaking, a mixin is a type that a class can implement in addition to its “primary type,” to declare that it provides some optional behavior. For example, Comparable is a mixin interface that allows a class to declare that its instances are ordered with respect to other mutually comparable objects. Such an interface is called a mixin because it allows the optional functionality to be “mixed in” to the type’s primary functionality. 

**mixin。可以在一个type的基本功能基础上添加额外的功能。**

**接口非常适合用来define mixin。而抽象类凉凉。**

Abstract classes can’t be used to define mixins for the same reason that they can’t be retrofitted onto existing classes: a class cannot have more than one parent, and there is no reasonable place in the class hierarchy to insert a mixin.

**Interfaces allow for the construction of nonhierarchical type frameworks.** 

Type hierarchies are great for organizing some things, but other things don’t fall neatly（整齐的） into a rigid（死板的，严格的） hierarchy. 

**Type hierarchies在有些地方有用，在某些地方他不适用。**

For example, suppose we have an interface representing a singer and another representing a songwriter:

```
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```

In real life, some singers are also songwriters. Because we used interfaces rather than abstract classes to define these types, it is perfectly permissible for a single class to implement both Singer and Songwriter. In fact, we can define a third interface that extends both Singer and Songwriter and adds new methods that are appropriate to the combination:

**在现实生活中，一些歌手也是词曲作者。**

```
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

You don’t always need this level of flexibility, but when you do, interfaces are a lifesaver.

 The alternative is a bloated（臃肿） class hierarchy containing a separate class for every supported combination of attributes. If there are n attributes in the type system, there are 2n possible combinations that you might have to support. This is what’s known as a combinatorial explosion. Bloated class hierarchies can lead to bloated classes with many methods that differ only in the type of their arguments because there are no types in the class hierarchy to capture common behaviors.

**组合爆炸。**

Interfaces enable safe, powerful functionality enhancements via the wrapper class idiom (Item 18). If you use abstract classes to define types, you leave the programmer who wants to add functionality with no alternative but inheritance. The resulting classes are less powerful and more fragile than wrapper classes.

**如果用接口，还能像18条那样用wrapper类做安全，强大的功能增强。**

When there is an obvious implementation of an interface method in terms of other interface methods, consider providing implementation assistance to programmers in the form of a default method. For an example of this technique, see the removeIf method on page 104. If you provide default methods, be sure to document them for inheritance using the @implSpec Javadoc tag (Item 19).

**使用default method，下面是default method的限制。**

**1、不能在接口里面给Object的equals方法等提供default method。（测试过确实如此）**

**2、接口不允许包含实例字段或非公共静态成员（私有静态方法除外）**

**3、只能在接口里面加**

There are limits on how much implementation assistance you can provide with default methods. Although many interfaces specify the behavior of Object methods such as equals and hashCode, you are not permitted to provide default methods for them. 

Also, interfaces are not permitted to contain instance fields or nonpublic static members (with the exception of private static methods). 

Finally, you can’t add default methods to an interface that you don’t control.

**模板方法模式 [Gamma95]。为一个接口提供一个骨架抽象类。**

You can, however, combine the advantages of interfaces and abstract classes by **providing an abstract skeletal implementation class to go with an interface**. The interface defines the type, perhaps providing some default methods, while the skeletal implementation class implements the remaining non-primitive interface methods atop the primitive interface methods. Extending a skeletal implementation takes most of the work out of implementing an interface. This is the Template Method pattern [Gamma95].

By convention, skeletal implementation classes are called **AbstractInterface**, where Interface is the name of the interface they implement. For example, the Collections Framework provides a skeletal implementation to go along with each main collection interface: **AbstractCollection, AbstractSet, AbstractList, and AbstractMap**. 

**Abstract真的非常常见。**

Arguably it would have made sense to call them SkeletalCollection, SkeletalSet, SkeletalList, and SkeletalMap, but the Abstract convention is now firmly established. 

When properly designed, skeletal implementations (whether a separate abstract class, or consisting solely of default methods on an interface) can make it very easy for programmers to provide their own implementations of an interface. 

For example, here’s a static factory method containing a complete, fully functional List implementation atop（在什么之上） AbstractList:

```
// Concrete implementation built atop skeletal implementation
static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);
        // The diamond operator is only legal here in Java 9 and later
        // If you're using an earlier release, specify <Integer>
        return new AbstractList<>() {
            @Override
            public Integer get(int i) {
                return a[i]; // Autoboxing (Item 6)
            }

            @Override
            public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val; // Auto-unboxing
                return oldVal; // Autoboxing
            }

            @Override
            public int size() {
                return a.length;
            }
        };
}
```

**这个例子真是骚操作。**

When you consider all that a List implementation does for you, this example is an impressive demonstration of the power of skeletal implementations.

 Incidentally（顺便说一句）, this example is an **Adapter** [Gamma95] that allows an int array to be viewed as a list of Integer instances. Because of all the translation back and forth between int values and Integer instances (boxing and unboxing), its performance is not terribly good. Note that the implementation takes the form of an anonymous class (Item 24).

The beauty of skeletal implementation classes is that they provide all of the implementation assistance of abstract classes without imposing the severe constraints that abstract classes impose when they serve as type definitions. 

For most implementors of an interface with a skeletal implementation class, extending this class is the obvious choice, but it is strictly optional. If a class cannot be made to extend the skeletal implementation, the class can always implement the interface directly. The class still benefits from any default methods present on the interface itself. Furthermore, the skeletal implementation can still aid the implementor’s task. 

**既可以继承骨架抽象类，也可以直接实现这个接口。**

The class implementing the interface can forward invocations of interface methods to a contained instance of a private inner class that extends the skeletal implementation. This technique, known as **simulated multiple inheritance**, is closely related to the wrapper class idiom discussed in Item 18. It provides many of the benefits of multiple inheritance, while avoiding the pitfalls.

**骚操作，模拟多继承。**

Writing a skeletal implementation is a relatively simple, if somewhat tedious（无聊）, process. 

First, study the interface and decide which methods are the primitives in terms of which the others can be implemented. These primitives will be the abstract methods in your skeletal implementation. 

**第一步从接口里面找出primitives methods，这些放到骨架类中，留意Object的方法**

Next, provide default methods in the interface for all of the methods that can be implemented directly atop the primitives, but recall that you may not provide default methods for Object methods such as equals and hashCode. 

**第二步在接口里给那些基于primitives methods的方法提供默认方法**

If the primitives and default methods cover the interface, you’re done, and have no need for a skeletal implementation class. Otherwise, write a class declared to implement the interface, with implementations of all of the remaining interface methods. The class may contain any nonpublic fields ands methods appropriate to the task.

**第三步，如果接口刚好可以分成上面这两部分东西，就不用写骨架类了。否则把剩下的方法塞到骨架类里面去。**

As a simple example, consider the Map.Entry interface. The obvious primitives are getKey, getValue, and (optionally) setValue. The interface specifies the behavior of equals and hashCode, and there is an obvious implementation of toString in terms of the primitives. Since you are not allowed to provide default implementations for the Object methods, all implementations are placed in the skeletal implementation class:

**作为一个简单的例子，考虑一下 `Map.Entry` 接口。**

```
// Skeletal implementation class
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

    // Entries in a modifiable map must override this method
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Implements the general contract of Map.Entry.equals
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    // Implements the general contract of Map.Entry.hashCode
    @Override public int hashCode() {
        return Objects.hashCode(getKey())^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

Note that this skeletal implementation could not be implemented in the Map.Entry interface or as a subinterface because default methods are not permitted to override Object methods such as equals, hashCode, and toString.

Because skeletal implementations are designed for inheritance, you should follow all of the design and documentation guidelines in Item 19. For brevity’s sake, the documentation comments were omitted from the previous example, but good documentation is absolutely essential in a skeletal implementation, whether it consists of default methods on an interface or a separate abstract class.

A minor variant on the skeletal implementation is the simple implementation, exemplified by AbstractMap.SimpleEntry. A simple implementation is like a skeletal implementation in that it implements an interface and is designed for inheritance, but it differs in that it isn’t abstract: it is the simplest possible working implementation. You can use it as it stands or subclass it as circumstances warrant（保证）.

**skeletal implementation的一个小变种是simple implementation，例如 `AbstractMap.SimpleEntry`。区别是他可以直接拿来用。**

To summarize, an interface is generally the best way to define a type that permits multiple implementations. If you export a nontrivial（adj. 非平凡的） interface, you should strongly consider providing a skeletal implementation to go with it. To the extent possible, you should provide the skeletal implementation via default methods on the interface so that all implementors of the interface can make use of it. That said, **restrictions on interfaces typically mandate （要求）that a skeletal implementation take the form of an abstract class.**

---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Introduction.md)**
- **Previous Item（上一条目）：[Item 19: Design and document for inheritance or else prohibit it（继承要设计良好并且具有文档，否则禁止使用）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)**
- **Next Item（下一条目）：[Item 21: Design interfaces for posterity（为后代设计接口）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-21-Design-interfaces-for-posterity.md)**
