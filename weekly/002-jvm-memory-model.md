JVM-java内存模型（JMM），注意区分和java内存结构的区别

《java并发编程的艺术》阅读

## 第一章

### 上下文切换

CPU通过分配时间片来循环任务，任务从保存状态到再次加载的过程就是一次上下文切换。

#### 多线程一定快吗？

不一定，会有线程创建和上下文切换的开销。那么什么情况下快？

#### 减少上下文切换

- 无锁并发。线程竞争锁时，会引起上下文切换
- CAS算法？
- 使用最少线程
- 协程。在单线程里实现多任务调度，并在单线程里维持多个任务间的切换（如goroutine）

### 死锁


## 第二章 并发机制底层实现的原理

### volatile
volatile是轻量级的synchronized, 用于在多线程中保证共享变量的可见性。

对 volatile 变量的写操作, 汇编代码多了 lock 操作，根据 IA32 手册，lock 会做的两件事情
- 将 当前处理器 缓存行的数据写回系统内存
- 这个写回内存操作使其他CPU缓存了该内存地址的数据无效

对于第一件事，在多处理器环境，Lock 前缀指令 会确保该处理器独占共享内存（锁总线）。但是锁总线开销大，最近的处理器一般锁缓存（锁自己处理器的缓存吗？那其他的处理器还是能访问自己的缓存和内存啊？）

为了保证各个处理器的缓存一致，即实现上述的第二件事，需要实现 缓存一致性协议：
- 每个处理器嗅探总线上传播的数据来检查自己的缓存是不是过期，当发现自己缓存行对应的内存地址被修改，就将其设置为无效，当需要再次读写的时候，会重新从内存读取到缓存。

### synchronized

synchronized 实现同步的基础：每个对象都可以作为锁，有三种形式：
- 普通同步方法，锁是当前实例对象
- 静态同步方法，锁是当前类的Class对象
- 同步代码块，锁是 Synchronized 括号里配置的对象

JVM 基于 Monitor 对象实现上述同步，对于方法暂不清楚具体实现。对于代码块是 用 monitorenter 和 monitorexit 指令实现，编译之后，这两个指令会被插入到代码块的开始和结束位置（异常处也会插入?）。当线程执行到 monitorenter 就会尝试获取对象对应的 Monitor 对象的所有权，即对象的锁，当 Monitor被持有后，对象处于锁定状态。

synchronized 用的锁存在 java 对象头里。

锁的四种状态：无锁、偏向锁、轻量级锁、重量级锁。

待细看

## 第三章 Java内存模型

并发编程的两个问题：线程间如何通信和同步

通信机制有两种：共享内存、消息传递。

共享内存：隐式通信（读写内存），显式同步
消息传递：显示通信，隐式同步（消息发送在消息传递之前）。

同步：程序中用于控制不同线程间操作的发生顺序的机制。

Java并发 采用的是共享内存模型。

### Java 内存模型的抽象结构

JMM 控制java线程间的通信，决定共享变量的写入何时对另一个线程可见。
共享变量存在主内存，每个线程还有一个本地内存（抽象概念，可以理解为缓存，存共享变量副本）。这个本地内存包含缓存、写缓冲区、寄存器等。

### 指令重排序
为了提高性能，编译器和处理器都会重排序

1. 编译器优化：在不改变单线程程序语义的前提下，重排
2. 指令级并行：指令级并行技术（instruction-level-paralelism, ILP），前提是不存在数据依赖。（比如可以利用GPU? 一些可以并行的指令发给GPU并行地执行？）
3. 内存系统的重排序。

1是编译器重排序，JMM的规则会进行约束，比如禁止特定类型的重排序。
2、3是处理器，JMM的规则会要求java编译器编译时插入特定的内存屏障来保证顺序。

### 并发编程模型的分类

处理器会有自己的 写缓冲区，临时保存向内存写入的数据，可以避免由于处理器停下来等待向内存写入数据的延迟。但是这个缓冲去仅对处理器自己可见，所以可能会导致  读写不一致。。

待再看

### happens-before

用来阐述操作之间的可见性。在JMM中，如果一个操作的结果需要对另一个操作可见，则这两个操作存在happen-before关系。（注意不一定要前一个操作发生在前，只要其结果对后一个操作可见即可。）

happens-before 规则对应一个或多个编译器和处理器重排序规则。

### 重排序

#### 数据依赖性

如果两个操作访问同一个变量，有一个操作为写，则这两个操作之间存在数据依赖性，如 读写、写写、写读。
重排序会遵循数据依赖性，因为改变有数据依赖关系的操作会导致结果改变。
这里数据依赖性，指单个处理器中执行的指令序列和单个线程中执行的操作。

#### as-if-serial 语义

单线程程序执行的结果不会被改变（就是遵循数据依赖性）

#### 重排序对多线程的影响

控制依赖 -> 猜想执行 -> 重排序缓存 -> 实际重排序了

在单线程中允许 对 控制依赖 重排序，不会改变执行结果。但是多线程可能会改变执行结果。

### 顺序一致性


JMM 的最小安全性：JVM在堆上分配对象时，会对内存空间清0。

这里不是很懂。顺序一致性和 JMM

### volatile 内存语义

volatile 特性
- 可见性
- 原子性：注意是单个 volatile 的读写有原子性，但是 volatile++ 这样对我符合操作不具有原子性

volatile 变量的写-读与锁的释放和获取有相同的效果？P39


#### volatile 读写的语义

写：会把共享变量写入主内存
读：会从主内存读取


### 锁的内存语义