# Spring：学习笔记-4

> 后端学习ING，在学完了Java语法部分，JavaWeb知识后，开始一套组合拳来带走经典框架SSM。
>
> 此文为第二拳：Spring学习笔记的 **4/4**。

## 1. Spring中的JdbcTemplate

### 1.1 JdbcTemplate概述

它是 spring 框架中提供的一个对象，是对原始 Jdbc API 对象的简单封装。 spring 框架为我们提供了很多的操作模板类。

操作关系型数据的：JdbcTemplate、HibernateTemplate

操作 nosql 数据库的：RedisTemplate

操作消息队列的：JmsTemplate

JdbcTemplate 在 `spring-jdbc-5.0.2.RELEASE.jar` 中，同时还需要导入一个与事务相关的包 `spring-tx-5.0.2.RELEASE.jar`

### 1.2 JdbcTemplate对象创建

可以参考它的源码，来一探究竟：  

```java
public JdbcTemplate() {
}

public JdbcTemplate(DataSource dataSource) {
    setDataSource(dataSource);
    afterPropertiesSet();
}

public JdbcTemplate(DataSource dataSource, boolean lazyInit) {
    setDataSource(dataSource);
    setLazyInit(lazyInit);
    afterPropertiesSet();
}
```

除了默认构造函数之外，**都需要提供一个数据源**。既然有set方法，依据之前学过的依赖注入，可以在配置文件中配置这些对象。  

### 1.3 Spring中配置数据源

**环境搭建：导包**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>
```

**编写Spring的配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

**配置数据源**

之前已经接触过了两个数据源， C3P0 和 DBCP。要想使用这两数据源都需要导入对应的依赖

**配置 C3P0 数据源**

导入到工程的 lib 目录。在 spring 的配置文件中配置：  

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///db2"></property>
    <property name="user" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

**配置 DBCP 数据源**

导入到工程的 lib 目录。在 spring 的配置文件中配置：  

```xml
<!-- 配置数据源 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql:// /db2"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

**配置 spring 内置数据源**（用了这个）

spring框架也提供了一个内置数据源，可以使用spring的内置数据源，它就在`spring-jdbc` 依赖中：  

```xml
<bean id="dataSource"
      class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql:///db2"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

**将数据库连接的信息配置到属性文件中**

```properties
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///db2
jdbc.username=root
jdbc.password=root
```

引入外部的属性文件，方式1：

```xml
<!-- 引入外部属性文件 -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:jdbc.properties"/>
</bean>
```

引入外部的属性文件，方式2：

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

### 1.4 JdbcTemplate增删改查操作

**前期准备**

```sql
-- 创建数据库：
create database db2;
use db2;

-- 创建表：
create table account(
    id int primary key auto_increment,
    name varchar(40),
    money float
)character set utf8 collate utf8_general_ci;  
```

**在 spring 配置文件中配置 JdbcTemplate**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 配置一个数据库的操作模板： JdbcTemplate -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///db2"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
</beans>
```

**最基本使用**

```java
public class JdbcTemplateDemo2 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作
        jt.execute("insert into account(name,money)values('eee',500)");
    }
}
```

**保存操作**

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:保存
        jt.update("insert into account(name,money)values(?,?)","fff",5000);
    }
}
```

**更新操作**

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:修改
        jt.update("update account set money = money-? where id = ?",300,6);
    }
}
```

**删除操作**

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:删除
        jt.update("delete from account where id = ?",6);
    }
}
```

**查询所有操作**

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:查询所有
        List<Account> accounts = jt.query("select * from account where money > ? ", new AccountRowMapper(), 500);
        
        for(Account o : accounts){
            System.out.println(o);
        }
    }
}

public class AccountRowMapper implements RowMapper<Account>{
    
    @Override
    public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
        Account account = new Account();
        account.setId(rs.getInt("id"));
        account.setName(rs.getString("name"));
        account.setMoney(rs.getFloat("money"));
        return account;
    }
}
```

> 上述代码中，关于 `new AccountRowMapper()`；采用spring的框架，也可以直接采用：`new BeanPropertyRowMapper<Account>(Account.class)`，不需要直接实现类，即直接将上述第二个参数替换一下即可。

**查询一个操作**：

使用 RowMapper 的方式：常用的方式

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:查询一个
        List<Account> as = jt.query("select * from account where id = ? ",
                                    new AccountRowMapper(), 55);
        System.out.println(as.isEmpty()?"没有结果":as.get(0));
    }
}
```

使用 ResultSetExtractor 的方式：不常用的方式

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作:查询一个
        Account account = jt.query("select * from account where id = ?", 
                                   new AccountResultSetExtractor(),3);
        System.out.println(account);
    }
}
```

**查询返回一行一列操作**

```java
public class JdbcTemplateDemo3 {
    public static void main(String[] args) {
        //1.获取 Spring 容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.根据 id 获取 bean 对象
        JdbcTemplate jt = (JdbcTemplate) ac.getBean("jdbcTemplate");
        //3.执行操作
        //  查询返回一行一列：使用聚合函数，在不使用 group by 字句时，都是返回一行一列。最用的就是分页中获取总记录条数
        Integer total = jt.queryForObject("select count(*) from account where money > ?", Integer.class, 500);
        System.out.println(total);
    }
}
```

### 1.5 Dao中使用 JdbcTemplate

**准备实体类**

```java
// 账户的实体
@Data
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Float money;
}
```

#### 1.5.1 方式1：在 dao 中定义 JdbcTemplate

**第一种方式：在 dao 中定义 JdbcTemplate**

```java
// 账户的接口
public interface IAccountDao {

    // 根据 id 查询账户信息
    Account findAccountById(Integer id);

    // 根据名称查询账户信息
    Account findAccountByName(String name);

    // 更新账户信息
    void updateAccount(Account account);
}

// 账户的持久层实现类:此版本的 dao，需要给 dao 注入 JdbcTemplate
public class AccountDaoImpl implements IAccountDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Account findAccountById(Integer id) {
        List<Account> list = jdbcTemplate.query("select * from account where id = ?",new AccountRowMapper(),id);
        return list.isEmpty()?null:list.get(0);
    }
    
    @Override
    public Account findAccountByName(String name) {
        List<Account> list = jdbcTemplate.query("select * from account where name = ?",new AccountRowMapper(),name);
        if(list.isEmpty()){
            return null;
        }
        if(list.size()>1){
            throw new RuntimeException("结果集不唯一，不是只有一个账户对象");
        }
        return list.get(0);
    }
    
    @Override
    public void updateAccount(Account account) {
        jdbcTemplate.update("update account set money = ? where id = ?",account.getMoney(),account.getId());
    }
}
```

**配置文件**  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 配置一个 dao -->
    <bean id="accountDao" class="cn.xyc.dao.impl.AccountDaoImpl">
        <!-- 注入 jdbcTemplate -->
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>
    
    <!-- 配置一个数据库的操作模板： JdbcTemplate -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///db2"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
</beans>
```

这里有个小问题。就是当 dao 有很多时，每个 dao 都有一些重复性的代码。下面就是重复代码，后续可以把它抽取出来

```java
private JdbcTemplate jdbcTemplate;

public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
}
```

#### 1.5.2 方式2：让 dao 继承 JdbcDaoSupport

**第二种方式：让 dao 继承 JdbcDaoSupport**

JdbcDaoSupport 是 spring 框架提供的一个类，该类中定义了一个 JdbcTemplate 对象，可以直接获取使用，但是要想创建该对象，**需要为其提供一个数据源**：

具体源码如下：  

```java
public abstract class JdbcDaoSupport extends DaoSupport {
    
    // 定义对象
    private JdbcTemplate jdbcTemplate;
    
    // set 方法注入数据源，判断是否注入了，注入了就创建 JdbcTemplate
    public final void setDataSource(DataSource dataSource) {
        if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource()){ 
            // 如果提供了数据源就创建 JdbcTemplate
            this.jdbcTemplate = createJdbcTemplate(dataSource);
            initTemplateConfig();
        }
    }
    
    //使用数据源创建 JdcbTemplate
    protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    // 当然，也可以通过注入 JdbcTemplate 对象
    public final void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        initTemplateConfig();
    }
    
    // 使用 getJdbcTmeplate 方法获取操作模板对象
    public final JdbcTemplate getJdbcTemplate() {
        return this.jdbcTemplate;
    }
}
```

此时的账户的Dao接口和对应的实现类

```java
// 账户的接口
public interface IAccountDao {

    // 根据 id 查询账户信息
    Account findAccountById(Integer id);

    // 根据名称查询账户信息
    Account findAccountByName(String name);

    // 更新账户信息
    void updateAccount(Account account);
}


// 账户的持久层实现类：此版本 dao，只需要给它的父类注入一个数据源
public class AccountDaoImpl2 extends JdbcDaoSupport implements IAccountDao {

    @Override
    public Account findAccountById(Integer id) {
        //getJdbcTemplate()方法是从父类上继承下来的。
        List<Account> list = getJdbcTemplate().query("select * from account where id = ? ",new AccountRowMapper(),id);

        return list.isEmpty()?null:list.get(0);
    }

    @Override
    public Account findAccountByName(String name) {
        //getJdbcTemplate()方法是从父类上继承下来的。
        List<Account> list = getJdbcTemplate().query("select * from account where name = ? ",new AccountRowMapper(),name);
        if(list.isEmpty()){
            return null;
        }
        if(list.size()>1){
            throw new RuntimeException("结果集不唯一，不是只有一个账户对象");
        }
        return list.get(0);
    }

    @Override
    public void updateAccount(Account account) {
        //getJdbcTemplate()方法是从父类上继承下来的。
        getJdbcTemplate().update("update account set money = ? where id = ?",account.getMoney(),account.getId());
    }
}
```

**配置文件**：  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 配置 dao2 -->
    <bean id="accountDao2" class="cn.xyc.dao.impl.AccountDaoImpl2">
        <!-- 注入 dataSource -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 配置数据源 -->
    <bean id="dataSource"  class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///db2"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
</beans>
```

这两版 Dao 的区别：

1. 第一种在 Dao 类中定义 JdbcTemplate 的方式，适用于所有配置方式（ xml 和注解都可以）。
2. 第二种让 Dao 继承 JdbcDaoSupport 的方式，只能用于基于 XML 的方式，注解用不了，由于其数据源无法通过 @Autowrite 注入 

## 2. Spring中的事务控制

### 2.1 Spring事务控制要明确

1. JavaEE 体系进行分层开发，**事务处理位于业务层**， Spring 提供了分层设计业务层的事务处理解决方案。
2. Spring 框架为我们提供了一组事务控制的接口。接口在 `spring-tx-x.x.x.jar`包中
3. Spring 的事务控制都是基于 AOP 的，它既可以使用编程的方式实现，也可以使用配置的方式实现。 

### 2.2 Spring事务控制API

#### 2.2.1 PlatformTransactionManager

> 这个只是一个接口，而真正的实现类见下面的两个

此接口是 spring 的事务管理器，它里面提供了常用的操作事务的方法，如下：  

1. **获取事物状态**：`TransactionStatus getTransaction(TransactionDefinition definition)`
2. **提交事物**：`void commit(TransactionStatus status)`
3. **回滚事物**：`void rollback(TransctionStatus status)`

在开发中都是使用它的实现类，真正管理事务的对象：

* `org.springframework.jdbc.datasource.DataSourceTransactionManager` 使用 Spring JDBC 或 iBatis 进行持久化数据时使用
* `org.springframework.orm.hibernate5.HibernateTransactionManager` 使用 Hibernate 版本进行持久化数据时使用  

#### 2.2.2 TransactionDefinition

它是事务的定义信息对象，里面有如下方法：

* 获取事物对象名称：`String getName()`
* 获取事物隔离级别：`int getIsolationLevel()`
* 获取事物传播行为：`int getPropagationBehavior()`
* 获取事物超时时间：`int getTimeout()`
* 获取是否是否只读：`boolean isReadOnly()`

**事务的隔离级别(isolation)**

* `ISOLATION_DEFAULT`：默认隔离级别，归属下列某一种（即：默认就是数据库的隔离级别）
* `ISOLATION_READ_UNCOMMITTED`：可以读取未提交数据
* `ISOLATION_READ_COMMITED`：只能读取已提交数据，解决脏读问题
* `ISOLATION_REPEATABLE_READ`：只能读取其他事物提交修改后的数据，解决不可重复度问题
* `ISOLATION_SERIALIZABLE`：只能读取其他事物提交添加后的数据，解决幻读问题

**事务的传播行为(propagation)**

* **REQUIRED**：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选择（默认值）
* **SUPPORTS**：支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）
* MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常
* REQUERS_NEW：新建事务，如果当前在事务中，把当前事务挂起。
* NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
* NEVER：以非事务方式运行，如果当前存在事务，抛出异常
* NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行 REQUIRED 类似的操作。  

**超时时间（Timeout）**：默认值是-1，没有超时限制。如果有，以秒为单位进行设置。

**是否是只读事务（isReadOnly）**：建议查询时设置为只读  

#### 2.2.3 TransactionStatus

此接口提供的是事务具体的运行状态，方法介绍如下：TransactionStatus 接口描述了某个时间点上事物对象的状态信息，包含6个具体的操作：

* 刷新事物：`void flush()`
* 获取是否存在存储点：`boolean hasSavepoint()`
* 获取事物是否完成：`boolean isCompleted()`
* 获取事物是否成为新的事务：`boolean isNewTransaction()`
* 获取事物是否回滚：`isRollbackOnly()`
* 设置事物回滚：`void setRollbackOnly()`

### 2.3 基于XML声明式事务控制

#### 2.3.1 环境搭建

**第一步：导包**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

**第二步：创建 spring 的配置文件并导入约束**

此处需要导入 aop 和 tx 两个名称空间  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/tx
             http://www.springframework.org/schema/tx/spring-tx.xsd
             htp://www.springframework.org/schema/aop
             http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

**第三步：准备数据库表和实体类**

准备数据库表

```sql
-- 创建数据库
create database db2;
use db2;

-- 创建表：
create table account(
    id int primary key auto_increment,
    name varchar(40),
    money float
)character set utf8 collate utf8_general_ci;
```

准备实体类

```java
// 账户的实体
@Data
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Float money;
}
```

**第四步：编写业务层接口和实现类**

```java
// 账户的业务层接口
public interface IAccountService {

    // 根据 id 查询账户信息
    Account findAccountById(Integer id);

    /**
	 * 转账
	 * @param sourceName 转出账户名称
	 * @param targeName 转入账户名称
	 * @param money 转账金额
	 */
    void transfer(String sourceName,String targeName,Float money);//增删改
}


// 账户的业务层实现类
public class AccountServiceImpl implements IAccountService {
    
    private IAccountDao accountDao;
    
    // xml方式注入
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
    
    @Override
    public Account findAccountById(Integer id) {
        return accountDao.findAccountById(id);
    }
    
    @Override
    public void transfer(String sourceName, String targeName, Float money) {
        //1.根据名称查询两个账户
        Account source = accountDao.findAccountByName(sourceName);
        Account target = accountDao.findAccountByName(targeName);
        //2.修改两个账户的金额
        source.setMoney(source.getMoney()-money);//转出账户减钱
        target.setMoney(target.getMoney()+money);//转入账户加钱
        //3.更新两个账户
        accountDao.updateAccount(source);
        // 手动设置异常
        int i=1/0;
        accountDao.updateAccount(target);
    }
}
```

**第五步：编写 Dao 接口和实现类**

```java
// 账户的持久层接口
public interface IAccountDao {

    // 根据 id 查询账户信息
    Account findAccountById(Integer id);

    // 根据名称查询账户信息
    Account findAccountByName(String name);

    // 更新账户信息
    void updateAccount(Account account);
}


// 账户的持久层实现类：此版本 dao，只需要给它的父类注入一个数据源
public class AccountDaoImpl extends JdbcDaoSupport implements IAccountDao {
    
    @Override
    public Account findAccountById(Integer id) {
        List<Account> list = getJdbcTemplate().query("select * from account where id = ? ",new AccountRowMapper(), id);
        return list.isEmpty()?null:list.get(0);
    }
    
    @Override
    public Account findAccountByName(String name) {
        List<Account> list = getJdbcTemplate().query("select * from account where name = ? ",new AccountRowMapper(),name);
        if(list.isEmpty()){
            return null;
        }
        if(list.size()>1){
            throw new RuntimeException("结果集不唯一，不是只有一个账户对象");
        }
        return list.get(0);
    }
    
    @Override
    public void updateAccount(Account account) {
        getJdbcTemplate().update("update account set money = ? where id = ?",account.getMoney(),account.getId());
    }
}

// 账户的封装类 RowMapper 的实现类
public class AccountRowMapper implements RowMapper<Account>{
    @Override
    public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
        Account account = new Account();
        account.setId(rs.getInt("id"));
        account.setName(rs.getString("name"));
        account.setMoney(rs.getFloat("money"));
        return account;
    }
}
```

**第六步：在配置文件中配置业务层和持久层**

```xml
<!-- 配置 service -->
<bean id="accountService" class="cn.xyc.service.impl.AccountServiceImpl">
    <property name="accountDao" ref="accountDao"></property>
</bean>

<!-- 配置 dao -->
<bean id="accountDao" class="cn.xyc.dao.impl.AccountDaoImpl">
    <!-- 注入 dataSource -->
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置数据源 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql:///db2"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

#### 2.3.2 配置步骤

**第一步： 配置事务管理器**

```xml
<!-- 配置一个事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 注入 DataSource -->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

**第二步：配置事务的通知引用事务管理器**

```xml
<!-- 事务的配置 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
</tx:advice>
```

**第三步：配置事务的属性**

```xml
<!--在 tx:advice 标签内部配置事务的属性，即在第二步标签中的内部 -->
<tx:attributes>
    <!-- 指定方法名称：是业务核心方法
		read-only：是否是只读事务。默认 false，不只读。
		isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
		propagation：指定事务的传播行为。
		timeout：指定超时时间。默认值为： -1。永不超时。
		rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。没有默认值，任何异常都回滚。
		no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，任何异常都回滚。
-->
    <tx:method name="*" read-only="false" propagation="REQUIRED"/>
    <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
</tx:attributes>
```

> 上述配置方式，对于`find*`的查询函数，不采用事务，采用只读的方式，对于其他的增删改函数，采用事务的方式进行处理

**第四步：配置 AOP 切入点表达式**

```xml
<!-- 配置 aop -->
<aop:config>
    <!-- 配置切入点表达式 -->
    <aop:pointcut expression="execution(* cn.xyc.service.impl.*.*(..))" id="pt1"/>
</aop:config>
```

**第五步：配置切入点表达式和事务通知的对应关系**

```xml
<!-- 在 aop:config 标签内部： 建立事务的通知和切入点表达式的关系 -->
<aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
```

#### 2.4.3 基于XML的配置整合

```xml
<!-- spring中基于XML的声明式事务控制配置步骤
        1、配置事务管理器
        2、配置事务的通知
            此时需要导入事务的约束 tx名称空间和约束，同时也需要aop的约束
            使用tx:advice标签配置事务通知，
			属性：
               id：给事务通知起一个唯一标识
               transaction-manager：给事务通知提供一个事务管理器引用
        3、配置AOP中的通用切入点表达式
        4、建立事务通知和切入点表达式的对应关系
        5、配置事务的属性，是在事务的通知tx:advice标签的内部-->

<!-- 1.配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 注入 DataSource -->
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 2.配置事务的通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!-- 5.配置事务的属性-->
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" read-only="false"/>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>

<!-- 3.配置AOP中的通用切入点表达式-->
<aop:config>
    <!-- 配置切入点表达式-->
    <aop:pointcut id="pt1" expression="execution(* cn.xyc.service.impl.*.*(..))"></aop:pointcut>
    <!-- 4.建立切入点表达式和事务通知的对应关系 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
</aop:config>
```

### 2.4基于注解的配置方式

#### 2.4.1 环境搭建

**第一步：导包**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

**第二步：创建 spring 的配置文件导入约束并配置扫描的包**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/aop
			http://www.springframework.org/schema/aop/spring-aop.xsd
			http://www.springframework.org/schema/tx
			http://www.springframework.org/schema/tx/spring-tx.xsd
			http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!-- 配置 spring 创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.xyc"></context:component-scan>
    
    <!-- 配置 JdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!-- 配置 spring 提供的内置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/db2"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
</beans>
```

**第三步：创建数据库表和实体类**

和基于XML配置方式内容中相同，略

**第四步：创建业务层接口和实现类并使用注解让 spring 管理**

```java
// 账户的业务层实现类
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
    
    @Autowired
    private IAccountDao accountDao;
    // 其余代码和基于XML的配置相同
}
```

**第五步：创建 Dao 接口和实现类并使用注解让 spring 管理**

```java
// 账户的持久层实现类
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    // 其余代码和基于 XML 的配置相同
}
```

#### 2.4.2 配置步骤

**第一步：配置事务管理器并注入数据源**

```xml
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

**第二步：在业务层使用@Transactional 注解**

```java
@Service("accountService")
@Transactional(readOnly=true, propagation=Propagation.SUPPORTS)
public class AccountServiceImpl implements IAccountService {
    
    @Autowired
    private IAccountDao accountDao;
    
    @Override
    public Account findAccountById(Integer id) {
        return accountDao.findAccountById(id);
    }
    
    @Override
    @Transactional(readOnly=false,propagation=Propagation.REQUIRED)
    public void transfer(String sourceName, String targeName, Float money) {
        //1.根据名称查询两个账户
        Account source = accountDao.findAccountByName(sourceName);
        Account target = accountDao.findAccountByName(targeName);
        //2.修改两个账户的金额
        source.setMoney(source.getMoney()-money);//转出账户减钱
        target.setMoney(target.getMoney()+money);//转入账户加钱
        //3.更新两个账户
        accountDao.updateAccount(source);
        //int i=1/0;
        accountDao.updateAccount(target);
    }
}
```

该注解的属性和 xml 中的属性含义一致。该注解可以出现在接口上，类上和方法上。

* 出现接口上，表示该接口的所有实现类都有事务支持。
* 出现在类上，表示类中所有方法有事务支持
* 出现在方法上，表示方法有事务支持。
* 以上三个位置的优先级：方法>类>接口  

**第三步：在配置文件中开启 spring 对注解事务的支持**

```xml
<!-- 开启 spring 对注解事务的支持 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

#### 2.4.3 不使用 xml 的配置方式

```java
@Configuration
@EnableTransactionManagement
public class SpringTxConfiguration {
	// 里面配置数据源，配置 JdbcTemplate,配置事务管理器。在之前的步骤已经写过了。
}
```

> 上述就是声明式事物的配置，当然还有**编程式事物的配置**，不过不常用...

## 3. Spring5的新特性

> 这里仅记录：与JDK相关的升级，其他升级当前还不是很理解...

**JDK版本要求**

spring5.0 在 2017 年 9 月发布了它的 GA（通用）版本。该版本是基于 jdk8 编写的， 所以 jdk8 以下版本将无法使用。 同时，可以兼容 jdk9 版本。

tomcat 版本要求 8.5 及以上。

**利用 JDK8 版本更新的内容**

第一： 基于 JDK8 的反射增强

```java

public class Test {
    //循环次数定义： 10 亿次
    private static final int loopCnt = 1000 * 1000 * 1000;
    public static void main(String[] args) throws Exception {
        //输出 jdk 的版本
        System.out.println("java.version=" + System.getProperty("java.version"));
        t1();
        t2();
        t3();
    }

    // 每次重新生成对象
    public static void t1() {
        long s = System.currentTimeMillis();
        for (int i = 0; i < loopCnt; i++) {
            Person p = new Person();
            p.setAge(31);
        }
        long e = System.currentTimeMillis();
        System.out.println("循环 10 亿次创建对象的时间： " + (e - s));
    }

    // 同一个对象
    public static void t2() {
        long s = System.currentTimeMillis();
        Person p = new Person();
        for (int i = 0; i < loopCnt; i++) {
            p.setAge(32);
        }
        long e = System.currentTimeMillis();
        System.out.println("循环 10 亿次给同一对象赋值的时间： " + (e - s));
    }
    
    //使用反射创建对象
    public static void t3() throws Exception {
        long s = System.currentTimeMillis();
        Class<Person> c = Person.class;
        Person p = c.newInstance();
        Method m = c.getMethod("setAge", Integer.class);
        for (int i = 0; i < loopCnt; i++) {
            m.invoke(p, 33);
        }
        long e = System.currentTimeMillis();
        System.out.println("循环 10 亿次反射创建对象的时间： " + (e - s));
    }
    
    static class Person {
        private int age = 20;
        public int getAge() {
            return age;
        }
        public void setAge(Integer age) {
            this.age = age;
        }
    }
}
```

> 过程略，因为本机没有装JDK7，反正，在反射创建对象上， JDK8 确实做了加强。

第二： `@NonNull` 注解和 `@Nullable` 注解的使用

用 `@Nullable` 和 `@NotNull` 注解来显示表明可为空的参数和以及返回值。这样就够在编译的时候处理空值而不是在运行时抛出 NullPointerExceptions。

第三： 日志记录方面

Spring Framework 5.0 带来了 Commons Logging 桥接模块的封装，它被叫做 `spring-jcl` 而不是标准的 Commons Logging。当然，无需任何额外的桥接，新版本也会对 `Log4j 2.x`， `SLF4J`，`JUL(java.util.logging)` 进行自动检测。  