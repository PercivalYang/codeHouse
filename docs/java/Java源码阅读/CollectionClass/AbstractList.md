
## modCount

函数签名：`protected transient int modCount = 0`

> 为什么要有这个成员变量？

因为`modCount`实现了fail-fast机制，这个机制是预防在多线程情况下遍历集合时，有其他线程对集合进行了修改，还产生不可预知的错误。

Fast-Fail机制在读取集合前会记录一个`expectedModCount`，在遍历完后会和当前对象的成员属性`modCount`进行比较，若不同则抛出`ConcurrentModificationException`异常

> 哪些操作会引发`modCount++`

通常是引发容器结构和元素发生变化的操作，例如`remove`, `clear`, `add`等

- 发现ArrayList中的`add`方法没有直接调用`modCount++`，而是通过检查容量大小`ensureCapacityInternal`来间接调用`modCount++`