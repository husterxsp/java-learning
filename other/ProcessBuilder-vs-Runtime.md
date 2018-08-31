## ProcessBuilder 和 Runtime 的区别



Runtime是java1.0就有的API，ProcessBuilder是1.5才添加，但是目前看源代码，Runtime的内部实现其实就是用的ProcessBuilder。

### Runtime

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    private Runtime() {}
}
```

Runtime 的构造函数私有，只能通过 getRuntime 方法获取对象。其实也就是 **单例模式**。在该类第一次被虚拟机加载的时候，实例就被创建了。

##### 使用

通过 exec 函数传入命令。有6个重载过的函数：

```java
public Process exec(String command) throws IOException {
    return exec(command, null, null);
}
public Process exec(String command, String[] envp) throws IOException {
    return exec(command, envp, null);
}
public Process exec(String command, String[] envp, File dir)
    throws IOException {
    if (command.length() == 0)
        throw new IllegalArgumentException("Empty command");

    StringTokenizer st = new StringTokenizer(command);
    String[] cmdarray = new String[st.countTokens()];
    for (int i = 0; st.hasMoreTokens(); i++)
        cmdarray[i] = st.nextToken();
    return exec(cmdarray, envp, dir);
}
public Process exec(String cmdarray[]) throws IOException {
    return exec(cmdarray, null, null);
}
public Process exec(String[] cmdarray, String[] envp) throws IOException {
    return exec(cmdarray, envp, null);
}
public Process exec(String[] cmdarray, String[] envp, File dir)
    throws IOException {
    return new ProcessBuilder(cmdarray)
        .environment(envp)
        .directory(dir)
        .start();
}
```

参数不同，但最终都是调用 上述代码中的最后一个exec函数，同时可以看到该函数内部调用的是 ProcessBuilder的构造函数创建 ProcessBuilder 对象，然后调用 start 方法，最终返回一个 Process 对象。

所以一般可以这么使用

```java
Process process = Runtime.getRuntime().exec("javac HelloWorld.java");
```

### ProcessBuilder

就两个构造函数，可以直接传一个命令数组，或者把命令分隔开成多个字符串。还有两个对应的 command 函数，不过代码都是重复的，感觉没啥用。

```java
public ProcessBuilder(List<String> command) {
    if (command == null)
        throw new NullPointerException();
    this.command = command;
}
public ProcessBuilder(String... command) {
    this.command = new ArrayList<>(command.length);
    for (String arg : command)
        this.command.add(arg);
}
public ProcessBuilder command(List<String> command) {
    if (command == null)
        throw new NullPointerException();
    this.command = command;
    return this;
}
public ProcessBuilder command(String... command) {
    this.command = new ArrayList<>(command.length);
    for (String arg : command)
        this.command.add(arg);
    return this;
}
```

所以一般这么用：

```java
Process process = new ProcessBuilder("javac", "HelloWorld.java").start();
```



#### 主要不同点

传参不同：

- Runtime.exec()可接受一个单独的字符串，这个字符串是通过空格来分隔可执行命令程序和参数的；也可以接受字符串数组参数。


- ProcessBuilder的构造函数是一个字符串列表（`List<String>`）或者 多个字符串。列表中第一个参数是可执行命令程序，其他的是命令行执行是需要的参数。 



### demo

```java
public class Test {
    static ExecutorService threadPool = Executors.newCachedThreadPool();

    public static void main(String[] args) throws IOException, InterruptedException {
        Process process = Runtime.getRuntime().exec("java HelloWorld");
        // Process process = new ProcessBuilder("javac", "HelloWorld.java").start();

        printMessage(process.getInputStream());
        printMessage(process.getErrorStream());

        int value = process.waitFor();

        System.out.println(value == 0 ? "运行成功" : "运行失败");

    }

    private static void printMessage(final InputStream input) {

        threadPool.execute(() -> {
            BufferedReader bf = new BufferedReader(new InputStreamReader(input));

            String line;
            try {
                while ((line = bf.readLine()) != null) {
                    System.out.println(line);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}
```



### 参考

- http://desert3.iteye.com/blog/1596020
- http://www.hollischuang.com/archives/1383





