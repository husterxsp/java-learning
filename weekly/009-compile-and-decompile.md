## Java中的编译与反编译。

### 1. 什么是编译和反编译？

编译是将高级语言（如java）转换为机器能够识别的低级语言的过程（如汇编），反编译则相反，是将编译过的代码再转换为源代码。

http://www.hollischuang.com/archives/2322

### 2. Java如何编译代码，如何反编译代码？

JDK中的javac工具负责编译工作，可以将后缀名为.java的源文件编译为后缀名为.class的字节码文件，该文件可以运行于Java虚拟机。

反汇编的工具较多，如：javap、jad、cfr

http://www.hollischuang.com/archives/58

#### javap

JDK自带的工具。
以下面的代码为例。

```java
// Hello.java
package test;
public class Hello {
    private String hello;
    public static void main(String[] args) {
        String str = "world";
        System.out.println(str);
        switch (str) {
            case "hello":
                System.out.println("hello");
                break;
            case "world":
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

执行 `javap Hello.class` 结果如下：

```java
public class test.Hello {
  public test.Hello();
  public static void main(java.lang.String[]);
}
```

默认 `javap Hello.class` 相当于 `javap -package Hello.class` , -package是默认选项，即显示 package/protected/public类和成员。

如果运行 `javap -private test/Hello` 就会连private都会显示。结果如下：

```java
public class test.Hello {
  private java.lang.String hello;
  public test.Hello();
  public static void main(java.lang.String[]);
}
```

以上示例展示javap可以用来显示源代码中的一些属性，方法。

如果使用 `-c` 选项，会显示一些反汇编代码，主要是是一些java字节码指令。运行 `javap -c Hello.class`， 部分结果如下：

```
public class test.Hello {
  public test.Hello();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String world
       2: astore_1
       3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       6: aload_1
       7: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      10: aload_1
      11: astore_2
      12: iconst_m1
      13: istore_3
      14: aload_2
      15: invokevirtual #5                  // Method java/lang/String.hashCode:()I
      18: lookupswitch  { // 2
              99162322: 44
             113318802: 58
               default: 69
          }
          ...
```



如果使用 `-v  -verbose ` 选项，则输出更详细的信息，如下所示。

```java
 Last modified 2018-9-30; size 848 bytes
  MD5 checksum bff12735bbe633433a201b5a0ac13b87
  Compiled from "Hello.java"
public class test.Hello
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #9.#28         // java/lang/Object."<init>":()V
   #2 = String             #29            // world
   #3 = Fieldref           #30.#31        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #32.#33        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Methodref          #34.#35        // java/lang/String.hashCode:()I
   #6 = String             #10            // hello
   #7 = Methodref          #34.#36        // java/lang/String.equals:(Ljava/lang/Object;)Z
   #8 = Class              #37            // test/Hello
   #9 = Class              #38            // java/lang/Object
  #10 = Utf8               hello
  ···
  {
  public test.Hello();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ltest/Hello;
···
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: ldc           #2                  // String world
         2: astore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: aload_1
         7: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
···
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0     119     0  args   [Ljava/lang/String;
            3     116     1   str   Ljava/lang/String;
      StackMapTable: number_of_entries = 6
        frame_type = 254 /* append */
          offset_delta = 44
          locals = [ class java/lang/String, class java/lang/String, int ]
        frame_type = 13 /* same */
        frame_type = 10 /* same */
        frame_type = 26 /* same */
        frame_type = 10 /* same */
        frame_type = 249 /* chop */
          offset_delta = 10
```

主要如果要显示  `LocalVariableTable` 则需要在javac的时候加上 `-g` 参数，即编译的时候加上各种调试信息。[参考](https://stackoverflow.com/questions/31777957/javap-c-v-l-s-helloworld-class-doesnt-show-localvariabletable)

可以看到编译之后，会生成一个无参的构造函数。

args_size 包含一个默认的 this 参数的。



https://www.jianshu.com/p/c8928d98cb8e





javap文档：https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javap.html

#### jad

jad 下载地址 http://www.javadecompilers.com/jad

运行 `./jad Hello.class`，会生成 Hello.jad 文件，内容如下：

```java
package test;

import java.io.PrintStream;

public class Hello
{

    public Hello()
    {
    }

    public static void main(String args[])
    {
        String s = "world";
        System.out.println(s);
        String s1 = s;
        byte byte0 = -1;
        switch(s1.hashCode())
        {
        case 99162322:
            if(s1.equals("hello"))
                byte0 = 0;
            break;

        case 113318802:
            if(s1.equals("world"))
                byte0 = 1;
            break;
        }
        switch(byte0)
        {
        case 0: // '\0'
            System.out.println("hello");
            break;

        case 1: // '\001'
            System.out.println("world");
            break;
        }
    }

    private String hello;
}
```

可以看出，该工具生成的几乎就是源代码了。

#### CFR

jad几乎停止更新，相对来说，CFR 可以支持更多的java新特性。

http://www.benf.org/other/cfr/

运行 `java -jar cfr_0_132.jar Hello.class` ,  结果如下：

```java
package test;

import java.io.PrintStream;

public class Hello {
    private String hello;

    public static void main(String[] arrstring) {
        String string = "world";
        System.out.println(string);
        switch (string) {
            case "hello": {
                System.out.println("hello");
                break;
            }
            case "world": {
                System.out.println("world");
                break;
            }
        }
    }
}
```

以上结果，是对class反编译的比较彻底了，如果加上参数 `--decodestringswitch false` (--decodestringswitch 默认 true)，则结果如下：

```java
package test;

import java.io.PrintStream;

public class Hello {
    private String hello;

    public static void main(String[] arrstring) {
        String string = "world";
        System.out.println(string);
        String string2 = string;
        int n = -1;
        switch (string2.hashCode()) {
            case 99162322: {
                if (!string2.equals("hello")) break;
                n = 0;
                break;
            }
            case 113318802: {
                if (!string2.equals("world")) break;
                n = 1;
            }
        }
        switch (n) {
            case 0: {
                System.out.println("hello");
                break;
            }
            case 1: {
                System.out.println("world");
                break;
            }
        }
    }
}
```

可以看出 `--decodestringswitch` 表示是否是 switch(string) 进行反编译。更多参数可以查看 `java -jar cfr_0_132.jar --help`。

综上，如果只是想看下源代码里有哪些方法跟属性的话，JDK自带的 javap 就可以满足了，但是如果想看完整的源代码，可以使用 CFR。

#### 如何防止反编译

- 隔离Java程序：让用户接触不到你的Class文件
- 对Class文件进行加密：提到破解难度
- 代码混淆：将代码转换成功能上等价，但是难于阅读和理解的形式
- 转换成本地代码

参考： http://it4j.iteye.com/blog/2157422

### 3. 尝试反编译switch、String的“+”、lambda表达式、java 10的本地变量推断等。

#### switch

switch 目前支持 原子类型`byte, short, char, int`，以及 `enumerated types`, `String`。另外还支持一些包装类型：`Character, Byte, Short, and Integer`。  https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html

需要注意的是 switch(string) 是java7支持的新语法，具体可以参看上面的反编译代码。其实也只是将 switch(string) 转换为 switch(string.hashCode())，hashCode()返回的是 int。

switch(string) 会先比较 hashcode，如果 hashcode 相等，还要用 equals 方法判断是否完全相等。因为即使 hashcode 相等，两个字符串也不一定相等，会有hash碰撞的情况。

参考：http://www.hollischuang.com/archives/61

#### String "+"

https://docs.oracle.com/javase/tutorial/java/data/strings.html

在java中，string对象是不可变的。一些操作改变string对象的方法其实只是创建了新的string对象返回。

为什么String对象不可变？看下源代码：

```java
// JDK 10
// String.java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final byte[] value;
}
```

如上所示，java 的 String 其实是对 字节数组 (jdk8还是字符数组 char value[]...) 的封装，而且这个value是final修饰的，所以 一旦初始化后，value不可变，所以String对象不可变。

一些改变字符串值的方法，如replace, 其实是通过 `new String()` 创建了一个新的字符串对象并赋值给这个引用。需要注意，`String a = "Hello";` 这句代码中，变量 a 只是某个字符串对象的引用，所以通过这些方法或者直接给a重新赋值，改变的只是这个变量 a 的引用。

再看下 String 的 "+"：

```java
// Hello.java
package test;

public class Hello {
    public static void main(String[] args) {
        String a = "hello";
        String b = "world";
        String c = a + b;
        System.out.println(c + 2);
    }
}
```

运行 `java -jar cfr_0_132.jar --stringbuilder false Hello.class` , 结果如下：

```java
package test;

import java.io.PrintStream;

public class Hello {
    public static void main(String[] arrstring) {
        String string = "hello";
        String string2 = "world";
        String string3 = new StringBuilder().append(string).append(string2).toString();
        System.out.println(new StringBuilder().append(string3).append(2).toString());
        System.out.println(new StringBuilder().append(string3).append(1).toString());
    }
}
```

可以看到string的 "+" 是通过 StringBuilder() 创建新的string来实现的。

再考虑以下代码：

```java
final String a1 = "hello";
String b1 = "hello2";
String c1 = a1 + 2;
System.out.println(b1 == c1); // true
System.out.println(b1.equals(c1)); // true

String a2 = "hello";
String b2 = "hello2";
String c2 = a2 + 2;
System.out.println(b2 == c2); // false
System.out.println(b2.equals(c2)); // true
```

先看下反编译结果:

```java
String string = "hello2";
String string2 = "hello2";
System.out.println(string == string2);
System.out.println(string.equals(string2));
String string3 = "hello";
String string4 = "hello2";

// JDK 8
String string5 = new StringBuilder().append(string3).append(2).toString();
System.out.println(string4 == string5);

// JDK 10
CallSite callSite = StringConcatFactory.makeConcatWithConstants(new Object[]{"\u00012"}, string3);
System.out.println(string4 == callSite);

System.out.println(string4.equals(callSite));
```

因为 a1 是final修饰的，所以 a1 (引用)不可变，所以因为 "编译时优化"，编译时会直接计算 "+" 两边的常量，所以编译后 `c1 = "hello2"` , 并且 a1已经被删掉了。

java中有两种方式创建字符串：

- 字面量方式创建字符串：`String name = "tom";`
- new关键字创建字符串：`String name2 = new String("jerry");`

Java中的String pool(字符串常量池)是java堆内存(heap memory)中的存储字符串的一块区域。

1.当使用字面量的方式创建字符串时,虚拟机会检查字符串池中的字符串，如果有相同的字符串，那么并不会为新的字符串分配内存空间，而是令它指向字符串常量池中已经存在的那个字符串。这样做的好处是节省了内存的消耗。
注意使用字面量的方式创建的字符串是存储在字符串常量池中的。

2.使用new关键字创建的字符串不存储在字符串常量池中，而是直接在堆内存中。

因而，再回到上面的代码，执行 `String b1 = "hello2";` 时在常量池创建的 "hello2" 对象，在 执行 `c1 = "hello2"`, 直接将 c1 指向常量池中的对象，所以 b1 和 c1 指向的内存地址相同。 （注意 `==` 比较的是引用，即地址。equals() 方法则是比较的是字符串的值。）

而 c2 处的 "+" 则是调用了 StringBuilder （jdk8, jdk10用的是StringConcatFactory）创建了新的 string 对象，所以 b2 和 c2指向的地址不同。

StringBuilder 对象和 String对象？

StringBuilder是 可变的，它内部维护一个数组，一些修改它的方法是修改这个数组，。https://docs.oracle.com/javase/tutorial/java/data/buffers.html

既然 string 是不可变的，那么一般final修饰有什么意义？

final 修饰后，变量就不能重新赋值了，也即 string 变量对应的引用也不可变了。不用final的话，string对象可以重新赋值，只不过重新赋值的时候是创建一个新的对象返回，即改变了引用。
参考：https://stackoverflow.com/questions/10233309/does-it-make-sense-to-define-a-final-string-in-java

- 关于 String pool, 参考: https://blog.csdn.net/dhassa/article/details/73075852
- 为什么String是不可变的，参考：https://blog.csdn.net/csdn_aiyang/article/details/68942964
- java编译时优化，参考：https://blog.csdn.net/baidu_30809315/article/details/78737293

#### lambda表达式

https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html

http://www.importnew.com/16436.html

java8新语法。类似的语法在javascript中称作 "箭头函数"。

可以理解为是 函数表达式 的一种简化的写法。

```java
package test;

import java.util.Arrays;
import java.util.List;

public class Hello {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("a", "b", "c");

        // java8之前
        for (String s : list) {
            System.out.println(s);
        }
        // java8。注意forEach也是java8的新语法
        list.forEach(n -> System.out.println(n));
    }
}
```

以上代码反编译如下：

`java -jar cfr_0_132.jar --decodelambdas false Hello.class`

```java
package test;

import java.io.PrintStream;
import java.lang.invoke.LambdaMetafactory;
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class Hello {
    public static void main(String[] arrstring) {
        List<String> list = Arrays.asList("a", "b", "c");
        for (String string : list) {
            System.out.println(string);
        }
        list.forEach((Consumer<String>)LambdaMetafactory.metafactory(null, null, null, (Ljava/lang/Object;)V, lambda$main$0(java.lang.String ), (Ljava/lang/String;)V)());
    }

    private static /* synthetic */ void lambda$main$0(String string) {
        System.out.println(string);
    }
}
```

#### java 10 的本地变量推断

http://www.hollischuang.com/archives/2064

http://www.hollischuang.com/archives/2187

看到这个语法想到之前看过的这个： 弱类型、强类型、动态类型、静态类型语言的区别是什么？ - vczh的回答 - 知乎
https://www.zhihu.com/question/19918532/answer/21645395

其实javascript里的所有局部变量都是用 var 声明的。

java是静态类型语言，定义变量的时候需要指明变量的具体类型。javascript是动态类型语言，定义变量的时候，直接用 var 。

但是这个语法并不意味着 java 向动态语言转换了，因为java中的 var 在编译的时候就会转换为对应的确定类型了，不存在因为类型错误导致的运行时错误。

反编译查看：

```java
List<String> list = Arrays.asList("a", "b", "c");
for (var string : list) {
    System.out.println(string);
}
```

以上代码编译之后，再反编译之后结果如下：

```java
List<String> list = Arrays.asList("a", "b", "c");
for (String string : list) {
    System.out.println(string);
}
```

结果和源代码完全一样。
