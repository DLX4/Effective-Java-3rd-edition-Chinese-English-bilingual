## Chapter 4. Classes and Interfaces（类和接口）

#### Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）

The single most important factor that distinguishes a well-designed component from a poorly designed one is the degree to which the component hides its internal data and other implementation details from other components. A welldesigned component hides all its implementation details, cleanly separating its API from its implementation. Components then communicate only through their APIs and are oblivious to each others’ inner workings. This concept, known as information hiding or encapsulation, is a fundamental tenet of software design [Parnas72].

**封装（信息隐藏）是最最最重要的软件设计基本宗旨。**

Information hiding is important for many reasons, most of which stem from（源于） the fact that it decouples（解耦） the components that comprise（构成） a system, allowing them to be developed, tested, optimized, used, understood, and modified in isolation（n. 隔离；孤立）.This speeds up system development because components can be developed in parallel. It eases（简化了） the burden of maintenance because components can be understood more quickly and debugged or replaced with little fear of harming other components. While information hiding does not, in and of itself, cause good performance, it enables effective performance tuning: once a system is complete and profiling（剖析） has determined which components are causing performance problems (Item 67), those components can be optimized without affecting the correctness of others. Information hiding increases software reuse because components that aren’t tightly coupled often prove useful in other contexts besides the ones for which they were developed. Finally, information hiding decreases the risk in building large systems because individual components may prove successful even if the system does not.

**封装（信息隐藏）的好处**

**解耦了组成系统的组件，允许它们被独立开发、测试、优化、使用、理解和修改。**

**加快了系统开发进度，因为组件可以并行开发。**

**减轻了维护的负担，因为组件可以被更快地理解、调试或替换，而不必担心会损害其他组件。**

**虽然信息隐藏本身不会获得良好的性能，但它可以实现有效的性能调优：一旦系统完成，概要分析确定了哪些组件会导致性能问题，就可以在不影响其他组件正确性的情况下对这些组件进行优化。**

**信息隐藏增加了软件的复用性，提升了外部有用性（useful）。**

**最后，信息隐藏降低了构建大型系统的风险，因为即使系统没有成功，单个组件也可能被证明是成功的。**

Java has many facilities to aid in information hiding. The access control mechanism [JLS, 6.6] specifies the accessibility of classes, interfaces, and members. The accessibility of an entity is determined by the location of its declaration and by which, if any, of the access modifiers (private,protected, and public) is present on the declaration. Proper use of these modifiers is essential to information hiding.

**java通过访问控制机制来实现封装（信息隐藏）**

The rule of thumb（经验法则） is simple: make each class or member as inaccessible as possible. In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

**在不影响软件正常功能时，使用尽可能低的访问级别。**

For top-level (non-nested) classes and interfaces, there are only two possible access levels: package-private and public. If you declare a top-level class or interface with the public modifier, it will be public; otherwise, it will be package-private. If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients. If you make it public, you are obligated to support it forever to maintain compatibility.

**对于顶级（非嵌套）类和接口，只有两个可能的访问级别：包私有和公共。**

If a package-private top-level class or interface is used by only one class,consider making the top-level class a private static nested class of the sole class that uses it (Item 24). This reduces its accessibility from all the classes in its package to the one class that uses it. 

**如果包级私有顶级类或接口只被一个类使用，那么可以考虑：在使用它的这个类中，将顶级类设置为私有静态嵌套类（[Item-24](../Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。**

But it is far more important to reduce the accessibility of a gratuitously（平白，没用） public class than of a package-private top-level class: the public class is part of the package’s API, while the package-private top-level class is already part of its implementation.

**减少public class的访问级别比减少package-private top-level class 的访问级别更关键。**

For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:

**对于成员（字段、方法、嵌套类和嵌套接口），有四个可能的访问级别**，这里列出了按可访问性依次递增的顺序：

- **private** —The member is accessible only from the top-level class where it is declared.

- **package-private** —The member is accessible from any class in the package where it is declared. Technically known as default access, this is the access level you get if no access modifier is specified (except for interface members,which are public by default).

- **protected** —The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.

- **public** —The member is accessible from anywhere.

After carefully designing your class’s public API, your reflex（反应） should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. If you find yourself doing this often, you should reexamine（重新审视） the design of your system to see if another decomposition（分解） might yield （让，let，make）classes that are better decoupled from one another. That said, both private and package-private members are part of a class’s implementation and do not normally impact its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable (Items 86 and 87).

**在仔细设计了类的公共 API 之后，你的反应应该是使所有其他成员都是私有的。只有当同一包中的另一个类确实需要访问一个成员时，你才应该删除私有修饰符，使成员变为包级私有的。如果你发现自己经常这样做，那么你应该重新确认系统的设计，看看是否有其他方式能产生更好地相互解耦的类。**

**私有成员和包级私有成员都是类实现的一部分，通常不会影响其导出的 API。但是，如果类实现了 Serializable（[Item-86](../Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md) 和 [Item-87](../Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)），这些字段可能会「泄漏」到导出的 API 中。**

For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protected member is part of the class’s exported API and must be supported forever. Also, a protected member of an exported class represents a public commitment to an implementation detail (Item 19). The need for protected members should be relatively rare.

**members 是protected 的场景应该是不常见的。**

There is a key rule that restricts your ability to reduce the accessibility of methods. If a method overrides a superclass method, it cannot have a more restrictive access level in the subclass than in the superclass [JLS, 8.4.8.3]. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the Liskov substitution principle, see Item15). If you violate this rule, the compiler will generate an error message when you try to compile the subclass. A special case of this rule is that if a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

**有一个关键规则限制了你减少方法可访问性的能力。如果一个方法覆盖了超类方法，那么它在子类中的访问级别就不能比在超类 [JLS, 8.4.8.3] 中更严格。这对于确保子类的实例在超类的实例可用的任何地方都同样可用是必要的（Liskov 替换原则，请参阅 [Item-15](../Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。如果违反此规则，编译器将在尝试编译子类时生成错误消息。**

To facilitate（vt. 促进；帮助；使容易） testing your code, you may be tempted to make a class, interface,or member more accessible than otherwise necessary. This is fine up to a point. It is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher. In other words, it is not acceptable to make a class, interface, or member a part of a package’s exported API to facilitate testing. Luckily, it isn’t necessary either because tests can be made to run as part of the package being tested, thus gaining access to its package-private elements.

**没有必要为了便于测试放开范围级别，测试代码可以和被测模块同包。**

**Instance fields of public classes should rarely be public** (Item 16). If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field.This means you give up the ability to enforce invariants(保证不变) involving the field.Also, you give up the ability to take any action when the field is modified, so **classes with public mutable fields are not generally thread-safe.** 

**public class 的instance field很少采用public修饰（[Item-16](../Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)）**。

Even if a field is final and refers to an immutable object, by making it public you give up the flexibility to switch to a new internal data representation in which the field does not exist.

**field不是final或者指向可变的对象的时候，这个field不是线程安全的。**

**field是final并且指向不可变对象，意味着不能表示这个field为空的状态。**

The same advice applies to static fields, with one exception. 

You can expose constants via public static final fields, assuming the constants form an integral part（组成部分） of the abstraction provided by the class. By convention（惯例）, such fields have names consisting of capital letters, with words separated by underscores (Item 68). 

**按照惯例public static final field 的名字由大写字母+下划线组成（[Item-68](../Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)）。**

It is critical that these fields contain either primitive values or references to immutable objects (Item 17). a field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results.

**public static final field要么是primitive values或者是不可变对象的引用。**

Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a field. If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes:

**非零长度的数组总是可变的，因此对于类来说，拥有一个public static final 数组字段或accessor 是错误的。**

```
// Potential security hole!
public static final Thing[] VALUES = { ... };
```

Beware of the fact that some IDEs generate accessors that return references to private array fields, resulting in exactly this problem. 

**accessors 返回private array field会同样导致上面的问题。**

There are two ways to fix the problem. You can make the public array private and add a public immutable list:

**解决方法1：将数组变成private，并添加public static final的不可变的List。**

```
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

Alternatively, you can make the array private and add a public method that returns a copy of a private array:

**解决方法2：将数组变成private,  并用一个public方法返回数组的克隆。**

```
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

To choose between these alternatives, think about what the client is likely to do with the result. Which return type will be more convenient? Which will give better performance?

As of Java 9, there are two additional, implicit（隐式的） access levels introduced as part of the module system. 

**java9引进了两个隐式的访问级别。**

A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via export declarations in its module declaration (which is by convention contained in a source file named module-info.java).

**模块可以在模块声明文件中通过声明显式导出部分packages 。**

 Public and protected members of unexported packages in a module are inaccessible outside the module; within the module, accessibility is unaffected by export declarations. Using the module system allows you to share classes among packages within a module without making them visible to the entire world. 

Public and protected members of public classes in unexported packages give rise to the two implicit access levels, which are intramodular analogues of the normal public and protected levels. The need for this kind of sharing is relatively rare and can often be eliminated by rearranging the classes within your packages.

**由此产生了两种访问级别。但是这个东西是鸡肋的，可以通过重新组织包内的类来达到同样的效果。**

Unlike the four main access levels, the two module-based levels are largely advisory. If you place a module’s JAR file on your application’s class path instead of its module path, the packages in the module revert to their nonmodular behavior: all of the public and protected members of the packages’ public classes have their normal accessibility, regardless of whether the packages are exported by the module [Reinhold, 1.2]. The one place where the newly introduced access levels are strictly enforced is the JDK itself: the unexported packages in the Java libraries are truly inaccessible outside of their modules.

**JDK本身用到了这两个新的访问级别。**

Not only is the access protection afforded（给予） by modules of limited utility to the typical Java programmer, and largely advisory in nature; （**不仅提供了而且建议去用**）

in order to take advantage of it, you must group your packages into modules, make all of their dependencies explicit in module declarations, rearrange your source tree, and take special actions to accommodate（和解） any access to non-modularized packages from within your modules [Reinhold, 3]. It is too early to say whether modules will achieve widespread use outside of the JDK itself. In the meantime, it seems best to avoid them unless you have a compelling need.

**广泛使用还为时尚早。**

To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray（流浪） classes, interfaces, or members from becoming part of the API. **With the exception of public static final fields, which serve as constants,public classes should have no public fields**. Ensure that objects referenced by public static final fields are immutable.

除了public static final常量，public class不应该有public的field。

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-4/Chapter-4-Introduction.md)**
- **Next Item（下一条目）：[Item 16: In public classes use accessor methods not public fields（在公共类中，使用访问器方法，而不是公共字段）](../Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)**
