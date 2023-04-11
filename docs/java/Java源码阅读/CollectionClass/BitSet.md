
# 基本概念

BitSet用于表示一组二进制位，可以对数组中对应下标的二进制位进行`set`, `clear`, `flip`等操作。同样因为是二进制数组，不同的BitSet数组之间还支持`and`, `or`, `xor`, `andNot`, `intersects`等操作。

BitSet在初始化时，所有字节默认为false，即0。

# public方法

## set

将给定的下标或下标范围内元素的字节码设定为一个bool值，有4种Overload方式如下：

- `set(int bitIndex)`：将指定下标的元素设为`true`
- `set(int bitIndex, boolean value)`：将指定下标的元素设为`value`
- `set(int fromIndex, int toIndex)`：将指定下标范围内的元素设为`true`
- `set(int fromIndex, int toIndex, boolean value)`：将指定下标范围内的元素设为`value`

## flip

将给定的下标或下标范围内元素的字节码取反，有2种Overload方式如下：

- `flip(int bitIndex)`：将指定下标的元素取反
- `flip(int fromIndex, int toIndex)`：将指定下标范围内的元素取反

## cardinality

返回BitSet中`true`元素的个数

## intersects

判断两个BitSet是否相交，即在字节集A中设置为`true`的下标和字节集B中的相同。例如：

A=[T,F,T], B=[T,F,T], C=[F,T,T]

那么A和B相交，A和C不相交。

## ClearBit 和 SetBit

`ClearBit`返回的是`false`的下标，`SetBit`返回的是`true`的下标，在BitSet中又根据返回前一个或返回后一个分为如下4种方法：

- `nextClearBit`: 返回`false`的下标，如果没有则返回`-1`
- `nextSetBit`: 返回`true`的下标，如果没有则返回`-1`
- `previousClearBit`: 略
- `previousSetBit`: 略
