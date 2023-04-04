- [io](#io)
  - [字节流](#字节流)
  - [字符流](#字符流)
  - [缓冲流](#缓冲流)
- [jvm](#jvm)
  - [垃圾回收](#垃圾回收)
    - [gc要点](#gc要点)
    - [并发漏标问题(待完善)](#并发漏标问题待完善)
    - [不同的垃圾回收器](#不同的垃圾回收器)
    - [导致内存溢出的情况](#导致内存溢出的情况)
- [juc](#juc)
  - [对象头](#对象头)
    - [mark word(待完善)](#mark-word待完善)
    - [重量级锁](#重量级锁)
    - [轻量级锁](#轻量级锁)
    - [偏向锁](#偏向锁)
    - [无锁](#无锁)
  - [线程池（待完善）](#线程池待完善)
    - [ExecutorService](#executorservice)
  - [线程组](#线程组)
  - [线程通信模式](#线程通信模式)
    - [happens-before](#happens-before)
    - [线程共享](#线程共享)
      - [volatile（待完善）](#volatile待完善)
  - [线程同步](#线程同步)
    - [互斥锁](#互斥锁)
      - [synchronized（待完善）](#synchronized待完善)
      - [ReentrantLock（待完善）](#reentrantlock待完善)
      - [condition（待完善）](#condition待完善)
      - [ReadWriteLock（待完善）](#readwritelock待完善)
      - [StampedLock（待完善）](#stampedlock待完善)
    - [信号量（待完善）](#信号量待完善)
      - [**P、V操作**](#pv操作)

# io

## 字节流

> **为什么字节流更快？**  

1. 字节流相比字符流有更大的缓冲区
2. 字节流直接操作底层字节，字符流还需要对字符进行编码和译码，耗费时间

**jdk9新增：**

- `readallbytes()` ：读取输入流中的所有字节，返回字节数组。
- `readnbytes(byte[] b, int off, int len)` ：阻塞直到读取 `len` 个字节。
- `transferto(outputstream out)` ： 将所有字节从一个输入流传递到一个输出流。

## 字符流

> **为什么要有字符流？**

    当文本中包含中文等字符时，其unicode编码包含2个字节(utf8包含3个字节)，字节流的读取一次只能读取一个字节，会发生乱码的问题

## 缓冲流

> **为什么要有缓冲流？**

    减少i/o次数，先将读取的字节或者字符存储在缓冲区(缓冲区是运行期间在内存中开辟，字节流通常是一个`8kb`的数组)，待缓冲区满或者调用`.flush`方法后将缓冲区数据发送，触发io中断

# jvm

## 垃圾回收

### gc要点

- 主要是********************堆内存********************，本地方法栈和jvm栈会自动释放已结束的方法所占内存；
- 通过可达性分析算法、三色标记法来标记存活对象，回收未标记对象；
- 用分代回收思想，根据对象特性将回收区域分为新生代和老年代，不同区域不同回收策略；
- 根据gc回收规模分为：minor gc，mixed gc，full gc
  - minor gc：新生代，异步，不中断主程序
  - mixed gc：新生代+老年代，g1 gc特有
  - full gc：新生代+老年代，中断主程序

### 并发漏标问题(待完善)

### 不同的垃圾回收器

- **parallel gc**
  - eden内存不足发生minor gc，标记复制，同时**stw（stop-the-world）**
  - old内存不足发生full gc，标记复制stw
  - 吞吐量优（但会经常stw，响应时间差）
- **cms gc**（concurrentmarksweep gc）：
  - 老年代的并发标记清除（会有内存碎片问题）
  - 如果失败则full gc
  - 响应时间优（因为和主程序并发）
- **g1 gc**
  - 兼顾响应时间和吞吐量

### 导致内存溢出的情况

- **线程池中有线程阻塞**，但是调用的线程池方法中对runnable对象的数量没有限制，导致runnable对象无限加载，直至outofmemorry；
- **查询数据量太大导致内存溢出**，例如对数据库查询没有加`limit`限制，一次用户查询返回的数据量可能多至`300m`，查询用户多后则可能触发oom；
- **动态生成类导致的内存溢出**，例如定义了一个静态的对象，由于它是静态即根对象，若通过该静态对象调用方法无限制的加载新的对象，这些对象都会被gc通过*可到达性标记*进行标记，这样垃圾回收无法回收这些对象而导致oom；
  - 解决：可以将静态对象设置为方法内部变量，这样gc便可以回收其在堆上新创建出的对象

# JUC

## 对象头

### mark word(待完善)

### 重量级锁

**线程饥饿**:指的是线程长时间竞争不到锁，无法获取CPU时间片和资源的情况

- 通常由`synchronized`实现，关键在于多线程竞争锁时，会产生阻塞和唤醒线程的开销
- 保证公平性，按照“先来先服务”原则对阻塞的线程排序

> 自旋和阻塞的区别在于：被阻塞的线程会被挂起，从而不占用CPU资源；自旋锁仍占用CPU资源，在这期间重复执行简单的操作并尝试获取锁
>

### 轻量级锁

- 通常由**CAS**机制实现，不会被阻塞，期间仍能利用CPU资源执行操作，效率高
- 轻量级锁自旋等待一定次数后会升级为重量级锁

### 偏向锁

### 无锁

## 线程池（待完善）

### ExecutorService

**常用功能：**

- 创建线程池
  - `FixedThreadPool`：线程数固定的线程池；
  - `CachedThreadPool`：线程数根据任务动态调整的线程池；
  - `SingleThreadExecutor`：仅单线程执行的线程池。
  - `ScheduledThreadPool`：计划性线程池，可配置定时任务
- 提交任务
- 执行定时任务

```java
// 创建线程池，线程数为10
ExecutorService executor = Executors.newFixedThreadPool(10);
// 提交任务给线程池
executor.submit(new RunnableTask());  

// 创建有定时功能的线程池，线程数为10
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
// 延迟 1 秒执行
scheduledExecutorService.schedule(() -> {
    // 代码块
}, 1, TimeUnit.SECONDS);

// 延迟 1 秒后每隔 2 秒执行一次
scheduledExecutorService.scheduleAtFixedRate(() -> {
    // 代码块
}, 1, 2, TimeUnit.SECONDS);
```

**Future**

- `Future`是接口，`FutureTask`是其实现类，代表异步计算的结果
- 把`Runnable`或者`Callable`接口的实现类提交(`.submit()`)给线程池后，会返回一个`FutureTask`对象，如右

```java
Future<String> future = executor.submit(task);
String result = future.get(); // 等待任务执行并获取结果
```

## 线程组

每个线程**必然**存在于一个`ThreadGroup`中，如果没有显示指明，则该线程的线程组**默认为父线程的线程组。**例如在`main()`中创建一个`Thread`，其线程组名字则设置为其父线程`main`

`ThreadGroup`是**向下引用**的树结构，目的是**防止“上级”线程被“下级”引用而无法有效的GC**

## 线程通信模式

|                           |                                                 |
|-------------------|---------------------------------------|
| 消息传递并发模型          | 线程之间没有公共状态，通过发送消息进行线程间通信 |
| 共享内存并发模型（JVM采用） | 读写内存中的公共状态通信，需要互斥               |
|                           |                                                 |

### happens-before

- 具有**可见性**，即线程A happens-before 线程B，则其在公共区的执行结果对线程B是可见的
- 若执行结果一致，happens-before原则不影响JVM**重排序**

### 线程共享

#### volatile（待完善）

用来修饰**变量**，不能修饰方法

- 保证变量可见性
- 禁止指令重排序
- 不保证原子性
- 内存屏障
- 实现高效的单例模式、双重检查锁定等

## 线程同步

### 互斥锁

#### synchronized（待完善）

考点

#### ReentrantLock（待完善）

- 具备**尝试获取锁**的功能，例如下述代码尝试获取锁，等待1秒后获取不到则放弃

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

#### condition（待完善）

#### ReadWriteLock（待完善）

#### StampedLock（待完善）

### 信号量（待完善）

> 待完善：如何实现互斥和同步，具体怎么操作
>

#### **P、V操作**

- `sem`表示资源的数量
- P：`sem`减1后，若`sem<0`则阻塞，否则线程继续
- V：`sem`加1后，若`sem<=0`则唤醒阻塞的线程
- 具有原子性

# 反射

## 动态代理

创建动态代理实例`Proxy.newProxyInstance(classLoader, interfaces, invocationHandler)`三个参数分别代表如下：

- `classLoader`：目标对象的类加载起
- `interfaces`：目标对象实现的所有接口
- `invocationHandler`：如何重写目标对象的方法过程

其中`invocationHandler`是一个接口，要实现`invoke`方法来重写目标对象的方法过程。

### JDK代理

- 代理对象和目标对象实现同个接口

### cglib代理

- 代理对象继承自目标对象

# 注解
## 常用注解
- `@Target`: 指定该注解可用于哪些元素上，如方法、字段、类等
  - ElementType.TYPE：可以应用于类、接口和枚举上。
  - ElementType.FIELD：可以应用于字段上。
  - ElementType.METHOD：可以应用于方法上。
  - ElementType.PARAMETER：可以应用于方法参数上。
  - ElementType.CONSTRUCTOR：可以应用于构造函数上。
  - ElementType.LOCAL_VARIABLE：可以应用于局部变量上。
  - ElementType.ANNOTATION_TYPE：可以应用于注解上。
  - ElementType.PACKAGE：可以应用于包声明上。
  - ElementType.TYPE_PARAMETER：可以应用于类型参数上。
- `@Retention`: 指定注解的存活时间，如编译、类加载、运行时
  - RetentionPolicy.SOURCE：注解仅在源代码中保留，编译时会被丢弃。
  - RetentionPolicy.CLASS：注解在编译时保留，在类加载时被丢弃。
  - RetentionPolicy.RUNTIME：注解在运行时保留，可以通过反射机制读取。