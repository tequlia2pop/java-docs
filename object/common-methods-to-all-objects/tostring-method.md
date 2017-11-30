# toString() 方法

`java.lang.Object` 的 `toString()` 返回的是类名，以及一个“@”符号，跟着散列码的无符号十六进制值，例如“PhoneNumber@163b91”。

toString 的通用约定指出：

*   返回的字符串应该是一个“简洁的，但信息丰富、并且易于阅读的表达形式”[JavaSE6]。

*   “建议所有的子类都覆盖这个方法”。

提供良好的 `toString()` 实现可以使类用起来更加舒适，这不仅有益于类的实例，同样也有益于那些包含这些实例的引用的对象，特别是集合对象。

在实际应用中，`toString()` 应该返回对象中包含的所有值得关注的信息。如果对象太大，或者对象中包含的状态信息难以用字符串来表达，`toString()` 应该返回一个摘要信息，例如“Manhattan white pages(1487536 listings)”或者“Thread[main, 5, main]”。

在实现 `toString()` 时，必须决定是否指定返回值的格式。对于值类（value class），比如电话号码类、矩阵类，是建议这么做的。指定格式可以被用作一种标准的、明确的、适合人阅读的对象表示法。这种表示法可以用于输入和输出，以及用在永久的适合人类阅读的数据对象中，例如 XML 文档。

*   如果指定了格式，最好再提供一个相匹配的静态工厂或者构造器，以便程序员可以很容易地在对象和它的字符串表示法之间来回转换。Java 中的许多值类都采用了这种做法，包括 `BigInteger`、`BigDecimal` 和绝大多数基本类型包装类。

*   一旦指定了格式，就必须始终如一地坚持这种格式。程序员将会编写出相应的代码来解析这种字符串表示法，产生字符串表示法，以及把字符串表示法嵌入到持久的数据中。如果将来的发行版本中改变了这种表示法，就会破坏它们的代码和数据。

无论你是否决定指定格式，就应该在文档中明确表明你的意图。如果你要指定格式，就应该严格地这样去做。

### 示例代码

以 `PhoneNumber` 类为例，如果你决定指定格式：

```java
/**
 * 返回此电话号码的字符串表示。
 * 该字符串由14个字符组成，格式为"(XXX) YYY-ZZZZ"，
 * 其中"XXX"是 areaCode，"YYY"是 prefix，"ZZZZ" 是 lineNumber。
 * （每个大写字母代表一个十进制数字。）
 * 
 * 如果此电话号码的三个部分中的任何一个由于太短而不能填满它的字段，
 * 该字段将使用零来填充前面的空位。例如，如果 lineNumber 值是123，
 * 该字符串表示的最后4个字符就是"0123"。
 * 
 * 注意，在 areaCode 之后，有一个单独的空格用于分隔右括号和 prefix 的第一个数字。
 */
@Override
public String toString() {
	return String.format("(%03d) %03d-%04d", areaCode, prefix, lineNumber);
}
```

如果你决定不指定格式：

```java
/**
 * 返回电话号码的简要说明。
 *
 * 该表示的具体细节没有说明，可能会有变动。通常的表示如下：
 * "PhoneNumber [areaCode=123, prefix=123, lineNumber=1234]"
 */
@Override
public String toString() {
	return "PhoneNumber [areaCode=" + areaCode + ", prefix=" + prefix + ", lineNumber=" + lineNumber + "]";
}
```