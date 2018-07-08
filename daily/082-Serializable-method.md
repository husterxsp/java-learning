### Serializable 接口有几个方法? 如果没有方法，那么为什么要定义这样的接口？Java又是如何保证必须实现了Serializable才可以被序列化的呢？

Serializable定义如上，就是一个空接口，没有任何方法。目的是为了作为一个可序列化的标识。

Java是通过在序列化操作过程中对类型进行检查，要求被序列化的类必须属于Enum、Array和Serializable类型其中的任何一种，来保证实现Serializable接口的可被序列化。

### 参考

- [http://www.hollischuang.com/archives/1140#What%20Serializable%20Did](http://www.hollischuang.com/archives/1140#What%20Serializable%20Did)
