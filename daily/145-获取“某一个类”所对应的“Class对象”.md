### 问：获取“某一个类”所对应的“Class对象”有哪几种方法？

1. 根据对象的引用.getClass()方法获取：MyObject object=new MyObject();  Class c=object.getClass();
2. 根据类名.class获取：Class c=MyObject.class;
3. 根据Class中的静态方法Class.forName(); Class c=Class.forName("MyObject");
