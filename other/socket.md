## socket 学习

### 单客户端连接

Server

```java
public class Server {
    public static void main(String[] args) throws IOException {

        // 端口号是从0~65535之间的,前1024个端口已经被Tcp/Ip 作为保留端口,因此你所分配的端口只能是1024个之后的
        // 创建一个服务器socket，即serversocket, 指定绑定的端口，并监听此端口
        ServerSocket serverSocket = new ServerSocket(12345);

        //调用accept()方法开始监听，等待客户端的连接
        System.out.println("***服务器即将启动，等待客户端的连接***");
        Socket socket = serverSocket.accept();

        // 获取输入流，并读入客户端的信息
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream());

        while (true) {
            String str = in.readLine();
            System.out.println(str);

            out.println("has receive...");
            out.flush();

            if (str.equals("end")) {
                break;
            }

        }

        socket.close();
    }
}

```

Client

```java
public class Client {
    public static void main(String[] args) throws IOException {
        // Socket server = new Socket(InetAddress.getLocalHost(), 5678);
        // 创建客户端socket建立连接，指定服务器地址和端口

        Socket socket = new Socket("127.0.0.1", 12345);

        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));

        PrintWriter out = new PrintWriter(socket.getOutputStream());

        BufferedReader systemIn = new BufferedReader(new InputStreamReader(System.in));

        while (true) {
            String str = systemIn.readLine();

            out.println(str);
            out.flush();

            if (str.equals("end")) {
                break;
            }

            System.out.println(in.readLine());
        }
        socket.shutdownInput();
        socket.close();
    }
}
```

### 多客户端连接

#### 1. 直接给Server代码加上 While(true)。

以下代码虽然能处理多客户端，但是一次只能 accept 一个客户端。其他的客户端得等前一个连接结束了，才能再建立连接。

所以是排队执行的。怎么排队？等多久？队列最多放多少？

accept()方法是阻塞的？

```java
public class Server {
    public static void main(String[] args) throws IOException {

        // 端口号是从0~65535之间的,前1024个端口已经被Tcp/Ip 作为保留端口,因此你所分配的端口只能是1024个之后的
        // 创建一个服务器socket，即serversocket, 指定绑定的端口，并监听此端口
        ServerSocket serverSocket = new ServerSocket(12345);

        //调用accept()方法开始监听，等待客户端的连接
        System.out.println("***服务器即将启动，等待客户端的连接***");

        while (true) {
            Socket socket = serverSocket.accept();

            System.out.println("建立连接");

            // 获取输入流，并读入客户端的信息
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());

            while (true) {
                String str = in.readLine();
                System.out.println(str);

                out.println("has receive...");
                out.flush();

                if (str.equals("end")) {
                    break;
                }

            }
            socket.close();
        }
    }
}
```

#### 2. 线程

2.1 为每个客户端连接建立一个线程, 修改 Server 实现如下：

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(12345);
        while (true) {
            Thread thread = new Thread(new Connection(serverSocket.accept()));
            thread.start();
        }
    }
}

class Connection implements Runnable {
    private Socket socket;

    public Connection(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());

            while (true) {
                String str = in.readLine();
                System.out.println(str);

                out.println("has receive...");
                out.flush();

                if (str.equals("end")) {
                    break;
                }
            }
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

测试的Client 代码：

```java
public class Client {
    public static void main(String[] args) {
        for (int i = 0 ; i < 10 ; i++) {
            Thread thread = new Thread(new Request());
            thread.start();
        }
    }
}

class Request implements Runnable {
    @Override
    public void run() {
        try {
            Socket socket = new Socket("127.0.0.1", 12345);

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());
            BufferedReader systemIn = new BufferedReader(new InputStreamReader(System.in));

            String str = "hello world";
            out.println(str);
            out.flush();

            String str1 = "end";
            out.println(str1);
            out.flush();

            socket.shutdownInput();
            socket.close();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

但是这种创建线程的方式是有问题的，一个进程能创建的线程数是有限的。代码中创建的线程数过多会失败的, 出现 `OutOfMemoryError` 错误。所以需要线程池。

怎么测试这样的并发程序的性能？

2.2 线程池

修改代码：

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(12345);
        while (true) {
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(new Connection(serverSocket.accept()));
        }
    }
}

public class Client {
    public static void main(String[] args) {
        for (int i = 0 ; i < 50 ; i++) {
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(new Request());
        }
    }
}
```

2.3 NIO

使用NIO构建Server

```java
public class NIOServer {
    public static void main(String[] args) {
        Selector selector = null;
        try {
            // 1. open a selector
            selector = Selector.open();
            // 2. listen for server socket channel
            ServerSocketChannel ssc = ServerSocketChannel.open();
            // must to be nonblocking mode before register
            ssc.configureBlocking(false);
            // bind server socket channel to port 8899
            ssc.bind(new InetSocketAddress(12345));
            // 3. register it with selector
            ssc.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {
                // run forever
                // 4. select ready SelectionKey for I/O operation
                if (selector.select(3000) == 0) {
                    continue;
                }
                // 5. get selected keys
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                // 6. handle selected key's interest operations
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();

                    if (key.isAcceptable()) {

                        System.out.println("key.isAcceptable()");

                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                        // get socket channel from server socket channel
                        SocketChannel clientChannel = serverSocketChannel.accept();
                        // must to be nonblocking before register with selector
                        clientChannel.configureBlocking(false);
                        // register socket channel to selector with OP_READ
                        clientChannel.register(key.selector(), SelectionKey.OP_READ);

                    }

                    if (key.isReadable()) {

                        System.out.println("key.isReadable()");

                        // read bytes from socket channel to byte buffer
                        SocketChannel clientChannel = (SocketChannel) key.channel();

                        ByteBuffer readBuffer = ByteBuffer.allocate(100);

                        int readBytes = clientChannel.read(readBuffer);

                        if (readBytes == -1) {
                            System.out.println("closed.......");
                            clientChannel.close();
                        } else if (readBytes > 0) {

                            // MessageInfo ?
                            String info = (String)key.attachment();

                            byte[] arr = readBuffer.array();
                            String s = new String(arr, 0, readBytes);

                            System.out.println("Client said: " + s);

                            // 将 buffer 指针移到0，这样就能再读到client 发来的数据了。
                            readBuffer.flip();
                            clientChannel.write(readBuffer);

                            // 不懂。。。
                            if (s.trim().equals("Hello")) {
                                // attachment is content used to write
                                key.interestOps(SelectionKey.OP_WRITE);
                                key.attach("Welcome!!!");
                            }

                        }
                    }


                    if (key.isValid() && key.isWritable()) {

                        System.out.println("key.isValid() && key.isWritable()");

                        SocketChannel clientChannel = (SocketChannel) key.channel();
                        // get content from attachment
                        String content = (String) key.attachment();
                        // write content to socket channel
                        clientChannel.write(ByteBuffer.wrap(content.getBytes()));
                        key.interestOps(SelectionKey.OP_READ);

                    }

                    // remove handled key from selected keys
                    iterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // close selector
            if (selector != null) {
                try {
                    selector.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        for (int i = 0 ; i < 1 ; i++) {
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(new Request());
        }
    }
}

class Request implements Runnable {
    @Override
    public void run() {
        try {
            Socket socket = new Socket("127.0.0.1", 12345);

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());
            BufferedReader systemIn = new BufferedReader(new InputStreamReader(System.in));

            String str = "hello world";
            out.print(str);
            out.flush();

            while(true) {

                char[] arr = new char[100];
                int len = in.read(arr);
                System.out.println(new String(arr,0, len));

            }

//            socket.shutdownInput();
//            socket.close();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

参考

- https://www.jianshu.com/p/792f942143ec
- https://blog.csdn.net/John8169/article/details/69487291

2.4 NIO2 (AIO)





2.5 Netty



## 其他问题

为什么 client 不需要绑定端口号？
为啥用PrintWriter

## 参考

- socket编程
    - https://blog.csdn.net/qq_32832023/article/details/78703810
    - http://daoyongyu.iteye.com/blog/265677
- socket 通信关于bind那点事 ： https://blog.csdn.net/suxinpingtao51/article/details/11809011
- Jmeter 测试socket性能：
    - https://www.cnblogs.com/0201zcr/p/6690900.html
    - http://20120923lina.iteye.com/blog/2372821
