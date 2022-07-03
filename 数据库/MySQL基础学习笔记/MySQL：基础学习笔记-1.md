# MySQL：基础学习笔记-1

> 记录一下 MySQL 基础的一些语法，便于查询，该部分内容主要是参考：bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444)** 的课程而做的笔记。

## 1. 概述

### 1.1 DBMS

DBMS (DataBase Management System) / 数据库管理系统：指一种操作和管理数据库的大型软件，用于建立、使用和维护数据库，对数据库进行统一管理和控制，以保证数据库的安全性和完整性。用户通过数据库管理系统访问数据库中表内的数据。

### 1.2 SQL

Structured Query Language 结构化查询语言，有以下分类：

* DDL：Data Definition Language，数据定义语言，如：建库，建表
* DML：Data Manipulation Language，数据操纵语言，如：对表中的记录操作增删改
* DQL：Data Query Language，数据查询语言，如：对表中的查询操作
* DCL：Data Control Language，数据控制语言，如：对用户权限的设置  

## 2. DDL：数据定义语言

### 2.1 操作数据库

**创建数据库**

* 创建数据库：`create database 数据库名;`

* 判断数据库是否已经存在，不存在则创建数据库：`create database if not exists 数据库名;`

* 创建数据库并指定字符集：`create database 数据库名 character set 字符集;`


**查看数据库**

* 查看所有的数据库：`show databases;`

* 查看某个数据库的定义信息：`show create database 数据库名;`


**修改数据库**

* 修改数据库默认的字符集：`alter database 数据库名 default character set 字符集名;`


**删除数据库**

* 删除数据库：`drop database 数据库名;`

**使用数据库**

* 查看正在使用的数据库：`select database();`

  > 使用的一个 MySQL 中的全局函数

* 使用/切换数据库：`use 数据库名;`

### 2.2 操作表结构

**创建表**

* 正常创建：

	```sql
	create table 表名(
		字段1  字段标识1，
	    字段2  字段标识2，
	    ...
	);
	```
	
	> ```sql
	> create table student (
	>     id int,
	>     name varchar(20),
	>     birthday date
	> );
	> ```

* 快速创建一个表结构相同的表：`create table 新表名 like 旧表名;`


**MySQL 常用数据类型**

| 类型    | 描述     |
| ------- | -------- |
| int     | 整形     |
| double  | 浮点型   |
| varchar | 字符串型 |
| data    | 日期型   |

> 不具体展开，后续会有补充

**查找表**

* 查看某个数据库中的所有表：`show tables;`

* 查看表结构：`desc 表名;`

* 查看创建表的 SQL 语句：`show create table 表名;`

**删除表**

* 直接删除表：`drop table 表名;`

* 判断表是否存在，如果存在则删除表：`drop table if exists 表名;` 

  > 与直接删除的区别：如果表不存在，不删除，存在则删除  

**修改表结构**

* 添加表列 add：`alter table 表名 add 列名 字段标识;`
* 修改列类型 modify：`alter table 表名 modify 列名 新的字段标识;`
* 修改列名 change ：`alter table 表名 change 旧列名 新列名 字段标识;`
* 删除列 drop ：`alter table 表名 drop 列名;`
* 修改表名：
  * `rename table 旧表名 to 新表名;`
  * `alter table 表名 rename to 新表名   -- 在sqllite中支持，上面的语句却不支持`
* 修改字符集：`alter table 表名 character set 字符集;`

### 2.3 数据表的约束

**约束种类**

| 约束名   | 约束关键字              |
| -------- | ----------------------- |
| 主键     | primary key             |
| 唯一     | unique                  |
| 非空     | not null                |
| 外键     | foreign key             |
| 检查约束 | check 注： mysql 不支持 |

#### 2.3.1 主键约束

**说明**：用来唯一标识数据库中的每一条记录：通常**不用业务字段作为主键**，单独给每张表设计一个 id 的字段，把 id 作为主键。 **主键是给数据库和程序使用的，不是给最终的客户使用的。所以主键有没有含义没有关系，只要不重复，非空就行**。  

主键关键字： primary key  

**主键的特点**：

1. 非空 not null
2. 唯一

**创建主键**

1. 在创建表的时候给字段添加主键：`字段名 字段类型 primary key`

2. 在已有表中添加主键：`alter table 表名 add primary key(字段名);`


**删除主键**

1. `alter table 表名 drop primary key;`

**主键自增**

* 默认：auto_increment 表示自动增长(字段类型必须是整数类型)

* 修改自增长的默认值起始值：

  默认地 AUTO_INCREMENT 的开始值是 1，如果希望修改起始值,请使用下列 SQL 语法

  ```mysql
  create table 表名(
  	列名 int primary key auto_increment
  ) auto_increment=起始值;
  ```

* delete 和 truncate 对自增长的影响

  * delete ： 删除所有的记录之后，自增长没有影响  
  * truncate：删除以后，自增长又重新开始。  

#### 2.3.2 唯一约束

表中某一列不能出现重复的值

基本格式：`字段名 字段类型 unique`

> null 没有数据，不存在重复的问题  

#### 2.3.3 非空约束

某一列不能为 null

基本语法格式：`字段名 字段类型 not null`

**默认值**：`字段名 字段类型 default 默认值`

> 如果一个字段设置了非空与唯一约束，该字段与主键的区别?  
>
> * 主键数在一个表中，只能有一个；不能出现多个主键；主键可以单列，也可以是多列
> * 自增长只能用在主键上

#### 2.3.4 外键约束

**外键**：在从表中与主表主键对应的那一列  

* **主表**： 一方，用来约束别人的表  
* **从表**： 多方，被别人约束的表

**创建约束的语法**

* 新建表时增加外键：

  `[constraint] [外键约束名] foreign key(外键字段名) references 主表名(主键字段名) [on update/delete cascade];`

* 已有表增加外键：

  `alter table 从表 add [constraint][外键约束名] foreign key(外键字段名) references 主表名(主键字段名);`

**删除外键**

* `alter table 从表 drop foreign key 外键名称;`

**外键的级联**

在修改和删除主表的主键时，同时更新或删除副表的外键值，称为级联操作

| 级联操作语法      | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| on update cascade | 级联更新， 只能是创建表的时候创建级联关系。<br />更新主表中的主键， 从表中的外键 列也自动同步更新 |
| on delete cascade | 级联删除                                                     |

## 3. DML：数据操纵语言：

### 3.1 插入记录

* 插入全部字段

  `insert into 表名 (字段名1, 字段名2, 字段名3,...) values (值1, 值2, 值3, ...);`

* 不写字段名

  `insert into 表名 values (值1, 值2, 值3, ...);`

* 插入部分数据

  `insert into 表名 (字段名1, 字段名2, 字段名3,...) values (值1, 值2, 值3, ...);`

  > 有添加数据的字段会使用 NULL  

### 3.2 蠕虫复制

将一张已经存在的表中的数据复制到另一张表中

语法格式：

* 将表名 2 中的所有的列复制到表名 1 中：

  `insert into 表名1 select * from 表名2;`

* 只复制部分列

  `insert into 表名1(列1, 列2) select 列1, 列2 from 表名2;`

### 3.3 更新表记录

* 不带条件修改数据：`update 表名 set 字段名=值; --修改所有行`
* 带条件修改数据：`update 表名 set 字段名=值 where 字段名=值;`

### 3.4 删除表记录

* 不带条件删除数据：`delete from 表名;`

* 带条件删除数据：`delete from 表名 where 字段名=值;`

* truncate：truncate 相当于删除表的结构，再创建一张表。

## 4. DCL：数据控制语言

### 4.1 创建用户

`create user '用户名'@'主机名' identified by '密码';`

* 用户名：将创建的用户名
* 主机名：指定该用户在哪个主机上可以登陆，如果是本地用户可用 localhost，如果想让该用户可以从任意远程主机登陆，可以使用**通配符%**
* 密码：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器

### 4.2 用户授权

`grant 权限1, 权限2, ... on 数据库名.表名 to '用户名'@'主机名';`

* `grant...on...to`： 授权关键字
* 权限：授予用户的权限，如 `CREATE`、 `ALTER`、 `SELECT`、 `INSERT`、 `UPDATE` 等。如果要授予所有的权限则使用 `ALL`
* 数据库名.表名：用户可以操作哪个数据库的哪些表。如果要授予该用户对所有数据库和表的相应操作权限则可用`*`表示，如`*.*`
* '用户名'@'主机名'：给哪个用户授权，注：有 2 对单引号

### 4.3 撤销授权

`revoke 权限1, 权限2, ... on 数据库.表名 from '用户名'@'主机名';`

* `revoke ... on ... from`：撤销授权的关键字

> 撤销 user1 用户对 test 数据库所有表的操作的权限：
>
> `revoke all on test.* from 'user1'@'localhost';  `

### 4.4 查看权限

`show grants for '用户名'@'主机名';`

### 4.5 删除用户

`drop user '用户名'@'主机名';`

### 4.6 修改密码

* 修改管理员密码：

  `mysqladmin -uroot -p password 新密码`

  > 需要在未登陆 MySQL 的情况下操作，新密码不需要加上引号  

* 修改普通用户密码

  `set password for'用户名'@'主机名'=password('新密码'); `

  > 需要在登陆 MySQL 的情况下操作，新密码要加单引号



