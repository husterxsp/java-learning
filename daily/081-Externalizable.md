### Serializable 和 Externalizable 接口有何不同？

Externalizable 是 Serializable的子类，有两个抽象方法需要实现 writeExternal()和readExternal()，开发者有更多的控制，可以自定义需要序列化的属性。
