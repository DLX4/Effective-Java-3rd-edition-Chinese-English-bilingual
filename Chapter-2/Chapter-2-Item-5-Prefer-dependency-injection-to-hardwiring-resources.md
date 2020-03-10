## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）

Many classes depend on one or more underlying（adj.潜在的，根本的） resources. For example, a spell checker depends on a dictionary. It is not uncommon to see such classes implemented as static utility classes (Item 4):

**静态工具类**

```
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // Noninstantiable
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

Similarly, it’s not uncommon to see them implemented as singletons (Item 3):

**单例**

```
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

Neither of these approaches is satisfactory, because they assume that there is only one dictionary worth using. In practice, each language has its own dictionary, and special dictionaries are used for special vocabularies. Also, it may be desirable to use a special dictionary for testing. It is wishful（一厢情愿） thinking to assume that a single dictionary will suffice（满足） for all time.

You could try to have SpellChecker support multiple dictionaries by making the dictionary field nonfinal and adding a method to change the dictionary in an existing spell checker, 

**不用final**

but this would be awkward, error-prone,and unworkable in a concurrent setting. 

麻烦，容易出错，不支持并发

**Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.**

What is required is the ability to support multiple instances of the class (in our example, SpellChecker), each of which uses the resource desired by the client (in our example, the dictionary). A simple pattern that satisfies this requirement is to **pass the resource into the constructor when creating a new instance.** This is one form of dependency injection: the dictionary is a dependency of the spell checker and is injected into the spell checker when it is created.

**这就是依赖注入的例子，checker在创建的时候需要注入一本字典**

```
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

The dependency injection pattern is so simple that many programmers use it for years without knowing it has a name. While our spell checker example had only a single resource (the dictionary), dependency injection works with an arbitrary（adj.任意的） number of resources and arbitrary dependency graphs. 

It preserves immutability (Item 17), 

**保留了不变性**

so multiple clients can share dependent objects(assuming the clients desire the same underlying resources（底层资源）). Dependency injection is equally applicable（适用） to constructors, static factories (Item 1), and builders (Item 2).

A useful variant of the pattern is to pass a resource factory to the constructor.A factory is an object that can be called repeatedly to create instances of a type.Such factories embody（体现） the Factory Method pattern [Gamma95]. 

**这就是设计模式里面的工程方法**

The `Supplier<T>` interface, introduced in Java 8, is perfect for representing factories. Methods that take a `Supplier<T>` on input should typically constrain the factory’s type parameter using a bounded wildcard type (Item 31) to allow the client to pass in a factory that creates any subtype of a specified type. For example, here is a method that makes a mosaic using a client-provided factory to produce each tile:

```
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

Although dependency injection greatly improves flexibility and testability, it can clutter up（使杂乱） large projects, which typically（adv.通常） contain thousands of dependencies.

**几千个依赖的大工程**

This clutter（混乱） can be all but eliminated（消除） by using a dependency injection framework, such as Dagger [Dagger], Guice [Guice], or Spring [Spring]. 

依赖注入的框架

The use of these frameworks is beyond the scope of this book, but note that APIs designed for manual（n.手册；adj.手工的） dependency injection are trivially（adv.繁琐地） adapted for（适用于） use by these frameworks.

In summary, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder). This practice, known as dependency injection, will greatly enhance the flexibility, reusability, and testability of a class.



---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 4: Enforce noninstantiability with a private constructor（用私有构造函数实施不可实例化）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)**
- **Next Item（下一条目）：[Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)**
