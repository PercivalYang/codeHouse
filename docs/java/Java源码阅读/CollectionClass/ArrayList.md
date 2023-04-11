- [内部成员](#内部成员)
  - [常量和成员属性](#常量和成员属性)
  - [内部类](#内部类)
- [方法](#方法)
  - [构造方法](#构造方法)
  - [public 方法](#public-方法)
    - [indexOf 和 lastIndexOf](#indexof-和-lastindexof)
    - [add](#add)
    - [trimToSize](#trimtosize)
    - [remove](#remove)
    - [retainAll](#retainall)
    - [iterator](#iterator)
    - [listIterator](#listiterator)
    - [subList](#sublist)
    - [removeIf](#removeif)
    - [replaceAll](#replaceall)
    - [sort](#sort)
  - [一些private方法](#一些private方法)
    - [容量检查](#容量检查)
    - [grow()](#grow)
    - [batchRemove](#batchremove)
    - [Itr](#itr)
    - [ListItr](#listitr)
    - [SubList](#sublist-1)

# 内部成员

## 常量和成员属性

**常量**：

```java
// 默认容量
int DEFAULT_CAPACITY = 10;
// 这两个目前暂时没分清楚
Object[] EMPTY_ELEMENTDATA = {};
Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
// 数组最大容量
int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

**成员属性**：

- `elementData`：存储元素的数组，用`transient`修饰，不参与序列化
- `size`：成员个数（不是数组容量）

## 内部类

# 方法

## 构造方法

有3种构造方法，分别是无参构造、指定容量的构造、指定Collection构造。

- 无参构造：使用默认容量创建一个空数组
- 指定容量：如果指定容量大于0，则创建指定容量的数组，否则抛出异常
- 指定Collection构造：先将传入的Collection对象通过[`toArray`](./Collection.md#toarray)转换成数组，
  - 首先判断数组长度是否为0，为0则返回容量为0的空数组。
  - 不为0再判断是否为`ArrayList`类，是的话直接将数组赋值给`elementData`，否则使用`Arrays.copyOf`方法复制数组。

```java
// 无参构造
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 指定容量
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException(...)
        }
}

// 指定Collection构造
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

## public 方法

### indexOf 和 lastIndexOf

两个方法都是传入`Object o`，返回`int`类型的数组下标；区别在于`indexOf`是从前往后找，`lastIndexOf`是从后往前找。而两者找的方法都是循环遍历的方式

### add

添加元素的`add`方法有两种方式：

- `add(E e)`：在数组末尾添加元素
- `add(int index, E element)`：在指定位置添加元素，采用

    ```java
    System.arraycopy(elementData, index, elementData, index + 1, size - index); 
    ```

这两种方式在添加前都要调用`ensureCapacityInternal`来检查当前`elementData`容量是否足够

### trimToSize

讲ArrayList中数组的容量修剪为当前元素的个数，即将`elementData`数组的长度设置为`size`。

```java
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
            ? EMPTY_ELEMENTDATA
            // Arrays.copyOf会重新返回一个新数组
            // 新数组复制了elementData数组前size个元素
            : Arrays.copyOf(elementData, size);
    }
}
```

### remove

删除操作都会让`modCount`加1，删除元素的`remove`方法有两种方式：

- `remove(int index)`：删除指定下标的元素，返回被删除的元素
- `remove(Object o)`：删除指定元素，如果存在多个相同元素，则删除第一个，返回布尔值代表是否删除成功

### retainAll

函数签名: `boolean retainAll(Collection<?> c)`，只保留集合中与指定集合`c`相同的元素。具体实现是调用了[`batchRemove`](#batchremove)

### iterator

返回一个[`Itr`](#itr)对象，该对象实现了`Iterator`接口，用于遍历`ArrayList`。

### listIterator

与`iterator`不同的是，此方法可以从列表的指定位置返回对应的迭代器，同时还新增了`previous`, `add`, `set`一类的方法，详情见[`ListItr`]

### subList

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}
```

类似MySQL的视图，返回一个`ArrayList`的子列表[`SubList`](#sublist-1)，子列表和主列表的非结构性修改(例如值的修改，但不包括对数组大小结构的修改)，都会互相影响到对方

### removeIf

如函数名称一样，这个函数的作用是在满足一定条件下将元素移除。它传入的参数类型是[`Predicate`](../FunctionClass/Predicate.md)，会调用这个参数的`test`方法将数组中的元素传入，然后返回bool值来判断是否要移除该元素。

除了传入的参数外，`removeIf`移除元素的过程也值得学习，在方法内会初始化一个字节集([`BitSet`](BitSet.md))。当元素要被移除时，字节集中对应元素下标的位置会被置为`false`。

这样元素移除的任务就被放到了后面执行，前面只做了**哪些元素需要被移除**的工作。而后面移除元素的过程来直接看代码吧：

```java
final boolean anyToRemove = removeCount > 0;
if (anyToRemove) {
    final int newSize = size - removeCount;
    for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
        i = removeSet.nextClearBit(i);
        elementData[j] = elementData[i];
    }
    for (int k=newSize; k < size; k++) {
        elementData[k] = null;  // Let gc do its work
    }
    this.size = newSize;
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
```

- `removeCount`是被移除的元素个数，在前面每满足一次`test`方法时就会+1。
- `removeSet`中**为`true`表示要被移除的元素，为`false`表示要保留的元素**。调用[`nextClearBit`](BitSet.md#clearbit-和-setbit)方法可以返回字节集中下个为`false`的下标，即下一个被保留的元素，这样就能跳过被移除的元素。

### replaceAll

通过调用[`Function`](../FunctionClass/Function.md)的`apply`方法，将`ArrayList`中的每个元素都传入，然后将返回值赋值给该元素。

### sort

通过`Arrays.sort`对底层的`elementData`数组进行排序

## 一些private方法

### 容量检查

在向ArrayList中添加元素前，都会调用`ensureCapacityInternal`方法检查[`elementData`](#常量和成员属性)的容量是否足够，如果不够的话会调用`grow`方法进行扩容。

- `ensureCapacityInternal`会调用`calculateCpacity`判断当数组容量为0时，返回默认容量大小(`int`值)
- 然后再调用`ensureExplicitCapacity`来判断是否调用`grow`（[`modCount`](./AbstractList.md#modcount)在该方法内会+1）

### grow()

函数签名：`void grow(int minCapacity)`，用于扩容，扩容后的容量为原来的1.5倍(假设名为`newCapacity`)。然后

- `newCapacity`会和`minCapacity`做比较，并取他们俩中的最大值;
- 再判断`newCapacity`是否大于[`MAX_ARRAY_SIZE`](#常量和成员属性)，如果大于则使用`hugeCapacity`方法计算新的容量。

> `hugeCapacity`方法的作用是：如果`minCapacity`大于`MAX_ARRAY_SIZE`，则返回`Integer.MAX_VALUE`，否则返回`MAX_ARRAY_SIZE`。

### batchRemove

函数签名：`boolean batchRemove(Collection<?> c, boolean complement)`，用于删除集合中与指定集合`c`相同的元素。

```java
try {
    for (r = 0; r < size; r++)
        if (c.contains(elementData[r]) == complement)
            elementData[w++] = elementData[r];
}
```

- 其中`c`是传入的集合参数，`elementData`是实例对象本身的数组，`r`用于`elementData`的下标；
- 当`complement`为`true`时，表示只保留`c`中与实例对象数组相同的元素；当为`false`时，只保留不同的元素；

同样为了确保`modCount`和多线程下的问题，在上述`try`用来比较元素的模块后，还有一个`finally`用来检验当前`size`是否变化的模块：

```java
finally {
    // Preserve behavioral compatibility with AbstractCollection,
    // even if c.contains() throws.
    // 为什么r会不等于size，目前猜测是在多线程场景下，导致size发生变化
    if (r != size) {
        // 如果size大于r，将多出的size-r个元素复制到下标w之后
        System.arraycopy(elementData, r,
                            elementData, w,
                            size - r);
        w += size - r;
    }
    if (w != size) {
        // clear to let GC do its work
        for (int i = w; i < size; i++)
            elementData[i] = null;
        // 因为可能存在没有保留不相同的元素
        // 因此size-w代表未保留的不同的元素个数
        // 数组每发生一次变化，modCount就需要+1
        modCount += size - w;
        size = w;
        modified = true;
    }
}
```

### Itr

`Itr`是`ArrayList`的内部类，用于实现`Iterator`接口，用于遍历`ArrayList`。

分别实现了`next`, `hasNext`, `remove`, `forEachRemaining`方法。其中`forEachRemaining`方法的输入参数类型是`Consumer`，目的是对当前光标之后的元素执行指定的操作(指定操作由`Consumer`输入参数指定)

### ListItr

1. 函数签名

```java
private class ListItr extends Itr implements ListIterator<E>
```

其中`ListIterator`详情见[这里](ListIterator.md)

2. 构造方法

仅有一个有参构造方法如下：

```java
ListItr(int index) {
    super();
    cursor = index;
}
```

可以理解为将指定的数组下标`index`值赋给光标`cursor`，这样光标就位于指定的数组下标处，可以从该下标开始遍历。

3. 新增方法

- `previous`：返回光标前一个元素，并将光标前移一位
- `set`：将光标指向的元素替换为指定的元素
- `add`：在光标指向的元素**前**插入指定的元素，并将光标后移一位（因为之前指向的元素下标加1了），同时将`lastRet`置为-1

### SubList

是一个私有内部类，签名如下：

```java
private class SubList extends AbstractList<E> implements RandomAccess
```

主要说下它的构造方法：

```java
SubList(AbstractList<E> parent,
        int offset, int fromIndex, int toIndex) {
    this.parent = parent;
    this.parentOffset = fromIndex;
    this.offset = offset + fromIndex;
    this.size = toIndex - fromIndex;
    this.modCount = ArrayList.this.modCount;
}
```

结合[`subList`](#sublist)构造`SubList`类时传入的参数来看，就能理解了
