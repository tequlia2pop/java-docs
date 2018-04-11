# Hamcrest 入门

## 概述

Hamcrest 是一个用于构建测试表达式的匹配器库，包含了大量有用的匹配器对象（也称为约束或者谓语）。

注意，Hamcrest 本身并不是一个测试框架，但确切地说，它可以帮助通过声明方式指定简单的匹配规则。这些匹配规则可以在许多不同的情况下使用，但是它们尤其适用于单元测试。

Hamcrest 诞生于 Java，现在已经移植到了多种语言之中：

*   [Java](http://hamcrest.org/JavaHamcrest)
*   [Python](http://github.com/hamcrest/PyHamcrest)
*   [Ruby](http://github.com/hamcrest/ramcrest)
*   [Objective-C](http://github.com/hamcrest/OCHamcrest)
*   [PHP](http://code.google.com/p/hamcrest/downloads/list?q=label:PHP)
*   [Erlang](http://github.com/hyperthunk/hamcrest-erlang)
*   [Swift](https://github.com/nschum/SwiftHamcrest)

Hamcrest 的优点：

*   Hamcrest 匹配器能够互相嵌套。

*   Hamcrest 能够在断言失败时提供一些可读的描述信息。

*   Hamcrest 极具扩展性。编写用来检查某个特定条件的匹配器是非常容易的。

## 下载

可以从 [Maven 中央仓库](http://search.maven.org/#search|ga|1|g%3Aorg.hamcrest) 获取 Hamcrest 文件。

Hamcrest 由不同的 jar 包组成，以满足应用程序的不同需求。

*   hamcrest-core.jar

	这是核心 API，供第三方框架提供商所使用。它包括了一系列基础的匹配器实现，可用于常见的操作。该 API 是稳定的，很少会改变。这是你所需要的最基本的库。

*   hamcrest-library.jar

	基于 hamcrest-core.jar 核心功能的匹配器实现库，处于不断增长中。它将随着版本发布而增长。

*   hamcrest-generator.jar

	一个工具，允许将许多匹配器实现组合成一个单独的类，类中的静态方法可以返回不同的匹配器。 这样的话，用户就不必记住许多要导入的类/包。生成代码。这个库只在编译时供内部使用。运行时不需要使用任何其他的 hamcrest 库。

*   hamcrest-integration.jar

	提供了 Hamcrest 和其他测试工具之间的集成，包括 Junit（3和4）、Testng、jMock 和 EasyMock。它使用了 hamcrest-core.jar 和 hamcrest-library.jar。

*   hamcrest-all.jar

	该 jar 包含了所有其他 jar 的所有类。

**Maven 版本**

上面的描述也适用于 hamcrest Maven 构件。jar 之间的依赖由 Maven 依赖机制来表示，比如 hamcrest-integration 使用了 hamcrest-library，而 hamcrest-library 使用了 hamcrest-core。Maven 仓库中没有 hamcrest-all 这个库。只需使用 hamcrest-integration 即可，它引用了所有其他的 hamcrest 库。

## 引入 Hamcrest

下面是一个 Hamcrest 库编写的测试方法：

```java
import static org.junit.Assert.assertTrue;

import java.util.ArrayList;
import java.util.List;

import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.assertThat;
import static org.hamcrest.CoreMatchers.anyOf;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.junit.matchers.JUnitMatchers.hasItem;

public class HamcrestTest {

	private List<String> values;

	@Before
	public void setUpList() {
		values = new ArrayList<String>();
		values.add("x");
		values.add("y");
		values.add("z");
	}

	@Test
	public void testWithoutHamcrest() {
		assertTrue(values.contains("one") || values.contains("two")
				|| values.contains("three"));
	}

	@Test
	public void testWithHamcrest() {
		assertThat(values, hasItem(anyOf(equalTo("one"), equalTo("two"),
			equalTo("three"))));
	}
}
```

当上述测试失败后，栈跟踪信息如下

```java
java.lang.AssertionError: 
Expected: a collection containing ("one" or "two" or "three")
 got: <[x, y, z]>

at com.manning.junitbook.ch03.mastering.HamcrestTest.testWithHamcrest(HamcrestTest.java:61)
```

## 匹配器概述

### 核心匹配器

`org.hamcrest.CoreMatchers` 类提供了以下核心匹配器：

*   allOf

*   anyOf

	测试是否与任一包含的匹配器匹配（相当于 &#124;&#124; 运算符）。

	```java
	assertThat("myValue", anyOf(startsWith("foo"), containsString("Val")))
	```

*   both

*   either

*   describedAs

*   everyItem

	测试集合的所有元素是否都符合匹配器的要求。

	```java
	assertThat(Arrays.asList("bar", "baz"), everyItem(startsWith("ba")))`
	```

*   is 

	用于装饰另一个匹配器，保留它的行为，使得测试更具可读性。

	比如，下面的断言是等价的：

	```java
	assertThat(cheese, is(smelly))

	assertThat(cheese, is(equalTo(smelly)))

	assertThat(cheese, equalTo(smelly))
	```

*   isA

*   anything

*   hasItem

	测试集合是否包含符合匹配器要求的那一个元素。

	比如，注意前两个断言是等价的：

	```java
	assertThat(Arrays.asList("foo", "bar"), hasItem("bar"))
	
	assertThat(Arrays.asList("foo", "bar"), IsCollectionContaining.hasItem("bar"))
	
	assertThat(Arrays.asList("foo", "bar"), hasItem(startsWith("ba")))
	```

*   hasItems

	测试集合是否包含符合匹配器要求的多个元素。
	
	比如，注意前两个断言是等价的：
	
	```java
	assertThat(Arrays.asList("foo", "bar", "baz"), hasItems("baz", "foo"))
	
	assertThat(Arrays.asList("foo", "bar", "baz"), IsCollectionContaining.hasItems("baz", "foo"))
	
	assertThat(Arrays.asList("foo", "bar", "baz"), hasItems(endsWith("z"), endsWith("o")))
	```

*   equalTo

*   any

*   instanceOf

*   not

	与包含的匹配器的意思相反（相当于 `!` 运算符）。

	```java
	assertThat(cheese, is(not(equalTo(smelly))))
	```

*   nullValue

*   notNullValue

*   sameInstance

*   theInstance

*   containsString

*   startsWith

*   endsWith

### 其他匹配器

**集合（Collection）**

*   IsCollectionWithSize.hasSize()

	测试集合的大小。

	```java
	assertThat(Arrays.asList("foo", "bar"), hasSize(2))
	```

*   IsIterableContainingInOrder.contains()

	测试集合是否包含指定的元素，且顺序一致。
	
	```java
	assertThat(Arrays.asList("foo", "bar"), contains("foo", "bar"))
	```

*   IsIterableContainingInAnyOrder.containsInAnyOrder()

	测试集合是否包含指定的元素，顺序无关。
	
	```java
	assertThat(Arrays.asList("foo", "bar"), containsInAnyOrder("bar", "foo"))
	```

*   IsEmptyCollection.empty()

	测试集合是否为空（Collection#isEmpty()）。
	
	```java
	assertThat(new ArrayList<String>(), is(empty()))
	```
	
**映射（Map）**

*   IsMapContaining - 测试映射是否包含符合匹配器要求的元素

	* hasEntry
	* hasKey
	* hasValue
	
	```java
	assertThat(myMap, hasEntry("bar", "foo"))
	
	assertThat(myMap, hasEntry(equalTo("bar"), equalTo("foo")))
	```
	
**排序**

*   OrderingComparison - 测试排序
	
	* greaterThan
	* greaterThanOrEqualTo
	* lessThan
	* lessThanOrEqualTo
	
	```java
	assertThat(2, greaterThan(1))
	```

## 常用匹配器列表

最常用的 Hamcrest 匹配器，如下表：

核心 | 逻辑
---- | ----
anything | 绝对匹配，在你想使得 assert 语句更具可读性的情况下非常有用
is | 仅用于改善语句的可读性
allOf | 检查是否与所有包含的匹配器匹配（相当于 && 运算符）
anyOf | 检查是否与任一包含的匹配器匹配（相当于 &#124;&#124; 运算符）
not | 与包含的匹配器的意思相反（相当于 ! 运算符）
instanceOf、isCompatibleType | 匹配对象是否是兼容类型（是另一个对象的实例）
sameInstance | 测试对象标识
notNullValue、nullValue | 测试 null 值（或者非 null 值）
hasProperty | 测试 JavaBean 是否具有某种属性
hasEntry、hasKey、hasValue | 测试给定的 Map 是否包含给定的实体、键或者值
hasItem、hasItems | 测试给定的集合包含了一个或多个元素
closeTo、greaterThan、greaterThanOrEqual、lessThan、lessThanOrEqual | 测试给定的数字是否接近于、大于、大于或等于、小于、小于或等于某个给定的值
equalToIgnoringCase | 通过忽略大小写，测试给定的字符串是否与另一个字符串相同
equalToIgnoringWhiteSpace | 通过忽略空格，测试给定的字符串是否与另一个字符串相同
containsString、endsWith、startWith | 测试给定的字符串是否包含了某个字符串，是否是以某个字符串开始或结束

## 参考资料

*   [Hamcrest 官网](http://hamcrest.org/)

*   [源码仓库](https://github.com/hamcrest/JavaHamcrest)

**Wiki**

*   [Hamcrest 入门](https://github.com/hamcrest/JavaHamcrest/wiki/The-Hamcrest-Tutorial)

**博客**

*   [Hamcrest使用](https://blog.csdn.net/fanxiaobin577328725/article/details/52862945)