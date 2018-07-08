### 什么是try-with-resources?

https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html

java7特性。try-with-resources 语句是 try 语句里声明了一个或多个resources。resource对象在程序使用完后需要close。try-with-resources语句就是确保每个resource在最后都被关闭。任何实现了 java.lang.AutoCloseable 接口的对象都能被当做resource。当然包括也实现Closeable接口的。

```java
public interface Closeable extends AutoCloseable {

    /**
     * Closes this stream and releases any system resources associated
     * with it. If the stream is already closed then invoking this
     * method has no effect.
     *
     * <p> As noted in {@link AutoCloseable#close()}, cases where the
     * close may fail require careful attention. It is strongly advised
     * to relinquish the underlying resources and to internally
     * <em>mark</em> the {@code Closeable} as closed, prior to throwing
     * the {@code IOException}.
     *
     * @throws IOException if an I/O error occurs
     */
    public void close() throws IOException;
}
```

如下代码读取文件的第一行，其中 BufferedReader 就是一个需要被关闭的对象。（FileReader呢?需要显式关闭吗？）
此处即使br.readLine()抛出异常，BufferedReader 实例对象也会正确关闭。

```java
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

以上这段代码反编译如下：
```java
static String readFirstLineFromFile(String var0) throws IOException {
    BufferedReader var1 = new BufferedReader(new FileReader(var0));
    Throwable var2 = null;

    String var3;
    try {
        var3 = var1.readLine();
    } catch (Throwable var12) {
        var2 = var12;
        throw var12;
    } finally {
        if (var1 != null) {
            if (var2 != null) {
                try {
                    var1.close();
                } catch (Throwable var11) {
                    var2.addSuppressed(var11);
                }
            } else {
                var1.close();
            }
        }

    }

    return var3;
}
```

但是在java7以前，则可以用finally来关闭，代码如下。
注意，如果readLine()和close() 都抛出异常的话，readFirstLineFromFileWithFinallyBlock方法只会抛出finally里的异常，而try里的异常被覆盖。
相反，在上面的readFirstLineFromFile方法中，try语句块的异常会覆盖try-with-resources语句中的异常。

```java
static String readFirstLineFromFileWithFinallyBlock(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        if (br != null) br.close();
    }
}
```


如果 try-with-resources 语句包含了多个声明，以分号分隔。close方法会按照和声明相反的顺序调用。

try-with-resources 语句也可以有catch 和finally, 但是需要注意 catch or finally 在声明的resource关闭之后才执行。

#### Suppressed Exceptions

上面说到有多个异常的时候可能有抑制的情况，可以通过 Throwable.getSuppressed 来获取这些被抑制的异常。当然需要先 addSuppressed

好像是java7 编译之后会自动调用addSuppressed的，那么我们应该可以直接get

### 参考

- https://blog.csdn.net/u013435893/article/details/79611608
- https://juejin.im/entry/57f73e81bf22ec00647dacd0
