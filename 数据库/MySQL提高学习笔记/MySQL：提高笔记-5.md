# MySQL：提高笔记-5

> 之前学了一波 MySQL 在使用层面的内容，现在则将其进行一步升华，稍稍深入的学习一些 MySQL 的内部实现等进阶知识点，这当然也是跟着视频学习时做的笔记，课程地址：[MySQL入门到精通](https://www.bilibili.com/video/BV1zJ411M7TB)，笔记总共分为5篇，此为 **5/5**。

## 1. MySQL中常用工具

### 1.1 MySQL

该MySQL不是指MySQL服务，而是指MySQL的客户端工具。

语法 ：

```
mysql [options] [database]
```

#### 1.1.1 连接选项

```mysql
-- 参数 ： 
	-u, --user=name			# 指定用户名
	-p, --password[=name]	# 指定密码
	-h, --host=name			# 指定服务器IP或域名
	-P, --port=#			# 指定连接端口

-- 示例 ：
	mysql -h 127.0.0.1 -P 3306 -u root -p
	
	mysql -h127.0.0.1 -P3306 -uroot -proot
```

#### 1.1.2 执行选项

```mysql
-e, --execute=name		# 执行SQL语句并退出
```

此选项可以在 MySQL 客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。

```mssql
-- 示例：
C:\Users\ZhuCC>mysql -uroot -proot db1 -e "select * from emp";
+----+--------+-----+--------+
| id | name   | age | salary |
+----+--------+-----+--------+
|  1 | Tom    |  25 |   2300 |
|  2 | Jerry  |  30 |   3500 |
|  3 | Luci   |  25 |   2800 |
|  4 | Jay    |  36 |   3500 |
|  5 | Tom2   |  21 |   2200 |
|  6 | Jerry2 |  31 |   3300 |
|  7 | Luci2  |  26 |   2700 |
|  8 | Jay2   |  33 |   3500 |
|  9 | Tom3   |  23 |   2400 |
| 10 | Jerry3 |  32 |   3100 |
| 11 | Luci3  |  26 |   2900 |
| 12 | Jay3   |  37 |   4500 |
+----+--------+-----+--------+ 
```

### 1.2 mysqladmin

mysqladmin 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

可以通过 ： `mysqladmin --help`  指令查看帮助文档 

示例 ：

```mysql
C:\Users\ZhuCC>mysqladmin -uroot -proot create 'test01';

C:\Users\ZhuCC>mysqladmin -uroot -proot drop 'test01';
Dropping the database is potentially a very bad thing to do.
Any data stored in the database will be destroyed.

Do you really want to drop the ''test01';' database [y/N] y
Database "'test01';" dropped

C:\Users\ZhuCC>mysqladmin -uroot -proot -V;
mysqladmin  Ver 8.42 Distrib 5.5.40, for Win32 on x86
```

### 1.3 mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到 **mysqlbinlog 日志管理工具**。

```mysql
-- 语法:
mysqlbinlog [options]  log-files1 log-files2 ...

-- 选项：
	
	-d, --database=name : 指定数据库名称，只列出指定的数据库相关操作。
	
	-o, --offset=# : 忽略掉日志中的前n行命令。
	
	-r,--result-file=name : 将输出的文本格式日志输出到指定文件。
	
	-s, --short-form : 显示简单格式， 省略掉一些信息。
	
	--start-datatime=date1  --stop-datetime=date2 : 指定日期间隔内的所有日志。
	
	--start-position=pos1 --stop-position=pos2 : 指定位置间隔内的所有日志。
```

### 1.4 mysqldump

mysqldump 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句。

```mysql
-- 语法：
mysqldump [options] db_name [tables]

mysqldump [options] --database/-B db1 [db2 db3...]

mysqldump [options] --all-databases/-A
```

#### 1.4.1 连接选项

```mysql
-- 参数 ： 
	-u, --user=name			# 指定用户名
	-p, --password[=name]	# 指定密码
	-h, --host=name			# 指定服务器IP或域名
	-P, --port=#			# 指定连接端口
```

#### 1.4.2 输出内容选项

```mysql
-- 参数：
	--add-drop-database		# 在每个数据库创建语句前加上 Drop database 语句
	--add-drop-table		# 在每个表创建语句前加上 Drop table 语句 , 默认开启 ; 
							# 若选择不开启 (--skip-add-drop-table)
	
	-n, --no-create-db		# 不包含数据库的创建语句
	-t, --no-create-info	# 不包含数据表的创建语句
	-d, --no-data			# 不包含数据
	
	-T, --tab=name			# 自动生成两个文件：一个.sql文件，创建表结构的语句；
	 						# 一个.txt文件，数据文件，相当于select into outfile  
	 						
	 						
-- 示例 ： 
	-- 会在相应的文件夹下生成a.sql文件
	mysqldump -uroot -proot db01 tb_book --add-drop-database --add-drop-table > a.sql
	
	-- 会在指定的文件夹下生成.sql 和.txt 文件
	mysqldump -uroot -proot -T /tmp db1 tb_book
```

### 1.5 mysqlimport/source

mysqlimport 是客户端数据导入工具，用来导入 mysqldump 加 -T 参数后导出的文本文件。

```mysql
-- 语法：
mysqlimport [options]  db_name  textfile1  [textfile2...]

-- 示例：
mysqlimport -uroot -proot db1 /tmp/tb_book.txt

-- 如果需要导入sql文件,可以使用mysql中的source 指令 : 
source /root/tb_book.sql
```

### 1.6 mysqlshow

mysqlshow 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

```mysql
-- 语法：
mysqlshow [options] [db_name [table_name [col_name]]]

-- 参数：
    --count		显示数据库及表的统计信息（数据库，表 均可以不指定）
    
    -i			显示指定数据库或者指定表的状态信息
    
-- 示例：
#查询每个数据库的表的数量及表中记录的数量
C:\Users\ZhuCC>mysqlshow -uroot -proot --count
+--------------------+--------+--------------+
|     Databases      | Tables |  Total Rows  |
+--------------------+--------+--------------+
| information_schema |     40 |         6122 |
| db1                |      6 |           32 |
| demo_01            |     14 |           60 |
| demo_03            |      3 |           14 |
| mysql              |     24 |         2080 |
| performance_schema |     17 |           14 |
| test               |      0 |            0 |
+--------------------+--------+--------------+
7 rows in set.

#查询test库中每个表中的字段书，及行数
mysqlshow -uroot -proot test --count

#查询test库中book表的详细情况
mysqlshow -uroot -proot test book --count    
```

## 2. MySQL日志

在任何一种数据库中，都会有各种各样的日志，记录着数据库工作的方方面面，以帮助数据库管理员追踪数据库曾经发生过的各种事件。MySQL 也不例外，在 MySQL 中，有 4 种不同的日志，分别是**错误日志**、**二进制日志（BINLOG 日志）**、**查询日志**和**慢查询日志**，这些日志记录着数据库在不同方面的踪迹。

### 2.1 错误日志

错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障导致无法正常使用时，可以首先查看此日志。

该日志是默认开启的 ， 默认存放目录为 mysql 的数据目录（var/lib/mysql）, 默认的日志文件名为 hostname.err（hostname是主机名）。

查看日志位置指令 ： 

```mysql
mysql> show variables like 'log_error%';
+---------------+----------------------------------------------------------------+
| Variable_name | Value                                                          |
+---------------+----------------------------------------------------------------+
| log_error     | C:\ProgramData\MySQL\MySQL Server 5.5\Data\LAPTOP-F6OAPAAB.err |
+---------------+----------------------------------------------------------------+
1 row in set (0.00 sec)
```

查看日志内容 ： 

```shell
tail -f /var/lib/mysql/xaxh-server.err

# ...
210226 18:22:02 [Note] Event Scheduler: Loaded 0 events
210226 18:22:02 [Note] C:\Software\MySQL\MySQL Server 5.5\bin\mysqld: ready for connections.
Version: '5.5.40'  socket: ''  port: 3306  MySQL Community Server (GPL)
# ... 
```

### 2.2 二进制日志

#### 2.2.1 概述

**二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但是不包括数据查询语句。**此日志对于灾难时的数据恢复起着极其重要的作用，**MySQL 的主从复制， 就是通过该binlog实现的。**

二进制日志，默认情况下是没有开启的，需要到 MySQL 的配置文件中开启，并配置 MySQL 日志的格式。 

```mysql
mysql> show variables like "log_%";  -- 查看日志情况，可以看到log_bin默认是没有开启的
+---------------------------------+----------------------------------------------------------------+
| Variable_name                   | Value                                                          |
+---------------------------------+----------------------------------------------------------------+
| log_bin                         | OFF                                                            |
| log_bin_trust_function_creators | OFF                                                            |
| log_error                       | C:\ProgramData\MySQL\MySQL Server 5.5\Data\LAPTOP-F6OAPAAB.err |
| log_output                      | FILE                                                           |
| log_queries_not_using_indexes   | OFF                                                            |
| log_slave_updates               | OFF                                                            |
| log_slow_queries                | OFF                                                            |
| log_warnings                    | 1                                                              |
+---------------------------------+----------------------------------------------------------------+
8 rows in set (0.00 sec)
```

配置文件位置 : 

* Linux：`/usr/my.cnf`

* windows：MySQL安装目录下的my.ini，通过命令：`select @@basedir`可查看安装MySQL的本机目录

日志存放位置：配置时，给定了文件名但是没有指定路径，日志默认写入 MySQL 的数据目录，通过以下命令查看：

* `show variables like 'datadir';`
* `select @@datadir;`

```mysql
# 1. 找到MySQL的安装位置，打开my.ini文件
# 2，找到[mysqld]栏

# 3.配置开启binlog日志， 
# 日志的文件前缀为mysqlbin -----> 生成的文件名如:mysqlbin.000001, mysqlbin.000002
log_bin=mysqlbin  # 这里没有给出具体路径，因此文件存放在show variables like 'datadir';这个路径下
# 配置二进制日志的格式
binlog_format=STATEMENT

# 4.重启mysql服务，再次查看日志打开情况
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show binary logs;
+-----------------+-----------+
| Log_name        | File_size |
+-----------------+-----------+
| mysqlbin.000001 |       341 |
+-----------------+-----------+
1 row in set (0.01 sec)
```

#### 2.2.2 日志格式

**STATEMENT**

该日志格式在日志文件中记录的都是SQL语句（statement），**每一条对数据进行修改的SQL都会记录在日志文件中**，通过MySQL提供的mysqlbinlog工具，可以清晰的查看到每条语句的文本。主从复制的时候，从库（slave）会将日志解析为原文本，并在从库重新执行一次。

**ROW**

该日志格式在日志文件中记录的是**每一行的数据变更，而不是记录SQL语句**。比如，执行SQL语句 ： update tb_book set status='1' , 如果是 STATEMENT 日志格式，在日志中会记录一行 SQL 语句； 如果是 ROW，由于是对全表进行更新，也就是每一行记录都会发生变更，ROW 格式的日志中会记录每一行的数据变更。

**MIXED**

这是目前MySQL默认的日志格式，即**混合了 STATEMENT 和 ROW 两种格式**。默认情况下采用 STATEMENT，但是在一些特殊情况下采用ROW 来进行记录。MIXED 格式能尽量利用两种模式的优点，而避开他们的缺点。

#### 2.2.3 日志读取

由于日志以二进制方式存储，不能直接读取，需要用 mysqlbinlog 工具来查看，语法如下 ：

```mysql
mysqlbinlog log-file；
```

**查看STATEMENT格式日志** 

执行插入语句 ：

```SQL
mysql> insert into emp values (null, "Jackbin", 33, 6666);
Query OK, 1 row affected (0.04 sec)
```

 查看日志文件 ： 

`mysqlbin.index` : 该文件是日志索引文件 ， 记录日志的文件名；

`mysqlbing.000001` ：日志文件

查看日志内容 ：

```mysql
C:\Software\MySQL\MySQL Server 5.5\bin>mysqlbinlog ../../mysqlbin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#210228 15:13:12 server id 1  end_log_pos 107   Start: binlog v 4, server v 5.5.40-log created 210228 15:13:12 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
iEI7YA8BAAAAZwAAAGsAAAABAAQANS41LjQwLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACIQjtgEzgNAAgAEgAEBAQEEgAAVAAEGggAAAAICAgCAA==
'/*!*/;
# at 107
#210228 15:15:36 server id 1  end_log_pos 174   Query   thread_id=1     exec_time=0     error_code=0
SET TIMESTAMP=1614496536/*!*/;
SET @@session.pseudo_thread_id=1/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1344274432/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=33/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 174
#210228 15:15:36 server id 1  end_log_pos 202   Intvar
SET INSERT_ID=13/*!*/;
# at 202
#210228 15:15:36 server id 1  end_log_pos 314   Query   thread_id=1     exec_time=0     error_code=0
use `db1`/*!*/;
SET TIMESTAMP=1614496536/*!*/;
insert into emp values (null, "Jackbin", 33, 6666)  -- !!!!statement：对表的插入语句被记录下来了
/*!*/;
# at 314
#210228 15:15:36 server id 1  end_log_pos 341   Xid = 9
COMMIT/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

**查看 ROW 格式日志**

配置 :

```mysql
# 配置开启binlog日志
# 日志的文件前缀为 mysqlbin -----> 生成的文件名如 : mysqlbin.000001,mysqlbin.000002
log_bin=mysqlbin

# 配置二进制日志的格式
binlog_format=ROW
# 重启服务器
```

插入数据 :

```sql
mysql> insert into emp values (null, "Jackrow", 44, 8888);
Query OK, 1 row affected (0.01 sec)
```

如果日志格式是 ROW , 直接查看数据 , 是查看不懂的 ; 可以在mysqlbinlog 后面加上参数 -vv  

```SQL
mysqlbinlog -vv mysqlbin.000002 

C:\Software\MySQL\MySQL Server 5.5\bin>mysqlbinlog -vv ../../mysqlbin.000002

-- ...

BINLOG '
60g7YBMBAAAALgAAANwAAAAAACEAAAAAAAEAA2RiMQADZW1wAAQDDwMDApABCA==
60g7YBcBAAAAMwAAAA8BAAAAACEAAAAAAAEABP/wDgAAAAcASmFja3JvdywAAAC4IgAA
'/*!*/;
### INSERT INTO `db1`.`emp`  -- !!!row:这里记录了插入的信息
### SET
###   @1=14 /* INT meta=0 nullable=0 is_null=0 */   -- !!!ID号
###   @2='Jackrow' /* VARSTRING(400) meta=400 nullable=0 is_null=0 */  -- !!!名字
###   @3=44 /* INT meta=0 nullable=0 is_null=0 */  -- !!!age
###   @4=8888 /* INT meta=0 nullable=1 is_null=0 */  -- !!!salary
# at 271
#210228 15:40:27 server id 1  end_log_pos 298   Xid = 5
COMMIT/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/; 
```

#### 2.2.4 日志删除

对于比较繁忙的系统，由于每天生成日志量大 ，这些日志如果长时间不清除，将会占用大量的磁盘空间。下面我们将会讲解几种删除日志的常见方法 ：

**方式一** 

通过 Reset Master 指令删除全部 binlog 日志，删除之后，日志编号，将从 `xxxx.000001`重新开始 。

查询之前 ，先查询下日志文件：`mysqlbin.000001`，`mysqlbin.000002`  

执行删除日志指令： 

```mysql
mysql> Reset Master;
Query OK, 0 rows affected (0.06 sec)
```

执行之后， 查看日志文件 ：`mysqlbin.000001` 

**方式二**

执行指令` purge master logs to 'mysqlbin.******'` ，该命令将删除  ` ******` 编号之前的所有日志。 

**方式三**

执行指令 ` purge master logs before 'yyyy-mm-dd hh24:mi:ss'` ，该命令将删除日志为 `"yyyy-mm-dd hh24:mi:ss"` 之前产生的所有日志 。

**方式四**

设置参数 `--expire_logs_days=#` ，此参数的含义是设置日志的过期天数， 过了指定的天数后日志将会被自动删除，这样将有利于减少DBA 管理日志的工作量。

配置如下 ： 

```mysql
log_bin=mysqlbin
binlog_format=ROW
--expire_logs_days=3
```

### 2.3 查询日志

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。

默认情况下， 查询日志是未开启的。如果需要开启查询日志，可以设置以下配置 ：

```mysql
mysql> show variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

# 在my.ini[mysql]标签下修改
# 该选项用来开启查询日志 ， 可选值:0/1:  0代表关闭  1代表开启 
general_log=1
# 设置日志的文件名,如果没有指定,默认的文件名为 host_name.log 
general_log_file=file_name.log
# 配置完成后重启mysql服务
# 查看是否开启
mysql> show variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

在 mysql 的配置文件 my.cnf 中配置如下内容 ： 

```bash
[mysqld]

# binlog配置
log_bin=mysqlbin.log
binlog_format=ROW	
# querylog配置
general_log=1
general_log_file=mysql_query.log
```

配置完毕之后，在数据库执行以下操作 ：

```mysql
mysql> use db1;
Database changed
mysql> select * from emp;
+----+---------+-----+--------+
| id | name    | age | salary |
+----+---------+-----+--------+
|  1 | Tom     |  25 |   2300 |
|  2 | Jerry   |  30 |   3500 |
|  3 | Luci    |  25 |   2800 |
|  4 | Jay     |  36 |   3500 |
|  5 | Tom2    |  21 |   2200 |
|  6 | Jerry2  |  31 |   3300 |
|  7 | Luci2   |  26 |   2700 |
|  8 | Jay2    |  33 |   3500 |
|  9 | Tom3    |  23 |   2400 |
| 10 | Jerry3  |  32 |   3100 |
| 11 | Luci3   |  26 |   2900 |
| 12 | Jay3    |  37 |   4500 |
| 13 | Jackbin |  33 |   6666 |
| 14 | Jackrow |  44 |   8888 |
+----+---------+-----+--------+
14 rows in set (0.01 sec)

mysql> select * from emp where id=1;
+----+------+-----+--------+
| id | name | age | salary |
+----+------+-----+--------+
|  1 | Tom  |  25 |   2300 |
+----+------+-----+--------+
1 row in set (0.00 sec)

mysql> update emp set name='Jay333'where id=12;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from emp where id < 8;
+----+--------+-----+--------+
| id | name   | age | salary |
+----+--------+-----+--------+
|  1 | Tom    |  25 |   2300 |
|  2 | Jerry  |  30 |   3500 |
|  3 | Luci   |  25 |   2800 |
|  4 | Jay    |  36 |   3500 |
|  5 | Tom2   |  21 |   2200 |
|  6 | Jerry2 |  31 |   3300 |
|  7 | Luci2  |  26 |   2700 |
+----+--------+-----+--------+
7 rows in set (0.01 sec)
```

执行完毕之后， 再次来查询日志文件 ： 

```bash
C:\Software\MySQL\MySQL Server 5.5\bin\mysqld, Version: 5.5.40-log (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: (null)
Time                 Id Command    Argument
210228 15:52:47	    1 Connect	root@localhost on 
		    1 Query	select @@version_comment limit 1
210228 15:52:51	    1 Query	show variables like 'general_log'
210228 15:54:02	    1 Query	SELECT DATABASE()
		    1 Init DB	db1
210228 15:54:06	    1 Query	select * from emp
210228 15:54:19	    1 Query	select * from emp where id=1
210228 15:54:54	    1 Query	update emp set name='Jay333'where id=12
210228 15:55:08	    1 Query	select * from emp where id < 8
```

### 2.4 慢查询日志

慢查询日志记录了所有**执行时间超过参数 long_query_time** 设置值并且**扫描记录数不小于 min_examined_row_limit** 的所有的SQL语句的日志。long_query_time 默认为 10 秒，最小为 0， 精度可以到微秒。

#### 2.4.1 文件位置和格式

慢查询日志默认是关闭的 。可以通过两个参数来控制慢查询日志 ：

```mysql
mysql> show variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | OFF   |
+----------------+-------+
1 row in set (0.00 sec)

# 在mysql 的配置文件my.ini下修改后重启mysql服务
# 该参数用来控制慢查询日志是否开启,可取值:1/0: 1 代表开启  0 代表关闭
slow_query_log=1 
# 该参数用来指定慢查询日志的文件名
slow_query_log_file=slow_query.log
# 该选项用来配置查询的时间限制， 超过这个时间将认为值慢查询， 将需要进行日志记录， 默认10s
long_query_time=0.01
```

#### 2.4.2 日志的读取

和错误日志、查询日志一样，慢查询日志记录的格式也是纯文本，可以被直接读取。

1. 查询long_query_time 的值。

```mysql
mysql> show variables like 'long%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec) 
```

2. 执行查询操作

```sql
mysql> select * from emp;
+----+---------+-----+--------+
| id | name    | age | salary |
+----+---------+-----+--------+
|  1 | Tom     |  25 |   2300 |
|  2 | Jerry   |  30 |   3500 |
|  3 | Luci    |  25 |   2800 |
|  4 | Jay     |  36 |   3500 |
|  5 | Tom2    |  21 |   2200 |
|  6 | Jerry2  |  31 |   3300 |
|  7 | Luci2   |  26 |   2700 |
|  8 | Jay2    |  33 |   3500 |
|  9 | Tom3    |  23 |   2400 |
| 10 | Jerry3  |  32 |   3100 |
| 11 | Luci3   |  26 |   2900 |
| 12 | Jay333  |  37 |   4500 |
| 13 | Jackbin |  33 |   6666 |
| 14 | Jackrow |  44 |   8888 |
+----+---------+-----+--------+
14 rows in set (0.00 sec)
```

由于该语句执行时间很短，为0.00s ， 所以不会记录在慢查询日志中。

```mysql
mysql> select * from emp e1, emp e2, emp e3, emp e4;
```

该SQL语句 ， 执行时长为 0.05s ，超过0.01s ， 所以会记录在慢查询日志文件中

```bash
C:\Software\MySQL\MySQL Server 5.5\bin\mysqld, Version: 5.5.40-log (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: (null)
Time                 Id Command    Argument
# Time: 210228 16:04:56
# User@Host: root[root] @ localhost [127.0.0.1]
# Query_time: 0.057034  Lock_time: 0.001003 Rows_sent: 38416  Rows_examined: 14
use db1;
SET timestamp=1614499496;
select * from emp e1, emp e2, emp e3, emp e4;
```

3. 查看慢查询日志文件

直接通过cat 指令查询该日志文件 / 直接打开看

如果慢查询日志内容很多， 直接查看文件，比较麻烦， 这个时候可以借助于 mysql 自带的 mysqldumpslow 工具， 来对慢查询日志进行分类汇总。 

```
mysqldumpslow xxx.log
```

> windows上好像没这个工具，没有尝试

## 3. MySQL复制

### 3.1 复制概述

复制是指将主数据库的 DDL 和 DML 操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步。

MySQL 支持一台主库同时向多台从库进行复制，从库同时也可以作为其他从服务器的主库，实现链状复制。

### 3.2 复制原理

MySQL 的主从复制原理如下。

![主从复制](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251655096.jpg) 

从上层来看，复制分成三步：

- Master 主库在事务提交时，会把数据变更作为时间 Events 记录在二进制日志文件 Binlog 中。
- 主库推送二进制日志文件 Binlog 中的日志事件到从库的中继日志 Relay Log 。

- Slave 重做中继日志中的事件，将改变反映它自己的数据。

### 3.3 复制优势

MySQL 复制的有点主要包含以下三个方面：

- 主库出现问题，可以快速切换到从库提供服务。

- 可以在从库上执行查询操作，从主库中更新，实现读写分离，降低主库的访问压力。

- 可以在从库中执行备份，以避免备份期间影响主库的服务。

### 3.4 搭建步骤

> 这部分内容我没有尝试搭建，后续有机会再尝试尝试吧

#### 3.4.1 master

1. 在master 的配置文件（/usr/my.cnf）中，配置如下内容：

```properties
# mysql 服务ID,保证整个集群环境中唯一
server-id=1

# mysql binlog 日志的存储路径和文件名
log-bin=/var/lib/mysql/mysqlbin

# 错误日志,默认已经开启
# log-err

# mysql的安装目录
# basedir

# mysql的临时目录
# tmpdir

# mysql的数据存放目录
# datadir

# 是否只读,1 代表只读, 0 代表读写
read-only=0

# 忽略的数据, 指不需要同步的数据库
binlog-ignore-db=mysql

# 指定同步的数据库
#binlog-do-db=db01
```

2. 执行完毕之后，需要重启Mysql：

```mysql
service mysql restart;
```

3. 创建同步数据的账户，并且进行授权操作：

```mysql
grant replication slave on *.* to 'itcast'@'192.168.192.131' identified by 'itcast';	

flush privileges;
```

4. 查看master状态：

```sql
mysql> show master status;
+-----------------+----------+--------------+------------------+
| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-----------------+----------+--------------+------------------+
| mysqlbin.000001 |      413 |              |             mysql|
+-----------------+----------+--------------+------------------+
1 row in set (0.00 sec) 
```

字段含义：

* File : 从哪个日志文件开始推送日志文件 
* Position ： 从哪个位置开始推送日志
* Binlog_Ignore_DB : 指定不需要同步的数据库

#### 3.4.2 slave

1. 在 slave 端配置文件中，配置如下内容：

```properties
# mysql服务端ID,唯一
server-id=2

# 指定binlog日志
log-bin=/var/lib/mysql/mysqlbin
```

2. 执行完毕之后，需要重启MySQL：

```
service mysql restart；
```

3. 执行如下指令 ：

```sql
change master to master_host= '192.168.192.130', master_user='itcast', master_password='itcast', master_log_file='mysqlbin.000001', master_log_pos=413;
```

指定当前从库对应的主库的IP地址，用户名，密码，从哪个日志文件开始的那个位置开始同步推送日志。

4. 开启同步操作

```mysql
start slave;

show slave status;

**************** 1.row **************** 
Slave_IO_State: Waiting for master to send event
Master_host: 192.168.192.130
Master_User: itcast
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: mysqlbin.000001
Read_Master_Log_Pos: 413
Relay_Log_File: localhost-relay-bin.000003
Relay_Master_Log_File: mysqlbin.000001
Slave_IO_Running: Yes
Slave_SQL_Runing: Yes 
```

5. 停止同步操作

```mysql
stop slave;
```

#### 3.4.3 验证同步操作

1. 在主库中创建数据库，创建表，并插入数据 ：

```sql
create database db01;

user db01;

create table user(
	id int(11) not null auto_increment,
	name varchar(50) not null,
	sex varchar(1),
	primary key (id)
)engine=innodb default charset=utf8;

insert into user(id,name,sex) values(null,'Tom','1');
insert into user(id,name,sex) values(null,'Trigger','0');
insert into user(id,name,sex) values(null,'Dawn','1');
```

2. 在从库中查询数据，进行验证 ：

```mysql
-- 在从库中，可以查看到刚才创建的数据库：
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
-- ...
| db01               |
-- ...
+--------------------+
7 rows in set (0.01 sec)
```

在该数据库中，查询user表中的数据：

```mysql
mysql> select * from user;
+----+---------+-----+
| id | name    | sex |
+----+---------+-----+
|  1 | Tom     |  1  |
|  2 | Trigger |  0  |
|  3 | Dawn    |  1  |
```



