### join 和 wait 区别？

Thread 类里定义了三个join 方法：

```java
// 等待当前这个线程结束，但是最多等待 millis 毫秒，millis=0表示永远等待
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        // 主要是这一部分
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
// 等待当前这个线程结束，但是最多等待 millis 毫秒 + nanos 微秒
public final synchronized void join(long millis, int nanos) throws InterruptedException {

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++;
    }

    join(millis);
}
// 一直等待, 等价于调用 join(0)
public final void join() throws InterruptedException {
    join(0);
}
```



核心代码就是这一部分：

```java
while (isAlive()) {
    long delay = millis - now;
    if (delay <= 0) {
    	break;
    }
    wait(delay);
    now = System.currentTimeMillis() - base;
}
```

循环判断当前线程是否 alive，如果 alive，就调用 `wait()` 方法等待。当线程结束时会调用 `notifyAll` 方法。

不建议在线程实例上调用 `wait`, `notify`, or `notifyAll`。（为什么？）

注意 `join` 方法是 thread类中定义的，`wait`, `notify`, or `notifyAll` 这几个是Object类中定义的。