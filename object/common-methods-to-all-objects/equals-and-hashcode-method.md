# equals() 和 hashCode() 方法

简而言之，覆盖 `equals()` 时请遵守通用约定；覆盖 `equals()` 时总是覆盖 `hashCode()`。

## equals()

首先，`Object` 的 `equals()` 只是比较对象的地址，即检查两个对象引用是否指向同一个对象。这也是所有类默认拥有的 `equals()`。

如果类具有自己特定的“逻辑相等”概念，而且超类还没有覆盖 `equals()` 以实现期望的行为，这时我们就需要覆盖 `equals()`。这样我们在使用 `equals()` 来比较对象的引用时，就可以知道它们在逻辑上是否相等，而不关心它们是否指向同一个对象，而且这样做也使得这个类的实例可以被用作 `Set` 的元素或 `Map` 的键。

有一种“值类”不需要覆盖 `equals()` ，即用实例受控确保“每个值至多只存在一个对象”的类，例如枚举类型。对于这样的类而言，逻辑相等与对象等同是一回事，因此 `Object` 的 `equals()` 等同于逻辑意义上的 `equals()`。

### 不覆盖 equals() 的情况

如果满足以下任何一个条件，那么就不应该覆盖 `equals()`：

*   类的每个实例本质上都是唯一的。

	对于代表活动实体而不是值的类（例如 `Thread`）来说，`Object` 的 `equals()` 正是正确的行为。
	
*   不关心类是否提供了“逻辑相等（logical equality）”的测试功能。

	例如，Java 设计者认为 `java.util.Random` 类并不需要提供逻辑相等的功能，例如检查两个 `Random` 实例是否产生相同的随机数序列，所以从 `Object` 继承得到的 `equals()` 已经足够了。
	
*   超类已经覆盖了 `equals()`，从超类继承过来的行为对于子类也是合适的。

	例如，大多数的 `Set` 实现都从 `AbstractSet` 继承 `equals()`。
	
### equals 约定

在覆盖 `equals()` 时，你必须要遵守它的通用约定。下面是约定的内容，来自 `Object` 的规范[JavaSE6]：

1.  自反性(reflexive)：对任意非 `null` 的引用值 `x`，`x.equals(x)` 必须返回 `true`。

	自反性实际上说明对象必须等于自身。该特性通常会自动满足。

2.  对称性（symmetric）：对任意非 `null` 的引用值 `x` 和 `y`，如果 `y.equals(x)` 返回 `true`，`x.equals(y)` 也必须返回 `true`。

	对称性说明任何两个对象对于“它们是否相等”的问题都必须保持一致。

3.  传递性（transitivity）：对任意非 `null` 的引用值 `x`、`y`、`z`，如果有 `x.equals(y)` 返回 `true`，且 `y.equals(z)` 返回 `true`，那么 `x.equals(z)` 也必须返回 `true`。

	传递性说明的是，如果一个对象等于第二个对象，并且第二个对象又等于第三个对象，那么第一个对象一定等于第三个对象。
	
	我们无法在扩展可实例化的类（非 `abstract` 类）的同时，增加新的值组件，同时又保留 `equals()` 约定。如果在 `equals()` 中使用 `getClass()` 测试替代 `instanceof` 测试，貌似可以实现这个目标，但是这样却不适用于子类型的比较，这违反了*里氏替换原则（Liskov substitution principle）*。要实现上述目标，除非使用组合的方式，虽然组合优于继承，但也意味着放弃了面向对象的抽象所带来的优势。
	
	注意，你可以在一个 `abstract` 类的子类中增加新的值组件，这不会违反 `equals()` 约定。只要不能直接创建超类的实例，就不会违反 `equals()` 约定。

4.  一致性（consistency）：对任意非 `null` 的引用值 `x` 和 `y`，如果对象中用于 `equals()` 比较的信息没有改变，那么无论调用 `x.equals(y)` 多少次，返回的结果应该保持一致，要么一直是 `true`，要么一直是 `false`。

	一致性说明的是，如果两个对象相等，它们就必须始终保持相等，除非它们中有一个对象（或者两个）被修改了。换句话说，可变对象在不同的时候可以与不同的对象相等，而不可变对象则不会这样；对于不可变对象来说，相等的对象永远相等，不等的对象永远不等。当你在写一个类的时候，应该仔细考虑它是否应该是不可变的。

5.  对任何非 `null` 的引用值 `x`，`x.equals(null)` 一定返回 `false`。

	该约定说明所有的对象都必须不等于 `null`。`o.equlas(null)` 调用当然不会返回 `true`，但是可能会抛出 `NullPointerException`。要预防这种情况，可以执行一个显式的 `null` 检查。其实，为了测试参数的等同性，必须使用 `instanceof` 操作符检查参数是否为正确的类型。如果 `instanceof` 的第一个参数为 `null` 就会返回 `false`。因此，如果把 `null` 传递给 `equals()`，类型检查就会返回 `false`，所以不需要单独的 `null` 检查。

### 覆盖 equals() 的最佳实践

下面是实现高质量的 `equals()` 的最佳实践：

1.  使用 `==` 操作符检查“参数是否为这个对象的引用”。

	这只不过是一种性能优化，如果比较操作有可能很昂贵，就值得这么做。
	
2.  使用 `instanceof` 操作符检查“参数是否为正确的类型”。

	一般来说，所谓“正确的类型”是指 `equals()` 所在的那个类。有些情况下，是指该类所实现的某个接口。如果类实现的接口改进了 `equals()` 约定，允许在实现了该接口的类之间进行比较，那么就使用接口。集合接口，如 `Set`、`List`、`Map` 和 `Map.Entry` 具有这样的特性。
	
3.  把参数转换成正确的类型。

	因为转换之前进行过 `instanceof` 测试，所以确保会成功。
	
4.  对于该类中的每个“关键（significant）”域，检查参数中的域是否与该对象中对应的域相匹配。

	* 对于对象引用，可以递归的调用 `equals()`；
	
	* 对于非 `float` 或 `double` 的基本类型，可以使用 `==` 操作符进行比较；
	
	* 对于 `float` 类型，可以使用 `Float.compare()`；对于 `double` 类型，则使用 `Double.compare()`。
	
	* 对于数组，则要把以上元素递归地应用到每个元素上。如果数组中的每个元素都很重要，可以使用 `Arrays.equals()`；
	
	域的比较顺序可能会影响到 `equals()` 的性能。为了获得最佳的性能，应该最先比较最有可能不一致的域，或者是开销最低的域，最理想的情况是两个条件同时满足的域。
	
	不应该比较那些不属于对象逻辑状态的域，例如用于同步操作的 `Lock` 域。也不需要比较冗余域（redundant field），因为这些冗余域可以由关键域计算获得，但是如果这样做有可能提高 `equals()` 的性能的话就另当别论了。
	
5.  当你编写完成 `equals()` 后，应该问自己三个问题：它是否是对称的、传递的、一致的？

## hashCode()

在每个覆盖了 `equals()` 的类中，也必须覆盖 `hashCode()`。如果不这样做的话，就会违反 hashCode 的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作。这样的集合包括 `HashMap`、`HashSet`、`LinkedHashMap` 和 `LinkedHashSet`。

### hashCode 约定

下面是约定的内容，摘自 `Object` 规范[JavaSE6]：

1.  在应用程序的执行期间，只要对象的 `equals()` 比较操作所用到的信息没有被修改，那么无论对该对象调用多少次 `hashCode()`，都必须返回相同的整数。在同一个应用程序的多次执行过程中，每次执行所返回的整数可以不一样。

2.  如果两个对象根据 `equals()` 比较是相等的，那么调用这两个对象的 `hashCode()` 必须产生相同的整数结果。

3.  如果两个对象根据 `equals()` 比较是不相等的，那么调用这两个对象的 `hashCode()` 也不一定要产生不同的整数结果。但是程序员应该知道，给不相等的对象产生不同的散列码，有可能提高散列表（hash table）的性能。

如果不覆盖 `hashCode()`，就违反了以上约定的第二条。

### 覆盖 hashCode() 的最佳实践

一个好的散列函数通常倾向于“为不相等的对象产生不相等的散列码”。这正是 hashCode 约定中第三条的含义。理想情况下，散列函数应该将集合中不相等的实例均匀地分布到所有可能的散列值上。

下面给出了一种简单的解决方案：

1.  把某个非零的常数值，比如说17，赋予一个名为 `result` 的 `int` 型变量。

2.  为对象中每个关键域 `f`（即 `equals()` 涉及到的每个域），计算 `int` 型的散列码 `c`：

	域类型 | 计算
	------ | ----
	`boolean` | `c = (f ? 0 : 1);`
	`byte`、`char`、`short` 或 `int` | `c = (int)f;`
	`long` | `c = (int)(f ^ (f >>> 32));`
	`float` | `c = Float.floatToIntBits(f);`
	`double` | `long l = Double.doubleToLongBits(f);`<br/>`c = (int)(l ^ (l >>> 32));`
	`Object`（其 `equals()` 调用了这个域的 `equals()`） | `c = f.hashCode();`
	数组 | 对每个元素，递归地应用上述规则。如果数组中的每个元素都很重要，可以使用 `Arrays.hashCode()`。

3.  把计算得到的散列码 `c` 合并到 `result` 中。

	```java
	result = 31*result + c;
	```
	
4.  返回 `result`。

5.  检查 `hashCode()` 最后生成的结果，确保相等的对象具有相等的散列码。

在散列码的计算中，必须排除 `equals()` 比较计算中没有用到的任何域，否则很有可能违反 hashCode 约定的第二条。

在步骤1中用到了一个非零的初始值。因此即便某些初始域的散列值为0，也会影响到散列码。值17是任选的。

步骤2的乘法部分使得散列值依赖于域的顺序。如果一个类包含多个相似的域，这样的乘法运算就会产生一个更好的散列函数。例如，如果 `String` 散列函数忽略了这个乘法部分，那么只是字母顺序不同的所有字符串都会有相同的散列码。之所以选择31，是因为它是奇素数。如果乘数是偶数，并且乘法溢出的话，信息就会丢失，因为与2相乘等价于移位运算。使用素数的好处并不明显，但是习惯上都使用素数来计算散列结果。31有个很好的特性，即使用移位和减法来代替乘法，可以得到更好的性能：31 * i == (i << 5) - i。现代的 VM 可以自动完成这种优化。

如果一个类是不可变的，并且计算散列码的开销也比较大，就应该考虑把散列码缓存在对象内部，而不是每次请求的时候都重新计算散列码。一般可以选择延迟初始化散列码，使得在第一次调用 `hashCode()` 时才初始化。参考 `String` 的 `hashCode()`：

```java
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
	
public int hashCode() {
	int h = hash;
	if (h == 0 && value.length > 0) {
		char val[] = value;

		for (int i = 0; i < value.length; i++) {
			h = 31 * h + val[i];
		}
		hash = h;
	}
	return h;
}
```

## 示例代码

```
public final class PhoneNumber {
	private static void rangeCheck(int arg, int max, String name) {
		if (arg < 0 || arg > max) {
			throw new IllegalArgumentException(name + ": " + arg);
		}
	}

	private final short areaCode;
	private final short prefix;
	private final short lineNumber;

	// 缓存的散列码，延迟初始化。
	private volatile int hashCode;

	public PhoneNumber(short areaCode, short prefix, short lineNumber) {
		rangeCheck(areaCode, 999, "area code");
		rangeCheck(prefix, 999, "prefix");
		rangeCheck(lineNumber, 9999, "line number");
		this.areaCode = areaCode;
		this.prefix = prefix;
		this.lineNumber = lineNumber;
	}

	@Override
	public int hashCode() {
		int result = hashCode;
		if (result == 0) {
			final int prime = 31;
			result = 17;
			result = prime * result + areaCode;
			result = prime * result + prefix;
			result = prime * result + lineNumber;
			hashCode = result;
		}
		return result;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (!(obj instanceof PhoneNumber))
			return false;
		PhoneNumber other = (PhoneNumber) obj;
		return (areaCode == other.areaCode)//
				&& (prefix == other.prefix)//
				&& (lineNumber == other.lineNumber);
	}

}
```
