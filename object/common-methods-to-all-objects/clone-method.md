# clone() 方法

`Cloneable` 接口的目的是作为对象的一个 mixin 接口，表明这样的对象允许克隆。遗憾的是，它并没有成功地达到这个目的。其主要的缺陷在于，它缺少一个 `clone()`。由于 `Object#clone()` 是受保护的，所以不能仅仅因为一个对象实现了 `Cloneable`，就可以调用它的 `clone()`；即使是借助于反射，也可能会失败。

`Cloneable` 决定了 `Object` 中受保护的 `clone()` 实现的行为：如果一个类实现了 `Cloneable`，`Object#clone()` 就返回该对象的逐域拷贝，否则就会抛出 `CloneNotSupportedExecption`。这是接口的一种极端非典型的用法，也不值得效仿。

`Object#clone()` 的通用约定是非常弱的。下面是来自 `java.lang.Object` 规范中的约定内容[JavaSE6]：

>创建和返回该对象的一个拷贝。这个“拷贝”的精确含义取决于该对象的类。一般的含义是，对于任何对象 `x`，
>
>*   表达式 `x.clone() != x` 将会是 `true`；
>
>*   表达式 `x.clone().getClass() == x.getClass()` 将会是 `true`
>
>但以上都不会绝对的要求。
>
>虽然通常情况下，表达式 `x.clone().equals(x)` 将会是 `true`，但是这也不是一个绝对的要求。
>
>拷贝对象往往导致创建它的类的一个新实例，但它同时也会要求拷贝内部的数据结构。这个过程中没有调用构造器。

简而言之，假设你希望在一个类中实现 `Cloneable`，那么应该覆盖 `clone()`，并将它声明为公有的，以及忽略 `CloneNotSupportedException` 异常（因为不会抛出受检异常的方法使用起来更加轻松）。此方法首先调用 `super.clone()` 返回一个正确类的实例，然后修正任何需要修正的域。

注意，前提条件是该类的所有超类都提供了行为良好的 `clone()`。这样的话，调用 `super.clone()` 最终会调用 `Object#clone()`，从而创建出正确类的实例。这种机制大体上类似于自动的构造器调用链，只不过它不是强制要求的。

对于类中不同的域，需要执行对应的修正，具体如下：

*   如果域是基本类型，或者是指向不可变对象的引用，那么不需要进行任何修正。
	
	注意也有例外情况，代表序列号或其它唯一 ID 值的域，或者代表对象创建时间的域，不管这些域是基本类型还是不可变的，它们也都需要被修正。
		
*   如果域引用了可变的对象，那么还需要拷贝这个域的数据，可以递归地调用这个域的 `clone()`。
	
	注意，如果引用可变对象的域是 `final` 的，上述方案就不能正常工作。因为 `clone()` 是被禁止给 `final` 域赋新值的。这是个根本的问题：clone 架构与引用可变对象的 `final` 域的正常用法是不相兼容的，除非在原始对象和克隆对象之间可以安全地共享此可变对象。为了使类成为可克隆的，可能有必要从某些域中去掉 `final` 修饰符。
		
*   对于包含内部“深层结构”的复杂对象（例如散列表），则需要单独地拷贝并组成该对象的结构。这些内部拷贝操作的实现有两种方案：

	*   可以通过递归地地调用 `clone()` 来完成。如果有栈空间溢出的风险，也可以考虑使用迭代方式来取代递归方式。

	*   先把结果对象中的所有域都设置成空白状态，然后调用高层的方法来重新产生对象的状态。

注意，`clone()` 不应该在构造的过程中，调用新对象中任何非 `final` 的方法。如果 `clone()` 调用了一个被覆盖的方法，那么在该方法所在的子类有机会修正它在克隆对象中的状态之前，该方法就会先被执行，这样很有可能会导致克隆对象和原始对象之间的不一致。

注意，如果你决定用线程安全的类实现 `Cloneable` 接口，要记得它的 `clone()` 必须得到很好的同步，就像任何其它方法一样。

假设你是在专门为了继承而设计的类中覆盖的 `clone()`，则应该模拟 `Object#clone()` 的行为：它应该被声明为 `protected`，抛出 `CloneNotSupportedException` 异常，并且该类不应该实现 `Cloneable` 接口。这样做可以使子类具有实现或不实现 `Cloneable` 接口的自由，就仿佛它们直接扩展了 `Object` 一样。

由于 `Cloneable` 具有这么多缺点，其他的接口都不应该扩展这个接口，为了继承而设计的类也不应该实现这个接口；有些专家级的程序员干脆从来不去覆盖 `clone()`，也从来不去调用它（除非拷贝数组）。
	
## 应用示例

### 克隆简单对象

如果每个域都是基本类型，或者是指向不可变对象的引用，那么返回的对象正是你所需要的对象。

```java
public final class PhoneNumber implements Cloneable {

	private static void rangeCheck(int arg, int max, String name) {
		if (arg < 0 || arg > max) {
			throw new IllegalArgumentException(name + ": " + arg);
		}
	}

	public static void main(String[] args) {
		PhoneNumber pn = new PhoneNumber(707, 867, 5309);
		Map<PhoneNumber, String> m = new HashMap<PhoneNumber, String>();
		m.put(pn, "Jenny");
		System.out.println(m.get(pn.clone()));
	}

	private final short areaCode;
	private final short prefix;
	private final short lineNumber;

	public PhoneNumber(int areaCode, int prefix, int lineNumber) {
		rangeCheck(areaCode, 999, "area code");
		rangeCheck(prefix, 999, "prefix");
		rangeCheck(lineNumber, 9999, "line number");
		this.areaCode = (short) areaCode;
		this.prefix = (short) prefix;
		this.lineNumber = (short) lineNumber;
	}

	@Override
	public PhoneNumber clone() {
		try {
			return (PhoneNumber) super.clone();
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();// 不会发生
		}
	}
	
	@Override
	public boolean equals(Object o) {
		if (o == this)
			return true;
		if (!(o instanceof PhoneNumber))
			return false;
		PhoneNumber pn = (PhoneNumber) o;
		return pn.lineNumber == lineNumber && pn.prefix == prefix
				&& pn.areaCode == areaCode;
	}

	@Override
	public int hashCode() {
		int result = 17;
		result = 31 * result + areaCode;
		result = 31 * result + prefix;
		result = 31 * result + lineNumber;
		return result;
	}
}
```
	
注意，上述 `clone()` 返回的是 `PhoneNumber`，而不是返回 `Object`。从 Java 1.5 开始引入了协变返回类型（covariant return type）作为泛型；换句话说，覆盖方法的返回类型可以是被覆盖方法的返回类型的子类型。

如果域引用了可变的对象，我们还需要拷贝这个域的数据，可以递归地调用这个域的 `clone()`。

```
public class Stack implements Cloneable {

	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	// To see that clone works, call with several command line arguments
	public static void main(String[] args) {
		args = new String[] { "a", "b", "c" };
		Stack stack = new Stack();
		for (String arg : args)
			stack.push(arg);
		Stack copy = stack.clone();
		while (!stack.isEmpty())
			System.out.print(stack.pop() + " ");
		System.out.println();
		while (!copy.isEmpty())
			System.out.print(copy.pop() + " ");
	}

	private Object[] elements;
	private int size = 0;

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

	public boolean isEmpty() {
		return size == 0;
	}

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

	// Ensure space for at least one more element.
	private void ensureCapacity() {
		if (elements.length == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}
```

注意，我们不一定要将 `elements.clone()` 的结果转换为 `Object[]`。从 Java 1.5 开始，在数组上调用 `clone()` 返回的数组，其编译时类型与被克隆的数组的类型相同。

### 克隆复杂对象

假设你正在为一个散列表编写 `clone()`，它的内部数据包含一个散列桶数组，每个散列桶都指向键值对链表的第一个项；如果桶是空的，则为 `null`。出于性能方面的考虑，该类实现了自己的轻量级单向链表，而没有使用 Java 内部的 `LinkedList`。

你不能仅仅递归地克隆这个散列桶数组。虽然被克隆对象有它自己的散列桶数组，但是这个数组引用的链表与原始对象是一样的。为了修正这个问题，有两种解决方案：

*   单独地拷贝并组成每个桶的链表。

	在这个例子中，`clone()` 分配了一个新的散列桶数组，并且遍历原始的散列桶数组，对每一个非空散列桶进行深度拷贝。键值对对象的深度拷贝方法递归地或迭代地调用自身，以便拷贝整个链表（它是链表的头节点）。
	
	注意，如果散列桶不是很长的话，采用递归方式的深度拷贝方法也会工作得很好；如果链表比较长，由于针对列表中的每个元素，它都要消耗一段栈空间，这很容易导致栈溢出。为了避免这种情况，可以使用迭代方式来取代递归方式。

*   先调用 `super.clone()`，然后把结果对象中的所有域都设置成空白状态（virgin state），然后调用高层的方法来重新产生对象的状态。这种做法往往会产生一个简单单、合理且相当优美的 `clone()`，但是它运行起来通常没有“直接操作对象及其克隆对象的内部状态的 `clone()`”快。

	在这个散列表的例子中，散列桶将被初始化为一个新的散列桶数组，然后对于正在被克隆的散列表中的每一个键值对映射，都调用 `put(key, value)`。当然，这个 `put(key, value)` 必须是 `final` 的；当然，如果是私有的的也可以（隐式 `final`）。

```java
public class HashTable implements Cloneable {
	private Entry[] buckets= ...;

	// Broken - results in shared internal state!
	/*@Override
	public HashTable clone() {
		try {
			HashTable result = (HashTable)super.clone();
			result.buckets = buckets.clone();
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}*/

	@Override
	public HashTable clone() {
		try {
			HashTable result = (HashTable) super.clone();
			result.buckets = new Entry[buckets.length];
			for (int i = 0; i < buckets.length; i++) {
				if (buckets[i] != null)
					result.buckets[i] = buckets[i].deepClone();
			}
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}

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
		/*Entry deepClone() {
			return new Entry(key, value, (next == null ? null : next.deepClone()));
			
		}*/

		// Interatively copy the linked list headed by this Enty
		Entry deepClone() {
			Entry result = new Entry(key, value, next);
			for (Entry p = result; p.next != null; p = p.next)
				p.next = new Entry(p.next.key, p.next.value, p.next.next);
			return result;

		}
	}
}
```

## 拷贝构造器和拷贝工厂

另一个实现对象拷贝的好办法是提供一个拷贝构造器（copy constructor）或拷贝工厂（copy factory）。

*   拷贝构造器只是一个构造器，它唯一的参数是包含该构造器的类：

	```java
	public Yum(Yum yum);
	```
	
*   拷贝工厂是类似于拷贝构造器的静态工厂：

	```java
	public static Yum newInstance(Yum yum);
	```
	
拷贝构造器和拷贝工厂，都比 `Cloneable/clone()` 具有更多的优势：

*   它们不依赖于某一种很有风险的、语言之外的对象创建机制；

*   它们不要求遵守尚未制定好文档的规范；

*   它们不会与 `final` 域的正常使用发生冲突；

*   它们不会抛出不必要的受检异常；

*   它们不需要进行类型转换。

虽然你不可能把拷贝构造器或拷贝工厂放到接口中，但是由于 `Cloneable` 接口缺少一个公有的 `clone()`，所以它也没有提供一个接口该有的功能。因此，使用拷贝构造器或者拷贝工厂来代替 `clone()` 时，并没有放弃接口的功能特性。

更进一步，拷贝构造器或拷贝工厂可以带一个参数，参数类型是通过该类实现的接口。例如，按照惯例，所有通用集合实现类都提供了一个拷贝构造器，它的参数类型为 `Collection` 或 `Map`。基于接口的拷贝构造器和拷贝工厂（更准确的叫法是转换构造器（conversion constructor）和转换工厂（conversion factory）），允许客户选择拷贝的实现类型，而不是强迫客户接受原始的实现类型。例如，假设你有一个 `HashSet`，并且希望把它拷贝成一个 `TreeSet`。`clone()` 无法提供这样的功能，但是用转换构造器却很容易实现：`new TreeSet(s)`。