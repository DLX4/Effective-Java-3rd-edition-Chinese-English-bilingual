## Chapter 4. Classes and Interfaces（类和接口）

### Item 16: In public classes, use accessor methods, not public fields（在公共类中，使用访问器方法，而不是公共字段）

Occasionally, you may be tempted to write degenerate（退化） classes that serve no purpose other than to group instance fields:

**有时候，可能会编写一些退化类，这些类除了对实例字段进行分组之外，没有其他用途：**

```
// Degenerate classes like this should not be public!
class Point {
    public double x;
    public double y;
}
```

Because the data fields of such classes are accessed directly, these classes do not offer the benefits of encapsulation (Item 15). You can’t change the representation without changing the API, you can’t enforce invariants, and you can’t take auxiliary（辅助） action when a field is accessed. Hard-line （强硬路线）object-oriented programmers feel that such classes are anathema（厌恶） and should always be replaced by classes with private fields and public accessor methods (getters) and, for mutable classes, mutators (setters):

```
// Encapsulation of data by accessor methods and mutators
class Point {
    private double x;
    private double y;
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

Certainly, the hard-liners are correct when it comes to public classes: if a class is accessible outside its package, provide accessor methods to preserve the flexibility to change the class’s internal representation. If a public class exposes（暴露） its data fields, all hope of changing its representation is lost because client code can be distributed far and wide.

**如果把field暴露出去并且被人用了就很麻烦。**

However, if a class is package-private or is a private nested class, there is nothing inherently（固有的，本质的） wrong with exposing its data fields—assuming they do an adequate（充足） job of describing the abstraction provided by the class. 

This approach generates less visual clutter than the accessor-method approach, both in the class definition and in the client code that uses it. While the client code is tied to the class’s internal representation, this code is confined（密闭） to the package containing the class. If a change in representation becomes desirable, you can make the change without touching any code outside the package. In the case of a private nested class, the scope of the change is further restricted to the enclosing class.

**如果是package-private class或者private nested class，上面这样用就没有什么问题。并且看起来用起来都更清爽**

Several classes in the Java platform libraries violate the advice that public classes should not expose fields directly. Prominent（突出的） examples include the Point and Dimension classes in the java.awt package. Rather than examples to be emulated, these classes should be regarded as cautionary tales（警示故事）.As described in Item 67, the decision to expose the internals of the Dimension class resulted in a serious performance problem that is still with us today.

While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants. For example, this class guarantees that each instance represents a valid time:

**如果field是不可变的，危害将会大大减小。**

```
// Public class with exposed immutable fields - questionable
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    } ... // Remainder omitted
}
```

In summary, public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields.It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.



---
**[Back to contents of the chapter（返回章节目录）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Introduction.md)**
- **Previous Item（上一条目）：[Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)**
- **Next Item（下一条目）：[Item 17: Minimize mutability（减少可变性）](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)**
