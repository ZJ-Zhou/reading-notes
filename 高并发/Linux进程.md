# Linux进程

## 进程和线程

### 进程

对正在运行中的程序的抽象

#### 进程模型

所有计算机上运行的程序，包括操作系统，被组织为若干顺序进程，一个进程就是一个正在执行的程序实例。

进程包括程序计数器、寄存器和变量的当前值；每个进程都有各自的虚拟CPU和虚拟内存空间；CPU会在各个进程之间进行切换，进程在切换时将逻辑寄存器（进程表中）的值装载到CPU的物理寄存器（PC、SP……）上，切换至其他进程时再将物理寄存器的值放回进程的逻辑寄存器中

CPU的单个核一次只能运行一个进程

#### 进程的创建

- 系统初始化（init、systemd，pid=1，初始化时还会创建若干其它进程）
- 正在运行的程序执行了创建进程的系统调用（fork、clone，fork新进程时内存空间通过写时复制拷贝，可调用execve执行其他程序，父进程通过waitpid处理结束的子进程回收PCB，否则会产生僵尸进程）
- 用户请求创建一个新进程（运行可执行文件、在终端运行命令）
- 初始化批处理工作（大型机）

#### 进程的终止

- 正常退出：系统调用exit
- 错误退出：程序中的错误（除0等），可以自行处理
- 严重错误：系统错误（如找不到文件等）
- 被kill

#### 进程的层次结构

- Unix中，进程和所有子孙进程共同组成一个进程组，当用户用键盘发出信号后江北发送给当前与键盘相关的进程组中所有的成员，每个人进程可以分别捕获、忽略或默认处理（被kill）该信号；孤儿进程（父进程早于子进程结束）的父进程是init进程
- Windows中的所有进程都是平等的，父进程通过句柄空值子进程

#### 进程状态

- 运行态 running
- 就绪态 ready
- 阻塞态 waiting

#### 进程的实现

操作系统维护进程表，每个表项代表一个进程，该表项包含进程状态的重要信息，包括PC、堆栈指针、内存状态、打开的文件的状态、账号、地导读信息等等

![img](https://mmbiz.qpic.cn/mmbiz_png/libYRuvULTdUGos87DMJibWy8Kib1P4rzXkC2WWXEkRAKErcia0ib3Hia2DWsLtPRzqQLdt4Mo326QWfF7LfyXfcUApQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

中断处理的流程：

1. 硬件压入堆栈程序计数器等
2. 硬件从中断向量装入新的程序计数器
3. 汇编语言过程保存寄存器的值
4. 汇编语言过程设置新的堆栈
5. C 中断服务器运行（典型的读和缓存写入）
6. 调度器决定下面哪个程序先运行
7. C 过程返回至汇编代码
8. 汇编语言过程开始运行新的当前进程



### 线程

- 多个线程共享同一块地址空间
- 比进程更轻量级，创建和撤销都更容易，创建比进程快10-100倍
- 线程切换时需要交换的数据更少（不需要切换内存空间、快表tlb等，只需要切换寄存器和堆栈指针）

![img](https://mmbiz.qpic.cn/mmbiz_png/libYRuvULTdUGos87DMJibWy8Kib1P4rzXkciaYNEibh4VsBF79p911Fgt7Ca558CpyNyuppk9wt7DrvuN1Tfibysvmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 多线程解决方案

由一个调度线程向多个工作线程分配任务，worker模式

#### 线程实现

- 用户空间实现线程
- 内核空间
- 混合实现

### 进程间通信  IPC

问题：

- 消息传递
- 数据竞争
- 顺序

#### 信号

#### 管道

#### 共享内存

#### 先入先出队列（命名管道）

#### 消息队列

#### 套接字

### 进程调度

Linux线程分类： 实时先入先出、实时轮询、分时

算法：

- O(1)  选择活动数组中优先级最高的任务，失效/运行完时间片后移动到失效数组
- CFS 采用红黑树作为调度队列，按CPU时间排序，总是优先调度CPU时间最小的任务