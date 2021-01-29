# 优先选择接口而不是抽象类

针对于一个类型的多种实现这一问题，Java提供了两种机制，分别是接口与抽象类。由于Java 8中引入了接口的默认方法\[JLS 9.4.3\]，因此这两种机制都可以为实例方法提供实现。一个主要的差别在于，要想实现由抽象类所定义的类型，这个类必须是抽象类的子类。由于Java只允许单继承，因此这种对抽象类的限制严重违背了其作为类型定义的使用。因为Java只允许单继承，所以这种对抽象类的限制严重限制了它们作为类型定义的使用。无论什么类，只要它定义了所有必须的方法并且遵循通用契约，那么它就可以实现接口，无论这个类位于类层次体系中的哪个地方均如此。

**我们可以轻松改造既有类，使之实现新的接口。**你所要做的只不过是添加必要的方法（如果这些方法尚不存在），并在类声明处添加一个`implements`从句即可。比如说，很多既有类在添加到平台后，都被改造以实现`Comparable`、`Iterable`与`Autocloseable`接口。一般来说，既有类无法被改造以继承新的抽象类。如果想让两个类继承同一个抽象类，那么你就需要将这个抽象类放置到类层次较高的位置上，使之成为这两个类的祖先。但遗憾的是，这会对类型体系造成严重的破坏作用，需要强制要求新抽象类的所有后代都要继承它，无论是否恰当均如此。

**接口是定义混合类型（**`mixins`**）的一种理想方式**。大致来说，混合类型指的是这样一种类型：一个类除了其『主要类型』外，还可以实现该类型，表示会提供一些可选的行为。比如说，`Comparable`就是个混合类型接口，可以让一个类声明其实例与其他相互可比较的对象之间是有序的。这种接口之所以叫做混合类型是因为它将可选功能『混合』到了类型的主功能中。抽象类不能用作定义混合类型，原因与之前一样，即他们无法被改造进入到既有的类中：一个类不能有一个以上的双亲，而且在类层次中也并没有恰当的位置来插入混合类型。

**接口允许构造非层次化类型的框架**。类型层次对于组织一些东西来说是非常棒的，不过其他一些东西却无法恰到好处地融入到严格的层次体系中。比如，假设我们有一个代表歌手的接口和另一个代表词曲作者的接口：

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```

在现实生活中，一些歌手也是词曲作者。因为我们使用接口而不是抽象类来定义这些类型，所以完全允许单个类同时实现`Singer`和`Songwriter`。事实上，我们可以定义第3个接口，同时继承`Singer`和`Songwriter`接口，并添加适合于该组合的新方法：

```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

你并不总是需要这种级别的灵活性，但是当你需要时，接口就是救命稻草。另外一种实现方式则是使用一个膨胀的类层次体系，该体系为每个受支持的属性组合都包含单独的类。如果在类型系统中有n个属性，那就需要支持`2^n`个可能的组合。这就是所谓的组合爆炸。膨胀的类层次体系会导致很多类方法仅仅在参数类型上不同，因为类层次体系中没有类型能够捕获到常见的行为。

**接口通过包装类的做法实现了安全、强大的功能增强（条款18）**。如果你使用抽象类来定义类型，那么你将让希望添加功能的程序员除了继承之外别无选择。这样，相比于包装类来说，所得到的类功能更差且更加脆弱。

当一个接口方法相对于其他接口方法而言有明显的实现时，请考虑以默认方法的形式提供一个实现，以此来帮助程序员。比如说，请参阅第104页的`removeIf`方法。如果提供了默认方法，请务必通过`@implSpec Javadoc`标记将其标记为可继承（条款19）。

借助于默认方法所提供的实现辅助是有限制要求的。虽然很多接口指定了Object方法如`equals`和`hashCode`的行为，不过你不能为其提供默认方法。此外，接口不允许包含实例段或是非公有静态成员（除了private static方法）。最后，你不能向自己没有控制权的接口添加默认方法。

不过，你可以通过提供一个抽象的骨架实现类与接口来组合接口与抽象类的优势。接口定义一个类型，可能提供了一些默认方法，而骨架实现类则在主要的接口方法基础上实现了其余非主要的接口方法。继承骨架实现就可以完成接口实现的大部分工作。这叫做模板方法模式\[Gamma95\]。

按照惯例，骨架实现类叫做`AbstractInterface`，其中的`Interface`就是他们所实现的接口名。比如说，集合框架为每个主要的集合接口都提供了一个骨架实现：`AbstractCollection`, `AbstractSet` ,  `AbstractList` ,和`AbstractMap` 。有争议的一点是，叫他们`SkeletalCollection`、`SkeletalSet`、`SkeletalList`和`SkeletalMap`似乎更有意义，不过`Abstract`这个约定现在已经深入人心了。如果设计得当，那么骨架实现（无论是单独的抽象类，抑或仅仅在接口中包含默认方法）可以使得程序员能够非常轻松地提供自己的接口实现。比如说，如下是个静态工厂方法，包含一个基于`AbstractList`的完整、功能完善的List实现：

```java
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

在考虑一个List实现能够做什么时，该示例就是个阐释骨架实现强大之处的最好佐证。有时，这个示例会被叫做适配器\[Gamma95\]，它可以将一个int数组看作是一个Integer实例列表。由于需要在int值与Integer实例之间来回转换（装箱与拆箱），因此其性能并不好。注意到该实现采用了匿名类的形式（条款24）。

骨架实现类的美妙之处在于，它们提供了抽象类的所有实现帮助，而不像抽象类作为类型定义时那样受到严格的约束。对于拥有骨架实现类的接口的大多数实现者来说，继承这个类是显而易见的选择，不过这一点并非强制。如果一个类无法继承骨架实现，那么它永远都可以直接实现接口。这个类依然可以从接口本身的默认方法获益。此外，骨架实现还可以辅助实现者完成任务。实现接口的类可以将对接口方法的调用转发到其所包含的继承了骨架实现的私有内部类。这种技术叫做模拟的多继承，它与条款18所介绍的包装类用法紧密相关。它提供了多继承的很多好处，同时又避免了不少陷阱。

编写骨架实现是个相对来说比较轻松，但是有点单调的事情。首先，研究接口，确定哪些方法是主要方法，其他方法可以根据这些方法来实现。这些主要方法将会成为骨架实现中的抽象方法。接下来，在接口中为所有可以直接根据这些主要方法实现的方法提供默认方法，不过请记住，你不能为`equals`与`hashCode`等Object方法提供默认方法。如果主要方法与默认方法涵盖了整个接口，那就完成了任务，无需骨架实现类了。否则，请编写一个实现该接口的类，这个类要实现其余所有的接口方法。该类可以包含适合于所处理的任务的任何非公有的字段与方法。

举一个简单的例子，考虑`Map.Entry`接口。其显而易见的主要方法有`getKey`、`getValue`与可选的`setValue`。该接口指定了`equals`与`hashCode`的行为，还有一个基于这些主要方法实现的`toString`。由于无法为`Object`方法提供默认实现，因此所有实现都放到了骨架实现类中：

```java
// Skeletal implementation class
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    // Entries in a modifiable map must override this method
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Implements the general contract of Map.Entry.equals
    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map.Entry))
            return false;

        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) 
            && Objects.equals(e.getValue(), getValue());
    }
    // Implements the general contract of Map.Entry.hashCode
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

注意这个骨架实现类不能去实现`Map.Entry`接口，或者它作为一个子接口因为默认方法不允许重写`Object`的方法，如：`equals`，`hashCode`， 和`toString`。

因为骨骼实现类是为继承而设计的，所以你应该遵循条款19中的所有设计和文档准则。为了简洁起见，在前面的示例中省略了文档注释，**但是在一个骨骼实现类中**，**好的文档是绝对必要的**，无论它是包含了接口中的默认方法还是单独的抽象类。

骨架实现的一个小变种是『简单实现』，比如说`AbstractMap.SimpleEntry`。简单实现就像是一个骨架实现，因为它实现了接口，并且针对于继承，不过区别在于它并不是抽象的：它是可用的最简单实现。你可以正常使用它，或是根据需要继承它。

总结一下，接口通常是定义可以拥有多个实现的类型的最佳方式。如果有一个重要接口，那么你绝对应该考虑提供一个与之相关的骨架实现。在可能的范围内，你应该通过接口的默认方法来提供⻣骨架实现，这样接口的所有实现者就都可以使用这些方法了。即便如此，接口本身的限制会要求骨架实现采取抽象类的形式。

