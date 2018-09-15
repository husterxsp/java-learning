创建线程的几种方式

### 直接创建线程

Java 中实现多线程有两种方法：继承 Thread 类、实现 Runnable 接口。在程序开发中只要是多线程，肯定永远以实现 Runnable 接口为主，因为实现 Runnable 接口相比继承 Thread 类有如下优势：

可以避免由于 Java 的单继承特性而带来的局限；
增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；
适合多个相同程序代码的线程区处理同一资源的情况。

多线程的实现方法：http://wiki.jikexueyuan.com/project/java-concurrency/function.html
如何创建并运行 java 线程： http://wiki.jikexueyuan.com/project/java-concurrent/creating-and-starting-java-threads.html

1.继承 Thread 类

```java
public class Task extends Thread {
    @Override
    public void run() {

    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        Thread thread = new Task();
        thread.start();
    }
}
```

写成 `Thread thread = new Thread(new Task());` 也可以，因为 Thread 也继承了 Runnable 接口， `public class Thread implements Runnable {}`。

2. 实现 Runnable 接口。

需要重写 run 方法 `public abstract void run();` 。
调用 `start()` 方法会进行一些初始化操作，然后会调用 `run()` 执行。

```java
public class Task implements Runnable {
    @Override
    public void run() {

    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        Thread thread = new Thread(new Task());
        thread.start();
    }
}
```

可以直接调用任务，但是这样是直接占用调用该 run 方法的 线程。

```java
public static void main(String[] args) {
    Task task = new Task();
    task.run();
}
```

常见错误：创建并运行一个线程所犯的常见错误是调用线程的 run()方法而非 start()方法。
事实上,run()方法并非是由刚创建的新线程所执行的，而是被创建新线程的当前线程所执行了。也就是被执行上面两行代码的线程所执行的。想要让创建的新线程执行 run()方法，必须调用新线程的 start 方法。

```java
Thread newThread = new Thread(MyRunnable());
newThread.run();  //should be start();
```

### Executor 框架 & 线程池

http://wiki.jikexueyuan.com/project/java-concurrent/thread-pools.html

http://wiki.jikexueyuan.com/project/java-concurrency/executor.html

java5 新功能。
Executor 框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable 等。

#### Executor 执行 Runnable 任务

Executor 接口中之定义了一个方法 execute（Runnable command），来执行一个任务。

```java
public class TestThreadPool {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
//      ExecutorService executorService = Executors.newScheduledThreadPool(5);
//      ExecutorService executorService = Executors.newFixedThreadPool(5);
//      ExecutorService executorService = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            executorService.execute(new TestRunnable());
            System.out.println("************* a" + i + " *************");
        }
        executorService.shutdown();
    }
}

class TestRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程被调用了。");
    }
}
```

源代码
```java
public interface Executor {
    void execute(Runnable command);
}
public interface ExecutorService extends Executor {
    ...
}
```

#### Executor 执行 Callable 任务

实现了 Callable 接口的类的任务有返回值。通过 ExecutorService 的 submit(Callable task) 方法来执行。

可以看到 `Callable` 接口中只有一个 `call` 方法，需要重写。

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

```java
public class CallableDemo {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        List<Future<String>> resultList = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
            //使用ExecutorService执行Callable类型的任务，并将结果保存在future变量中
            Future<String> future = executorService.submit(new TaskWithResult(i));
            //将任务执行结果存储到List中
            resultList.add(future);
        }

        //遍历任务的结果
        for (Future<String> fs : resultList) {
            try {
                //Future返回如果没有完成，则一直循环等待，直到Future返回完成
                while (!fs.isDone()) { }
                //打印各个线程（任务）执行的结果
                System.out.println(fs.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            } finally {
                //启动一次顺序关闭，执行以前提交的任务，但不接受新任务
                executorService.shutdown();
            }
        }
    }
}

class TaskWithResult implements Callable<String> {
    private int id;

    public TaskWithResult(int id) {
        this.id = id;
    }

    /**
     * 任务的具体过程，一旦任务传给ExecutorService的submit方法，
     * 则该方法自动在一个线程上执行
     */
    @Override
    public String call() throws Exception {
        System.out.println("call()方法被自动调用！ " + Thread.currentThread().getName());

        Thread.sleep(10000);
        //该返回结果将被Future的get方法得到
        return "call()方法被自动调用，任务返回的结果是：" + id + " " + Thread.currentThread().getName();
    }
}
```

`submit` 之后，`call` 方法就会执行，并在执行完之后将结果返回，通过 `get()` 方法获取。`isDone()` 方法有没有必要？因为即使不用 isDone 检查，get 方法也会一直阻塞知道获取结果的。



需要注意，看以下源代码可知，submit方法其实也可以传入 Runnable 对象，只不过返回值是 null，或者可以一开始传入一个值用于返回。



```java
public abstract class AbstractExecutorService implements ExecutorService {
	public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
	public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }
}
```



newCachedThreadPool 会先检查是否有空闲的线程，有的话重用，没有则创建新的。

注意几种创建线程池的方法的区别。

### 自定义线程池

```java
public class ThreadPoolTest {
    public static void main(String[] args) {
        //创建等待队列
        BlockingQueue<Runnable> bqueue = new ArrayBlockingQueue<>(20);

        //创建线程池，池中保存的线程数为3，允许的最大线程数为5
        ThreadPoolExecutor pool = new ThreadPoolExecutor(3, 5, 50, TimeUnit.MILLISECONDS, bqueue);
        //创建七个任务
        Runnable t1 = new MyThread();
        Runnable t2 = new MyThread();
        Runnable t3 = new MyThread();
        Runnable t4 = new MyThread();
        Runnable t5 = new MyThread();
        Runnable t6 = new MyThread();
        Runnable t7 = new MyThread();
        //每个任务会在一个线程上执行
        pool.execute(t1);
        pool.execute(t2);
        pool.execute(t3);
        pool.execute(t4);
        pool.execute(t5);
        pool.execute(t6);
        pool.execute(t7);
        //关闭线程池
        pool.shutdown();
    }
}

class MyThread implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "正在执行。。。");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 参考

- 《think in java》 21章
