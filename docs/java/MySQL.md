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
  - [触发器](#触发器)
    - [优缺点](#优缺点)
  - [默认数据库](#默认数据库)
  - [用户权限和管理(待完善)](#用户权限和管理待完善)
- [数据库的逻辑架构](#数据库的逻辑架构)
  - [服务器处理客户端请求](#服务器处理客户端请求)
- [索引](#索引)
  - [哪些情况适合创建索引](#哪些情况适合创建索引)
- [InnoDB](#innodb)
  - [索引](#索引-1)
    - [如何避免索引回表？](#如何避免索引回表)
  - [不同数据引擎对比](#不同数据引擎对比)
  - [数据页](#数据页)
  - [缓冲池(Buffer Pool)](#缓冲池buffer-pool)
- [锁](#锁)
  - [全局锁](#全局锁)
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

![zichaxun_Any](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/zichaxun_Any.png)

- 只要满足`salary`小于[9000,6000,4800,4200]中任意一个值，`WHERE`条件成立

![zichaxun_All](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/zichaxun_All.png)

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

## 触发器

触发器的思想类似监听器，会监听某个时间的发生，同时可以选择在事件发生前或后，创建一个触发器的语句格式如下：

```sql
CREATE TRIGGER 触发器名称 
{BEFORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名 
FOR EACH ROW 
触发器执行的语句块;
```

例如通过触发器来检查：新插入的员工工资是否高于他自己的领导工资，如果高于则抛出异常，语句如下：

```sql
DELIMITER // 

CREATE TRIGGER salary_check_trigger 
BEFORE INSERT ON employees FOR EACH ROW 
BEGIN
  -- mgrsalary代表领导工资
  DECLARE mgrsalary DOUBLE; 
  -- 找到员工领导的工资，赋值给mgrsalary
  SELECT salary INTO mgrsalary FROM employees WHERE employee_id = NEW.manager_id; 
  -- 如果员工工资高于领导工资，则抛出异常
  IF NEW.salary > mgrsalary THEN SIGNAL SQLSTATE 'HY000' SET MESSAGE_TEXT = '薪资高于领导薪资错误'; 
  END IF; 
END // 

DELIMITER ;
```

### 优缺点

**优点**：

- **能保证数据的完整性**。如果我们将数据分成多个表进行保存，通过外键约束进行关联。当我们只对一个表进行增删改操作时，就需要触发器来计算其他表的数据，或抛出数据缺失的异常；
- **能帮助记录日志**；
- **检查操作的合法性**。

**缺点**：

- **可读性差**。因为触发器存储在数据库中，不受应用层控制；
- **相关数据的变更可能会导致触发器报错**

## 默认数据库

1. `mysql`

存储了用户信息，权限信息，运行过程的日志信息，时区信息等；

2. `information_schema`

保存的数据库的`元信息`，包括表、列、索引、视图、存储过程、事件、触发器等的定义信息，以及表的统计信息；

3. `performance_schema`

保存MySQL服务器运行的状态信息，用来监控MySQL服务的各类性能指标

4. `sys`

通过视图结合`information_schema`和`performance_schema`的信息。

## 用户权限和管理(待完善)

# 数据库的逻辑架构

## 服务器处理客户端请求

![20230419152053](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230419152053.png)

1. 连接管理
   - 首先要建立`TCP`连接；
   - 验证完用户名和密码后，将权限表中账号权限与该连接进行关联；
   - 服务器和客户端的交互需要从线程池中专门分配一个线程

2. 解析优化
   - `解析器`会对SQL语句先进行词法分析，识别SQL语句中每段字符串代表什么；然后进行语法分析，如果语法正确，会生成一棵`语法树`(如下所示）

    ![20230419152520](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230419152520.png)

   - `优化器`会对语法树进行优化，比如选择最优的索引，或者将多表查询转换为单表查询。查询策略有`选取-投影-连接`，例如：

    ```sql
    SELECT id,name FROM student WHERE gender = '女';
    ```

    先对`WHERE`进行选取，再只投影`id`和`name`，最后再进行连接，得到最终结果

3. 存储引擎

# 索引

索引通常有：

- 主键索引
- 唯一索引
- 全文索引
- 单列索引
- 多列(联合)索引

用来修饰索引的SQL语句形势如下：

```sql
[UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY] [index_name] (col_name [length]) [ASC | DESC]
```

- 第一个方括号内分别代表指定索引为唯一、全文、空间类型
- 第二个方括号表示该字段是索引，用`INDEX`或`KEY`都行
- `index_name`是索引名，通常联合索引的时候需要指定索引名，像单列索引不指定的情况写默认是字段名
- 在字段名后有个`length`，用来表示索引长度，针对字符串使用，比如`name`字段，如果只想索引前10个字符，可以写成`name(10)`

## 哪些情况适合创建索引

- 业务上具有唯一特性的，例如身份证号；
- 高频用作条件的字段或字段组合；
- 高频用作`GROUP BY`或`ORDER BY`的字段(组合)；
- 索引可以提高`DISTINCT`的效率；
- 对于`VARCHAR`类的索引，最好指定前缀长度，既能提升查询效率，也能节省索引空间；

# InnoDB

## 索引

```sql
create table T(
id int primary key, 
k int not null, 
name varchar(16),
index (k))engine=InnoDB;
```

如上面这段创建表格的SQL语句，其中有**主键索引**`id`和**非主键索引**`k`(通过`index(k)`建立索引)。给`k`加上索引的目的是为了加快查询速度，这里我们就需要先了解InnoDB存储数据的结构。

InnoDB存储数据的结构是B+树，每个节点的值是主键的值，如下图所示：

![20230429221115](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230429221115.png)

如果不给`k`加索引，那么查询`k`的值为`1`的数据时，需要遍历整个B+树，时间复杂度为`O(n)`。

当给`k`加上索引后，如上图所示，如果语句是`select * from table where k=5`，那么会先搜索k索引数得到主键id值后，再到主键索引树搜索一次，这个过程称作**回表**。

### 如何避免索引回表？

由于存在回表这一过程，导致查询效率下降，那么该如何去优化呢？

**覆盖索引**

举个例子：`select * from T where k between 3 and 5`，这句查询语句存在回表的过程，但是如果换做`select id from T where k between 3 and 5`，回表的过程就不存在了！

原因是k索引树上的叶子节点存储的是主键id，第二句SQL要查询的只有id值，而k索引树返回的结果已经“覆盖了”我们的查询需求，所以不需要进行回表，提升了查询的效率

**通过联合索引优化高频请求**

举个例子：一个主键是id，有name, age, id_card等字段的表中，这时候存在经常用身份证(id_card)查询个人姓名、年龄的高频请求。那么我可以将`id_card`作为我们的非主键索引，`(name, age)`作为联合索引，这样我们通过`select name,age from T where id_card=1`进行覆盖索引，就避免了回表的过程。

**最左前缀原则**

![20230429230416](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230429230416.png)

上图创建了`(name,age)`的联合索引，在B+树中会先以最左侧字段顺序排序，如果相同再比较下一个字段，以此类推。

这样的话当我们创建了一个`(a,b)`的联合索引，就不需要再单独针对`a`建立索引。而对于`b`来说在高频请求下则需要单独建立索引。

但我们通常是对高频请求建立联合索引，例如我们维护了`(id_card, name)`的联合索引，当我们想要通过`id_card`查询`address`时候，可以通过联合索引查询到主键id值，再通过回表来查询`address`。这样就不用为每个字段的组合都建立联合索引。

**索引下堆**

假设我们建立了联合索引`(name,age)`，要查询的语句如下：

```sql
select * from tuser where name like '张%' and age=10 and ismale=1;
```

在MySQL5.6之前，根据最左侧前缀原则，匹配到'张'开头的name得到主键值后，会进行回表，再进行age和ismale的判断，如下图所示

![20230429231856](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230429231856.png)

在MySQL5.6加入了索引下堆的优化，如下图所示，InnoDB在联合索引内部就对`age`字段进行了判断，这样回表的次数就减小了！

![20230429232157](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230429232157.png)

## 不同数据引擎对比

- InnoDB支持事务，优势是`可以安全恢复`；
- MyISAM支持全文索引、压缩、空间函数(GIS)等，优势是`速度快`；但不支持事务、外键等，崩溃后无法安全恢复数据。

## 数据页

InnoDB的数据是按照「**数据页**」为单位读写，默认大小为**16KB**，包含有7个部份，如右图所示，其中：

![20230412144440](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230412144440.png)

- File Header有两个指针，分别指向上个和下个数据页，形成**双向链表**
- 数据页中的行记录按照「主键」顺序形成**单向链表**，为了提高检索效率，因此加入了「页目录」

**页目录**

如右图所示

1. 将所有的记录划分成几个组，这些记录包括最小记录和最大记录，但**不包括标记为“已删除”的记录；**
2. 每个记录组的最后一条记录就是组内最大的那条记录，并且最后一条记录的头信息中会存储该组一共有多少条记录，作为 n_owned 字段（右图中粉红色字段）
3. 页目录用来存储**每组最后一条记录的地址偏移量**，这些地址偏移量会按照先后顺序存储起来，每组的地址偏移量也被称之为槽（slot），**每个槽相当于指针指向了不同组的最后一个记录**。

![数据页细节](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/%E6%95%B0%E6%8D%AE%E9%A1%B5%E7%BB%86%E8%8A%82.png)

## 缓冲池(Buffer Pool)

为提高读写效率，DBMS会申请占用内存作为数据缓冲池。

缓冲池中包括：数据页，索引页，插入缓存，自适应哈希索引，数据字典信息，锁信息等。

缓冲池中查询和更新数据的流程如下：

![20230419155311](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230419155311.png)

可以看到更新数据不会立即进行持久化，而是定期进行。

# 锁

根据加锁的范围可以分为：全局锁、表级锁和行锁三类

## 全局锁

加全局锁的命令：`Flush tables with read lock`，会让整个库处于只读状态，不允许更新操作。阻塞包括`DDL`、`DML`和事物提交的语句。

主要应用场景是：做全库逻辑备份。

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

![isolationDegree](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/isolationDegree.png)

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

![ReadView](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/ReadView.png)

- **creator_trx_id**：当前事务的ID
- **m_ids**：当前事务启动时，系统中所有「活跃但未提交」事务的ID
- **min_trx_id**: 当前事务启动时，系统中所有「活跃但未提交」事务中**最小**ID
- **max_trx_id**: 同上，但是**最大**ID

**MVCC**

中文名：多版本并发控制

根据当前事务的`min_trx_id`和`max_trx_id`，我们可以讲对于当前事务的其他事务划分为三类，如下图所示：

![MVCC](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/MVCC.png)

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
