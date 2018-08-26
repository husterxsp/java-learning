## Java 自动装箱和拆箱

> WHY:为什么需要自动拆装箱。
> WHAT:什么是自动拆装箱。
> HOW:自动拆装箱是如何实现的。
> WHEN:什么时候会用到自动拆装箱
> WHERE:什么地方可能会自动进行自动拆装箱，如三目运算符
> OTHER:自动拆装箱可能会带来那些问题？
> 即 WWW.WHO

### 是什么

Java SE5 提供的功能，用于在 基本类型 和 包装器类型 之间相互转换。

### 什么时候发生

- 给方法传参。
- 给变量赋值。
- 运算符操作

基本类型和对应包装类型的对照表。


| Primitive type | Wrapper class |
| -------------- | ------------- |
| boolean        | Boolean       |
| byte           | Byte          |
| char           | Character     |
| float          | Float         |
| int            | Integer       |
| long           | Long          |
| short          | Short         |
| double         | Double        |

#### 内部实现

以代码为例

```java
public class Test {
    public static void main(String[] args) {
        Integer a = 1;
        int b = a;
    }
}
```

执行 `javap -c Test` 结果如下：

```java
public class Test {
  public Test();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       4: astore_1
       5: aload_1
       6: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
       9: istore_2
      10: return
}
```

可以看到，装箱过程是通过调用包装器的 `valueOf` 方法实现的，而拆箱过程是通过调用包装器的  `xxxValue `方法实现的。（xxx代表对应的基本数据类型）。



#### 缓存

以`Integer`为例，部分源代码如下：

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
	    return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。需要注意这个缓存区间的下限 -128 是固定的，但是上限可以在运行时调整 , 但上限最小是127，`Math.max(i, 127);`。



例如代码：

```java
public class Test {
    public static void main(String[] args) {
        Integer i1 = 200;
        Integer i2 = 200;

        System.out.println(i1 == i2);
    }
}
```

编译后直接执行 `java Test`输出 false。因为 200超出了 127，所以 i1, i2 是两个不同的对象。

但是执行  `java -Djava.lang.Integer.IntegerCache.high=200 Test`  则输出true。

或者执行 `java -XX:AutoBoxCacheMax=200 Test `来设置。（参考 https://stackoverflow.com/a/15052272）

注意，Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。

Double、Float的valueOf方法的实现是类似的。

为什么Double类的valueOf方法会采用与Integer类的valueOf方法不同的实现？很简单：在某个范围内的整型数值的个数是有限的，而浮点数却不是。

#### Integer i = new Integer(xxx)和Integer i =xxx; 的区别？

`Integer i = new Integer(xxx)` 没有自动装箱的过程。而且也没有用到缓存数组，所以相对来讲 `Integer i =xxx;` 性能好一些。

#### 问题

```java
Integer i1 = null;
int a = i1;
```

以上代码，编译时没有问题，但是运行时会报空指针异常NullPointerException。 因为自动拆箱会调用 `i1.intValue()`, 此时 i1 是 null。就报错了。

##### 三目运算符

 ```java
Double a = null;
boolean flag = true;
Double b = (flag) ? a : 0d;
 ```

以上代码也会报错 NullPointerException。

这是自动装箱/拆箱的特性，只要一个运算中有不同的类型，涉及到类型转换，那么编译器会往下（**基本类型**）转型，再进行运算。 就是说，如果运算中有int和Integer，Integer会先转成int再计算。

## 参考

- https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
- https://www.cnblogs.com/dolphin0520/p/3780005.html
- https://blog.csdn.net/tiwerbao/article/details/34244139
- https://www.jianshu.com/p/cc9312104876
