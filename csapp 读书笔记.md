# csapp 读书笔记

# 第一章 

## 1.7 

每个进程看到的虚拟区如下:

内核虚拟存储器

栈(往下)

...

共享库

...

堆 (往上)

只读的 程序代码和数据



多核处理器:  独立的 L1, L2 ,  共享L3   

超线程:

 让一个核执行两个线程,常规处理器要用20000 个cycle 来做线程转换,而超线程可以用一个周期.

[Welcome to LWN.net [LWN.net\]](https://lwn.net/)

linux 的文档.

# 第二章

## 2.1

typedef int * int_pointer 可以让可读性好很多.





# 第三章程序的机器级表示

## 3.11

gdb , 命令行, 很多程序员会用ddd,  是gdb的拓展, 提供了图形化界面

所有的gnu包使用 -o2 优化编译的, 常常用循环来替代递归.这显著提高了程序的性能, 但是让源代码和机器代码的映射更加难以辨别, 难以调试. 



#### 缓冲区溢出攻击

1. 用一个指向攻击代码的指针覆盖返回地址, 执行ret的指令的效果就是跳转到攻击代码.

2. 使用系统调用启动一个shell, 给攻击者提供一个操作系统函数. 
3. 攻击代码会执行一些未授权的任务, 然后修复对栈的破坏, 然后第二次执行ret指令. 表面上正常返回.



#### 对抗 缓冲区溢出攻击

1. 栈随机化

如果攻击者确定一个常见的web服务器的栈空间, 就可以设计一个在许多机器上都能实施的攻击,  这叫安全单一化  .security monoculture

随机化的方法, 一个是随机malloc 一个空间, 让后面栈的位置都变化, 

Linux中栈随机化已经是标准行为, 是地址空间布局随机化的一种  address -space layout randomization.

有的攻击者会用 nop sled 反复用不同地址枚举来攻击, 

2. 栈破坏检测

 新版gcc 在栈帧中存储一个随机产生的canary 值, 也叫guard value , 在恢复寄存器状态和从函数返回之前检测, 如果被改变了, 程序异常中止.

3. 限制可执行代码区域

引入 no execute 位, 把读和执行分开, 检查页是否可执行由硬件来完成, 效率上没有损失.

这三种方法性能代价小, 不需要程序员做任何特殊努力, 组合起来更加有效. 

x86-64被大多数amd和intel所支持,IA32是传统32位

-m64 可以让gcc 产生 x86-64的代码. 





# 第四章 处理器体系结构



## 4.4

现代处理器用了15或更多阶段的流水线, 









## 4.5



数据冒险:

暂停

转发

load/use 冒险不能只转发, 还是需要暂停一下,这叫load interlock



### 4.5.9

三个异常, 

1. halt 
2. 由非法指令和功能码组合
3. 访问非法地址.

 怎么中断指令流是很challenging的.

要求 异常指令后所有指令都不能影响系统状态.

多个异常同时出现

基本原则: 流水线中最深的指令发生的异常, 优先级最高.

### 4.5.13

多周期指令

浮点加法3或4个周期, 整数除法32个周期,我们需要额外的硬件来执行这些计算,还需要让流水线能协调.

如果逗留的话会让fetch和decode 暂停,影响性能.

一条指令进入译码阶段,    可以发射到特殊单元 ,



一个经典的处理器有两个L1 cache, 一个是icache 一个 dcache.

cache miss的时候, 流水线会简单暂停, 完全硬件处理短时间的cache miss

page fault 缺页的时候, 是硬件产生page fault 异常信号, 然后处理器调用操作系统的异常处理代码, 然后发起磁盘到主存的传送操作.完成后, 操作系统会返回原来的程序, 导致缺页的指令会重新执行. 这就是用异常处理来处理长时间的缺页(几百万个时钟周期)



硬件设计人员必须非常谨慎小心, 要仔细分析各种指令类型, 用系统的模拟 测试程序彻底测试.





## 第五章 优化程序性能

 编译器优化, 要考虑指针指向存储器中同一个位置的情况.

327页 
5。14。1
程序剖析是研究运行时间， unix的gprof 可以用来剖析 profiling程序。对于运行时间少于1秒的程序来说， 得到的数字不精确， 如果运行时间比较长还是比较精确的。
gcc －pg然后 gprof prog 可以看到函数调用的历史
剖析可以把一个3分钟的时间降低到一秒以下。 

即使我们把60%的部分提高到不需要任何时间， 也就1/0。4 = 2。5 
所以我们应该 提高系统更多部分的速度， 而不是对着一个地方优化很多。 
在循环中， 有一个变量存储结果 而不是 存了再取会有难以想象的加速，可以快好几倍。
\## 5。5 有时候为了速度， 经常必须要损害一些模块性和抽象性， 比如把get element 改为数组直接访问不检查越界

## 第六章

6。3。2
系统中处处有缓存， 寄存器， tlb， cache，虚拟存储器， 磁盘缓存， 网络缓存（network file system nfs， 客户）， 浏览器缓存， web缓存（代理服务器）

\### cache友好的代码编写
一是常见的情况下运行的快， 
二 是让每个循环内部cache miss数量最小。
三， 

## 第7章
声明为 static 的全局变量或者函数是模块私有的，不能被其他模块访问。 尽可能用static 来保护变量和函数。

###  unix 链接器

强符号， 弱符号， 
不允许多个强符号， 多个弱符号随机选择一个。 未初始化的全局变量是弱符号。 
静态库 从左到右解析， 如果定义符号的库在另一个引用符号的目标文件之前就有可能失败。 

### 7。9 加载可执行文件

loading： 把代码和数据拷贝到内存中， 然后跳转到 entry point 
代码段总是从 0x08048000 开始， 数据段是 。data 。bss 后接下来4kb对齐的地方， malloc向上， 用户栈从最大的合法用户地址开始， 向下增长， 栈上面是给内核保留。 

#### 加载器
unix系统中， 父进程生成一个子进程， 子进程 通过exec 系统调用 启动加载器，  加载器删除子进程现有的虚拟存储器段， 创建一组新的代码， 数据，堆和栈段。 通过将虚拟地址空间中的页映射到可执行文件的页大小的片chunk， 新的代码和数据段被初始化为 可执行文件的内容， 加载过程中没有从磁盘到内存的数据拷贝， 直到cpu引用一个被映射到虚拟页才会进行拷贝

### 7.10共享库shared library

Shared library也叫shared object， 是只读的，windows中是dll， linux后缀为。So， 

一个是所有可执行目标文件共享。So的代码和数据， 不用每个都复制。 其次， 内存中， 共享库。Text节的副本可以被不同的正在运行的进程共享。 

### 7。11 在application中加载和链接共享库

linux系统dlopen函数，

与位置无关的代码 pic，position－independent code
被编译为 pic 的共享库可以加载到任何地方， 也可以运行时被多个进程共享。

 7.11 从application运行时 加载

不用编译时把动态库链接， 而是运行时要求dynamic linker 加载和链接共享库。

用于： 分发软件， 更新软件， 构建高性能服务器， 

Web浏览器请求到达， 服务器动态加载和链接适当的函数， 运行时无需停止服务器就可以更新函数。

## 第八章 异常控制流

异常分为四类，

1 中断 interrupt 终端， 是处理器外I/O设备的信号， 异步的， 总是返回到下一条指令。

I/O 设备 向处理器芯片上一个引脚发送信号，把异常号放到bus上，这个异常号标识了引起中断的设备， 处理器发现中断引脚的电压变高了， 就从系统总线读取异常号， 然后调用适当的中断处理程序。

 

下面三个叫faulting instruction，是同步发生的。

2 trap 是有意的异常， 最重要的用途是用户程序和内核中提供系统调用 。 syscall指令会导致一个到异常处理程序的trap。

3 故障 ，如果可以处理，那就重新执行引起故障的指令， 否则返回到内核中的abort程序， 终止引起故障的应用程序。  常见的是缺页异常，读入内存后再执行指令。

4终止， 处理程序把控制返回给abort程序。终止application。

### 8.1 linux中的异常

0-31是intel架构师定义的异常， 32-255是操作系统定义的interrupt和 trap。

#### 故障和终止

除法错误， unix不会恢复而是会终止application报告为floating exception

一般保护错误，  通常引用了一个未定义的虚拟内存区域， 或者写一个只读的文本段， linux shell报告为段故障 segmentation fault。

缺页和 机器检查

#### 系统调用

Read， write， open，close， stat 获得文件信息， mmap 将内存页映射到文件 ， brk 重置堆顶， dup2复制文件描述符， pause 挂起进程直到信号到达， getpid 获得进程id， fork ， execve 执行， _exit 终止进程， wait4等待终止， kill。

Write（1，“helloworld”，10）； 相当于printf

### 8.2 进程

每个进程执行一段，然后被抢占preempted也叫暂时挂起，

#### 8.2.4

处理器用某一个控制寄存器中的一个模式位mode bit 来提供这种功能。 这个寄存器描述了进程当前的privilege，设置了模式位，进程就运行在内核模式，可以访问任何内存位置， 执行指令集中的任何指令。

 否则就在用户模式，不能执行privileged instruction， 比如 发起I/O操作。

Linux 利用/proc 文件系统允许用户模式进程访问内核数据结构， 比如 /proc/cpuinfo , 或者某个特殊的进程使用的内存段 / proc/<process-id>/maps。 2.6 版本的linux 引入/sys文件系统， 输出关于 bus和设备的额外的底层信息。

### 8.3

错误处理包装函数， 让代码简洁。 可以在网站上得到。

### 8.4 进程控制

获取进程id

Getpid， getppid

Fork 被父进程调用一次， 但是返回两次。 并发执行， 相同但是独立的地址空间。共享文件描述符。

两个fork 产生四个进程。

#### 8.4.3 回收子进程

进程终止后， 会保持zombie， 直到被父进程调用wait/waitpid回收reaped，内核才把子进程的退出状态传递给父进程，抛弃子进程。父进程终止了的话， 内核会安排init进程成为他的孤儿进程的养父(现在内核的处理不是直接由init接管了，应该是由关系最近的上位进程接管)， init 的pid为1， 是在系统启动时由内核创建的， 不会终止， 是所有进程的祖先。 不过， 长时间运行的程序， 如shell或者服务器， 总是应该回收zombie。

Waitpid 挂起调用进程的执行， 直到它的wait set中的一个子进程终止。 可以修改option来改变waitpid 的返回值

Statusp参数可以找到子进程的状态信息。 # include<sys/wait.h>

可以画出进程图的拓扑排序来研究fork waitpid等函数的输出。

Wait函数是waitpid的简单版本， wait（&status） = waitpid（-1，&status， 0）；

Pid[i] = Fork() 保存子进程的pid

Retpid = waitpid（pid[i++]，。。。） 按创建顺序来回收zombie子进程。

Sleep（100）如果被中断，返回剩下还要休眠的秒数

Pause（void）让调用函数休眠， 直到该进程收到信号。

Execve 在当前进程的context中加载并运行一个新的程序。 覆盖当前进程的地址空间， 继承它的文件描述符。有相同的pid。



### 8.5 信号

一个进程可以发送信号给他自己。 发送信号的原因： 内核检测到系统异常。 或者一个进程调用了kill函数 要求内核发送信号给目的进程。 

接收信号：  进程可以忽略信号，终止或者通过叫signal handler 的用户层函数来捕获信号。

一个发出但没有被接收的信号叫pending signal， 任何时候， 一个类型最多一个pending signal， 之后同一个类型的信号不会排队等待而是被丢弃， 一个进程可以阻塞接收某种信号。 内核给每个进程在pending位向量中维护pending信号的集合， blocked 位向量中维护被阻塞的信号集合。

#### 发送信号

Getpgrp 返回当前process group的进程组id

Setpgid 改变进程组

#### 接收信号

内核把进程p从内核模式转换到用户模式时， 它会检查p的未被阻塞的待处理信号的集合。

### 8.7 

Strace 打印程序轨迹

Pmap 显示进程的内存映射

Cat /proc/loadavg 看linux系统的平均负载。

 

 

## 9虚拟内存

1. 把主存看做存储在磁盘上的地址空间的高速缓存，主存只保存活动区域，根据需要在磁盘和主存之间来回传送数据。
2. 每个进程有一致的地址空间，简化了内存管理

3. 保护了每个进程的地址空间不被其他进程破坏。

 

Cpu产生虚拟地址， 然后address translation，通过 Cpu芯片上叫做mmu的专用硬件， 利用存放在主存中的查询表（page table）来动态翻译虚拟地址。 这个表内容由os管理。

 

虚拟页分为，未分配的，

缓存的： 缓存在物理内存中的

未缓存的：没有缓存在物理内存中的已分配页。

页表存放在物理内存中，

程序导致 page不断换入换出， 叫做抖动thrashing

可以用linux 的getrusage 函数检测缺页数量。

操作系统给每个进程一个独立的页表。

不同进程虚拟页面映射到相同的物理页面， 就可以实现共享代码和数据。

页表的pte page table entry 多一些许可位可以控制访问， sup位表示进程运行在supervisor超级用户模式才能访问该页，read和write位控制读写，如果违反了许可， 那就触发segmentation fault段错误。

 

页面命中：

1 处理器生成虚拟地址， 传给mmu

2 mmu生成pte地址， 然后问cache或者内存， 内存返回pte。

3 mmu构造物理地址， 传送给cache或者内存

4 cache 或内存返回数据

 

缺页：

1 处理器生成虚拟地址， 传给mmu

2 mmu生成pte地址， 问cache或者内存， 内存返回pte。

3 pte的有效位是0， mmu触发异常， cpu执行操作系统内核中的缺页异常处理程序。

4 缺页异常处理程序调出一个牺牲页， 调入新页面， 更新内存中的pte

5 缺页异常处理程序返回原来进程， 

6 处理器虚拟地址重新 传给mmu

 

大多数系统用物理地址来访问cache，mmu翻译visual address后，可以同时发送pte地址和物理地址给cache。

 

 

TLB 

为了更快查找pte，会在mmu中包含一个pte的缓存， 叫做translation lookaside buffer TLB。快表

这样 mmu 不用问cache或者内存，

 

多级页表

一级页表中的一个pte是空的， 那么相应的二级页表就不会存在。 许多未分配的空间就可以节省下来。

一级页表放内存， 需要时再创建二级页表， 减少了内存压力。

 

### 9.8 内存映射

进程2 把共享对象映射到它的地址空间，内核可以判断进程1已经映射了这个对象，就把进程2的pbe页表条目指向相应的物理页面。

私有变量可以用copy on write，依旧指向同一个物理页面， 直到一个进程企图写这个页面， 故障处理程序在物理内存再创建新副本，

































































































