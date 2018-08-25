### 什么是枚举？

```java
public enum Season {
    SPRING, SUMMER, AUTUMN, WINER;
}
```

上述代码运行 `java -jar cfr_0_127.jar --sugarenums false Season` 反编译结果如下：

```java
/*
 * Decompiled with CFR 0_127.
 */
public final class Season
extends Enum<Season> {
    public static final /* enum */ Season SPRING = new Season();
    public static final /* enum */ Season SUMMER = new Season();
    public static final /* enum */ Season AUTUMN = new Season();
    public static final /* enum */ Season WINER = new Season();
    private static final /* synthetic */ Season[] $VALUES;

    public static Season[] values() {
        return (Season[])$VALUES.clone();
    }

    public static Season valueOf(String string) {
        return Enum.valueOf(Season.class, string);
    }

    private Season() {
        super(string, n);
    }

    static {
        $VALUES = new Season[]{SPRING, SUMMER, AUTUMN, WINER};
    }
}
```

可见 enum 内部实现其实也是用的 `static final` ，只是JDK5 提供的一个语法糖。

枚举 和  `public static final` 的区别?

### 参考

- http://www.hollischuang.com/archives/195
