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