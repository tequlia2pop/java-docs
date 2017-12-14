# 异常

Java 使用异常来提供一致的错误报告模型，使得构件能够与客户端代码可靠地进行沟通。

## 概念

早期语言的错误处理模式往往建立在约定俗成的基础之上，而这并不属于语言的一部分。通常会返回某个特殊值或设置某个标志，并且假定接受者将对这个返回值或标志进行检查，以判定是否发生了错误。

现代语言使用强制规定的形式来消除错误处理过程中随心所欲的因素。你也许不清楚该如何处理问题，但你不应该置之不理：在当前的环境中也许还没有足够的信息来解决这个问题，所以就把这个问题提交到一个更高级别的环境中，在这里将作出正确的决定。

异常的精髓是将所有错误都以异常形式进行报告。一致的错误报告系统意味着，你再也不必对所有的每一段代码，都质问自己“错误是否正在称为漏网之鱼？”。只要你没有“吞噬”异常，这是关键所在；即使你是通过抛出 `RuntimeException` 来实现这一点的。

使用异常还有下面的好处：

*   它往往能够降低错误处理代码的复杂度。

	异常捕获机制将保证能够捕获错误，并且只需在一个地方处理错误（即所谓的*异常处理程序*）即可。这种方式不仅节省代码，而且把“描述在正常执行过程中做什么事”的代码和“出了问题怎么办”的代码相分离。总之，异常机制使代码的阅读、编写和调试工作更加井井有条。

*   异常还可以看作是一种内建的恢复（undo）系统，因为（在细心使用的情况下）我们在程序中可以拥有各种不同的恢复点。如果程序的某部分失败了，异常将“恢复”到程序中某个已知的稳定点上。

	实际上真正能够实现“恢复”的情况非常少，有些只是将栈展开到某个已知的稳定状态，而并没有实际执行任何种类的恢复性行为。
	
下面给出了异常使用指南，指出了应该在哪些情况下使用异常：

1.  在恰当的级别处理问题。在知道该如何处理的情况下才捕获异常。

1.  解决问题并且重新调用产生异常的方法。

1.  进行少许修补，然后绕过异常发生的地方继续执行。

1.  用别的数据进行计算，以代替方法预计会返回的值。

1.  把当前运行环境下能做的事情尽量做完，然后把相同的或不同的异常重抛到更高层。

1.  终止程序。

## 基本异常

*异常情形（exceptional condition)*指阻止当前方法或作用域继续执行的问题。异常情形和普通问题的区别在于，异常情形在当前环境下无法获得必要的信息来解决问题，所能做的就是从当前环境跳出，并且把问题提交给上一级环境。这就是抛出异常时所发生的事情。

当抛出异常后，有几件事会随之发生。首先，使用 `new` 在堆上创建异常对象。然后，当前的执行路径被终止，并且从当前环境中弹出对异常对象的引用。此时，*异常处理机制*接管程序，并在*异常处理程序*中来继续执行程序。异常处理程序的任务是将程序从错误状态中恢复，以使程序能要么换一种方式运行，要么继续运行下去。

可以使用关键字 `throw` 来抛出一个异常：

```java
if(t == null)
	throw new NullPointerException();
```

`Throwable` 类是异常类型的根类。通常，对于不同类型的错误，要抛出相应的异常。错误信息可以保存在异常对象的内部，或者用异常类的名称来暗示。上一层环境通过这些信息来决定如何处理异常。

## 捕获异常

### 使用 try 块捕获异常

如果在方法内部抛出了异常（或者在方法内部调用的其他地方抛出了异常），这个方法将在抛出异常的过程中结束。要是不希望方法就此结束，可以在方法内设置一个 `try` 块来捕获异常：

```java
try {
	// Code that might generate exceptions
}
```

### 使用 catch 定义异常处理程序

针对每个要抛出的异常，需要准备相应的异常处理程序来进行处理。异常处理程序紧跟在 `try` 块之后，以关键字 `catch` 表示：

```java
try {
	// Code that might generate exceptions
} catch (Type1 id1) {
	// Handle exceptions of Type1
} catch (Type2 id2) {
	// Handle exceptiosn of Type2
} catch (Type3 id3) {
	// Handle exceptiosn of Type3
}

// etc...
```

当异常被抛出后，异常处理程序负责搜寻参数与异常类型相匹配的第一个处理程序。然后进入 `catch` 子句执行，此时认为异常得到了处理。一旦 `catch` 子句结束，则处理程序的查找过程结束。

查找的时候并不要求抛出的异常同处理程序所声明的异常完全匹配。派生类的对象也可以匹配其基类的处理程序。换句话说，`catch` 子句会捕获指定的异常以及所有从它派生的异常。

因此通过捕获 `Exception`，可以捕获所有的异常。事实上还有其他的基类，但 `Exception` 是同编程活动相关的基类。如果要捕获所有的异常，所以最好将它放在处理程序的末尾，以防它抢在其他处理程序之前先把异常捕获了。

**multicatch**

从 Java 7 开始支持 multicatch：

```java
public Configuration getConfig(String fileName) {
	Configuration cfg = null;
	try {
	  String fileText = getFile(fileName);
	  cfg = verifyConfig(parseConfig(fileText));
	} catch (FileNotFoundException | ParseException | ConfigurationException e) {
	  System.err.println("Config file '" + fileName
		  + "' is missing or malformed");
	} catch (IOException iox) {
	  System.err.println("Error while processing file '" + fileName + "'");
	}

	return cfg;
}
```

异常 `e` 的确切类型在编译时还无法得知，这意味着在 `catch` 块中只能把它当作可能异常的共同父类来处理（在实际编码时经常用 `Exception` 或 `Thowable` 来处理）。

### 终止与恢复

异常处理理论上有两种基本模型：

*   终止模型：假设错误非常关键，以至于程序无法返回到异常发生的地方继续执行。一旦异常被抛出，就表明错误已无法挽回，也不能回来继续执行。

*   恢复模型：异常处理程序会修正错误，然后重新尝试调用出问题的方法，并认为第二次能成功。

	如果要用 Java 实现类似恢复的行为，那么在遇见错误时就不能抛出异常，而是调用方法来修正该错误。或者，把 `try` 块放在 `while` 循环里，这样就不断地进入 `try` 块，直到得到满意的结果。
	
长久以来，程序员们最终还是转向使用类似“终止模型”的代码，并且忽略恢复行为。恢复模型的主要问题可能是它所导致的耦合：恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于抛出位置的非通用性代码。这增加了代码编写和维护的困难。

## 异常说明

Java 强制你使用特定的语法告知客户端程序员某个方法可能会抛出的异常类型，然后客户端程序员就可以进行相应的处理。这就是*异常说明*。

异常说明可能有两种意思。一个是“我的代码会产生这种异常，这由你来处理”。另一个是“我的代码忽略了这些异常，这由你来处理”。

异常说明属于方法声明的一部分，它在形式参数列表之后使用关键字 `throws`，后面再接一个所有潜在异常类型的列表。

```java
void f() throws TooBig, TooSmall, DivZero {...}
```

普通的方法定义表示不会抛出任何异常（除了从 `RuntimeException` 继承的异常，它们可以在没有异常说明的情况下被抛出）；

```java
void f() { // ...
```

代码必须与异常说明保持一致。如果方法里的代码产生了异常却没有进行处理，编译器会发现这个问题并提醒你：要么处理这个异常，要么就在异常说明中表明此方法将产生异常。通过这种自顶向下强制执行的异常说明机制，Java 在编译时就可以保证一定水平的异常正确性。

注意，可以声明方法将抛出异常，实际上却不抛出。这样做的好处是，为异常先占个位子，以后就可以抛出这种异常而不用修改已有的代码。在定义抽象基类和接口时可能会经常这么做，这样派生类或接口实现就能够抛出这些预先声明的异常。

## 重抛异常

有时希望把刚捕获的异常重新抛出，尤其是在使用 `Exception` 捕获所有异常的时候。既然已经得到了当前异常对象的引用，可以直接把它重新抛出：

```java
catch (Exception e) {
	System.out.println("An exception was thrown");
	throw e;
}
```

重抛异常会把异常抛给上一级环境中的异常处理程序，同一个 `try` 块的后续 `catch` 子句将被忽略。此外，异常对象的所有信息都得以保持，所以高一级环境中捕获此异常的处理程序可以从这个异常对象中得到所有信息。

如果只是把当前异常对象重新抛出，那么 `printStackTrace()` 方法显示的将是原来异常抛出点的调用栈信息，而并非重新抛出点的信息。要想更新这个信息，可以调用 `fillStackTrace()` 方法，这将返回一个 `Throwable` 对象，它是通过把当前调用栈信息填入原来那个异常对象而建立的。就像这样：

```java
catch (Exception e) {
	throw (Exception) e.fillInStackTrace();
}
```

有可能在捕获异常之后抛出另一种异常。这么做的效果类似于使用 `fillInStackTrace()`，有关原来异常发生点的信息会丢失，剩下的是与新的抛出点有关的信息：

```java
try {
	f();
} catch (OneException e) {
	throw new TwoException();
}
```

### final 重抛

从 Java 7 开始，可以使用 final 重抛表示实际抛出的异常就是运行时遇到的异常。在下面的代码中就是 `IOException` 或 `SQLException`：

```java
try {
	doSomethingWhichMightThrowIOException();
	doSomethingWhichMightThrowSQLException();
} catch(final Exception e) {
	...
	throw e;
}
```

### 异常链

常常会想要在捕获一个异常后抛出另一个异常，并且希望把原始异常的信息保存下来，这被称为*异常链*。所有的 `Throwable` 的子类在构造器中都可以接受一个 `cause`（因由）对象作为参数，这个 `cause` 就用来表示原始异常，这样通过把原始异常传递给新的异常，使得即使在当前位置创建并抛出了新的异常，也能通过这个异常链追踪到异常最初发生的位置。

```java
try {
	lowLevelOp();
} catch (LowLevelException le) {
	throw new HighLevelException(le);  // Chaining-aware constructor
}
```

在 `Throwable` 的子类中，只有三种基本的异常类提供了带 `cause` 参数的构造器：`Error`、`Exception` 以及 `RuntimeException`。如果要把其他类型的异常链接起来，应该使用 `initCause()` 方法：

```java
try {
	lowLevelOp();
} catch (LowLevelException le) {
 	throw (HighLevelException)
         new HighLevelException().initCause(le);  // Legacy constructor
}
```

## Java 标准异常

`Throwable` 类表示任何可以作为异常被抛出的类。`Throwable` 对象可分为两种类型：

* `Error` 表示编译时和系统错误。除特殊情况外，一般不用关心。

* `Exception` 是可以被抛出的基本类型。Java 程序员关心的基类型通常是 `Exception`。

异常的基本概念是用名称代表发生的问题，并且异常的名称应该可以望文知义。

### Exception

`Exception` 是与编程有关的所有异常类的基类，所以它不会含有太多具体的信息，不过可以调用它从其基类 `Throwable` 继承的方法：

方法 | 说明
---- | ----
`String getMessage()`<br/><br/>`String getLocalizedMessage()` | 获取异常的详细信息，或用本地语言表示的详细信息。
`String toString()` | 返回对 `Throwable` 的简单描述，也包括了详细信息（如果有的话）。
`void printStackTrace()`<br/><br/>`void printStackTrace(PrintStream)`<br/><br/>`void printStackTrace(PrintWriter)` | 打印 `Throwable` 的调用栈轨迹。调用栈显示了“从方法调用处直到异常抛出处”的方法调用序列。其中第一个版本输出到标准错误，后两个版本允许选择要输出的流。
`Throwable fillInStackTrace()` | 用于在 `Throwable` 对象的内部记录栈帧的当前状态。这在程序重新抛出错误或异常时很有用。

对于 `getMessage()`、`getLocalizedMessage()`、`toString()` 和 `printStackTrace()`，每个方法都比前一个提供了更多的信息——实际上它们每一个都是前一个的超集。

**栈轨迹**

`printStackTrace()` 方法所提供的信息可以通过 `getStackTrace()` 方法来直接访问，这个方法将返回一个由栈轨迹中的元素所构成的数组，其中每一个元素表示栈中的一帧。元素0是栈顶元素，并且是调用序列中的最后一个方法调用（这个 `Throwable` 被创建和抛出之处）。数组中的最后一个元素和栈底是调用序列中的第一个方法调用。

### RuntimeException

`RuntimeException` 是 `Exception` 的特例，它表示运行时异常。所有从 `RuntimeException` 继承而来的异常都是运行时异常。运行时异常会自动由 Java 虚拟机检测并抛出，所以不必在异常说明中列出来。尽管通常不用捕获 `RuntimeException` 异常，但还是可以在代码中抛出 `RuntimeException` 类型的异常（或者任何从 `RuntimeException` 继承的异常）。所以它们也被称为“不受检查的异常”。

如果不捕获 `RuntimeException` 会发生什么事呢？因为编译器不会对异常说明进行强制检查，`RuntimeException` 类型也许会穿越所有的执行路径直达 `main()` 方法，而不会被捕获，那么在程序退出前将调用异常的 `printStackTrace()` 方法。

只能在代码中忽略 `RuntimeException` 类型的异常，其他类型异常的处理都是由编译器强制实施的。究其原因，`RuntimeException` 代表的是编程错误：

1. 无法预料的错误。比如从你控制范围之外传递进来的 `null` 引用（对应 `NullPointerException`）。

2. 作为程序员，应该在代码中进行检查的错误。（比如对于 `ArrayIndexOutOfBoundsException`，就得注意一下数组的大小了。）在一个地方发生的异常，常常会在另一个地方导致错误。

### 关于“被检查的异常”

异常处理的一个重要原则是：只有在你知道如何处理的情况下才捕获异常。然而，被检查的异常可能强制你在可能还没准备好处理错误的时候被迫加上 `catch` 子句，这就导致了*吞食则有害（harmful if swallowed）*的问题。

```java
try {
	// ... to do something useful
} catch(ObligatoryException e) {} // Gulp!
```

仅从示意性的例子和小程序来看，“被检查的异常”的好处很明显。但是当程序开始变大的时候，过多的类型检查，特别是“被检查的异常”可能已经变得无法管理了。

“被检查的异常”和强静态类型检查的好处来自于：

1.  不在于编译器是否会强制程序员去处理错误，而是要有一致的、使用异常来报告错误的模型。

2.  不在于什么时候进行检查，而是一定要有类型检查。也就是说，必须强制程序使用正确的类型，至于这种强制施加于编译时还是运行时，那倒没关系。

此外，减少编译时施加的约束能显著提高程序员的效率。事实上，反射和泛型就是用来补充静态类型检查所带来的过多限制。

如果你发现“被检查的异常”造成了问题，可以考虑下列解决方法：

1.  把异常传递到控制台。

	对于简单的程序，最简单的能保护异常信息的方法，就是把它们从 `main()` 传递到控制台。
	
	```java
	public static void main(String[] args) throws Exception {
		...
	}
	```

2.  把“被检查的异常”转换为“不受检查的异常”。

	考虑这样一个问题：当在一个普通方法里调用别的方法时，如果不知道该怎样处理这个异常，但是也不想把它“吞”了。异常链提供了一种思路来解决这个问题，可以直接把“被检查的异常”包装进 `RuntimeException` 里面，然后再重新抛出。异常链还能保证你不会丢失任何原始异常的信息。
	
	```java
	try {
		// ... to do something useful
	} catch (IDontKnowWhatToDoWithThisCheckedException e){
		throw new RuntimeException(e);
	}
	```
	
	这种技巧给了你一种选择，你可以不写 `try/catch` 子句和/或异常说明，直接忽略异常，让它自己沿着调用栈往上“冒泡”。同时，还可以用 `getCause()` 捕获并处理特定的异常。

3.  创建自己的 `RuntimeException` 的子类。在这种方式中，不必捕获它，但是希望得到它的其他代码都可以捕获它。

## 自定义异常

Java 提供的异常体系不可能预见所有的希望加以报告的错误，所以可以自己定义异常类来表示程序中可能会遇到的特定问题。

要自己定义异常类，必须从已有的异常类继承，最好是选择意思相近的异常类继承。

建立新的异常类型最简单的方法就是让编译器为你产生默认构造器。还可以更进一步自定义异常，比如加入额外的构造器和成员，以及覆盖 `Throwable` 的方法。既然异常也是对象的一种，所以可以继续修改异常类，以得到更强的功能。但是，客户端程序员可能仅仅只是查看一下抛出的异常类型，其他的就不管了（大多数 Java 库里的异常都是这么用的）。

```java
class MyException extends Exception {
	private int x;

	public MyException() {
	}

	public MyException(String msg) {
		super(msg);
	}

	public MyException(String msg, int x) {
		super(msg);
		this.x = x;
	}

	public int val() {
		return x;
	}

	@Override
	public String getMessage() {
		return "Detail Message: " + x + " " + super.getMessage();
	}
}
```

当捕获异常时，如果要将结果打印到控制台上，建议通过写入 `System.err` 而将错误发送给标准错误流，而非输出到 `System.out`；因为 `System.out` 也许会被重定向。

### 异常与记录日志

可以使用 `java.util.logging` 工具将输出记录到日志中。

更常见的情形是我们需要捕获和记录其他人编写的异常，因此我们必须在异常处理程序中生成日志消息。

```java
public class LoggingExceptions {
	private static Logger logger = Logger.getLogger("LoggingExceptions");

	static void logException(Exception e) {
		StringWriter trace = new StringWriter();
		e.printStackTrace(new PrintWriter(trace));
		logger.severe(trace.toString());
	}

	public static void main(String[] args) {
		try {
			throw new NullPointerException();
		} catch (NullPointerException e) {
			logException(e);
		}
	}
}
```

## 异常的限制

在继承过程中，编译器对异常说明有一些强制要求：

* 当覆盖方法的时候，只能抛出在基类方法的异常说明里列出的那些异常或其派生类异常。这意味着，当基类使用的代码应用到其派生类对象的时候一样能够工作（当然，这是面向对象的概念），异常也不例外。

* 当覆盖方法的时候，派生类的方法可以不抛出任何异常，即使它是基类所定义的异常。这是因为，即使基类的方法会抛出异常，这样做也不会破坏已有的程序。

* 如果一个方法同时覆盖自基类和父接口中的某个方法，且两者的方法声明抛出了不同的异常，那么这个覆盖的方法只能选择不抛出任何异常。

* 异常限制对构造器不起作用。派生类的构造器可以抛出任何异常，而不必理会基类构造器所抛出的异常。然而，因为基类构造器必须以这样或那样的方法被调用（比如默认构造器将自动被调用），派生类构造器的异常说明必须包含基类构造器的异常说明。

注意，异常说明并不属于方法类型的一部分，方法类型是由方法名称和参数类型组成的。因此，不能基于异常说明来重载方法。另外，一个出现在基类方法的异常说明中的异常，不一定会出现在派生类的异常说明中。这点同继承的规则明显不同。换句话说，在继承和覆盖的过程中，某个特定方法的“异常说明的接口”不是变大了而是变小了——这恰好和类接口在继承时的情形相反。

## 清理资源

对于一些代码，可能会希望无论 `try` 块中的异常是否抛出，它们都能得到执行。这通常适用于清理除内存之外的资源（因为内存回收由垃圾回收器完成），这些资源包括：已经打开的文件或网络连接、在屏幕上画的图形，甚至可以是外部世界的某个开关。

### 使用 finally 进行清理

为了达到清理的效果，可以在异常处理程序后面使用 `finally` 子句：

```java
try {
	// The guarded region: Dangerous activities
	// that might throw A，B or C
} catch (A a1) {
	// Handler for situation A
} catch (B b1) {
	// Handler for situation B
} catch (C c1) {
	// Handler for situation C
} finally {
	// activities that happen every time
}
```

当 Java 中的异常不允许我们回到异常抛出的地点时，那么该如何应对呢？如果把 `try` 块放在循环里，就建立了一个“程序继续执行之前必须要达到”的条件。还可以加入一个 static 类型的计数器或者别的装置，使循环在放弃以前能尝试一定的次数。这将使程序的健壮性更上一个台阶。

```java
public class FinallyWorks {
	static int count = 0;

	public static void main(String[] args) {
		while (true) {
			try {
				// Post-increment is zero first time:
				if (count++ == 0)
					throw new ThreeException();
				System.out.println("No exception");
			} catch (ThreeException e) {
				System.out.println("ThreeException");
			} finally {
				System.out.println("In finally clause");
				if (count == 2)
					break; // out of "while"
			}
		}
	}
}
```

当涉及 `break` 和 `continue` 语句时，`finally` 子句也会得到执行。注意，如果把 `finally` 子句和带标签的 `break` 和 `continue` 配置使用，在 Java 里就没有必要使用 goto 语句了。

因为 `finally` 子句总是会执行，所以在一个方法中，可以从多个点返回（return），并且可以保证重要的清理工作仍旧会执行。

### 构造器

在清理资源涉及到构造器时，可能会出现问题。构造器会把对象设置成安全的初始状态，但还会有别的动作，比如打开一个文件，这样的动作只有在对象使用完毕并且用户调用了特殊的清理方法之后才能得以清理。如果在构造器内部抛出了异常，这些清理行为也许就不能正常工作了。

并不是使用 `finally` 就能解决问题。因为 `finally` 会每次都执行清理代码。如果构造器在其执行过程中半途而废，也许该对象的某些部分还没有被成功创建，而这些部分在 `finally` 子句中却是要被清理的。

对于在构造阶段可能会抛出异常，并且要求清理的类，
最安全的使用方式是使用嵌套的 `try` 子句。嵌套的 `try` 子句是一种通用的清理惯用法，即使在构造器不抛出任何异常时也应该运用，其基本规则是：在创建需要清理的对象后，立即进入一个 `try-finally` 语句块：

```java
// 每个要清理的对象必须跟随一个 try-finally
class NeedsCleanup { // 构造不会失败
	private static long counter = 1;
	private final long id = counter++;

	public void dispose() {
		System.out.println("NeedsCleanup " + id + " disposed");
	}
}

class ConstructionException extends Exception {
	private static final long serialVersionUID = 8505343907529579918L;
}

class NeedsCleanup2 extends NeedsCleanup {
	// 构造可能失败：
	public NeedsCleanup2() throws ConstructionException {
	}
}

public class CleanupIdiom {
	public static void main(String[] args) {
		// Section 1:
		// 如果对象构造不会失败，就不需要任何 catch：
		NeedsCleanup nc1 = new NeedsCleanup();
		try {
			// ...
		} finally {
			nc1.dispose();
		}

		// Section 2:
		// 对于构造不会失败的对象，可以组合在一起：
		NeedsCleanup nc2 = new NeedsCleanup();
		NeedsCleanup nc3 = new NeedsCleanup();
		try {
			// ...
		} finally {
			nc3.dispose(); // 构造的反序
			nc2.dispose();
		}

		// Section 3:
		// 如果构造可能失败，对于每一个构造都必须包含在其自己的 try-finally 语句块中：
		try {
			NeedsCleanup2 nc4 = new NeedsCleanup2();
			try {
				NeedsCleanup2 nc5 = new NeedsCleanup2();
				try {
					// ...
				} finally {
					nc5.dispose();
				}
			} catch (ConstructionException e) { // nc5 constructor
				System.out.println(e);
			} finally {
				nc4.dispose();
			}
		} catch (ConstructionException e) { // nc4 constructor
			System.out.println(e);
		}
	}
}
```

### 异常丢失

用某些特殊的方式使用 `finally` 子句，可能会造成丢失异常的情形：

```java
try {
	LostMessage lm = new LostMessage();
	try {
		lm.f();// throws VeryImportantException
	} finally {
		lm.dispose();// throws HoHumException
	}
} catch (Exception e) {// catch HoHumException
	System.out.println(e);
}
```

一种更简单的丢失异常的方式是从 `finally` 子句中返回：

```java
try {
	throw new RuntimeException();
} finally {
	// Using 'return' inside the finally block
	// will silence any thrown exception.
	return;
}
```

### try-with-resources（TWR）

Java 7 开始支持 try-with-resources（TWR）。TWR 的基本思想是把资源（比如文件）的作用域限定在代码块内，当程序离开这个代码块时，资源会被自动关闭。

TWR 的优点在于：

*   减少编写错误代码的几率。

*   能够更准确地跟踪堆栈中的异常。

资源自动化管理代码块的基本形式是——把资源放在 `try` 的圆括号内。在这段代码块中使用的资源在处理完成后会自动关闭。

TWR 从句中出现的资源类必须实现 `AutoCloseable` 接口。注意，不是全部资源相关的类都采用这项新技术。

```java
try (OutputStream out = new FileOutputStream(file);
    InputStream is = url.openStream()) {
  byte[] buf = new byte[4096];
  int len;
  while ((len = is.read(buf)) > 0) {
    out.write(buf, 0, len);
  }
}
```

注意，要确保 try-with-resources 生效，正确的做法是为各个资源声明独立变量。

```java
// 假定文件存在，但可能不是 ObjectInputStream 类型的文件，所以文件无法正确打开。
// 因此不能构建 ObjectInputStream，所以 FileInputStream 也没法关闭。
try (ObjectInputStream in = new ObjectInputStream(
	new FileInputStream("someFile.bin"))){
	...
}

try (FileInputStream fin = new FileInputStream("someFile.bin");
	ObjectInputStream in = new ObjectInputStream(fin)){
	...
}
```