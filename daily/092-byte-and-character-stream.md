### 字节流和字符流的区别

流是一种处理文件的方式。

字节流逐字节地访问文件。字节流适用于任何类型的文件，但不太适合文本文件（txt文件）。例如，如果文件使用的是unicode编码，而字符用两个字节表示，则字节流将会将这个字符分开处理，因而自己还需要做转换。

字符流将逐个字符地读取文件。需要为字符流提供文件的编码才能正常工作。

字节流类继承自抽象类 InputStream 和 OutputStream。
字符流类继承自抽象类 Reader 和 Writer。

字符流类通常是对字节流类的一个封装，即物理的读写还是逐字节，但是字符流处理了字节到字符的转换。例如 FileReader 底层用的是 FileInputStream。

- https://docs.oracle.com/javase/tutorial/essential/io/bytestreams.html
- https://stackoverflow.com/questions/3013996/byte-stream-and-character-stream
- https://www.geeksforgeeks.org/character-stream-vs-byte-stream-java/
