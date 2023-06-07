- [Redis缓存常见问题和解决方案](#redis缓存常见问题和解决方案)
  - [缓存雪崩](#缓存雪崩)
    - [大量数据同时过期](#大量数据同时过期)
    - [Redis故障宕机](#redis故障宕机)
  - [缓存击穿](#缓存击穿)
  - [缓存穿透](#缓存穿透)
    - [布隆过滤器](#布隆过滤器)
- [常用命令](#常用命令)
  - [通用命令](#通用命令)
  - [不同数据类型命令](#不同数据类型命令)
    - [String](#string)
    - [Hash](#hash)
    - [List](#list)
    - [Set](#set)
    - [Sorted Set](#sorted-set)
- [SpringData](#springdata)
  - [RedisTemplate](#redistemplate)
- [Redis实战](#redis实战)
  - [缓存一致性](#缓存一致性)
    - [主动更新](#主动更新)
- [Redisson](#redisson)
  - [可重入锁原理](#可重入锁原理)
- [3种缓存读写策略](#3种缓存读写策略)
  - [旁路缓存模式](#旁路缓存模式)
  - [读写穿透](#读写穿透)
  - [异步缓存写入](#异步缓存写入)

# Redis缓存常见问题和解决方案

## 缓存雪崩

问题描述：

**大量缓存数据在同一时间过期（失效）或者 Redis 故障宕机**

### 大量数据同时过期

解决方案如下：

- **均匀设置过期时间**：给数据过期时间设置一个**随机数**，避免同时过期
- **互斥锁**：若发现没有缓存，发起一个构建缓存的请求（即从数据库读取再加载到Redis），给这个请求加**互斥锁**，以避免大量用户同时请求
- **双 key 策略**：主key设置过期时间，备key不设置，主key过期后直接返回备key的数据，后续再通知后台程序重构主key的数据后台更新缓存：
- **设置缓存永久有效**，即使不设置过期时间，当系统内存紧张时也会有缓存数据被淘汰；
- 方案一：后台线程负责定时更新缓存，和检测缓存是否有效，检测的时间间隔应为毫秒级；
- 方案二：业务线程发现缓存数据失效后，通过消息队列通知后台线程更新缓存，后台线程先判断缓存数据是否失效，若失效则重构缓存数据（🌟）

### Redis故障宕机

- 服务熔断或请求限流机制：（**雪崩已发生后的解决方案**）
  - 服务熔断：暂停业务应用对缓存服务的访问，直接返回错误，此时全部业务都无法正常工作
  - 请求限流：只讲少部分请求发送到数据库处理，其他在入口直接拒绝，直到Redis恢复正常且缓存预热完毕后再解除请求限流
- 构建 Redis 缓存高可靠集群：（**预防雪崩的方案**）
  - 主从节点的方式构建Redis缓存高可靠集群

## 缓存击穿

类似雪崩，不同的在于：雪崩是宕机或大量数据过期，击穿是**热点数据**的过期

解决方案：

- **互斥锁**：保证在访问缓存时只有一个线程能够访问数据库，其他线程则需要等待;
- **提前预加载**: 系统启动时，提前将数据加载到缓存；
- **缓存不过期**: 对于热点数据设置缓存永不过期；
- **逻辑过期**：如下图所示，会在`value`中存储一个`expireTime`，当缓存过期时，先判断`expireTime`是否过期，若过期则线程1会开启一个新的线程2去**异步**更新数据，而其他线程会先返回旧的数据，等线程2更新完数据后，再返回新的数据

  ![1653328663897](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/1653328663897.png)

## 缓存穿透

访问的数据既**不在缓存，也不在数据库**，解决方案：

- **非法请求的限制**：
  - 在API入口处判断请求参数是否合理
- **缓存空值或者默认值**：（针对穿透发生后）
  - 针对非法查询的数据，在缓存中设置一个空值或者默认值，这样之后的查询不会再发给数据库
- **布隆过滤器**:
  - 快速判断数据是否存在数据库中，避免通过查询数据库来判断数据是否存在；Redis失效后，会去布隆过滤器判断数据是否存在，存在则向数据库发送请求，否则返回错误值

### 布隆过滤器

由「初始值为0的位图数组」和「N个哈希函数」组成，例如「8位位图数组」和「3个哈希函数」

![](https://cdn.nlark.com/yuque/0/2023/png/22893446/1679295156004-1afa9bf4-6f00-4fee-92d6-bc2c70765e0a.png)

- 数据库写入数据x后，会把其标记在布隆过滤器，即和「3个哈希函数」计算后取模得到位图数组的下标，然后将下标处的值设置为1；
- 当查询数据是否存在数据库中，**必须要3个下标处的值都为1才表示数据存在，有一处为0都表示数据不存在**
- 布隆过滤器可能发生数据x和数据y都命中相同下标（即哈希冲突）
- 即布隆过滤器**查到存在的数据不一定存在数据库中，但查到不存在的数据一定不存在数据库中**

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

# SpringData

依赖，配合SpringBoot开发使用：

```xml
<!--redis依赖-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## RedisTemplate

`RedisTemplate`提供了对Redis的各种操作，例如`opsForValue`返回的是`ValueOperations`，提供了对String类型的操作。

`RedisTemplate`可以接受任意类型的数据，但是写入数据库前会进行序列化，默认采用JDK序列化，存在以下问题：

- 可读性差
- 占用空间大

可以采用JSON序列化，但是需要自己实现序列化器，例如：

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
```

# Redis实战

## 缓存一致性

缓存更新策略：

- 内存淘汰：内存不足时淘汰部分数据，下次查询更新缓存
- 超时剔除：设置TTL，到期后删除，下次查询更新缓存
- 主动更新：编写业务逻辑，修改数据库时更新缓存

一致性：主动更新 > 超时剔除 > 内存淘汰

### 主动更新

**读操作**

- 缓存命中则直接返回
- 未命中，查询数据库，更新缓存

**写操作**

- 先更新数据库，再删除缓存
- 确保数据库与缓存操作的原子性

> 为什么不先删缓存再更新数据库？

更新数据库的速率是较慢的，如果在删除缓存后的更新数据库过程中，有其他线程查询缓存，会导致缓存未命中，从而查询数据库，此时数据库还未更新，会导致数据不一致。

因此先更新数据库再删缓存的原因是由更新数据库速率较慢导致的。

# Redisson

## 可重入锁原理

之前通过Redis加锁是利用`setnx`判断指定`key`是否已经赋值，这样的锁是不能重入的。

> 重入是指同一个线程可以多次获取同一把锁

Redisson的可重入锁原理：

- 利用Redis的Hash结构，`key`为锁的名称，`field`为线程ID，`value`为重入次数;
- 重入时，先判断`field`是否存在，存在则重入次数加1，不存在则加锁，即将`field`设置为1;
- 解锁时则减1，同时判断`value`为0时，删除`field`。

# 3种缓存读写策略

## 旁路缓存模式

比较适合**读请求较多的场景**，它的读写策略如下

读：

先更新数据库，再删除cache

> 为什么不先删cache再更新数据库？
>
> **因为更新数据库的速率是较慢的**，如果在删除cache后的更新数据库过程中，有其他线程查询cache，会导致cache未命中，从而查询数据库，此时数据库还未更新完成，会导致数据不一致。

写：

先读cache，读不到从db读取，然后再把db读到的数据写入cache

缺陷：

1. 首次读取的数据不存在Cache中：可以提前将热点数据放入Cache中国呢
2. 写操作频繁会导致Cache频繁被删除，影响缓存命中率。解决方法分两种场景：
   1. db和cache**强一致性**: 更新db时同步更新cache，要给更新线程加锁保证线程安全
   2. db和cache**允许短暂不一致性**: 更新db同时更新cache，但不给更新线程加锁，而是给cache加一个短的expire time，这样即使数据不一致cache也会在短时间内过期。

## 读写穿透

需要缓存应用提供将数据写入db的功能，这种的读写策略如下：

读：

从cache中读取，不存在则读取db并更新cache，更新完后再返回给客户端

写：

先查cache，不存在则直接更新db。存在则更新cache，然后cache服务自己去更新db

和旁路缓存模式比较类似，但客户端不需要再去实现写入db的功能，而是由cache服务自己去实现。

## 异步缓存写入

和读写穿透类似，都是由cache服务自己更新db。

不同的是：

- 读写穿透是同步更新db和cache
- 异步缓存写入是异步批量的方式更新db

这种策略的db写性能高，适合对数据一致性要求不高的场景如浏览量、点赞量。
