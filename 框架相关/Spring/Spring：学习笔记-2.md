# Spring：学习笔记-2

> 后端学习ING，在学完了Java语法部分，JavaWeb知识后，开始一套组合拳来带走经典框架SSM。
>
> 此文为第二拳：Spring学习笔记的 **2/4**。

## 1. 使用Spring IoC实现账户CRUD

### 1.1 需求和技术要求

**需求**：实现账户的 CRUD 操作

**技术要求**

* 使用 spring 的 IoC 实现对象的管理
* 使用 DBAssit 作为持久层解决方案
* 使用 c3p0 数据源

### 1.2 环境搭建

**导入依赖**

**创建数据库和编写实体类**  

1. 创建表和插入数据

```SQL
create table account(
    id int primary key auto_increment,
    name varchar(40),
    money float
)character set utf8 collate utf8_general_ci;

insert into account(name,money) values('aaa',1000);
insert into account(name,money) values('bbb',1000);
insert into account(name,money) values('ccc',1000);
```

2. 编写实体类

```java
// 账户的实体类
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Float money;

    // get/set
}
```

**编写持久层代码**  

```java
// 账户的持久层接口
public interface IAccountDao {
    // 保存
    void save(Account account);
    // 更新
    void update(Account account);
    // 删除
    void delete(Integer accountId);
    // 根据 id 查询
    Account findById(Integer accountId);
    // 查询所有
    List<Account> findAll();
}

// 账户的持久层实现类
public class AccountDaoImpl implements IAccountDao {

    private DBAssit dbAssit;

    public void setDbAssit(DBAssit dbAssit) {
        this.dbAssit = dbAssit;
    }

    @Override
    public void save(Account account) {
        dbAssit.update("insert into account(name,money)values(?,?)", account.getName(), account.getMoney());
    }

    @Override
    public void update(Account account) {
        dbAssit.update("update account set name=?,money=? where id=?", account.getName(), account.getMoney(), account.getId());
    }

    @Override
    public void delete(Integer accountId) {
        dbAssit.update("delete from account where id=?", accountId);
    }
    
    @Override
    public Account findById(Integer accountId) {
        return dbAssit.query("select * from account where id=?", 
                             new BeanHandler<Account>(Account.class), 
                             accountId);
    }
    
    @Override
    public List<Account> findAll() {
        return dbAssit.query("select * from account where id=?", 
                             new BeanListHandler<Account>(Account.class));
    }
}
```

**编写业务层代码**  

```java
// 账户的业务层接口
public interface IAccountService {
    // 保存账户
    void saveAccount(Account account);
    // 更新账户
    void updateAccount(Account account);
    // 删除账户
    void deleteAccount(Integer accountId);
    // 根据 id 查询账户
    Account findAccountById(Integer accountId);
    // 查询所有账户
    List<Account> findAllAccount();
}

// 账户的业务层实现类
public class AccountServiceImpl implements IAccountService {
    
    private IAccountDao accountDao;
    
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
    @Override
    public void saveAccount(Account account) {
        accountDao.save(account);
    }
    @Override
    public void updateAccount(Account account) {
        accountDao.update(account);
    }
    @Override
    public void deleteAccount(Integer accountId) {
        accountDao.delete(accountId);
    }
    @Override
    public Account findAccountById(Integer accountId) {
        return accountDao.findById(accountId);
    }
    @Override
    public List<Account> findAllAccount() {
        return accountDao.findAll();
    }
}
```

**创建并编写配置文件**  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

### 1.3 配置对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 配置 service -->
    <bean id="accountService"
          class="cn.xyc.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>
    
    <!-- 配置 dao -->
    <bean id="accountDao" class="cn.xyc.dao.impl.AccountDaoImpl">
        <property name="dbAssit" ref="dbAssit"></property>
    </bean>
    
    <!-- 配置 dbAssit 此处我们只注入了数据源，表明每条语句独立事务-->
    <bean id="dbAssit" class="cn.xyc.dbassit.DBAssit">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///db2"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
</beans>
```

### 1.4 测试案例

**测试类代码**  

```java
public class AccountServiceTest {
    
    // 测试保存
    @Test
    public void testSaveAccount() {
        Account account = new Account();
        account.setName("黑马程序员");
        account.setMoney(100000f);
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        as.saveAccount(account);
    }

    // 测试查询一个
    @Test
    public void testFindAccountById() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        Account account = as.findAccountById(1);
        System.out.println(account);
    }

    // 测试更新
    @Test
    public void testUpdateAccount() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        Account account = as.findAccountById(1);
        account.setMoney(20301050f);
        as.updateAccount(account);
    }

    // 测试删除
    @Test
    public void testDeleteAccount() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        as.deleteAccount(1);
    }

    // 测试查询所有
    @Test
    public void testFindAllAccount() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        List<Account> list = as.findAllAccount();
        for(Account account : list) {
            System.out.println(account);
        }
    }
}
```

**分析测试中的问题**

通过上面的测试类，可以看出，每个测试方法都重新获取了一次 spring 的核心容器，造成了不必要的重复代码，增加了开发的工作量。

这种情况，在开发中应该避免发生。可以把容器的获取定义到类中去。例如：  

```java
public class AccountServiceTest {
    private ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

    private IAccountService as = ac.getBean("accountService",IAccountService.class);
}
```

这种方式虽然能解决问题，但是任需要自己写代码来获取容器。能不能测试时直接就编写测试方法，而不需要手动编码来获取容器呢？

## 2. 基于注解的IoC配置

明确注解配置和 xml 配置要实现的功能都是一样的，都是要降低程序间的耦合。只是配置的形式不一样。

### 2.1 环境搭建

**第一步：导包**

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
</dependencies>
```

**第二步：使用@Component注解配置管理的资源**

```java
// 账户的业务层实现类
@Component("accountService")
public class AccountServiceImpl implements IAccountService {
    
    private IAccountDao accountDao;
    
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
}

// 账户的持久层实现类
@Component("accountDao")
public class AccountDaoImpl implements IAccountDao {
    
    private DBAssit dbAssit;
}
```

> 注意：当使用注解注入时， set 方法不用写  

**第三步： 创建 spring 的 xml 配置文件并开启对注解的支持**  

> 注意：基于注解整合时，导入约束时需要多导入一个 context 名称空间下的约束。由于此处使用了注解配置，此时不能再继承 JdbcDaoSupport，需要自己配置一个 JdbcTemplate  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!-- 告知 spring 创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.xyc"></context:component-scan>
    
    <!-- 配置 dbAssit -->
    <bean id="dbAssit" class="cn.xyc.dbassit.DBAssit">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///db2"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
    
</beans>
```

### 2.2 常用注解

#### 2.2.1 用于创建对象

相当于： `<bean id="" class="">`

**@Component**

* 作用：把资源让 spring 来管理。相当于在 xml 中配置一个 bean。

* 属性：value：指定 bean 的 id。如果不指定 value 属性，默认 bean 的 id 是当前类的类名。首字母小写。

**@Controller @Service @Repository**

* 他们三个注解都是针对一个的衍生注解，他们的作用及属性都是一模一样的。他们只不过是提供了更加明确的语义化。
* @Controller： 一般用于表现层的注解。
* @Service： 一般用于业务层的注解。
* @Repository： 一般用于持久层的注解。

> 细节：如果注解中有且只有一个属性要赋值时，且名称是 value， value 在赋值是可以不写。

#### 2.2.2 用于注入数据

相当于： `<property name="" ref="">` /  `<property name="" value="">`

**@Autowired**

* 作用：

  自动按照类型注入。当使用注解注入属性时， set 方法可以省略。

  它**只能注入其他 bean 类型**。

  只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功；

  而当有多个类型匹配时，使用要注入的对象变量名称作为 bean 的 id，在 spring 容器查找，找到了也可以注入成功。找不到就报错。  

* 出现位置：可以是变量上，也可以是方法上；

##### @Qualifier

* 作用：在自动按照类型注入的基础之上，**再按照 Bean 的 id 注入**。它在给字段注入时不能独立使用，**必须和 @Autowire 一起使用**；但是给方法参数注入时，可以独立使用。
* 属性：value：指定 bean 的 id。

> 但是给方法参数注入时，可以独立使用，实例：
>
> ```java
> @Bean(name = "runner")
> @Scope("prototype")
> public QueryRunner createQueryRunner(@Qualifier("dataSource") DataSource dataSource){
>      return new QueryRunner(dataSource);
> }
> ```

**@Resource**

* 作用：直接按照 Bean 的 id 注入。它也只能注入其他 bean 类型。
* 属性：name：指定 bean 的 id。

> 以上三个注入@AutoWrite，@Qualifier，@Resource都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。
>
> 另外，集合类型的注入只能通过XML来实现。

**@Value**

* 作用：注入基本数据类型和 String 类型数据的，可以使用 spring 中SpEL(也就是spring的el表达式），SpEL的写法：`${表达式}`

* 属性：value：用于指定值

```java
@Value("${jdbc.driver}")
private String driver;
@Value("${jdbc.url}")
private String url;
@Value("${jdbc.user}")
private String user;
@Value("${jdbc.password}")
private String password;
```

#### 2.2.3 用于改变作用范围

相当于： `<bean id="" class="" scope="">`

**@Scope**

* 作用：指定 bean 的作用范围。
* 属性：value：指定范围的值。
* 取值： singleton(单例模式) / prototype(多例模式) / request / session / global session

#### 2.2.4 和生命周期相关

相当于： `<bean id="" class="" init-method="" destroy-method="" />`

**@PostConstruct**

作用：用于指定初始化方法。

```java
@PostConstruct
public void init(){
    System.out.println("初始化方法执行了");
}
```

**@PreDestroy**

作用：用于指定销毁方法。

```java
@PreDestroy
public void destory(){
    System.out.println("对象销毁了");
}
```

#### 2.2.5 关于Spring注解和XML的选择问题

注解的优势：配置简单，维护方便（找到类，就相当于找到了对应的配置）。

XML 的优势：修改时，不用改源码。不涉及重新编译和部署。

Spring 管理 Bean 方式的比较：  

|                            | 基于XML配置                                | 基于注释配置                                                 |
| -------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Bean定义                   | `<bean id="..." class="...">`              | @Component<br />衍生类：@Repository / @Service / @Controller |
| Bean名称                   | 通过id或name指定                           | @Component("...")                                            |
| Bean注入                   | `<property>`或p命名空间                    | @Autowired 按类型注入<br />@Qualifier 按名称注入             |
| 生命过程<br />Bean作用范围 | init-method<br />destory-method<br />scope | @PostConstruct<br />@PreDestory<br />@Scope 设置作用范围     |
| 适合场景                   | Bean 来自第三方                            | Bean 的实现类由用户自己开发                                  |

### 2.4 Spring管理对象细节

基于注解的 Spring IoC 配置中， bean 对象的特点和基于 XML 配置是一模一样的。

### 2.5 Spring的纯注解配置

写到此处，基于注解的 IoC 配置已经完成，但是有一个问题：此处依然离不开 spring 的 xml 配置文件，那么能不能不写这个 bean.xml，所有配置都用注解来实现呢？

#### 2.5.1 待改造的问题

发现之所以现在离不开 xml 配置文件，是因为有一句很关键的配置：

```xml
<!-- 告知spring框架在，读取配置文件，创建容器时，扫描注解，依据注解创建对象，并存入容器中 -->
<context:component-scan base-package="cn.xyc"></context:component-scan>
```

如果他要也能用注解配置，那么离脱离 xml 文件又进了一步。

另外，数据源和 JdbcTemplate 的配置也需要靠注解来实现。  

```xml
<!-- 配置 dbAssit -->
<bean id="dbAssit" class="cn.xyc.dbassit.DBAssit">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///db2"></property>
    <property name="user" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

#### 2.5.2 新注解说明

##### 2.5.2.1 @Configuration

作用：用于指定当前类是一个 spring 配置类， 当创建容器时会从该类上加载注解。 获取容器时需要使用 `AnnotationApplicationContext` (有 @Configuration 注解的类.class)。

属性：value：用于指定配置类的字节码  

示例代码：  

```java
// spring 的配置类，相当于 bean.xml 文件
@Configuration
public class SpringConfiguration {
}
```

注意：此时已经把配置文件用类来代替了， 但是如何配置创建容器时要扫描的包呢？请看下一个注解。  

##### 2.5.2.2 @ComponentScan

作用：用于指定 spring 在初始化容器时要扫描的包。 作用和在 spring 的 xml 配置文件中的：`<context:component-scan base-package="cn.xyc"/>`是一样的。

属性：basePackages：用于指定要扫描的包。和该注解中的 value 属性作用一样。

示例代码：  

```java
// spring 的配置类，相当于 bean.xml 文件
@Configuration
@ComponentScan("cn.xyc")
public class SpringConfiguration {
}
```

注意：此时已经配置好了要扫描的包，但是数据源和 JdbcTemplate 对象如何从配置文件中移除呢？请看下一个注解。

##### 2.5.2.3 @Bean

作用：**该注解只能写在方法上**，表明使用此方法创建一个对象，并且放入 spring 容器。

属性：name：给当前@Bean 注解方法创建的对象指定一个名称(即 bean 的 id）。

示例代码：  

```java
// 连接数据库的配置类
public class JdbcConfig {

    //  创建一个数据源，并存入 spring 容器中
    @Bean(name="dataSource")
    public DataSource createDataSource() {
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setUser("root");
            ds.setPassword("root");
            ds.setDriverClass("com.mysql.jdbc.Driver");
            ds.setJdbcUrl("jdbc:mysql:///db2");
            return ds;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    //  创建一个 DBAssit，并且也存入 spring 容器中
    @Bean(name="dbAssit")
    public DBAssit createDBAssit(DataSource dataSource) {
        return new DBAssit(dataSource);
    }
}
```

注意：此时已经把数据源和 DBAssit 从配置文件中移除了，此时可以删除 bean.xml 了。

但是由于没有了配置文件，创建数据源的配置又都写死在类中了。如何把它们配置出来呢？请看下一个注解。

##### 2.5.2.4 @PropertySource

作用：用于加载 `.properties` 文件中的配置。例如在配置数据源时，可以把连接数据库的信息写到 properties 配置文件中，就可以使用此注解指定 properties 配置文件的位置。

属性：`value[]`：用于指定 properties 文件位置。如果是在类路径下，需要写上 `classpath:`

示例代码：

```java
// 连接数据库的配置类
@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
    // 创建一个数据源，并存入 spring 容器中
    @Bean(name="dataSource")
    public DataSource createDataSource() {
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

`jdbc.properties` 文件：

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db2
jdbc.username=root
jdbc.password=root
```

注意：此时已经有了两个配置类，但是他们还没有关系。如何建立他们的关系呢？请看下一个注解。  

##### 2.5.2.5 @Import

作用：用于导入其他配置类，在引入其他配置类时，可以不用再写 `@Configuration` 注解。 当然，写上也没问题。

属性：`value[]`：用于指定其他配置类的字节码。

示例代码：

```java
@Configuration
@ComponentScan(basePackages = "cn.xyc.spring")
@Import({ JdbcConfig.class})
public class SpringConfiguration {
}

@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig{
}
```

注意：此时已经把要配置的都配置好了，但是新的问题产生了，由于没有配置文件了，如何获取容器呢？请看下一小节  

##### 2.5.2.6 通过注解获取容器

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
```

## 3. Spring整合Junit  

### 3.1 测试类中的问题和解决思路

**问题**

* 在测试类中，每个测试方法都有以下两行代码：

  ```java
  ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
  IAccountService as = ac.getBean("accountService",IAccountService.class);
  ```

* 这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常。所以又不能轻易删掉。

**解决思路分析**

* 针对上述问题，这里需要的是程序能自动帮我们创建容器。一旦程序能自动的创建 spring 容器，则就无须手动创建了，问题也就解决了。
* 考虑到 junit 单元测试的原理，但显然， junit 是无法实现的，因为它自己都无法知晓我们是否使用了 spring 框架，更不用说创建 spring 容器了。不过好在， junit 暴露了一个注解，可以让我们替换掉它的运行器。
* 这时，我们需要依靠 spring 框架，因为它提供了一个运行器，可以读取配置文件（或注解）来创建容器。我们只需要告诉它配置文件在哪就行了。

### 3.2 配置步骤

**第一步：导包，在pom.xml文件中导入以下的依赖包**

```xml
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.0.2.RELEASE</version>
</dependency>
```

**第二步：使 用@RunWith 注解替换原有运行器**

```java
// 测试类
@RunWith(SpringJUnit4ClassRunner.class)
public class AccountServiceTest {
}
```

**第三步：使用@ContextConfiguration 指定 spring 配置文件的位置**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations= {"classpath:bean.xml"})
public class AccountServiceTest {
}  
```

* `@ContextConfiguration` 注解：
  * locations 属性： 用于指定配置文件的位置。如果是类路径下，需要用 `classpath:`表明
  * classes 属性： 用于指定注解的类。当不使用 xml 配置时，需要用此属性指定注解类的位置，如：`@ContextConfiguration(classes = SpringConfiguration.class)`

**第四步：使用@Autowired 给测试类中的变量注入数据**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations= {"classpath:bean.xml"})
public class AccountServiceTest {

    @Autowired
    private IAccountService as ;
}  
```

### 3.3 为什么不把测试类配到xml中

首先，配到 XML 中能不能用呢？答案是肯定的，没问题，可以使用。

那么为什么不采用配置到 xml 中的方式呢？

这个原因是这样的：

* 第一：当在 xml 中配置了一个 bean， spring 加载配置文件创建容器时，就会创建对象。
* 第二：测试类只是在测试功能时使用，而在项目中它并不参与程序逻辑，也不会解决需求上的问题，所以创建完了，并没有使用。那么存在容器中就会造成资源的浪费。

所以，基于以上两点，不应该把测试配置到 xml 文件中。  