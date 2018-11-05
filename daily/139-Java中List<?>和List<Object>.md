### Java中`List<?>`和`List<Object>`之间的区别是什么?

List<?> 是一个未知类型的List，而List<Object> 其实是任意类型的List。

可以把List<String>, List<Integer>赋值给List<?>，却不能把List<String>赋值给 List<Object>。


https://docs.oracle.com/javase/tutorial/extra/generics/wildcards.html
