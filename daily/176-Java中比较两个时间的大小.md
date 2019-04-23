### 在Java中，如何比较两个时间的大小？

```java
Date a = new Date();
try {
    TimeUnit.SECONDS.sleep(1);
} catch (InterruptedException e) {
    e.printStackTrace();
}
Date b = new Date();

// 1.使用Date的compareTo()方法，大于、等于、小于分别返回1、0、-1
System.out.println(a.compareTo(b));

// 2.使用时间戳（指的是从1970年1月1日起到该日期的毫秒数）直接比较大小
System.out.println(a.getTime() > b.getTime());

// 3.使用Date的before()、after()方法
// 如果前者比后者小返回true，否则为false
System.out.println(a.before(b));
// 如果前者比后者大返回true，否则为false
System.out.println(a.after(b));
```

本质还是根据时间戳进行比较。

```java
// Date.java
public int compareTo(Date anotherDate) {
    long thisTime = getMillisOf(this);
    long anotherTime = getMillisOf(anotherDate);
    return (thisTime<anotherTime ? -1 : (thisTime==anotherTime ? 0 : 1));
}
public boolean before(Date when) {
    return getMillisOf(this) < getMillisOf(when);
}
public boolean after(Date when) {
    return getMillisOf(this) > getMillisOf(when);
}
```

