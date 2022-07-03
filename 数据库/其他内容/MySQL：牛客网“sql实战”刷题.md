# MySQL：牛客网“sql实战”刷题

> 在对MySQL有一定了解后，抽空刷了一下 [牛客网](https://www.nowcoder.com/)上的 [数据库SQL 实战](https://www.nowcoder.com/ta/sql)，在此做一点小小的记录

### SQL1：查找最晚入职员工的所有信息

```mysql
select
    *
from 
    employees
where 
    employees.hire_date 
    ==(select max(hire_date) from employees);
```

### SQL2：查找入职员工时间排名倒数第三的员工所有信息

```mysql
select * from employees 
where hire_date = 
(
    select
    distinct hire_date  -- 这里的distinct需要注意
    from 
    employees
    order by 
    hire_date desc
    limit 2,1
);
```

### SQL3：查找各个部门当前领导当前薪水详情以及其对应部门编号dept_no

```mysql
select 
    s.emp_no, s.salary, s.from_date, s.to_date, d.dept_no
from 
    salaries s,
    dept_manager d
where
    d.to_date='9999-01-01' and s.to_date='9999-01-01' and d.emp_no = s.emp_no
order by 
    s.emp_no;
```

### SQL4：查找所有已经分配部门的员工的last_name和first_name

```mysql
select
    e.last_name, e.first_name, d.dept_no
from
    dept_emp d,
    employees e
where
    d.emp_no = e.emp_no;
```

### SQL5：查找所有员工的last_name和first_name以及对应部门编号dept_no

```mysql
select 
    e.last_name, e.first_name, d.dept_no
from 
    employees e
left join 
    dept_emp d
on 
    e.emp_no = d.emp_no;
```

### SQL7：查找薪水涨幅超过15次的员工号emp_no以及其对应的涨幅次数t

```mysql
-- 解法1：
select 
    distinct s.emp_no, cs.times as t
from 
    salaries as s,
    (
        select
            count(*) as times, salaries.emp_no as emp_no
        from
            salaries
        group by
            salaries.emp_no
    ) as cs
where
    cs.times > 15 and cs.emp_no = s.emp_no;


-- 解法2：
select 
    salaries.emp_no, count(*) as t
from
     salaries
group by salaries.emp_no having count(*)>15;
```
### SQL8：找出所有员工当前具体的薪水salary情况

```mysql
select
    distinct salaries.salary
from
    salaries
where 
    salaries.to_date='9999-01-01'
order by salaries.salary desc;
```
### SQL10：获取所有非manager的员工emp_no

```mysql
select
    e.emp_no
from 
    employees e
where e.emp_no not in (select d.emp_no from dept_manager d);
```
### SQL11：获取所有员工当前的manager

```mysql
-- 第一步：找出所有的manager
select 
    d.emp_no
from
    dept_manager d
where 
    d.to_date='9999-01-01';
    
-- 第二步：找出不是manage的员工
select
    e.emp_no, e.dept_no
from
    dept_emp e
where e.emp_no not in (
    select d.emp_no from dept_manager d where d.to_date='9999-01-01'
);

-- 最后：再根据e.emp_no, e.dept_no找出其manager对应的emp_no
select
    ed.emp_no, d.emp_no
from 
    dept_manager d,
    (select
        e.emp_no, e.dept_no
    from
        dept_emp e
    where 
        e.emp_no not in (select d.emp_no from dept_manager d where d.to_date='9999-01-01')) as ed
where
    ed.dept_no = d.dept_no and d.to_date='9999-01-01';
```
### SQL12：获取所有部门中当前员工薪水最高的相关信息

```mysql
-- 取各部门中薪水最高
SELECT
	de.dept_no, de.emp_no, MAX(s.salary) AS max_salary
FROM
	dept_emp de INNER JOIN salaries s ON de.emp_no=s.emp_no
WHERE
	de.to_date='9999-01-01' AND s.to_date='9999-01-01'
GROUP BY
	de.dept_no;
-- 错误代码：!!!GROUP BY 默认取非聚合的第一条记录！！！！！！
-- 因此进行MAX(s.salary)会出现数据不匹配的情况出现

-- 方式1：
-- 第一步：取各部门中薪水最高
SELECT
	de.dept_no, de.emp_no, MAX(s.salary) AS max_salary
FROM
	dept_emp de INNER JOIN salaries s ON de.emp_no=s.emp_no
WHERE
	de.to_date='9999-01-01' AND s.to_date='9999-01-01'
GROUP BY
     de.dept_no;
-- 第二补：进行后续匹配
select
    de.dept_no, de.emp_no, tab.max_salary
from
    dept_emp de, salaries s,
    (SELECT
        de.dept_no, de.emp_no, MAX(s.salary) AS max_salary
    FROM
        dept_emp de INNER JOIN salaries AS s ON de.emp_no=s.emp_no
    WHERE
        de.to_date='9999-01-01' AND s.to_date='9999-01-01'
    GROUP BY
        de.dept_no) as tab
where
    tab.max_salary = s.salary
    and s.emp_no = de.emp_no
    and tab.dept_no = de.dept_no
    and de.to_date='9999-01-01' and s.to_date='9999-01-01'
order by
    de.dept_no;
    

-- 方式2：和上述方式1大同小异
SELECT 
	d.dept_no, d.emp_no,  sub_tab.max_salary
FROM 
	dept_emp d, salaries s,
	(SELECT
		d.`dept_no`, MAX(s.salary) AS max_salary
	FROM 
		dept_emp d INNER JOIN salaries s
	ON
		d.to_date = '9999-01-01'
		AND s.to_date = '9999-01-01'
		AND d.`emp_no` = s.`emp_no`
	GROUP BY 
		d.`dept_no`) AS sub_tab
WHERE 
	d.dept_no = sub_tab.dept_no 
	AND s.salary = sub_tab.max_salary 
	AND d.emp_no = s.emp_no 
	AND d.to_date = '9999-01-01' 
	AND s.to_date = '9999-01-01'
ORDER BY d.dept_no;
```
### SQL15：查找employees表所有emp_no为奇数

```mysql
SELECT 
	*
FROM 
	employees
WHERE 
	employees.`last_name` != "Mary" AND employees.`emp_no` % 2=1
ORDER BY
	employees.`hire_date` DESC;
```
### SQL16：统计出当前各个title类型对应的员工当前薪水对应的平均工资

```mysql
SELECT 
	t.title, AVG(s.salary)
FROM
	titles t
JOIN
	salaries s
ON
	t.emp_no = s.emp_no
WHERE
	t.to_date='9999-01-01' AND s.to_date='9999-01-01'
GROUP BY
	t.title;
```
### SQL17：获取当前薪水第二多的员工的emp_no以及其对应的薪水salary

```mysql
-- 最高的salary没有重复的解法：
SELECT
	s.emp_no, s.salary
FROM
	salaries s
WHERE 
	s.to_date='9999-01-01'
GROUP BY
	s.emp_no
ORDER BY
	s.salary DESC
LIMIT 1,1;

-- 更加标准的解法：最高salary重复也没关系
select
    s.emp_no, s.salary
from
    salaries s
where
    s.salary = 
    (select distinct s.salary from salaries s where s.to_date = '9999-01-01' order by s.salary desc limit 1, 1);
```
### SQL18：查找当前薪水排名第二多的员工编号emp_no

```mysql
-- 解法1：题目要求是不能使用order by完成
-- 第一步：先找出最大薪水
SELECT
	MAX(salaries.salary) AS max_salary
FROM
	salaries
WHERE
	salaries.to_date='9999-01-01'
	
-- 第二步：找出第二大薪水
SELECT
	MAX(s.salary) AS second_max
FROM
	salaries s,
	(SELECT
		MAX(salaries.salary) AS max_salary
	FROM
		salaries
	WHERE
		salaries.to_date='9999-01-01') AS ss
WHERE
	s.salary != max_salary AND s.to_date='9999-01-01';
	
-- 最后：对应的员工信息	
SELECT 
	e.emp_no, s.salary, e.last_name, e.first_name
FROM
	employees e,
	salaries s,
	(SELECT
		MAX(s.salary) AS second_max
	FROM
		salaries s,
		(SELECT
			MAX(salaries.salary) AS max_salary
		FROM
			salaries
		WHERE
			salaries.to_date='9999-01-01') AS ss
	WHERE
		s.salary != max_salary AND s.to_date='9999-01-01') ss
WHERE
	s.salary = ss.second_max AND s.emp_no = e.emp_no;
	
-- 解法2：使用order by，但是无法通过，因为题目要求无法使用orderby
-- 第一步：找薪水第二大的
select
    s.emp_no, s.salary
from
    salaries s
where
    s.salary = 
    (select distinct s.salary from salaries s where s.to_date = '9999-01-01' order by s.salary desc limit 1, 1);
    
-- 第二步：根据薪水第二大的会去查找对应的员工
select 
    e.emp_no, tab.salary, e.last_name, e.first_name
from
    (select
        s.emp_no, s.salary as salary
    from
        salaries s
    where
        s.salary = 
        (select distinct s.salary from salaries s where s.to_date = '9999-01-01' order by s.salary desc limit 1, 1)
    ) as tab,
    employees as e
where
    tab.emp_no = e.emp_no;
```
### SQL19：查找所有员工的last_name和first_name以及对应的dept_name

```mysql
-- 第一步：先合并department表和dept_emp表
SELECT
	d.dept_name, de.emp_no
FROM
	dept_emp de
INNER JOIN 
	departments d
ON
	de.dept_no = d.dept_no;
	
-- 第二步：加上员工信息
SELECT
	e.last_name, e.first_name, d.dept_name
FROM 
	employees e
LEFT JOIN
	(SELECT
		d.dept_name, de.emp_no
	FROM
		dept_emp de
	INNER JOIN 
		departments d
	ON
		de.dept_no = d.dept_no) d
ON
	e.emp_no = d.emp_no;
```
### SQL21：查找所有员工自入职以来的薪水涨幅情况

```mysql
-- 第一步：查出入职薪水
SELECT
	e.`emp_no`, s.`salary` AS hire_salary
FROM
	employees e,
	salaries s
WHERE
	e.`emp_no` = s.`emp_no` AND e.`hire_date` = s.`from_date`;
	
-- 第二步：最终薪水
SELECT
	e.`emp_no`, s.`salary` AS last_salary
FROM
	employees e,
	salaries s
WHERE
	e.`emp_no` = s.`emp_no` AND s.`to_date`='9999-01-01';

-- 第三步：结合上述两张表，给出最终结果
SELECT
	salary_1.emp_no, (salary_2.last_salary - salary_1.hire_salary) AS growth
FROM 
	(SELECT
		e.`emp_no`, s.`salary` AS hire_salary
	FROM
		employees e,
		salaries s
	WHERE
		e.`emp_no` = s.`emp_no` AND e.`hire_date` = s.`from_date`) AS salary_1,
	(SELECT
		e.`emp_no`, s.`salary` AS last_salary
	FROM
		employees e,
		salaries s
	WHERE
		e.`emp_no` = s.`emp_no` AND s.`to_date`='9999-01-01') AS salary_2
WHERE 
	salary_1.emp_no = salary_2.emp_no
ORDER BY
	growth;
```
### SQL22：统计各个部门的工资记录数

```mysql
SELECT
	d.`dept_no`, d.`dept_name`, COUNT(s.`salary`) AS `sum`
FROM 
	dept_emp de 
JOIN
	salaries s
ON
	de.`emp_no` = s.`emp_no`
JOIN
	departments d
ON
	de.`dept_no` = d.`dept_no`
GROUP BY
	d.`dept_no`;
```
### SQL23：对所有员工的当前薪水按照salary进行按照1-N的排名

```mysql
-- 标准答案，参考了讨论区
SELECT 
	s1.emp_no, s1.salary, 
	COUNT(DISTINCT s2.salary) AS rank
FROM 
	salaries AS s1, 
	salaries AS s2
WHERE 
	s1.to_date = '9999-01-01' AND s2.to_date = '9999-01-01' 
	AND s1.salary <= s2.salary
GROUP BY 
	s1.emp_no
ORDER BY 
	s1.salary DESC, s1.emp_no ASC;
	
-- 本题的精髓在于：
-- s1.salary <= s2.salary，
-- 意思是在输出s1.salary的情况下，有多少个s2.salary大于等于s1.salary
-- 最后再利用COUNT(DISTINCT s2.salary)去重
```
### SQL24：获取所有非manager员工当前的薪水情况

```mysql
-- 写法1：
SELECT
	de.`dept_no`, e.`emp_no`, s.`salary`
FROM 
	employees e
JOIN
	dept_emp de
ON
	e.`emp_no` = de.`emp_no` 
JOIN
	salaries s
ON
	e.`emp_no` = s.`emp_no` AND s.`to_date`='9999-01-01'
WHERE
	e.`emp_no` NOT IN (
	SELECT 
		dm.`emp_no`
	FROM 
		dept_manager dm
	WHERE
		dm.`to_date`='9999-01-01');
		
-- 写法2：注意多表联查的问题！！！ 最好还是用join连接表
SELECT
	de.`dept_no`, e.`emp_no`, s.`salary`
FROM
	dept_emp de,
	employees e,
	salaries s,
	(SELECT 
		dm.`emp_no`
	FROM 
		dept_manager dm
	WHERE
		dm.`to_date`='9999-01-01') AS m
WHERE
	e.`emp_no` = de.`emp_no` 
	AND e.`emp_no` = s.`emp_no` 
	AND s.`to_date`='9999-01-01'
	AND (e.`emp_no` NOT IN (m.`emp_no`))
```
### SQL25：获取员工其当前的薪水比其manager当前薪水还高的相关信息

```mysql
-- 第一步：找出manager的薪水
SELECT
	dm.`emp_no`, dm.`dept_no`, s.`salary`
FROM
	dept_manager dm,
	salaries s
WHERE
	dm.`emp_no` = s.`emp_no`
	AND dm.`to_date`='9999-01-01'
	AND s.`to_date`='9999-01-01';

-- 第二步：将dept_emp表和salaries表相连，即查出所有人的薪水
SELECT
	de.`emp_no`, de.`dept_no`, s.`salary`
FROM
	dept_emp de
JOIN
	salaries s
ON
	de.`emp_no` = s.`emp_no` 
	AND de.`to_date`= '9999-01-01' 
	AND s.`to_date`='9999-01-01';

-- 第三步：融合上述两张表
SELECT
	e_tab.emp_no AS emp_no, m_tab.emp_no AS manager_no, e_tab.salary AS emp_salary, m_tab.salary AS manager_salary
FROM 
	(SELECT
		dm.`emp_no`, dm.`dept_no`, s.`salary`
	FROM
		dept_manager dm,
		salaries s
	WHERE
		dm.`emp_no` = s.`emp_no`
		AND dm.`to_date`='9999-01-01'
		AND s.`to_date`='9999-01-01') AS m_tab,  -- manager的薪水
	(SELECT
		de.`emp_no`, de.`dept_no`, s.`salary`
	FROM
		dept_emp de
	JOIN
		salaries s
	ON
		de.`emp_no` = s.`emp_no` 
		AND de.`to_date`= '9999-01-01' 
		AND s.`to_date`='9999-01-01'
		) AS e_tab	-- 所有人的薪水表
WHERE
	m_tab.dept_no = e_tab.dept_no  -- 1.部门相同
	AND m_tab.emp_no != e_tab.emp_no  -- 2.不是部门manager
	AND m_tab.salary < e_tab.salary;  -- 3.员工的薪水比你manager高
```
### SQL26：汇总各个部门当前员工的title类型的分配数目

```mysql
SELECT
	d.`dept_no`, d.`dept_name`, t.`title`, COUNT(t.`title`)
FROM  
	dept_emp de
JOIN
	titles t
ON
	de.`emp_no` = t.`emp_no`
	AND de.`to_date`='9999-01-01'
	AND t.`to_date`='9999-01-01'
JOIN 
	departments d
ON
	d.`dept_no`=de.`dept_no`
GROUP BY
	d.`dept_no`, t.`title`  -- 这里是关键
```
### SQL28：查找描述信息中包括robot的电影对应的分类名称以及电影数目

```mysql
-- 第一步：先将找出含有robot描述信息的电影
SELECT
	f.`film_id`
FROM 
	film f
WHERE
	f.`description` LIKE "%robot%";
	
-- 第二步：先将找出含有robot描述信息的电影+其类别id
SELECT
	f.`film_id`, fc.`category_id`
FROM 
	film f
JOIN
	film_category fc
ON
	f.`film_id` = fc.`film_id`
WHERE
	f.`description` LIKE "%robot%";
	
-- 第三步：先将找出含有robot描述信息的电影+其类别+类别名称
SELECT
	f.`film_id`, fc.`category_id`, c.`name`
FROM 
	film f
JOIN
	film_category fc
ON
	f.`film_id` = fc.`film_id`
JOIN
	category c
ON
	fc.`category_id` = c.`category_id`
WHERE
	f.`description` LIKE "%robot%";
	
-- 最终：分类包含电影总数量(count(film_category.category_id))>=5部
SELECT 
	fcn.`name`, COUNT(DISTINCT fcn.`film_id`)  -- 注意这个DISTINCT！！！
FROM
	film_category fc,
	(SELECT
		f.`film_id`, fc.`category_id`, c.`name`
	FROM 
		film f
	JOIN
		film_category fc
	ON
		f.`film_id` = fc.`film_id`
	JOIN
		category c
	ON
		fc.`category_id` = c.`category_id`
	WHERE
		f.`description` LIKE "%robot%") fcn
WHERE
	fc.`category_id` = fcn.`category_id`
GROUP BY
	fc.`category_id`
HAVING 
	COUNT(fc.`film_id`)>=5;  -- 该分类的部数要大于5
```
### SQL29：使用join查询方式找出没有分类的电影id以及名称

```mysql
-- 第一步：根据电影id找出其对应的类别id，用左外连接，然后再一个左外连接查出对应的类别名称
SELECT
	f.`film_id`, f.`title`, c.`name`
FROM 
	film f
LEFT JOIN
	film_category fc
ON
	f.`film_id` = fc.`film_id`
LEFT JOIN
	category c
ON
	fc.`category_id` = c.`category_id`

-- 第二步：找名字是null的就可以了
SELECT 
	t.`film_id`, t.`title`
FROM
	(SELECT
		f.`film_id`, f.`title`, c.`name`
	FROM 
		film f
	LEFT JOIN
		film_category fc
	ON
		f.`film_id` = fc.`film_id`
	LEFT JOIN
		category c
	ON
		fc.`category_id` = c.`category_id`) AS t
WHERE
	t.name IS NULL;
```

### SQL30：使用子查询的方式找出属于Action分类的所有电影对应的title,description

```mysql
-- 查询1： 根据电影id查出电影对应的类别id
SELECT f.`title`, f.`description`, fc.`category_id` FROM film AS f JOIN film_category fc ON f.`film_id`= fc.`film_id`;

-- 查询2：找出类别名字为action的类别id
SELECT c.`category_id` FROM category c WHERE c.`name`="Action"

-- 查询3：融合上述查询，条件就是id相同，即action类别对应的id
SELECT 
	fc.title, fc.description
FROM 
	(SELECT f.`title`, f.`description`, fc.`category_id` FROM  film AS f JOIN film_category fc ON f.`film_id`= fc.`film_id`) AS fc,
	(SELECT c.`category_id` FROM category c WHERE c.`name`="Action") AS c
WHERE 
	fc.category_id = c.category_id;
```

### SQL32：将employees表的所有员工的last_name和first_name拼接起来作为Name，中间以一个空格区分

```mysql
-- sqllite,字符串拼接为 || 符号，不支持concat函数，mysql支持concat函数
SELECT 
	e.`last_name` || " " || e.`first_name`
FROM
	employees e;
```

### SQL33：创建一个actor表，包含如下列信息

```mysql
CREATE TABLE actor(
	actor_id smallint(5) NOT NULL,
	first_name varchar(45)	NOT NULL,
	last_name varchar(45) NOT NULL,
	last_update timestamp NOT NULL DEFAULT (datetime('now','localtime')),  -- 这里不多加个括号无法通过
	PRIMARY KEY (actor_id)
)
```

### SQL34：批量插入数据

```mysql
INSERT INTO actor (actor_id, first_name, last_name, last_update)
VALUES (1, "PENELOPE", "GUINESS", "2006-02-15 12:34:33"),
	(2, "NICK", "WAHLBERG", "2006-02-15 12:34:33");
```

### SQL35：批量插入数据,如果数据已经存在，请忽略，不使用replace操作

```mysql
insert or ignore into actor
values(3,'ED','CHASE','2006-02-15 12:34:33');
```

### SQL36：创建一个actor_name表，将actor表中的所有first_name以及last_name导入改表

```mysql
create table if not exists actor_name
as select first_name, last_name from actor;
```

### SQL37：对first_name创建唯一索引uniq_idx_firstname，对last_name创建普通索引idx_lastname

```mysql
-- 创建唯一索引
CREATE UNIQUE INDEX uniq_idx_firstname ON actor(first_name);
-- 创建普通索引
CREATE INDEX idx_lastname ON actor(last_name);
```

### SQL38：针对actor表创建视图actor_name_view

```mysql
CREATE VIEW actor_name_view(first_name_v, last_name_v)
AS SELECT a.first_name, a.last_name  FROM actor a;
```

### SQL39：针对上面的salaries表emp_no字段创建索引idx_emp_no，查询emp_no为10005,

```mysql
-- SQLite中，使用 INDEXED BY 语句进行强制索引查询
SELECT * FROM salaries INDEXED BY idx_emp_no WHERE emp_no=10005;

-- MySQL中，使用 FORCE INDEX 语句进行强制索引查询，可参考：
SELECT * FROM salaries FORCE INDEX idx_emp_no WHERE emp_no = 10005
```

### SQL40：在last_update后面新增加一列名字为create_date

```mysql
ALTER TABLE actor ADD create_date datetime NOT NULL DEFAULT '0000-00-00 00:00:00';
```

### SQL41：构造一个触发器audit_log，在向employees表中插入一条数据的时候，触发插入相关的数据到audit中

```mysql
CREATE TRIGGER 
    audit_log
AFTER INSERT
ON 
    employees_test
BEGIN
	INSERT INTO audit (EMP_no, NAME) VALUES (new.id, new.name);
END;
```

### SQL42：删除emp_no重复的记录，只保留最小的id对应的记录。

```mysql
-- 第一步：找最小的id记录
SELECT
	MIN(t.id) AS id
FROM 
	titles_test t
GROUP BY
	t.emp_no;
	
-- 第二步：进行删除
DELETE FROM 
    titles_test 
WHERE 
    titles_test.id 
NOT IN (
    SELECT 
        tab.id 
    FROM (SELECT MIN(t.id) AS id FROM  titles_test t GROUP BY t.emp_no) AS tab
);  -- 这里的手法！
```

### SQL43：将所有to_date为9999-01-01的全部更新为NULL,且

```mysql
UPDATE 
	titles_test
SET
	from_date = '2001-01-01', to_date=NULL
WHERE
	to_date='9999-01-01';
```

### SQL44：将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，使用replace实现。

```mysql
replace into titles_test select 5, 10005, title, from_date, to_date FROM titles_test WHERE id=5;
```

### SQL45：将titles_test表名修改为titles_2017

```mysql
-- mysql 可以
RENAME table titles_test TO titles_2017;

-- 在sqllite中
alter table titles_test rename to titles_2017;
```

### SQL46：在audit表上创建外键约束，其emp_no对应employees_test表的主键id

```mysql
-- SQLite只能首先删除表，然后再新建表的时候添外键约束。
DROP TABLE audit;
CREATE TABLE audit(
    EMP_no INT NOT NULL,
    create_date datetime NOT NULL,
	FOREIGN KEY(EMP_no) REFERENCES employees_test(ID)
);

-- mysql实现
ALTER TABLE audit ADD FOREIGN KEY (emp_no) REFERENCES employees_test (id);
```

### SQL48：将所有获取奖金的员工当前的薪水增加10%

```mysql
UPDATE salaries SET salary=(salary*1.1)
WHERE
	emp_no IN (SELECT emp_no FROM emp_bonus) AND to_date='9999-01-01';
```

### SQL50：将employees表中的所有员工的last_name和first_name通过(')连接起来。

```mysql
-- SQLite数据库中，只支持用连接符号"||"来连接字符串，不支持用函数连接
SELECT 
	last_name || "'" ||first_name AS NAME
FROM
	employees;
	
-- mysql库函数：最外层的两个单引号表示字符串，从左往右数第二个是转义符，第三个是题目需要拼接的单引号
select 
	concat(last_name, '''', first_name) as name
from 
	employees;
```

### SQL51：查找字符串'10,A,B' 中逗号','出现的次数cnt。

```mysql
-- 技巧题，先用任意两个字符替换,  然后计算长度  最后相减
SELECT (LENGTH(REPLACE('10,A,B', ',', '--')) - LENGTH('10,A,B')) AS cnt;
```

### SQL52：获取Employees中的first_name，查询按照first_name最后两个字母，按照升序进行排列

```mysql
-- mysql 方式
SELECT 
	first_name
FROM
	employees
ORDER BY
	RIGHT(first_name, 2);
	
-- SQLite
SELECT 
	first_name
FROM
	employees
ORDER BY
	SUBSTR(first_name, -2, 2);
```

### SQL53：按照dept_no进行汇总，属于同一个部门的emp_no按照逗号进行连接，结果给出dept_no以及连接出的结果employees

```mysql
-- GROUP_CONCAT 实现分组聚合
SELECT
	dept_no, GROUP_CONCAT(emp_no)
FROM
	dept_emp
GROUP BY
	dept_no;
```

### SQL54：查找排除当前最大、最小salary之后的员工的平均工资avg_salary

```mysql
-- 第一步：找最大最小工资
SELECT 
	MAX(salary) max_s, MIN(salary) 
AS 
	min_s 
FROM 
	salaries 
WHERE 
	to_date = '9999-01-01';
	
-- 第二步：排除了之后的平均工资
SELECT
	AVG(s.salary) AS avg_salary
FROM
	salaries s,
	(SELECT MAX(salary) max_s, MIN(salary) AS min_s FROM salaries WHERE to_date = '9999-01-01') mm
WHERE
	s.to_date = '9999-01-01' AND s.`salary`!=mm.max_s AND s.`salary`!=min_s;
```

### SQL55：分页查询employees表，每5行一页，返回第2页的数据

```mysql
SELECT
	*
FROM 
	employees
LIMIT 5,5;  -- 从第5条数据开始的后5条数据
```

### SQL57：使用含有关键字exists查找未分配具体部门的员工的所有信息。

```mysql
-- 用exists关键字
SELECT
	*
FROM 
	employees e
WHERE NOT EXISTS
	(SELECT emp_no FROM dept_emp de WHERE e.`emp_no` = de.`emp_no`);


-- 用not in也可以实现
SELECT 
	*
FROM
	employees e
WHERE
	e.`emp_no` NOT IN (SELECT emp_no FROM dept_emp);
```

### SQL59：获取有奖金的员工相关信息

```mysql
SELECT
	e.`emp_no`, e.`first_name`, e.`last_name`, eb.`btype`, s.`salary`, (s.`salary`*eb.`btype` / 10.0) AS bonus
FROM 
	emp_bonus eb
JOIN 
	employees e
ON
	eb.`emp_no` = e.`emp_no`
JOIN
	salaries s
ON
	eb.`emp_no` = s.`emp_no`
WHERE
	s.`to_date`='9999-01-01';
```

### SQL60：统计salary的累计和running_total

```mysql
SELECT 
	s1.`emp_no`, s1.`salary`, SUM(s2.`salary`)
FROM 
	salaries s1,
	salaries s2
WHERE 
	s1.`emp_no` >= s2.`emp_no`
	AND s1.to_date = '9999-01-01'
	AND s2.to_date = '9999-01-01'
GROUP BY s1.`emp_no`;
```

### SQL61：对于employees表中，给出奇数行的first_name

```mysql
SELECT
	e1.`first_name`
FROM
	employees e1
WHERE
	(SELECT COUNT(*) FROM employees e2 WHERE e1.`first_name` >= e2.`first_name`) % 2 =1;
```

### SQL62：出现三次以上相同积分的情况

```mysql
select
    number
from 
    grade
group by
    number
having
    count(number) >=3;
```

### SQL63：刷题通过的题目排名

```mysql
select
    pn1.id, pn1.number, count(distinct(pn2.number)) as rank
from
    passing_number pn1,
    passing_number pn2
where
    pn1.number <= pn2.number
group by
    pn1.id
order by
    pn1.number desc, pn1.id asc;
```

### SQL64：找到每个人的任务

```mysql
select
    p.id, p.name, t.content
from 
    person p
left join
    task t
on 
    p.id = t.person_id
order by
    p.id asc;
```

### SQL65：异常的邮件概率

```mysql
-- 进行计算
select
	-- 后续的概率计算
	-- 1. round 四舍五入函数，保留3位小数
	-- 2. sum 计算失败的次数，用了case语句
	-- 3. count 计算总的发送次数，通过主键id计算
    e.date, round(sum(case e.type when 'completed' then 0 else 1 end) * 1.0 / count(e.id), 3) as p  
from
    email e,  -- 邮件表
    (select id from user where is_blacklist=1) ub  -- 黑名单用户
where
	-- 条件：
	-- 1.发送者不是黑名单
	-- 2.接受者也不再黑名单中
    e.send_id not in (ub.id) and e.receive_id not in (ub.id)
group by
    e.date  -- 按照日期进行分组
order by
    e.date;  -- 按照日期进行排序
```

### SQL67：牛客每个人最近的登录日期(一)

```mysql
select
    max(date) as d
from
    login
group by
    user_id
order by 
    user_id;
```

### SQL68：牛客每个人最近的登录日期(二)

```mysql
select
    u.name as u_n, c.name as c_n, max(l.date)
from
    login l
join
    user u
on
    l.user_id = u.id
join
    client c
on
    l.client_id = c.id
group by
    l.user_id
order by
    u.name;
```

### SQL69：牛客每个人最近的登录日期(三)

```mysql
-- 第一次登陆的时间
SELECT 
    user_id, MIN(DATE) AS first_login
FROM
    login
GROUP BY
    user_id;
    
-- 总人数计算
SELECT COUNT(DISTINCT user_id) FROM login;

-- 结合一下
SELECT
     ROUND(COUNT(*)*1.0 / (SELECT COUNT(DISTINCT user_id) FROM login), 3)
FROM 
     login l
JOIN
    (SELECT 
        user_id, MIN(DATE) AS first_login
    FROM
        login
    GROUP BY
        user_id) AS fl
ON
    l.user_id = fl.user_id AND l.date != fl.first_login
WHERE
	date(fl.first_login, '+1 day')=l.date;  -- 这里筛选了满足第一天登陆后第二天也登陆的记录
```

### SQL70：牛客每个人最近的登录日期(四)

```mysql
-- 第一步：总共有几天
SELECT
	DISTINCT(DATE)
FROM 
	login;
	
-- 第二步：找出用户第一次登录日期
SELECT
    user_id, MIN(DATE) AS first_date
FROM
    login
GROUP BY
    user_id;
    
-- 结合上述两步
SELECT 
	l1.real_date, IFNULL(COUNT(l2.user_id), 0)  -- count就计算新用户数
FROM 
	(SELECT
		DISTINCT(DATE) AS real_date
	FROM 
		login) AS l1  -- 天数表
LEFT JOIN
	(SELECT
	    user_id, MIN(DATE) AS first_date
	FROM
	    login
	GROUP BY
	    user_id) AS l2  -- 新用户登录表
ON
	l1.real_date = l2.first_date  -- 合并的条件就是日期相同
GROUP BY
	l1.real_date
```

### SQL71：牛客每个人最近的登录日期(五)

```mysql
SELECT
	 tab1.real_date as date, ROUND(IFNULL(IFNULL(tab2.last_num, 0) * 1.0 / IFNULL(tab1.all_num, 0), 0), 3) AS p
FROM
	(SELECT 
		l1.real_date, IFNULL(COUNT(l2.user_id), 0) AS all_num
	FROM 
		(SELECT
			DISTINCT(DATE) AS real_date
		FROM 
			login) AS l1  -- 总天数表
	LEFT JOIN
		(SELECT
		    user_id, MIN(DATE) AS first_date
		FROM
		    login
		GROUP BY
		    user_id) AS l2  -- 用户最早登陆表
	ON
		l1.real_date = l2.first_date
	GROUP BY
		l1.real_date) tab1  -- 记录了该天所有新用户登陆的数据
LEFT JOIN
	(SELECT
	     l1.date, COUNT(l1.user_id) AS last_num
	FROM 
	     login l1
	JOIN
	    (SELECT 
			user_id, MIN(DATE) AS first_date
	    FROM
			login
	    GROUP BY
			user_id) AS l2  -- 用户最早登陆表
	ON
	    l1.user_id = l2.user_id AND l1.date != l2.first_date
	WHERE
		date(l2.first_date, '+1 day')=l1.date
	GROUP BY
		l1.date) tab2  -- tab2 记录了满足第一天登陆后第二天也登陆的用户
ON
	date(tab1.real_date, '+1 day') = tab2.date;
```

### SQL72：牛客每个人最近的登录日期(六)

```mysql
-- 关键在此！ 
SELECT
	pn1.user_id, SUM(pn2.number) AS nums, pn1.date AS DATE
FROM
	passing_number pn1,
	passing_number pn2
WHERE
	pn1.user_id = pn2.user_id
	AND pn1.date >= pn2.date
GROUP BY
	pn1.user_id, pn1.date;
	
-- 根据上面的表去匹配其他信息
SELECT
    u.name AS u_n, c.name AS c_n, tb.date AS DATE, tb.nums AS ps_num
FROM
    (SELECT
	pn1.user_id, SUM(pn2.number) AS nums, pn1.date AS DATE
	FROM
		passing_number pn1,
		passing_number pn2
	WHERE
		pn1.user_id = pn2.user_id
		AND pn1.date >= pn2.date
	GROUP BY
		pn1.user_id, pn1.date) AS tb
JOIN 
    login l
ON
    tb.user_id = l.user_id AND tb.date = l.date
JOIN
    USER u
ON
    tb.user_id = u.id
JOIN
    CLIENT c
ON
    l.client_id = c.id
ORDER BY
    tb.date ASC, u.name ASC;
```

### SQL72：考试分数(一)

```mysql
select
    job, round(avg(score), 3) as score
from
    grade
group by
    job
order by
    score desc;
```

### SQL73：考试分数(二)

```mysql
-- 第一步：找出平均成绩
select
    job, avg(score)
from
    grade
group by
    score;
   
-- 第二步：找出大于平均分数的数据
SELECT
    g1.id, g1.job, g1.score AS score
FROM
    grade g1
LEFT JOIN
    (SELECT
        job, AVG(score) AS avg_score
    FROM
        grade
    GROUP BY
        job) AS g2
ON 
    g1.job = g2.job
WHERE
    g1.score > g2.avg_score;
```

### SQL74：考试分数(三)

```mysql
-- 合并一下score表和language表
SELECT
		g.id AS id, 
		l.name AS NAME, 
		g.score AS score, 
		dense_rank() over (PARTITION BY g.language_id ORDER BY g.score DESC) AS score_order  -- 窗口函数：序号函数，用于并列排序的
FROM
	grade g
JOIN
	LANGUAGE l
ON
	g.language_id = l.id;
	

-- 根据上表给出结果
SELECT
	t.id, t.name, t.score
FROM
	(SELECT
		g.id AS id, l.name AS NAME, g.score AS score, dense_rank() over (PARTITION BY g.language_id ORDER BY g.score DESC) AS score_order
	FROM
		grade g
	JOIN
		LANGUAGE l
	ON
		g.language_id = l.id) t
WHERE
	t.score_order <=2  -- 在经过窗口函数排序之后的结果
ORDER BY
	t.name, t.score DESC, t.id;
```

### SQL75：考试分数(四)

```mysql
-- 表1：使用窗口函数按照job分组对成绩进行降序排序
SELECT
	*, 
	-- 窗口函数：用于顺序排序，并列的时候也是顺序排序
	row_number() over (PARTITION BY job ORDER BY score DESC) AS score_order
FROM
	grade;
	
-- 表2：按照job进行分组，计算每个job中中间记录数，奇数时就是一个值，偶数是为.5小数
SELECT 
	job, 
	(COUNT(*)+1)*1.0/2 AS middle 
FROM 
	grade 
GROUP BY 
	job;

-- 对表1和表2进行合并，给出最终结果
SELECT
	t1.job,
	cast(t2.middle as integer) AS START,  -- 奇/偶都一样，即向下取整
	(CASE (t2.middle-cast(t2.middle as integer)) WHEN 0 THEN cast(t2.middle as integer) ELSE cast(t2.middle as integer)+1 END) AS END  
	-- 上述对奇/偶进行区别，通过case when ...then ...else ..end
FROM
	(SELECT
	*, row_number() over (PARTITION BY job ORDER BY score DESC) AS score_order
	FROM
		grade) AS t1,
	(SELECT job, (COUNT(*)+1)*1.0/2 AS middle FROM grade GROUP BY job) AS t2
WHERE
	t1.job = t2.job
GROUP BY
	t1.job;
```

### SQL76：考试分数(五)

```mysql
-- 表1：使用窗口函数按照job分组对成绩进行降序排序
SELECT
	*, 
	row_number() over (PARTITION BY job ORDER BY score DESC) AS score_order
FROM
	grade;
	
-- 表2：确定每个job中间的人数，奇数为1，偶数为2
SELECT
    job,
    cast((COUNT(id)+1)/2 as integer) AS START,
    (cast((COUNT(id)+1)/2 as integer) + (CASE COUNT(*)%2=1 WHEN 1 THEN 0 ELSE 1 END)) AS END
FROM
    grade
GROUP BY
    job;
    
-- 进行合并
SELECT 
	t1.id, t1.job, t1.score, t1.score_order
FROM
	(SELECT
		*, row_number() over (PARTITION BY job ORDER BY score DESC) AS score_order
	FROM
		grade) AS t1
JOIN
	(SELECT
        job,
        cast((COUNT(id)+1)/2 as integer) AS START,
        (cast((COUNT(id)+1)/2 as integer) + (CASE COUNT(*)%2=1 WHEN 1 THEN 0 ELSE 1 END)) AS END
    FROM
        grade
    GROUP BY
        job) AS t2
ON
	-- 合并条件：
	-- 1. job相同
	-- 2. 分数的排序为中间的值
	t1.job = t2.job AND (t1.score_order = t2.START OR t1.score_order = t2.END)
ORDER BY t1.id;
```

