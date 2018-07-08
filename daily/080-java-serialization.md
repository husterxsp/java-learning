### Java中如何实现序列化和反序列化?

java 实现序列化和反序列化的接口主要有

> java.io.Serializable
java.io.Externalizable
ObjectOutput
ObjectInput
ObjectOutputStream
ObjectInputStream

#### Serializable 接口

类通过实现 `java.io.Serializable` 接口以启用其序列化功能。
未实现此接口的类将无法使其任何状态序列化或反序列化。
可序列化类的所有子类型本身都是可序列化的。序列化接口没有方法或字段，仅用于标识可序列化的语义。 


```java
import java.io.Serializable;

public class User implements Serializable {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
import java.io.*;

public class SerializableDemo {
    public static void main(String[] args) {

        //Write Obj to File
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"))) {

            User user = new User();
            user.setName("hello");

            System.out.println(user);
            oos.writeObject(user);

        } catch (Exception e) {
            e.printStackTrace();
        }

        //Read Obj from File
        File file = new File("tempFile");
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file))) {

            User user = (User) ois.readObject();
            System.out.println(user);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### Externalizable接口

Externalizable继承了Serializable，该接口中定义了两个抽象方法：writeExternal()与readExternal()。当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写writeExternal()与readExternal()方法。由于上面的代码中，并没有在这两个方法中定义序列化实现细节，所以输出的内容为空。还有一点值得注意：在使用Externalizable进行序列化的时候，在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。所以，实现Externalizable接口的类必须要提供一个public的无参的构造器。

Externalizable 通过提供这么两个方法，可以完全控制序列化的形式和内容。

```java
import java.io.*;

public class User implements Externalizable {
    private String name;

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User:" + this.getName();
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {

    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {

    }
}
```

```java
import java.io.*;

public class SerializableDemo {
    public static void main(String[] args) {

        //Write Obj to File
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"))) {

            User user = new User("aaa");
            user.setName("hello");

            System.out.println(user);
            oos.writeObject(user);

        } catch (Exception e) {
            e.printStackTrace();
        }

        //Read Obj from File
        File file = new File("tempFile");
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file))) {

            User user = (User) ois.readObject();
            System.out.println(user);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### ObjectOutput和ObjectInput 接口


#### Transient 关键字

Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。


### 参考

- [http://www.hollischuang.com/archives/1150](http://www.hollischuang.com/archives/1150)
