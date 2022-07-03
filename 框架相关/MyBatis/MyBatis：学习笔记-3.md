# MyBatis：学习笔记-3

> 后端学习ING，在学完了Java语法部分，JavaWeb知识后，开始一套组合拳来带走经典框架SSM。
>
> 此文为第一拳：MyBatis学习笔记的 **3/4**。

## 1. MyBatis连接池与事务

### 1.1 MyBatis的连接池技术

Mybatis 中采用的是自己的连接池技术。在 Mybatis 的 SqlMapConfig.xml 配置文件中， 通过`<dataSource type="pooled">`来实现 Mybatis 中连接池的配置。  

#### 1.1.1 MyBatis连接池的分类

在 MyBatis 中将它的数据源 dataSource 分为以下几类：  

1. `org.apache.ibatis.datasource`;
2. `org.apache.ibatis.datasource.jndi`;
3. `org.apache.ibatis.datasource.pooled`;
4. `org.apache.ibatis.datasource.unpooled`;

可以看出 Mybatis 将它自己的数据源分为三类：  

1. UNPOOLED：不使用连接池的数据源；
2. **POOLED：使用连接池的数据源；**
3. JNDI：使用 JNDI 实现的数据源

具体结构如下：

![MyBatis的连接池结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252255026.PNG)

相应地， MyBatis 内部分别定义了实现了 `java.sql.DataSource` 接口的 `UnpooledDataSource`，`PooledDataSource` 类来表示 UNPOOLED、 POOLED 类型的数据源。  

![MyBatis中DataSource实现UML图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252255276.PNG)

在这三种数据源中，一般采用的是 POOLED 数据源（很多时候数据源就是为了更好的管理数据库连接，也就是所谓的连接池技术） 。  

#### 1.1.2 MyBatis中数据源的配置

数据源配置就是在 SqlMapConfig.xml 文件中， 具体配置如下：  

```xml
<!-- 配置数据源（连接池）信息 -->
<dataSource type="POOLED">
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</dataSource>
```

MyBatis 在初始化时， 根据`<dataSource>`的 type 属性来创建相应类型的的数据源 DataSource，即：

* `type=”POOLED”`： MyBatis 会创建 PooledDataSource 实例
* `type=”UNPOOLED”` ： MyBatis 会创建 UnpooledDataSource 实例
* `type=”JNDI”`： MyBatis 会从 JNDI 服务上查找 DataSource 实例，然后返回使用  

#### 1.1.3 MyBatis中DataSource的存取

MyBatis 是通过工厂模式来创建数据源 DataSource 对象的，MyBatis定义了抽象的工厂接口：`org.apache.ibatis.datasource.DataSourceFactory`，通过其 `getDataSource()` 方法返回数据源 DataSource。  

下面是 DataSourceFactory 源码，具体如下：

```java
package org.apache.ibatis.datasource;
import java.util.Properties;
import javax.sql.DataSource;
/**
  * @author Clinton Begin
  */
public interface DataSourceFactory {
    
    // 设置属性接口
    void setProperties(Properties props);
    // 返回数据源 DataSource
    DataSource getDataSource();
}
```

MyBatis 创建了 DataSource 实例后，会将其放到 Configuration 对象内的 Environment 对象中， 供以后使用。

具体分析过程如下：  

1. 先进入 XMLConfigBuilder 类中，可以找到如下代码：  

```java
public Configuration parse() {
    if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}
```

2. 分析 configuration 对象的 environment 属性，结果如下  

![configuration对象的environment属性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252256297.PNG)

#### 1.1.4 MyBatis中连接的获取过程分析

当需要创建 SqlSession 对象并需要执行 SQL 语句时，这时候 MyBatis 才会去调用 dataSource 对象来创建 `java.sql.Connection` 对象。**也就是说， `java.sql.Connection` 对象的创建一直延迟到执行SQL语句的时候。**  

```java
@Test
public void testSql() throws Exception {
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
    SqlSession sqlSession = factory.openSession();
    // 这里才创建Connection对象
    List<User> list = sqlSession.selectList("findUserById",41);
    System.out.println(list.size());
}
```

只有当第 4 句 `sqlSession.selectList("findUserById")`，才会触发 MyBatis 在底层执行下面这个方法来创建 `java.sql.Connection` 对象。  

> 具体源码分析过暂略...

下面是连接获取的源代码：

```java
@Override
public Connection getConnection() throws SQLException {
    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
}

@Override
public Connection getConnection(String username, String password) throws SQLException {
    return popConnection(username, password).getProxyConnection();
}
```

**结论**：可以发现，**真正连接打开的时间点，只是在执行SQL语句时，才会进行**。其实这样做是因为数据库连接是最为宝贵的资源，只有在要用到的时候，才去获取并打开连接，当用完了就再立即将数据库连接归还到连接池中。  

### 1.2 MyBatis的事务控制

#### 1.2.1 JDBC中事务的回顾

在 JDBC 中可以通过手动方式将事务的提交改为手动方式，通过 `setAutoCommit()` 方法就可以调整。

通过 JDK 文档，找到该方法如下：  

![setAutoCommit方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252256630.PNG)

那么 Mybatis 框架因为是对 JDBC 的封装，所以 Mybatis 框架的事务控制方式，本身也是用 JDBC 的`setAutoCommit()`方法来设置事务提交方式的。  

#### 1.2.2 MyBatis中事务提交方式

Mybatis 中事务的提交方式，本质上就是调用 JDBC 的 `setAutoCommit()` 来实现事务控制。

运行之前所写的代码：  

```java
@Test
public void testSaveUser() throws Exception {
    User user = new User();
    user.setUsername("mybatis user09");
    //6.执行操作
    int res = userDao.saveUser(user);
    System.out.println(res);
    System.out.println(user.getId());
}

@Before//在测试方法执行之前执行
public void init()throws Exception {
    //1.读取配置文件
    in = Resources.getResourceAsStream("SqlMapConfig.xml");
    //2.创建构建者对象
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    //3.创建 SqlSession 工厂对象
    factory = builder.build(in);
    //4.创建 SqlSession 对象
    session = factory.openSession();
    //5.创建 Dao 的代理对象
    userDao = session.getMapper(IUserDao.class);
}

@After//在测试方法执行完成之后执行
public void destroy() throws Exception{
    //7.提交事务
    session.commit();
    //8.释放资源
    session.close();
    in.close();
}
```

观察在它在控制台输出的结果：  

```bash
Opening JDBC Connection
Created connection xxxx
Setting autocommit to false on JDBC Connection [xxx]
# ...执行的流程
Committing JDBC Connection [xxx]
Resetting autocommit to true on JDBC Conneciton [xxx]
Colsing JDBC Connection [xxx]
Returned connection xxx to pool.
```

Connection 的整个变化过程， 通过分析能够发现之前的 CUD 操作过程中，都要手动进行事务的提交，原因是 `setAutoCommit()` 方法，在执行时它的值被设置为 false 了，所以在 CUD 操作中，必须通过 `sqlSession.commit()` 方法来执行提交操作。  

#### 1.2.3 MyBatis自动提交事务的设置

通过上面的研究和分析，为什么 CUD 过程中必须使用 `sqlSession.commit()` 提交事务？

主要原因就是在连接池中取出的连接，都会将调用 `connection.setAutoCommit(false)`方法，这样就必须使用 `sqlSession.commit()` 方法，相当于使用了 JDBC 中的 `connection.commit()` 方法实现事务提交。

明白这一点后，现在一起尝试不进行手动提交，一样实现 CUD 操作。

```java
@Before//在测试方法执行之前执行
public void init()throws Exception {
    //1.读取配置文件
    in = Resources.getResourceAsStream("SqlMapConfig.xml");
    //2.创建构建者对象
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    //3.创建 SqlSession 工厂对象
    factory = builder.build(in);
    //4.创建 SqlSession 对象
    session = factory.openSession(true);  // 这里设置为true了，表明自动提交事物，见后续openSession方法代码
    //5.创建 Dao 的代理对象
    userDao = session.getMapper(IUserDao.class);
}

@After//在测试方法执行完成之后执行
public void destroy() throws Exception{
    //7.释放资源
    session.close();
    in.close();
}
```

所对应的 DefaultSqlSessionFactory 类的源代码：

```java
@Override
public SqlSession openSession(boolean autoCommit) {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, autoCommit);
}
```

运行的结果如下：

```bash
Opening JDBC Connection
Created connection xxxx
# ...执行的流程
Committing JDBC Connection [xxx]
Colsing JDBC Connection [xxx]
Returned connection xxx to pool.
```

可以发现，此时事务就设置为自动提交了，同样可以实现CUD操作时记录的保存。虽然这也是一种方式，但就编程而言，设置为自动提交方式为 false 再根据情况决定是否进行提交，这种方式更常用。因为可以根据业务情况来决定提交是否进行提交。  

## 2. MyBatis的动态SQL语句

Mybatis 的映射文件中，前面 SQL 都是比较简单的，有些时候业务逻辑复杂时，SQL 是**动态变化**的，此时在前面的学习中的 SQL 就不能满足要求了。

参考的官方文档，描述如下：  

![动态SQL](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252256371.PNG)

### 2.1 动态SQL之`<if>`标签

根据实体类的不同取值，使用不同的 SQL 语句来进行查询。比如在 id 如果不为空时可以根据 id 查询，如果 username 不同空时还要加入用户名作为条件。这种情况在多条件组合查询中经常会碰到。  

**持久层 Dao 接口**

```java
// 根据用户信息，查询用户列
List<User> findByUser(User user);
```

**持久层 Dao 映射配置**

```xml
<select id="findByUser" resultType="user" parameterType="user">
    select * from user where 1=1
    <if test="username!=null and username != '' ">
        and username like #{username}
    </if>
    <if test="address != null">
        and address like #{address}
    </if>
</select>
```

> 注意：
>
> 1. `<if>`标签的 test 属性中写的是对象的属性名，如果是包装类的对象要使用 OGNL 表达式的写法。
> 2. 另外要注意 `where 1=1` 的作用~！  

**测试**  

```java
@Test
public void testFindByUser() {
    User u = new User();
    u.setUsername("%王%");
    u.setAddress("%顺义%");
    //6.执行操作
    List<User> users = userDao.findByUser(u);
    for(User user : users) {
        System.out.println(user);
    }
}
```

### 2.2 动态SQL之`<where>`标签

为了简化上面 `where 1=1` 的条件拼装，可以采用`<where>`标签来简化开发。  

**持久层 Dao 映射配置**

```xml
<!-- 了解的内容：抽取重复的sql语句-->
<sql id="defaultUser">
    select * from user
</sql>

<!-- 根据用户信息查询 -->
<select id="findByUser" resultType="user" parameterType="user">
    <include refid="defaultSql"></include>
    <where>
        <if test="username!=null and username != '' ">
            and username like #{username}
        </if>
        <if test="address != null">
            and address like #{address}
        </if>
    </where>
</select>
```

### 2.3 动态SQL之`<foreach>`标签

**需求：**

* 传入多个 id 查询用户信息，用下边两个 sql 实现：

  `SELECT * FROM USERS WHERE username LIKE '%张%' AND (id =10 OR id =89 OR id=16)`

  `SELECT * FROM USERS WHERE username LIKE '%张%' AND id IN (10,89,16)`

* 这样在进行范围查询时，就要将一个集合中的值，作为参数动态添加进来。那将如何进行参数的传递？  


**在 QueryVo 中加入一个 List 集合用于封装参数**

```java
public class QueryVo implements Serializable {
    
    private List<Integer> ids;

    // get/set
}
```

**持久层 Dao 接口**

```java
// 根据 id 集合查询用户
List<User> findInIds(QueryVo vo);
```

**持久层 Dao 映射配置**

```xml
<sql id="defaultUser">
    select * from user
</sql>

<!-- 查询所有用户在 id 的集合之中 -->
<select id="findInIds" resultType="user" parameterType="queryvo">
    <!-- select * from user where id in (1,2,3,4,5); -->
    <include refid="defaultSql"></include>
    <where>
        <if test="ids != null and ids.size() > 0">
            <foreach collection="ids" open="id in ( " close=")" item="uid" separator=",">
                #{uid}
            </foreach>
        </if>
    </where>
</select>
```

* SQL 语句：`select 字段 from user where id in (?) `
* `<foreach>`标签用于遍历集合，它的属性：  
  * collection：代表要遍历的集合元素，注意编写时不要写`#{}`
  * open：代表语句的开始部分  
  * close：代表结束部分  
  * item：代表遍历集合的每个元素，生成的变量名
  * sperator：代表分隔符  

**编写测试方法**

```java
@Test
public void testFindInIds() {
    QueryVo vo = new QueryVo();
    List<Integer> ids = new ArrayList<Integer>();
    ids.add(41);
    ids.add(42);
    ids.add(43);
    ids.add(46);
    ids.add(57);
    vo.setIds(ids);
    //6.执行操作
    List<User> users = userDao.findInIds(vo);
    for(User user : users) {
        System.out.println(user);
    }
}
```

### 2.4 MyBatis中简化编写的SQL片段

sql 中可将重复的 sql 提取出来，使用时用 include 引用即可，最终达到 sql 重用的目的。  

**定义代码片段**

```xml
<!-- 抽取重复的语句代码片段 -->
<sql id="defaultSql">
    select * from user
</sql>
```

**引用代码片段**

```xml
<!-- 配置查询所有操作 -->
<select id="findAll" resultType="user">
    <include refid="defaultSql"></include>
</select>

<!-- 根据 id 查询 -->
<select id="findById" resultType="UsEr" parameterType="int">
    <include refid="defaultSql"></include>
    where id = #{uid}
</select>
```

## 3. MyBatis多表查询之一对多

本次案例主要以最为简单的用户和账户的模型来分析 Mybatis 多表关系。

用户为 User 表，账户为Account表。一个用户（User）可以有多个账户（Account）。

具体关系如下：  

![一对多关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252256975.PNG)

### 3.1 一对一查询

需求：查询所有账户信息，关联查询下单用户信息。

注意：因为一个账户信息只能供某个用户使用，所以从查询账户信息出发关联查询用户信息为一对一查询。如果从用户信息出发查询用户下的账户信息则为一对多查询，因为一个用户可以有多个账户。  

#### 3.1.1 方式1

**定义账户信息的实体类**

```java
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
    // get/set/toString
}
```

**编写 Sql 语句**：实现查询账户信息时，也要查询账户所对应的用户信息。  

```SQL
SELECT
	account.*,
	user.username,
	user.address
FROM
	account,
	user
WHERE account.uid = user.id
```

**定义 AccountUser 类**

为了能够封装上面 SQL 语句的查询结果，定义 AccountCustomer 类中要包含账户信息同时还要包含用户信息，所以要在定义 AccountUser 类时可以继承 User 类。  

```java
public class AccountUser extends Account implements Serializable {
    private String username;
    private String address;

    // set/set/toString
}
```

**定义账户的持久层 Dao 接口**

```java
public interface IAccountDao {
    // 查询所有账户，同时获取账户的所属用户名称以及它的地址信息
    List<AccountUser> findAll();
}
```

**定义 AccountDao.xml 文件中的查询配置信息**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.xyc.dao.IAccountDao">
    <!-- 配置查询所有操作-->
    <select id="findAll" resultType="accountuser">
        select a.*,u.username,u.address from account a,user u where a.uid =u.id;
    </select>
</mapper>
```

> 注意：因为上面查询的结果中包含了账户信息同时还包含了用户信息，所以返回值类型 returnType 的值设置为 AccountUser 类型，这样就可以接收账户信息和用户信息了。  

**创建 AccountTest 测试类**

```java
public class AccountTest {
    
    private InputStream in ;
    private SqlSessionFactory factory;
    private SqlSession session;
    private IAccountDao accountDao;
    
    @Test
    public void testFindAll() {
        //6.执行操作
        List<AccountUser> accountusers = accountDao.findAll();
        for(AccountUser au : accountusers) {
            System.out.println(au);
        }
    }
    
    @Before//在测试方法执行之前执行
    public void init()throws Exception {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建构建者对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //3.创建 SqlSession 工厂对象
        factory = builder.build(in);
        //4.创建 SqlSession 对象
        session = factory.openSession();
        //5.创建 Dao 的代理对象
        accountDao = session.getMapper(IAccountDao.class);
    }
    
    @After//在测试方法执行完成之后执行
    public void destroy() throws Exception{
        session.commit();
        //7.释放资源
        session.close();
        in.close();
    }
}
```

**小结：**定义专门的 pojo 类作为输出类型，其中定义了 sql 查询结果集所有的字段。此方法较为简单，企业中使用普遍。  

#### 3.1.2 方式2

使用 resultMap，定义专门的 resultMap 用于映射一对一查询结果。

通过面向对象的(has a)关系可以得知，可以在 Account 类中加入一个 User 类的对象来代表这个账户是哪个用户的。  

**修改 Account 类**  

```java
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;
	
    // 添加一个 User 成员对象
    private User user;
	
    // get/set/toString
}
```

**修改 AccountDao 接口中的方法**

```java
public interface IAccountDao {
    
    // 查询所有账户，同时获取账户的所属用户名称以及它的地址信息
    List<Account> findAll();
}
```

> 注意：第二种方式，将返回值改为了 Account 类型。因为 Account 类中包含了一个 User 类的对象，它可以封装账户所对应的用户信息。  

**重新定义 AccountDao.xml 文件**  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.xyc.dao.IAccountDao">
    <!-- 建立对应关系 -->
    <resultMap type="account" id="accountMap">
        <!-- 注意此处的aid，用于查询语句中采用了别名： a.id as aid account as a-->
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
        <!-- 它是用于指定从表方的引用实体属性的 -->
        <association property="user" javaType="user">
            <id column="id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="birthday" property="birthday"/>
            <result column="address" property="address"/>
        </association>
    </resultMap>
    
    <select id="findAll" resultMap="accountMap">
        select u.*, a.id as aid, a.uid, a.money from account a, user u where a.uid =u.id;
    </select>
</mapper>
```

**在 AccountTest 类中加入测试方法  **  

```java
@Test
public void testFindAll() {
    List<Account> accounts = accountDao.findAll();
    for(Account au : accounts) {
        System.out.println(au);
        System.out.println(au.getUser());
    }
}
```

### 3.2 一对多查询

需求：查询所有用户信息及用户关联的账户信息。

分析：用户信息和他的账户信息为一对多关系，并且查询过程中如果用户没有账户信息，此时也要将用户信息查询出来，采用左外连接查询比较合适。  

**编写 SQL 语句：**

```SQL
SELECT
	u.*, acc.id id,
	acc.uid,
	acc.money
FROM
	user u
LEFT JOIN 
	account acc 
ON 
	u.id = acc.uid
```

**User 类加入 `List<Account>`**  

```java
public class User implements Serializable {
    
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    
    // User类中添加成员变量，表示账户信息：List<Account> accounts
    private List<Account> accounts;
    
    // get/set/toString
}
```

**用户持久层 Dao 接口中加入查询方法**

```java
// 查询所有用户，同时获取出每个用户下的所有账户信息
List<User> findAll();
```

**用户持久层 Dao 映射文件配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.xyc.dao.IUserDao">
    
    <resultMap type="user" id="userMap">
        <id column="id" property="id"></id>
        <result column="username" property="username"/>
        <result column="address" property="address"/>
        <result column="sex" property="sex"/>
        <result column="birthday" property="birthday"/>
        <!-- collection 是用于建立一对多中集合属性的对应关系
			 ofType 用于指定集合元素的数据类型-->
        <collection property="accounts" ofType="account">
            <id column="aid" property="id"/>
            <result column="uid" property="uid"/>
            <result column="money" property="money"/>
        </collection>
    </resultMap>
    
    <!-- 配置查询所有操作 -->
    <select id="findAll" resultMap="userMap">
        select u.*,a.id as aid ,a.uid,a.money from user u left outer join account a on u.id =a.uid
    </select>
</mapper>
```

* `collection`：部分定义了用户关联的账户信息。表示关联查询结果集
* `property="accounts"`：关联查询的结果集存储在 User 对象的上哪个属性。
* `ofType="account"`：指定关联查询的结果集中的对象类型即List中的对象类型。此处可以使用别名，也可以使用全限定名。  

**测试方法**

```java
public class UserTest {
    private InputStream in ;
    private SqlSessionFactory factory;
    private SqlSession session;
    private IUserDao userDao;
    
    @Test
    public void testFindAll() {
        //6.执行操作
        List<User> users = userDao.findAll();
        for(User user : users) {
            System.out.println("-------每个用户的内容---------");
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
    }
    
    @Before//在测试方法执行之前执行
    public void init()throws Exception {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建构建者对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //3.创建 SqlSession 工厂对象
        factory = builder.build(in);
        //4.创建 SqlSession 对象
        session = factory.openSession();
        //5.创建 Dao 的代理对象
        userDao = session.getMapper(IUserDao.class);
    }
    
    @After//在测试方法执行完成之后执行
    public void destroy() throws Exception{
        session.commit();
        //7.释放资源
        session.close();
        in.close();
    }
}
```

## 4. MyBatis多表查询之多对多

### 4.1 实现Role到User多对多

用户与角色的多对多关系模型如下：  

![多对多关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252256886.PNG)

**业务要求及实现SQL**

**需求**：实现查询所有对象并且加载它所分配的用户信息。

**分析**：查询角色需要用到Role表，但角色分配的用户的信息并不能直接找到用户信息，而是要通过中间表(USER_ROLE)才能关联到用户信息。  

下面是实现的 SQL 语句：

```SQL
SELECT
    r.*, 
    u.id uid,
    u.username username,
    u.birthday birthday,
    u.sex sex,
    u.address address
FROM
	ROLE r
INNER JOIN
	USER_ROLE ur
ON ( r.id = ur.rid)
INNER JOIN 
	USER u 
ON (ur.uid = u.id);
```

**编写角色实体类**

```java
public class Role implements Serializable {
    private Integer roleId;
    private String roleName;
    private String roleDesc;
    // 多对多的关系映射：一个角色可以赋予多个用户
    private List<User> users;

    // get/set/toString
}
```

**编写Role持久层接口**

```java
public interface IRoleDao {
    // 查询所有角色
    List<Role> findAll();
}
```

**编写映射文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.xyc.dao.IRoleDao">
    
    <!--定义 role 表的 ResultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="rid"></id>
        <result property="roleName" column="role_name"></result>
        <result property="roleDesc" column="role_desc"></result>
        <collection property="users" ofType="user">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </collection>
    </resultMap>
    
    <!--查询所有-->
    <select id="findAll" resultMap="roleMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from role r
        left outer join user_role ur on r.id = ur.rid
        left outer join user u on u.id = ur.uid
    </select>
</mapper>
```

**编写测试类**

```java
public class RoleTest {
    
    private InputStream in;
    private SqlSession sqlSession;
    private IRoleDao roleDao;
    
    @Before//用于在测试方法执行之前执行
    public void init()throws Exception{
        //1.读取配置文件，生成字节输入流
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.获取 SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        //3.获取 SqlSession 对象
        sqlSession = factory.openSession(true);
        //4.获取 dao 的代理对象
        roleDao = sqlSession.getMapper(IRoleDao.class);
    }
    
    @After//用于在测试方法执行之后执行
    public void destroy()throws Exception{
        //提交事务
        // sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }
    
    /**
      * 测试查询所有
      */
    @Test
    public void testFindAll(){
        List<Role> roles = roleDao.findAll();
        for(Role role : roles){
            System.out.println("---每个角色的信息----");
            System.out.println(role);
            System.out.println(role.getUsers());
        }
    }
}
```

### 4.2 实现User到Role多对多

**User到Role的多对多**：从 User 出发，也可以发现一个用户可以具有多个角色，这样用户到角色的关系也还是一对多关系。这样就可以认为 User 与 Role 的多对多关系，可以被拆解成两个一对多关系来实现。  

**编写用户实体类**

```java
pubic class User implements Serializable {

    private Integer id;
    private String username;
    private String address;
    private String sex;
    private Date birthday;
    // 多对多的关系映射：一个用户可以有多个角色
    private List<Role> roles;

    // get/set/toString
}
```

**编写User持久层接口**

```java
// UserDao接口
public interface UserDao {

    // 查询所有角色
    List<User> findAll();
}
```

**编写映射文件**：UserDao.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.xyc.dao.UserDao">

    <resultMap id="userMap" type="User">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>
        <result column="birthday" property="birthday"/>
        <collection property="roles" ofType="Role">
            <id column="rid" property="roldId"/>
            <result column="ROLE_NAME" property="roleName"/>
            <result column="ROLE_DESC" property="roleDesc"/>
        </collection>
    </resultMap>

    <select id="findAll" resultMap="userMap">
        SELECT u.*, r.`ID` AS rid, r.`ROLE_NAME`, r.`ROLE_DESC` FROM USER AS u
        LEFT OUTER JOIN user_role AS ur ON u.`id`=ur.`UID`
        LEFT OUTER JOIN role AS r ON r.`ID`=ur.`RID`;
    </select>
</mapper>
```

**编写测试方法**

```java
@Test
public void testFindAll(){
    List<User> users = userDao.findAll();
    for(User user:users){
        System.out.println("============");
        System.out.println(user.toString());
        System.out.println(user.getRoles());
    }
}
```

