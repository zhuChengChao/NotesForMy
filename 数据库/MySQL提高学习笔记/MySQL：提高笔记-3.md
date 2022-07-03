# MySQL：提高笔记-3

> 之前学了一波 MySQL 在使用层面的内容，现在则将其进行一步升华，稍稍深入的学习一些 MySQL 的内部实现等进阶知识点，这当然也是跟着视频学习时做的笔记，课程地址：[MySQL入门到精通](https://www.bilibili.com/video/BV1zJ411M7TB)，笔记总共分为5篇，此为 **3/5**。

## 1. 索引的使用

索引是数据库优化最常用也是最重要的手段之一，通过索引通常可以帮助用户解决大多数的 MySQL 的性能优化问题。

### 1.1 验证索引提升查询效率

在准备的表结构 tb_item 中， 一共存储了 300 万记录；

> 建表语句
>
> ```mysql
> -- tb_item 表结构
> CREATE TABLE `tb_item` (
>     `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '商品id',
>     `title` varchar(100) NOT NULL COMMENT '商品标题',
>     `price` decimal(20,2) NOT NULL COMMENT '商品价格，单位为：元',
>     `num` int(10) NOT NULL COMMENT '库存数量',
>     `categoryid` bigint(10) NOT NULL COMMENT '所属类目，叶子类目',
>     `status` varchar(1) DEFAULT NULL COMMENT '商品状态，1-正常，2-下架，3-删除',
>     `sellerid` varchar(50) DEFAULT NULL COMMENT '商家ID',
>     `createtime` datetime DEFAULT NULL COMMENT '创建时间',
>     `updatetime` datetime DEFAULT NULL COMMENT '更新时间',
>     PRIMARY KEY (`id`)
> ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='商品表'
> ```

1. **根据ID查询** 

```mysql
select * from tb_item where id = 1999\G;

*************************** 1. row ***************************
	     id: 1999
	  title: 诺基亚（NOKIA）1050 （RM-908）蓝色 移动互联2G手机1999
	  price：2054.00
		num：226
 categoryid：1199
	 status：0
   sellerid：oppo
currenttime：2088-03-13 09:42:23
 updatetime：2088-03-13 09:42:23
```

查询速度很快， 接近0s，主要的原因是因为id为主键， 有索引；

```mysql
mysql> explain select * from tb_item where id = 1999\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_item
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
        Extra: NULL
1 row in set (0.00 sec) 
```

2. **根据 title 进行精确查询**

```mysql
select * from tb_item where title = 'iphoneX 移动3G 32G941'\G; 

*************************** 1. row ***************************
	     id: 941
      title: iphoneX 移动3G 32G941
      price: 7036.00
 categoryid: 963
     status: 0
   sellerid: xiaomi
currenttime：2088-03-13 09:42:23
 updatetime：2088-03-13 09:42:23
1 row in set (9.15 sec)  -- 查询的时间很长
```

查看 SQL 语句的执行计划 ： 

```mysql
mysql> explain select * from tb_item where title = 'iphoneX 移动3G 32G941'\G; 
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_item
         type: ALL
possible_keys: NULL
          key: NULL     -- 查询是没有使用索引查询
      key_len: NULL
          ref: NULL
         rows: 9816098
        Extra: NULL
1 row in set (0.02 sec)  
```

处理方案，针对 title 字段， 创建索引 ： 

```SQL
create index idx_item_title on tb_item(title);
Query OK, 0 row affected (3 min 3.19 sec)  -- 建立索引的时间还是比较长的
Records: 0  Duplicates: 0  Warnings: 0 
```

索引创建完成之后，再次进行查询 ： 

```mysql
select * from tb_item where title = 'iphoneX 移动3G 32G941'\G; 

*************************** 1. row ***************************
	     id: 941
      title: iphoneX 移动3G 32G941
      price: 7036.00
 categoryid: 963
     status: 0
   sellerid: xiaomi
currenttime：2088-03-13 09:42:23
 updatetime：2088-03-13 09:42:23
1 row in set (0.00 sec)       -- 对比上面的时间，这里大大大大的减少了 
```

通过 explain，查看执行计划，执行SQL时使用了刚才创建的索引 

```mysql
mysql> explain select * from tb_item where title = 'iphoneX 移动3G 32G941'\G; 
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_item
         type: ref
possible_keys: idx_item_title
          key: idx_item_title   -- 这里就使用了索引进行查询
      key_len: 302
          ref: const
         rows: 1
        Extra: Using index condition
1 row in set (0.02 sec)  
```

### 1.2 索引的使用

#### 1.2.1 准备环境

```sql
create table `tb_seller` (
	`sellerid` varchar (100),
	`name` varchar (100),
	`nickname` varchar (50),
	`password` varchar (60),
	`status` varchar (1),
	`address` varchar (100),
	`createtime` datetime,
    primary key(`sellerid`)
)engine=innodb default charset=utf8mb4; 

insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('alibaba','阿里巴巴','阿里小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('baidu','百度科技有限公司','百度小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('huawei','华为科技有限公司','华为小店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itcast','传智播客教育科技有限公司','传智播客','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('itheima','黑马程序员','黑马程序员','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('luoji','罗技科技有限公司','罗技小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('oppo','OPPO科技有限公司','OPPO官方旗舰店','e10adc3949ba59abbe56e057f20f883e','0','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('ourpalm','掌趣科技股份有限公司','掌趣小店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('qiandu','千度科技','千度小店','e10adc3949ba59abbe56e057f20f883e','2','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('sina','新浪科技有限公司','新浪官方旗舰店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('xiaomi','小米科技','小米官方旗舰店','e10adc3949ba59abbe56e057f20f883e','1','西安市','2088-01-01 12:00:00');
insert into `tb_seller` (`sellerid`, `name`, `nickname`, `password`, `status`, `address`, `createtime`) values('yijia','宜家家居','宜家家居旗舰店','e10adc3949ba59abbe56e057f20f883e','1','北京市','2088-01-01 12:00:00');

-- 创建索引 对 name status address 字段建立索引
create index idx_seller_name_sta_addr on tb_seller(name, status, address);
```

#### 1.2.2 避免索引失效

1. **全值匹配 ，对索引中所有列都指定具体值。**

该情况下，索引生效，执行效率高。

```mysql
explain select * from tb_seller where name='小米科技' and status='1' and address='北京市';
```

![使用索引查询](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653551.PNG)

2. **最左前缀法则**

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始，并且不跳过索引中的列。

匹配最左前缀法则，则索引：

```mysql
explain select * from tb_seller where name='小米科技';
explain select * from tb_seller where name='小米科技' and status='1';
explain select * from tb_seller where name='小米科技' and status='1' and address='北京市';
```

![左前缀法则](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653226.png)  

违法最左前缀法则，索引失效：

```mysql
explain select * from tb_seller where status='1';
explain select * from tb_seller where status='1' and address='北京市';
```

![违反最左前缀法则](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653340.png) 

如果符合最左法则，但是出现跳跃某一列，只有最左列索引生效：

```mysql
explain select * from tb_seller where name='小米科技'and address='北京市';  
-- 跳过了status，则address='北京市'没有使用索引
```

![最左前缀跳跃某一列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653410.png) 

3. **在范围查询条件右边的列，不能使用索引：**

```mysql
explain select * from tb_seller where name='小米科技' and status>'1' and address='北京市';  
-- address='北京市'在范围查询条件右边没有使用索引，即可以看出address没有用索引
```

![索引匹配在范围查询条件右边的列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653822.png) 

根据前面的两个字段name，status 查询是走索引的， 但是最后一个条件address 没有用到索引。

4. **不要在索引列上进行运算操作， 索引将失效：**

```mysql
explain select * from tb_seller where substring(name, 3, 2)="科技";
```

![索引列上进行运算操作](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653752.png) 

5. **字符串不加单引号，造成索引失效：**

```mysql
-- 使用索引
explain select * from tb_seller where name="科技" and status="1";

-- 不使用索引
explain select * from tb_seller where name="科技" and status=1;
```

![字符串不加单引号索引失效](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653713.png) 

由于，在查询时，没有对字符串加单引号，MySQL的查询优化器，会自动的进行类型转换，造成索引失效。

6. **尽量使用覆盖索引，避免 `select *`**

尽量使用覆盖索引（只访问索引的查询（索引列完全包含查询列）），减少 `select *` 

```mysql
-- using index condition：需要回表查数据
explain select * from tb_seller where name="科技" and status="0" and address="北京市";

-- using index ; using where  不需要回表查数据
explain select name from tb_seller where name="科技" and status="0" and address="北京市";
explain select name, status from tb_seller where name="科技" and status="0" and address="北京市";
explain select name, status, address from tb_seller where name="科技" and status="0" and address="北京市";
```

![尽量使用覆盖索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653283.png) 

如果查询列，超出索引列，也会降低性能

```mysql
-- password 超出了索引列，退化为 using index condition
explain select name, status, address, password from tb_seller where name="科技" and status="0" and address="北京市";
```

![超出索引列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653856.png) 

> **TIP :** 
>
> * using index ：使用覆盖索引的时候就会出现
>
> * using where：在查找使用索引的情况下，需要回表去查询所需的数据
>
> * using index condition：查找使用了索引，但是需要回表查询数据
>
> * using index ; using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

7. **用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。**

```sql
-- 示例，name字段是索引列，而createtime不是索引列，中间是or进行连接是不走索引的
explain select * from tb_seller where name='黑马程序员' or createtime = '2088-01-01 12:00:00';	
```

![or分开不走索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653449.png) 

8. **以%开头的Like模糊查询，索引失效。**

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

```mysql
-- 索引不失效
explain select * from tb_seller where name like "科技%";

-- 索引失效
explain select * from tb_seller where name like "%科技";
```

![索引之模拟匹配](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653221.png) 

> 解决方案 ： 
>
> 通过覆盖索引来解决，即查询的列都建立了索引的
>
> ```mysql
> explain select sellerid from tb_seller where name like "%科技";
> explain select sellerid, name from tb_seller where name like "%科技";
> explain select sellerid, name, address from tb_seller where name like "%科技%";
> ```
>
> ![覆盖索引来解决模糊匹配](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653333.png)

9. **如果MySQL评估使用索引比全表更慢，则不使用索引。**

```mysql
explain select * from tb_seller where address = '北京市';
create index idx_address ON tb_seller(address);
explain select * from tb_seller where address = '北京市';
```

![mysql自己评估是否用索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653473.png) 

10. **`is NULL`，`is NOT NULL`  有时索引失效。**

依据：若所查询的列数据中，根据数据是 NULL 还是 NOT NULL 的占比来决定是否使用索引。

![null时索引可能失效](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653114.png)  

11. **in则索引， not in索引失效。**

```mysql
explain select * from tb_seller where sellerid in ("alibaba", "baidu", "xiaomi");

explain select * from tb_seller where sellerid not in ("alibaba", "baidu", "xiaomi");
```

![notin导致索引失效](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251653975.png)  

12. **单列索引和复合索引。**

尽量使用复合索引，而少使用单列索引 。

创建复合索引 

```mysql
create index idx_name_sta_address on tb_seller(name, status, address);

-- 就相当于创建了三个索引 ： 
-- 	name
-- 	name + status
-- 	name + status + address
```

创建单列索引 

```mysql
create index idx_seller_name on tb_seller(name);
create index idx_seller_status on tb_seller(status);
create index idx_seller_address on tb_seller(address);
```

数据库会选择一个最优的索引（辨识度最高索引）来使用，并不会使用全部索引 。

### 1.3 查看索引使用情况

```sql
show status like 'Handler_read%';	-- 当前会话

show global status like 'Handler_read%';	-- 全局

+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Handler_read_first    | 75    |
| Handler_read_key      | 105   |
| Handler_read_last     | 0     |
| Handler_read_next     | 27    |
| Handler_read_prev     | 0     |
| Handler_read_rnd      | 0     |
| Handler_read_rnd_next | 764   |
+-----------------------+-------+
7 rows in set (0.00 sec)
```

* Handler_read_first：索引中第一条被读的次数。如果较高，表示服务器正执行大量全索引扫描（这个值越低越好）。

* Handler_read_key：如果索引正在工作，这个值代表一个行被索引值读的次数，如果值越低，表示索引得到的性能改善不高，因为索引不经常使用（这个值越高越好）。

* Handler_read_next ：按照键顺序读下一行的请求数。如果你用范围约束或如果执行索引扫描来查询索引列，该值增加。

* Handler_read_prev：按照键顺序读前一行的请求数。该读方法主要用于优化 `ORDER BY ... DESC`。

* Handler_read_rnd ：根据固定位置读一行的请求数。如果你正执行大量查询并需要对结果进行排序该值较高。你可能使用了大量需要 MySQL 扫描整个表的查询或你的连接没有正确使用键。这个值较高，意味着运行效率低，应该建立索引来补救。

* Handler_read_rnd_next：在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明你的表索引不正确或写入的查询没有利用索引。

## 2. SQL优化

### 2.1 大批量插入数据

环境准备 ： 

```sql
-- 分别建立tb_user_1 和 tb_user_2 表
CREATE TABLE `tb_user_1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(45) NOT NULL,
  `password` varchar(96) NOT NULL,
  `name` varchar(45) NOT NULL,
  `birthday` datetime DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `email` varchar(45) DEFAULT NULL,
  `phone` varchar(45) DEFAULT NULL,
  `qq` varchar(32) DEFAULT NULL,
  `status` varchar(32) NOT NULL COMMENT '用户状态',
  `create_time` datetime NOT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_user_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ;
```

当使用 load 命令导入数据的时候，适当的设置可以提高导入的效率。

![load导入数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654211.png) 

对于 InnoDB 类型的表，有以下几种方式可以提高导入的效率：

1. **主键顺序插入**

因为InnoDB类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效的提高导入数据的效率。如果InnoDB表没有主键，那么系统会自动默认创建一个内部列作为主键，所以如果可以给表创建一个主键，将可以利用这点，来提高导入数据的效率。

脚本文件介绍 :

* sql1.log  ----> 主键有序
* sql2.log  ----> 主键无序

插入ID顺序排列数据：

```mysql
load data local infile “文件地址和文件名” into table 表名 fields terminated by "字段分割符" lines terminated by "行数据分割符"

-- 例如：
load data local infile "./sql1.log" into table tb_user_1 fields terminated by "," lines terminated by "\n";
load data local infile "./sql2.log" into table tb_user_2 fields terminated by "," lines terminated by "\n";
```

![load导入有序与无序](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654823.png)

插入ID 无序排列数据：花费时间更长

![load导入无序](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654626.png) 

2. **关闭唯一性校验**

在导入数据前执行 `SET UNIQUE_CHECKS=0`，关闭唯一性校验，在导入结束后执行`SET UNIQUE_CHECKS=1`，恢复唯一性校验，可以提高导入的效率。

![load时关闭唯一性校验](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654839.png) 

3. **手动提交事务**

如果应用使用自动提交的方式，建议在导入前执行 `SET AUTOCOMMIT=0`，关闭自动提交，导入结束后再执行 `SET AUTOCOMMIT=1`，打开自动提交，也可以提高导入的效率。

![手动提交事物](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654807.png)

### 2.2 优化insert语句

当进行数据的insert操作的时候，可以考虑采用以下几种优化方案。

1. 如果需要同时对一张表插入很多行数据时，应该尽量**使用多个值表的insert语句**，这种方式将大大的缩减客户端与数据库之间的连接、关闭等消耗。使得效率比分开执行的单个insert语句快。

      示例， 原始方式为：

      ```sql
      insert into tb_test values(1,'Tom');
      insert into tb_test values(2,'Cat');
      insert into tb_test values(3,'Jerry');
      ```

      优化后的方案为 ： 

      ```sql
      insert into tb_test values(1,'Tom'),(2,'Cat')，(3,'Jerry');
      ```

2. **在事务中进行数据插入。**

      ```sql
      start transaction;
      insert into tb_test values(1,'Tom');
      insert into tb_test values(2,'Cat');
      insert into tb_test values(3,'Jerry');
      commit;
      ```

3. **数据有序插入**

      ```sql
      insert into tb_test values(4,'Tim');
      insert into tb_test values(1,'Tom');
      insert into tb_test values(3,'Jerry');
      insert into tb_test values(5,'Rose');
      insert into tb_test values(2,'Cat');
      ```

      优化后

      ```sql
      insert into tb_test values(1,'Tom');
      insert into tb_test values(2,'Cat');
      insert into tb_test values(3,'Jerry');
      insert into tb_test values(4,'Tim');
      insert into tb_test values(5,'Rose');
      ```

### 2.3 优化order by语句

#### 2.3.1 环境准备

```mysql
CREATE TABLE `emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `age` int(3) NOT NULL,
  `salary` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4;

insert into `emp` (`id`, `name`, `age`, `salary`) values('1','Tom','25','2300');
insert into `emp` (`id`, `name`, `age`, `salary`) values('2','Jerry','30','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('3','Luci','25','2800');
insert into `emp` (`id`, `name`, `age`, `salary`) values('4','Jay','36','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('5','Tom2','21','2200');
insert into `emp` (`id`, `name`, `age`, `salary`) values('6','Jerry2','31','3300');
insert into `emp` (`id`, `name`, `age`, `salary`) values('7','Luci2','26','2700');
insert into `emp` (`id`, `name`, `age`, `salary`) values('8','Jay2','33','3500');
insert into `emp` (`id`, `name`, `age`, `salary`) values('9','Tom3','23','2400');
insert into `emp` (`id`, `name`, `age`, `salary`) values('10','Jerry3','32','3100');
insert into `emp` (`id`, `name`, `age`, `salary`) values('11','Luci3','26','2900');
insert into `emp` (`id`, `name`, `age`, `salary`) values('12','Jay3','37','4500');

-- 创建索引
create index idx_emp_age_salary on emp(age,salary);
```

#### 2.3.2 两种排序方式

1. 第一种是通过对返回数据进行排序，也就是通常说的 filesort 排序，所有不是通过索引直接返回排序结果的排序都叫 filesort 排序。

```mysql
explain select * from emp order by age desc;
```

![通过对返回数据进行排序](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654595.png) 

2. 第二种通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

```mysql
explain select id from emp order by age asc;
```

![索引排序](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654285.png) 

多字段排序

```mysql
-- 同时升序 使用索引
explain select id, age, salary from emp order by age, salary; 

-- 同时降序 使用索引
explain select id, age, salary from emp order by age desc, salary desc;

-- 排序的顺序和索引的顺序不一致 不适用索引
explain select id, age, salary from emp order by salary desc, age desc;

-- 一升一降 不适用索引
explain select id, age, salary from emp order by age, salary desc;
```

![多字段排序](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654253.png) 

了解了MySQL的排序方式，优化目标就清晰了：

1. 尽量减少额外的排序，通过索引直接返回有序数据。
2. where 条件和 Order by 使用相同的索引；
3. 并且 Order By 的顺序和索引顺序相同， 并且 Order  by 的字段都是升序，或者都是降序。

否则肯定需要额外的操作，这样就会出现 filesort 。

#### 2.3.3 filesort 的优化

通过创建合适的索引，能够减少 filesort  的出现，但是在某些情况下，条件限制不能让 filesort 消失，那就需要加快 filesort 的排序操作。

对于filesort ，MySQL 有两种排序算法：

1. 两次扫描算法 ：MySQL4.1 之前，使用该方式排序。**首先根据条件取出排序字段和行指针信息**，然后在排序区 sort buffer 中排序，如果 sort buffer 不够，则在临时表 temporary table 中存储排序结果。完成排序之后，再根据行指针回表读取记录，该操作可能会导致大量随机 I/O 操作。

2. 一次扫描算法：**一次性取出满足条件的所有字段**，然后在排序区 sort  buffer 中排序后直接输出结果集。排序时内存开销较大，但是排序效率比两次扫描算法要高。

MySQL 通过比较系统变量 max_length_for_sort_data 的大小和 Query 语句取出的字段总大小， 来判定使用那种排序算法，如果 max_length_for_sort_data 更大，那么使用第二种优化之后的算法；否则使用第一种。

可以适当提高 sort_buffer_size 和 max_length_for_sort_data 系统变量，来增大排序区的大小，提高排序的效率。

```mysql
mysql> show variables like "sort_buffer_size";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| sort_buffer_size | 262144 |
+------------------+--------+
1 row in set (0.00 sec)

mysql> show variables like "max_length_for_sort_data";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| max_length_for_sort_data | 1024  |
+--------------------------+-------+
1 row in set (0.00 sec)
```

### 2.4 优化group by 语句

由于 GROUP BY 实际上也同样会进行排序操作，而且与 ORDER BY 相比，GROUP BY 主要只是多了排序之后的分组操作。当然，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算。所以，在GROUP BY 的实现过程中，与 ORDER BY 一样也可以利用到索引。

如果查询包含 group by 但是用户想要避免排序结果的消耗， 则可以执行 order by null 禁止排序。如下 ：

```SQL
drop index idx_emp_age_salary on emp;

explain select age, count(*) from emp group by age;
```

![groupby未优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654020.png)  

```sql
-- 优化后
explain select age, count(*) from emp group by age order by null;
```

![groupby优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654157.png)  

从上面的例子可以看出，第一个SQL语句需要进行"filesort"，而第二个 SQL 由于`order by  null`不需要进行 "filesort"， 而上文提过filesort往往非常耗费时间。

```SQL
-- 创建索引
create index idx_emp_age_salary on emp(age,salary)

explain select age, count(*) from emp group by age order by null;
```

![groupby创建索引优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654749.png) 

### 2.5 优化嵌套查询

Mysql4.1版本之后，开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询是可以被更高效的连接（join）替代。

示例 ，查找有角色的所有的用户信息 : 

```SQL
explain select * from t_user where id in (select user_id from user_role );
```

执行计划为 : 

![子查询未优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654275.png)   

优化后 :

```SQL
explain select * from t_user u , user_role ur where u.id = ur.user_id;
```

![子查询优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654813.png)   

连接(Join)查询之所以更有效率一些 ，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。

### 2.6 优化OR条件

对于包含 OR 的查询子句，如果要利用索引，**则OR之间的每个条件列都必须用到索引，而且不能使用到复合索引；** 如果没有索引，则应该考虑增加索引。

获取 emp 表中的所有的索引 ：

```mysql
show index from emp;
```

![查看索引](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654541.png)  

示例 ： 

```mysql
-- name条件没有索引，故没有使用索引
mysql> explain select * from emp where id=1 or name="a";
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | emp   | ALL  | PRIMARY       | NULL | NULL    | NULL |   12 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

-- salary为复合索引，故没有使用索引
mysql> explain select * from emp where id=1 or salary=10;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | emp   | ALL  | PRIMARY       | NULL | NULL    | NULL |   12 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)
```

```SQL
-- id 和 age 都有单独索引：
explain select * from emp where id = 1 or age = 30;
```

![优化or1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654765.png)

```mysql
explain select * from emp where id=1 or id=10;
```

![优化or2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654141.png)  

建议使用 union 替换 or ： 

```mysql
explain select * from emp where id=1 union select * from emp where id=10;
```

![优化or3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654511.png) 

我们来比较下重要指标，发现主要差别是 type 和 ref 这两项

type 显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：

```
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
```

* UNION 语句的 type 值为 const，OR 语句的 type 值为 range，可以看到这是一个很明显的差距

* UNION 语句的 ref 值为 const，OR 语句的 ref 值为 null，const 表示是常量值引用，非常快

这两项的差距就说明了 UNION 要优于 OR 。

### 2.7 优化分页查询

一般分页查询时，通过创建覆盖索引能够比较好地提高性能。一个常见又非常头疼的问题就是 `limit 2000000,10`，此时需要MySQL排序前2000010 记录，仅仅返回2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 。

```mysql
explain select * from tb_item limit 2000000, 10
```

![优化分页查询](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654138.png) 

#### 2.7.1 优化思路一

在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

```mysql
explain select * from tb_item t, (select id from tb_item order by id limit 2000000, 10) a where t.id=a.id;
```

![优化分页查询2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654840.png) 

#### 2.7.2 优化思路二

该方案适用于主键自增的表，可以把Limit 查询转换成某个位置的查询。

```mysql
explain select * from tb_item where id > 1000000 limit 10;
```

![优化分页查询3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654509.png) 

### 2.8 使用SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

#### 2.8.1 USE INDEX

在查询语句中表名的后面，添加 use index 来提供希望MySQL去参考的索引列表，就可以让MySQL不再考虑其他可用的索引。

```mysql
-- 创建索引
create index idx_seller_name on tb_seller(name);

-- 采用idx_seller_name_sta_addr索引
explain select * from tb_seller where name="小米科技";

-- 采用idx_seller_name索引
explain select * from tb_seller use index(idx_seller_name) where name="小米科技";
```

![使用SQL提示useindex](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654157.png) 

#### 2.8.2 IGNORE INDEX

如果用户只是单纯的想让MySQL忽略一个或者多个索引，则可以使用 ignore index 作为 hint 。

```mysql
explain select * from tb_seller ignore index(idx_seller_name) where name = '小米科技';

explain select * from tb_seller ignore index(idx_seller_name_sta_addr) where name = '小米科技';
```

![使用SQL提示ignoreindex](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654719.png) 

#### 2.8.3 FORCE INDEX

为强制MySQL使用一个特定的索引，可在查询中使用 force index 作为hint 。 

``` SQL
-- 创建索引
create index idx_seller_address on tb_seller(address);

-- 没有索引
explain select * from tb_seller where address="北京市";
-- 没有索引
explain select * from tb_seller use index(idx_seller_address) where address="北京市";
-- 使用索引
explain select * from tb_seller force index(idx_seller_address) where address="北京市";
```

![使用SQL提示forceindex](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251654975.png) 

