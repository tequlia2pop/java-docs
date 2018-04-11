# Mockito 参考

原文地址：https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html

Mockito 库支持创建 mock、验证和 stubbing。

该 Javadoc 内容也可以在 http://mockito.org  网页上找到。所有文档都保存在 Javadoc 中，因为这保证了网页上的内容与源代码中的内容是一致的。即使脱机工作，你也可以直接从 IDE 访问文档。这激励 Mockito 开发人员 在每次提交日常代码时确保文档也是最新的。

## 0. Migrating to Mockito 2
### 0.1 Mockito Android support
### 0.2 Configuration-free inline mock making

## 1. 让我们来验证一些行为！ 

下面的例子 mock 一个 `List`，因为大多数人都熟悉接口（比如 `add()`、`get()`、`clear()`方法）。在现实中，请不要 mock `List` 类。改用真实的实例。

```java
//Let's import Mockito statically so that the code looks clearer
import static org.mockito.Mockito.*;

//mock creation
List mockedList = mock(List.class);

//using mock object
mockedList.add("one");
mockedList.clear();

//verification
verify(mockedList).add("one");
verify(mockedList).clear();
```

一旦创建 mock，它将记住所有的交互。那么你可以选择性地验证你感兴趣的任何交互。

## 2. 做一些 stubbing 如何？

```java
// 你可以 mock 具体类，而不仅仅是接口
LinkedList mockedList = mock(LinkedList.class);

// stubbing
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());

// 下面会打印 "first"
System.out.println(mockedList.get(0));

// 下面会抛出运行时异常
System.out.println(mockedList.get(1));

// 下面会打印 "null"，因为 get(999) 尚未被 stub
System.out.println(mockedList.get(999));

// 尽管可以验证 stub 的调用，但这通常是多余的
// 如果你的代码关心 get(0) 返回的东西，那么其他东西就会中断（通常甚至在执行 verify() 之前）。
// 如果你的代码不关心 get(0) 返回的内容，那么它不应该被 stub。
verify(mockedList).get(0);
```

*   默认情况下，对于所有有返回值的方法，mock 根据情况可能会返回 `null`、基本类型/基本类型包装器的值，或空的集合。比如，对于 `int`/`Integer` 为0，对于 `boolean`/`Boolean` 为 `false`。

*    可以覆盖 stubbing：比如，common stubbing can go to fixture setup，但测试方法可以覆盖它。Please note that overridding stubbing is a potential code smell that points out too much stubbing

*   一旦 stub 了某个方法，该方法总是会返回一个 stubbed 值，不管它被调用了多少次。

*   当你多次使用相同的参数对相同的方法 stubbing 时，最后的 stubbing 更为重要。换句话说：stubbing 的顺序很重要，但它很少会有意义，比如 stubbing 完全相同的方法调用时，或有时使用参数匹配器时，等等。

## 3. 参数匹配器

Mockito 以自然的 Java 风格来验证参数值：通过使用 `equals()` 方法。有时，当需要额外的灵活性时，你可以使用参数匹配器：

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

参数匹配允许灵活的验证或 stubbing。点击[这里](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/ArgumentMatchers.html)或[这里](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/hamcrest/MockitoHamcrest.html)查看更多内置的匹配器和**自定义参数匹配器/hamcrest 匹配器**的示例。

关于**自定义参数匹配器**的信息，请查阅 [`ArgumentMatcher`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/ArgumentMatcher.html) 类的 javadoc。

使用复杂的参数匹配是合理的。使用 `equals()` 和偶尔的 `anyX()` 匹配器的自然匹配风格倾向于给出干净和简单的测试。有时候重构代码以允许 `equals()` 匹配会更好，或者甚至可以实现 `equals()` 来帮助进行测试。

另外，请阅读第15节或 `ArgumentCaptor` 类的 javadoc。`ArgumentCaptor` 是参数匹配器的一个特殊实现，它可以捕获参数值用于进一步的断言。

**参数匹配器的警告：**

如果你使用了参数匹配器，那么**所有参数**都必须由匹配器提供。

以下示例展示了验证，但同样适用于 stubbing：

```java
verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
// 上面是正确的——eq() 也是一个参数匹配器

verify(mock).someMethod(anyInt(), anyString(), "third argument");
// 上面是不正确的——因为给出的第三个参数不是一个参数匹配器，所以会抛出异常。
```

像 `anyObject()`、`eq()` 这样的匹配器方法**不会**返回匹配器。在内部，它们在堆栈上记录一个匹配器并返回一个虚拟值（通常为空）。这个实现是由于 java 编译器强加的静态类型安全。结果是你不能在已验证/stubbed 方法之外使用 `anyobject()`、`eq()` 方法。

## 4. 验证确切的调用次数/至少x次/不调用

```java
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
```

**`times(1)` 是默认值。**因此当验证调用一次时可以省略使用它。

## 5. Stubbing void methods with exceptions 
## 6. Verification in order 
## 7. Making sure interaction(s) never happened on mock 
## 8. Finding redundant invocations 
## 9. Shorthand for mocks creation - @Mock annotation 
## 10. Stubbing consecutive calls (iterator-style stubbing) 
## 11. Stubbing with callbacks 

## 12. doReturn()|doThrow()|doAnswer()|doNothing()|doCallRealMethod() 系列方法

由于编译器不喜欢括号内是 `void` 方法，所以 stubbing `void` 方法需要与 `when(Object)` 不同的方式……

当你想 stub 一个会抛出异常的 `void` 方法时，使用 `doThrow()`：

```java
doThrow(new RuntimeException()).when(mockedList).clear();

// 下面会抛出 RuntimeException:
mockedList.clear();
```

对于任何方法，你可以使用 `doThrow()`、`doAnswer()`、`doNothing()`、`doReturn()` 和 `doCallRealMethod()` 来代替与 `when()` 对应的调用。当你遇到下面的情况时，这是必要的：

*   stub `void` 方法

*   stub spy 对象上的方法（见下文）

*   多次 stub 相同的方法，以便在测试过程中改变 mock 的行为。

详细了解这些方法：

[`doReturn(Object)`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doReturn-java.lang.Object-)

[`doThrow(Throwable...)`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doThrow-java.lang.Throwable...-)

[`doThrow(Class)`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doThrow-java.lang.Class-)

[`doAnswer(Answer)`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doAnswer-org.mockito.stubbing.Answer-)

[`doNothing()`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doNothing--)

[`doCallRealMethod()`](https://static.javadoc.io/org.mockito/mockito-core/2.18.0/org/mockito/Mockito.html#doCallRealMethod--)

## 13. Spying on real objects 
## 14. Changing default return values of unstubbed invocations (Since 1.7) 
## 15. Capturing arguments for further assertions (Since 1.8.0) 
## 16. Real partial mocks (Since 1.8.0) 
## 17. Resetting mocks (Since 1.8.0) 
## 18. Troubleshooting & validating framework usage (Since 1.8.0) 
## 19. Aliases for behavior driven development (Since 1.8.0) 
## 20. Serializable mocks (Since 1.8.1) 
## 21. New annotations: @Captor, @Spy, @InjectMocks (Since 1.8.3) 
## 22. Verification with timeout (Since 1.8.5) 
## 23. Automatic instantiation of @Spies, @InjectMocks and constructor injection goodness (Since 1.9.0)
## 24. One-liner stubs (Since 1.9.0)
## 25. Verification ignoring stubs (Since 1.9.0)
## 26. Mocking details (Improved in 2.2.x)
## 27. Delegate calls to real instance (Since 1.9.5)
## 28. MockMaker API (Since 1.9.5)
## 29. BDD style verification (Since 1.10.0)
## 30. Spying or mocking abstract classes (Since 1.10.12, further enhanced in 2.7.13 and 2.7.14)
## 31. Mockito mocks can be serialized / deserialized across classloaders (Since 1.10.0)
## 32. Better generic support with deep stubs (Since 1.10.0)
## 33. Mockito JUnit rule (Since 1.10.17)
## 34. Switch on or off plugins (Since 1.10.15)
## 35. Custom verification failure message (Since 2.1.0)
## 36. Java 8 Lambda Matcher Support (Since 2.1.0)
## 37. Java 8 Custom Answer Support (Since 2.1.0)
## 38. Meta data and generic type retention (Since 2.1.0)
## 39. Mocking final types, enums and final methods (Since 2.1.0)
## 40. (*new*) Improved productivity and cleaner tests with "stricter" Mockito (Since 2.+)
## 41. (**new**) Advanced public API for framework integrations (Since 2.10.+)
## 42. (**new**) New API for integrations: listening on verification start events (Since 2.11.+)
## 43. (**new**) New API for integrations: MockitoSession is usable by testing frameworks (Since 2.15.+)
## 44. Deprecated org.mockito.plugins.InstantiatorProvider as it was leaking internal API. it was replaced by org.mockito.plugins.InstantiatorProvider2 (Since 2.15.4)
## 45. (**new**) New JUnit Jupiter (JUnit5+) extension