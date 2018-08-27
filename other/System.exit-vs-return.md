## System.exit()和return的区别?

在main()中的区别：
当用return返回时，main()并不能立即运行结束，因为可能有其他的非后台线程正在执行。
而用System.exit(数值);时，main()将立即无条件的结束。
system.exit(0)表示程序正常退出，system.exit(1)表示非正常退出，都没有返回值。

参考：
- https://blog.csdn.net/xiaoliangmeiny/article/details/6940807
