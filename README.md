# Java

## 对象

*   [原始类型](object/primitive-types.md)

*   [协变、逆变和不变](object/covariant-contravariant-invariant.md)

### 对于所有对象都通用的方法

`Object` 类所有的非 `final` 方法都有明确的通用约定（general contract），因为它们被设计成是要被覆盖（override）的。任何一个类，在覆盖这些方法的时候，都有责任遵守这些通用约定；否则，其他依赖于这些约定的类（例如 `HashMap` 和 `HashSet`）就无法结合该类一起正常运作。

*   [equals() 和 hashCode() 方法](object/common-methods-to-all-objects/equals-and-hashcode-method.md) 覆盖 `equals()` 时请遵守通用约定；覆盖 `equals()` 时总是覆盖 `hashCode()`。

*   [toString() 方法](object/common-methods-to-all-objects/tostring-method.md) 始终要覆盖 `toString()`。

*   [clone() 方法](object/common-methods-to-all-objects/clone-method.md) 谨慎地覆盖 `clone()`。

*   finalize() 方法

*   [Comparable#compareTo() 方法](object/common-methods-to-all-objects/compareto-method.md) 虽然不是 `Object` 方法，但是它具有类似的特征。
	
**参考资料**

*   [Why Copying an Object is a terrible thing to do（为什么复制对象很可怕）](http://www.agiledeveloper.com/articles/cloning072002.htm)

## 语法基础

*   [操作符](grammer/operators.md)

*   [访问权限控制](grammer/access-control.md)  

*   [final 关键字](grammer/final-keyword.md)

*   [日期和时间](date-and-time.md)

## 容器

*   [散列和散列码](containers/hashing-and-hash-codes.md)

## 注解

*   [注解介绍](annotations/annotation-introduction.md)

*   [使用 apt 处理注解](annotations/using-apt-to-process-annotations.md)

## 异常

*   [异常](exception.md)

## 类型信息

Java 如何在运行时识别对象和类的信息？主要有两种方式：

*   “传统的” [RTTI（运行时类型信息）](type-info/rtti.md)

	假定我们在编译时已经知道了所有的类型，那么就可以在运行时通过传统的类型转换、`Class` 对象或关键字 `instanceof` 来发现和使用类型信息。

*   [反射机制](type-info/reflection.md)

	如果在编译时不知道类的类型，那么就无法使用 RTTI 来识别类型信息。但是通过反射，我们仍然能够在运行时发现和使用类的信息。
	
RTTI 和反射之间的真正区别只在于，对 RTTI 来说，编译器在编译时打开和检查 `.class` 文件。（换句话说，我们可以用“普通”方式调用对象的所有方法。）而对于反射来说，`.class` 文件在编译时是不可获取的，所以是在运行时打开和检查 `.class` 文件。
	
## JavaEE

*   [Bean Validation](bean-validation/README.md)

## 测试

### Hamcrest

Hamcrest 是一个用于构建测试表达式的匹配器库，主要用于单元测试。

*   [Hamcrest 入门](hamcrest/hamcrest-start.md)

*   [Hamcrest 教程（翻译ING）](hamcrest/hamcrest-tutorial.md)

	官方提供的入门教程，可以在 https://github.com/hamcrest/JavaHamcrest/wiki/The-Hamcrest-Tutorial 找到。

### Mockito

Mockito 是用于 Java 单元测试的 mocking 框架。Mockito 库支持创建 mock、验证和 stubbing。

*   [Mockito 入门](mockito/mockito-start.md)

*   [Mockito 参考（翻译ING）](mockito/mockito-reference.md)

	Mockito 的 Javadoc，可以在源代码或者 http://mockito.org 网页上找到。

## 参考资料

### ASCII

*   [ASCII码对照表](http://tool.oschina.net/commons?type=4)

*   [ASCII码表——在线ASCII字符转换](ascii/ASCII码表——在线ASCII字符转换.mht)

*   [ASCII Table and Description](ascii/Ascii Table - ASCII character codes and html, octal, hex and decimal chart conversion.mht)
