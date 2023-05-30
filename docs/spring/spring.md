- [常用注解](#常用注解)
- [国际化](#国际化)
- [容器](#容器)
  - [BeanFactory](#beanfactory)
  - [ApplicationContext](#applicationcontext)
    - [事件监听和发送](#事件监听和发送)
    - [常见的实现类](#常见的实现类)
- [AOP](#aop)
  - [基于XML的AOP模板](#基于xml的aop模板)
- [Spring事务](#spring事务)
  - [Spring事务的关键接口](#spring事务的关键接口)
    - [PlatformTransactionManager](#platformtransactionmanager)
    - [TransactionDefinition](#transactiondefinition)
    - [TransactionStatus](#transactionstatus)
  - [事务的传播行为](#事务的传播行为)
    - [REQUIRED(默认)](#required默认)
    - [REQUIRES\_NEW](#requires_new)
    - [NESTED](#nested)
    - [MANDATORY](#mandatory)
    - [SUPPORTS](#supports)
    - [NOT\_SUPPORTED](#not_supported)
    - [NEVER](#never)
  - [事务隔离级别](#事务隔离级别)
    - [DEFAULT](#default)
    - [READ\_UNCOMMITTED](#read_uncommitted)
    - [READ\_COMMITTED](#read_committed)
    - [REPEATABLE\_READ](#repeatable_read)
    - [SERIALIZABLE](#serializable)
  - [Transactional注解](#transactional注解)
    - [注解工作原理](#注解工作原理)
    - [属性说明](#属性说明)
    - [自调用问题](#自调用问题)

# 常用注解

- `@Autowired`: 只能通过类型匹配，多类型时优先选同名称或类型匹配最严格的bean;
- `@Qualifier`: 协助`@Autowired`匹配名称;
- `@Resource`: 可以指定名称匹配；
- `@Value`: 可以注入`String`和基本类型(`@Autowired`只能注入Bean对象)
- `@ConfigurationProperties`: 注入配置文件的字段到类的成员变量中([例子](https://bolder-macaroon-7fa.notion.site/ConfigurationProperties-436344970628419ea6440319ccd5a5dc))
- `@Transactional`

# 国际化

# 容器

## BeanFactory

`BeanFactory`主要只有一个`getBean()`方法，可以通过名称或者类型获取Bean对象

## ApplicationContext

是一个接口，相比`BeanFactory`多出的功能有：

- 国际化
- 通过通配符获取Resource资源

  ```java
  context.getResource("classpath*:*.xml");
  ```

- 整合Environment环境

  ```java
  context.getEnvironment().getProperty("os.name");
  ```

- 事件的监听与发送（帮助事件解耦）

### 事件监听和发送

该功能常用来进行模块之间的解耦，例如用户注册(`Componet1`)和短信验证(`Componet2`)两个模块

```java
@Componet
public class Component1 {
    @Autowired
    private ApplicationEventPublisher context;

    public void register() {
        log.debug("用户注册");
        // 注册成功后发布事件给监听者
        context.publishEvent(new UserRegisteredEvent(this));
    }
}
```

```java
@Componet
public class Component2 {
    // 监听事件注解
    @EventListener
    public void sendSms(UserRegisteredEvent event) {
        log.debug("发送短信");
    }
}
```

这样就完成了两个模块的解耦，其中`UserRegisteredEvent`是自定义类，继承自`ApplicationEvent`，如下：

```java
public class UserRegisteredEvent extends ApplicationEvent {
    public UserRegisteredEvent(Object source) {
        super(source);
    }
}
```

### 常见的实现类

1. `ClassPathXmlApplicationContext`
2. `FileSystemXmlApplicationContext`
3. `AnnotationConfigApplicationContext`
4. `AnnotationConfigWebApplicationContext`

# AOP

**术语:**

- 横切关注点：即模块中需要加入增强方法的位置，例如日志管理、事务处理、用户验证、数据缓存等；
- 增强(通知)：即需要增强的功能，如事务、日志等
- 切面：封装增强方法的类
- 连接点：Spring允许使用增强方法的位置，例如：方法开始前、捕获异常、方法开始后；
- 切入点：即PointCut，Spring利用切入点定位到指定的连接点

**AspectJ：**

- 本质是静态代理，将代理逻辑“织入”被代理的目标类编译得到的字节码文件（动态代理），以达到动态的效果

**切入点表达式：**

![AOPexpression](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/AOPexpression.png)

**切面优先级：**

- 相同方法存在多个切面时：采用内外嵌套的顺序，即外部的切面优先级更高
- 可以用`@Order`控制优先级，数值越小优先级越高

## 基于XML的AOP模板

```xml
<!--配置切面类的扫描包路径-->
<context:component-scan base-package="com.atguigu.aop.xml"></context:component-scan>

<aop:config>
    <!--配置切面类, ref是切面类名称（要换成驼峰命名）-->
    <aop:aspect ref="loggerAspect"> 
    <!--配置重用切入点-->
        <aop:pointcut id="pointCut" 
                   expression="execution(* com.atguigu.aop.xml.CalculatorImpl.*(..))"/>
    <!--配置前置通知、后至通知、返回通知、异常通知、环绕通知-->
        <aop:before method="beforeMethod" pointcut-ref="pointCut"></aop:before>
        <aop:after method="afterMethod" pointcut-ref="pointCut"></aop:after>
        <aop:after-returning method="afterReturningMethod" returning="result" pointcut-ref="pointCut"></aop:after-returning>
        <aop:after-throwing method="afterThrowingMethod" throwing="ex" pointcut-ref="pointCut"></aop:after-throwing>
        <aop:around method="aroundMethod" pointcut-ref="pointCut"></aop:around>
    </aop:aspect>
</aop:config>
```

# Spring事务

## Spring事务的关键接口

### PlatformTransactionManager

事务管理接口，主要包含3个方法：

- 获取事务
- 提交事务
- 回滚事务

```java
public interface PlatformTransactionManager {
    //获得事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    //提交事务
    void commit(TransactionStatus var1) throws TransactionException;
    //回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

### TransactionDefinition

事务的属性，包含：

- 隔离级别
- 传播行为
- 回滚规则
- 是否只读
- 事务超时

```java
public interface TransactionDefinition {
    // ...省略其中包含的枚举信息
    // 返回事务的传播行为，默认值为 REQUIRED。
    int getPropagationBehavior();
    //返回事务的隔离级别，默认值是 DEFAULT
    int getIsolationLevel();
    // 返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
    int getTimeout();
    // 返回是否为只读事务，默认值为 false
    // 只读事务是不对数据进行操作，仅做查询的事务
    boolean isReadOnly();

    @Nullable
    String getName();
}
```

### TransactionStatus

事务的状态，直接看源码：

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事务
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```

## 事务的传播行为

先对下面的术语解释：

- 外部方法、内部方法：

```java
// aMethod是外部方法
public void aMethod(){
  // bMethod是内部方法
    bMethod();
}
public void bMethod(){
    //...
}
```

事务的传播行为关系到不同方法之间的调用，如果内部方法出现异常，是否要同时回滚内部和外部方法的事务。

### REQUIRED(默认)

- 外部方法没有开启事务，传播行为为`REQUIRED`的内部方法为开启自己的事务
- 外部方法开启了事务，内部方法加入外部方法的事务，出现异常后同时回滚

### REQUIRES_NEW

不管外部方法是否开启事务，传播行为为`REQUIRES_NEW`的内部方法都会开启自己的事务

- 外部方法异常回滚 -> 内部方法不回滚
- 内部方法异常回滚 -> 外部方法也回滚

> 内部方法异常会被外部方法的事务管理机制捕捉到

### NESTED

和`REQUIRES_NEW`反过来:

- 外部方法异常回滚 -> 内部方法也回滚
- 内部方法异常回滚 -> 外部方法不回滚

### MANDATORY

如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

### SUPPORTS

如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

### NOT_SUPPORTED

以非事务方式运行，如果当前存在事务，则把当前事务挂起。

### NEVER

以非事务方式运行，如果当前存在事务，则抛出异常。

## 事务隔离级别

### DEFAULT

采用指定数据库的隔离级别：

- MySQL: `REPEATABLE_READ`
- Oracle: `READ_COMMITTED`

### READ_UNCOMMITTED

允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读，是最低的隔离级别

> 脏读、幻读、不可重复读的解释点[这里](../DataBase/MySQL.md#并发事务的问题)

### READ_COMMITTED

允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

### REPEATABLE_READ

对同一字段的多次读取结果都是一致的，可以阻止脏读和不可重复读，但幻读仍有可能发生。

### SERIALIZABLE

最高级别，事务都是串行执行，可以阻止脏读、不可重复读和幻读，但会影响程序性能。

## Transactional注解

### 注解工作原理

基于AOP实现，如果一个类或方法被`@Transaction`标注，Spring容器会创建一个代理类。实际调用的时候是调用`TransactionInterceptor`中的`invoke()`方法

### 属性说明

- `rollbackFor`指定回滚的异常类型，`propagation`指定事务的传播行为，例如：

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
```

- `rollbackOn`方法中指定了默认回滚的异常，默认回滚的异常有`RuntimeException`和`Error`及它们的子类，源码如下

```java
@Override
public boolean rollbackOn(Throwable ex) {
  return (ex instanceof RuntimeException || ex instanceof Error);
}
```

- 不能回滚被`try...catch`捕获的异常

### 自调用问题

因为`@Transactional`是基于AOP实现的，自调用无法调用到代理类，所以事务无法生效，解决方法：

- 利用AspectJ取代String AOP代理

> 自调用：一个类中的方法调用了该类中的另一个方法，例如：`this.method`