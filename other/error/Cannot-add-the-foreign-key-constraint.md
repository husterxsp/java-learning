### Cannot add the foreign key constraint

在两个数据表之间添加外键的时候报错 Cannot add the foreign key constraint

原因：mysql要求两个进行关联的键之间有相同数据类型和属性。一般我们关联的是一个表的主键和另一个表的某一列。
而创建主键时可能会默认加上 "unsigned" 以及 not Allow Null等。

所以进行关联的时候要注意是否另一个表的某一列的一些属性是否和该主键一致。
