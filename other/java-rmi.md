### Java RMI (远程方法调用)

#### 1. 创建 远程对象 (Remote Object)

1. 创建定义client/server 交流的接口(interface)

首先为远程对象创建接口，该接口继承 `java.rmi.Remote`。接口中的每个方法都要抛出异常 `java.rmi.RemoteException`。

注意RMI支持全部的Java的方法签名，只要对应的Java类型实现了 `java.io.Serializable` 接口。（因为消息需要进行网络传输，需要序列化？）

在RMI中, client/server 都会用到这个接口。

server 对应的接口实现成为 Remote Object。

client 对应的实现成为 stub。

```java
public interface RemoteHelloWord extends Remote {
    String sayHello() throws RemoteException;
}
```

2. 创建接口的实现(implementation)

即实现远程对象（Remote Object）。

注意这里不需要 `throws RemoteException ` 。因为 `RemoteException` 主要是为了告知client交流出错，在 remote object中抛出异常没啥用处。

另外，在 remote object 定义的方法，但没有在 interface 中定义的。对client是不可见的。


```java
public class RemoteHelloWordImpl implements RemoteHelloWord {
    @Override
    public String sayHello() {
        try {
            sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Hello Word!";
    }
}
```

#### 2. 创建 server

`exportObject` 方法创建了 stub，该方法的第一个参数是远程对象，第二个参数是 端口号。如果端口号是0表示？

接着，将 stub 绑定到 registry，这样 client 才能发现。

```java
public class RMIServer {
    public static void main(String[] args) {
        try {
            RemoteHelloWord hello = new RemoteHelloWordImpl();
            RemoteHelloWord stub = (RemoteHelloWord) UnicastRemoteObject.exportObject(hello, 0);

            LocateRegistry.createRegistry(1099);
            Registry registry = LocateRegistry.getRegistry();
            registry.bind("helloword", stub);

            System.out.println("绑定成功!");
        } catch (RemoteException e) {
            e.printStackTrace();
        } catch (AlreadyBoundException e) {
            e.printStackTrace();
        }
    }
}
```

#### 3. 创建 client

因为代码是运行在本地的，所以 `getRegistry` 的一些参数可以省略。1099是默认端口号。

通过 registry 找到 stub 对象后，就可以调用远程对象的方法了。远程方法执行后，会再将结果返回 client。

```java
public class RMIClient {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 1099);
            // 都对
//            Registry registry = LocateRegistry.getRegistry("127.0.0.1");
//            Registry registry = LocateRegistry.getRegistry();
            RemoteHelloWord hello = (RemoteHelloWord) registry.lookup("helloword");

            String ret = hello.sayHello();

            System.out.println(ret);
        } catch (RemoteException e) {
            e.printStackTrace();
        } catch (NotBoundException e) {
            e.printStackTrace();
        }
    }
}
```


参考:

- http://www.baeldung.com/java-rmi
- https://github.com/eugenp/tutorials/tree/master/java-rmi
