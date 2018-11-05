### Java中创建对象的方法

参考：http://www.importnew.com/22405.html

#### 使用new关键字

```Java
User user = new User();
```

#### 使用反射机制

1. 使用Class类的newInstance方法


```java
//创建方法1
User user = (User)Class.forName("根路径.User").newInstance();　
//创建方法2（用这个最好）
User user = User.class.newInstance();
```

用 `User.class` 这种类字面量的方式更简单也更安全，因为编译时就会受到检查，所以不需要try语句块，不需要调用 forName 方法，所以更高效。

2. 使用Constructor类的newInstance方法

```java
Constructor<User> constructor = User.class.getConstructor();
User user = constructor.newInstance();
```

#### 使用clone方法

无论何时我们调用一个对象的clone方法，jvm就会创建一个新的对象，将前面对象的内容全部拷贝进去。用clone方法创建对象并不会调用任何构造函数。
要使用clone方法，我们需要先实现Cloneable接口并实现其定义的clone方法。


```java
public class CloneTest implements Cloneable {
    private String name;

    public CloneTest(String name) {
        super();
        this.name = name;
    }

    public static void main(String[] args) {
        try {
            CloneTest cloneTest = new CloneTest("hello");
            CloneTest copyClone = (CloneTest) cloneTest.clone();
            System.out.println("newclone:" + cloneTest.getName());
            System.out.println("copyClone:" + copyClone.getName());
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

#### 使用反序列化

对一个对象进行序列化和反序列化，JAVA虚拟机都会为我们创建一个单独的对象。在反序列化中，JAVA虚拟机不会使用任何构造函数来创建对象。
