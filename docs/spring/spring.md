- [国际化](#国际化)
- [容器](#容器)
  - [BeanFactory](#beanfactory)
  - [ApplicationContext](#applicationcontext)
    - [常见的实现类](#常见的实现类)
- [AOP](#aop)
  - [基于XML的AOP模板](#基于xml的aop模板)

## 常用注解

- `@Autowired`: 只能通过类型匹配，多类型时优先选同名称或类型匹配最严格的bean;
- `@Qualifier`: 协助`@Autowired`匹配名称;
- `@Resource`: 可以指定名称匹配；
- `@Value`: 可以注入`String`和基本类型(`@Autowired`只能注入Bean对象)
- `@ConfigurationProperties`: 注入配置文件的字段到类的成员变量中([例子](https://bolder-macaroon-7fa.notion.site/ConfigurationProperties-436344970628419ea6440319ccd5a5dc))

# 国际化



# 容器

## BeanFactory

`BeanFactory`主要只有一个`getBean()`方法，可以通过名称或者类型获取Bean对象

## ApplicationContext

是一个接口，具备的功能有：

- 国际化
- 通过通配符获取Resource资源
- 整合Environment环境
- 事件的监听与发送（帮助事件解耦）

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
