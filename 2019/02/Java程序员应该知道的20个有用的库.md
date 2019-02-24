# Java程序员应该知道的20个有用的库
一个优秀且经验丰富的Java开发人员的特点之一是对API的广泛了解，包括JDK和第三方库。我花了很多时间学习API，特别是在阅读Effective Java 3rd Edition之后，Joshua Bloch建议如何使用现有的API进行开发，而不是为常用的东西写新的代码。
在本文中，我将分享一些Java开发人员应该熟悉的最有用和最重要的库和API。但是，我没有包含框架，例如Spring和Hibernate，因为它们非常有名且具有特定功能。
总的来说，我在日常项目包含了有用的库，包括Log4j日志库，Jackson JSON解析库，以及JUnit和Mockito等单元测试API。如果需要在项目中使用，则在项目的classpath包含这些JAR，也可以使用Maven进行依赖管理。
当你使用Maven进行依赖管理时，它会自动下载这些库，包括它们所依赖的库，称为传递依赖。
例如，如果你下载Spring Framework，它还将下载Spring所依赖的所有其他JAR，例如Log4j。
你可能没注意到，但有正确版本的JAR是一个令人头疼的问题。如果是错误的JAR版本，那么你将遇到 ClassNotFoundException， NoClassDefFoundError或 UnsupportedClassVersionError。

# Java程序员20个有用的开源库
这是我收集的一些有用的第三方库，Java开发可以使用它们在应用中来完成许多有用的功能。要使用这些库，Java开发人员应该熟悉它，这就是本文的重点。如果你觉得有用，你可以研究该库并使用它。

1.日志库
日志库非常常见，因为在每个项目中都需要它们。它们是服务器端应用最重要的东西，因为日志只放在可以看到应用程序当前运行时情况的地方。尽管JDK附带了自己的日志库，但还有更好的替代方案，例如Log4j，SLF4j和LogBack。
![image.png](0)
Java开发人员应该熟悉日志库的优缺点，并且知道 为什么使用SLF4j比普通Log4j更好。

2. JSON解析库
在当今的Web服务和物联网领域，JSON已成为将信息从客户端传送到服务器的首选协议。他们已经替换XML成为在独立平台间传输信息的最佳方式。
遗憾的是，JDK没有JSON库。但是，有许多优秀的第三方库允许你解析和创建JSON消息，如Jackson和Gson。
Java Web开发人员应该熟悉这些库中的至少一个。如果你想了解有关Jackson和JSON的更多信息，我建议你看看 Udemy的课程JSON with the Java API。

3.单元测试库
单元测试是将普通开发人员与优秀开发人员区分开来的最重要的事情。程序员经常有理由不写单元测试，但逃避写单元测试的最常见的借口是缺乏常用单元测试库的经验和知识，包括JUnit，Mockito和PowerMock。

我在2018年有一个目标就是提高我对单元测试和集成测试库的了解，比如JUnit 5，Cucumber，Robot框架和一些其他的。
我还在Udemy注册了 JUnit and Mockito Crash Course 。即使你了解JUnit和单元测试的基础知识，可能也希望更新并进阶自己的知识。

4.通用库
Java开发人员可以使用几个很好的通用第三方库，比如Apache Commons和Google Guava。我总是在我的项目中包含这些库，因为它们简化了很多功能。
正如Joshua Bloch在Effective Java中所说的那样，重复造轮子是没有意义的。我们应该更偏向于使用久经考验的库而不是时不时自己来实现。

对Java开发人员来说，熟悉Google Guava和Apache Commons库是件好事。

5. HTTP库
我不喜欢JDK的一点是他们对HTTP支持的缺乏。虽然你可以使用java.net包中的类建立HTTP连接 ，但使用开源的第三方库（如Apache HttpClient和HttpCore）并不容易或不能无缝结合。
虽然JDK 9带来了HTTP 2.0的支持和更好的HTTP支持，但我强烈建议所有Java开发人员熟悉流行的HTTP客户端库，包括HttpClient和HttpCore。
你还可以查看此文章What's New in Java 9 - Modules and More以了解有关JDK 9对HTTP 2支持的更多信息。

6. XML解析库
有许多XML解析库，包括Xerces，JAXB，JAXP，Dom4j和Xstream。Xerces2是Apache Xerces下一高性能版本，完全兼容的XML解析器。这个新版本的Xerces引入了Xerces Native Interface（XNI），这是一个完整的框架，用于构建非常模块化且易于编程的解析器组件和配置。

Apache Xerces2解析器是XNI的参考实现，但是其他解析器组件，配置和解析器可以使用Xerces Native Interface编写。Dom4j是另一个适用于Java应用程序的灵活XML框架。如果你想了解有关Java中XML解析的更多信息，建议你查看Udemy 上的 Java Web Services and XML 在线课程。

7. Excel库
信不信由你 - 所有现实世界的应用程序都必须以某种形式与Microsoft Office进行交互。许多应用程序需要提供在Excel中导出数据的功能，如果必须从Java应用程序执行相同操作，则需要Apache POI API。

这是一个非常丰富的库，允许你 从Java程序读取和写入XLS文件。你可以看到该链接（http://www.java67.com/2014/09/how-to-read-write-xlsx-file-in-java-apache-poi-example.html），以获取在核心Java应用程序中读取Excel文件的工作示例。

8.字节码库
如果你正在编写生成代码或与字节码交互的框架，那么你需要一个字节码库。
它们允许你读取和修改应用程序生成的字节码。Java世界中一些流行的字节码库是javassist和Cglib Nodep。

Javassist（Java programming assistant）使Java字节码操作变得非常简单。它是一个用于在Java中编辑字节码的类库。ASM是另一个有用的字节码编辑库。如果你不熟悉字节码，我建议你查看Introduction to Java Programmers以了解有关它的更多信息。

9.数据库连接池库
如果你正在从Java应用程序与数据库交互但不使用数据库连接池库，那么你将丢失一些内容。
由于在运行时创建数据库连接需要花费时间并使请求处理速度变慢，因此始终建议使用数据库连接库。一些流行的是Commons Pool和DBCP。
在Web应用程序中，它的Web服务器通常提供这些功能，但在核心Java应用程序中，你需要将这些连接池库包含在类路径中以使用数据库连接池。
如果你想了解有关JDBC和Web应用程序中的连接池的更多信息，我建议你查看Udemy 中的JSP, Servlet, and JDBC for Beginners课程。

10.消息传递库
与日志记录和数据库连接类似，消息传递也是许多现实世界Java应用程序的常见功能。
Java提供的JMS，Java Messaging Service不属于JDK。对于此组件，你需要包含一个单独的组件 jms.jar。
同样，如果你正在使用第三方消息传递协议（如Tibco RV），则需要使用第三方JAR tibrv.jar 放在应用程序类路径中。

11. PDF库
与Microsoft Excel类似，PDF库是另一种普遍存在的格式。如果你需要在应用程序中支持PDF功能，例如 导出数据到PDF文件，则可以使用iText和Apache FOP库。
两者都提供有用的PDF相关功能，但iText更丰富，更好。请参阅此处以了解有关iText的更多信息。

12.日期和时间库
在Java 8之前，JDK的数据和时间库有很多缺陷，因为它们不是线程安全的，不可变的，并且容易出错。许多Java开发人员依靠JodaTime来实现他们的日期和时间要求。
从JDK 8开始，没有理由使用Joda，因为你在JDK 8的新日期和时间API中获得了所有功能，但如果你使用的是较旧的Java版本，那么JodaTime是一个值得学习的库。
如果你想了解有关新的日期和时间API的更多信息，我建议你查看Udemy上的What's new in Java 8课程。它提供了Java 8所有重要功能的精彩概述，包括日期和时间API。

13.Collection库
尽管JDK拥有丰富的集合库，但仍有一些第三方库提供了更多选项，例如Apache Commons集合，Goldman Sachs集合，Google集合和Trove。
Trove库特别有用，因为它为Java提供了高速的常规和原始集合。

FastUtil是另一个类似的API。它通过提供特定类型的映射，集合，列表和优先级队列来扩展Java集合框架，较小的内存占用，快速访问和插入; 它还提供大型（64位）数组，集和列表，以及用于二进制和文本文件快速实用的I / O类。

14.Email API
javax.mail和Apache Commons Email都提供了一个用于从Java发送电子邮件的API 。它建立在JavaMail API的基础之上，旨在简化它。

15. HTML解析库
与JSON和XML类似，HMTL是我们许多人必须处理的另一种常见格式。值得庆幸的是，我们有JSoup，它极大地简化了在Java应用程序中使用HTML的过程。
你不仅可以使用JSoup解析HTML，还可以创建HTML文档

它提供了一个非常方便的API，用于提取和操作数据，使用DOM，CSS和类似jquery的方法。JSoup实现了WHATWG HTML5规范，并将HTML解析到同一个DOM，就像现代浏览器一样。

16.Cryptographic库
Apache Commons Codec软件包包含各种格式的简单编码器和解码器，如Base64和Hexadecimal。
除了这些广泛使用的编码器和解码器之外，编解码器包还维护一组语音编码实用程序。

17.Embedded SQL Database库
我真的很喜欢像H2这样的内存数据库，你可以将它嵌入你的Java应用程序中。它们非常适合测试SQL脚本和运行需要数据库的单元测试。但是，H2并不是唯一的DB，你也可以选择Apache Derby和HSQL。

18. JDBC问题排查库
存在一些很好的JDBC扩展库，可以使调试更容易，比如P6spy。
这是一个库，可以无缝地拦截和记录数据库数据，而无需对应用程序进行代码更改。你可以使用它们来记录SQL查询及其计时。
例如，如果你在代码中使用PreparedStatment和CallableStatement，则这些库可以记录一次完全调用的参数和执行所花费的时间。

如果你想了解有关JDBC的更多信息，可以查看JDBC for Beginners。

19.序列化库
Google Protocol Buffers是一种以高效可扩展的格式编码结构化数据的方法。它是Java序列化的更丰富，更好的替代品。我强烈建议有经验的Java开发人员学习Google Protobuf。你可以查看此文章以了解有关Google协议缓冲区的更多信息 。

20.网络库
一些有用的网络库是Netty和Apache MINA。如果你正在编写需要执行底层网络任务的应用程序，请考虑使用这些库。如果你想了解有关Java网络编程的更多信息，请查看 Java Network Programming - TCP/IP Socket Programming。

这些就是对于每个Java开发人员都应该使用的一些有用的库。Java的世界是浩瀚无穷的，你会发现数不胜数的库用于做不同的事情。
如果你想用Java做任何事情，很可能你会找到一个如何实现的库。与往常一样，Google是你找到有用的Java库的最好朋友，但你也可以查看Maven中央存储库，找到适合你手头任务的一些有用的库。