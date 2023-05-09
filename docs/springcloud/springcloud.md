- [Redis](#redis)
  - [缓存](#缓存)
  - [缓存雪崩](#缓存雪崩)
    - [大量数据同时过期](#大量数据同时过期)
    - [Redis故障宕机](#redis故障宕机)
  - [缓存击穿](#缓存击穿)
  - [缓存穿透](#缓存穿透)
    - [布隆过滤器](#布隆过滤器)
  - [Spring Cache](#spring-cache)
    - [注解使用](#注解使用)
- [MQ](#mq)
  - [异步](#异步)
  - [RabbitMQ](#rabbitmq)
  - [SpringAMQP](#springamqp)
    - [AMQP中不同的消息队列](#amqp中不同的消息队列)


# Redis

## 缓存

**缓存预热**：业务刚上线时，提前将缓存数据构建，而不是等用户访问触发后才构建

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

## Spring Cache

### 注解使用

![](https://cdn.nlark.com/yuque/0/2023/png/22893446/1678438465113-9500839b-27bc-4a64-b7eb-872c953af0fa.png)

- `@EnableCaching`：开启SpringCache缓存注解功能，在Application文件上添加
- `@Cacheable`：通常放在表现层的查询方法上
- `@CachePut`：通常在表现层的`save`或者`update`方法上，在每次更新数据库的值后存入缓存
- `@CacheEvict`：通常放在表现层的`delete`方法上，将数据库中的要删除的数据也从缓存中移除

# MQ

MessageQueue：消息队列，事件驱动架构中的Broker。常用消息队列：RabbitMQ、ActiveMQ、RocketMQ、Kafka

## 异步

**优点：**

- 降低耦合度
- 提升吞吐量
- 隔离模块故障 
- 流量削峰

**缺点：**

- 依赖于Broker的可靠性、安全性、吞吐能力
- 对于复杂架构，没有明显流程线，不好追踪管理（调用链不清晰）

## RabbitMQ

RabbitMQ的结构和概念：

- channel：操作MQ的工具
- exchange：消息路由
- queue：缓存消息
- virtual host：虚拟主机

![RMQ](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/RMQ.png)

## SpringAMQP

AMQP: Advanced Message Queuing Protocol，SpringAMQP是官方对RabbitMQ实现的封装类，方便使用者调用更快的发送消息

主要通过`RabbitTemplate`来进行发消息、收消息等操作，如右所示

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAmqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSendMessage2SimpleQueue() {
        String queueName = "simple.queue";
        String message = "hello, spring amqp!";
        rabbitTemplate.convertAndSend(queueName, message);
    }
```

### AMQP中不同的消息队列

**Work Queue**

该队列只包括：Publisher, Queue, Consumer三种角色，一个Queue可以对应一个及以上的Consumer

消息(事件)到达Queue时，Consumer会有prefetch(预取)的操作，即在手上消息未处理完时先将Queue中的消息取过来。但有的Consumer的处理效率低下的话，预取的消息则会造成阻塞，影响整个系统的性能。

处理方案是在配置文件中配置prefetch的数目，如下：

```java
spring:
  rabbitmq:
    host: 192.168.150.101 # rabbitMQ的ip地址
    port: 5672 # 端口
    username: itcast
    password: 123321
    virtual-host: / # 该用户可以访问的虚拟机路径
    listener:
      simple:
        prefetch: 1 # 表示此时prefetch的数量为1
```

![WorkQueue](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/WorkQueue.png)

**FanoutExchange**

在WorkQueue的基础上加入了Exchange(交换机)角色，主要功能是转发消息；

FanoutExchange中的交换机采用**广播机制**来转发消息

即消息发给交换机后，交换机会将该消息发给所有和自己绑定的Queue

![FanoutExchange](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/FanoutExchange.png)

**DirectExchange**

队列和交换机之间可以绑定多个`bindingKey`（如图所示），交换机在发送消息时需要指定`RoutingKey`

程式中可以通过注解`@RabbitListener`声明Exchange、Queue之间的关系，如下：(其中`type`用来指定交换机类型)

```java
@RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue1"),
            exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
	          key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者接收到direct.queue1的消息：【" + msg + "】");
}
```

![DirectExchange](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/DirectExchange.png)

**TopicExchange**

相比DirectExchange的优势是可以使用通配符，因为TopicExchange的`bindingKey`必须是多个单词的列表，且中间用`.`分割

例如右图所示，以程式为例辅助说明：

例如`publisher`要发送routingKey为china.weather的消息

```java
rabbitTemplate.convertAndSend(exchangeName, "china.weather", message);
```

图中可看出Queue1和Queue3的通配符都适配chinan.weather，因此消息会被交换机转发到这两个队列中。

而在Listener处的注解绑定和DirectExchange相似，只对`key`和`type`修改如下：

```java
	key = "china.#" 以及 type = ExchangeTypes.TOPIC
```

![topicExchange](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/topicExchange.png)