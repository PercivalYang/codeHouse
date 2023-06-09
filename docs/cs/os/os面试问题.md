- [用户态和内核态](#用户态和内核态)
  - [用户态切换到内核态的方式](#用户态切换到内核态的方式)
  - [什么时候需要系统调用](#什么时候需要系统调用)
- [多线程](#多线程)
  - [线程的同步方式有哪些](#线程的同步方式有哪些)
  - [PCB包含哪些信息](#pcb包含哪些信息)
  - [产生死锁的条件是什么](#产生死锁的条件是什么)
  - [如何避免死锁？银行家算法是什么？（待完善）](#如何避免死锁银行家算法是什么待完善)
- [多进程](#多进程)
  - [什么是僵尸进程和孤儿进程](#什么是僵尸进程和孤儿进程)


## 用户态和内核态

### 用户态切换到内核态的方式

1. 系统调用: 用户程序主动要求切换到内核态，执行读写磁盘等操作；
2. 中断: 如用户程序读写磁盘结束，会给CPU发中断信号，CPU会暂停执行下一条指令，切换到内核态，执行中断处理程序，处理完后再切换回用户态；
3. 异常: 用户程序发生异常时，会触发切换到内核态，执行异常处理程序，处理完后再切换回用户态，比如缺页异常。

### 什么时候需要系统调用

1. 设备管理：如读写磁盘，读写网络等；
2. 文件管理：文件的读、写、创建、删除
3. 进程管理
4. 内存管理

## 多线程

### 线程的同步方式有哪些

1. 互斥锁：例如Java中的`synchronized`；
2. 读写锁：允许多个线程同时读，但是只允许一个线程写，例如Java中的`ReadWriteLock`；
3. 信号量：允许多个线程同时访问，但是要控制访问的最大线程数；
4. 屏障：允许多个线程同时访问，但是要等到所有线程都到达屏障处，才能继续执行；
5. 事件：通过通知方式来保持多线程同步(Wait/Notify)

### PCB包含哪些信息

PCB (Process Control Block): 进程控制块，包含了：

- 进程的**描述信息**，包括进程的名称、标识符等等；
- 进程的**调度信息**，包括进程阻塞原因、进程状态（就绪、运行、阻塞等）、进程优先级（标识进程的重要程度）等等；
- 进程对**资源的需求情况**，包括 CPU 时间、内存空间、I/O 设备等等;
- 进程**打开的文件信息**，包括文件描述符、文件类型、打开模式等等;
- 通用寄存器、指令计数器、程序状态字 PSW、用户栈指针等。

### 产生死锁的条件是什么

1. 互斥：资源是非共享状态
2. 占有并等待：一个进程占有并等待另一个资源，且这个“另一个“资源被其他进程占有；
3. 非抢占：资源不能被抢占，只能由占有它的进程自己释放；
4. 循环等待：存在一个进程等待链，链中每个进程都在等待下一个进程占有的资源。

> 这四个条件是产生死锁的**必要条件**

### 如何避免死锁？银行家算法是什么？（待完善）

1. 死锁的预防
2. 死锁的避免
3. 死锁的检测：类似乐观锁的思路，检测到系统出现死锁时再去接触死锁，检测的方式是：资源分配图、等待图等。


## 多进程

### 什么是僵尸进程和孤儿进程

父进程在子进程结束时，需要调用`wait()`或`waitpid()`等来获取子进程状态信息(PCB)并释放PCB占用的资源。

- 僵尸进程：如果子进程结束但没有被父进程回收，导致PCB资源一直占用系统资源，就成了僵尸进程。

- 孤儿进程：父进程意外终止，未及时回收子进程占用的系统资源，导致子进程成为孤儿进程。
