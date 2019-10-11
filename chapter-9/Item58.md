# 优先选择for-each循环而非传统的for循环

正如在条款45中所讨论的，一些任务最好通过`Stream`来完成，而另一些任务则通过迭代来完成。这是一个传统的`for`循环去遍历一个集合：

```java
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
	Element e = i.next();
	... // Do something with e
}
```

这是一个传统的`for`循环去遍历一个数组：

```java
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
	... // Do something with a[i]
}
```

这些用法好于`while`循环（条款57），但是他们并不完美。迭代器和索引变量都是杂乱的——你需要的仅仅是元素。而且，他们会增加出错的机会。每次循环，迭代器会出现3次，索引变量会出现4次。这些会增加你使用到错误变量的机会。如果你这样做，无法保证编译器可以捕获到问题。最后，这两个循环完全不同，对容器类型的不必要注意并且增加了更改该类型的麻烦。

`for-each`循环（官方叫法，增强的`for`语句）解决了所有这些问题。它摆脱了杂乱和出错的机会，通过隐藏迭代器或者索引变量。最终的用法同样地适用于集合和数组，简化了遍历容器时选择元素类型的过程。

```java
// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
	... // Do something with e
}
```

当你看到冒号时，可以将其读作“in”。因此，上面的循环读作“遍历元素中的每一个元素e”。使用`for-each`循环不会造成性能损失，即使对于数组也是如此：它们生成的代码本质上与你手写的代码相同。

当涉及到嵌套迭代时，`for-each`循环相对于传统`for`循环的优势甚至更大。下面是人们在进行嵌套迭代时经常犯的一个错误：

```java
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, 
           NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
deck.add(new Card(i.next(), j.next()));
```
