### 在泛型为Integer的ArrayList中存放一个String类型的对象

```java
List<Integer> list = new ArrayList<>();
// 利用反射机制动态调用 list 对象的 add 方法，绕过类型检查
Class listClass = list.getClass();
try {
    Method addMethod = listClass.getMethod("add", Object.class);
    addMethod.invoke(list, "hello");
    System.out.println("hello".equals(list.get(0)));
} catch (Exception e) {
    e.printStackTrace();
}
```



> 实例工作中一般用不到。主要是考察面试者对泛型和反射的理解和运用

