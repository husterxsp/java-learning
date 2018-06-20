### java7关于exception的一些新特性除了try-with-resource还有哪些？

#### 1. Multi-Catch Exceptions

一个catch语句可以同时处理多个异常，使用或 | 分割

```java
catch (IOException | SQLException ex) {
    logger.log(ex);
    throw ex;
}
```

在java7之前如果需要捕获多个exception需要如下这样写，比较繁琐。

```java
catch(ParseException exception) {

}
catch(IOException exception) {

}
```

当然还可以直接使用Exception来捕获所有类型的异常，代码如下，但是这样不容易调试，课能会捕获我们不需要的异常

```java
catch(Exception exception) {
    // I am an inexperienced or lazy programmer here.
}
```

#### 2. Rethrowing Exceptions

如果想再次throw一个已经捕获的错误可能会写出如下代码：
```java
// example1
public class ExampleExceptionRethrowInvalid {
    public static void demoRethrow() throws IOException {
        try {
            throw new IOException("error");
        } catch (Exception exception) {
            throw exception;
        }
    }
    public static void main(String[] args) {
        try {
            demoRethrow();
        } catch (IOException exception) {
            System.err.println(exception.getMessage());
        }
    }
}
```

但是这样写，用java6其实就编译不过。一种解决办法是在包裹一层RuntimeException, 但是这样就需要注意再次抛出的异常就不是原有的异常了。
注意和上面的代码相比，(省去了 demoRethrow() 方法的 throw ?)

```java
public class ExampleExceptionRethrowOld {
    public static void demoRethrow() {
        try {
            throw new IOException("error");
        } catch (Exception exception) {
            throw new RuntimeException(exception);
        }
    }

    public static void main(String[] args) {
        try {
            demoRethrow();
        } catch (RuntimeException exception) {
            System.err.println(exception.getMessage());
        }
    }
}
```

java7 则是提供了这么一个re-throw 的语法吧，ExampleExceptionRethrowInvalid的代码用java7时可以编译通过的。


### 参考

- http://www.oracle.com/technetwork/articles/java/java7exceptions-486908.html
- https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html
- https://donaldojdk.files.wordpress.com/2011/11/55j7.pdf
