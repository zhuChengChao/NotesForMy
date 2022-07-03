# MySQL：补充知识

> 在学习完 MySQL 基础与提高内容后，本文主要是补充一些上述笔记中未记录的知识点。
>
> 之前学了MySQL的基础知识与提高部分知识后，后续抽空在牛客网刷题时遇到了一些新的内容，故在此做些个人记录。

## 导入数据到表中

**方式1**：通过`insert into table`方式

**方式2：**通过加载文件的方式将数据导入到表中；

0. 创建的表为：

```sql
CREATE TABLE pet(
	NAME VARCHAR(20),
	OWNER VARCHAR(20),
	species VARCHAR(20),
	sex CHAR(1),
	birth DATE,
	death DATE
);
```

1. 创建一个txt文件（注：每个字段中用tab键隔开，字段没有值得记录用\N代替）；

```
Fluffy	Harold	cat	f	1993‐02‐04
Claws	Gwen	cat	m	1994‐03‐17
Buffy	Harold	dog	f	1989‐05‐13
Fang	Benny	dog	m	1990‐08‐27
Bowser	Diane	dog	m	1979‐08‐31	1995‐07‐29
Chirpy	Gwen	bird	f	1998‐09‐11
Whistler	Gwen	bird	\N	1997‐12‐09	\N
Slim	Benny	snake	m	1996‐04‐29
```

2. 加载数据

```mysql
mysql> load data local infile 'txt文件路径' into table 表名字

mysql> load data local infile 'C:\\Users\\ZhuCC\\Desktop\\pet.txt' into table pet;
Query OK, 8 rows affected, 7 warnings (0.00 sec)
Records: 8  Deleted: 0  Skipped: 0  Warnings: 7
```

3. 校验是否导入成功

```mysql
mysql> select * from pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | 0000-00-00 |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
+----------+--------+---------+------+------------+------------+
8 rows in set (0.00 sec)
```

## 全外连接

全外连接：左外连接和右外连接的结果合并，但会去掉重复的记录。 

```mysql
select * from 表1 full outer join 表2 on 条件
```

> 但是mysql数据库不支持此语法

## IN 和 EXISTS

**IN：**在 MySQL 基础中有讲解；

**EXISTS：**表示存在，当子查询的结果存在，就会显示主查询中的所有数据 

```sql
SELECT * FROM tab_route r WHERE EXISTS (SELECT * FROM tab_category c WHERE c.`cid` = r.`cid` AND c.`cname`="国内游");
```

> 参考：https://www.cnblogs.com/dreamtecher/p/5128668.html
>
> 牛客网中的SQL题：https://www.nowcoder.com/questionTerminal/c39cbfbd111a4d92b221acec1c7c1484

## UNION 和 UNION ALL

**UNION 语句：**用于将不同表中相同列中查询的数据展示出来；（**不包括重复数据**） 

```mysql
mysql> SELECT * FROM table_a;
+------+-------+
| id   | name  |
+------+-------+
|    1 | one   |
|    2 | two   |
|    3 | three |
|    4 | four  |
|    5 | five  |
+------+-------+

mysql> SELECT * FROM table_b;
+------+-------+
| id   | NAME  |
+------+-------+
|    4 | four  |
|    5 | five  |
|    6 | six   |
|    7 | seven |
|    8 | eight |
+------+-------+

mysql> SELECT * FROM table_a UNION SELECT * FROM table_b;
+------+-------+
| id   | name  |
+------+-------+
|    1 | one   |
|    2 | two   |
|    3 | three |
|    4 | four  |
|    5 | five  |
|    6 | six   |
|    7 | seven |
|    8 | eight |
+------+-------+
```

**UNION ALL 语句：**用于将不同表中相同列中查询的数据展示出来；（**包括重复数据**） 

```mysql
mysql> SELECT * FROM table_a UNION ALL SELECT * FROM table_b;
+------+-------+
| id   | name  |
+------+-------+
|    1 | one   |
|    2 | two   |
|    3 | three |
|    4 | four  |     -- 重复数据
|    5 | five  |
|    4 | four  |     -- 重复数据
|    5 | five  |
|    6 | six   |
|    7 | seven |
|    8 | eight |
+------+-------+
```

## CASE WHEN 语句

```mysql
SELECT *,
	CASE <表达式>
	WHEN <表达式值> THEN <sql语句或结果>
	WHEN <表达式值> THEN <sql语句或结果>
	...
	ELSE <默认情况下的sql语句或结果>
END

-- 说明：
--    case后面紧跟要被作为判断的字段
--    when后面跟判断条件
--    then后面跟结果
--    else相当于default
--    end是语句结束语

-- 举例：
SELECT *,
	CASE 
	WHEN a.`id`=1 THEN "1=one"
	WHEN a.`id`=2 THEN "2=two"
	WHEN a.`id`=3 THEN "3=three"
	WHEN a.`id`=4 THEN "4=four"
	ELSE "None" END AS "type"
FROM table_a AS a;
```

## IF 和 CASE 语句

MySQL 的 if 既可以作为表达式用，也可在存储过程中作为流程控制语句使用

```mysql
-- 表达式：
IF(expr1, expr2, expr3)

-- 如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL)，则IF()的返回值为expr2; 
-- 否则返回值则为 expr3;
-- IF() 的返回值为数字值或字符串值，具体情况视其所在语境而定。

-- 示例：
select *, if(sva=1,"男", "女") as ssva from taname where sva != "";
```

表达式的if也可以用CASE when来实现：

```mysql
select CASE sva WHEN 1 THEN '男' ELSE '女' END as ssva from taname where sva != '';

-- 上述返回结果中
-- 如果没有匹配的结果值，则返回结果为ELSE后的结果
-- 如果没有ELSE 部分，则返回值为 NULL
SELECT CASE 1 WHEN 1 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END as testCol;  -- 输出one
SELECT CASE 1 WHEN 2 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END as testCol;  -- 输出two
SELECT CASE 1 WHEN 3 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END as testCol;  -- 输出more
SELECT CASE 1 WHEN 3 THEN 'one' WHEN 2 THEN 'two' END as testCol;  -- 输出null
```

## 获取服务器元数据

| 命令               | 描述                      | 结果                       |
| ------------------ | ------------------------- | -------------------------- |
| SELECT VERSION();  | 服务器版本信息            | version()<br />5.5.40      |
| SELECT DATABASE(); | 当前数据库名 (或者返回空) | DATABASE()<br />db1        |
| SELECT USER();     | 当前用户名                | USER()<br />root@localhost |
| SHOW STATUS;       | 服务器状态                | 略                         |
| SHOW VARIABLES;    | 服务器配置变量            | 略                         |

## 内置函数

### 字符串函数

| 函数                                  | 描述                                                         | 实例                                                         |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ASCII(s)                              | 返回字符串 s 的第一个字符的 ASCII 码                         | `SELECT ASCII("abc");`                                       |
| CHAR_LENGTH(s)                        | 返回字符串 s 的字符 数                                       | `SELECT CHAR_LENGTH("abc")`                                  |
| CHARACTER_LENGTH(s)                   | 返回字符串 s 的字符 数                                       | `SELECT CONCAT("a", "b", "c");`                              |
| CONCAT(s1,s2...sn)                    | 字符串 s1,s2 等多个 字符串合并为一个字 符串                  | `SELECT CONCAT("a", "b", "c");`                              |
| CONCAT_WS(x, s1,s2...sn)              | 同 CO NCAT(s1,s2,...) 函数，但是每个字符 串直接要加上 x，x 可以是分隔符 | `SELECT CONCAT_WS("," , "a", "b", "c");`                     |
| FIELD(s,s1,s2...)                     | 返回第一个字符串 s 在字符串列表 (s1,s2...)中的位置           | `SELECT FIELD("c", "a", "b", "c", "d", "e");`                |
| FIND_IN_SET(s1,s2)                    | 返回在字符串s2中与 s1匹配的字符串的位 置                     | `SELECT FIND_IN_SET("c", "a,b,c,d,e");`                      |
| FORMAT(x,n)                           | 函数可以将数字 x 进 行格式化 "#,###.##", 将 x 保 留到小数点后 n 位，最后一位四舍五入 | `SELECT FORMAT(250500.5634, 3);`                             |
| INSERT(s1,x,len,s2)                   | 字符串 s2 替换 s1 的 x 位置开始长度为 len 的字符串           | `SELECT INSERT("google.com", 1, 6, "runnob");`               |
| LOCATE(s1,s)                          | 从字符串 s 中获取 s1 的开始位置                              | `SELECT INSTR('abc','b');`                                   |
| LCASE(s)                              | 将字符串 s 的所有字 母变成小写字母                           | `SELECT LOWER('ABC');`                                       |
| LEFT(s,n)                             | 返回字符串 s 的前 n 个字符                                   | `SELECT LEFT('hello',2);`                                    |
| LOCATE(s1,s)                          | 从字符串 s 中获取 s1 的开始位置                              | `SELECT LOCATE('b', 'abc');`                                 |
| LOWER(s)                              | 将字符串 s 的所有字 母变成小写字母                           | `SELECT LOWER('ABC1234');`                                   |
| LPAD(s1,len,s2)                       | 在字符串 s1 的开始 处填充字符串 s2， 使字符串长度达到 len    | `SELECT LPAD('abc',10,'xx');`                                |
| LTRIM(s)                              | 去掉字符串 s 开始处 的空格                                   | `SELECT LTRIM("   hello");`                                  |
| MID(s,n,len)                          | 从字符串s的n位置截取长度为length的子字符串，同 SUBSTRING(s,n,len) | SELECT MID("zhuchengchao", 4, 5);<br/>SELECT SUBSTRING("zhuchengchao", 4, 5); |
| POSITION(s1 IN s)                     | 从字符串 s 中获取 s1 的开始位置                              | `SELECT POSITION('b' IN 'abc');`                             |
| REPEAT(s,n)                           | 将字符串 s 重复 n 次                                         | `SELECT REPEAT('hello',3);`                                  |
| REPLACE(s,s1,s2)                      | 将字符串 s2 替代字 符串 s 中的字符串 s1                      | `SELECT REPLACE('abcabc','a','x');`                          |
| REVERSE(s)                            | 将字符串s的顺序反 过来                                       | `SELECT REVERSE('abc');`                                     |
| RIGHT(s,n)                            | 返回字符串 s 的后 n 个字符                                   | `SELECT RIGHT('hello',2);`                                   |
| RPAD(s1,len,s2)                       | 在字符串 s1 的结尾 处添加字符串 s2， 使字符串的长度达到 len  | `SELECT RPAD('abc',10,'xx');`                                |
| RTRIM(s)                              | 去掉字符串 s 结尾处 的空格                                   | `SELECT RTRIM("hello   ");`                                  |
| SPACE(n)                              | 返回 n 个空格                                                | `SELECT CHAR_LENGTH(SPACE(10));`                             |
| STRCMP(s1,s2)                         | 比较字符串 s1 和 s2，如果 s1 与 s2 相等返回 0 ，如果 s1>s2 返回 1，如果 s1<s2 返回 -1 | `SELECT STRCMP("Hello", "hello");`                           |
| SUBSTR(s, start, length)              | 从字符串 s 的 start 位置截取长度为 length 的子字符串         | `SELECT SUBSTR("Hello", 2, 3);`                              |
| SUBSTRING(s, start, length)           | 从字符串 s 的 start 位置截取长度为 length 的子字符串         | `SELECT SUBSTRING("zhuchengchao", 4, 5);`                    |
| SUBSTRING_INDEX(s, delimiter, number) | 返回从字符串 s 的第 number 个出现的分 隔符 delimiter 之后 的子串<br />如果 number 是正数，返回第 number 个字符左边的字符串<br />如 果 number 是负 数，返回第(number 的绝对值(从右边数)) 个字符右边的字符串 | `SELECT SUBSTRING_INDEX('a*b*c','*', 2);`<br />`SELECT SUBSTRING_INDEX('a*b*c','*', -2);` |
| TRIM(s)                               | 去掉字符串 s 开始和 结尾处的空格                             | `SELECT TRIM('   hello   ');`                                |
| UCASE(s)                              | 将字符串转换为大写                                           | `SELECT UCASE("hello");`                                     |
| UPPER(s)                              | 将字符串转换为大写                                           | `SELECT UPPER("hello");`                                     |

### 数学函数

| 函数名                             | 描述                                                         | 实例                                       |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| ABS(x)                             | 返回 x 的绝对值                                              | `SELECT ABS(-1);`                          |
| ACOS(x)                            | 求 x 的反余弦值(参数 是弧度)                                 | `SELECT ACOS(0.25);`                       |
| ASIN(x)                            | 求反正弦值(参数是弧 度)                                      | `SELECT ASIN(0.25);`                       |
| ATAN(x)                            | 求反正切值(参数是弧 度)                                      | `SELECT ATAN(2.5);`                        |
| ATAN2(n, m)                        | 求反正切值(参数是弧 度)                                      | `SELECT ATAN2(-0.8, 2);`                   |
| AVG(expression)                    | 返回一个表达式的平均值，expression 是 一个字段               | `SELECT AVG(Price)  FROM Products;`        |
| CEIL(x)                            | 返回大于或等于 x 的 最小整数                                 | `SELECT CEIL(1.5) `                        |
| CEILING(x)                         | 返回大于或等于 x 的 最小整数                                 | `SELECT CEIL(1.5)`                         |
| COS(x)                             | 求余弦值(参数是弧度)                                         | `SELECT COS(2);`                           |
| COT(x)                             | 求余切值(参数是弧度)                                         | `SELECT COT(6);`                           |
| COUNT(expression)                  | 返回查询的记录总数，expression 参数 是一个字段或者 * 号      | `SELECT COUNT(ProductID)  FROM Products;`  |
| DEGREES(x)                         | 将弧度转换为角度                                             | `SELECT DEGREES(3.1415926535898);`         |
| n DIV m                            | 整除，n 为被除数，m 为除数                                   | `SELECT 10 DIV 5; `                        |
| EXP(x)                             | 返回 e 的 x 次方                                             | `SELECT EXP(3) `                           |
| FLOOR(x)                           | 返回小于或等于 x 的 最大整数                                 | `SELECT FLOOR(1.5)`                        |
| GREATEST(expr1, expr2, expr3, ...) | 返回列表中的最大值                                           | `SELECT GREATEST(3, 12, 34, 8, 25); `      |
| LEAST(expr1, expr2, expr3, ...)    | 返回列表中的最小值                                           | `SELECT LEAST("apple1", "apple2"); `       |
| LN                                 | 返回数字的自然对数                                           | `SELECT LN(2);`                            |
| LOG(x)                             | 返回自然对数(以 e 为 底的对数)                               | `SELECT LOG(20.085536923188)`              |
| LOG10(x)                           | 返回以 10 为底的对数                                         | `SELECT LOG10(100) `                       |
| LOG2(x)                            | 返回以 2 为底的对数                                          | ` SELECT LOG2(6); `                        |
| MAX(expression)                    | 返回字段 expression 中的最大值                               | `SELECT MAX(Price) FROM Products;`         |
| MIN(expression)                    | 返回字段 expression 中的最小值                               | `SELECT MIN(Price)  FROM Products;`        |
| MOD(x,y)                           | 返回 x 除以 y 以后的 余数                                    | `SELECT MOD(5,2) `                         |
| PI()                               | 返回圆周率 (3.141593）                                       | `SELECT PI() `                             |
| POW(x,y)                           | 返回 x 的 y 次方                                             | `SELECT POW(2,3)`                          |
| POWER(x,y)                         | 返回 x 的 y 次方                                             | `SELECT POWER(2,3)`                        |
| RADIANS(x)                         | 将角度转换为弧度                                             | `SELECT RADIANS(180) `                     |
| RAND()                             | 返回 0 到 1 的随机数                                         | `SELECT RAND() `                           |
| ROUND(x)                           | 返回离 x 最近的整数 ，四舍五入                               | `SELECT ROUND(1.23456) `                   |
| SIGN(x)                            | 返回 x 的符号，x 是负 数、0、正数分别返回 -1、0 和 1         | `SIGN(‐10) `                               |
| SIN(x)                             | 求正弦值(参数是弧度)                                         | `SELECT SIN(RADIANS(30)) `                 |
| SQRT(x)                            | 返回x的平方根                                                | `SELECT SQRT(25)`                          |
| SUM(expression)                    | 返回指定字段的总和                                           | `SELECT SUM(Quantity)  FROM OrderDetails;` |
| TAN(x)                             | 求正切值(参数是弧度)                                         | `SELECT TAN(1.75); `                       |
| TRUNCATE(x,y)                      | 返回数值 x 保留到小 数点后 y 位的值（与 ROUND 最大的区别是 不会进行四舍五入） | `SELECT TRUNCATE(1.23456,3) `              |

### 日期函数

| 函数名                            | 描述                                                         | 实例                                                         |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ADDDATE(d,n)                      | 计算其实日期 d 加上 n 天的日期                               | `SELECT ADDDATE("2020-06-19", 10);`                          |
| ADDTIME(t,n)                      | 时间 t 加上 n 秒的时间                                       | `SELECT ADDTIME(NOW(), 100);`                                |
| CURDATE()                         | 返回当前日期                                                 | 略                                                           |
| CURRENT_DATE()                    | 返回当前日期                                                 | 略                                                           |
| CURRENT_TIME                      | 返回当前时间                                                 | `SELECT CURRENT_TIME;`                                       |
| CURRENT_TIMESTAMP()               | 返回当前日期和时间                                           | `SELECT CURRENT_TIMESTAMP;`                                  |
| CURTIME()                         | 返回当前时间                                                 | 略                                                           |
| DATE()                            | 从日期或日期时间表达式中提取日期值                           | `SELECT DATE("2020,6,19");`                                  |
| DATEDIFF(d1,d2)                   | 计算日期 d1->d2 之间相隔的天数                               | `SELECT DATEDIFF("2020-06-19","2020-06-29")`                 |
| DATE_ADD(d，INTERVAL expr type)   | 计算起始日期 d 加上一个时间段后的日期                        | `SELECT DATE_ADD("2020-06-19", INTERVAL 2 DAY);`             |
| DATE_FORMAT(d,f)                  | 按表达式f的要求显示日期 d                                    | `SELECT DATE_FORMAT("2020-06-19","%y+%M+%d %r");`            |
| DATE_SUB(date,INTERVAL expr type) | 函数从日期减去指定的时间间隔。                               | 略                                                           |
| DAY(d)                            | 返回日期值 d 的日期部分                                      | `SELECT DAY("2020-06-19");`                                  |
| DAYNAME(d)                        | 返回日期 d 是星期几，如 Monday,Tuesday                       | `SELECT DAYNAME("2020-06-19");`                              |
| DAYOFMONTH(d)                     | 计算日期 d 是本月的第几天                                    | `SELECT DAYOFMONTH("2020-06-19");`                           |
| DAYOFWEEK(d)                      | 日期 d 今天是星期几，1 星期日，2 星期一，以此类推            | `SELECT DAYOFWEEK("2020-06-19");`                            |
| DAYOFYEAR(d)                      | 计算日期 d 是本年的第几天                                    | `SELECT DAYOFYEAR("2020-06-19");`                            |
| EXTRACT(type FROM d)              | 从日期 d 中获取指定的值，type 指定返回的值。                 | `SELECT EXTRACT(YEAR FROM "2020-06-19");`                    |
| ROM_DAYS(n)                       | 计算从 0000 年 1 月 1 日开始 n 天后的日期                    | `SELECT FROM_DAYS(737960);`                                  |
| HOUR(t)                           | 返回 t 中的小时值                                            | `SELECT HOUR('10:05:03');`                                   |
| LAST_DAY(d)                       | 返回给给定日期的那一月份的最后一天                           | `SELECT LAST_DAY("2020-06-19")`                              |
| LOCALTIME()                       | 返回当前日期和时间                                           | 略                                                           |
| LOCALTIMESTAMP()                  | 返回当前日期和时间                                           | 略                                                           |
| MAKEDATE(year, day-of year)       | 基于给定参数年份 year 和所在年中的天数序号 day-of-year 返回一个日期 | `SELECT MAKEDATE(2020,31)`                                   |
| MAKETIME(hour, minute, second)    | 组合时间，参数分别为小时、分钟、秒                           | `SELECT MAKETIME(12,15,30);`                                 |
| MICROSECOND(date)                 | 返回日期参数所对应的毫秒数                                   | `SELECT MICROSECOND('12:00:00.12');`                         |
| MINUTE(t)                         | 返回 t 中的分钟值                                            | `SELECT MINUTE('12:13:14');`                                 |
| MONTHNAME(d)                      | 返回日期当中的月份名称，如 Janyary                           | `SELECT MONTHNAME("2020-06-19");`                            |
| MONTH(d)                          | 返回日期d中的月份值，1 到 12                                 | `SELECT MONTH("2020-06-19");`                                |
| NOW()                             | 返回当前日期和时间                                           | `SELECT NOW();`                                              |
| PERIOD_ADD(period, number)        | 为 年-月 组合日期添加一个时段                                | `SELECT PERIOD_ADD(199801,2);`                               |
| PERIOD_DIFF(period1, period2)     | 返回两个时段之间的月份差值                                   | `SELECT PERIOD_DIFF(9802,199703);`                           |
| QUARTER(d)                        | 返回日期d是第几季节，返回 1 到 4                             | `SELECT QUARTER("2020-06-19");`                              |
| SECOND(t)                         | 返回 t 中的秒钟值                                            | `SELECT SECOND('12:13:14');`                                 |
| SEC_TO_TIME(s)                    | 将以秒为单位的时间 s 转换为时分秒的格式                      | `SELECT SEC_TO_TIME('54925');`                               |
| STR_TO_DATE(string, format_mask)  | 将字符串转变为日期                                           | `SELECT STR_TO_DATE('06/19/2020', '%m/%d/%Y');`              |
| SUBDATE(d,n)                      | 日期 d 减去 n 天后的日期                                     | `SELECT SUBDATE("2020-06-19", 1);`                           |
| SUBTIME(t,n)                      | 时间 t 减去 n 秒的时间                                       | `SELECT SUBTIME('12:13:14', 14);`                            |
| SYSDATE()                         | 返回当前日期和时间                                           | 略                                                           |
| TIME(expression)                  | 提取传入表达式的时间部分                                     | `SELECT TIME("19:30:10");`                                   |
| TIME_FORMAT(t,f)                  | 按表达式 f 的要求显示时间 t                                  | `SELECT TIME_FORMAT(NOW(), '%l:%i %p');`                     |
| TIME_TO_SEC(t)                    | 将时间 t 转换为秒                                            | `SELECT TIME_TO_SEC(NOW());`                                 |
| TIMEDIFF(time1, time2)            | 计算时间差值                                                 | `SELECT TIMEDIFF('12:00','10:00');`                          |
| TIMESTAMP(expression, interval)   | 单个参数时，函数返回日期或日期时间表达式；有2个参数时，将参数加和 | `SELECT TIMESTAMP("2020-06-19");`<br />`SELECT TIMESTAMP("2020-06-19", "01:00:00");` |
| TO_DAYS(d)                        | 计算日期 d 距离 0000 年 1 月 1 日的天数                      | `SELECT TO_DAYS("2020-06-19");`                              |
| WEEK(d)                           | 计算日期 d 是本年的第几个星期，范围是 0 到 53                | `SELECT WEEK("2020-06-19");`                                 |
| WEEKDAY(d)                        | 日期 d 是星期几，0 表示星期一，1 表示星期二                  | `SELECT WEEKDAY("2020-06-19");`                              |
| WEEKOFYEAR(d)                     | 计算日期 d 是本年的第几个星期，范围是 0 到 53                | `SELECT WEEKOFYEAR("2020-06-19");`                           |
| YEAR(d)                           | 返回年份                                                     | `SELECT YEAR("2020-06-19");`                                 |
| YEARWEEK(date, mode)              | 返回年份及第几周（0到53），mode 中 0 表示周天，1表示周一，以此类推 | `SELECT YEARWEEK("2020-06-19", 0);`                          |

### 高级函数

| 函数名                               | 描述                                                         | 实例                                                         |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BIN(x)                               | 返回 x 的二进制编码                                          | `SELECT BIN(15);`                                            |
| BINARY(s)                            | 将字符串 s 转换为二进制字符串                                | `SELECT BINARY("itcast");`                                   |
| CAST(x AS type)                      | 转换数据类型                                                 | `SELECT CAST("2017-08-29" AS DATE);`                         |
| COALESCE(expr1, expr2, ...., expr_n) | 返回参数中的第一个非空表达式（从左 向右）                    | `SELECT COALESCE(NULL, NULL, 'a', NULL, 'b');`               |
| CONNECTION_ID()                      | 返回服务器的连接数                                           | `SELECT CONNECTION_ID();`                                    |
| CONV(x,f1,f2)                        | 返回 f1 进制数变成 f2 进制数                                 | `SELECT CONV(15, 10, 2);`                                    |
| CONVERT(s USING cs)                  | 函数将字符串 s 的字符集变成 cs                               | `SELECT CHARSET('ABC');`<br/>`SELECT CHARSET(CONVERT('ABC' USING gbk));` |
| CURRENT_USER()                       | 返回当前用户                                                 | `SELECT CURRENT_USER();`                                     |
| DATABASE()                           | 返回当前数据库名                                             | `SELECT DATABASE();`                                         |
| IF(expr,v1,v2)                       | 如果表达式 expr 成立，返回结果 v1； 否则，返回结果 v2。      | `SELECT IF(1 > 0,'正确','错误');`                            |
| IFNULL(v1,v2)                        | 如果 v1 的值不为 NULL，则返回 v1， 否则返回 v2。             | `SELECT IFNULL(NULL,'Hello Word')`                           |
| ISNULL(expression)                   | 判断表达式是否为空                                           | `SELECT ISNULL(NULL);`                                       |
| LAST_INSERT_ID()                     | 返回最近生成的 AUTO_INCREMENT 值                             | `SELECT LAST_INSERT_ID();`                                   |
| NULLIF(expr1, expr2)                 | 比较两个字符串，如果字符串 expr1 与 expr2 相等 返回 NULL，否则返回 expr1 | `SELECT NULLIF("258", "248");`                               |
| SESSION_USER()                       | 返回当前用户                                                 | 略                                                           |
| SYSTEM_USER()                        | 返回当前用户                                                 | 略                                                           |
| USER()                               | 返回当前用户lue                                              | 略                                                           |
| VERSION()                            | 返回数据库的版本号                                           | 略                                                           |

### 其他函数

* **group_concat**

`group_concat()`实现分组聚合

语法：

```mysql
group_concat([DISTINCT] 要连接的字段 [Order BY 排序字段 ASC/DESC] [Separator ‘分隔符’])
```

首先创建表并加入数据

```mysql
-- 创建表
CREATE TABLE goods(
	id INT NOT NULL,
	price INT NOT NULL
)

INSERT INTO goods (id, price) VALUES
(1, 10),(1,20),(1,30),(1,40),(1,40),
(2, 10),(2,20),(2,30),(2,30),
(3, 10),(3,20);

SELECT * FROM goods;
-- 结果：
-- id	price
-- 1	10
-- 1	20
-- 1	30
-- 1	40
-- 1	40
-- 2	10
-- 2	20
-- 2	30
-- 2	30
-- 3	10
-- 3	20
```

使用演示：

```mysql
-- 以id分组，把price字段的值在同一行打印出来，逗号分隔(默认)
SELECT id, GROUP_CONCAT(price) FROM goods GROUP BY id;
-- 结果
-- id	group_concat(price)
-- 1	10,20,30,40,40
-- 2	10,20,30,30
-- 3	10,20

-- 以id分组，把price字段的值在一行打印出来，分号分隔
SELECT id, GROUP_CONCAT(price SEPARATOR ';') FROM goods GROUP BY id;
-- 结果
-- id	GROUP_CONCAT(price separator ';')
-- 1	10;20;30;40;40
-- 2	10;20;30;30
-- 3	10;20

-- 以id分组，把去除重复冗余的price字段的值打印在一行，逗号分隔
SELECT id,GROUP_CONCAT(DISTINCT price) FROM goods GROUP BY id; 
-- 结果
-- id	group_concat(distinct price)
-- 1	10,20,30,40
-- 2	10,20,30
-- 3	10,20

-- 以id分组，把price字段的值去重打印在一行，逗号分隔，按照price倒序排列
SELECT id,GROUP_CONCAT(DISTINCT price ORDER BY price DESC) FROM goods GROUP BY id;  
-- 结果
-- id	group_concat(DISTINCT price order by price desc)
-- 1	40,30,20,10
-- 2	30,20,10
-- 3	20,10
```

* **substring_index**

substring_index实现切分

格式：

```mysql
substring_index(str, delim, count)

-- str:要处理的字符串
-- delim:分隔符
-- count:计数
```

使用：

```mysql
SELECT SUBSTRING_INDEX('www.wikibt.com', '.', 1);
-- 结果是：www

SELECT SUBSTRING_INDEX('www.wikibt.com','.',2);
-- 结果是：www.wikibt
-- 也就是说，如果count是正数，那么就是从左往右数，第N个分隔符的左边的全部内容
-- 相反，如果是负数，那么就是从右边开始数，第N个分隔符右边的所有内容，如：
SELECT SUBSTRING_INDEX('www.wikibt.com','.',-2)
-- 结果为：wikibt.com

-- 若要取wikibt怎么办？
-- 从右数第二个分隔符的右边全部，再从左数的第一个分隔符的左边：
SELECT SUBSTRING_INDEX(SUBSTRING_INDEX('www.wikibt.com','.',-2),'.',1);
-- 结果是：wikibt
```

## 窗口函数

### 概述

![窗口函数](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251658109.png)

**窗口**：记录集合

**窗口函数**：在满足某些条件的记录集合上执行的特殊函数，对于每条记录都要在此窗口内执行函数。

* 静态窗口：有的函数随着记录的不同，窗口大小都是固定的；
* 滑动窗口：有的函数则相反，不同的记录对应着不同的窗口，称为滑动窗口。

**窗口函数和普通聚合函数的区别**

* 聚合函数是将多条记录聚合为一条；窗口函数是每条记录都会执行，**有几条记录执行完还是几条**；
* 聚合函数也可以用于窗口函数

**基本语法**：`函数名 over 子句`

over：用来指定函数执行的窗口范围，若后面括号中什么都不写，则意味着窗口包含满足WHERE条件的所有行，窗口函数基于所有行进行计算；如果不为空，则支持以下4种语法来设置窗口

* window_name：给窗口指定一个别名；

  > 给窗口指定别名：**WINDOW** 别名 **AS** (PARTITION BY  字段1 ORDER BY 字段2)

* PARTITION BY 子句：窗口按照哪些字段进行分组，窗口函数在不同的分组上分别执行；

* ORDER BY子句：按照哪些字段进行排序，窗口函数将按照排序后的记录顺序进行编号；

* FRAME子句：`FRAME`是当前分区的一个子集，子句用来定义子集的规则，通常用来作为滑动窗口使用。

### 序号函数

用途：显示分区中的当前行号

`ROW_NUMBER()`：顺序排序——1、2、3

`RANK()`：并列排序，跳过重复序号——1、1、3

`DENSE_RANK()`：并列排序，不跳过重复序号——1、1、2

举例：

```mysql
-- 查询学生的每门课成绩，进行排序
-- 创建表
CREATE TABLE stu_score(
	stu_id INT,
	lesson_id VARCHAR(4),
	score INT
);

INSERT INTO stu_score VALUES
(1, 'L001', 90), (1, 'L002', 95), (1, 'L003', 91), (1, 'L004', 92),
(2, 'L001', 95), (2, 'L002', 90), (2, 'L003', 90), (2, 'L004', 90),
(3, 'L001', 85), (3, 'L002', 85), (3, 'L003', 95), (3, 'L004', 95),
(4, 'L001', 92), (4, 'L002', 92), (4, 'L003', 92), (4, 'L004', 92);

SELECT * FROM stu_score;
-- 结果
-- stu_id	lesson_id	score
-- 1	L001	90
-- 1	L002	95
-- 1	L003	91
-- 1	L004	92
-- 2	L001	95
-- 2	L002	90
-- 2	L003	90
-- 2	L004	90
-- 3	L001	85
-- 3	L002	85
-- 3	L003	95
-- 3	L004	95
-- 4	L001	92
-- 4	L002	92
-- 4	L003	92
-- 4	L004	92
```

`ROW_NUMBER()`：顺序排序——1、2、3

```mysql
SELECT
	stu_id, lesson_id, score, ROW_NUMBER() over (PARTITION BY stu_id ORDER BY score DESC) AS score_order
FROM
	stu_score;

-- 结果
-- stu_id	lesson_id	score	score_order
-- 1	     L002	       95	  1
-- 1	     L004	       92	  2
-- 1	     L003	       91	  3
-- 1	     L001	       90	  4
-- 2	     L001	       95	  1
-- 2	     L002	       90	  2
-- 2	     L003	       90	  3
-- 2	     L004	       90	  4
-- 3	     L003	       95	  1
-- 3	     L004	       95	  2
-- 3	     L001	       85	  3
-- 3	     L002	       85	  4
-- 4	     L001	       92	  1
-- 4	     L002	       92	  2
-- 4	     L003	       92	  3
-- 4	     L004	       92	  4
```

`RANK()`：并列排序，跳过重复序号——1、1、3

```mysql
SELECT
	stu_id, lesson_id, score, RANK() over (PARTITION BY stu_id ORDER BY score DESC) AS score_order
FROM
	stu_score;

-- 结果
-- stu_id	lesson_id	score	score_order
-- 1	L002	95	1
-- 1	L004	92	2
-- 1	L003	91	3
-- 1	L001	90	4
-- 2	L001	95	1
-- 2	L002	90	2
-- 2	L003	90	2
-- 2	L004	90	2
-- 3	L003	95	1
-- 3	L004	95	1
-- 3	L001	85	3
-- 3	L002	85	3
-- 4	L001	92	1
-- 4	L002	92	1
-- 4	L003	92	1
-- 4	L004	92	1
```

`DENSE_RANK()`：并列排序，不跳过重复序号——1、1、2

```mysql
SELECT
	stu_id, lesson_id, score, DENSE_RANK() over (PARTITION BY stu_id ORDER BY score DESC) AS score_order
FROM
	stu_score;
	
-- 结果
-- stu_id	lesson_id	score	score_order
-- 1	L002	95	1
-- 1	L004	92	2
-- 1	L003	91	3
-- 1	L001	90	4
-- 2	L001	95	1
-- 2	L002	90	2
-- 2	L003	90	2
-- 2	L004	90	2
-- 3	L003	95	1
-- 3	L004	95	1
-- 3	L001	85	2
-- 3	L002	85	2
-- 4	L001	92	1
-- 4	L002	92	1
-- 4	L003	92	1
-- 4	L004	92	1
```

### 分布函数

`PERCENT_RANK()`、`CUME_DIST()`

**`PERCENT_RANK()`**：每行按照公式`(rank-1) / (rows-1)`进行计算。其中，`rank`为`RANK()`函数产生的序号，`rows`为当前窗口的记录总行数；即计算百分比等级值

> 不常用

```mysql
SELECT
	stu_id, lesson_id, score,
	rank() over w AS rk,
	percent_rank() over w AS prk
FROM stu_score
WHERE stu_id=1
window w AS (PARTITION BY stu_id ORDER BY score);  -- 起了别名

-- 结果：
-- stu_id	lesson_id	score	rk	prk
-- 1	L001	90	1	0
-- 1	L003	91	2	0.3333333333333333
-- 1	L004	92	3	0.6666666666666666
-- 1	L002	95	4	1
```

**`CUME_DIST()`**：分组内小于、等于当前rank值的行数 / 分组内总行数

```mysql
-- 查询小于等于当前成绩（score）的比例
SELECT
	stu_id, lesson_id, score,
	-- 没有分区，则所有数据均为一组，总行数为8
	cume_dist() over (ORDER BY score) AS cd1,	
	-- 按照lesson_id分成了两组，行数各为4		
	cume_dist() over (PARTITION BY lesson_id ORDER BY score) AS cd2  
FROM
	stu_score
WHERE
	lesson_id IN ('L001', 'L003');
	
-- 结果
-- stu_id	lesson_id	score	cd1	cd2
-- 3	L001	85	    0.125	0.25
-- 1	L001	90	    0.375	0.5
-- 4	L001	92	    0.75	0.75
-- 2	L001	95	    1	    1
-- 2	L003	90	    0.375	0.25
-- 1	L003	91	    0.5	    0.5
-- 4	L003	92	    0.75	0.75
-- 3	L003	95	    1	    1
```

### 前后函数

`LAG(expr,n)`、`LEAD(expr,n)`：

返回位于当前行的前n行（`LAG(expr,n)`）或后n行（`LEAD(expr,n)`）的expr的值

```mysql
-- 查询前1名同学的成绩和当前同学成绩的差值

-- 1. 先查前一名同学的成绩
SELECT
	stu_id, lesson_id, score,
	lag(score, 1) over w AS pre_score  -- 返回当前行前一行的score
FROM 
	stu_score
WHERE
	lesson_id IN ('L001', 'L003')
window w AS (PARTITION BY lesson_id ORDER BY score);

-- 2. 再查差值
SELECT 
	stu_id, lesson_id, score, pre_score, IFNULL((score - pre_score), 0) AS diff
FROM
	(SELECT
		stu_id, lesson_id, score,
		lag(score, 1) over w AS pre_score  -- 返回当前行前一行的score
	FROM 
		stu_score
	WHERE
		lesson_id IN ('L001', 'L003')
	window w AS (PARTITION BY lesson_id ORDER BY score)) AS t;
```

### 头尾函数

`FIRST_VALUE(expr)`、`LAST_VALUE(expr)`：

返回第一个（`FIRST_VALUE(expr)`）或最后一个（`LAST_VALUE(expr)`）expr的值

```mysql
-- 按照课程成绩降序排序后找出第一个与最后一个score
SELECT
	stu_id, lesson_id, score,
	first_value(score) over w AS frist_score,
	last_value(score) over w AS last_score
FROM
	stu_score
WHERE
	lesson_id IN ('L001', 'L003')
window w AS (PARTITION BY lesson_id ORDER BY score DESC);
```

### 其它函数

`NTH_VALUE(expr, n)`、`NTILE(n)`

**`NTH_VALUE(expr,n)`**：返回窗口中第n个`expr`的值。`expr`可以是表达式，也可以是列名

```mysql
-- 截止到当前成绩，显示每个同学的成绩中排名第2和第3的成绩的分数
SELECT
	stu_id, lesson_id, score,
	nth_value(score, 2) over w AS second_score,
	nth_value(score, 3) over w AS third_score
FROM
	stu_score
WHERE
	stu_id IN (1, 3)
window w AS (PARTITION BY stu_id ORDER BY score);

-- 结果
-- stu_id	lesson_id	score	second_score	third_score
-- 1	L001	90	\N	\N
-- 1	L003	91	91	\N
-- 1	L004	92	91	92
-- 1	L002	95	91	92
-- 3	L001	85	85	\N
-- 3	L002	85	85	\N
-- 3	L003	95	85	95
-- 3	L004	95	85	95
```

**`NTILE(n)`**：将分区中的有序数据分为n个等级，记录等级数

```mysql
-- 将每门课程按照成绩分成3组
SELECT
	stu_id, lesson_id, score,
	ntile(3) over w AS nf
FROM
	stu_score
WHERE
	lesson_id IN ('L001', 'L003')
window w AS (PARTITION BY lesson_id ORDER BY score);

-- 结果：
-- stu_id	lesson_id	score	nf
-- 3	L001	85	1
-- 1	L001	90	1
-- 4	L001	92	2
-- 2	L001	95	3
-- 2	L003	90	1
-- 1	L003	91	1
-- 4	L003	92	2
-- 3	L003	95	3
```

> `NTILE(n)`函数在数据分析中应用较多，比如由于数据量大，需要将数据平均分配到n个并行的进程分别计算，此时就可以用`NTILE(n)`对数据进行分组（由于记录数不一定被n整除，所以数据不一定完全平均），然后将不同桶号的数据再分配。

### 聚合函数作为窗口函数

在窗口中每条记录动态地应用聚合函数（`SUM()`、`AVG()`、`MAX()`、`MIN()`、`COUNT()`），可以动态计算在指定的窗口内的各种聚合函数值

```mysql
-- 截止到当前时间，查询stu_id=1的学生的累计分数、分数最高的科目、分数最低的科目
SELECT
	stu_id, lesson_id, score,
	SUM(score) over w AS score_num,
	MAX(score) over w AS score_max,
	MIN(score) over w AS score_min
FROM
	stu_score
WHERE
	stu_id=1
window w AS (PARTITION BY stu_id);
```

