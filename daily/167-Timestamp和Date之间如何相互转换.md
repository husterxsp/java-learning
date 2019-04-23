### Timestamp和Date之间如何相互转换?



```java
public class Test {
    public static void main(String[] args) {
        System.out.println(new Date().getTime());

        System.out.println(System.currentTimeMillis());

        System.out.println(new Date(System.currentTimeMillis()));
    }
}
```