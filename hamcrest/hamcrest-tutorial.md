# Hamcrest 教程

原文地址：https://github.com/hamcrest/JavaHamcrest/wiki/The-Hamcrest-Tutorial

## 介绍

Hamcrest is a framework for writing matcher objects allowing 'match' rules to be defined declaratively. There are a number of situations where matchers are invaluble, such as UI validation, or data filtering, but it is in the area of writing flexible tests that matchers are most commonly used. This tutorial shows you how to use Hamcrest for unit testing.

When writing tests it is sometimes difficult to get the balance right between overspecifying the test (and making it brittle to changes), and not specifying enough (making the test less valuable since it continues to pass even when the thing being tested is broken). Having a tool that allows you to pick out precisely the aspect under test and describe the values it should have, to a controlled level of precision, helps greatly in writing tests that are "just right". Such tests fail when the behaviour of the aspect under test deviates from the expected behaviour, yet continue to pass when minor, unrelated changes to the behaviour are made.

## 第一个 Hamcrest 测试

我们首先编写一个非常简单的 JUnit 3 测试，但是不使用 JUnit 的 `assertEquals` 方法，而是使用 Hamcrest 的 `assertThat` 和一系列标准的匹配器。我们需要静态导入这两部分：

```java
import static org.hamcrest.MatcherAssert.assertThat; 
import static org.hamcrest.Matchers.*;
import junit.framework.TestCase;

public class BiscuitTest extends TestCase { 
  public void testEquals() { 
    Biscuit theBiscuit = new Biscuit("Ginger"); 
    Biscuit myBiscuit = new Biscuit("Ginger"); 
    assertThat(theBiscuit, equalTo(myBiscuit)); 
  } 
} 
```

`assertThat` 方法是用于生成测试断言的程式化语句。在这个例子中，断言的主题是第一个方法参数 biscuit 对象。第二个方法参数是 Biscuit 对象的匹配器，这里使用了 Object 的 equals 方法来检查一个对象是否等于另一个对象。由于 Biscuit 类定义了一个 equals 方法，所以测试通过。

如果测试中有多个断言，则可以为断言中的测试值声明一个标识符：

```java
assertThat("chocolate chips", theBiscuit.getChocolateChipCount(), equalTo(10)); 

assertThat("hazelnuts", theBiscuit.getHazelnutCount(), equalTo(3));
```

## 其他测试框架

Hamcrest has been designed from the outset to integrate with different unit testing frameworks. For example, Hamcrest can be used with JUnit 3 and 4 and TestNG. (For details have a look at the examples that come with the full Hamcrest distribution.) It is easy enough to migrate to using Hamcrest-style assertions in an existing test suite, since other assertion styles can co-exist with Hamcrest's.

Hamcrest can also be used with mock objects frameworks by using adaptors to bridge from the mock objects framework's concept of a matcher to a Hamcrest matcher. For example, JMock 1's constraints are Hamcrest's matchers. Hamcrest provides a JMock 1 adaptor to allow you to use Hamcrest matchers in your JMock 1 tests. JMock 2 doesn't need such an adaptor layer since it is designed to use Hamcrest as its matching library. Hamcrest also provides adaptors for EasyMock 2. Again, see the Hamcrest examples for more details.

## 常见匹配器概览

Hamcrest 自带了一个有用的匹配器库。下面是最重要的几个。

### 核心

`anything` - 总是匹配，如果你不在乎被测试的对象是什么，那就很有用

`describedAs` - 装饰，用于添加自定义的失败描述

`is` - 提高可读性的装饰 - 查看下面的“语法糖”

### 逻辑

`allOf` - 如果所有匹配器均匹配，则认为是匹配的，短路（如 Java &&）

`anyOf` - 如果任意的匹配器匹配，则认为是匹配的，短路（如 Java ||）

`not` - 如果与包含的匹配器不匹配，则认为是匹配的，反之亦然

### 对象

`equalTo` - 使用 Object.equals 测试对象的等同性

`hasToString` - 测试 Object.toString

`instanceOf`, `isCompatibleType` - 测试类型

`notNullValue`, `nullValue` - null 测试

`sameInstance` - 测试对象是否是同一个

### Beans

`hasProperty` - 测试 JavaBeans 属性

### 集合（Collection）

`array` - test an array's elements against an array of matchers

`hasEntry`, `hasKey`, `hasValue` - 测试映射是否包含 entry、键或值

`hasItem`, `hasItems` - 测试集合是否包含元素

`hasItemInArray` - 测试数组是否包含一个元素

### 数字

`closeTo` - test floating point values are close to a given value

`greaterThan`, `greaterThanOrEqualTo`, `lessThan`, `lessThanOrEqualTo` - 测试排序

### 文本

`equalToIgnoringCase` - test string equality ignoring case

`equalToIgnoringWhiteSpace` - test string equality ignoring differences in runs of whitespace

`containsString`, `endsWith`, `startsWith` - 测试字符串匹配

### 语法糖

Hamcrest 努力使测试尽可能容易理解。比如，is 匹配器是一个包装，它不会向底层匹配器添加任何额外的行为。以下断言都是相同的：

```java
assertThat(theBiscuit, equalTo(myBiscuit)); 
assertThat(theBiscuit, is(equalTo(myBiscuit))); 
assertThat(theBiscuit, is(myBiscuit));
```

由于 `is(T value)` 被重载为返回 `is(equalTo(value))`，所以最后一种形式也是允许的。

## 编写自定义的匹配器

Hamcrest comes bundled with lots of useful matchers, but you'll probably find that you need to create your own from time to time to fit your testing needs. This commonly occurs when you find a fragment of code that tests the same set of properties over and over again (and in different tests), and you want to bundle the fragment into a single assertion. By writing your own matcher you'll eliminate code duplication and make your tests more readable!

Let's write our own matcher for testing if a double value has the value NaN (not a number). This is the test we want to write:

```java
public void testSquareRootOfMinusOneIsNotANumber() { 
  assertThat(Math.sqrt(-1), is(notANumber())); 
}
```

And here's the implementation:

```java 
package org.hamcrest.examples.tutorial;

import org.hamcrest.Description; 
import org.hamcrest.Factory; 
import org.hamcrest.Matcher; 
import org.hamcrest.TypeSafeMatcher;

public class IsNotANumber extends TypeSafeMatcher {

  @Override 
  public boolean matchesSafely(Double number) { 
    return number.isNaN(); 
  }

  public void describeTo(Description description) { 
    description.appendText("not a number"); 
  }

  @Factory 
  public static Matcher notANumber() { 
    return new IsNotANumber(); 
  }

} 
```

The `assertThat` method is a generic method which takes a Matcher parameterized by the type of the subject of the assertion. We are asserting things about Double values, so we know that we need a `Matcher<Double>`. For our Matcher implementation it is most convenient to subclass `TypeSafeMatcher`, which does the cast to a Double for us. We need only implement the `matchesSafely` method - which simply checks to see if the Double is NaN - and the `describeTo` method - which is used to produce a failure message when a test fails. Here's an example of how the failure message looks:

```java
assertThat(1.0, is(notANumber()));

// fails with the message

java.lang.AssertionError: Expected: is not a number got : <1.0>

```
The third method in our matcher is a convenience factory method. We statically import this method to use the matcher in our test:

```java
import static org.hamcrest.MatcherAssert.assertThat; 
import static org.hamcrest.Matchers.*;

import static org.hamcrest.examples.tutorial.IsNotANumber.notANumber;

import junit.framework.TestCase;

public class NumberTest extends TestCase {

  public void testSquareRootOfMinusOneIsNotANumber() { 
    assertThat(Math.sqrt(-1), is(notANumber())); 
  } 
} 
```

Even though the `notANumber` method creates a new matcher each time it is called, you should not assume this is the only usage pattern for your matcher. Therefore you should make sure your matcher is stateless, so a single instance can be reused between matches.

## Sugar generation (TBD)

If you produce more than a few custom matchers it becomes annoying to have to import them all individually. It would be nice to be able to group them together in a single class, so they can be imported using a single static import much like the Hamcrest library matchers. Hamcrest helps out here by providing a way to do this by using a generator.

First, create an XML configuration file listing all the Matcher classes that should be searched for factory methods annotated with the org.hamcrest.Factory annotation. For example:

```
TBD
```

Second, run the org.hamcrest.generator.config.XmlConfigurator command-line tool that comes with Hamcrest. This tool takes the XML configuration file and generates a single Java class that includes all the factory methods specified by the XML file. Running it with no arguments will display a usage message. Here's the output for the example.

```java 
package org.hamcrest.examples.tutorial;

public class Matchers {

  public static org.hamcrest.Matcher is(T param1) { 
    return org.hamcrest.core.Is.is(param1); 
  }

  public static org.hamcrest.Matcher is(java.lang.Class param1) { 
    return org.hamcrest.core.Is.is(param1); 
  }

  public static org.hamcrest.Matcher is(org.hamcrest.Matcher param1) { 
    return org.hamcrest.core.Is.is(param1); 
  }

  public static org.hamcrest.Matcher notANumber() { 
    return org.hamcrest.examples.tutorial.IsNotANumber.notANumber(); 
  }

} 
```

Finally, we can update our test to use the new Matchers class.

```java
import static org.hamcrest.MatcherAssert.assertThat;

import static org.hamcrest.examples.tutorial.Matchers.*;

import junit.framework.TestCase;

public class CustomSugarNumberTest extends TestCase {

  public void testSquareRootOfMinusOneIsNotANumber() { 
    assertThat(Math.sqrt(-1), is(notANumber())); 
  } 
} 
```

Notice we are now using the Hamcrest library is matcher imported from our own custom Matchers class.