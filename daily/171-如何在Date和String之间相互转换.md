### 如何在Date和String之间相互转换

最简单的方式就是使用SimpleDateFormat。SimpleDateFormat是Java提供的一个格式化和解析日期的工具类。

```java
// Date转String
Date data = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dataStr = sdf.format(data);
System.out.println(dataStr);
// String转Data
try {
    System.out.println(sdf.parse(dataStr));
} catch (ParseException e) {
    e.printStackTrace();
}
```



