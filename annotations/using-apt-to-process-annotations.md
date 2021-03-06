# 使用 apt 处理注解

如果你找不到可用的类库，那就只能自己创建新的注解以及相应的处理器。使用 apt 工具，程序员可以同时编译新产生的源文件，以及简化构建过程。不过就目前的情况看，mirror API 只能给予你一些基本功能，帮助你找到 Java 类定义中的元素。另外，Javassist 能够用来操作字节码，或者你也可以编写自己的字节码操作工具。

-----------------------------------------------------------------------------------------

注解处理工具 apt（annotation processing tool） 是 Sun 为了帮助注解的处理过程而提供的工具。

与 javac 一样，apt 被设计为操作 Java 源文件，而不是编译后的类。默认情况下，apt 会在处理完源文件后编译它们。如果在系统构建的过程中会自动创建一些新的源文件，那么这个特性会非常有用。事实上，apt 会检查新生成的源文件中的注解，然后将所有文件一同编译。

当注解处理器生成一个新的源文件时，该文件会在新一轮（round）的注解处理中接受检查。该工具会一轮一轮地处理，直到不再有新的源文件产生为止。然后它再编译所有的源文件。

程序员自定义的每一个注解都需要自己的处理器,而 apt 工具能够很容易地将多个注解处理器组合在一起。有了它，程序员就可以指定多个要处理的类，这比程序员自己遍历所有的类文件简单多了。此外还可以添加监听器，并在一轮注解处理过程结束的时候收到通知消息。
	
通过使用 `AnnotationProcessorFactory`，apt 能够为每一个它发现的注解生成一个正确的注解处理器。当你使用 apt 时，必须指明一个工厂类，或者指明能找到 apt 所需的工厂类的路径。

使用 apt 生成注解处理器时，我们无法利用 Java 的反射机制，因为我们操作的是源代码，而不是编译后的类。使用 mirror API 能够解决这个问题，它使我们能够在未经编译的源代码中查看方法、域以及类型。可以将 tools.jar 设置在你的 classpath 中，这个工具类库同时包含了 `com.sun.mirror.*` 接口。

通过实现 `com.sun.mirror.apt.AnnotationProcessor` 接口，我们可以实现一个注解处理器。

*   处理器的所有工厂都在 `process()` 中完成。

*   处理器类的构造器以 `AnnotationProcessorEnvironment` 对象为参数。通过该对象，我们就能知道 apt 正在处理的所有类型（类定义），并且可以通过它获得 `Messager` 对象和 `Filer` 对象。`Messager` 对象可以用来向用户报告信息，比如处理过程中发生的任何错误，以及错误在源代码中的位置等。`Filer` 是一种 `PrintWriter`，我们可以通过它创建新的文件。不使用普通的 `PrintWriter` 而使用 `Filer` 对象的主要原因是，这样 apt 才能知道我们创建的文件，从而对新文件进行注解处理，并且在需要的时候编译它。

apt 工具还需要一个工厂类来为其指明正确的处理器，然后它才能调用处理器上的 `process()`。通过实现 `com.sun.mirror.apt.AnnotationProcessorFactory` 接口可以实现一个处理器工厂。

**将观察者模式用于 apt**

mirror API 提供了对访问者设计模式的支持。这是一种具备扩展能力的解决方案。当你的注解处理器的复杂性越来越高的时候，如果还是编写自己独立的处理器，那么很快你的处理器就将变得非常复杂。