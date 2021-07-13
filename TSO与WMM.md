## TSO与WMM



什么是 x86 TSO

- 首先 store buffers 被设计成了 FIFO 的队列，如果某个线程需要读取内存里的变量，务必优先读取本地 store buffer 中的值（如果有的话），否则去主内存里读取；
- MFENCE 指令用于清空本地 store buffer，并将数据刷到主内存；
- 某线程执行 Lock 前缀的指令集时，会去争抢全局锁，拿到锁后其他线程的读取操作会被阻塞，在释放锁之前，会清空该线程的本地的 store buffer，这里和 MFENCE 执行逻辑类似；
- store buffers 被写入变量后，除了被其他线程持有锁以外的情况，在任何时刻均有可能写回内存。





### NUMA 

全称 Non-Uniform Memory Access，译为“非一致性内存访问”。这种构架下，不同的内存器件和CPU核心从属不同的 Node，每个 Node 都有自己的集成内存控制器（IMC，Integrated Memory Controller）。

在 Node 内部，架构类似SMP，使用 IMC Bus 进行不同核心间的通信；不同的 Node 间通过QPI（Quick Path Interconnect）进行通信，如下图所示：

![](https://pic4.zhimg.com/v2-ee9a115806bae6341fc724707e4058cf_b.jpg)

一般来说，一个内存插槽对应一个 Node。需要注意的一个特点是，QPI的延迟要高于IMC Bus，也就是说CPU访问内存有了远近（remote/local）之别，而且实验分析来看，这个**差别非常明显**。

