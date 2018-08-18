### 源码阅读-String

以下代码阅读基于 JDK8。

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

`CharSequence` 接口，即字符串序列。String, StringBuffer, StringBuilder 都实现了该接口。

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
计算方式 `s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`。 为啥这么算？参考 《Effective java》第三章 第9条。
一个好的散列函数倾向于 “为不相等的对象产生不相等的散列码”。
31 有个很好的特性，即用以为和减法来代替乘法，可以得到更好的性能。`31*i == (i<<5)-i`。java虚拟机可以自动完成这种优化。

[为什么用偶数会导致信息丢失？](https://www.zhihu.com/question/24381016/answer/112861010)

这里为啥要执行 `char val[] = value;` 对value再进行一次拷贝呢？

`serialVersionUID` 是一个标识，主要用于反序列化。

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

##### unicode 相关

```java
public String(int[] codePoints, int offset, int count) {}
```

以上代码使用 unicode编码的整型数组来创建字符串。举例如下：

```java
String ustr = new String(new int[] {0x5b57, 0x7b26, 0x7f16, 0x7801}, 0, 4);  // "字符编码"(\u5b57是'字'的unicode编码)。0表示起始位置，4表示长度。
System.out.printf("ustr=%s\n", ustr); //ustr=字符编码

// 其他相关的 API：
//  获取位置0的元素对应的unciode编码
System.out.printf("%-30s = 0x%x\n", "ustr.codePointAt(0)", ustr.codePointAt(0));

// 获取位置2之前的元素对应的unciode编码
System.out.printf("%-30s = 0x%x\n", "ustr.codePointBefore(2)", ustr.codePointBefore(2));

// 获取位置1开始偏移2个代码点的索引
System.out.printf("%-30s = %d\n", "ustr.offsetByCodePoints(1, 2)", ustr.offsetByCodePoints(1, 2));

// 获取第0~3个元素之间的unciode编码的个数
System.out.printf("%-30s = %d\n", "ustr.codePointCount(0, 3)", ustr.codePointCount(0, 3));

// 结果：
// ustr=字符编码
// ustr.codePointAt(0)            = 0x5b57
// ustr.codePointBefore(2)        = 0x7b26
// ustr.offsetByCodePoints(1, 2)  = 3
// ustr.codePointCount(0, 3)      = 3
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
// 根据字符编码 返回字节数组
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

```java
public String[] split(String regex, int limit) {}
public String[] split(String regex) {
    return split(regex, 0);
}
```

以 "boo:and:foo", 为例，结果如下：

regex | limit  | result
--- | --- | ---
:	| 2	| { "boo", "and:foo" }
:	| 5	| { "boo", "and", "foo" }
:	| -2| { "boo", "and", "foo" }
o	| 5	| { "b", "", ":and:f", "", "" }
o	| -2| { "b", "", ":and:f", "", "" }
o	| 0	| { "b", "", ":and:f" }

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
```

```java
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

执行 intern() 的时候会先检查字符串常量池中是否有对应的 字符串，如果有，则返回该地址，如果没有，则在常量池中创建该字符串，
然后返回。

```
public native String intern();
```

例1：

```java
String s = new String("abc");
String s1 = "abc";
// true
System.out.println(s1 == s.intern());
```

一种解释：
执行 `new String("abc")` 会在堆内存和常量池分别创建一个 "abc" 对象。执行  `String s1 = "abc";` 发现常量池有 对应的对象，直接返回常量池的地址。
执行 `intern()` 会到常量池中找。返回常量池中的 `abc` 的地址，和 s1相同。

例2：

```java
String str2 = new String("1") + new String("2");
String str3 = new String("1") + new String("2");

str2.intern();

String str1 = "12";

// false
System.out.println(str3 == str1);
// true
System.out.println(str2 == str1);
```

一种解释：执行 `str2.intern();` 时，发现常量池没有 `12` 这个对象，直接把堆内存的 str2 对象放进常量池了。 所以有 `str2 == str1` 。

### 参考

- http://www.hollischuang.com/archives/1330
- https://docs.oracle.com/javase/8/docs/api/java/lang/String.html
- https://www.cnblogs.com/wanlipeng/archive/2010/10/21/1857513.html
- http://www.cnblogs.com/paddix/p/5326863.html
- https://www.baeldung.com/java-char-sequence-string
- https://www.baeldung.com/java-9-compact-string
- https://wangkuiwu.github.io/2012/04/11/charsequence/
- https://blog.csdn.net/claram/article/details/53770830
