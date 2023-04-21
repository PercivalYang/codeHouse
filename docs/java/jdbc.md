- [基本概念](#基本概念)
  - [提供给厂商的核心接口](#提供给厂商的核心接口)
- [非查询用法](#非查询用法)
  - [主键回显](#主键回显)
  - [批量插入](#批量插入)
- [查询用法](#查询用法)
- [事务](#事务)
  - [事务特性](#事务特性)
  - [业务层使用事务](#业务层使用事务)
- [数据库连接池](#数据库连接池)
  - [Druid](#druid)

# 基本概念

jdbc: Java Database Connectivity，用于Java连接数据库的技术，由java语言规范和各个厂商的驱动jar包组成。

SUN公司统一制定了Java连接数据库的规范(接口)，而我们调用厂商的数据驱动jar包，其实就是调用厂商对Java连接数据库接口的实现。

## 提供给厂商的核心接口

Java连接数据库的语言规范都存放在`java.sql`这个包中，其中常用到的核心接口和类有：

- `DriverMannager`:
  - 用来注册数据库厂商的驱动jar包，例如注册mysql的驱动：

    ```java
    import com.mysql.cj.jdbc.Driver;
    import java.sql.DriverMannager;
    DriverManager.registerDriver(new Driver());
    ```

  - 注册完后可以获取指定数据库的信息，例如获取atguigu数据库的信息

    ```java
    import java.sql.Connection;
    Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/atguigu", "root", "root");
    ```

- `Connection`:
  - 可以获取`statement`、`preparedstatement`、`callablestatement`等对象，用于执行sql语句

- `Statement` | `PreparedStatement` | `CallableStatement`:

  - `Statement`用于执行静态sql语句，例如：

    ```java
    Statement statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery("select * from emp");
    ```

> `Statement`存在注入攻击和静态sql语句的缺点，推荐使用`PreparedStatement`

- `PreparedStatement`用于执行动态sql语句，例如：

    ```java
    PreparedStatement preparedStatement = connection.prepareStatement("select * from emp where empno = ?");
    // 1代表第一个占位符，7369代表占位符的值
    preparedStatement.setInt(1, 7369);
    ResultSet resultSet = preparedStatement.executeQuery();
    ```

- `CallableStatement`用于执行存储过程，例如：

    ```java
    CallableStatement callableStatement = connection.prepareCall("{call getEmpByEmpno(?, ?)}");
    callableStatement.setInt(1, 7369);
    // 表示第2个占位符是输出参数，同时类型是可变字符串
    callableStatement.registerOutParameter(2, Types.VARCHAR);
    // 返回是否有结果集
    boolean ex = callableStatement.execute();
    String ename = callableStatement.getString(2);
    ```

- `Result`:
  - 用于存储查询结果，例如常用的`ResultSet`，其思想就是利用SQL中的光标，最开始时位于表格第一行，调用`.next()`方法会返回bool值判断是否还有下一行，如果有则光标移动到下一行，然后可以通过`.getString()`等方法获取当前行的数据。

# 非查询用法

DML语句例如插入、删除等操作，返回的结果通常是个int值，代表DML语句影响了多少行记录。jdbc中调用DML语句的方法是：

```java
int rows = preparedStatement.executeUpdate();
```

## 主键回显

在执行DML语句时，因为自增长的主键id是由MySQL维护的，要通过jdbc获取到id值则需要主键回显功能，例如在statement中开启主键回显：

```java
connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
```

## 批量插入

在连续插入大量数据时，可以使用批量插入的方式，例如：

```java
for (int i = 0; i < 10000; i++) {
    // 设置sql语句中占位符的值
    statement.setObject(1,"ergouzi"+i);
    statement.setObject(2,"lvdandan");
    statement.setObject(3,"驴蛋蛋"+i);
    // 将本次设置的值添加到批处理中
    statement.addBatch();
}
// 执行批处理
statement.executeBatch();
```

# 查询用法

与DML语句执行结果不同，DQL语句执行结果通常是返回一个`结果集`，例如`ResultSet`，我们可以通过`ResultSet`的光标机制获取每一行查询结果。

同样我们还可以获取表格的元信息，例如`ResultSet`自带有`.getMetaData()`方法来返回`ResultSetMetaData`对象，该对象包括：

- `getColumnLabel(int index)`：返回index列的列名；
- `getColumnCount()`：返回字段数

# 事务

## 事务特性

1. 原子性（Atomicity）原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

2. 一致性（Consistency）事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

3. 隔离性（Isolation）并发执行的各个事务之间不能互相干扰。

4. 持久性（Durability）持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的

## 业务层使用事务

每个SQL语句都是一个事务，当开启自动提交的时候，执行成功自动提交事务，否则回滚事务。

在DAO层中的方法通常都是一个事务，即一个SQL语句，例如方法`selectById(long id)`的SQL语句为：

```sql
SELECT * FROM t_user WHERE id = ?
```

但是在Service层中的方法通常是多个事务，例如转账方法`transfer(long fromId, long toId, double money)`，该方法存在多次`UPDATE`方法，为保证数据的`一致性`，需要将其放在一个事务中。例如

```java
//利用try代码块,调用dao
try {
    //关闭事务自动提交
    connection.setAutoCommit(false);
    BankDao bankDao = new BankDao();
    //调用加钱 和 减钱
    bankDao.addMoney(addAccount,money,connection);
    System.out.println("--------------");
    bankDao.subMoney(subAccount,money,connection);
    flag = 1;
    //不报错,提交事务
    connection.commit();
}catch (Exception e){
    //报错回滚事务
    connection.rollback();
    throw e;
}finally {
    connection.close();
}
```

# 数据库连接池

之前的程序中连接在每次用完后都被直接关掉，这样会导致连接的频繁创建和销毁，影响性能。同样如果不管理连接数，大量的连接建立并访问数据库会给数据库造成很大的压力。

JDBC中提供了`DataSource`接口来管理数据库连接，第三方厂商通过实现该接口来提供数据库连接池，例如`Druid`

## Druid

Druid是阿里巴巴开源的数据库连接池，其特点是性能好、扩展性好、并发性能好，是目前比较流行的数据库连接池。

通常会创建一个配置文件`druid.properties`，用来记录创建连接所需要的信息，例如：

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test
username=root
password=root
```

配置好后就可以创建Druid的连接池，例如：

```java
Properties properties = new Properties();
// 通过类加载的方式获取输入流
// 类加载请参考JVM类加载机制
InputStream druidProperties = ClassLoader.getSystemResourceAsStream("druid.properties");
properties.load(druidProperties);
// 通过工厂模式创建DataSource
DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
```