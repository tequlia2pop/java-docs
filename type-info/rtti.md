# 运行时类型信息（RTTI）

RTTI，全称是 Run-Time Type Information，即运行时类型信息。运行时类型信息使得你可以在程序运行时发现和使用类型信息。

“传统”的 RTTI 假定我们在编译时已经知道了所有的类型，它包含以下三种形式：

1.  传统的类型转换，如 `(Shape)`，由 RTTI 确保转型的正确性，如果执行了一个错误的转换，就会抛出一个 `ClassCastException` 异常。

	```java
	Shape shape = new Circle();
	Circle circle = (Circle)shape;
	```
	
	在执行类型转换的时候，Java 会执行类型检查。编译器允许自由地做向上转型的赋值操作，因为它知道待转型的类肯定是它的父类的类型。如果要做的是向下转型的话，必须使用显式的类型转换，以便告知编译器你拥有额外的信息，这些信息使你知道该类型是某种特定类型。编译器将检查向下转型是否合理，因此它不允许向下转型到实际上不是待转型类的子类的类型上。这通常称为“类型安全的向下转型”。

2.  代表对象的类型的 `Class` 对象。通过查询 `Class` 对象可以获取运行时所需的信息。

3.  使用关键字 `instanceof` 和 `Class#isInstance()` 可以判断对象是否是某个特定类型的实例。

可以使用 RTTI 把程序组织成一系列的 `switch` 语句，但是这样就损失了多态机制的重要价值。面向对象编程语言的目的是让我们在凡是可以使用的地方都使用多态机制，只在必需的时候使用 RTTI 来发现类型信息。

然而使用多态机制的方法调用，要求我们拥有基类定义的控制权，因为在你扩展程序的时候，可能会发现基类并未包含我们想要的方法。如果基类来自别人的类，或者由别人控制，这时候 RTTI 便是一种解决之道：可以继承一个新类，然后添加你需要的方法。在代码的其他地方，可以检查你自己特定的类，并调用自己的方法。这样做不会破坏多态性以及程序的扩展能力，因为这样添加一个新的类并不需要在程序中搜索 `switch` 语句。但如果在程序主体中添加需要的新特性的代码，就必须使用 RTTI 来检查你的特定的类型。

另外，RTTI 有时能解决多态造成的效率问题。也许你的程序漂亮地运用了多态，但其中某个对象是以极端缺乏效率的方式达到这个目的的。你可以挑出这个类，使用 RTTI，并且为其编写一段特别的代码以提高效率。然而必须要注意，不要太早地关注程序的效率问题。最好首先让程序运作起来，然后再考虑它的速度。

## 为什么需要 RTTI

假定有一个使用了多态的类层次结构的例子。最通用的类型（泛型）是基类 `Shape`，而派生出的具体类有 `Circle`、`Square` 和 `Triangle`。其中 `Shape` 接口动态绑定了 `draw()`。这样客户端程序员可以使用泛化的 `Shape` 引用来调用 `draw()`，同时也能产生正确的行为。这就是多态。

因此，通常会创建一个具体对象，把它向上转型为基类型，并在后面的程序中使用基类型引用：

```java
List<Shape> shapeList = Arrays.asList(new Circle(), new Square(), new Triangle());
for(Shape shape : shapeList)
	shape.draw();
```

在这个例子中，当把 `Shape` 对象放入 `List<Shape>` 的数组时会向上转型。但在向上转型为 `Shape` 的时候也丢失了 `Shape` 对象的具体类型。对于数组而言，它们只是 `Shape` 类的对象。

数组实际上将所有的事物都当做 `Object` 持有。但是当从数组中取出元素时，它会自动将结果转型为 `Shape`。这是 RTTI 的基本使用形式。**因为在 Java 中，所有的类型转换都是在运行时进行类型检查的。这也是 RTTI 名字的含义：在运行时，识别一个对象的类型。**

在这个例子中，RTTI 类型转换并不彻底：`Object` 被转型为 `Shape`，而不是转型为 `Circle`、`Square` 或者 `Triangle`。但是，多态会帮助对象产生正确的行为：	
`Shape` 对象实际执行什么样的代码，是由引用所指向的具体对象 `Circle`、`Square` 或者 `Triangle` 而决定的。

但是，假如你碰到了一个特殊的编程问题——如果能够知道某个泛化引用的确切类型，就可以使用最简单的方式去解决它。例如，可能要用某种方法来旋转列出的所有图形，但想跳过圆形，因为对圆形进行旋转没有意义。使用 RTTI，可以在运行时查询一个对象的确切类型，然后选择或剔除特例。

## Class 对象

`Class` 对象包含了与类有关的信息。实际上，`Class` 对象就是用来创建类的所有“常规”对象的。Java 在运行时使用 `Class` 对象来表示类型信息，并执行其 RTTI。`Class` 类还拥有大量的使用 RTTI 的其他方式。

每个类都有一个 `Class` 对象。每当编写并且编译了一个新类，就会产生一个同名的 `.class` 文件。为了生成这个类的对象，运行这个程序的 Java 虚拟机（JVM）将使用“类加载器”子系统。

类加载器子系统实际上可以包含一条类加载器链，但是只有一个*原生类加载器*，它是 JVM 实现的一部分。原生类加载器加载的是所谓的可信类，包括 Java API 类，它们通常是从本地盘加载的。如果你有特殊需求（例如以某种特殊的方式加载类，以支持 Web 服务器应用，或者在网络中下载类），那么你可以挂接额外的类加载器。

所有的类都是在对其第一次使用时，动态加载到 JVM 中的。当程序创建第一个对类的静态成员的引用时，就会加载这个类。值得说明的是，构造器是隐式的静态方法，所以使用 `new` 操作符可以创建并加载一个类的对象。因此，Java 程序在开始运行前并非完全加载，其各个部分是在必需时才加载的。

类加载器首先检查这个类的 `Class` 对象是否已经加载。如果尚未加载，默认的类加载器就会根据类名查找同名的 `.class` 文件（例如，某个附加类加载器可能会在数据库中查找字节码）。在这个类的字节码被加载时，它们会接受验证，以确保其没有被破坏，并且不包含不良 Java 代码。一旦某个类的 `Class` 对象被载入内存，它就被用来创建这个类的所有对象。

--------------------------------------------------------

如果想在运行时使用类型信息，必须首先获得对恰当的 `Class` 对象的引用。如果你还没有持有某个类型的对象，那就可以使用 `Class.forName()`，并传入该类的全限定名（包含包名）；如果你已经拥有了该类型的对象，那就可以通过该对象的 `getClass()` 来获取 `Class` 引用；或者你还可以使用类字面常量：

*   使用 `Class.forName()`，并传入类的全限定名（包含包名）。

	该方法会返回一个未知类型的 `Class` 对象，可能需要使用转型，而这会产生编译期警告。如果找不到要加载的类，它会抛出 `ClassNotFoundException`。
	
	除了 `forName(String className)` 外，`forName(String name, boolean initialize, ClassLoader loader)` 还可以指定是否进行初始化，以及指定要使用哪个类加载器。
	
*   使用对象的 `getClass()`。例如：`obj.getClass();`。

	该方法属于 `Object` 的一部分，它将返回表示该对象的实际类型的 `Class` 引用。

*   使用类字面常量。例如：`String.class`。

	这样不仅更简单，而且更安全和高效。因为它在编译期就会受到检查，并且根除了对 `forName()` 的调用。我们知道类加载包括加载、链接和初始化三个步骤，当使用“.class”来创建对 `Class` 对象的引用时，不会自动地初始化该 `Class` 对象。初始化有效地实现了尽可能的“惰性”，它被延迟到了对静态方法或者非常数静态域的首次引用时才执行。

`Class` 类包含了很多有用的方法，下面是其中的一部分：

方法 | 说明
---- | ----
`getName()` | 返回全限定的类名。
`getSimpleName()` | 返回不含包名的类名。
`getCanonicalName()` | 返回全限定的类名。
`isInterface()` | 判定指定的 `Class` 对象是否表示一个接口类型。
`getInterfaces()` | 返回指定的 `Class` 对象所包含的接口。
`getSuperclass()` | 返回指定的 `Class` 对象的直接基类。
`getMethods()` | 返回 `Method` 对象的数组。`Method` 对象提供了深层的方法，用以解析其对象所代表的方法，并获取其名字、输入参数以及返回值。
`getConstructors()` | 返回 `Constructor` 对象的数组。`Constructor` 对象提供了深层的方法，用以解析其对象所代表的方法，并获取其名字、输入参数以及返回值。
`newInstance()` | 实现“虚拟构造器”的一种途径。虚拟构造器允许你声明：“我不知道你的确切类型，但是无论如何要正确地创建你自己”。注意，使用 `newInstance()` 来创建的类，必须带有默认的构造器。另外，可以使用反射 API，用任意的构造器来动态地创建类的对象。
`cast()` | 该方法接受参数类型，并将其转换为 `Class` 引用的类型。新的转型语法对于无法使用普通转型的情况显得非常有用，在你编写泛型代码时，如果你存储了 `Class` 引用，并希望以后通过这个引用来执行转型，这种情况就会时有发生。
`asSubClass()` | 将一个类对象转换为更加具体的类型。

下面展示了通过 `Class#getName()` 返回的 Java 类名的表达形式：

```java
String.class.getName();
// returns "java.lang.String"
byte.class.getName();
// returns "byte"
(new Object[3]).getClass().getName();
// returns "[Ljava.lang.Object;"
(new int[3][4][5][6][7][8][9]).getClass().getName();
// returns "[[[[[[[I"
```

### 类字面常量

类字面常量也可以应用于类、接口、数组以及基本数据类型。对于基本数据类型的包装器类，还有一个标准字段 `Type`。`Type` 字段是一个引用，指向对应的基本数据类型的 `Class` 对象，如下所示：

... | ...
--- | ---
boolean.class | Boolean.Type
char.class | Char.Type
byte.class | Byte.Type
short.class | Short.Type
int.class | Integer.Type
long.class | Long.Type
float.class | Float.Type
double.class | Double.Type
void.class | Void.Type

建议使用 ".class" 的形式，以保持与普通类的一致性。

### 泛化的 Class 引用
	
可以使用泛型语法对 `Class` 引用所指向的 `Class` 对象的类型进行限定。

为了在使用泛化的 `Class` 引用时放松限制，可以使用“?”通配符，表示“任何事物”。`Class<?>` 优于平凡的 `Class`。`Class<?>` 的好处是它表示你故意选择了一个非具体的类引用。例如：

```java
Class<?> intClass = int.class;
intClass = double.class;
```

要限定一个 `Class` 引用为某种类型的任何子类型，需要将通配符与 `extends` 关键字结合，创建一个范围。例如：

```java
Class<? extends Number> bounded = int.class;
bounded = double.class;
bounded = Number.class;
// Or anything else derived from Number.
```

要限定一个 `Class` 引用为某种类型的任何父类型，需要将通配符与 `super` 关键字一起使用：

```java
Class<FancyToy> ftClass = FancyToy.class;
// Produces exact type:
FancyToy fancyToy = ftClass.newInstance();
Class<? super FancyToy> up = ftClass.getSuperclass();
// This won't compile:
// Class<Toy> up2 = ftClass.getSuperclass();
// Only produces Object:
Object obj = up.newInstance();
```

## 类型转换前先做检查

使用关键字 `instanceof` 可以判断对象是否是某个特定类型的实例。

```java
if(x instanceof Dog)
	((Dog)x).bark();
```

在将 `x` 转型成一个 `Dog` 前，上面的 `if` 语句会检查对象 `x` 是否从属于 `Dog` 类。进行向下转型前，如果没有其他信息告诉你这个对象是什么类型，那么使用 `instanceof` 是非常重要的，否则会得到一个 `ClassCastException` 异常。

`instanceof` 的限制是只能与命名类型进行比较，而不能与 `Class` 对象作比较。`Class#isInstance()` 方法提供了一种动态测试对象的途径。两种方式的效果是完全等价的。

```java
Pet pet = ...;
if(clazz.isInstance(pet) {
	//...
}
```

**instanceof 与 Class 的等价性**

不管是测试基类还是导出类，`instanceof` 和 `isInstance()` 两种的结果完全相同，`equals()` 和 `==` 的结果也完全一样。但是，`instanceof` 保持了类型的概念，它指的是“你是这个类吗，或者你是这个类的派生类吗”；而 `==` 和 `equals()` 没有考虑继承，它指的是“你是这个确切的类型吗，或者不是”。

## 接口和类型信息
	
接口的一种重要目标就是允许程序员隔离构件，进而降低耦合性。但是通过类型信息，这种耦合性还是会传播出去——接口并非是对解耦的一种无懈可击的保障。下面是一个例子：

```java
Shape s = new Box();
s.draw();
System.out.println(s.getClass().getName());// Box
if (s instanceof Box) {
	Box b = (Box) s;
	// Shape 接口并没有 rotate()，因为有些图形（如圆形）的旋转没有意义
	b.rotate();
}
```

通过使用 RTTI，`s` 是被当作 `Box` 实现的。通过将其转型为 `Box`，我们可以调用不在 `Shape` 中的方法。换句话说，通过将接口转型为具体类型，我们可以调用不在接口中的方法。这是完全合法和可接受的，但是你也许并不想让客户端程序员这么做，因为这会使得他们的代码与你的代码的耦合程度超过你的期望。对于该问题有以下几种解决方案：

1.  直接使用实际的类而不是接口。这样程序员需要自己对自己负责，而这在很多情况下都是合理的。

	```java
	// Shape s = new Box();
	Box s = new Box();
	s.draw();
	s.rotate();
	```

2.  最简单的方式是对实现使用包访问权限，这样在包外部的客户端就不能看到它了。

	```java
	package typeinfo.interfacea;

	public interface A {
	  void f();
	}
	```
	
	```java
	package typeinfo.packageaccess;

	import typeinfo.interfacea.*;
	import static net.mindview.util.Print.*;

	class C implements A {
		@Override
		public void f() {
			print("public C.f()");
		}

		public void g() {
			print("public C.g()");
		}

		void u() {
			print("package C.u()");
		}

		protected void v() {
			print("protected C.v()");
		}

		private void w() {
			print("private C.w()");
		}
	}

	public class HiddenC {
		public static A makeA() {
			return new C();
		}
	}
	```
	
	在调用 `HiddenC` 的 `makeA()` 将产生 `A` 类型的对象。但是，即使你从 `makeA()` 返回的是 `C` 类型，你在包的外部仍旧不能使用 `A` 之外的任何方法，因为你不能在包的外部命名 `C`。

	但是通过反射，仍旧可以到达并调用所有方法，甚至 `private` 方法。即使接口的实现是一个私有内部类，或者匿名类，也无法阻止反射达到并调用那些非公共访问权限的方法。
	
	通常，这些违反访问权限的操作并非世上最糟之事。如果有人使用这样的计数去调用标识为 `private` 或包访问权限的方法（很明显这些访问权限表示这些人不应该调用它们），那么对他们来说，如果你修改了这些方法的某些方面，他们不应该抱怨。另一方面，总是在类中留下后门的这一事实，也许可以使得你能够就觉某些特定类型的问题，但如果不这样做，这些问题将难以后者不可能解决，通常反射带来的好处是不可否认的。