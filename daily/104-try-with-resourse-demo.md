### 请将以下代码，改成使用try-with-resources的形式。

```java
//Initializes The Object
User1 user = new User1();
user.setName("hollis");
user.setAge(23);
System.out.println(user);

//Write Obj to File
ObjectOutputStream oos = null;
try {
    oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
    oos.writeObject(user);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    IOUtils.closeQuietly(oos);
}

//Read Obj from File
File file = new File("tempFile");
ObjectInputStream ois = null;
try {
    ois = new ObjectInputStream(new FileInputStream(file));
    User1 newUser = (User1) ois.readObject();
    System.out.println(newUser);
} catch (IOException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} finally {
    IOUtils.closeQuietly(ois);
    try {
        FileUtils.forceDelete(file);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

修改后：

```java
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;

import java.io.*;

/**
 * @author xushaopeng
 * @date 2018/07/31
 */
public class Hello2 {
        public static void main(String[] args) {
            User1 user = new User1();
            user.setName("hollis");
            user.setAge(23);
            System.out.println(user);

            //Write Obj to File
            try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"))) {
                oos.writeObject(user);
            } catch (IOException e) {
                e.printStackTrace();
            }

            //Read Obj from File
            File file = new File("tempFile");
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file))) {
                User1 newUser = (User1) ois.readObject();
                System.out.println(newUser);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } finally {
                try {
                    FileUtils.forceDelete(file);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
