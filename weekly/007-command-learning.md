常用Java命令学习——javac、javap、jps、jstack。

### javac

编译源代码

### javap

http://www.hollischuang.com/archives/1107

可以对代码反编译，也可以查看java编译器生成的字节码。

默认 `javap Hello` 相当于 `javap -package Hello` 显示程序包/protected/public类和成员

`javap -c Hello` 查看编译后的字节码。 ？？？

可以运行 `javap -help` 查看帮助

### jps

http://www.hollischuang.com/archives/105

查找当前用户的Java进程

### jstack

http://www.hollischuang.com/archives/110

主要用来查看Java线程的调用堆栈的，可以用来分析线程问题（如死锁）

### 参考

- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/javac.html
- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/javap.html
- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html
