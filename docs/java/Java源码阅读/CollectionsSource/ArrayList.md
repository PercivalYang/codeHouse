- [内部成员](#内部成员)
  - [常量和成员属性](#常量和成员属性)
  - [内部类](#内部类)
- [方法](#方法)
  - [构造方法](#构造方法)
  - [public 方法](#public-方法)
    - [indexOf 和 lastIndexOf](#indexof-和-lastindexof)
  - [一些private方法](#一些private方法)
    - [容量检查](#容量检查)
    - [grow()](#grow)
  - [工具类方法](#工具类方法)
    - [trimToSize](#trimtosize)
    - [ensureCapacity](#ensurecapacity)

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

### remove

删除操作都会让`modCount`加1，删除元素的`remove`方法有两种方式：

- `remove(int index)`：删除指定下标的元素，返回被删除的元素
- `remove(Object o)`：删除指定元素，如果存在多个相同元素，则删除第一个，返回布尔值代表是否删除成功

### retainAll

函数签名: `boolean retainAll(Collection<?> c)`，只保留集合中与指定集合`c`相同的元素。

## 一些private方法

### 容量检查

在向ArrayList中添加元素前，都会调用`ensureCapacityInternal`方法检查[`elementData`](#常量和成员属性)的容量是否足够，如果不够的话会调用`grow`方法进行扩容。

- `ensureCapacityInternal`会调用`calculateCpacity`判断当数组容量为0时，返回默认容量大小(`int`值)
- 然后在调用`ensureExplicitCapacity`来判断是否调用`grow`（[`modCount`](./AbstractList.md#modcount)在该方法内会+1）

### grow()

函数签名：`void grow(int minCapacity)`，用于扩容，扩容后的容量为原来的1.5倍(假设名为`newCapacity`)。然后

- `newCapacity`会和`minCapacity`做比较，并取他们俩中的最大值;
- 再判断`newCapacity`是否大于[`MAX_ARRAY_SIZE`](#常量和成员属性)，如果大于则使用`hugeCapacity`方法计算新的容量。

> `hugeCapacity`方法的作用是：如果`minCapacity`大于`MAX_ARRAY_SIZE`，则返回`Integer.MAX_VALUE`，否则返回`MAX_ARRAY_SIZE`。

## 工具类方法

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

### ensureCapacity
