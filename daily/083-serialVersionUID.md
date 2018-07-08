###  serialVersionUID 有何用途? 如果没定义会有什么问题？

相当于一个标识，如果没有，对于实现了java.io.Serializable接口的类，java会根据编译的class自动生成，如果在序列化和反序列化的中间，class没有改变，那么标识不会改变，反序列化也就会成功。

但是如果在反序列化的时候，如果原先的class 增加或者删除了一些属性之类的，就会导致反序列化失败，InvalidCastException。

所以一般手动设置一个 serialVersionUID， 这样，即使修改了类，也能反序列化成功。

### 参考

- [对象序列化为何要定义serialVersionUID - iteye.com](http://lenjey.iteye.com/blog/513736)
