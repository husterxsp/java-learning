### SimpleDateFormat线程不安全



### 解决

1. 局部变量
2. Threadlocal
3. 加锁
4. 第三方类库：Apache commons 里的FastDateFormat、Joda-Time
5. JDK1.8 DateTimeFormatter



### 参考

1. https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650122732&idx=1&sn=2e49fcf3b88623f7415162be2d099e1e&chksm=f36bb4cdc41c3ddbcc3389489bb465f33f8a52f6c39a3e920b7d983738ea2523d1462c18e458&scene=0#rd
2. http://3a97dfb8.wiz03.com/share/s/0WBZ-U0cfkax2O1aCs039sgh3vt4ID33bkqA2ak63P3NmCIZ

