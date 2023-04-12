- [SQL基本概念](#sql基本概念)
  - [SQL语言分类](#sql语言分类)
  - [表的连接](#表的连接)
  - [WHERE 和 HAVING的区别](#where-和-having的区别)
  - [子查询](#子查询)
    - [单行子查询](#单行子查询)
    - [多行子查询](#多行子查询)
    - [结合`EXISTS`使用](#结合exists使用)
  - [约束性](#约束性)
    - [非空约束](#非空约束)
    - [唯一约束](#唯一约束)
    - [主键约束](#主键约束)
    - [外键约束](#外键约束)
  - [约束等级](#约束等级)
  - [视图](#视图)
    - [视图的创建](#视图的创建)
  - [过程](#过程)
    - [过程的创建](#过程的创建)
    - [过程的调用](#过程的调用)
    - [过程的优缺点](#过程的优缺点)
  - [函数](#函数)
    - [函数的创建](#函数的创建)
    - [函数的调用](#函数的调用)
    - [如何查看数据库的自定义过程和函数](#如何查看数据库的自定义过程和函数)
  - [变量](#变量)
    - [查看系统变量](#查看系统变量)
    - [用户变量](#用户变量)
    - [定义条件与处理程序](#定义条件与处理程序)
  - [流程控制](#流程控制)
    - [条件判断](#条件判断)
    - [循环语句](#循环语句)
    - [跳转语句](#跳转语句)
  - [游标](#游标)
- [InnoDB](#innodb)
  - [数据页](#数据页)
- [事务](#事务)
  - [并发事务的问题](#并发事务的问题)
  - [特性](#特性)
  - [隔离级别](#隔离级别)
  - [Read View](#read-view)
- [MySQL8新特性](#mysql8新特性)
  - [DDL 原子性](#ddl-原子性)
  - [计算列](#计算列)
  - [自增列持久化](#自增列持久化)
  - [全局变量持久化](#全局变量持久化)
- [面试问题](#面试问题)

# SQL基本概念

## SQL语言分类

- DDL（Data Definition Language）：数据定义语言，用于定义数据库对象，如表、索引等，常见包括：
  - `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `CREATE INDEX`, `DROP INDEX`
- DML（Data Manipulation Language）：数据操作语言，用于对数据库中的表进行增删改查操作,常见SQL语句包括
  - `INSERT`, `UPDATE`, `DELETE`, `SELECT`
- DCL（Data Control Language）：数据控制语言，用于定义数据库的访问权限和安全级别，常见包括：
  - `GRANT`, `REVOKE`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `LOCK TABLE`

## 表的连接

- 笛卡尔积（CARTESIAN PRODUCT）：两个表的所有行的组合，例如

    ```sql
    SELECT * FROM A, B;
    ```

- 内连接（[INNER] JOIN）：返回两个表中**符合连接条件**的记录，例如

    ```sql
    SELECT * FROM A INNER JOIN B ON A.id = B.id;
    ```

- 左连接（LEFT [OUTER] JOIN）：返回左表中**符合连接条件**的记录，即使右表中没有符合条件的记录，也会返回左表中的记录，例如

    ```sql
    SELECT * FROM A LEFT JOIN B ON A.id = B.id;
    ```

> 右连接（RIGHT [OUTER] JOIN）：和左连接类似，只是方向反了过来，即使左表中没有符合条件的记录，也会返回右表的内容

- 全连接（FULL JOIN）：返回左表和右表中**符合连接条件**的记录，即使左表或右表中没有符合条件的记录，也会返回左表或右表中的记录，例如

    ```sql
    SELECT * FROM A FULL JOIN B ON A.id = B.id;
    ```

- 自连接（SELF JOIN）：表连接自身，例如

    ```sql
    SELECT * FROM A AS a1, A AS a2 WHERE a1.id = a2.id;
    ```

- 多表连接：多个表之间的连接，例如

    ```sql
    SELECT * FROM A INNER JOIN B ON A.id = B.id INNER JOIN C ON B.id = C.id;
    ```

> 内连接和左右连接不同的是，内连接被连接的表都满足条件才返回，而左右连接只需要左/右表满足条件即可返回，不在乎右/左表是否满足条件

## WHERE 和 HAVING的区别

1. WHERE子句出现在SELECT语句的后面，而HAVING子句出现在GROUP BY语句的后面。
2. WHERE子句用于指定过滤条件，而HAVING子句用于指定**分组后**的过滤条件。
3. WHERE子句不能与聚合函数一起使用，而HAVING子句可以与聚合函数一起使用。例如

   ```sql
    HAVING MAX(num) > 1000; // 正确
    WHERE MAX(num) > 1000; // 错误
   ```

## 子查询

子查询即在查询语句中嵌套其他查询语句，分为单行子查询和多行子查询，下面重点讲解多行子查询。

### 单行子查询

例如：

```sql
SELECT * FROM A WHERE id = (SELECT id FROM B WHERE name = 'Tom');
```

后方被括号圈起来的部分，就是子查询，因为只有一个子查询所以称为单行子查询

### 多行子查询

**多行比较作符号**:

| 符号 | 说明 |
| --- | --- |
| IN | 等于列表中的任意一个 |
| ANY | 与子查询中的任意一个值进行比较 |
| ALL | 与子查询中的所有值进行比较 |
| SOME | 与ANY作用相同 |

例如：

![](./imgs/zichaxun_Any.png)

- 只要满足`salary`小于[9000,6000,4800,4200]中任意一个值，`WHERE`条件成立

![](./imgs/zichaxun_All.png)

- `salary`要小于[9000,6000,4800,4200]中所有的值，`WHERE`条件才成立

### 结合`EXISTS`使用

- 子查询存在满足条件的行时：`EXISTS`返回`TRUE`
- 子查询不存在满足条件的行时：`EXISTS`返回`FALSE`

## 约束性

约束是对表中数据的规定，可以在创建表(`CREATE TABLE`)时定义，也可以在表创建后再定义(`ALTER TABLE`)。

例如：

通过`CREATE TABLE`创建表时定义约束：

```sql
CREATE TABLE A (
    id INT NOT NULL,
    name VARCHAR(20) NOT NULL,
    PRIMARY KEY (id)
);
```

通过`ALTER TABLE`创建表后定义约束：

```sql
-- 添加非空约束
ALTER TABLE 表名 MODIFY 字段名 数据类型 NOT NULL;
-- 删除非空约束，把NOT去掉即可
ALTER TABLE 表名 MODIFY 字段名 数据类型 NULL;
```

常见的表约束有：

- NOT NULL 非空约束，规定某个字段不能为空
- UNIQUE 唯一约束，规定某个字段在整个表中是唯一的
- PRIMARY KEY 主键(非空且唯一)约束
- FOREIGN KEY 外键约束
- CHECK 检查约束
- DEFAULT 默认值约束

### 非空约束

### 唯一约束

唯一约束可以指定单列，也可以通过组合的方式指定多列，例如：

```sql
-- 给NAME和PASSWORD的组合添加唯一约束
CREATE TABLE USER(
   id INT NOT NULL,
   NAME VARCHAR(25),
   PASSWORD VARCHAR(16),
   -- 使用表级约束语法
   -- uk_name_pwd是约束的名称，可以随便取
   CONSTRAINT uk_name_pwd UNIQUE(NAME,PASSWORD) 
);
```

> MySQL会给唯一约束的列上默认创建一个唯一索引。这样可以提高查询效率

### 主键约束

等同于唯一约束+非空约束，可以指定单列或者**多列(组合)**

**自增列**

对应关键字：`AUTO_INCREMENT`

一个表中只能有一个自增列，且必须是主键列或者唯一键列

> MySQL8.0后添加了[自增列持久化特性](#自增列持久化)

### 外键约束

使用外键约束的称为从表(子表)，被引用的表成为主表(父表)，如下所示：

```sql
create table dept( --主表 
  did int primary key, --部门编号 
  dname varchar(50) --部门名称 
);
  
create table emp(--从表 
  eid int primary key, --员工编号 
  ename varchar(5), --员工姓名 
  deptid int, --员工所在的部门
  foreign key (deptid) references dept(did) --在从表中指定外键约束 
);

-- 说明： 
--   （1）主表dept必须先创建成功，然后才能创建emp表，指定外键成功。 
--   （2）删除表时，先删除从表emp，再删除主表dept
```

**特点**：

1. 主表中被外键约束引用的列，必须是主键或者唯一键
2. 创建时先主表再从表，删除时先从表再主表
3. 一个表可以有多个外键约束

如果想要在建表后添加外键约束，格式如下：

格式：

```sql
ALTER TABLE 从表名 ADD [CONSTRAINT 约束名] FOREIGN KEY (从表的字段) REFERENCES 主表名(被引用 字段) [on update xx][on delete xx];
```

例如：

```sql
ALTER TABLE emp1 ADD [CONSTRAINT emp_dept_id_fk] FOREIGN KEY(dept_id) REFERENCES dept(dept_id);
```

**外键约束的缺点**

外键约束只适用于**单机低并发**场景，在**分布式**和**高并发**场景不适用；尤其是使用`Cascade`(级联)时，因为级联更新是强阻塞，存在数据库`更新风暴`的风险。同样外键还影响数据库的插入效率。

因此外键约束应该建立在`应用层`面，来保证数据一致性同时减少外键带来的效率低下和其他风险。

## 约束等级

- `Cascade` ：在父表上update/delete记录时，同步update/delete掉子表的匹配记录
- `Set null`：在父表上update/delete记录时，将子表上匹配记录的列设为null，但是要注意子
表的外键列不能为not null
- `No action` ：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作
- `Restrict`：同no action， 都是立即检查外键约束
- `Set default`（在可视化工具SQLyog中可能显示空白）：父表有变更时，子表将外键列设置成一个默认的值，但Innodb不能识别

如果不指定等级，默认为`Restrict`。指定等级的例子如下：

```sql
...
foreign key (deptid) references dept(did) on update cascade on delete set null
```

上述程式指定了指向`dept`表的`did`列的外键约束`deptid`的约束等级，在`UPDATE`操作上约束等级为`Cascade`，在`DELETE`操作上约束等级为`Set null`。

## 视图

视图是一种虚拟表，它的数据并不真实存在，视图建立在基表(已有的被视图依赖的数据表)的基础上。可以把视图理解为**存储起来的`SELECT`语句**，目的是在大项目中将常用的查询结果放到视图中，提升效率

### 视图的创建

```sql
CREATE [OR REPLACE] 
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] 
VIEW 视图名称 [(字段列表)] 
AS 查询语句 
[WITH [CASCADED|LOCAL] CHECK OPTION]
```

其中`[OR REPLACE]`是可选项，加上的话会在视图已存在时替换掉原来的视图。

`ALGORITHM`指定了创建视图的不同算法，解释如下：

- `UNDEFINED`：使用默认算法，即`MERGE`算法
- `MERGE`：无需创建临时表，而是将视图和基表合并形成一个新的虚拟表，可以对视图进行增删改的操作，但是多表联合时数据库管理系统会将所有表的数据逐一比较合并，影响查询效率
- `TEMPTABLE`：MySQL创建一个临时表来存储视图数据，临时表是只读的，不允许增删改操作。但是数据库管理系统只需要将基表的数据复制粘贴，不需要逐一比较合并，从而提升了查询效率

## 过程

将预先编译好的SQL语句进行封装，这么做的好处是：

- 提高SQL重用性，减少开发人员工作；
- 减少网络传输量，客户端不需要将所有SQL语句发送给服务器，只需要发送调用过程的SQL语句；
- 减少SQL语句暴露的风险，提高查询安全性

**与视图和函数比较**

- 视图是虚拟表，通常用来查询而不对基表进行增删改操作，而存储过程通常是对基表进行复杂的数据处理；
- 函数是有返回值的，而过程没有返回值；并且函数的输入参数默认为`IN`类型，过程则有`IN`、`OUT`、`INOUT`三种可选项。

### 过程的创建

```sql
CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名 参数类型,...)
[characteristics ...] 
BEGIN
  存储过程体 
END
```

- 其中`IN`, `OUT`, `INOUT`分别表示参数是入参，出参，入参和出参；(详情见后面过程的调用[例子](#过程的调用))
- `characteristics`可以指定存储过程的特性，如下：

```sql
LANGUAGE SQL 
| [NOT] DETERMINISTIC 
| { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA } 
| SQL SECURITY { DEFINER | INVOKER } 
| COMMENT 'string
```

- `LANGUAGE SQL`：指定存储过程的语言为SQL，如果不指定，默认为SQL
- `[NOT] DETERMINISTIC`：指定存储过程是否是确定性的(即相同的输入会得到相同的输出)，如果不指定，默认为`DETERMINISTIC`
- `{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }`：指定子程序使用SQL语句的限制：
  - `CONTAINS SQL`: 存储过程的子程序中包含SQL语句，但不包含读写数据的SQL语句
  - `NO SQL`: 子程序中不包含SQL语句
  - `READS SQL DATA`: 子程序中包含**读取**数据的SQL语句
  - `MODIFIES SQL DATA`: 子程序中包含**写入**数据的SQL语句
  - 默认为`CONTAINS SQL`
- `SQL SECURITY`: 指明哪些用户能执行当前存储的过程，`DEFINER`表示创建者，`INVOKER`表示调用者。默认`DEFINER`
- `COMMENT`: 存储过程的注释

### 过程的调用

- 先写一个存储过程(假设保存在服务器端)

  ```sql
  -- 为避免和SQL语句的结束符冲突，使用DELIMITER修改存储过程的结束符
  DELIMITER // 
  CREATE PROCEDURE CountProc(IN sid INT,OUT num INT) 
  BEGIN
    SELECT COUNT(*) INTO num FROM fruits 
    WHERE s_id = sid; 
  END // 
  DELIMITER ;
  ```

- 在客户端调用该存储过程：

  ```sql
  -- 通过CALL调用存储过程
  CALL CountProc(1,@num);
  SELECT @num;
  ```

- 如果存储过程中有`INOUT`参数，例如`name`，则在客户端的调用应该类似如下形式：

  ```sql
  SET @name='apple';
  CALL CountProc(1,@num,@name);
  SELECT @num,@name;
  ```

### 过程的优缺点

**优点**

- 一次编译多次使用，提高SQL执行效率
- 减少网络传输量
- 良好的封装，保证过程安全性，减小开发人员工作量

**缺点**

- 可移植性差，不能跨数据库使用
- 调试困难；
- 没有版本控制，版本迭代困难；
- 不适合高并发

## 函数

### 函数的创建

```sql
CREATE FUNCTION 函数名(参数名 参数类型,...) 
RETURNS 返回值类型 
[characteristics ...] 
BEGIN
  函数体 -- 函数体中肯定有RETURN 语句 
END
```

上述格式中[`characteristics`](#过程的创建)见过程的创建部分。参数默认都是`IN`

### 函数的调用

```sql
SELECT 函数名(参数名,...)
-- 例如
SELECT CountProc(1);
```

### 如何查看数据库的自定义过程和函数

**查看创建信息**

```sql
SHOW CREATE PROCEDURE|FUNCTION 函数名|过程名
-- 例如
SHOW CREATE PROCEDURE CountProc \G;
-- 最后\G是为了调整输出格式，使其更加美观
```

**查看状态信息**

```sql
SHOW {PROCEDURE | FUNCTION} STATUS [LIKE 'pattern']
-- 例如
SHOW PROCEDURE STATUS LIKE 'SELECT%' \G
```

状态信息回显示Definer、Comment、函数/过程的Name、Security_Type(`DEFINER` | `INVOKER`)等

## 变量

分为**环境系统变量**(GLOBAL)和**会话系统变量**(SESSION)两种。默认是会话变量(也可称作局部变量)。在开启与服务器的MySQL会话时，对会话变量的初始化是对环境变量进行复制。

- 全局系统变量仅对所有会话系统变量有效，不能**跨重启**
- 对当前会话系统变量的修改，不会影响其他会话系统变量；
- 当前会话对环境系统变量的修改，会影响其他会话系统变量。

### 查看系统变量

**查看全部系统变量**

```sql
--查看所有全局变量 
SHOW GLOBAL VARIABLES; 
--查看所有会话变量 
SHOW SESSION VARIABLES; 
--或
SHOW VARIABLES;
```

**查看指定系统变量**

```sql
-- 查看指定的系统变量的值 
SELECT @@global.变量名; 
-- 查看指定的会话变量的值 
SELECT @@session.变量名; 
-- 或者 
SELECT @@变量名;
```

> 不指定SESSION或者GLOBAL的话，`@@`先查看SESSION，SESSION不存在再去看GLOBAL

### 用户变量

分为**会话用户变量**和**局部变量**

- 会话用户变量：只对当前连接会话有效，用`@`表示
- 局部变量：只在BEGIN和END语句块中有效，用于存储过程和函数，用`DECLARE`定义

```sql
DELIMITER // 

CREATE PROCEDURE set_value() 
BEGIN
  -- DECLARE 变量名 类型 [default 值]
  -- 不声明默认值默认为Null
  DECLARE emp_name VARCHAR(25); 
  DECLARE sal DOUBLE(10,2); 
  SELECT last_name,salary INTO emp_name,sal 
  FROM employees 
  WHERE employee_id = 102; 
  SELECT emp_name,sal; 
END // 

DELIMITER ;
```

### 定义条件与处理程序

定义条件和处理程序就类似Java里的try-catch-finally，用于处理异常。其中：

- 定义条件：指程序运行中可能出现的问题
- 处理程序：指出现问题时采取什么处理方式

语法格式如下：

```sql
-- 定义错误条件
DECLARE 错误名称 CONDITION FOR 错误码(或错误条件)
-- 定义处理程序
DECLARE 处理方式 HANDLER FOR 错误类型 处理语句
```

- 错误码
  - 有**数值类型的错误码**`MySQL_error_code`和**字符串类型错误码**`sqlstate_value`
  - 例如在`ERROR 1418 (HY000)`中1418是`MySQL_error_code`，HY000是`sqlstate_value`(长度5位)
- 处理方式有3种：`CONTINUE`, `EXIT`, `UNDO`
  - 其中`CONTINUE`和`EXIT`是继续执行不处理和马上退出；
  - `UNDO`是撤回之前的操作，MySQL暂时不支持该操作
- 错误类型：
  - `SQLEXCEPTION`：所有的SQL错误
  - `NOT FOUND`：没有找到记录，通常为02开头的`sqlstate_value`
  - `SQLSTATE 'sqlstate_value'`：指定的SQL状态值
  - `SQLWARNING`：所有的SQL警告，通常为01开头的`sqlstate_value`
  - `MySQL_error_code`：匹配数值类型错误码
  - `错误名称`：匹配错误名称

## 流程控制

- 条件判断语句 ：IF 语句和 CASE 语句
- 循环语句 ：LOOP、WHILE 和 REPEAT 语句
- 跳转语句 ：ITERATE 和 LEAVE 语句

### 条件判断

**IF**

语句格式

```sql
IF 表达式1 THEN 操作1 
[ELSEIF 表达式2 THEN 操作2]
[ELSE 操作N] 
END IF
```

**CASE**

`CASE`有两种格式，一种类似swtich，另一种类似多重if。先来看类似switch的语句格式：

```sql
CASE 表达式
WHEN 值1 THEN 操作1
WHEN 值2 THEN 操作2
...
END [case] -- 放在BEGIN END语句块中时需要加上[case]
```

然后是多重if格式：

```sql
CASE
WHEN 条件1 THEN 操作1
...
END [case]
```

### 循环语句

`LOOP`语句格式：

```sql
-- loop_label 是语句块的标注，可以省略
[loop_label:] LOOP 
循环执行的语句 
END LOOP [loop_label]
```

`WHILE`语句格式：

```sql
[while_label:] WHILE 循环条件 DO 
循环体 
END WHILE [while_label];
```

`REPEAT`语句格式：(等同`do..while`)

```sql
[repeat_label:] REPEAT 
循环体的语句 
UNTIL 结束循环的条件表达式 
END REPEAT [repeat_label]
```

### 跳转语句

`LEAVE`：跳出循环，语句格式

```sql
LEAVE [loop_label]
```

`ITERATE`：等同`continue`，语句格式同上

## 游标

SQL通常是面向集合的编程，而游标的出现让我们可以实现面向过程编程。游标可以随意定位到某一行记录，充当了指针的作用。使用游标的步骤如下：

1. 声明游标

    MySQL的游标声明方法如下：

    ```sql
    -- select_statement代指select语句块
    -- 例如SELECT id, salary FROM employees
    DECLARE cursor_name CURSOR FOR select_statement;
    ```

2. 打开游标

因为游标需要占用系统资源，所以存在打开和关闭的步骤，如下：

```sql
OPEN cursor_name
CLOSE cursor_name
```

3. 使用游标

```sql
FETCH cursor_name INTO var_name [, var_name] ...
```
将游标指向的当前行数据读取到`var_name`变量中，如果有多个字段，用逗号分隔（**游标查询的字段数要和变量数一致**）。例如：

```sql
FETCH cursor_emp INTO emp_id, emp_sal;
```

> 注意`emp_id`和`emp_sal`这些都是变量，要在声明游标前就定义好


# InnoDB

## 数据页

InnoDB的数据是按照「**数据页**」为单位读写，默认大小为**16KB**，包含有7个部份，如右图所示，其中：

![Untitled](./imgs/InnoDB%E6%95%B0%E6%8D%AE%E9%A1%B5.png)

- File Header有两个指针，分别指向上个和下个数据页，形成**双向链表**
- 数据页中的行记录按照「主键」顺序形成**单向链表**，为了提高检索效率，因此加入了「页目录」

**页目录**

如右图所示

1. 将所有的记录划分成几个组，这些记录包括最小记录和最大记录，但**不包括标记为“已删除”的记录；**
2. 每个记录组的最后一条记录就是组内最大的那条记录，并且最后一条记录的头信息中会存储该组一共有多少条记录，作为 n_owned 字段（右图中粉红色字段）
3. 页目录用来存储**每组最后一条记录的地址偏移量**，这些地址偏移量会按照先后顺序存储起来，每组的地址偏移量也被称之为槽（slot），**每个槽相当于指针指向了不同组的最后一个记录**。

![Untitled](./imgs/数据页细节.png)

# 事务

## 并发事务的问题

- 脏读：一个事务「读到」了另一个「未提交事务修改过的数据」
- 不可重复读：在**一个事务内**多次读取同一个数据，如果出现**前后两次读到的数据不一样**的情况。
- 幻读：在一个事务内多次查询某个符合查询条件的「记录数量」，如果出现前后两次查询到的记录数量不一样的情况。

> 区分幻读和不可重复读
A：幻读是指查询的**记录数量**前后不同，不可重复读是指**具体数据值**的前后不同
>
## 特性

**概念：**

- **原子性（Atomicity）**：一个事务要么完成要么不完成
- **一致性（Consistency）**：是指事务操作前和操作后，数据满足完整性约束，数据库保持一致性状态。
- **隔离性（Isolation）**：每个事务都有一个完整的数据空间，对其他并发事务是隔离的。
- **持久性（Durability）**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

**如何保证：**

- 持久性是通过 redo log （重做日志）来保证的；
- 原子性是通过 undo log（回滚日志） 来保证的；
- 隔离性是通过 MVCC（多版本并发控制） 或锁机制来保证的；
- 一致性则是通过持久性+原子性+隔离性来保证；

## 隔离级别

**概念**

级别越高，性能越低

- **读未提交（read uncommitted）**：指一个事务还没提交时，它做的变更就能被其他事务看到；
- **读提交（read committed)**：指一个事务提交之后，它做的变更才能被其他事务看到；
- **可重复读（repeatable read）**：指一个事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，**MySQL InnoDB 引擎的默认隔离级别**；
- **串行化（serializable）**：会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行；

![不同隔离级别会出现的并发问题](./imgs/isolationDegree.png)

不同隔离级别会出现的并发问题

**实现**

- 读未提交：不加锁直接读
- 读提交：通过**ReadView**实现，在「每个语句执行前」都重新生成一个ReadView
- 可重复读：「启动事务时」生成一个ReadView
- 串行化：加读写锁进行访问，同一时刻只允许一个线程读取

## Read View

需要了解的：

- Read View四个字段的作用
- 聚簇索引记录中两个跟事务有关的隐藏列

**四个字段的作用**

![](./imgs/ReadView.png)

- **creator_trx_id**：当前事务的ID
- **m_ids**：当前事务启动时，系统中所有「活跃但未提交」事务的ID
- **min_trx_id**: 当前事务启动时，系统中所有「活跃但未提交」事务中**最小**ID
- **max_trx_id**: 同上，但是**最大**ID

**MVCC**

中文名：多版本并发控制

根据当前事务的`min_trx_id`和`max_trx_id`，我们可以讲对于当前事务的其他事务划分为三类，如下图所示：

![Untitled](./imgs/MVCC.png)

- 「已提交事务」，即小于`min_trx_id`，是对当前事务**可见**的;
- 「还没开始事务」，即大于`max_trx_id`，是对当前事务**不可见**的
- 「已启动未提交事务」，即在`min_trx_id`和`max_trx_id`之间，需要判断这个事务的`trx_id`是否在`m_ids`
  - 如果在，说明该事务还未提交，是对当前事务**不可见**的
  - 如果不在，说明该事务已经提交，是对当前事务**可见**的

# MySQL8新特性

## DDL 原子性

例如数据库中存在表`book1`，但是执行删除操作：

```sql
DROP TABLE book1, book2;
```

- 在 MySQL 8之前，虽然不存在`book2`会报错，但是`book1`表也会被删除；
- 而在 MySQL 8中，为保证事务完整性，即**DDL要么操作成功要么回滚**，因此在报错无法找到`book2`后，会发生事务回滚，这样`book1`就不会被删除

## 计算列

计算列指该列是通过其他列计算得到。MySQL8中支持创建表格的时候直接定义计算列，例如：

```sql
CREATE TABLE t1 (
    a INT,
    b INT,
    c INT AS (a + b) VIRTUAL
);
```

## 自增列持久化

MySQL8中支持将自增列的值持久化到磁盘中，这样在重启数据库后，自增列的值不会被重置。

当主键id是[1,2,3,4]时，删除id=4的行数据后，再次添加一行数据，id会从5开始，而不是4。

由于MySQL5.7的计数器只在内存中进行维护，重启数据库中再次添加数据时，会从id=4开始。而MySQL8.0对自增列的计数器也进行了持久化，即使重启数据库，也会从id=5开始添加。

## 全局变量持久化

通常设置全局变量的方法如下：

```sql
-- 设置服务器语句执行的超时时间
SET GLOBAL MAX_EXECUTION_TIME=2000;
```

虽然这样设置能够影响当前的所有会话系统变量，但是当数据库重启后，**又会从配置文件中读取默认值**。在MySQL8.0中新增`SET PERSIST`，会将配置的全局变量保存到**数据目录下的`mysql-auto.cnf`文件**，这样重启后会读取该文件来覆盖默认值

```sql
SET PERSIST GLOBAL MAX_EXECUTION_TIME=2000;
```

# 面试问题

**Q: 为什么一般不使用`null`值？**

建表的时候通常会用`not null default ''`或`default 0`来杜绝使用`null`值，原因是：

1. `null`值在索引中不起作用，会导致索引失效，这样检索只能通过比较其他键值，降低检索效率
2. `null`值只能用专门的`is null`和`is not null`来比较，如果用运算符比较通常会返回`null`而不是布尔值。
