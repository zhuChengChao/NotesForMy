# MySQL：基础学习笔记-3

> 记录一下 MySQL 基础的一些语法，便于查询，该部分内容主要是参考：bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444)** 的课程而做的笔记。

## 1. 事物

### 1.1 概述

事务执行是一个整体， 所有的 SQL 语句都必须执行成功。如果其中有 1 条 SQL 语句出现异常， 则所有的 SQL 语句都要回滚，整个业务执行失败  

MYSQL 中可以有两种方式进行事务的操作 ：

* 手动提交事务  

* 自动提交事务（默认）

  MySQL 默认每一条 DML(增删改)语句都是一个单独的事务，每条语句都会自动开启一个事务，语句执行完毕自动提交事务，MySQL 默认开始自动提交事务  

### 1.2 手动提交

SQL 语句：

* 开启事物：`start transaction;`
* 提交事务：`commit;`
* 回滚事务：`rollback;`

执行成功的情况： 开启事务--->执行多条 SQL 语句--->成功提交事务  

执行失败的情况： 开启事务--->执行多条 SQL 语句--->事务的回滚  

### 1.3 事务原理

事务开启之后，所有的操作都会临时保存到事务日志中，事务日志只有在得到 commit 命令才会同步到数据表中，其他任何情况都会清空事务日志（rollback，断开连接）

![事物原理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644844.JPG)

**事务的步骤**：

1. 客户端连接数据库服务器，创建连接时创建此用户临时日志文件
2. 开启事务以后，所有的操作都会先写入到临时日志文件中
3. 所有的查询操作从表中查询，但会经过日志文件加工后才返回
4. 如果事务提交则将日志文件中的数据写到表中，否则清空日志文件。  

### 1.4 回滚点

在某些成功的操作完成之后， 后续的操作有可能成功有可能失败， 但是不管成功还是失败， 前面操作都已经成功， 可以在当前成功的位置设置一个回滚点。可以供后续失败操作返回到该位置， 而不是返回所有操作， 这个点称之为回滚点。  

**回滚点的操作语句**：

* 设置回滚点：`savepoint 回滚点名字`
* 回到回滚点：`rollback to 回滚点名字`

### 1.5 事务的隔离级别

#### 1.5.1 事务的四大特性 ACID  

| 事务特性             | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| 原子性：Atomicity    | 每个事务都是一个整体，不可再拆分，事务中所有的 SQL 语句要么都执行成功， 要么都失败。 |
| 一致性：Consistency  | 事务在执行前数据库的状态与执行后数据库的状态保持一致。<br />如：转账前2个人的总金额是 2000，转账后 2 个人总金额也是 2000 |
| 隔离性：Isolation    | 事务与事务之间不应该相互影响，执行时保持隔离的状态。         |
| 持久性：Durability） | 一旦事务执行成功，对数据库的修改是持久的。就算关机，也是保存下来的。 |

#### 1.5.2 事务的隔离级别

事务在操作时的理想状态： 所有的事务之间保持隔离，互不影响。因为并发操作，多个用户同时访问同一个数据。可能引发并发访问的问题：  

| 并发访问的问题 | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| 脏读           | 一个事务读取到了另一个事务中尚未提交的数据                   |
| 不可重复读     | 一个事务中两次读取的数据内容不一致，要求的是一个事务中多次读取时数据是一致的， 这是事务 update 时引发的问题 |
| 幻读           | 一个事务中两次读取的数据的数量不一致，要求在一个事务多次读取的数据的数量是一致的，这是 insert 或 delete 时引发的问题 |

#### 1.5.3 MySQL 数据库的四种隔离级别

| 级别 | 名字     | 隔离级别           | 脏读 | 不可重复读 | 幻读 | 数据库默认隔离级别   |
| ---- | -------- | ------------------ | ---- | ---------- | ---- | -------------------- |
| 1    | 读未提交 | `read uncommitted` | 是   | 是         | 是   |                      |
| 2    | 读已提交 | `read committed`   | 否   | 是         | 是   | Oracle 和 SQL Server |
| 3    | 可重复读 | `repeatable read`  | 否   | 否         | 是   | MySQL                |
| 4    | 串行化   | `serializable`     | 否   | 否         | 否   |                      |

隔离级别越高，性能越差，安全性越高

**MySQL 事务隔离级别相关的命令**

* 查询全局事务隔离级别

  ```mysql
  mysql> select @@tx_isolation;
  +-----------------+
  | @@tx_isolation  |
  +-----------------+
  | REPEATABLE-READ |
  +-----------------+
  1 row in set (0.00 sec)
  ```
  
* 设置事务隔离级别，需要退出 MySQL 再重新登录才能看到隔离级别的变化

  ```mysql
  set global transaction isolation level 级别字符串;  -- 上面的隔离级别
  ```

### 1.6 事物隔离级别演示

#### 1.6.1 前置准备

```mysql
-- 创建数据表，并往里面添加相应数据
CREATE TABLE account (
    id INT PRIMARY KEY AUTO_INCREMENT,
    NAME VARCHAR(10),
    balance DOUBLE
);

-- 加入数据
INSERT INTO account (NAME, balance) VALUES ('zhangsan', 1000), ('lisi', 1000);

-- 查看数据
select * from account;
+----+----------+---------+
| id | NAME     | balance |
+----+----------+---------+
|  1 | zhangsan |    1000 |
|  2 | lisi     |    1000 |
+----+----------+---------+
2 rows in set (0.00 sec)
```

> 需要注意的是，后面的演示，起初数据都恢复成最开始的状态

#### 1.6.2 脏读演示

问题描述：**一个事务读取到了另一个事务中尚未提交的数据**

解决脏读的问题：将全局的隔离级别进行提升，**read commit 及其以上的隔离级别**，即可阻止脏读发生

```mysql
-- 设置数据库的隔离级别为最低
mysql> set global transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)

-- 查看现在的隔离级别
mysql> select @@tx_isolation;
+------------------+
| @@tx_isolation   |
+------------------+
| READ-UNCOMMITTED |
+------------------+
1 row in set (0.00 sec)

-- 窗口1：选择数据库，开启事物
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

-- 窗口2：选择数据库，开启事物
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

-- 窗口1：更新zhangsan和lisi的账户数据，但是不提交
mysql> update account set balance=balance-500 where id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> update account set balance=balance+500 where id=2;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

-- 窗口2：查看数据，居然发现数据变了！！！！
mysql> select * from account;
+----+----------+---------+
| id | NAME     | balance |
+----+----------+---------+
|  1 | zhangsan |     500 |
|  2 | lisi     |    1500 |
+----+----------+---------+
2 rows in set (0.00 sec)

-- 窗口1：回滚数据
mysql> rollback;
Query OK, 0 rows affected (0.03 sec)

-- 窗口2：再查看数据，数据又变了！！！
mysql> select * from account;
+----+----------+---------+
| id | NAME     | balance |
+----+----------+---------+
|  1 | zhangsan |    1000 |
|  2 | lisi     |    1000 |
+----+----------+---------+
2 rows in set (0.00 sec)
```

#### 1.6.4 不可重复读的演示

问题描述：一个事务中两次读取的数据内容不一致，要求的是一个事务中多次读取时数据是一致的， 这是事务 update 时引发的问题

解决不可重复读的问题：将全局的隔离级别进行提升为 repeatable read 及其以上，才能保证**在同一个事务中为了保证多次查询数据一致**  

```mysql
-- 设置数据库的隔离级别为最低
mysql> set global transaction isolation level read committed;
Query OK, 0 rows affected (0.00 sec)

-- 查看现在的隔离级别
mysql> select @@tx_isolation;
+----------------+
| @@tx_isolation |
+----------------+
| READ-COMMITTED |
+----------------+
1 row in set (0.00 sec)

-- 窗口2：选择数据库，开启事物
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

-- 窗口1：选择数据库，开启事物，并更新数据后提交
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
mysql> update account set balance=balance+500 where id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit;
Query OK, 0 rows affected (0.01 sec)

-- 窗口2：查看数据，发现数据变了，在B窗口开启的同一个事物中，两次查询一个数据结果不同！
mysql> select * from account;
+----+----------+---------+
| id | NAME     | balance |
+----+----------+---------+
|  1 | zhangsan |    1500 | 
|  2 | lisi     |    1000 |
+----+----------+---------+
2 rows in set (0.00 sec)
```

两次查询输出的结果不同，到底哪次是对的？不知道以哪次为准。 很多人认为这种情况就对了，无须困惑，当然是后面的为准。

**但是**：我们可以考虑这样一种情况， 比如银行程序需要将查询结果分别输出到电脑屏幕和发短信给客户， 结果在一个事务中针对不同的输出目的地进行的两次查询不一致， 导致文件和屏幕中的结果不一致， 银行工作人员就不知道以哪个为准了  

#### 1.6.4 幻读的演示

问题描述：一个事务中两次读取的数据的数量不一致，要求在一个事务多次读取的数据的数量是一致的，这是 insert 或 delete 时引发的问题

解决：使用 serializable 隔离级别，一个事务没有执行完，其他事务的 SQL 执行不了，可以挡住幻读，即：使用 serializable 隔离级别，一个事务没有执行完，其他事务的 SQL 执行不了，可以挡住幻读

> 但是，在 MySQL 中无法看到幻读的效果？
>
> 后续通过只是展示一下 serializable 隔离级别的效果

```mysql
-- 设置数据库的隔离级别为最低
mysql> set global transaction isolation level read committed;
Query OK, 0 rows affected (0.00 sec)

-- 查看现在的隔离级别
mysql> select @@tx_isolation;
+----------------+
| @@tx_isolation |
+----------------+
| SERIALIZABLE   |
+----------------+
1 row in set (0.00 sec)

-- 窗口1：选择数据库，开启事物，查询记录
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
mysql> select count(*) from account;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)

-- 窗口2：选择数据库，开启事物，添加一条记录，会等待，直到超时
mysql> use db1;
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
mysql> insert into account (name, balance) values ('wangwu', 1000);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

-- 窗口1：接着查询，发现数据不变
mysql> select count(*) from account;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)

-- 窗口1：提交事物
mysql> commit;
Query OK, 0 rows affected (0.00 sec)

-- 窗口2：窗口1提交事物后，等待的语句马上执行，返回结果
mysql> insert into account (name, balance) values ('wangwu', 1000);
Query OK, 1 row affected (3.33 sec)

-- 窗口1：继续查询，发现还是没有变，这才是正常的！
mysql> select count(*) from account;
+----------+
| count(*) |
+----------+
|        2 |
+----------+
1 row in set (0.00 sec)

-- 窗口2：提交事物
mysql> commit;
Query OK, 0 rows affected (0.01 sec)

-- 窗口1：再次查询，这回终于变了
mysql> select count(*) from account;
+----------+
| count(*) |
+----------+
|        3 |
+----------+
1 row in set (0.00 sec)
```

## 2. 数据库设计

### 2.1 数据规范化

**范式**：好的数据库设计对数据的存储性能和后期的程序开发， 都会产生重要的影响。建立科学的， 规范的数据库就需要满足一些规则来优化数据的设计和存储，这些规则就称为范式  

目前关系数据库有六种范式：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式（4NF）和第五范式（5NF，又称完美范式）。

满足最低要求的范式是第一范式（1NF）。在第一范式的基础上进一步满足更多规范要求的称为第二范式（2NF），其余范式以次类推。**一般说来，数据库只需满足第三范式（3NF）就行了**。  

### 2.2 第一范式：1NF

数据库表的每一列都是不可分割的原子数据项，不能是集合、数组等非原子数据项。即表中的某个列有多个值时，必须拆分为不同的列。 **简而言之，第一范式每一列不可再拆分，称为原子性**。

> 以下的表格就不满足 1NF ：
>
> <img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644495.PNG" alt="第一范式" style="zoom:50%;" />
>
> 对其进行修改，令其满足 1NF：
>
> ![第一范式2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644650.PNG)

### 2.3 第二范式：2NF

在满足第一范式的前提下，表中的每一个字段都完全依赖于主键。  

所谓完全依赖是指不能存在仅依赖主键一部分的列。 **简而言之， 第二范式就是在第一范式的基础上所有列完全依赖于主键列**。 

当存在一个**复合主键包含多个主键列**的时候， 才会发生不符合第二范式的情况。比如有一个主键有两个列，不能存在这样的属性，它只依赖于其中一个列，这就是不符合第二范式。  

第二范式的特点：

1. 一张表只描述一件事情。
2. 表中的每一列都完全依赖于主键  

> 有如下：
>
> ![第二范式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644330.PNG)

### 2.4 第三范式：3NF

在满足第二范式的前提下，表中的每一列都直接依赖于主键，而不是通过其它的列来间接依赖于主键。

简而言之，第三范式就是所有列不依赖于其它非主键列，也就是在满足 2NF 的基础上， 任何**非主列不得传递依赖于主键**。 所谓传递依赖， 指的是如果存在"A → B → C"的决定关系， 则 C 传递依赖于 A。因此， 满足第三范式的数据库表应该不存在如下依赖关系：主键列 → 非主键列 x → 非主键列 y  

> 有如下：
>
> ![第三范式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644087.PNG)

### 2.5 三大范式小结

| 范式 | 特点                                                         |
| ---- | ------------------------------------------------------------ |
| 1NF  | 原子性：表中每列不可再拆分。                                 |
| 2NF  | 不产生局部依赖，一张表只描述一件事情                         |
| 3NF  | 不产生传递依赖，表中每一列都直接依赖于主键；<br />而不是通过其它列间接依赖于主键。 |