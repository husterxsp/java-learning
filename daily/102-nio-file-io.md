### nio文件读写

```java
public class ReadAndWriteNIO {
    /**
     * 使用IO读取指定文件的前1024个字节的内容。
     */
    public static void read() throws Exception {
        FileInputStream is = new FileInputStream("test.txt");

        byte[] buffer = new byte[1024];

        is.read(buffer);

        System.out.println(new String(buffer));

        is.close();
    }

    public static void main(String[] args) throws Exception {
//        readNIO();
        writeNIO();
    }

    /**
     * 使用NIO读取指定文件的前1024个字节的内容。
     */
    public static void readNIO() throws IOException {
        FileInputStream is = new FileInputStream("test.txt");

        //为该文件输入流生成唯一的文件通道  FileChannel
        FileChannel channel = is.getChannel();

        //开辟一个长度为1024的字节缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        channel.read(buffer);

        System.out.println(new String(buffer.array()));
        System.out.println(buffer.isDirect() + ", " + buffer.isReadOnly());

        channel.close();
        is.close();
    }

    public static void writeNIO() throws IOException {
        // 首先从 FileOutputStream 获取一个通道：
        FileOutputStream fout = new FileOutputStream("test.txt");
        FileChannel fc = fout.getChannel();
        // 下一步是创建一个缓冲区并在其中放入一些数据，这里，用message来表示一个持有数据的数组。

        ByteBuffer buffer = ByteBuffer.allocate(1024);

        int message[] = new int[100];

        for (int i = 0; i < message.length; ++i) {
            buffer.put((byte) i);
        }
        buffer.flip();
        // 最后一步是写入缓冲区中：
        fc.write(buffer);
    }

}

```

http://www.importnew.com/20784.html
