- [内存结构](#内存结构)
  - [程序计数器PC](#程序计数器pc)
  - [Java虚拟机栈(Java 栈)](#java虚拟机栈java-栈)
    - [活动栈](#活动栈)
    - [局部变量表](#局部变量表)
    - [操作数栈](#操作数栈)
    - [静态链接](#静态链接)
    - [动态链接](#动态链接)
    - [非虚方法](#非虚方法)
    - [Java虚拟机栈的特点](#java虚拟机栈的特点)
  - [本地方法栈(C栈)](#本地方法栈c栈)
  - [堆](#堆)
    - [特点](#特点)
    - [新生代和老年代](#新生代和老年代)
    - [Full GC和Major GC](#full-gc和major-gc)
    - [逃逸分析](#逃逸分析)
    - [TLAB](#tlab)
    - [字符串常量池](#字符串常量池)
    - [四种引用方式](#四种引用方式)
  - [方法区](#方法区)
    - [元空间和永生代](#元空间和永生代)
    - [特点](#特点-1)
    - [运行时常量池](#运行时常量池)
  - [直接内存(堆外内存)](#直接内存堆外内存)
    - [堆外内存的优缺点](#堆外内存的优缺点)
- [对象](#对象)
  - [对象的内存布局](#对象的内存布局)
    - [对象头](#对象头)
    - [实例数据](#实例数据)
    - [对齐填充](#对齐填充)
  - [对象的访问方式](#对象的访问方式)
    - [句柄访问](#句柄访问)
    - [直接访问](#直接访问)
- [垃圾回收](#垃圾回收)
  - [gc要点](#gc要点)
  - [不同的垃圾回收器](#不同的垃圾回收器)
  - [导致内存溢出的情况](#导致内存溢出的情况)
  - [判断对象是否存活](#判断对象是否存活)
    - [引用计数法](#引用计数法)
    - [可达性分析算法](#可达性分析算法)
  - [引用的种类](#引用的种类)
    - [强引用](#强引用)
    - [软引用](#软引用)
    - [弱引用](#弱引用)
    - [虚引用](#虚引用)
  - [回收无效对象](#回收无效对象)
    - [回收堆中无效对象](#回收堆中无效对象)
    - [回收方法区内存](#回收方法区内存)
  - [垃圾回收算法](#垃圾回收算法)
    - [分代回收理论](#分代回收理论)
    - [标记-清除算法](#标记-清除算法)
    - [标记-复制算法(新生代)](#标记-复制算法新生代)
    - [标记-整理算法(老年代)](#标记-整理算法老年代)
- [类加载](#类加载)
  - [类的生命周期](#类的生命周期)
  - [类加载过程(待完善)](#类加载过程待完善)
    - [加载](#加载)
    - [验证](#验证)
    - [准备](#准备)
    - [解析](#解析)
    - [初始化](#初始化)
    - [类卸载](#类卸载)
  - [类加载器](#类加载器)
    - [类型](#类型)
    - [双亲委派模式](#双亲委派模式)
- [IDEA的操作](#idea的操作)
  - [设置JVM参数](#设置jvm参数)
  - [分析堆内存](#分析堆内存)
  - [垃圾回收](#垃圾回收-1)
    - [并发漏标问题(待完善)](#并发漏标问题待完善)

# 内存结构

## 程序计数器PC

**定义**

是当前线程正在执行的那条**字节码**指令的地址。若当前线程正在执行的是一个本地方法，那么此时程序计数器为`Undefined`

[字节码解释器]()通过改变PC来依次读取指令，实现流程控制

**特点**

- 线程私有
- 生命周期随着线程创建而创建、结束而销毁
- 唯一不会`OutOfMemoryError`的内存区域

## Java虚拟机栈(Java 栈)

为每个即将运行的Java方法创建一个**栈帧**区域，如下图所示：

![20230412150205](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230412150205.png)

### 活动栈

当前栈帧又称**活动栈**，即当前正在执行的方法，只有这个栈帧中的本地变量能够被**操作数栈**使用。当方法结束栈帧被弹出时，当前方法的返回值会作为下一个活动栈中操作数栈的一个操作数。

### 局部变量表

用于存储方法参数、定义在方法体内的局部变量。如图所示包含了3种数据类型：基本类型、对象引用和return Address

对象引用的例子如下：

```java
public void exampleMethod() {
   // myObject是指向对象起始地址的reference类型
   Object myObject = new Object();
   // objRef是只想一个代表对象句柄的reference类型
   Object objRef = myObject;
}
```

> return Address: 通常视为一个指向地址的指针，指示程序执行完方法后该回到哪个位置继续执行

表的容量大小在**编译期间**就定下来了，方法运行期间表的大小不会改变

基本的存储单位是：`slot`。32位数据类型占1个slot，64位数据类型(long, double)占2个slot。

对于slot：

- JVM会对每一个slot分配一个访问索引，通过索引可以访问到局部变量值
- 当一个局部变量过了作用域后，它所占用的slot是可以被新的局部变量覆盖
- 若活动栈是构造方法，那么第一个slot是用来存储`this`的引用的 (这个目的是什么？)

局部变量表中直接或间接引用的对象，都不会被GC回收，因为这些变量是根结点。

### 操作数栈

用于存储计算过程中的中间结果，也是方法调用时的参数传递。操作数栈的容量大小也是在**编译期间**就确定下来了。32位数据类型占一个栈单位，64占两个栈单位。

- 栈顶缓存：因为操作数存储在内存中，为提高效率减小读写内存的次数，将栈顶元素缓存到CPU寄存器中来提高执行效率；
- 存储单位是栈，访问的方式不再是通过索引访问，而是入栈、出栈的方式，例如子程序返回的值会先入主程序的操作数栈，然后再出栈返回给主程序的变量值。

### 静态链接

被调用的方法在编译期可知，且运行期不变，称为静态链接。如：`System.out.println()`等静态方法。

> 会直接在编译期将调用方法的符号引用转换为直接引用（称为**早期绑定**）

### 动态链接

即将调用方法的符号引用转换为其在内存地址中的直接引用。被调用的方法无法在编译期间确定，例如接口、继承和反射等，只能在运行期将调用方法的符号引用转换为直接引用（称为**晚期绑定**）。这种称为动态链接。

> 接口和继承是当子类实现/重写了父类的方法时，可能会出现动态链接

### 非虚方法

非虚方法通常包括：静态方法、`private`方法、`final`方法、构造方法等。因为这些方法无法被子类重写，在编译期就可以确定方法的版本，因此就不存在动态链接。

除开上述提到的非虚方法，Java中任意一个普通方法都是虚方法（晚期绑定）

**虚方法表**

为了提高执行效率，避免每次动态分配过程都要去类中方法的元数据中搜索，JVM会为每个类维护一个虚方法表。

- 表中存放各虚方法的实际入口
- 表在类加载的**链接阶段**创建和初始化

**方法重写的本质**（[待完善](https://doocs.gitee.io/jvm/01-jvm-memory-structure.html#%E6%96%B9%E6%B3%95%E7%9A%84%E8%B0%83%E7%94%A8)）

### Java虚拟机栈的特点

- 会出现的两种异常: `StackOverFlowError`, `OutOfMemoryError`
  - `StackOverFlow`: 设置了Java虚拟机栈的大小不允许**动态扩展**，当线程请求的栈深度超过Java虚拟机栈的最大深度时会抛出该异常；
  - `OutOfMemory`: 允许Java虚拟机栈动态扩展，但是当扩展到内存用光的时候，就会抛出该异常
- Java虚拟机栈是线程私有的，生命周期同线程一样

## 本地方法栈(C栈)

**定义**

本地方法栈是为运行`native`方法准备的空间，`native`方法通常是由C语言实现。

与Java虚拟栈功能类似，两者都是内存模型。区别在于Java虚拟栈为Java方法服务，本地方法栈为`native`方法服务。

本地方法执行时，同样会创建活动栈用于存放局部变量表、操作数栈、动态链接、方法出口信息等。方法结束后出栈释放资源。同时也会抛出`StackOverFlow`和`OutOfMemory`异常。

## 堆

### 特点

- 线程共享的，在JVM启动时创建，是GC的主要区域
- 分为新生代、老年代
- 堆可以在运行时动态扩展或收缩，但是在扩展时可能会出现`OutOfMemory`异常
- 堆可以处于物理不连续的内存空间，但是逻辑上是连续的

### 新生代和老年代

- 老年代生命周期更长；
- 新生代与老年代空间**默认比例 1:2**。XX:NewRatio=2，用来设置新生代和老年代内存大小比例，此处表示老年代的内存大小是新生代的2倍，则新生代占整个堆内存的1/3；
- HotSpot 中，Eden 空间和另外两个 Survivor 空间缺省所占的比例是：8:1:1。
- 几乎所有的 Java 对象都是在 Eden 区被 new 出来的，Eden 放不了的大对象，就直接进入老年代了。

### Full GC和Major GC

**触发条件**

- 老年代和方法区内存不足 $\rightarrow$ 触发Full GC；
- 回收新生代区会用Minor GC，当老年代内存还是不够时触发Major GC

**STW(Stop The World)**

- Full GC和Major GC都会触发STW，即在GC之前，所有的线程都会被暂停，直到GC结束，所有的线程才会被唤醒；
- Full GC STW 时间最长，Major GC STW 时间次之，Minor GC STW 时间最短。

### 逃逸分析

**方法逃逸**指对象定义在一个方法中，但是它可能被外部方法所引用，例如作为参数传递给外部方法，则称作方法逃逸，例如：（类似的还有**线程逃逸**）



```java
public static StringBuffer createStringBuffer(String s1, String s2) {

    StringBuffer s = new StringBuffer();

    s.append(s1);

    s.append(s2);

    // s是方法内的局部变量，但是代码将s作为返回值
    // 即s可能会被外部方法所引用，因此发生了`方法逃逸`
    return s;
}
```

逃逸分析是一种基于**指针分析**的**优化技术**，它的目的是分析**对象的作用域**，以此来决定是否在栈上分配对象，还是在堆上分配对象。

**标量替换**

标量对应Java中的基本数据类型，意思是不可再分的量。与之对应的就是**聚合量**，即可再分的量，如数组、对象等。

**替换过程是通过逃逸分析确定该对象不会被外部访问，JVM在方法内不会创建该对象，而是分解对象的成员变量为方法内的替代变量**。这样就不需要在堆上分配内存了，这些替代变量在栈或寄存器上分配空间

- 对象和数组不一定会分配在堆上，如果对象不会逃逸出方法，那么它就分配在栈上，这样就不需要GC来回收了。

**参数设置**

- `-XX:+DoEscapeAnalysis`：开启逃逸分析，jdk1.7之后默认开启；（`+`换成`-`就是关闭）

### TLAB

全称：Thread Local Allocation Buffer，即线程本地分配缓冲区。

因为堆是所有线程共享的，每次线程在堆上分配内存时都需要进行同步（JVM通过CAS实现同步）。

为了减少同步的开销，每个线程**预先**在堆中分配一块小内存，等到TLAB用完时分配新的TLAB时，才需要进行同步。

如果线程在TLAB中分配内存失败了，会使用加锁的机制来保证原子性。

- `-XX:+UseTLAB`: 开启TLAB，默认开启
- `-XX:+TLABSize`: 设置TLAB的大小，默认为1MB

### 字符串常量池

JDK1.6及之前，字符串常量池在永久代中，JDK1.7及之后，字符串常量池在堆中。

移到堆中的目的是因为方法区中的GC效率太低，只有在Full GC时才会执行。而字符串常量池中通常有大量新创建的字符串等待回收，放到堆中能更快的被回收，减小内存溢出风险。

### 四种引用方式

- 强引用：`Object obj = new Object()`，`obj`就是强引用变量，除非将`obj`赋值为`null`，否则即使JVM发生OOM，GC也不会回收强引用变量指向的堆上的变量；
- 弱引用：只要发生GC，都会进行回收
- 软引用：只有在内存不足时才会进行回收
- 虚引用：不会决定对象生命周期，该对象随时可能被GC回收

## 方法区

方法区是堆的一个**逻辑部份**。存放以下信息：

- 已经被虚拟机加载的类信息、字段信息、方法信息
- 常量
- 静态变量
- 即时编译器编译后的代码

### 元空间和永生代

方法区是抽象概念，类似Java中的接口；它的具体实现对应着JDK1.7的永生代和JDK1.8之后的元空间。

元空间使用的是本地内存，不受JVM内存的限制，只受本地内存的限制。这样的好处是可以存放更多的信息

### 特点

- 线程共享
- 永久代：方法区中的信息需要长期存在，称为永久代
- 回收效率低：变量都是永生代需要长期存在，因此一次GC只会有少数变量会被回收；
- 可以固定大小/动态扩展/不实现GC等。

### 运行时常量池

常量就存在其中。`.class`文件除了有类的版本、字段、方法、接口等描述信息，还有**常量池表**(Constant Pool Table)，用于存放编译期生成的字面量与符号引用，将在类加载后存入方法区中的运行时常量池。

同时运行时常量池具备**动态性**，可以在运行期间将新的常量加入池中，例如通过`String`的`intern()`方法。

> 用`new`显式创建字符串时，Java会创建新的实例对象(不论字符串常量池中是否存在该字符)，`intern`方法是将该String对象添加到字符串池中，如果池中已经存在该对象，则返回池中的对象。

## 直接内存(堆外内存)

JDK1.4种新加入**NIO**(New Input/Output)，引入一种基于管道和缓冲区的I/O方式。可以使用Native函数库直接申请堆外内存，然后用存储在Java堆中的`DirectByteBuffer`来引用这块堆外内存进行操作。

NIO的操作是非阻塞的，因此可以通过一个线程来管理多个通道，从而提高系统的并发性能。

### 堆外内存的优缺点

优点：

- 改善GC回收停顿。因为堆外内存由操作系统管理，通过使用堆外内存减小堆内内存规模，可以减小GC回收停顿的影响；
- 提高读写性能，减小拷贝次数。

# 对象

## 对象的内存布局

HotSpot的对象内存布局如下：

![](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230415221544.png)

### 对象头

记录了对象的元信息，包括：

`makrWord`:

- 哈希码
- GC 分代年龄
- 锁状态标志
- 线程持有的锁
- 偏向线程 ID
- 偏向时间戳

`Class对象指针`: 指明对象属于什么类

`数组长度(Optional)`：如果是数组，记录数组长度

### 实例数据

成员变量的值，其中包括父类成员变量和本类成员变量。

### 对齐填充

为了保证对象的大小是8字节的整数倍，可能会在对象头和实例数据之间添加一些填充字节。

> HotSpot VM 的自动内存管理系统要求对象的大小必须是 8 字节的整数倍，提高对象的访问效率
>
> 因为对齐填充帮助Java堆的内存是规整的，所有空闲区在一边，被使用区在另一边，中间有一个指针作为分界点。下次分配内存只用将指针移动和对象大小相同的距离，这个操作称为**指针碰撞**。

## 对象的访问方式

### 句柄访问

句柄的访问可以理解为间接访问，即在堆中申请一个"访问代理"，代理中包含了对象的实例数据和类型数据各自的具体地址信息，然后通过访问代理来访问对象。如下图所示

![20230417173021](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417173021.png)

### 直接访问

直接访问是省去了代理，直接访问对象的实例数据和类型数据。如下图所示

![20230417173136](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417173136.png)

# 垃圾回收

## gc要点

- 主要是**堆内存**，本地方法栈和jvm栈会自动释放已结束的方法所占内存；
- 通过可达性分析算法、三色标记法来标记存活对象，回收未标记对象；
- 用分代回收思想，根据对象特性将回收区域分为新生代和老年代，不同区域不同回收策略；
- 根据gc回收规模分为：minor gc，mixed gc，full gc
  - minor gc：新生代，异步，不中断主程序
  - mixed gc：新生代+老年代，g1 gc特有
  - full gc：新生代+老年代，中断主程序

## 不同的垃圾回收器

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

## 导致内存溢出的情况

- **线程池中有线程阻塞**，但是调用的线程池方法中对runnable对象的数量没有限制，导致runnable对象无限加载，直至outofmemorry；
- **查询数据量太大导致内存溢出**，例如对数据库查询没有加`limit`限制，一次用户查询返回的数据量可能多至`300m`，查询用户多后则可能触发oom；
- **动态生成类导致的内存溢出**，例如定义了一个静态的对象，由于它是静态即根对象，若通过该静态对象调用方法无限制的加载新的对象，这些对象都会被gc通过*可到达性标记*进行标记，这样垃圾回收无法回收这些对象而导致oom；
  - 解决：可以将静态对象设置为方法内部变量，这样gc便可以回收其在堆上新创建出的对象

## 判断对象是否存活

### 引用计数法

在对象头中维护了一个`counter`计数器，对象被引用一次则计数器+1，引用失效一次则-1。当计数器为0时，说明对象不再被引用，可以被回收。

虽然该方法效率高，实现简单。但Java并没有采用该算法，原因是它无法解决对象之间循环引用的问题。例如：

```java
public class ReferenceCountingGC {
    public Object instance = null;
    private static final int _1MB = 1024 * 1024;
    private byte[] bigSize = new byte[2 * _1MB];

    public static void main(String[] args) {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
        System.gc();
    }
}
```

从上面的代码可以看出，objA和objB都已经被赋值为`null`无法再访问，但是由于他们互相之间有引用导致`counter`计数器不为0，如果采用引用计数法的话，在堆上新建的两个`ReferenceCountingGC`对象将无法被回收。

> 注意这里回收的是堆上的`ReferenceCountingGC`实例对象，不是回收虚拟机栈上的局部变量objA和objB!

### 可达性分析算法

该算法的基本思路是通过一系列的称为`GC Roots`的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到`GC Roots`没有任何引用链相连时，则证明此对象是不可用的。

在Java中，可以作为`GC Roots`的对象包括：

- Java虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中常量引用的对象
- 方法区中静态属性引用的对象

与引用计数法相比，可达性分析算法不会出现对象之间循环引用的问题，例如上面的objA和objB问题，objA和objB都是Java虚拟机栈中的局部变量，分别在初始化的使用引用了在堆上新建的`ReferenceCountingGC`对象。而当objA和objB被赋值null时，即将引用断开后，堆上的这两个对象都是不可达的，可以被GC回收。

## 引用的种类

JDK1.2之后对引用类型细分为4种如下所示，我们将按引用强度从强到弱的顺序进行讲解

### 强引用

类似`Objcet obj = new Object()`都是强引用，只要强引用存在，垃圾回收器永远不会回收掉被引用的对象。

### 软引用

相比强引用弱一些，只有当内存不足时，垃圾回收器才会回收掉被软引用关联的对象。在JVM抛出`OutOfMemory`异常前会清理所有的软引用，如果内存还是不足，才会抛出异常。

### 弱引用

比软引用弱一些，不管内存是否足够，GC都会回收掉被弱引用关联的对象。

### 虚引用

最弱的一种引用关系，无法通过虚引用来获取对象，唯一的作用是在对象被回收时收到一个系统通知。

## 回收无效对象

### 回收堆中无效对象

在对象被回收前，JVM会先判断**是否有必要执行该对象的`finalize`方法**，当：

- 对象没有Override `finalize`方法;
- `finalize`方法已经被JVM调用过一次。

这时候JVM不会去执行对象的`finalize`方法，而是会直接回收对象。否则对象会被放入一个`F-Queue`队列中，由一个低优先级的线程去执行该对象的`finalize`方法，但该线程不会保证`finalize`一定会被执行完，因为可能会发生死循环和异常等让其他对象永久等待，因此出现异常情况后JVM会直接清除对象。

> 不推荐使用`finalize`方法，因为它的执行时机不确定，而且会影响GC的性能。

### 回收方法区内存

方法区内保存的是类信息、常量和静态变量等生命周期较长的对象，而GC主要清除两种垃圾：

- 废弃常量
- 无用的类

**如何判断废弃常量**

只要常量不被任何变量或对象引用，则认为该常量是废弃常量，可以被回收。

**如何判断无用的类**

判断无用的类，用于清除方法区中的类信息，条件如下：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；
- 加载该类的ClassLoader已经被回收；
- 该类的`Class`对象没有被引用，无法通过反射访问该类。

## 垃圾回收算法

### 分代回收理论

是目前大多数垃圾回收器采用的理论，即将堆内存划分为新生代和老年代两个区域：

- 老年代：存放经历过多次GC而没有消亡的对象；
- 新生代：存放新创建的对象，分为Eden区和两个Survivor区。

但分代理论就存在**跨代引用**的问题，即老年代的对象可能会引用新生代。这样采用可达性算法不仅要遍历GC Roots，还要遍历老年代来确保被老年代引用的新生代不会被清除，这样会导致GC效率降低。

解决方案是在新生代维护一个`Remembered Set`用于记录老年代中哪些内存区域引用了新生代，这样触发Minor GC后只需要将`Remembered Set`中的老年代加入GC Roots即可。

### 标记-清除算法

通过遍历GC Roots对存活的对象进行标记，然后清除未标记的对象。存在的问题有：

- 会随着标记对象数量的增长，而导致效率降低；
- 内存碎片问题：清除后会产生大量不连续的内存碎片，导致无法分配较大的对象。

### 标记-复制算法(新生代)

最开始时又称为"半区复制算法"，即将新生代划分为两个区域，每次只使用其中一个区域，当这个区域满了后，将标记为存活的对象复制到另一个区域，然后清除该区域中的所有对象，这样就解决了内存碎片问题。但研究表明98%的新生代对象熬不过第一轮GC，因此半区的划分太浪费内存空间了。

于是便有了分为一个`Eden`区和两个`Survivor`区的划分，每次使用Eden区和其中一个Survivor区，当出发Minor GC后，会将Eden和Survivor中存活的对象复制到另一个Survivor区，然后清除Eden和Survivor中的所有对象。

Eden, Survivor划分的比例大小为：8:1:1

**分配担保**

如果标记的存活对象大于10%，用于存放对象的Survivor区不足以存放所有存活对象，这时候会直接将存活对象放入老年代中，这就是分配担保。

### 标记-整理算法(老年代)

因为老年代采用标记-复制算法还需要额外的空间进行分配担保，同时老年代对象存活率高，复制的方法效率低，因此采用标记-整理算法。

**整理**的过程是将存活的对象向一端移动，按照内存地址依次排列，然后清除掉边界以外的内存。

但相比标记-清除算法，标记-整理算法的效率更低，但不存在内存碎片问题。像CMS收集器采用的是"和稀泥式"，即先采用标记-清除算法，直到碎片化程度大到影响对象分配时，再采用标记-整理算法回收一次。

# 类加载

## 类的生命周期

![20230531204657](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230531204657.png)

## 类加载过程(待完善)

### 加载

### 验证

### 准备

### 解析

### 初始化

### 类卸载

## 类加载器

### 类型

JVM中有三个自带的类加载器：

- `BootstrapClassLoader`(启动类加载器): 由C++实现，在Java中表示为`null`，没有父级，主要用于加载：
  - JDK核心库：`$JAVA_HOME/lib`下的`rt.jar`, `resource.jar`等；
  - 被`-Xbootclasspath`参数指定的路径中的类库；
- `ExtensionClassLoader`(扩展类加载器): 由具体的Java类实现，父级为`BootstrapClassLoader`，主要用于加载：
  - `$JAVA_HOME/lib/ext`目录下的jar包；
  - `java.ext.dirs`系统变量指定的路径中的jar包；
- `AppClassLoader`(应用程序类加载器): 由具体的Java类实现，父级为`ExtensionClassLoader`，主要用于加载：
  - 当前应用classpath下的所有jar包

每个Java类都有一个引用指向它的类加载器，而数组是由JVM直接生成，当获取一个数组的类加载器时，会返回它元素的类加载器，例如：

```java
// int 数组
int[] a = new int[]{1, 2, 3};
ClassLoader classLoader = a.getClass().getClassLoader();
System.out.println(classLoader);
// person对象数组
person[] b = new person[]{new person(), new person()};
ClassLoader classLoader1 = b.getClass().getClassLoader();
System.out.println(classLoader1);
System.out.println(classLoader1.getParent());
System.out.println(classLoader1.getParent().getParent());
```

上面的执行结果如下：

```java
null
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@55f96302
null
```

其中int数组返回的是`int`类型的`ClassLoader`，即`BootstrapClassLoader`；而`person`是我们自定义的对象，所以它的类加载器是`AppClassLoader`

### 双亲委派模式

双亲委派的过程是：

- 当前类加载器在通过全类名加载类时，会先交给其父级类加载器加载，如果父级类加载无法加载该类(即在其负责的路径下找不到该类)，才会交给其子类进行加载；
- 如果当前类已经加载，则直接返回该类。

双亲委派的目的是为了保证Java核心类能够安全的加载。例如我们如果重写了`Object`类，没有双亲委派模式的情况下JDK核心类中的Object类将会被覆盖，这样会导致JVM无法正常运行。

而有了双亲委派模式，我们重写的`Object`类需要到`AppClassLoader`才会加载，而JDK核心类中的`Object`类已经在`BootstrapClassLoader`中加载了，之后再加载`Object`类时会直接返回，这样就保证了JVM的正常运行。

# IDEA的操作

 IDEA版本：2022.3.1(Ultimate Edition)

## 设置JVM参数

![20230417180104](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417180104.png)

在`Edit Configuration`中，如果没有VM options的话，如上图操作添加上即可；

## 分析堆内存

**Debug过程中**：

![20230417180353](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417180353.png)

要把Debug中的`Memory`视图打开

**使用memory快照**：

[官方教程](https://www.jetbrains.com/help/idea/create-a-memory-snapshot.html)

![20230417180617](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417180617.png)

捕获的快照`.hprof`文件中可以查看对象的创建个数等，如下图：

![20230417180722](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230417180722.png)

## 垃圾回收

### 并发漏标问题(待完善)
