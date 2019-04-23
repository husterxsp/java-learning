### 什么是Calendar

从JDK1.1版本开始，在处理日期和时间时，系统推荐使用Calendar类进行实现。在设计上，Calendar类的功能要比Date类强大很多，而且在实现方式上也比Date类要复杂一些。
Calendar类是一个抽象类，在实际使用时实现特定的子类的对象，创建对象的过程对程序员来说是透明的，只需要使用getInstance方法创建即可。

#### 1、使用Calendar类代表当前时间

```java
Calendar c = Calendar.getInstance();
```


由于Calendar类是抽象类，且Calendar类的构造方法是protected的，所以无法使用Calendar类的构造方法来创建对象，API中提供了getInstance方法用来创建对象。
使用该方法获得的Calendar对象就代表当前的系统时间，由于Calendar类toString实现的没有Date类那么直观，所以直接输出Calendar类的对象意义不大。

#### 2、使用Calendar类代表指定的时间



```java
Calendar c1 = Calendar.getInstance();
c1.set(2018, 11 , 11);
```

使用Calendar类代表特定的时间，需要首先创建一个Calendar的对象，然后再设定该对象中的年月日参数来完成。

