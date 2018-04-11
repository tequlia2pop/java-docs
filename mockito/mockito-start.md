# Mockito 入门

Mockito 是用于 Java 单元测试的 mocking 框架。

## 简介

Mockito 是一个优秀的 mocking 框架。它可以让你用干净和简单的 API 编写漂亮的测试。Mockito 不会让你“宿醉”，因为这些测试非常容易阅读，而且它们会产生干净的验证错误。阅读更多关于[功能和动机](https://github.com/mockito/mockito/wiki/Features-And-Motivations)的内容。

Mockito 当然也有一定的限制，入下面三种数据类型则不能够被测试：

*   final classes

*   anonymous classes

*   primitive types

在使用 Mockito 时请注意：

*   不要 mock 你不拥有的类型

*   不要 mock 有价值的对象

*   不要 mock 所有的东西

## 如何使用

获取 Mockito 的推荐方式是，使用你的构建系统声明 "mockito-core" 库的依赖。

使用 Gradle：

```
repositories { jcenter() }
dependencies { testCompile "org.mockito:mockito-core:2.+" }
```

## 验证交互

使用 `mock()` 可以创建 mock 对象，可以 mock 具体类和接口。

一旦创建 mock，它将记住所有的交互。那么你可以选择性地验证你感兴趣的任何交互。

```java
import static org.mockito.Mockito.*;

// 创建 mock
List mockedList = mock(List.class);

// 使用 mock 对象——它不会抛出任意的“未受检查的交互”异常
mockedList.add("one");
mockedList.clear();

// 可选择的、明确的、高度可读的验证
verify(mockedList).add("one");
verify(mockedList).clear();
```

使用 `verify()` 可以验证接受特定参数的方法是否被调用。`verify()` 默认使用 `times(1)`，也就是要求方法只能调用一次。

```java
// 下面两个验证工作完全相同——verify() 默认使用 times(1)
verify(mockedList).add("once");
verify(mockedList, times(1)).add("once");
```

还可以使用 `atLeastOnce()`、`atLeast()` 和 `atMost()` 方法指定方法调用的次数：

```java
verify(mockedList, atLeastOnce()).add("three times");
verify(mockedList, atLeast(2)).add("three times");
verify(mockedList, atMost(5)).add("three times"); 
```

## stub 方法调用

使用 `when()` 和 `thenReturn()` 设置 mock 对象的特定方法的预期返回值。语法格式为：`when(methodCall).thenReturn(returnValue)`。

```java
// 你可以 mock 具体类，而不仅仅是接口
LinkedList mockedList = mock(LinkedList.class);

// stubbing appears before the actual execution
when(mockedList.get(0)).thenReturn("first");

// 下面会打印 "first"
System.out.println(mockedList.get(0));

// 下面会打印 "null"，因为 get(999) 尚未被 stub
System.out.println(mockedList.get(999));
```

如果你需要定义多个返回值，可以多次定义。当你多次调用方法的时候，Mockito 会根据你定义的先后顺序来返回这些返回值。

```java
Iterator i = mock(Iterator.class);
when(i.next()).thenReturn("Mockito").thenReturn("rocks");
// 下面会打印 "Mockito rocks"
System.out.println(i.next() + " " + i.next());
```

stub `void` 方法应该使用 `doXXX(...).when(...).methodCall` 的方式：

*   doReturn(Object)

*   doThrow(Throwable...)

*   doThrow(Class)

*   doAnswer(Answer)

*   doNothing()

*   doCallRealMethod()

当你想 stub 一个会抛出异常的 `void` 方法时，使用 `doThrow()`：

```java
doThrow(new RuntimeException()).when(mockedList).clear();

// 下面会抛出 RuntimeException:
mockedList.clear();
```

## 引入匹配器

默认情况下，Mockito 使用 `equals()` 方法来验证参数值。当需要额外的灵活性时，你可以使用参数匹配器：

```java
// 使用内建的 anyInt() 参数匹配器，执行 stubbing
when(mockedList.get(anyInt())).thenReturn("element");

// 使用自定义的匹配器（假设 isValid()）返回你自己的匹配器实现，执行 stubbing
when(mockedList.contains(argThat(isValid()))).thenReturn("element");

// 下面会打印 "element"
System.out.println(mockedList.get(999));

// 你也可以使用参数匹配器来验证
verify(mockedList).get(anyInt());

// 参数匹配器也可以写成 Java 8 Lambdas
verify(mockedList).add(argThat(someString -> someString.length() > 5));
```

注意，如果你使用了参数匹配器，那么**所有参数**都必须由匹配器提供。

```java
verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
// 上面是正确的——eq() 也是一个参数匹配器

verify(mock).someMethod(anyInt(), anyString(), "third argument");
// 上面是不正确的——因为给出的第三个参数不是一个参数匹配器，所以会抛出异常。
```

## 更多内容

[参考文档](http://javadoc.io/page/org.mockito/mockito-core/latest/org/mockito/Mockito.html)描述了以下主要功能：

*   `mock()`/`@Mock`：创建 mock

	* 可以通过 `Answer`/`ReturnValues`/`MockSettings` 指定应该如何表现
	* `when()`/`given()` 指定 mock 应该如何表现
	* 如果提供的答案不符合你的需求，你可以自己编写一个继承自 `Answer` 接口的答案
	
*   `spy()`/`@Spy`： partial mocking, real methods are invoked but still can be verified and stubbed

*   `@InjectMocks`: 自动注入以 `@Spy` 或 `@Mock` 注释的 mock/spy 字段。

*   `verify()`: 检查是否调用了给定参数的方法

	* 可以使用灵活的参数匹配，比如通过 `any()` 表示任意表达式
	* 或者使用 `@Captor` 来捕获所调用的参数

*   尝试使用 [BDDMockito](http://javadoc.io/page/org.mockito/mockito-core/latest/org/mockito/BDDMockito.html) 的行为驱动（Behavior-Driven）的开发语法

*   在 Android 上使用 Mockito，感谢 [dexmaker](https://github.com/crittercism/dexmaker) 团队的工作

## JUnit 示例

**验证确切的调用次数/至少x次/不调用**

```
@Test
public void verifyInvocationNumbers() {
	LinkedList mockedList = mock(LinkedList.class);

	// 使用 mock
	mockedList.add("once");

	mockedList.add("twice");
	mockedList.add("twice");

	mockedList.add("three times");
	mockedList.add("three times");
	mockedList.add("three times");

	// 下面两个验证工作完全相同——verify() 默认使用 times(1)
	verify(mockedList).add("once");
	verify(mockedList, times(1)).add("once");

	// 确切的调用次数验证
	verify(mockedList, times(2)).add("twice");
	verify(mockedList, times(3)).add("three times");

	// 使用 never() 进行验证。never() 是 times(0) 的别名
	verify(mockedList, never()).add("never happened");

	// 使用 atLeast()/atMost() 进行验证
	verify(mockedList, atLeastOnce()).add("three times");
	verify(mockedList, atLeast(2)).add("three times");
	verify(mockedList, atMost(5)).add("three times");
}
```

**方法调用抛出异常**

```java
@Test(expected = RuntimeException.class)
public void testDoThrow(){
	LinkedList mockedList = mock(LinkedList.class);
	
	doThrow(new RuntimeException()).when(mockedList).clear();

	// 下面抛出 RuntimeException:
	mockedList.clear();
}
```

**根据参数返回值**

```java
// 如何根据参数值来返回值
@Test
public void testReturnValDependOnParamVal() {
	Comparable c = mock(Comparable.class);
	when(c.compareTo("Mockito")).thenReturn(1);
	when(c.compareTo("Eclipse")).thenReturn(2);
	assertEquals(1, c.compareTo("Mockito"));
}

// 如何根据参数类型来返回值
@Test
public void testReturnValInDependOnParamType() {
	Comparable c = mock(Comparable.class);
	when(c.compareTo(isA(Integer.class))).thenReturn(0);
	assertEquals(0, c.compareTo(new Integer(1)));
}

// 如何让返回值不依赖于特定参数
@Test
public void testReturnValInDependOnParam() {
	Comparable c = mock(Comparable.class);
	when(c.compareTo(anyInt())).thenReturn(-1);
	assertEquals(-1, c.compareTo(9));
}
```

## 参考资料

*   [Mockito 官网](http://site.mockito.org/)

*   [Mockito Github Wiki](https://github.com/mockito/mockito/wiki)

*   [Mockito 参考文档](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html)

**博客**

*   [使用强大的 Mockito 测试框架来测试你的代码](https://github.com/xitu/gold-miner/blob/master/TODO/Unit-tests-with-Mockito.md)
