### Java和C++同为面向对象语言，Java和C++主要区别有哪些？双方个有哪些优缺点？

C++ 被设计成主要用在系统性应用程序设计上的语言，对C语言进行了扩展。对于C语言这个为运行效率设计的过程式程序设计语言, C++ 特别加上了以下这些特性的支持：静态类型的面向对象程序设计的支持、异常处理、RAII以及泛型。另外它还加上了一个包含泛型容器和算法的C++库函数。

Java 最开始是被设计用来支持网络计算。它依赖一个虚拟机来保证安全和可移植性。Java包含一个可扩展的库用以提供一个完整的的下层平台的抽象。Java是一种静态面向对象语言，它使用的语法类似C++，但与之不兼容。为了使更多的人到使用更易用的语言，它进行了全新的设计。

其实，最主要的区别是他们分别代表了两种类型的语言：

C++是编译型语言（首先将源代码编译生成机器语言，再由机器运行机器码），执行速度快、效率高；依赖编译器、跨平台性差些。
Java是解释型语言（源代码不是直接翻译成机器语言，而是先翻译成中间代码，再由解释器对中间代码进行解释运行。），执行速度慢、效率低；依赖解释器、跨平台性好。

PS：也有人说Java是半编译、半解释型语言。Java 编译器(javac)先将java源程序编译成Java字节码(.class)，JVM负责解释执行字节码文件。

二者更多的主要区别如下：
C++是平台相关的，Java是平台无关的。
C++对所有的数字类型有标准的范围限制，但字节长度是跟具体实现相关的，不同操作系统可能。Java在所有平台上对所有的基本类型都有标准的范围限制和字节长度。
C++除了一些比较少见的情况之外和C语言兼容 。 Java没有对任何之前的语言向前兼容。但在语法上受 C/C++ 的影响很大
C++允许直接调用本地的系统库 。 Java要通过JNI调用, 或者 JNA
C++允许过程式程序设计和面向对象程序设计 。Java必须使用面向对象的程序设计方式
C++支持指针，引用，传值调用 。Java只有值传递。
C++需要显式的内存管理，但有第三方的框架可以提供垃圾搜集的支持。支持析构函数。 Java 是自动垃圾收集的。没有析构函数的概念。
C++支持多重继承，包括虚拟继承 。Java只允许单继承，需要多继承的情况要使用接口。
。。。