## Chapter 5. Generics（泛型）

### Item 27: Eliminate unchecked warnings（消除 unchecked 警告）

When you program with generics, you will see many compiler warnings: unchecked cast warnings, unchecked method invocation warnings, unchecked parameterized vararg type warnings, and unchecked conversion warnings. The more experience you acquire with generics, the fewer warnings you’ll get, but don’t expect newly written code to compile cleanly.

当你使用泛型编程时，你将看到许多编译器警告：unchecked 强制转换警告、unchecked 方法调用警告、unchecked 可变参数类型警告和 unchecked 自动转换警告。使用泛型获得的经验越多，得到的警告就越少，但是不要期望新编写的代码能够完全正确地编译。

Many unchecked warnings are easy to eliminate. For example, suppose you accidentally write this declaration:

许多 unchecked 警告很容易消除。例如，假设你不小心写了这个声明：

```
Set<Lark> exaltation = new HashSet();
```

The compiler will gently remind you what you did wrong:

编译器会精确地提醒你做错了什么：

```
Venery.java:4: warning: [unchecked] unchecked conversion
Set<Lark> exaltation = new HashSet();
^ required: Set<Lark>
found: HashSet
```

You can then make the indicated（v. 表明；指出；显示；adj. 表明的；指示的） correction, causing the warning to disappear. 

Note that you don’t actually have to specify the type parameter, merely（adv. 仅仅，只不过；只是） to indicate（vt. 表明；指出；预示；象征） that it’s present with the diamond operator (<>), introduced in Java 7. **The compiler will then infer the correct actual type parameter (in this case, Lark):**

**java 7 引入的钻石形状的操作符<>**

```
Set<Lark> exaltation = new HashSet<>();
```

Some warnings will be much more difficult to eliminate. This chapter is filled with examples of such warnings. When you get warnings that require some thought, persevere（坚持下去）! **Eliminate every unchecked warning that you can.** 

**If you eliminate all warnings, you are assured that your code is typesafe, which is a very good thing. It means that you won’t get a ClassCastException at runtime, and it increases your confidence that your program will behave as you intended.**

**消除所有编译告警即可保证typesafe。**

**If you can’t eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an @SuppressWarnings("unchecked") annotation.** If you suppress warnings without first proving that the code is typesafe, you are giving yourself a false sense of security. The code may compile without emitting any warnings, but it can still throw a ClassCastException at runtime. If, however, you ignore unchecked warnings that you know to be safe (instead of suppressing them), you won’t notice when a new warning crops up that represents a real problem. The new warning will get lost amidst all the false alarms that you didn’t silence.

**如果不能消除警告，但是可以（人为）证明引发警告的代码是类型安全的，那么（并且只有在那时）使用 SuppressWarnings("unchecked") 注解来抑制警告。** 

**没有人为证明就草率suppress掉，自欺欺人；**

**如果对告警视而不见，就好比是狼来了的故事。**

The SuppressWarnings annotation can be used on any declaration, from an individual local variable declaration to an entire class. **Always use the SuppressWarnings annotation on the smallest scope possible.** Typically this will be a variable declaration or a very short method or constructor. Never use SuppressWarnings on an entire class. Doing so could mask critical warnings.

SuppressWarnings 注解可以用于任何声明中，从单个局部变量声明到整个类。**总是在尽可能小的范围上使用 SuppressWarnings 注解。** 通常用在一个变量声明或一个非常短的方法或构造函数。不要在整个类中使用 SuppressWarnings。这样做可能会掩盖关键警告。

If you find yourself using the SuppressWarnings annotation on a method or constructor that’s more than one line long, you may be able to move it onto a local variable declaration. You may have to declare a new local variable, but it’s worth it. For example, consider this toArray method, which comes from ArrayList:

如果你发现自己在一个超过一行的方法或构造函数上使用 SuppressWarnings 注解，那么你可以将其移动到局部变量声明中。你可能需要声明一个新的局部变量，但这是值得的。例如，考虑这个 toArray 方法，它来自 ArrayList：

```
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

If you compile ArrayList, the method generates this warning:

如果你编译 ArrayList，这个方法会产生这样的警告：

```
ArrayList.java:305: warning: [unchecked] unchecked cast
return (T[]) Arrays.copyOf(elements, size, a.getClass());
^ required: T[]
found: Object[]
```

It is illegal to put a SuppressWarnings annotation on the return statement, because it isn’t a declaration [JLS, 9.7]. You might be tempted to put the annotation on the entire method, but don’t. Instead, declare a local variable to hold the return value and annotate its declaration, like so:

将 SuppressWarnings 注释放在 return 语句上是非法的，因为它不是声明 [JLS, 9.7]。你可能想把注释放在整个方法上，但是不要这样做。相反，**应该声明一个局部变量来保存返回值并添加注解**，如下所示：

```
// Adding local variable to reduce scope of @SuppressWarnings
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // This cast is correct because the array we're creating
        // is of the same type as the one passed in, which is T[].
        @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

The resulting method compiles cleanly and minimizes the scope in which unchecked warnings are suppressed.

生成的方法编译ok，并将抑制 unchecked 警告的范围减到最小。

**Every time you use a @SuppressWarnings("unchecked") annotation, add a comment saying why it is safe to do so.** This will help others understand the code, and more importantly, it will decrease the odds that someone will modify the code so as to make the computation unsafe. If you find it hard to write such a comment, keep thinking. You may end up figuring out that the unchecked operation isn’t safe after all.

**每次使用 SuppressWarnings("unchecked") 注解时，要添加一条注释，说明这样做是安全的。** 这将帮助他人理解代码，更重要的是，它将降低其他人修改代码而产生不安全事件的几率。**如果你觉得很难写出这样的注释，继续想，你可能发现这个unchecked operation压根儿就不安全**。

In summary, unchecked warnings are important. Don’t ignore them. Every unchecked warning represents the potential for a ClassCastException at runtime. Do your best to eliminate these warnings. If you can’t eliminate an unchecked warning and you can prove that the code that provoked it is typesafe, suppress the warning with a @SuppressWarnings("unchecked") annotation in the narrowest possible scope. Record the rationale for your decision to suppress the warning in a comment.

**总之，unchecked 警告很重要。不要忽视他们。每个 unchecked 警告都代表了在运行时发生 ClassCastException 的可能性。尽最大努力eliminate 这些警告。如果不能eliminate  unchecked 警告，并且可以证明引发该警告的代码是类型安全的，那么可以在尽可能小的范围内使用 @SuppressWarnings("unchecked") 注释来suppress警告。并把你suppress告警的理由注释下来。**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-5/Chapter-5-Introduction.md)**
- **Previous Item（上一条目）：[Item 26: Do not use raw types（不要使用原始类型）](../Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)**
- **Next Item（下一条目）：[Item 28: Prefer lists to arrays（list 优于数组）](../Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md)**
