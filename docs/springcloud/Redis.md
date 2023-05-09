- [常用命令](#常用命令)
  - [通用命令](#通用命令)
  - [不同数据类型命令](#不同数据类型命令)
    - [String](#string)
    - [Hash](#hash)
    - [List](#list)
    - [Set](#set)
    - [Sorted Set](#sorted-set)

# 常用命令

## 通用命令

- `KEYS`：查看符合模板的所有key
- `DEL`：删除一个指定的key
- `EXISTS`：判断key是否存在
- `EXPIRE`：给一个key设置有效期，有效期到期时该key会被自动删除
- `TTL`：查看一个KEY的剩余有效期

## 不同数据类型命令

### String

- `SET`和`GET`
- `MSET`和`MGET`：批量
- `INCR`：整型自增1
- `INCRBY`:整型自增指定步长
- `INCRBYFLOAT`：浮点型自增指定步长

**Key的结构**

当需要对Key分层时，分隔符为`:`

### Hash

- `HSET`和`HGET`
- `HMSET`和`HMGET`：批量
- `HGETALL`：获取所有的key-value
- `HKEYS`：获取所有的key

在前面加个`H`

### List

类似Java的LinkedList，并且是双向队列

- `LPUSH`和`RPUSH`：分别从左和右插入一个元素
- `LPOP`和`RPOP`：分别从左和右弹出一个元素
- `BLPOP`和`BRPOP`：阻塞弹出

### Set

支持交并集的查找

- `SADD`和`SREM`：添加和删除
- `SMEMBERS`和`SMEMBER`：获取所有元素和判断是否存在
- `SCARD`：获取元素个数
- `SINTER`和`SUNION`：交集和并集

### Sorted Set

比`Set`多了一个权重，可以用来做排行榜。命令类似，将`Set`命令首字母`S`换成`Z`即可。