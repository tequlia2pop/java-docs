# 协变、逆变和不变

## 定义

在一门程序设计语言的类型系统中，使用逆变与协变来描述类型转换（type transformation）后的继承关系。

如果 `A`、`B` 表示类型，`f(⋅)` 表示类型转换，`≤` 表示继承关系（比如，`A≤B` 表示 `A` 继承自 `B`），那么有如下定义：

*   协变（covariant）：如果类型转换保持了原有的继承关系，称之为协变。也就是说，当 `A≤B` 时，`f(A)≤f(B)` 成立，那么 `f(⋅)` 是协变的。

*   逆变（contravariant）：如果类型转换逆转了原有的继承关系，称之为逆变。也就是说，当 `A≤B` 时，`f(B)≤f(A)` 成立，那么 `f(⋅)` 是逆变的。

*   不变（invariant），如果上述两种情况均不适用，即类型转换后的两者不存在相互的继承关系，称之为不变。也就是说，当 `A≤B` 时，`f(A)≤f(B)` 和 `f(B)≤f(A)` 均不成立，那么 `f(⋅)` 是不变的。

## 协变数组

Java 数组是协变的（covariant）。例如，`Integer[]` 是 `Number[]` 是子类型。实际上，所有的对象数组都是 `Object[]` 的子类型。 

因为数组是协变的，所以可以将导出类型的数组作为其基类型的数组使用：

```java
Number[] numbers = new Integer[10];
numbers = new Double[20];
Object[] objects = numbers;
```

这看起来和向上转型一样，但是 Bruce Eckel 在 《Thinking in Java》中指出：

>向上转型不适合用在这里。你真正做的是将一个数组赋值给另一个数组。数组的行为是它可以持有其他对象，这里只是因为我们能够向上转型而已，所以很明显，数组对象可以保留它们包含的对象类型的规则。

## 不变泛型

Java 泛型是不变的（invariant）。例如，`List<Integer>` 和 `List<Number>` 是两个截然不同的类型，两者不存在相互的继承关系。

因为泛型是不变的，所以不能将导出类型的泛型作为其基类型的泛型使用：

```java
public void showInvariant(){
	List<Integer> integers = new ArrayList<>();
	// 编译错误：不兼容的类型：
	// List<Number> numbers = integers;
	// doSomething(integers);
}

public void doSomething(List<Number> numbers){
	// ...
}
```

为了能够将导出类型的泛型作为其基类型的泛型使用，Java 提供了**边界通配符类型（bounded wildcard type）**来处理。这包括子类型通配符和超类型通配符，分别可以实现泛型的协变和逆变。

### 实现泛型的协变

假如你想将 `List<Integer>` 类型赋值给一个 `List<Number>` 类型使用，那么需要使得前者成为后者的子类型。由于 `Integer` 是 `Number` 的子类型，所以你需要实现泛型的协变。

可以使用**子类型通配符**实现泛型的协变。具体做法是声明通配符是某个特定类型的子类型，即指定 `<? extends MyClass>`，它表示泛型类型是 `MyClass` 或 `MyClass` 的导出类型。子类型通配符实际上确定了泛型的上界，所以你不能传递一个类型对象到泛型类型中；换句话说，你不能调用那些参数列表中涉及通配符的方法（编译无法通过）。

```java
List<? extends Number> numbers = new ArrayList<Integer>();

// 编译错误：无法添加任意类型的对象。
// numbers.add(new Integer(0));
// numbers.add(new Object());
numbers.add(null);// 合法但无意义
Number n = numbers.get(0);

numbers = Arrays.asList(new Integer(0));
Integer i = (Integer)numbers.get(0);// 无警告
numbers.contains(new Integer(1));// 参数是 Object
numbers.indexOf(new Integer(1));// 参数是 Object
```

通配符引用的是明确的类型，它在这里意味着“某种 `numbers` 引用没有指定的具体类型”。`add()` 将接受一个具有泛型参数类型的参数。因此当你指定一个 `List<? extends Number>` 时，`add()` 的参数变成了 `? extends Number`。从这个描述中，编译器并不能了解这里需要 `Number` 的哪个子类型，因此它不会接受任何类型的 `Number`。

相反，在使用 `contains()` 和 `indexOf()` 时，参数类型是 `Object`，因此不涉及任何通配符，而编译器也将允许这个调用。

### 实现泛型的逆变

假如你想将 `List<Object>` 类型赋值给 `List<Integer>` 类型使用，那么需要使得前者成为后者的子类型。由于 `Object` 是 `Integer` 的父类型，所以你需要实现泛型的逆变。

可以使用**超类型通配符**实现泛型的逆变。具体做法是声明通配符是某个特定类型的基类型，即指定 `<? super MyClass>` 或 `<? super T>`（但是不能声明 `<T super MyClass>`），它表示泛型类型是 `MyClass` 的基类型。超类型通配符实际上确定了泛型的下界，这使得你可以安全地传递一个类型对象到泛型类型中。

```java
List<? super Integer> integers = new ArrayList<Object>();

integers.add(new Integer(0));
// integers.add(new Object());// 错误
```

泛型参数现在是 `List<? super Integer>`，因此这个 `List` 将持有从 `Integer` 导出的某种具体类型，这样就可以安全地将一个 `Integer` 或者从 `Integer` 导出的对象作为参数传递给 `List` 的方法。

### 选择协变还是逆变

可以使用子类型通配符实现泛型的协变，使用超类型通配符实现泛型的逆变。那么应该在什么时候选择协变，什么时候选择逆变呢？换句话说，应该使用哪一种通配符类型呢？

Joshua Bloch 在《Effective Java》中提出：

>为了获得最大限度的灵活性，*要在表示生产者或者消费者的输入参数上使用通配符类型*。如果某个输入参数既是生产者，又是消费者，那么这个通配符类型对你就没有什么好处了；因为你需要的是严格的类型匹配，这是不用任何通配符而得到的。
>
>下面的助记符便于让你记住要使用哪种通配符类型：
>
>**PECS** 表示 **producer-extends**，**consumer-super**。
>
>换句话说，如果参数化类型表示一个 `T` 生产者，就使用 `<? extends T>`；如果它表示一个 `T` 消费者，就使用 `<? super T>`。……PECS 这个助记符突出了使用通配符类型的基本原则，Naftalin 和 Wadler 称之为 **Get and Put Principle**。

注意，`Comparable` 和 `Comparator` 始终是消费者，因此两者使用的始终应该是 `Comparable<? super T>` 优先于 `Comparable<T>`，`Comparator<? super T>` 优先于 `Comparator<T>`。

### 应用示例

Java 7 的 `Collections.copy()` 方法完美地诠释了 PECS：

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
	int srcSize = src.size();
	if (srcSize > dest.size())
		throw new IndexOutOfBoundsException("Source does not fit in dest");

	if (srcSize < COPY_THRESHOLD ||
		(src instanceof RandomAccess && dest instanceof RandomAccess)) {
		for (int i=0; i<srcSize; i++)
			dest.set(i, src.get(i));
	} else {
		ListIterator<? super T> di=dest.listIterator();
		ListIterator<? extends T> si=src.listIterator();
		for (int i=0; i<srcSize; i++) {
			di.next();
			di.set(si.next());
		}
	}
}
```

下面再看几个 PECS 的应用示例：

**示例1**

```java
// 用作 E 生产者的参数的通配符类型
public static <T> Set<T> union(Set<? extends T> s1, Set<? extends T> s2) {
	Set<T> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

`s1` 和 `s2` 两个参数都是 `E` 生产者。

**示例2**

```java
public static <T extends Comparable<? super T>> T max(List<? extends T> list) {
	Iterator<? extends T> i = list.iterator();
	T result = i.next();
	while(i.hasNext()){
		T t = i.next();
		if(t.compareTo(result) > 0)
			result = t;
	}
	return result;
}
```

*   参数 `list` 产生 `T` 实例，因此将类型 `List<T>` 改为 `List<? extends T>`。
	
*   可以将通配符应用到类型参数。`T` 的 `Comparable` 消费 `T` 实例（并产生表示顺序关系的整数）。因为，参数化类型 `Comparable<T>` 被有限制通配符类型 `Comparable<? super T>` 取代。
	
## 协变返回类型
	
从 JavaSE 5 开始添加了协变返回类型，允许在导出类中的覆盖方法可以返回基类方法的返回类型的某种导出类型。这是完全合乎逻辑的事情，导出类方法应该能够返回比它覆盖的基类方法更具体的类型。

```
class Super {
	Number method() {
		return new Number(1);
	}
}

class Sub extends Super {
	@Override 
	Integer method() {
		return new Integer(1);
	}
}
```

## 附录-里氏替换原则

里氏替换原则（Liskov Substitution Principle, LSP）由 Barbara Liskov 于1987年提出，其定义如下：

>所有引用基类（父类）的地方必须能透明地使用其子类的对象。

里氏替换原则包含以下四层含义：

1.  子类完全拥有父类的方法，且具体子类必须实现父类的抽象方法。

2.  子类中可以增加自己的方法。

3.  当子类覆盖或实现父类的方法时，方法的形参要比父类方法的更为宽松。

4.  当子类覆盖或实现父类的方法时，方法的返回值要比父类更为严格。

## 附录-参考资料

*   [Covariance, Invariance and Contravariance explained in plain English?
](https://stackoverflow.com/questions/8481301/covariance-invariance-and-contravariance-explained-in-plain-english)

*   [Difference between super and extends type in Java
](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java)