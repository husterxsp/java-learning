### 以下关于异常处理的代码有哪些问题？

```java
public static void start() throws IOException, RuntimeException {
    throw new RuntimeException("Not able to Start");
}

public static void main(String args[]) {
    try {
        start();
    } catch (Exception ex) {
        ex.printStackTrace();
    } catch (RuntimeException re) {
        re.printStackTrace();
    }
}
```

解：start方法内不会发生IOException不需要throw，另外RuntimeException为运行时异常，不需要显示抛出。

两个catch需要更换下顺序，先catch范围小的。

另外，在捕获到异常之后，不能简单的printStackTrace，需要做进一步处理。
