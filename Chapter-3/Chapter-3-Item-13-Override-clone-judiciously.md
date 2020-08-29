## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 13: Override clone judiciously（明智地覆盖 clone 方法）

The Cloneable interface was intended（目的） as a mixin interface (Item 20) for classes to advertise that they permit cloning. 

Unfortunately, it fails to serve this purpose. 

**又不行，因为缺少克隆方法，而Object的克隆方法又是protected的。**

Its primary flaw（n. 瑕疵，缺点） is that it lacks a clone method, and Object’s clone method is protected. 

You cannot, without resorting（求助） to reflection (Item 65), invoke clone on an object merely（adv. 仅仅，只是） because it implements Cloneable.

**并不能够看他实现了Cloneable接口就去调用克隆接口。（我已经证明过了）**

Even a reflective invocation may fail, because there is no guarantee（n. 保证；担保） that the object has an accessible clone method. Despite this flaw and many others, the facility（n. 设施；设备） is in reasonably wide use, so it pays to understand it. This item tells you how to implement a well-behaved clone method, discusses when it is appropriate to do so, and presents alternatives.

So what does Cloneable do, given that it contains no methods? It determines the behavior of Object’s protected clone implementation: if a class implements Cloneable, Object’s clone method returns a field-byfield copy of the object; otherwise it throws CloneNotSupportedException. This is a highly atypical use of interfaces and not one to be emulated. 

Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

如果 Cloneable 不包含任何方法，它会做什么呢？它决定 Object 的受保护克隆实现的行为：如果一个类实现了 Cloneable，对象的克隆方法返回对象的逐域拷贝；否则它会抛出 CloneNotSupportedException。这是接口的一种高度非典型使用，而不是可模仿的。通常，实现接口说明了类可以为其客户做些什么。

在本例中，它修改了超类上受保护的方法行为。

**奇葩用法**

Though the specification（n. 规格；说明书；详述） doesn’t say it, in practice, a class implementing Cloneable is expected to provide a properly（adv. 适当地；正确地；恰当地） functioning public clone method. In order to achieve（vt. 取得；获得；实现；） this, the class and all of its superclasses must obey a complex（adj. 复杂的；合成的）, unenforceable, thinly documented protocol. The resulting mechanism is fragile, dangerous, and extralinguistic（adj. 语言以外的；语言学以外的）: it creates objects without calling a constructor.

**虽然规范没有说明，但是在实践中，一个实现 Cloneable 的类应该提供一个功能正常的公共 clone 方法。**

为了实现这一点，类及其所有超类必须遵守复杂的、不可强制执行的、文档很少的协议。产生的机制是脆弱的、危险的和非语言的：即它创建对象而不调用构造函数。

The general contract for the clone method is weak. Here it is, copied from the Object specification（n. 规格；说明书；详述） :

**clone 方法的契约很弱。**

Creates and returns a copy of this object. The precise meaning of “copy” may depend on the class of the object. The general intent（n. 意图；目的；含义，adj. 专心的；急切的；坚决的） is that, for any object x,the expression

```
x.clone() != x
```

will be true, and the expression

```
x.clone().getClass() == x.getClass()
```

will be true, but these are not absolute requirements（n. 要求；必要条件；）. While it is typically the case that

```
x.clone().equals(x)
```

will be true, this is not an absolute requirement.

By convention（n. 大会；惯例；约定；协定；习俗）, the object returned by this method should be obtained（v. 获得） by calling super.clone. If a class and all of its superclasses (except Object) obey this convention, it will be the case that

按照惯例，这个方法返回的对象应该通过调用 super.clone 来获得。如果一个类和它的所有超类（对象除外）都遵守这个约定，那么情况就是这样

```
x.clone().getClass() == x.getClass().
```

**By convention, the returned object should be independent of the object being cloned.** 

To achieve this independence, it may be necessary to modify one or more fields of the object returned by super.clone before returning it.



This mechanism is vaguely similar to constructor chaining, except that it isn’t enforced: 

if a class’s clone method returns an instance that is not obtained by calling super.clone but by calling a constructor, the compiler won’t complain（报错）, 

**调用构造方法会报错**

but if a subclass of that class calls super.clone, the resulting object will have the wrong class, preventing the subclass from clone method from working properly. 

**子类调用super.clone也是不行的**

If a class that overrides clone is final, this convention may be safely ignored, as there are no subclasses to worry about. But if a final class has a clone method that does not invoke super.clone, there is no reason for the class to implement Cloneable, as it doesn’t rely on the behavior of Object’s clone implementation.

**实现了Cloneable接口但是不调用也是不行的。**

**这种机制类似constructor chaining。**

Suppose you want to implement Cloneable in a class whose superclass provides a well-behaved clone method. First call super.clone. The object you get back will be a fully functional replica of the original. Any fields declared in your class will have values identical to those of the original. If every field contains a primitive value or a reference to an immutable object, the returned object may be exactly what you need, in which case no further processing is necessary. This is the case, for example, for the PhoneNumber class in Item 11, 

but note that **immutable classes should never provide a clone method** because it would merely encourage wasteful copying. With that caveat, here’s how a clone method for PhoneNumber would look:

**不可变类没有必要去克隆。**

假设你希望在一个类中实现 Cloneable，该类的超类提供了一个表现良好的克隆方法。第一个叫 super.clone。返回的对象将是原始对象的完整功能副本。类中声明的任何字段都具有与原始字段相同的值。如果每个字段都包含一个基元值或对不可变对象的引用，那么返回的对象可能正是你所需要的，在这种情况下不需要进一步的处理。例如，对于[Item-11](../Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)中的 PhoneNumber 类就是这样，但是要注意，**不可变类永远不应该提供克隆方法**，因为它只会鼓励浪费复制。有了这个警告，以下是 PhoneNumber 的克隆方法的外观：

```
// Clone method for class with no references to mutable state
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // Can't happen
    }
}
```

In order for this method to work, the class declaration for PhoneNumber would have to be modified to indicate that it implements Cloneable. Though Object’s clone method returns Object, this clone method returns PhoneNumber. It is legal and desirable to do this because Java supports covariant return types. In other words, an overriding method’s return type can be a subclass of the overridden method’s return type. This eliminates the need for casting in the client. We must cast the result of super.clone from Object to PhoneNumber before returning it, but the cast is guaranteed to succeed.

**为了让这个方法工作，必须修改 PhoneNumber 的类声明，以表明它实现了 Cloneable。**

**虽然 Object 的 clone 方法返回 Object，但是这个 clone 方法返回 PhoneNumber。这样做是合法的，也是可取的，因为 Java 支持协变返回类型。换句话说，覆盖方法的返回类型可以是被覆盖方法的返回类型的子类。这样就不需要在客户端中进行强制转换。**

The call to super.clone is contained in a try-catch block. This is because Object declares its clone method to throw CloneNotSupportedException, which is a checked exception. Because PhoneNumber implements Cloneable, we know the call to super.clone will succeed. The need for this boilerplate indicates that CloneNotSupportedException should have been unchecked (Item 71).

**对 super.clone 的调用包含在 try-catch 块中。这是因为 Object 声明其克隆方法来抛出 CloneNotSupportedException。因为 PhoneNumber 实现了 Cloneable，所以我们知道对 super.clone 的调用将会成功。**

If an object contains fields that refer to mutable objects, the simple clone implementation shown earlier can be disastrous. For example, consider the Stack class in Item 7:

**如果对象包含引用可变对象的字段，前面所示的简单克隆实现可能是灾难性的。**

```
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
      this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    // Ensure space for at least one more element.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

Suppose you want to make this class cloneable. If the clone method merely（adv. 仅仅，只不过；只是） returns super.clone(), the resulting Stack instance will have the correct value in its size field, but its elements field will refer to the same array as the original Stack instance. Modifying the original will destroy the invariants in the clone and vice versa. You will quickly find that your program produces nonsensical results or throws a NullPointerException.

This situation could never occur as a result of calling the sole constructor in the Stack class. In effect, the clone method functions as a constructor;you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone. In order for the clone method on Stack to work properly, it must copy the internals of the stack. The easiest way to do this is to call clone recursively on the elements array:

```
// Clone method for class with references to mutable state
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

Note that we do not have to cast the result of elements.clone to Object[]. Calling clone on an array returns an array whose runtime and compile-time types are identical to those of the array being cloned. This is the preferred idiom to duplicate an array. In fact, arrays are the sole compelling use of the clone facility（n. 设施；设备；容易；灵巧）.

Note also that the earlier solution would not work if the elements field were final because clone would be prohibited from assigning a new value to the field. 

This is a fundamental problem: like serialization, the Cloneable architecture is incompatible with normal use of final fields referring to mutable objects, except in cases where the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove final modifiers from some fields.

**克隆机制和final是不兼容的。**

It is not always sufficient（adj. 足够的；充分的） merely to call clone recursively. For example,suppose you are writing a clone method for a hash table whose internals consist of an array of buckets, each of which references the first entry in a linked list of key-value pairs. For performance, the class implements its own lightweight singly linked list instead of using java.util.LinkedList internally:

```
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    } ... // Remainder omitted
}
```

Suppose you merely clone the bucket array recursively, as we did for Stack:

```
// Broken clone method - results in shared mutable state!
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

Though the clone has its own bucket array, this array references the same linked lists as the original, which can easily cause nondeterministic behavior in both the clone and the original. To fix this problem, you’ll have to copy the linked list that comprises each bucket. Here is one common approach:

```
// Recursive clone method for class with complex mutable state
public class HashTable implements Cloneable {

private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        // Recursively copy the linked list headed by this Entry
        Entry deepCopy() {
            return new Entry(key, value,next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();

            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    } ... // Remainder omitted
}
```

The private class HashTable.Entry has been augmented to support a “deep copy” method. The clone method on HashTable allocates a new buckets array of the proper size and iterates over the original buckets array,deep-copying each nonempty bucket. The deepCopy method on Entry invokes itself recursively to copy the entire linked list headed by the entry. While this technique is cute and works fine if the buckets aren’t too long, it is not a good way to clone a linked list because it consumes one stack frame for each element in the list. If the list is long, this could easily cause a stack overflow. To prevent this from happening, you can replace the recursion in deepCopy with iteration:

```
// Iteratively copy the linked list headed by this Entry
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

A final approach to cloning complex mutable objects is to call super.clone, set all of the fields in the resulting object to their initial state,and then call higher-level methods to regenerate the state of the original object.In the case of our HashTable example, the buckets field would be initialized to a new bucket array, and the put(key, value) method (not shown) would be invoked for each key-value mapping in the hash table being cloned. This approach typically yields a simple, reasonably elegant clone method that does not run as quickly as one that directly manipulates the innards of the clone. While this approach is clean, it is antithetical to the whole Cloneable architecture because it blindly overwrites the field-by-field object copy that forms the basis of the architecture.

Like a constructor, a clone method must never invoke an overridable method on the clone under construction (Item 19). If clone invokes a method that is overridden in a subclass, this method will execute before the subclass has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original. Therefore, the put(key, value) method discussed in the previous paragraph should be either final or private. (If it is private, it is presumably the “helper method” for a nonfinal public method.)

Object’s clone method is declared to throw CloneNotSupportedException, but overriding methods need not. **Public clone methods should omit the throws clause,** as methods that don’t throw checked exceptions are easier to use (Item 71).

You have two choices when designing a class for inheritance (Item 19), but whichever one you choose, the class should not implement Cloneable. You may choose to mimic the behavior of Object by implementing a properly functioning protected clone method that is declared to throw CloneNotSupportedException. This gives subclasses the freedom to implement Cloneable or not, just as if they extended Object directly.Alternatively, you may choose not to implement a working clone method, and to prevent subclasses from implementing one, by providing the following degenerate clone implementation:

```
// clone method for extendable class not supporting Cloneable
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

There is one more detail that bears noting. If you write a thread-safe class that implements Cloneable, remember that its clone method must be properly synchronized, just like any other method (Item 78). Object’s clone method is not synchronized, so even if its implementation is otherwise satisfactory, you may have to write a synchronized clone method that returns super.clone().

To recap, all classes that implement Cloneable should override clone with a public method whose return type is the class itself. This method should first call super.clone, then fix any fields that need fixing. Typically, this means copying any mutable objects that comprise the internal “deep structure” of the object and replacing the clone’s references to these objects with references to their copies. While these internal copies can usually be made by calling clone recursively, this is not always the best approach. If the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID will need to be fixed even if it is primitive or immutable.

Is all this complexity really necessary? Rarely. If you extend a class that already implements Cloneable, you have little choice but to implement a well-behaved clone method. Otherwise, you are usually better off providing an alternative means of object copying. A better approach to object copying is to provide a copy constructor or copy factory. A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example,

**复制构造方法**

```
// Copy constructor
public Yum(Yum yum) { ... };
```

A copy factory is the static factory (Item 1) analogue of a copy constructor:

**复制工厂**

```
// Copy factory
public static Yum newInstance(Yum yum) { ... };
```

The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic（语言外的） object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.

**复制构造方法优点：它们不依赖于易发生风险的语言外对象创建机制（diss 克隆）；他们不要求无法强制执行的约定（diss 克隆）；和final不冲突（diss 克隆）；它们不会抛出不必要的检查异常（diss 克隆）；而且不需要强制类型转换（diss 克隆）。**

Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. 

For example, by convention all general purpose collection implementations provide a constructor whose argument is of type Collection or Map. 

Interface-based copy constructors and factories,more properly known as conversion constructors and conversion factories, allow the client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original. For example, suppose you have a HashSet, s, and you want to copy it as a TreeSet. The clone method can’t offer this functionality, but it’s easy with a conversion constructor:new TreeSet<>(s).

**转换构造方法（转换工厂）。例如，假设你有一个 HashSets，并且希望将它复制为 TreeSet。克隆方法不能提供这种功能，但是使用转换构造函数很容易：new TreeSet<>(s)。**

Given all the problems associated（adj. 关联的；联合的） with Cloneable, new interfaces should not extend it, and new extendable classes should not implement it. While it’s less harmful for final classes to implement Cloneable, this should be viewed as a performance optimization, reserved for the rare cases where it is justified (Item 67). As a rule, copy functionality is best provided by constructors or factories. A notable exception to this rule is arrays, which are best copied with the clone method.

**不要使用Cloneable 。**

**虽然 final 类实现 Cloneable 的危害要小一些。**

**通常，复制功能最好由构造函数或工厂提供。**

**最好使用 clone 方法来复制数组。**

---
**[Back to contents of the chapter（返回章节目录）](../Chapter-3/Chapter-3-Introduction.md)**
- **Previous Item（上一条目）：[Item 12: Always override toString（始终覆盖 toString 方法）](../Chapter-3/Chapter-3-Item-12-Always-override-toString.md)**
- **Next Item（下一条目）：[Item 14: Consider implementing Comparable（考虑实现 Comparable 接口）](../Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)**
