### 元注解

java 有三种标准注解，和四种元注解。元注解负责注解其他注解。

- @Target：表示注解可以用在什么地方。

- @Retention：表示需要在什么级别保存该注解信息。Retention的字面意思是保留，即定义了该Annotation被保留的时间长短：

  - 某些Annotation仅出现在源代码中，而被编译器丢弃；
  - 而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，
  - 而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

- @Document：将此注解包含在javadoc中。

- @Inherited：允许子类继承父类中的注解。


### 参考

- http://www.cnblogs.com/peida/archive/2013/04/24/3036689.html
- think in java







