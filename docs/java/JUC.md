## 线程安全类

常见的线程安全类有：

- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable
- JUC包下的类

# Monitor(重量级锁)

## 对象头

Java对象的内存布局见[这里](./JVM.md#对象的内存布局)，其中Mark Word的结构如下图所示：

![20230606204904](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230606204904.png)

State中的意思分别是：

- Normal: 基本对象类型
- Biased: 偏向锁
- Lightweight Locked: 轻量级锁
- Heavyweight Locked: 重量级锁
- Marked for GC: GC标记

## Monitor概念

每个Java对象都可以关联一个Monitor对象，通过`synchronized`(重量级锁)给对象上锁后，对象头中的Mark Word会被修改为指向Monitor对象的指针，即如上图的(Heavyweight Locked)

![20230606205256](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230606205256.png)

Monitor的结构如上图所示，其中：

- `Owner`标识目前拥有该Monitor对象的线程，即持锁线程
- `EntryList`标识等待获取该Monitor对象的线程队列
- `WaitSet`标识等待该Monitor对象的线程队列

# synchronized

## 轻量级锁

轻量级锁的语法也是`synchronized`，但是轻量级锁和重量级锁不同的是：

- 轻量级锁通过CAS来获取锁(具体步骤见后文)，重量级锁通过Monitor对象来获取锁

详细分析如下：

1. 线程在通过`synchronized`获取锁时，会先在当前线程的栈帧中创建一个`Lock Record`对象，该对象中包含了锁对象的引用和锁标志位
2. `Lock Record`中的对象引用指向被加锁的对象，同时通过CAS将锁对象的Mark Word修改为指向`Lock Record`对象的指针，同时将锁标志位修改为`01`，表示轻量级锁

![20230606213537](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230606213537.png)

3. CAS失败的话，有两种情况：

   - 其他线程竞争，会进行锁膨胀
   - 线程本身想要重入锁

4. 对于线程自身重入，线程会再创建`Lock Record`对象，如下图所示，当退出`synchronized`块时如果取值为null表示有重入，如果不为null则通过CAS将对象的hashcode, age等属性归还

![20230606214026](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230606214026.png)

## 锁膨胀

当轻量级锁发生竞争时，会膨胀为重量级锁:

- 会为对象申请`Monitor`锁，同时擦除对象头Mark Word中指向轻量级锁的指针，改为指向重量级锁，同时将标志位改为10；
- 将竞争的线程加入`EntryList`中
- 持锁线程退出同步块后，通过指针进入`Monitor`对象，将`Owner`设置为null，并唤醒`EntryList`中BLOCKED线程

## 自旋优化

自旋优化的是减少上下文切换的次数，适合多核CPU系统。
