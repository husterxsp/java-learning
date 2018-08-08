### 源码阅读-String

以下代码阅读基于 JDK8。基本上只考虑一些public 的方法。

#### CharSequence

源代码里好多地方用到了。


#### 定义

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {}
```

`final` 修饰表示 该类不可以被继承。

`java.io.Serializable` 接口表示可以序列化。

`Comparable<String>` 接口中只有一个函数  `public int compareTo(T o);`， String 中实现如下，主要是比较两个字符串：

```java
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}

// 忽略大小写地比较
public int compareToIgnoreCase(String str) {}
```

函数中先逐个比较是否有不同的字符，如果有不同的字符直接返回 `c1 - c2`。如果比较到最后都相等，则比较长度 `len1 - len2`。

#### 属性

```java
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0

/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = -6849794470754667710L;
```

`value` 字符数组（java10已经改成 `byte[]` 了） 是 String 用来存储字符串的。

`hash` 表示该字符串的hashcode。对应的 `hashCode` 方法如下, 该方法是重写了继承自 `Object` 的 方法 `public native int hashCode();`。

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

`serialVersionUID` 是一个标识，主要用于反序列化。这里有用吗？

#### 构造函数

##### 使用字符串 和字符数组构造

```java
public String() {
    this.value = "".value;
}
```
即直接调用 new String() 会创建一个字符串 ""

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```
根据已有的字符串来创建新的，直接拷贝 `value` 和 `hash`, 由上面的 `hashCode()` 可以知道 `hash` 值 是直接根据 `value` 计算出来的，所以 拷贝了`value`后可以直接拷贝 `hash`。

```java
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```
这里 `Arrays.copyOf` 会新创建一个数组对象并返回。

```java
public String(char value[], int offset, int count) {}
```
根据 value 数组的 `[offset, offset+count)` 范围的字符串创建。


```java
String(char[] value, boolean share) {
    // assert share : "unshared not supported";
    this.value = value;
}
```
这个 protected 的构造函数 和 `public String(char value[]) {}` 主要区别是没有 `Arrays.copyOf` 的过程了，效率更高一些。主要是包内部使用，一些函数中创建了一些字符数组后，这些数组没有其他的用途了，可以直接用来创建新的String对象。

##### ？

```java
public String(int[] codePoints, int offset, int count) {}
```

##### 使用字节数组构造

```java
public String(byte bytes[], int offset, int length, String charsetName) throws UnsupportedEncodingException {}
public String(byte bytes[], int offset, int length, Charset charset) {}
public String(byte bytes[], String charsetName) throws UnsupportedEncodingException {}
public String(byte bytes[], Charset charset) {}
public String(byte bytes[], int offset, int length) {}
public String(byte bytes[]) {}
```

##### 使用StringBuilder 和 StringBuffer构造

```java
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
public String(StringBuilder builder) {
    this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```
StringBuffer 类里面的大多数方法都是 `synchronized` 的，保证了线程安全性，但是也牺牲了一定的性能。

##### 一些常用的函数

```java
public int length() {
    return value.length;
}
```
返回字符串长度。因为String内部用的是 字符数组，所以没有必要再给String定义一个length属性，直接定义一个方法，返回 value 字符数组的长度属性即可。

```java
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}
// 拷贝字符串的一部分到 目标字符数组
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
    if (srcBegin < 0) {
        throw new StringIndexOutOfBoundsException(srcBegin);
    }
    if (srcEnd > value.length) {
        throw new StringIndexOutOfBoundsException(srcEnd);
    }
    if (srcBegin > srcEnd) {
        throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
    }
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
}
// 返回字节数组 不是很懂
public byte[] getBytes(String charsetName){}
public byte[] getBytes(Charset charset) {}

// 判断字符串的字符数组是否完全一样
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}

// 判断字符串区间是否相等
// this.charAt(toffset+k) != other.charAt(ooffset+k)
public boolean regionMatches(int toffset, String other, int ooffset, int len) {}
// if ignoreCase == true, 调用 toUpperCase 和 toLowerCase 再比较
public boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len) {}

// 判断在指定偏移位置 是否有某个前缀
public boolean startsWith(String prefix, int toffset) {}
public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}
public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}

// 从 fromIndex 开始查找第一个 ch 字符的位置
public int indexOf(int ch, int fromIndex) {}
public int indexOf(int ch) {
    return indexOf(ch, 0);
}

// 从 fromIndex 开始 反向 查找第一个 ch 字符的位置
public int lastIndexOf(int ch, int fromIndex) {}
public int lastIndexOf(int ch) {
    return lastIndexOf(ch, value.length - 1);
}
```

以下的几个方法与上面的类似，是查找字符串的位置

```java
public int indexOf(String str) {
    return indexOf(str, 0);
}
public int indexOf(String str, int fromIndex) {
    return indexOf(value, 0, value.length, str.value, 0, str.value.length, fromIndex);
}

static int indexOf(char[] source, int sourceOffset, int sourceCount, String target, int fromIndex) {
    return indexOf(source, sourceOffset, sourceCount, target.value, 0, target.value.length, fromIndex);
}
static int indexOf(char[] source, int sourceOffset, int sourceCount,
                   char[] target, int targetOffset, int targetCount, int fromIndex) {}

public int lastIndexOf(String str) {}
public int lastIndexOf(String str, int fromIndex) {}
static int lastIndexOf(char[] source, int sourceOffset, int sourceCount, String target, int fromIndex) {}
static int lastIndexOf(char[] source, int sourceOffset, int sourceCount,
                       char[] target, int targetOffset, int targetCount, int fromIndex) {
```

substring 从给定的位置开始创建一个新的字符串返回。注意java7中还是复用原有的字符数组，但是这种会 造成内存泄露。如果有一个很长的字符串，那么这个字符串会一直被占用。导致无法进行垃圾回收。

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
public String substring(int beginIndex, int endIndex) {}
```

```java
// 连接字符串
public String concat(String str) {}
// 字符串替换
public String replace(char oldChar, char newChar) {}
// 字符串正则匹配
public boolean matches(String regex) {
    return Pattern.matches(regex, this);
}
// 是否包含某个子串
public boolean contains(CharSequence s) {
    return indexOf(s.toString()) > -1;
}
// 正则匹配并替换，找到第一个 满足 regex 的子串并替换。
public String replaceFirst(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
}
public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
public String replace(CharSequence target, CharSequence replacement) {
    return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(this).
        replaceAll(Matcher.quoteReplacement(replacement.toString()));
}
```

分割字符串，limit 控制正则表达式被应用的次数.

- 如果 n 是正数，正则匹配至多进行 n - 1 次，结果数组的长度不会超过 n 。
- 如果 n 是负数，正则匹配会进行多次，结果数组可以任意长度。
- 如果 n 是 0 ，正则匹配会进行多次，结果和负数相比只是删去了 数组末尾的 空字符 "" 。

 If the limit n is greater than zero then the pattern will be applied at most n - 1 times, the array's length will be no greater than n, and the array's last entry will contain all input beyond the last matched delimiter.
 If n is non  then the pattern will be applied as many times as possible and the array can have any length.
 If n is zero then the pattern will be applied as many times as possible, the array can have any length, and trailing empty strings will be discarded.

```java
public String[] split(String regex, int limit) {}
public String[] split(String regex) {
    return split(regex, 0);
}
```

以 "boo:and:foo", 为例，结果如下：

:	2	{ "boo", "and:foo" }
:	5	{ "boo", "and", "foo" }
:	-2	{ "boo", "and", "foo" }
o	5	{ "b", "", ":and:f", "", "" }
o	-2	{ "b", "", ":and:f", "", "" }
o	0	{ "b", "", ":and:f" }


字符串通过给定的分隔符，进行拼接

```java
// String message = String.join("-", "Java", "is", "cool");
// message returned is: "Java-is-cool"
public static String join(CharSequence delimiter, CharSequence... elements) {}
// join 的对象是迭代器对象。
// List<String> strings = new LinkedList<>();
// strings.add("Java");strings.add("is");
// strings.add("cool");
// String message = String.join(" ", strings);
// message returned is: "Java is cool"

public static String join(CharSequence delimiter, Iterable<? extends CharSequence> elements) {}
```


大小写转换，locale 对象用于指明本地语言的转换规则。
```java
public String toLowerCase(Locale locale) {}
public String toLowerCase() {
    return toLowerCase(Locale.getDefault());
}
public String toUpperCase(Locale locale) {}
public String toUpperCase() {
    return toUpperCase(Locale.getDefault());
}
```

删除字符串两边的空格。

```java
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```

```java
public String toString() {
    return this;
}
// 创建新的字符数组返回
public char[] toCharArray() {
    // Cannot use Arrays.copyOf because of class initialization order issues
    char result[] = new char[value.length];
    System.arraycopy(value, 0, result, 0, value.length);
    return result;
}
```

格式化字符串
```java
public static String format(String format, Object... args) {
    return new Formatter().format(format, args).toString();
}
public static String format(Locale l, String format, Object... args) {}
```


获取对象对应的字符串，如果有 `toString` 就调用，如果没有，如 `char[]`, 则 通过 `new String()` 创建。

```java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
public static String valueOf(char data[]) {
    return new String(data);
}
public static String valueOf(char data[], int offset, int count) {
    return new String(data, offset, count);
}
// 这两个没用的？
public static String copyValueOf(char data[], int offset, int count) {
    return new String(data, offset, count);
}
public static String copyValueOf(char data[]) {
    return new String(data);
}
public static String valueOf(boolean b) {
    return b ? "true" : "false";
}
public static String valueOf(char c) {
    char data[] = {c};
    return new String(data, true);
}
public static String valueOf(int i) {
    return Integer.toString(i);
}
public static String valueOf(long l) {
    return Long.toString(l);
}
public static String valueOf(float f) {
    return Float.toString(f);
}
public static String valueOf(double d) {
    return Double.toString(d);
}
```

#### intern() 方法

```
public native String intern();
```

### 参考

- http://www.hollischuang.com/archives/1330
- https://www.baeldung.com/java-9-compact-string
- https://docs.oracle.com/javase/8/docs/api/java/lang/String.html
