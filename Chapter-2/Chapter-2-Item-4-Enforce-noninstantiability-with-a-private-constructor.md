## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 4: Enforce noninstantiability with a private constructor

Occasionally（adv.偶尔） you’ll want to write a class that is just a grouping of static methods and static fields. 

**只有静态方法和field的类**

Such classes have acquired（v.取得） a bad reputation（n.名声） because some people abuse them to avoid thinking in terms of objects, 

but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of java.lang.Math or java.util.Arrays. 

They can also be used to group static methods, including factories (Item 1), for objects that implement some interface, in the manner of java.util.Collections. 

**java.lang.Math 里面的静态方法参数和返回值全是基本类型的，java.util.Arrays里面的静态方法参数和返回值也全是基本类型的。java.util.Collections有很多排序比较的静态方法。**

(As of Java 8, you can also put such methods in the interface, assuming it’s yours to modify.)

 Lastly, such classes can be used to group methods on a final class, since you can’t put them in a subclass.

**也就是说上面这种操作也能用来扩展final类**

Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

**Attempting to enforce noninstantiability by making a class abstract does not work.** The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so **a class can be made noninstantiable by including a private constructor:**

**哈哈哈 Collections就是这么干的**

```
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    } ... // Remainder omitted
}
```

Because the explicit constructor is private, it is inaccessible outside the class.The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. 

It guarantees the class will never be instantiated under any circumstances. This idiom（惯用做法） is mildly counterintuitive（有点违法直觉） because the constructor is provided expressly（明确的） so that it cannot be invoked. It is therefore wise to include a comment, as shown earlier.

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly （显式）or implicitly（隐式）, and a subclass would have no accessible superclass constructor to invoke.



---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 3: Enforce the singleton property with a private constructor or an enum type（使用私有构造函数或枚举类型实施单例属性）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)**
- **Next Item（下一条目）：[Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2/Chapter-2-Item-5-Prefer-dependency-injection-to-hardwiring-resources.md)**
