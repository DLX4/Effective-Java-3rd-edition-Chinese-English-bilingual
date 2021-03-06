## Chapter 5. Generics（泛型）

### Item 28: Prefer lists to arrays（list 优于数组）

Arrays differ from generic types in two important ways. First, arrays are **covariant**. This scary-sounding word means simply that 

**if Sub is a subtype of Super, then the array type Sub[] is a subtype of the array type Super[].** 

**数组是协变的。**

Generics, by contrast, are invariant: for any two distinct types Type1 and Type2, `List<Type1>` is neither a subtype nor a supertype of `List<Type2>` [JLS, 4.10; Naftalin07, 2.5]. 

**泛型是不变的。**

You might think this means that generics are deficient, but arguably（可能，大概） it is arrays that are deficient. This code fragment is legal:

**你可能认为这意味着泛型是有缺陷的，其实数组可能才是有缺陷的。**

```
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
```

**哈哈哈哈，数组这有坑，说是协变，但是运行时又是会报错的。**

but this one is not:

但这一段代码就不是：

```
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

Either way you can’t put a String into a Long container, but with an array you find out that you’ve made a mistake at runtime; with a list, you find out at compile time. Of course, you’d rather find out at compile time.

**两种方法都不能将 String 放入 Long 容器，但使用数组，你会得到一个运行时错误；使用 list，你可以在编译时发现问题。显然，你更希望在编译时发现问题。**

The second major difference between arrays and generics is that arrays are reified（adj. 具体化的） [JLS, 4.7]. 

This means that arrays know and enforce their element type at runtime. As noted earlier, if you try to put a String into an array of Long, you’ll get an ArrayStoreException. 

**数组在运行时能知道并约束元素的类型。**

Generics, by contrast, are implemented by erasure [JLS, 4.6]. This means that they enforce their type constraints only at compile time and discard (or erase) their element type information at runtime. 

**泛型只在编译期约束类型，运行时的时候早就被擦掉了。**

Erasure is what allowed generic types to interoperate freely with legacy code that didn’t use generics (Item 26), ensuring a smooth（adj. 顺利的；光滑的；平稳的） transition to generics in Java 5.

**为啥要搞擦除？是为了兼容 java 5之前的代码。（[Item-26](../Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)）**

Because of these fundamental（adj. 基本的，根本的） differences, arrays and generics do not mix well. For example, it is illegal to create an array of a generic type, a parameterized type, or a type parameter. Therefore, none of these array creation expressions are legal: `new List<E>[]`, `new List<String>[]`, `new E[]`. All will result in generic array creation errors at compile time.

**因为上边那两大差异，数组和泛型不能混在一起用。创建泛型、参数化类型或类型参数的数组都是非法的（`new List<E>[]`、`new List<String>[]`、`new E[]`）。**

**Why is it illegal to create a generic array? Because it isn’t typesafe.** If it were legal, casts generated by the compiler in an otherwise correct program could fail at runtime with a ClassCastException. **This would violate the fundamental guarantee provided by the generic type system.**

为什么创建泛型数组是非法的？因为这不是typesafe的。如果合法，编译器在其他正确的程序中生成的强制转换在运行时可能会失败，并导致 ClassCastException。这将违反泛型系统提供的基本保证。

To make this more concrete, consider the following code fragment:

为了更具体，请考虑以下代码片段：

```
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

Let’s pretend that line 1, which creates a generic array, is legal. 

Line 2 creates and initializes a `List<Integer>` containing a single element. 

Line 3 stores the `List<String>` array into an Object array variable, which is legal because arrays are covariant. 

**因为数组是协变的所以这样赋值是没有问题的。**

Line 4 stores the `List<Integer>` into the sole element of the Object array, which succeeds because generics are implemented by erasure: the runtime type of a `List<Integer>` instance is simply List, and the runtime type of a `List<String>`[] instance is List[], so this assignment doesn’t generate an ArrayStoreException. 

**第四步居然也不报错的。`List<Integer>` 实例的运行时类型是 List，`List<String>`[] 实例的运行时类型是 List[]，因此这个赋值不会生成 ArrayStoreException。**

Now we’re in trouble. We’ve stored a `List<Integer>` instance into an array that is declared to hold only `List<String>` instances. 

In line 5, we retrieve the sole element from the sole list in this array. The compiler automatically casts the retrieved element to String, but it’s an Integer, so we get a ClassCastException at runtime. 

**第五步本来是没错的，人家是老老实实的泛型代码，结果给整报错了。**

In order to prevent this from happening, line 1 (which creates a generic array) must generate a compile-time error.

**为了防止这种情况发生，第 1 行（创建泛型数组）必须生成编译时错误。（哈哈哈 ，把这个口子给堵上了）**

Types such as E, `List<E>`, and `List<String>` are technically known as nonreifiable types [JLS, 4.7]. 

**专业术语来了，nonreifiable types。**

Intuitively speaking, a non-reifiable type is one whose runtime representation contains less information than its compile-time representation. 

**non-reifiable type就是它运行时的表示比编译期的表示要包含更少的信息。**

Because of erasure, the only parameterized types that are reifiable are unbounded wildcard types such as `List<?>` and `Map<?,?>` (Item 26). It is legal, though rarely useful, to create arrays of unbounded wildcard types.

无限制通配符类型是reifiable 的，如 `List<?>` 和 `Map<?,?>`（[Item-26](../Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)）。创建无边界通配符类型数组是合法的，但没啥用。

**The prohibition on generic array creation can be annoying**. 

It means, for example, that it’s not generally possible for a generic collection to return an array of its element type (but see Item 33 for a partial solution). 

让一个泛型集合返回一个element type的数组通常是不能的（这里实现了[Item-33](../Chapter-5/Chapter-5-Item-33-Consider-typesafe-heterogeneous-containers.md)）。

It also means that you get confusing（adj. 混乱的；混淆的；令人困惑的） warnings when using varargs methods (Item 53) in combination with generic types. This is because every time you invoke a varargs method, an array is created to hold the varargs parameters. If the element type of this array is not reifiable, you get a warning. The SafeVarargs annotation can be used to address this issue (Item 32).

带有泛型varargs的方法（[Item-53](../Chapter-8/Chapter-8-Item-53-Use-varargs-judiciously.md)）。SafeVarargs 注解可以用来解决这个问题（[Item-32](../Chapter-5/Chapter-5-Item-32-Combine-generics-and-varargs-judiciously.md)）。 

**译注：varargs 方法，指带有可变参数的方法。**

When you get a **generic array creation error** or an **unchecked cast warning on a cast to an array type**, the best solution is often to use the collection type `List<E>` in preference to the array type E[]. You might sacrifice some conciseness or performance, but in exchange you get better type safety and interoperability.

**使用集合类型 `List<E>`，而不是数组类型 E[]**

For example, suppose you want to write a Chooser class with a constructor that takes a collection, and a single method that returns an element of the collection chosen at random. Depending on what collection you pass to the constructor, you could use a chooser as a game die, a magic 8-ball, or a data source for a Monte Carlo simulation. Here’s a simplistic implementation without generics:

例子（不用泛型的简单实现）。

```
// Chooser - a class badly in need of generics!
public class Chooser {
  private final Object[] choiceArray;

  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
}

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

To use this class, you have to cast the choose method’s return value from Object to the desired type every time you use invoke the method, and the cast will fail at runtime if you get the type wrong. Taking the advice of Item 29 to heart, we attempt to modify Chooser to make it generic. Changes are shown in boldface:

要使用这个类，每次使用方法调用时，必须将 choose 方法的返回值从对象转换为所需的类型，如果类型错误，转换将在运行时失败。我们认真考虑了 [Item-29](../Chapter-5/Chapter-5-Item-29-Favor-generic-types.md) 的建议，试图对 Chooser 进行修改，使其具有通用性。变化以粗体显示：

```
// A first cut at making Chooser generic - won't compile
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }

  // choose method unchanged
}
```

If you try to compile this class, you’ll get this **error** message:

如果你尝试编译这个类，你将得到这样的错误消息：

```
Chooser.java:9: error: incompatible types: Object[] cannot be converted to T[]
choiceArray = choices.toArray();
^ where T is a type-variable:
T extends Object declared in class Chooser
```

No big deal, you say, I’ll cast the Object array to a T array:

没什么大不了的，你会说，我把对象数组转换成 T 数组：

```
choiceArray = (T[]) choices.toArray();
```

This gets rid of the error, but instead you get a **warning**:

这样就消除了错误，但你得到一个警告：

```
Chooser.java:9: warning: [unchecked] unchecked cast choiceArray = (T[]) choices.toArray();
^ required: T[], found: Object[]
where T is a type-variable:
T extends Object declared in class Chooser
```

The compiler is telling you that it can’t vouch for the safety of the cast at runtime because the program won’t know what type T represents—remember, element type information is erased from generics at runtime. Will the program work? Yes, but the compiler can’t prove it. You could prove it to yourself, put the proof in a comment and suppress the warning with an annotation, but you’re better off eliminating the cause of warning (Item 27).

编译器警告的原因，因为程序不知道类型 T 代表什么。记住，元素类型信息在运行时从泛型中删除。需要你自己证明这样做OK然后suppress，记得备注原因（[Item-27](../Chapter-5/Chapter-5-Item-27-Eliminate-unchecked-warnings.md)）。

**To eliminate the unchecked cast warning, use a list instead of an array.** Here is a version of the Chooser class that compiles without error or warning:

若要消除 unchecked 强制转换警告，请使用 list 而不是数组。**下面是编译时没有错误或警告的 Chooser 类的一个版本：**

```
// List-based Chooser - typesafe
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

**This version is a tad more verbose, and perhaps a tad slower, but it’s worth it for the peace of mind that you won’t get a ClassCastException at runtime.**

**可以安心了。**

**In summary, arrays and generics have very different type rules. Arrays are covariant and reified; generics are invariant and erased. As a consequence, arrays provide runtime type safety but not compile-time type safety, and vice versa for generics. As a rule, arrays and generics don’t mix well. If you find yourself mixing them and getting compile-time errors or warnings, your first impulse should be to replace the arrays with lists.**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-5/Chapter-5-Introduction.md)**
- **Previous Item（上一条目）：[Item 27: Eliminate unchecked warnings（消除 unchecked 警告）](../Chapter-5/Chapter-5-Item-27-Eliminate-unchecked-warnings.md)**
- **Next Item（下一条目）：[Item 29: Favor generic types（优先使用泛型）](../Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)**

