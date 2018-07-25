### 字节流和字符流之间如何相互转换。

```java
import java.io.*;

/**
 * @author xushaopeng
 * @date 2018/07/17
 */
public class Convert {

    static String readInput() {
        StringBuffer buffer = new StringBuffer();
        try {
            FileInputStream fis = new FileInputStream("test.txt");
            InputStreamReader isr = new InputStreamReader(fis, "UTF8");
            Reader in = new BufferedReader(isr);
            int ch;
            while ((ch = in.read()) > -1) {
                buffer.append((char)ch);
            }
            in.close();
            return buffer.toString();
        }
        catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    static void writeOutput(String str) {
        try {
            FileOutputStream fos = new FileOutputStream("test.txt");
            Writer out = new OutputStreamWriter(fos, "UTF8");
            out.write(str);
            out.close();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) {
        String jaString = "\u65e5\u672c\u8a9e\u6587\u5b57\u5217";
        writeOutput(jaString);
        String inputString = readInput();
        String displayString = jaString + " " + inputString;

        System.out.println(displayString);

    }
}
```

参考：https://docs.oracle.com/javase/tutorial/i18n/text/stream.html


以下是Holli的答案：
整个IO包实际上分为字节流和字符流，但是除了这两个流之外，还存在一组字节流-字符流的转换类。
OutputStreamWriter：是Writer的子类，将输出的字符流变为字节流，即将一个字符流的输出对象变为字节流输出对象。
InputStreamReader：是Reader的子类，将输入的字节流变为字符流，即将一个字节流的输入对象变为字符流的输入对象。

疑问：怎么没有OutputStreamReader之类的？难道是因为输出的主要是输出到文件之类的，存成字节。读入的话，就读成字符？


参考：
https://www.cnblogs.com/sjjsh/p/5269781.html
https://blog.csdn.net/syaguang2006/article/details/41422027
