# compareTo() 方法

`compareTo()` 是 `Comparable` 接口提供的唯一方法。 `compareTo()` 不仅允许进行简单的等同性比较，而且允许执行顺序比较。除此之外，它与 `Object` 的 `equals()` 具有相似的特征，但是它还是个泛型。

一旦类实现了 `Comparable` 接口，它就可以跟许多泛型算法以及依赖于该接口的集合实现进行协作。事实上，Java 平台类库中的所有值类（value classes）都实现了 `Comparable` 接口。如果你正在编写一个值类，它具有非常明显的内在排序关系，比如按字母顺序、按数值顺序或者按年代顺序，那你就应该坚决考虑实现这个接口。

如果违反了 compareTo 的通用约定，就会导致该类不能与依赖于比较关系的类一起正常运作。依赖于比较关系的类包括有序集合 `TreeSet` 和 `TreeMap`，以及工具类 `Collections` 和 `Arrays`，它们内部包含有搜索和排序算法。

例如，为实现 `Comparable` 接口的对象数组进行排序：

```java
Arrays.sort(a);
```

对存储在集合中的 `Comparable` 对象进行搜索、计算极限值以及自动维护也同样简单。例如，下面的程序依赖于 `String` 实现了 `Comparable` 接口，它去掉了命令行参数列表中的重负参数，并按字母顺序打印出来：

```java
public class WordList {
	public static void main(String[] args) {
		Set<String> s = new TreeSet<String>();
		Collections.addAll(s, args);
		System.out.println(s);
	}
}
```

## compareTo 约定

`compareTo()` 的通用约定与 `equals()` 的相似：

将这个对象与指定的对象进行比较。当该对象小于、等于、大于指定对象的时候，分别返回一个负整数、零或者正整数。如果由于指定对象的类型而无法与该对象进行比较，则抛出 `ClassCastException`。

在下面的说明中，符号 `sgn`（表达式）表示数学中的 `signum` 函数，它根据表达式的值为负值、零和正值，分别返回-1、0或1。

*   对称性：所有的 `x` 和 `y` 都满足 `sgn(x.compareTo(y))==-sgn(y.compareTo(x))`。

*   传递性：`(x.compareTo(y) > 0) && (y.compareTo(z) > 0 )` 暗示着 `x.compareTo(z) > 0`。

	传递性说明的是，如果一个对象大于第二个对象，并且第二个对象又大于第三个对象，那么第一个对象一定大于第三个对象。

*   `x.compareTo(y) == 0` 暗示着所有的 `z` 都满足 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`。

	该条说明的是，在比较时被认为相等的所有对象，它们在跟别的对象比较时一定会产生相同的结果。
	
*   强烈建议 `(x.compareTo(y) == 0) == (x.equals(y))`，但也并非绝对必要。

	该条说明的是，`compareTo()` 所施加的等同性测试，在通常情况下应该返回与 `equals()` 同样的结果。

	一般来说，任何实现了 `Comparable` 接口的类，若违反了这个条件，都应该明确予以说明。推荐使用这样的说法：“注意，该类具有内存的排序功能，但是与 equals 不一致”。
	
前三条的一个直接结果是，由 `compareTo()` 施加的等同性测试，也一定遵守相同于 `equals()` 约定所施加的限制条件：自反性、对称性和传递性。因此，下面的告诫也同样适用：无法在用新的值组件扩展可实例化的类时，同时保持 compareTo 预定，除非愿意放弃面向对象的抽象优势。针对 equals 的权益之计也同样适用于 compareTo，如果你想为一个实现了 `Comparable` 的类增加值组件，请不要扩展这个类；而是要编写一个不相关的类，其中包含第一个类的实例。然后提供了一个“视图（view）”方法返回这个实例。这样既可以让你自由地在第二个类上实现 `compareTo()`，同时也允许它的客户端在必要的时候，把第二个类的实例视同第一个类的实例。

第四条是一个强烈的建议，而不是真正的规则。如果一个类的 `compareTo()` 施加了一个与 `equals()` 不一致的顺序关系，它仍然能正常工作；但是，如果一个有序集合（sorted collection）包含了该类的元素，这个集合就可能无法遵守相应集合接口（`Collection`、`Set` 或 `Map`）的通用约定。这是因为，对于这些接口的通用约定是按照 `equals()` 来定义的，但是有序集合使用了由 `compareTo()` 而不是 `equals()` 所施加的等同性测试。尽管这不会造成灾难性的后果，但是应该有所了解。

例如，`BigDecimal` 类的 `compareTo()` 与 `equals()` 不一致。当你往一个 `Set` 中添加 `BigDecimal` 元素时，`HashSet` 会使用 `equals()` 来测试等同性；而 `TreeSet` 会使用 `compareTo()` 来测试等同性。这可能会产生不同的结果：

```java
// 在往 Set 中添加元素时， HashSet 使用 equals() 来测试等同性。
Set<BigDecimal> s1 = new HashSet<>();
s1.add(new BigDecimal("1.0"));
s1.add(new BigDecimal("1.00"));
System.out.println(s1);// [1.0, 1.00]

// 在往 Set 中添加元素时， TreeSet 使用 compareTo() 来测试等同性。
Set<BigDecimal> s2 = new TreeSet<>();
s2.add(new BigDecimal("1.0"));
s2.add(new BigDecimal("1.00"));
System.out.println(s2);// [1.0]
```

## 实现 compareTo()

编写 `CompareTo()` 与编写 `equals()` 非常相似。最重大的差别有几处：`Comparable` 接口是参数化的，而且 `compareTo()` 是静态的类型，因此不必进行类型检查，也不必对它的参数进行类型转换。如果参数为 `null`，这个调用应该抛出 `NullPointerException`。

`compareTo()` 中域的比较是顺序的比较，而不是等同性的比较。

*   比较对象引用域可以是通过递归地调用 `compareTo()` 来实现。

*   如果一个域并没有实现 `Comparable` 接口，或者你需要使用一个非标准的比较关系，就可以使用一个显式的 `Comparator` 来代替。

*   比较整数型基本类型的域，可以使用关系操作符 `<` 和 `>`。

*   比较浮点型的域，可以使用 `Double.compare` 或 `Float.compare`，而不用关系运算符，当应用到浮点值时，它们没有遵守 compareTo 的通用约定。

*   对于数组域，则要把以上指导原则应用到每个元素上。

如果一个类有多个关键域，那么你必须从关键的域喀什，逐步进行到所有的重要域。如果某个域的比较结果产生了非零的结果，则整个比较操作结束，并返回该结果。如果最关键的域是相等的，则进一步比较次最关键的域，以此类推。如果所有的域都是相等的，则对象就是相等的，并返回零。

```java
@Override
public int compareTo(PhoneNumber pn) {
	// 比较 areaCode
	if (areaCode < pn.areaCode)
		return -1;
	if (areaCode > pn.areaCode)
		return 1;

	// areaCode 相等，比较 prefix
	if (prefix < pn.prefix)
		return -1;
	if (prefix > pn.prefix)
		return 1;

	// areaCode 和 prefix 相等，比较 lineNumber
	if (lineNumber < pn.lineNumber)
		return -1;
	if (lineNumber > pn.lineNumber)
		return 1;

	return 0; // 所有域都相等
}
```