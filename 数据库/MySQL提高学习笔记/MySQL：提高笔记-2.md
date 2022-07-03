# MySQL：提高笔记-2

> 这是根据 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444)** 的课程 [mysql入门到精通](https://www.bilibili.com/video/BV1zJ411M7TB) 后做的笔记
>
> 之前学了一波 MySQL 在使用层面的内容，现在则将其进行一步升华，稍稍深入的学习一些 MySQL 的内部实现等进阶知识点，这当然也是跟着视频学习时做的笔记，课程地址：[MySQL入门到精通](https://www.bilibili.com/video/BV1zJ411M7TB)，笔记总共分为5篇，此为 **2/5**。

## 1. MySQL的体系结构概览

![MySQL的体系结构概览](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251652092.jpg) 

整个MySQL Server由以下组成

- Connection Pool : 连接池组件
- Management Services & Utilities : 管理服务和工具组件
- SQL Interface : SQL接口组件
- Parser : 查询分析器组件
- Optimizer : 优化器组件
- Caches & Buffers : 缓冲池组件
- Pluggable Storage Engines : 存储引擎
- File System : 文件系统

1） **连接层**

最上层是一些客户端和链接服务，包含本地 sock 通信和大多数基于客户端/服务端工具实现的类似于 TCP/IP 的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

2） **服务层**

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如 过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

3） **引擎层**

存储引擎层， 存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。

4）**存储层**

数据存储层， 主要是将数据存储在文件系统之上，并完成与存储引擎的交互。

和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。**主要体现在存储引擎上，插件式的存储引擎架构**，将查询处理和其他的系统任务以及数据的存储提取分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

## 2. 存储引擎

### 2.1 存储引擎概述

和大多数的数据库不同，MySQL中有一个存储引擎的概念，针对不同的存储需求可以选择最优的存储引擎。

存储引擎就是存储数据，建立索引，更新查询数据等等技术的实现方式 。**存储引擎是基于表的**，而不是基于库的。所以存储引擎也可被称为表类型。

Oracle，SqlServer 等数据库只有一种存储引擎。**MySQL提供了插件式的存储引擎架构**。所以MySQL存在多种存储引擎，可以根据需要使用相应引擎，或者编写存储引擎。

MySQL5.0支持的存储引擎包含 ： InnoDB 、MyISAM 、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、ARCHIVE、CSV、BLACKHOLE、FEDERATED等，其中 InnoDB 和 BDB 提供事务安全表，其他存储引擎是非事务安全表。

可以通过指定 `show engines`，来查询当前数据库支持的存储引擎 ： 

![存储引擎](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251652601.png) 

创建新表时如果不指定存储引擎，那么系统就会使用默认的存储引擎，MySQL5.5之前的默认存储引擎是 MyISAM，**5.5之后就改为了InnoDB**。

查看Mysql数据库默认的存储引擎，指令 ：

```mysql
 show variables like '%storage_engine%';
 
 mysql>  show variables like '%storage_engine%';
+------------------------+--------+
| Variable_name          | Value  |
+------------------------+--------+
| default_storage_engine | InnoDB |
| storage_engine         | InnoDB |
+------------------------+--------+
2 rows in set (0.01 sec)
```

### 2.2 各种存储引擎特性

下面重点介绍几种常用的存储引擎，并对比各个存储引擎之间的区别，如下表所示 ： 

| 特点         | InnoDB               | MyISAM   | MEMORY   | MERGE | NDB      |
| ------------ | -------------------- | -------- | -------- | ----- | -------- |
| 存储限制     | 64TB                 | 有       | 有       | 没有  | 有       |
| 事务安全     | **支持**             |          |          |       |          |
| 锁机制       | **行锁(适合高并发)** | 表锁     | 表锁     | 表锁  | **行锁** |
| B树索引      | 支持                 | 支持     | 支持     | 支持  | 支持     |
| 哈希索引     |                      |          | **支持** |       |          |
| 全文索引     | 支持(5.6版本之后)    | 支持     |          |       |          |
| 集群索引     | 支持                 |          |          |       |          |
| 数据索引     | 支持                 |          | 支持     |       | 支持     |
| 索引缓存     | 支持                 | 支持     | 支持     | 支持  | 支持     |
| 数据可压缩   |                      | **支持** |          |       |          |
| 空间使用     | 高                   | 低       | N/A      | 低    | 低       |
| 内存使用     | 高                   | 低       | 中等     | 低    | 高       |
| 批量插入速度 | 低                   | 高       | 高       | 高    | 高       |
| 支持外键     | **支持**             |          |          |       |          |

下面我们将重点介绍最长使用的两种存储引擎： InnoDB、MyISAM，另外两种 MEMORY、MERGE，了解即可。

#### 2.2.1 InnoDB

InnoDB存储引擎是MySQL的默认存储引擎。**InnoDB存储引擎提供了具有提交、回滚、崩溃恢复能力的事务安全**。但是对比MyISAM的存储引擎，InnoDB写的处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引。

InnoDB存储引擎不同于其他存储引擎的特点 ： 

**事务控制**

```mysql
-- 建表
create table goods_innodb(
	id int NOT NULL AUTO_INCREMENT,
	name varchar(20) NOT NULL,
    primary key(id)
)ENGINE=innodb DEFAULT CHARSET=utf8;

-- 开始事物 操作 提交
start transaction;

insert into goods_innodb(id,name)values(null,'Meta20');

commit;

-- 示例
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO goods_innodb(id,NAME)VALUES(NULL,'Meta20');
Query OK, 1 row affected (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

测试，发现在InnoDB中是存在事务的 ;

**外键约束**

**MySQL支持外键的存储引擎只有InnoDB**，在创建外键的时候，要求父表必须有对应的索引，子表在创建外键的时候，也会自动的创建对应的索引。

下面两张表中，country_innodb是父表，country_id为主键索引，city_innodb表是子表，country_id字段为外键，对应于country_innodb表的主键country_id 。

```mysql
-- 建表
create table country_innodb(
	country_id int NOT NULL AUTO_INCREMENT,
    country_name varchar(100) NOT NULL,
    primary key(country_id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

create table city_innodb(
	city_id int NOT NULL AUTO_INCREMENT,
    city_name varchar(50) NOT NULL,
    country_id int NOT NULL,
    primary key(city_id),
    -- 建立索引
    key idx_fk_country_id(country_id),
    -- 建立外键
    CONSTRAINT `fk_city_country` FOREIGN KEY(country_id) REFERENCES country_innodb(country_id) ON DELETE RESTRICT ON UPDATE CASCADE
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 插入数据
insert into country_innodb values(null,'China'),(null,'America'),(null,'Japan');
insert into city_innodb values(null,'Xian',1),(null,'NewYork',2),(null,'BeiJing',1);
```

在创建索引时， 可以指定在删除、更新父表时，对子表进行的相应操作，包括 `RESTRICT`、`CASCADE`、`SET NULL` 和 `NO ACTION`。

`RESTRICT`和`NO ACTION`相同， 是指限制在子表有关联记录的情况下， 父表不能更新；

`CASCADE`表示父表在更新或者删除时，更新或者删除子表对应的记录；

`SET NULL` 则表示父表在更新或者删除的时候，子表的对应字段被`SET NULL`。

针对上面创建的两个表，子表的外键指定是`ON DELETE RESTRICT ON UPDATE CASCADE` 方式的， 那么在主表删除记录的时候， 如果子表有对应记录， 则不允许删除， 主表在更新记录的时候， 如果子表有对应记录， 则子表对应更新 。

表中数据如下图所示 ： 

```mysql
mysql> select * from city_innodb;
+---------+-----------+------------+
| city_id | city_name | country_id |
+---------+-----------+------------+
|       1 | Xian      |          1 |
|       2 | NewYork   |          2 |
|       3 | BeiJing   |          1 |
+---------+-----------+------------+
3 rows in set (0.00 sec)

mysql> select * from country_innodb;
+------------+--------------+
| country_id | country_name |
+------------+--------------+
|          1 | China        |
|          2 | America      |
|          3 | Japan        |
+------------+--------------+
3 rows in set (0.00 sec) 
```

外键信息可以使用如下两种方式查看 ： 

```mysql
show create table city_innodb;

mysql> show create table city_innodb;
+-------------+-----------------+
| Table       | Create Table    |
+-------------+-----------------+
| city_innodb | CREATE TABLE `city_innodb` (
  `city_id` int(11) NOT NULL AUTO_INCREMENT,
  `city_name` varchar(50) NOT NULL,
  `country_id` int(11) NOT NULL,
  PRIMARY KEY (`city_id`),
  KEY `idx_fk_country_id` (`country_id`),
  CONSTRAINT `fk_city_country` FOREIGN KEY (`country_id`) REFERENCES `country_innodb` (`country_id`) ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 |
+-------------+-----------------+
1 row in set (0.01 sec)
```

删除country_id为1 的country数据： 

```mysql
mysql> delete from country_innodb where country_id = 1;
-- 报错，a foreign key constraint 有一个外键的限定
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`db1`.`city_innodb`, CONSTRAINT `fk_city_country` FOREIGN KEY (`country_id`) REFERENCES `country_innodb` (`country_id`) ON UPDATE CASCADE) 
```

更新主表country表的字段 country_id : 

```mysql
mysql> update country_innodb set country_id = 100 where country_id = 1;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

-- 更新后， 子表的数据信息为:
mysql> select * from city_innodb;
+---------+-----------+------------+
| city_id | city_name | country_id |
+---------+-----------+------------+
|       1 | Xian      |        100 |
|       2 | NewYork   |          2 |
|       3 | BeiJing   |        100 |
+---------+-----------+------------+
3 rows in set (0.00 sec)
```

**存储方式**	

InnoDB 存储表和索引有以下两种方式 ： 

1. 使用共享表空间存储， 这种方式创建的表的表结构保存在 `.frm` 文件中， 数据和索引保存在 innodb_data_home_dir 和 innodb_data_file_path定义的表空间中，可以是多个文件。

2. 使用多表空间存储， 这种方式创建的表的表结构仍然存在 `.frm` 文件中，但是每个表的数据和索引单独保存在 `.ibd` 中。

#### 2.2.2 MyISAM

MyISAM 不支持事务、也不支持外键，**其优势是访问的速度快**，对事务的完整性没有要求或者以 SELECT、INSERT 为主的应用基本上都可以使用这个引擎来创建表 。

有以下两个比较重要的特点： 

**不支持事务**

```mysql
-- 建表
create table goods_myisam(
	id int NOT NULL AUTO_INCREMENT,
	name varchar(20) NOT NULL,
    primary key(id)
)ENGINE=myisam DEFAULT CHARSET=utf8;  -- 使用myisam存储引擎

-- 查看是是否支持事物
mysql> start transaction;  -- 开启事物
Query OK, 0 rows affected (0.00 sec) 

mysql> insert into goods_myisam values(null, 'computer');  -- 在事物中插入数据
Query OK, 1 row affected (0.04 sec)

mysql> rollback;  -- 事物回滚
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from goods_myisam;  -- 查表，发现数据居然添加成功了
+----+----------+
| id | name     |
+----+----------+
|  1 | computer |
+----+----------+
1 row in set (0.00 sec)
```

通过测试，我们发现，在MyISAM存储引擎中，是没有事务控制的 ；

**文件存储方式**

每个 MyISAM 在磁盘上存储成3个文件，其文件名都和表名相同，但拓展名分别是 ： 

`.frm` (存储表定义)；

`.MYD` (MYData , 存储数据)；

`.MYI` (MYIndex , 存储索引)；

#### 2.2.3 MEMORY

Memory 存储引擎将**表的数据存放在内存**中。每个 MEMORY 表实际对应一个磁盘文件，格式是 `.frm`，该文件中只存储表的结构，而其数据文件，都是存储在内存中，这样有利于数据的快速处理，提高整个表的效率。MEMORY 类型的表访问非常地快，因为他的数据是存放在内存中的，并且默认使用 HASH 索引，但是服务一旦关闭，表中的数据就会丢失。

#### 2.2.4 MERGE

MERGE存储引擎是**一组MyISAM表的组合**，这些MyISAM表必须结构完全相同，**MERGE表本身并没有存储数据**，对MERGE类型的表可以进行查询、更新、删除操作，这些操作实际上是对内部的MyISAM表进行的。

对于MERGE类型表的插入操作，是通过 `INSERT_METHOD` 子句定义插入的表，可以有3个不同的值，使用 `FIRST` 或 `LAST` 值使得插入操作被相应地作用在第一或者最后一个表上，不定义这个子句或者定义为 `NO`，表示不能对这个MERGE表执行插入操作。

可以对MERGE表进行DROP操作，但是这个操作只是删除MERGE表的定义，对内部的表是没有任何影响的。

![merge搜索引擎](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251652702.png) 

下面是一个创建和使用MERGE表的示例 ： 

1. 创建3个测试表 order_1990, order_1991, order_all , 其中order_all是前两个表的MERGE表 ： 

```mysql
-- myisam 搜索引擎 的表
create table order_1990(
	order_id int ,
	order_money double(10,2),
	order_address varchar(50),
	primary key (order_id)
)engine = myisam default charset=utf8;

-- myisam 搜索引擎 的表
create table order_1991(
	order_id int ,
	order_money double(10,2),
	order_address varchar(50),
	primary key (order_id)
)engine = myisam default charset=utf8;

-- order_all 搜索引擎 的表
create table order_all(
	order_id int ,
	order_money double(10,2),
	order_address varchar(50),
	primary key (order_id)
)engine = merge union = (order_1990,order_1991) INSERT_METHOD=LAST default charset=utf8;
-- INSERT_METHOD子句定义插入的表，上述用了LAST
```

2. 分别向两张表中插入记录 

```sql
insert into order_1990 values(1,100.0,'北京');
insert into order_1990 values(2,100.0,'上海');

insert into order_1991 values(10,200.0,'北京');
insert into order_1991 values(11,200.0,'上海');
```

3. 查询3张表中的数据。

```mysql
mysql> select * from order_1991;
+----------+-------------+---------------+
| order_id | order_money | order_address |
+----------+-------------+---------------+
|       10 |      200.00 | 北京           |
|       11 |      200.00 | 上海           |
+----------+-------------+---------------+
2 rows in set (0.00 sec)

mysql> select * from order_1990;
+----------+-------------+---------------+
| order_id | order_money | order_address |
+----------+-------------+---------------+
|        1 |      100.00 | 北京           |
|        2 |      100.00 | 上海           |
+----------+-------------+---------------+
2 rows in set (0.00 sec)

mysql> select * from order_all;
+----------+-------------+---------------+
| order_id | order_money | order_address |
+----------+-------------+---------------+
|        1 |      100.00 | 北京           |
|        2 |      100.00 | 上海           |
|       10 |      200.00 | 北京           |
|       11 |      200.00 | 上海           |
+----------+-------------+---------------+
4 rows in set (0.00 sec)
```

4. 往order_all中插入一条记录 ，由于在MERGE表定义时，INSERT_METHOD 选择的是LAST，那么插入的数据会想最后一张表中插入。

```sql
insert into order_all values(100,10000.0,'西安');

mysql> select * from order_1991;
+----------+-------------+---------------+
| order_id | order_money | order_address |
+----------+-------------+---------------+
|       10 |      200.00 | 北京           |
|       11 |      200.00 | 上海           |
|      100 |    10000.00 | 西安           |
+----------+-------------+---------------+
3 rows in set (0.00 sec) 	 	
```

### 2.3 存储引擎的选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

以下是几种常用的存储引擎的使用环境。

- **InnoDB** : 是MySQL的默认存储引擎，用于事务处理应用程序，支持外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。InnoDB存储引擎除了有效的降低由于删除和更新导致的锁定(行锁)， 还可以确保事务的完整提交和回滚，对于类似于计费系统或者财务系统等对数据准确性要求比较高的系统，InnoDB是最合适的选择。
- **MyISAM** ： 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。
- **MEMORY**：将所有数据保存在RAM中，在需要快速定位记录和其他类似数据环境下，可以提供级快的访问。MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，其次是要确保表的数据可以恢复，数据库异常终止后表中的数据是可以恢复的。MEMORY表通常用于更新不太频繁的小表，用以快速得到访问结果。
- **MERGE**：用于将一系列等同的MyISAM表以逻辑方式组合在一起，并作为一个对象引用他们。MERGE表的优点在于可以突破对单个MyISAM表的大小限制，并且通过将不同的表分布在多个磁盘上，可以有效的改善MERGE表的访问效率。这对于存储诸如数据仓储等VLDB环境十分合适。

## 3. 优化SQL步骤

在应用的的开发过程中，由于初期数据量小，开发人员写 SQL 语句时更重视功能上的实现，但是当应用系统正式上线后，随着生产数据量的急剧增长，很多 SQL 语句开始逐渐显露出性能问题，对生产的影响也越来越大，此时这些有问题的 SQL 语句就成为整个系统性能的瓶颈，因此我们必须要对它们进行优化，本章将详细介绍在 MySQL 中优化 SQL 语句的方法。

当面对一个有 SQL 性能问题的数据库时，我们应该从何处入手来进行系统的分析，使得能够尽快定位问题 SQL 并尽快解决问题。

### 3.1 查看SQL执行频率

MySQL 客户端连接成功后，通过 `show [session|global] status` 命令可以提供服务器状态信息。`show [session|global] status` 可以根据需要加上参数`session`或者`global`来显示 session 级（当前连接）的统计结果和 global 级（自数据库上次启动至今）的统计结果。如果不写，默认使用参数是`session`。

```mysql
-- 下面的命令显示了当前 session 中所有统计参数的值：
show status like 'Com_______';
-- 结果：
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Com_binlog    | 0     |
| Com_commit    | 1     |
| Com_delete    | 1     |
| Com_insert    | 2     |
| Com_repair    | 0     |
| Com_revoke    | 0     |
| Com_select    | 11    |
| Com_signal    | 0     |
| Com_update    | 1     |
| Com_xa_end    | 0     |
+---------------+-------+
10 rows in set (0.00 sec)  


-- 查看 innodb 搜索引擎的状态
show status like 'Innodb_rows_%';
-- 结果
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| Innodb_rows_deleted  | 2     |
| Innodb_rows_inserted | 64    |
| Innodb_rows_read     | 315   |
| Innodb_rows_updated  | 4     |
+----------------------+-------+
4 rows in set (0.00 sec)
```

Com_xxx 表示每个 xxx 语句执行的次数，我们通常比较关心的是以下几个统计参数。

| 参数                 | 含义                                                         |
| :------------------- | ------------------------------------------------------------ |
| Com_select           | 执行 select 操作的次数，一次查询只累加 1。                   |
| Com_insert           | 执行 insert 操作的次数，对于批量插入的 insert 操作，只累加一次。 |
| Com_update           | 执行 update 操作的次数。                                     |
| Com_delete           | 执行 delete 操作的次数。                                     |
| Innodb_rows_read     | select 查询返回的行数。                                      |
| Innodb_rows_inserted | 执行 insert 操作插入的行数。                                 |
| Innodb_rows_updated  | 执行 update 操作更新的行数。                                 |
| Innodb_rows_deleted  | 执行 delete 操作删除的行数。                                 |
| Connections          | 试图连接 MySQL 服务器的次数。                                |
| Uptime               | 服务器工作时间。                                             |
| Slow_queries         | 慢查询的次数。                                               |

`Com_*** `：这些参数对于所有存储引擎的表操作都会进行累计。

`Innodb_***`：这几个参数只是针对InnoDB存储引擎的，累加的算法也略有不同。

### 3.2 定位低效率执行SQL

可以通过以下两种方式定位执行效率较低的 SQL 语句。

- **慢查询日志** : 通过慢查询日志定位那些执行效率较低的 SQL 语句，用 `--log-slow-queries[=file_name]` 选项启动时，mysqld 写一个包含所有执行时间超过 `long_query_time` 秒的 SQL 语句的日志文件。后续进行介绍。
- **通过命令`show processlist`** : 慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用`show processlist`命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。

命令使用及分析：

```mysql
mysql> show processlist;
+----+------+-----------------+---------+---------+------+-------+------------------+
| Id | User | Host            | db      | Command | Time | State | Info             |
+----+------+-----------------+---------+---------+------+-------+------------------+
| 12 | root | localhost:7519  | demo_01 | Sleep   |  761 |       | NULL             |
| 27 | root | localhost:11335 | demo_01 | Query   |    0 | NULL  | show processlist |
+----+------+-----------------+---------+---------+------+-------+------------------+
2 rows in set (0.00 sec)
```

1. id列，用户登录mysql时，系统分配的"connection_id"，可以使用函数`connection_id()`查看

2. user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句

3. host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户

4. db列，显示这个进程目前连接的是哪个数据库

5. command列，显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接（connect）等

6. time列，显示这个状态持续的时间，单位是秒

7. state列，显示使用当前连接的sql语句的状态，很重要的列。

   state描述的是语句执行中的某一个状态。

   一个sql语句，以查询为例，可能需要经过 copying to tmp table、sorting result、sending data 等状态才可以完成。

8. info列，显示这个sql语句，是判断问题语句的一个重要依据

### 3.3 explain分析执行计划

通过以上步骤查询到效率低的 SQL 语句后，可以通过 EXPLAIN 或者 DESC 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

查询SQL语句的执行计划 ： 

```mysql
mysql> explain select * from city where city_id = 1;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | city  | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

mysql> explain select * from city where city_name = 'NewYork';
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | city  | ALL  | NULL          | NULL | NULL    | NULL |    4 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)  
```

字段分析：

| 字段          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | select 查询的序列号，是一组数字，表示的是查询中执行 select 子句或者是操作表的顺序。 |
| select_type   | 表示 select 的类型，常见的取值有：<br />SIMPLE：简单表，即不使用表连接或者子查询<br />PRIMARY：主查询，即外层的查询<br />UNION：UNION 中的第二个或者后面的查询语句<br />SUBQUERY：子查询中的第一个 SELECT <br />等... |
| table         | 输出结果集的表                                               |
| type          | 表示表的连接类型，性能由好到差的连接类型为：<br />system>const>eq_ref>ref>ref_or_null>index_merge>index_subquery>range>index>all |
| possible_keys | 表示查询时，可能使用的索引                                   |
| key           | 表示实际使用的索引                                           |
| key_len       | 索引字段的长度                                               |
| rows          | 扫描行的数量                                                 |
| extra         | 执行情况的说明和描述                                         |

#### 3.3.1 环境准备

![explain分析环境准备](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251652934.png) 

```mysql
-- 创建角色表
CREATE TABLE `t_role` (
  `id` varchar(32) NOT NULL,
  `role_name` varchar(255) DEFAULT NULL,
  `role_code` varchar(255) DEFAULT NULL,
  `description` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_role_name` (`role_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 创建用户表
CREATE TABLE `t_user` (
  `id` varchar(32) NOT NULL,
  `username` varchar(45) NOT NULL,
  `password` varchar(96) NOT NULL,
  `name` varchar(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_user_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 用户表和角色表之间的关联关系表
CREATE TABLE `user_role` (
  `id` int(11) NOT NULL auto_increment ,
  `user_id` varchar(32) DEFAULT NULL,
  `role_id` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_ur_user_id` (`user_id`),
  KEY `fk_ur_role_id` (`role_id`),
  CONSTRAINT `fk_ur_role_id` FOREIGN KEY (`role_id`) REFERENCES `t_role` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `fk_ur_user_id` FOREIGN KEY (`user_id`) REFERENCES `t_user` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 插入数据
insert into `t_user` (`id`, `username`, `password`, `name`) values('1','super','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','超级管理员');
insert into `t_user` (`id`, `username`, `password`, `name`) values('2','admin','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','系统管理员');
insert into `t_user` (`id`, `username`, `password`, `name`) values('3','itcast','$2a$10$8qmaHgUFUAmPR5pOuWhYWOr291WJYjHelUlYn07k5ELF8ZCrW0Cui','test02');
insert into `t_user` (`id`, `username`, `password`, `name`) values('4','stu1','$2a$10$pLtt2KDAFpwTWLjNsmTEi.oU1yOZyIn9XkziK/y/spH5rftCpUMZa','学生1');
insert into `t_user` (`id`, `username`, `password`, `name`) values('5','stu2','$2a$10$nxPKkYSez7uz2YQYUnwhR.z57km3yqKn3Hr/p1FR6ZKgc18u.Tvqm','学生2');
insert into `t_user` (`id`, `username`, `password`, `name`) values('6','t1','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','老师1');

INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('5','学生','student','学生');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('7','老师','teacher','老师');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('8','教学管理员','teachmanager','教学管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('9','管理员','admin','管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('10','超级管理员','super','超级管理员');

INSERT INTO user_role(id,user_id,role_id) VALUES(NULL, '1', '5'),(NULL, '1', '7'),(NULL, '2', '8'),(NULL, '3', '9'),(NULL, '4', '8'),(NULL, '5', '10') ;

-- 查看目前表数据
SELECT * FROM t_role;
id      role_name        role_code     description      
------  ---------------  ------------  -----------------
10      超级管理员         super         超级管理员  
5       学生              student       学生           
7       老师              teacher       老师           
8       教学管理员         teachmanager  教学管理员  
9       管理员            admin         管理员     

SELECT * FROM t_user;
id      username  password                                                      name             
------  --------  ------------------------------------------------------------  -----------------
1       super     $2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe  超级管理员  
2       admin     $2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe  系统管理员  
3       itcast    $2a$10$8qmaHgUFUAmPR5pOuWhYWOr291WJYjHelUlYn07k5ELF8ZCrW0Cui  test02           
4       stu1      $2a$10$pLtt2KDAFpwTWLjNsmTEi.oU1yOZyIn9XkziK/y/spH5rftCpUMZa  学生1          
5       stu2      $2a$10$nxPKkYSez7uz2YQYUnwhR.z57km3yqKn3Hr/p1FR6ZKgc18u.Tvqm  学生2          
6       t1        $2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe  老师1     

SELECT * FROM user_role;
    id  user_id  role_id  
------  -------  ---------
     1  1        5        
     2  1        7        
     3  2        8        
     4  3        9        
     5  4        8        
     6  5        10       
```

#### 3.3.2 explain 之 id

id 字段是 select 查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。id 情况有三种 ： 

1. id 相同表示加载表的顺序是从上到下。

```mysql
explain select * from t_role r, t_user u, user_role ur where r.id = ur.role_id and u.id = ur.user_id ;

    id  select_type  table   type    possible_keys              
------  -----------  ------  ------  ---------------------------
     1  SIMPLE       r       ALL     PRIMARY                    
     1  SIMPLE       ur      ref     fk_ur_user_id,fk_ur_role_id
     1  SIMPLE       u       eq_ref  PRIMARY                    
     
  key            key_len  ref                 rows    Extra        
  -------------  -------  ------------------  ------  -------------
  (NULL)         (NULL)   (NULL)                   5               
  fk_ur_role_id  99       demo_01.r.id             1  Using where  
  PRIMARY        98       demo_01.ur.user_id       1               
```

2. id 不同id值越大，优先级越高，越先被执行。 

``` mysql
explain select * from t_role where id = (select role_id from user_role where user_id = (select id from t_user where username = 'stu1'));

    id  select_type  table      type    possible_keys       
------  -----------  ---------  ------  --------------------
     1  PRIMARY      t_role     const   PRIMARY             
     2  SUBQUERY     user_role  ref     fk_ur_user_id       
     3  SUBQUERY     t_user     const   unique_user_username
     
  key                   key_len  ref       rows  Extra        
  --------------------  -------  ------  ------  -------------
  PRIMARY               98       const        1               
  fk_ur_user_id         99                    1  Using where  
  unique_user_username  137                   1  Using index  
```

3. id 有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。

```mysql
explain select * from t_role r , (select * from user_role ur where ur.`user_id` = '2') a where r.id = a.role_id ; 

    id  select_type  table       type    possible_keys
------  -----------  ----------  ------  -------------
     1  PRIMARY      <derived2>  system  (NULL)       
     1  PRIMARY      r           const   PRIMARY      
     2  DERIVED      ur          ref     fk_ur_user_id
     
  key            key_len  ref       rows  Extra        
  -------------  -------  ------  ------  -------------
  (NULL)         (NULL)   (NULL)       1               
  PRIMARY        98       const        1               
  fk_ur_user_id  99                    1  Using where  
```

#### 3.3.3 explain 之 select_type

 表示 SELECT 的类型，常见的取值，如下表所示：

| select_type  | 含义                                                         | 距离                      |
| ------------ | ------------------------------------------------------------ | ------------------------- |
| SIMPLE       | 简单的select查询，查询中不包含子查询或者 union               | 参考上方1）               |
| PRIMARY      | 查询中若包含任何复杂的子查询，最外层查询标记为该标识         | 参考上方2）3）            |
| SUBQUERY     | 在SELECT 或 WHERE 列表中包含了子查询                         | 参考上方2）3）            |
| DERIVED      | 在 FROM 列表中包含的子查询，被标记为 DERIVED（衍生） MYSQL会递归执行这些子查询，把结果放在临时表中 | 参考上方3）<br />参考下面 |
| UNION        | 若第二个 SELECT 出现在 UNION 之后，则标记为 UNION ； 若 UNION 包含在 FROM子句的子查询中，外层SELECT将被标记为 ： DERIVED | 参考下面                  |
| UNION RESULT | 从 UNION 表获取结果的 SELECT                                 | 参考下面                  |

**DERIVED**：

```mysql
explain select a.* from (select * from t_user where id in ("1", "2")) as a;

    id  select_type  table       type    possible_keys
------  -----------  ----------  ------  -------------
     1  PRIMARY      <derived2>  ALL     (NULL)       
     2  DERIVED      t_user      range   PRIMARY      
     
  key      key_len  ref       rows  Extra        
  -------  -------  ------  ------  -------------
  (NULL)   (NULL)   (NULL)       2               
  PRIMARY  98       (NULL)       2  Using where  
```

**UNION 和 UNION RESULT：**

```mysql
-- 1. 第二个select出现在union之后
-- 2.
explain select * from t_user where id="1" union select * from t_user where id="2";

    id  select_type   table       type    possible_keys
------  ------------  ----------  ------  -------------
     1  PRIMARY       t_user      const   PRIMARY      
     2  UNION         t_user      const   PRIMARY      
(NULL)  UNION RESULT  <union1,2>  ALL     (NULL)       

  key      key_len  ref       rows  Extra   
  -------  -------  ------  ------  --------
  PRIMARY  98       const        1          
  PRIMARY  98       const        1          
  (NULL)   (NULL)   (NULL)  (NULL)          
```

#### 3.3.4 explain 之 table

即：展示这一行的数据是关于哪一张表的 

#### 3.3.5 explain 之 type

type 显示的是访问类型，是较为重要的一个指标，可取值为： 

| type   | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| NULL   | MySQL不访问任何表，索引，直接返回结果                        |
| system | 表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现 |
| const  | 表示通过索引一次就找到了，const 用于比较 primary key 或者 unique 索引。<br />因为只匹配一行数据，所以很快。<br />如将主键置于where列表中，MySQL 就能将该查询转换为一个常量。<br />const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较 |
| eq_ref | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常见于主键或唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| range  | 只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between，< , > , in 等操作。 |
| index  | index 与 ALL的区别为  index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件。 |
| all    | 将遍历全表以找到匹配的行                                     |

结果值从最好到最坏依次是：

```
NULL > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

system > const > eq_ref > ref > range > index > ALL
```

一般来说， 我们需要保证查询至少达到 range 级别， 最好达到ref 。

**NULL：MySQL不访问任何表，索引，直接返回结果**

```mysql
explain select now();

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       (NULL)  (NULL)  (NULL)       
     
  key     key_len  ref       rows  Extra           
  ------  -------  ------  ------  ----------------
  (NULL)  (NULL)   (NULL)  (NULL)  No tables used  
```

**system：**表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现

```mysql
explain select * from (select * from t_user where id='1') a;

    id  select_type  table       type    possible_keys
------  -----------  ----------  ------  -------------
     1  PRIMARY      <derived2>  system  (NULL)       
     2  DERIVED      t_user      const   PRIMARY      
     
  key      key_len  ref       rows  Extra   
  -------  -------  ------  ------  --------
  (NULL)   (NULL)   (NULL)       1          
  PRIMARY  98                    1          
```

**const:**  表示通过索引一次就找到了，const 用于比较 primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常量。const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较

```mysql
explain select * from t_user where id='1';

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  const   PRIMARY      
     
  key      key_len  ref       rows  Extra   
  -------  -------  ------  ------  --------
  PRIMARY  98       const        1          
```

**eq_ref：** 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常见于主键或唯一索引扫描

```mysql
explain select * from t_user u, t_role r where u.id=r.id;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       r       ALL     PRIMARY      
     1  SIMPLE       u       eq_ref  PRIMARY      
     
  key      key_len  ref             rows  Extra   
  -------  -------  ------------  ------  --------
  (NULL)   (NULL)   (NULL)             5          
  PRIMARY  98       demo_01.r.id       1          
```

**ref：**非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个）

```mysql
create index idx_username on t_user(name);  -- 创建非唯一性索引
explain select * from t_user where name="a";

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  ref     idx_username 
     
  key           key_len  ref       rows  Extra        
  ------------  -------  ------  ------  -------------
  idx_username  137      const        1  Using where  
```

**range：**只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between，< , > , in 等操作。

```mysql
explain select * from t_user where id in ("1", "2");

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  range   PRIMARY      
     
  key      key_len  ref       rows  Extra        
  -------  -------  ------  ------  -------------
  PRIMARY  98       (NULL)       2  Using where  
```

**index：**index 与 ALL的区别为  index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件。

```mysql
explain select id from t_user;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  index   (NULL)       
     
  key                   key_len  ref       rows  Extra        
  --------------------  -------  ------  ------  -------------
  unique_user_username  137      (NULL)       6  Using index  
```

**all：**将遍历全表以找到匹配的行

```mysql
explain select * from t_user;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  ALL     (NULL)       
     
  key     key_len  ref       rows  Extra   
  ------  -------  ------  ------  --------
  (NULL)  (NULL)   (NULL)       6          
```

#### 3.3.6 explain 之  key

* possible_keys: 显示可能应用在这张表的索引， 一个或多个。  

* key： 实际使用的索引， 如果为NULL， 则没有使用索引。 

* key_len: 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下， 长度越短越好 。 

#### 3.3.7 explain 之 rows

即扫描行的数量。

#### 3.3.8 explain 之 extra

其他的额外的执行计划信息，在该列展示 。

| extra            | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| using  filesort  | 说明 MySQL 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为 “文件排序”, 效率低。 |
| using  temporary | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和 group by； 效率低 |
| using  index     | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。 |

**using  filesort：**说明 MySQL 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为 “文件排序”, 效率低。

```mysql
mysql> explain select * from t_user order by password;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  ALL     (NULL)       
     
  key     key_len  ref       rows  Extra           
  ------  -------  ------  ------  ----------------
  (NULL)  (NULL)   (NULL)       6  Using filesort  
```

**using  index：**表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。

```mysql
explain select id from t_user order by id;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  index   (NULL)       
     
  key      key_len  ref       rows  Extra        
  -------  -------  ------  ------  -------------
  PRIMARY  98       (NULL)       6  Using index  
```

**using  temporary：**使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和 group by； 效率低

```mysql
explain select * from t_user group by password;

    id  select_type  table   type    possible_keys
------  -----------  ------  ------  -------------
     1  SIMPLE       t_user  ALL     (NULL)       
     
  key     key_len  ref       rows  Extra                            
  ------  -------  ------  ------  ---------------------------------
  (NULL)  (NULL)   (NULL)       6  Using temporary; Using filesort  
```

### 3.4 show profile 分析 SQL

Mysql从5.0.37 版本开始增加了对 `show profiles` 和 `show profile` 语句的支持。show profiles 能够在做 SQL 优化时帮助我们了解时间都耗费到哪里去了。

通过 have_profiling 参数，能够看到当前MySQL是否支持profile：

```mysql
mysql> select @@have_profiling;
+------------------+
| @@have_profiling |
+------------------+
| YES              |
+------------------+
1 row in set (0.00 sec) 
```

默认 profiling 是关闭的，可以通过 set 语句在 Session 级别开启 profiling：

```mysql
mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set (0.00 sec) 

mysql> set profiling=1; -- 开启profiling 开关
mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)
```

通过 profile，我们能够更清楚地了解 SQL 执行的过程。

首先，我们可以执行一系列的操作，如下图所示：

```mysql
show databases;
use demo_01;
show tables;

select * from t_user where id < 5;
select count(*) from t_user;
```

执行完上述命令之后，再执行`show profiles` 指令， 来查看SQL语句执行的耗时：

```mysql
mysql> show profiles;
+----------+------------+-----------------------------------+
| Query_ID | Duration   | Query                             |
+----------+------------+-----------------------------------+
|        1 | 0.00010350 | select @@profiling                |
|        2 | 0.00103225 | show databases                    |
|        3 | 0.00010525 | SELECT DATABASE()                 |
|        4 | 0.00037475 | show tables                       |
|        5 | 0.00007650 | select * from t_user where id < 5 |
|        6 | 0.00007575 | select count(*) from t_user       |
+----------+------------+-----------------------------------+
6 rows in set (0.00 sec)
```

通过`show profile for query query_id` 语句可以查看到该SQL执行过程中每个线程的状态和消耗的时间：

```mysql
mysql> show profile for query 2;
+--------------------+----------+
| Status             | Duration |
+--------------------+----------+
| starting           | 0.000042 |
| Opening tables     | 0.000217 |
| System lock        | 0.000008 |
| init               | 0.000007 |
| optimizing         | 0.000004 |
| statistics         | 0.000008 |
| preparing          | 0.000005 |
| executing          | 0.000626 |
| Sending data       | 0.000016 |
| end                | 0.000004 |
| query end          | 0.000008 |
| closing tables     | 0.000002 |
| removing tmp table | 0.000006 |
| closing tables     | 0.000003 |
| freeing items      | 0.000072 |
| logging slow query | 0.000003 |
| cleaning up        | 0.000002 |
+--------------------+----------+
17 rows in set (0.00 sec)
```

> TIP ：Sending data 状态表示 MySQL 线程开始访问数据行并把结果返回给客户端，而不仅仅是返回个客户端。由于在 Sending data 状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。
> 
>> 这里由于数据量很小，因此没有体现，一旦数据量上去后就会有体现的

在获取到最消耗时间的线程状态后，MySQL 支持进一步选择 all、cpu、block io 、context switch、page faults 等明细类型类查看 MySQL 在使用什么资源上耗费了过高的时间。例如，选择查看CPU的耗费时间  ：

```mysql
mysql> show profile cpu for query 2;
+--------------------+----------+----------+------------+
| Status             | Duration | CPU_user | CPU_system |
+--------------------+----------+----------+------------+
| starting           | 0.000042 | 0.000000 |   0.000000 |
| Opening tables     | 0.000217 | 0.000000 |   0.000000 |
| System lock        | 0.000008 | 0.000000 |   0.000000 |
| init               | 0.000007 | 0.000000 |   0.000000 |
| optimizing         | 0.000004 | 0.000000 |   0.000000 |
| statistics         | 0.000008 | 0.000000 |   0.000000 |
| preparing          | 0.000005 | 0.000000 |   0.000000 |
| executing          | 0.000626 | 0.000000 |   0.000000 |
| Sending data       | 0.000016 | 0.000000 |   0.000000 |
| end                | 0.000004 | 0.000000 |   0.000000 |
| query end          | 0.000008 | 0.000000 |   0.000000 |
| closing tables     | 0.000002 | 0.000000 |   0.000000 |
| removing tmp table | 0.000006 | 0.000000 |   0.000000 |
| closing tables     | 0.000003 | 0.000000 |   0.000000 |
| freeing items      | 0.000072 | 0.000000 |   0.000000 |
| logging slow query | 0.000003 | 0.000000 |   0.000000 |
| cleaning up        | 0.000002 | 0.000000 |   0.000000 |
+--------------------+----------+----------+------------+
17 rows in set (0.00 sec)
```

对上述字段解释：

| 字段       | 含义                           |
| ---------- | ------------------------------ |
| Status     | sql 语句执行的状态             |
| Duration   | sql 执行过程中每一个步骤的耗时 |
| CPU_user   | 当前用户占有的cpu              |
| CPU_system | 系统占有的cpu                  |

### 3.5 trace 分析优化器执行计划

> 我用了 MySQL 8 来进行分析

MySQL5.6 提供了对 SQL 的跟踪 trace，通过trace文件能够进一步了解为什么优化器选择A计划, 而不是选择B计划。

打开trace，设置格式为 JSON，并设置 trace 最大能够使用的内存大小，避免解析过程中因为默认内存过小而不能够完整展示。

```sql
SET optimizer_trace="enabled=on", end_markers_in_json=on;
set optimizer_trace_max_mem_size=1000000;
```

执行SQL语句 ：

```sql
SELECT * FROM t_user WHERE id < 4;
```

最后， 检查 information_schema.optimizer_trace就可以知道MySQL是如何执行SQL的 ：

```mysql
select * from information_schema.optimizer_trace;

-- 输出就略了
```

