### ：IO流需不需要关闭,如果关闭的话应该如何关闭。需要注意什么。

https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html


> Streams have a BaseStream.close() method and implement AutoCloseable, but nearly all stream instances do not actually need to be closed after use. Generally, only streams whose source is an IO channel (such as those returned by Files.lines(Path, Charset)) will require closing. Most streams are backed by collections, arrays, or generating functions, which require no special resource management. (If a stream does require closing, it can be declared as a resource in a try-with-resources statement.)
