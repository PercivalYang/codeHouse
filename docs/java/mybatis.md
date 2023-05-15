-集合中的元素代表的字段名 [配置文件](#配置文件)
- [配置文件](#配置文件)
- [Mapper xml文件的使用](#mapper-xml文件的使用)
  - [xml文件获取参数值的两种方式 (重点)](#xml文件获取参数值的两种方式-重点)
  - [利用自增主键返回值补充实体类的属性值](#利用自增主键返回值补充实体类的属性值)
  - [自定义映射](#自定义映射)
    - [resultMap](#resultmap)
    - [association](#association)
- [其他](#其他)
  - [分页功能](#分页功能)
  - [常用注解说明](#常用注解说明)
- [动态SQL](#动态sql)
  - [if](#if)
  - [where](#where)
  - [trim](#trim)
  - [choose, when, otherwise](#choose-when-otherwise)
  - [foreach](#foreach)
  - [SQL片段](#sql片段)
- [MyBatis缓存](#mybatis缓存)
  - [一级缓存](#一级缓存)
  - [二级缓存](#二级缓存)
  - [查询缓存的顺序](#查询缓存的顺序)
- [逆向工程(Generator)](#逆向工程generator)
  - [配置文件参考](#配置文件参考)

# 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--引入jdbc的properties配置文件-->
    <properties resource="jdbc.properties"/>

    <settings>
        <!--将表中字段的下划线自动转换为驼峰-->
        <!-- 例如user_name -> userName -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启懒加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>

    <!--给包路径的POJO类设置别名，方便之后调用-->
    <typeAliases>
        <!--type: 包路径-->
        <!--alias: 别名，如果不指定默认为类名且不区分大小写-->
        <typeAlias type="com.mb.entity.User" alias="User"/>
        <!--以包为单位，设置路径下所有类型都为默认的别名-->
        <package name="com.mb.entity"/>
    </typeAliases>

    <!--设置连接数据库的环境-->
    <!--default: 默认使用的环境id-->
    <environments default="development">

        <environment id="development">
            <!--配置事务管理-->
            <!--JDBC: 表示当前环境下的事务都需要手动管理-->
            <!--MANAGED: 当前环境的事务管理由容器管理-->
            <transactionManager type="JDBC"/>
            <!--设置数据源-->
            <!--POOLED: 使用数据库的连接池技术，会缓存数据库连接对象，使用时从缓存中取出，使用完毕后不会关闭连接，而是将连接归还到连接池中-->
            <!--UNPOOLED: 不使用数据库的连接池技术，每次使用完毕后都会关闭连接，再次使用需要重新创建链接-->
            <!--JNDI: 使用JNDI技术获取数据源，实现数据源的管理，不常用-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>

    </environments>

    <!--引入映射文件-->
    <mappers>

        <!-- 可以指定匹配路径下的xml文件 -->
        <mapper resource="mapper/UserMapper.xml"/>
        <!-- 也可以指定包路径，会自动匹配包中所有xml文件 -->
        <!-- 但上面两种不能存在重复的xml文件，否则会抛异常 -->
        <package name="com.mb.mapper"/>

    </mappers>
</configuration>
```

# Mapper xml文件的使用

## xml文件获取参数值的两种方式 (重点)

区别：`#{}`和`${}`

- `#{}`: 会将参数值转换为字符串，然后拼接到sql语句中，可以防止sql注入，`#{}`的本质是占位符
- `${}`: 会将参数值直接拼接到sql语句中，不会进行转换，可能会导致sql注入，`${}`的本质是字符串拼接

同时还需要注意`#{}`由于是通过占位符进行赋值，在面对字符串和日期等参数时可以省略添加单引号；而`${}`由于是字符串拼接，所以在面对字符串和日期等参数时**需要手动添加单引号**

## 利用自增主键返回值补充实体类的属性值

当我们像数据库添加对象时，通常不会指定自增主键的值(例如常用的自增主键`id`字段值)，在添加到数据库前实例对象的该`id`属性值为`null`。我们可以利用`useGeneratedKeys`和`keyProperty`属性来实现自动补充该属性值。

```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    insert into t_user
    values (null,  #{account}, #{password}, #{nickname})
</insert>
```

对应Java程序中向数据库添加对象的调用过程：

```java
User user = new User("zhangsan", "123", "张三");
mapper.insertUser(user);
// 在insertUser成功调用之后, user中的id属性值就被set为了返回的自增主键id的值
```

## 自定义映射

### resultMap

当Java中实体类的属性值名称和数据库中的字段名称不一致时，可以通过`resultMap`来自定义映射关系。
例如实体类中属性值名称为`name`需要对应数据库中的`nickname`字段

```xml
<resultMap id="userMap" type="User">
  <!--
    id: 主键映射关系
    result: 普通字段映射关系
    标签属性：
      column: 数据库中的字段名称
      property: 实体类中的属性名称
  -->
  <id column="id" property="id">
  <result column="nickname" property="name"/>
</resultMap>
```

定义好之后，便可以在其他DQL语句中使用该`resultMap`，例如：

```xml
<select id="selectUsers" resultMap="userMap">
    select * from t_user
</select>
```

### association

当涉及到多表查询时，返回的数据可能包含不止一种实体对象类型。例如员工表和部门表联合查询，返回的数据可能包含员工对象和部门对象。此时可以通过`association`标签来定义多表查询的映射关系。

```xml
<resultMap id="empMap" type="Emp">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
    <!--
      association: 定义多表查询的映射关系
      标签属性：
        property: 实体类中的属性名称
        javaType: 实体类的类型
    -->
    <association property="dept" javaType="Dept">
        <id column="dept_id" property="id"/>
        <result column="dept_name" property="name"/>
    </association>
</resultMap>
```

从上面可以看出最终返回的还是员工(`Emp`)类型的实体对象，由于`Emp`对象包含了`Dept`对象，所以多表查询包含了`Dept`对象的数据时，需要通过`association`标签来定义`Dept`对象的映射关系。

# 其他

## 分页功能

- 设置拦截器：

```java
@Configuration
public class MpCongfig {
 @Bean
 public MybatisPlusInterceptor pageInterceptor(){
  MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 加入分页拦截器
  interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
  return interceptor;
 }
}
```

- 业务层实现分页起

```java
public R<Page> page(int page,int pageSize){
    //分页构造器
    Page<Category> pageInfo = new Page<>(page,pageSize);
    //条件构造器
    LambdaQueryWrapper<Category> queryWrapper = new LambdaQueryWrapper<>();
    //添加排序条件，根据sort进行排序
    queryWrapper.orderByAsc(Category::getSort);

    //分页查询
    categoryService.page(pageInfo,queryWrapper);
    return R.success(pageInfo);
}
```

## 常用注解说明

- `@TableId`: 指定主键，其中`type`属性可以指定生成策略
- `@TableField`: 建立数据库字段名和实体类名的映射
  - `exist`属性可以指定是否为数据库字段
  - `select`属性可以指定是否为查询字段
  - `update`属性可以指定是否为更新字段
  - `insert`属性可以指定是否为插入字段
  - `fill`属性可以指定是否为自动填充字段
- `@Version`: 乐观锁注解，用于指定乐观锁的字段
- `@TableLogic`: 逻辑删除注解，用于指定逻辑删除的字段
  - `value`属性可以指定逻辑删除的值
  - `delval`属性可以指定逻辑未删除的值

# 动态SQL

## if

```xml
<select id="selectUsers" resultMap="userMap">
    select * from t_user
    <where>
        <if test="name != null">
            and nickname like concat('%', #{name}, '%')
        </if>
        <if test="age != null">
            and age = #{age}
        </if>
    </where>
```

当`if`中`test`语句为true时，才会执行`if`标签中的内容

## where

```xml
<select id="selectUsers" resultMap="userMap">
    select * from t_user
    <where>
        <if test="name != null">
            and nickname like concat('%', #{name}, '%')
        </if>
        <if test="age != null">
            and age = #{age}
        </if>
    </where>
```

`where`一般和`if`一起使用，`where`标签的目的是自动去除第一个条件多余的`and`或者`or`关键词，以此保证SQL语法的正确性。例如上式，第一个`if`标签成立时会自动去除`nickname`前的`and`

## trim

```xml
<select id="selectUsers" resultMap="userMap">
    select * from t_user
    <trim prefix="where" prefixOverrides="and | or">
        <if test="name != null">
            and nickname like concat('%', #{name}, '%')
        </if>
        <if test="age != null">
            and age = #{age}
        </if>
    </trim>
```

`trim`标签可以用来自定义SQL语句的前缀和后缀，有四个常用属性：

- `prefix`: 前缀
- `prefixOverrides`: 前缀覆盖，用于去除多余的前缀
- `suffix`: 后缀
- `suffixOverrides`: 后缀覆盖，用于去除多余的后缀

例如上式，`prefix`属性指定了前缀为`where`，`prefixOverrides`属性指定了去除多余的`and`和`or`关键词。

## choose, when, otherwise

```xml
<select id="selectUsers" resultMap="userMap">
    select * from t_user
    <where>
        <choose>
            <when test="name != null">
                and nickname like concat('%', #{name}, '%')
            </when>
            <when test="age != null">
                and age = #{age}
            </when>
            <otherwise>
                and id = 1
            </otherwise>
        </choose>
    </where>
```

`choose`标签相当于Java中的`switch`语句，`when`标签相当于`case`语句，`otherwise`标签相当于`default`语句。

## foreach

```xml
<!-- List<User> selectUsers(@Param("ids") Long[] ids); -->
<select id="selectUsers" resultMap="userMap">
    select * from t_user
    <where>
        <foreach collection="ids" item="id" open="and id in (" close=")" separator=",">
            #{id}
        </foreach>
    </where>
```

根据第一行注释可知`ids`是一个Long型数组，

`foreach`标签用于遍历集合，其中的属性值分别代表：

- `collection`: 集合对象
- `item`: 指定集合中对象的别名
- `open`: 语句的前缀符号
- `close`: 语句的后缀符号
- `separator`: 分隔符，如上面例子用于分割多个`id`，保证SQL语法正确

## SQL片段

```xml
<sql id="selectColumns">
    id, nickname, age
</sql>
select <include refid="selectColumns"/> from t_user
```

其中`sql`标签用于定义SQL片段，`id`属性用于指定SQL片段的名称。`include`标签用于引用SQL片段，`refid`属性用于指定引用的SQL片段名称。

# MyBatis缓存

## 一级缓存

一级缓存是SqlSession级别的缓存，是默认开启的。

> 创建一个SqlSession的流程: SqlSessionFactoryBuilder.build(config) -> SqlSessionFactory.openSession() -> DefaultSqlSession

当遇到以下情况时，缓存会失效：

- 两次查询之间执行了增删改操作
- 两次查询之间手动清除了缓存，即调用了`SqlSession`的`clearCache()`方法

## 二级缓存

二级缓存是SqlSessionFactory级别的缓存，是需要手动开启的。开启步骤和配置这里不赘述

## 查询缓存的顺序

1. 先查二级缓存
2. 如果二级缓存没有，再查一级缓存
3. 如果一级缓存没有，再查数据库

# 逆向工程(Generator)

什么是逆向工程？

正向工程指先创建Java实体类，再创建数据库表。而逆向工程则反之，先创建数据库表，由框架根据数据库表生成：

- 实体类
- Mapper接口
- Mapper映射文件

## 配置文件参考

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
        targetRuntime 代表逆向工程的版本，有如下选择：
            1. MyBatis3：生成带有条件查询的CRUD
            2. MyBatis3Simple：生成基本的CRUD
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/atguigu"
                        userId="root"
                        password="root">
        </jdbcConnection>
        <!--javaBean生成策略-->
        <javaModelGenerator targetPackage="com.mb.bean" targetProject=".\src\main\java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--SQL映射文件的生成策略-->
        <sqlMapGenerator targetPackage="com.mb.mapper" targetProject=".\src\main\resources">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>
        <!--Mapper接口生成策略-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.mb.mapper" targetProject=".\src\main\java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>
        <!--指定哪些表要进行逆向工程-->
        <table tableName="t_user" domainObjectName="User"/>
        <table tableName="t_bank" domainObjectName="Bank"/>
    </context>
</generatorConfiguration>
```