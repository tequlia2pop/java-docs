# 注解介绍

注解（也称为元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据。

注解可以提供用来完整地描述程序所需的信息，而这些信息是无法用 Java 来表达的。因此，我们通过使用注解可以：

*   以将由编译器来测试和验证的格式，存储程序的额外信息。

*   生成描述符文件，甚至是新的类定义，并且有助于减轻编写“样板”代码的负担。

注解还具有以下优点：

*   注解是在源代码级别保存所有的信息，这使得代码更整洁，且便于维护。

*   注解是语言级的概念，它享有编译期的类型检查保护。

通过使用注解，我们可以将这些元数据保存在 Java 源代码中，并利用 annotation API 为自己的注解构造处理工具。通过使用扩展的 annotation API，或外部的字节码工具类库，程序员拥有对源代码以及字节码强大的检查与操作能力。

每当你创建描述性质的类或接口时，一旦其中包含了重复性的工作，那就可以考虑使用注解来简化与自动化该过程。例如在 EJB 中存在许多额外的工作，EJB 3.0 就是使用注解消除了它们。

## Java 内置注解

Java SE5 内置了三种注解：

*   `@Override` 表示当前的方法定义将覆盖超类中的方法。如果方法签名对不上被覆盖的方法，编译器就会发出错误提示。

*   `@Deprecated` 表示废弃和过时的元素。如果使用了注解为它的元素，编译器就会发出警告信息。

*   `@SuppressWarnings` 关闭不当的编译器警告信息。

Java 还另外提供了四种元注解，专门负责新注解的创建。

## 基本语法

注解的定义很像接口的定义。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
  public int id();
  public String description() default "no description";
}
```

*   定义注解时，需要使用一些元注解来定义注解将应用于什么地方、注解在哪一个级别可用，以及其他选项。

*   在注解中，一般都会包含一些元素以表示某些值。当分析处理注解时，程序或工具可以利用这些值。注解的元素看起来就像接口的方法，唯一的区别是你可以为其指定默认值。如果在使用注解时没有给出元素的值，那么注解处理器就会使用此元素的默认值。

	注解的元素在使用时表现为名-值对的形式，并需要置于注解声明之后的括号内。

没有元素的注解称为**标记注解**（marker annotation）。

注解还具有下列特性：

*   注解支持嵌套。具体来说，可以将注解元素声明为另一个注解。

	```java
	@Target(ElementType.FIELD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface SQLInteger {
		String name() default "";

		// 嵌套注解。注意 constraints() 元素的默认值是 @Constraints，
		// 实际上是一个所有元素都为默认值的 @Constraints 注解。
		Constraints constraints() default @Constraints;
		
		// 嵌套注解。也可以指定嵌入的 @Constraints 注解的 unique() 元素为 true，
		// 并以此作为 constraints() 元素的默认值。
		// Constraints constraints() default @Constraints(unique=true);
	}
	```

*   注解不支持继承。即不能使用关键字 `extends` 来继承某个 `@interface`。

	如果注解支持继承的话，将大大减少打字的工作量，并且使语法更加简洁。同时，由于注解没有继承机制，所以在处理注解的时候要获得近似多态的行为，使用 `getDeclaredAnnotation()` 是唯一的办法。
	
*   可以对一个目标同时使用多个注解。但是，使用多个注解的时候，同一个注解不能重复使用。

### 元注解

定义注解时，会需要一些元注解（meta-annatation），如 `@Target` 和 `@Retention`。元注解专职注解其他的注解：

*   `@Target` 表示该注解应用于什么地方。元注解的 `ElementType` 只能是 `ANNOTATION_TYPE`。可能的 `ElementType` 枚举参数如下：

	* `TYPE`：类、接口（包括注解类型）或 enum 声明
	
	* `FIELD`：域声明，包括 enum 常量
	
	* `METHOD`：方法声明
	
	* `PARAMETER`：参数声明

	* `CONSTRUCTOR`：构造器的声明
	
	* `LOCAL_VARIABLE`：局部变量声明
	
	* `ANNOTATION_TYPE`：注解类型的声明
	
	* `PACKAGE`：包声明
	
	* `TYPE_PARAMETER`：类型参数的声明
	
	* `TYPE_USE`
	
	如果想要将注解应用于所有的 `ElementType`，那么可以省去 `@Target` 元注解，不过这并不常见。

*   `@Retention` 表示需要在什么级别保存该注解信息。`@Retention` 的 `RetentionPolicy` 是 `RUNTIME`。可选的 `RetentionPolicy` 枚举参数如下：

	* `SOURCE`：注解将被编译器丢弃。
	
	* `CLASS`：注解会由编译器记录在 class 文件中，但在运行时会被 VM 丢弃。这是默认的行为。
	
	* `RUNTIME`：注解会由编译器记录在 class 文件中，VM 在运行时也会保留注解，因此可以通过反射机制读取注解的信息。
	
*   `@Documented` 将此注解包含在 Javadoc 中。

*   `@Inherited` 允许子类继承父类中的注解。

### 注解元素的类型

注解元素可用的类型如下所示：

*   所有基本类型

*   String

*   Class

*   enum

*   Annotation

*   以上类型的数组

如果你使用了其他类型，那编译器会报错。注意：不允许使用任何包装类型，不过由于自动打包的存在，这不算是什么限制。注解也可以作为元素的类型，也就是说注解可以嵌套；这是一个很有用的技巧。

### 默认值限制

编译器对元素的默认值有有一些限制：

*   首先，元素不能有不确定的值。也就是说，元素必须要么具有默认值，要么在使用注解时提供元素的值。

*   其次，对于非基本类型的元素，不能以 `null` 作为其值。
这个约束使得处理器很难表现一个元素的存在或缺失的状态。为了绕开这个约束，我们只能自己定义一些特殊的值，例如空字符串或负数，以此表示某个值不存在：

	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface SimulatingNull {
		public int id() default -1;

		public String description() default "";
	}
	```
	
### 赋值的快捷方式

如果在注解中定义了名为 `value` 的元素，并且在应用该注解的时候，如果该元素是唯一需要赋值的一个元素，那么此时无需使用名-值对的语法，而只需在括号内给出 `value` 元素所需的值即可。这可以应用于任何合法类型的元素。例如：

```java
@SQLString(30)
```

## 编写注解处理器

JavaSE 扩展了反射机制的 API，以帮助程序员构造*注解处理器*。同时，它还提供了一个外部工具 apt 帮助程序解析带有注解的 Java 源代码。

`Class`、`Method` 和 `Field` 等类都实现了 `AnnotatedElement` 接口。

*   `getAnnotation()` 返回指定类型的注解对象。

### 基于注解生成外部文件

有些框架需要一些额外的信息才能与你的源代码协同工作，这种情况最适合应用注解了。例如 Enterprise JavaBean（EJB3 之前）的每一个 Bean 都需要大量的接口和部署描述文件，而这些都属于“样板”文件。Web Service、自定义标签库以及对象/关系映射工具（例如 Toplink 和 Hibernate 等），一般都需要 XML 描述文件，而这些描述文件脱离于源代码之外。因此，在定义了 Java 类之后，程序员还必须重复提供某些信息，例如类名和包名等已经在原始的类文件中提供了的信息。每当程序员使用外部的描述文件时，他就拥有了同一个类的两个单独的信息源，这经常导致代码同步问题。

假设你希望提供一些基本的对象/关系映射功能，能够自动生成数据库表，用以存储 JavaBean 对象。你可以选择使用 XML 描述文件，指明类的名字、每个成员以及数据库映射的相关信息。然而，如果使用注解的话，你可以将所有信息都保存在 JavaBean 源文件中。

现在已经有了很多可用的框架，可以将对象映射到关系数据库，并且，其中越来越多的框架已经开始利用注解了。
