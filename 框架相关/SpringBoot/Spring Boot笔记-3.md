# SpringBoot：学习笔记-3

> 虽然说之前已经大致过了 SpringBoot，但也仅限于了解层面，这里打算稍微再系统些的学习一下，于是找了 bilibili 上播放量较高的雷丰阳老师的课程进行学习，课程连接：[SpringBoot](https://www.bilibili.com/video/BV1Et411Y7tQ?from=search&seid=11084747628285566436)，在学习过程中顺带做个学习笔记。
>
> SpringBoot 学习笔记总共分为5篇，此为 3/5 篇。

## 5. SpringBoot与数据访问

### 5.1 配置环节

#### 5.1.1 基本使用

1. 创建springboot工程时，在pom文件中导入相应的依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2. 在application.yml配置文件中添加自定义配置

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/db1?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

3. 测试程序：

```java
@SpringBootTest
class Springboot09ApplicationTests {

    @Autowired
    HikariDataSource dataSource;  // 2.x

    @Test
    void contextLoads() throws SQLException {

        System.out.println(dataSource.getClass());

        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
}
```

4. 输出：

```java
class com.zaxxer.hikari.HikariDataSource
HikariProxyConnection@1825984232 wrapping com.mysql.cj.jdbc.ConnectionImpl@620c8641
```

5. 结论

   默认是用`com.zaxxer.hikari.HikariDataSource`作为数据源；

   数据源的相关配置都在**DataSourceProperties**里面；

#### 5.1.2 自动配置原理

在External Libraries下找到：`org.springframework.boot.autoconfigure.jdbc`：

1）参考`DataSourceConfiguration`，根据配置创建数据源，默认使用Hikari连接池；可以使用spring.datasource.type指定自定义的数据源类型；

2）SpringBoot默认可以支持；

* org.apache.tomcat.jdbc.pool.DataSource
* com.zaxxer.hikari.HikariDataSource
* org.apache.commons.dbcp2.BasicDataSource

3）自定义数据源类型

```java
/**
 * Generic DataSource configuration.
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        // 使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
        return properties.initializeDataSourceBuilder().build();
    }
}
```

4）DataSourceAutoConfiguration类在注解中中导入了DataSourceInitializationConfiguration：

```java
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {
    
}
```

DataSourceInitializationConfiguration类在注解中又导入了DataSourceInitializerInvoker

```java
@Configuration(proxyBeanMethods = false)
@Import({ DataSourceInitializerInvoker.class, DataSourceInitializationConfiguration.Registrar.class })
class DataSourceInitializationConfiguration {

}
```

DataSourceInitializerInvoker类中实现了一个ApplicationListener监听器

```java
/**
 * Bean to handle {@link DataSource} initialization by running {@literal schema-*.sql} on
 * {@link InitializingBean#afterPropertiesSet()} and {@literal data-*.sql} SQL scripts on
 * a {@link DataSourceSchemaCreatedEvent}.
 *
 * @author Stephane Nicoll
 * @see DataSourceAutoConfiguration
 */
class DataSourceInitializerInvoker implements ApplicationListener<DataSourceSchemaCreatedEvent>, InitializingBean {
}
```

这个类的作用：

​		1）在`afterPropertiesSet()`这个方法中运行建表语句；

​		2）在`onApplicationEvent()`这个方法中运行插入数据的sql语句；

建表/插入语句的规则：默认只需要将文件命名为：

```properties
建表：schema-*.sql
默认规则：schema.sql，schema-all.sql；
添加数据：data-*.sql

也可以使用如下自定义配置修改默认配置：   
	schema:
      - classpath:department.sql
      指定位置
```

测试：

1. 在resources文件下添加department.sql文件；

```sql
DROP TABLE IF EXISTS `department`;
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

2. 将配置文件修改如下：

```yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/db1?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    schema:
      - classpath:department.sql  # 这里没有空格
    initialization-mode: always
```

3. 测试：运行程序，在数据库中成功建表

5）操作数据库：在JdbcTemplateAutoConfiguration配置类中，自动配置了JdbcTemplate操作数据库

```java
// 进行测试：
@Controller
public class HelloController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @GetMapping("/query")
    @ResponseBody
    public List map(){
        List<Map<String, Object>> maps = jdbcTemplate.queryForList("SELECT * from city");

        return maps;
    }
}
```

访问，输出结果：

```json
[{"city_id":1,"city_name":"西安","country_id":1},{"city_id":2,"city_name":"NewYork","country_id":2},{"city_id":3,"city_name":"北京","country_id":1},{"city_id":4,"city_name":"上海","country_id":1}]
```

### 5.2 整合Druid数据源

#### 5.2.1 基本使用

在pom文件中引入Druid数据源

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```

修改application.yml配置文件：

```yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/db1?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
```

在测试文件中测试是否导入了Druid数据源：

```java
@SpringBootTest
class Springboot09ApplicationTests {

	@Autowired
    DataSource dataSource;  // 2.x

	@Test
	void contextLoads() throws SQLException {

        System.out.println(dataSource.getClass());

        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
}
```

输出：

```java
class com.alibaba.druid.pool.DruidDataSource
com.mysql.cj.jdbc.ConnectionImpl@5555ffcf
```

#### 5.2.2 添加自定义配置

在配置文件中再加入以下配置属性：

```yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/db1?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

	# 以下配置属性由于在DateSourceProperties中并没有对应属性，因此无法生效
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

上述添加的配置属性由于在DateSourceProperties中并没有对应属性，因此无法生效

可通过在注入`dataSource`是Debug查看

若想要上述配置生效，则需要自己添加配置类：

```java
// 导入druid数据源
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }
}
```

通过Debug后，可以看到配置文件中的属性都完成了注入。

#### 5.2.3 添加Servlet和Filter

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return  new DruidDataSource();
    }

    // 配置Druid的监控
    // 1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}
```

### 5.3 整合MyBatis

#### 5.3.1 基本配置

导入依赖：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

![mybatis依赖](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314511.png)

步骤：

1. 配置数据源相关属性（使用上一节的相关配置，包括）

2. 给数据库建表：通过在resource文件夹下添加sql文件建表

   通过在resource文件夹下添加sql文件建表

   在resources文件夹下新增sql文件夹，并添加以下文件：

   ```mysql
   -- department.sql
   SET FOREIGN_KEY_CHECKS=0;
   
   -- ----------------------------
   -- Table structure for department
   -- ----------------------------
   DROP TABLE IF EXISTS `department`;
   CREATE TABLE `department` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `departmentName` varchar(255) DEFAULT NULL,
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
   
   
   -- employee.sql
   SET FOREIGN_KEY_CHECKS=0;
   
   -- ----------------------------
   -- Table structure for employee
   -- ----------------------------
   DROP TABLE IF EXISTS `employee`;
   CREATE TABLE `employee` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `lastName` varchar(255) DEFAULT NULL,
     `email` varchar(255) DEFAULT NULL,
     `gender` int(2) DEFAULT NULL,
     `d_id` int(11) DEFAULT NULL,
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
   ```

   通过springboot创建数据库表，则需要在application.yml下添加：

   ```yml
   spring:
     datasource:
     	# ...
   	# 创建表
       schema:
         - classpath:sql/department.sql
         - classpath:sql/employee.sql
       initialization-mode: always
   ```

3. 创建JavaBean

   ```java
   public class Employee {
   
       private Integer id;
       private String lastName;
       private Integer gender;
       private String email;
       private Integer dId;
   }
   
   public class Department {
   
       private Integer id;
       private String departmentName;
   }
   ```

#### 5.3.2 注解版

1. 数据库操作的mapper

```java
// 指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);
	
    // Options 把自动生成的主键重新封装进Department对象中
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```

测试Controller：

```java
@RestController
public class DeptController {

    @Autowired
    DepartmentMapper departmentMapper;

    @GetMapping("/dept/{id}")
    public Department getDepartment(@PathVariable("id") Integer id){

        return departmentMapper.getDeptById(id);
    }

    @GetMapping("/dept")
    public Department insertDept(Department department){
        departmentMapper.insertDept(department);
        return department;
    }
}
```

进行测试：

```
url:http://localhost:8080/dept/1
相应：空

url：http://localhost:8080/dept?departmentName=AA
相应：{"id":1,"departmentName":"AA"}

url:http://localhost:8080/dept/1
响应：{"id":1,"departmentName":"AA"}
```

**问题：**

将department数据库中的departmentName列修改为department_name，则会出现数据库列和department类无法对应的问题，即无法完成注入了。

解决：自定义MyBatis的配置规则；

给容器中添加一个ConfigurationCustomizer；

```java
package cn.xyc.springboot10.config;

import org.apache.ibatis.session.Configuration;
import org.mybatis.spring.boot.autoconfigure.ConfigurationCustomizer;
import org.springframework.context.annotation.Bean;

@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                // 开启驼峰命名法映射规则
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

**另一个注解说明：**

可以使用MapperScan批量扫描所有的Mapper接口；

```java

@MapperScan(value = "com.atguigu.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
	}
}
```

#### 5.3.3 配置文件版

1. 添加相应的配置文件：

   resources/mabatis/mybatis-config.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <!--开启驼峰命名法映射规则-->
       <settings>
           <setting name="mapUnderscoreToCamelCase" value="true"/>
       </settings>
   </configuration>
   ```

   resources/mabatis/mapper/EmployeeMapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="cn.xyc.springboot10.mapper.EmployeeMapper">
   
       
       <select id="getEmpById" resultType="cn.xyc.springboot10.bean.Employee" parameterType="Integer">
           select * from employee where id=#{id}
       </select>
   
       <select id="insertEmp" parameterType="cn.xyc.springboot10.bean.Employee">
           insert into employee(lastName, email, gender, d_id) values (#{lastName}, #{email}, #{gender}, #{dId})
       </select>
   
   </mapper>
   ```

2. 在application.yml文件下添加内容：

   ```yml
   mybatis:
     config-location: classpath:mybatis/mybatis-config.xml # 指定全局配置文件的位置
     mapper-locations: classpath:mybatis/mapper/*.xml  	# 指定sql映射文件的位置
   ```

3. 添加Contorller进行测试：

   ```java
   @RestController
   public class EmpController {
   
       @Autowired
       EmployeeMapper employeeMapper;
   
   
       @GetMapping("/emp/{id}")
       public Employee getEmp(@PathVariable("id") Integer id){
           return employeeMapper.getEmpById(id);
       }
   
       @GetMapping("/emp")
       public Employee insertEmp(Employee employee){
           employeeMapper.insertEmp(employee);
           return employee;
       }
   
   }
   ```

4. 测试结果：

   ```
   URL：http://localhost:8080/emp/1
   结果：空
   
   URL：http://localhost:8080/emp?lastName=zhangsan&gender=1&email=zhagnsan@163.com&dId=1
   结果：{"id":null,"lastName":"zhangsan","gender":1,"email":"zhagnsan@163.com","dId":1}
   
   URL：http://localhost:8080/emp/1
   结果：{"id":null,"lastName":"zhangsan","gender":1,"email":"zhagnsan@163.com","dId":1}
   ```

更多使用参照：[官方文档](https://mybatis.org/mybatis-3/zh/getting-started.html)

### 5. 4 整合SpringData JPA

#### 5.4.1 SpringData简介

![springdata](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314379.png)

#### 5.4.2 整合SpringData JPA

1. 编写application.yml

```properties
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/db1?serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

2. 由于SpringData JPA是基于ORM（Object Relational Mapping）思想的，因此编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
package cn.xyc.springboot11.enitty;

import javax.persistence.*;

// 使用JPA注解配置映射关系
@Entity // 告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") // @Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {
    
    @Id  // 这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 自增主键
    private Integer id;

    @Column(name = "last_name",length = 50)  // 这是和数据表对应的一个列
    private String lastName;
    @Column  // 省略默认列名就是属性名
    private String email;
}
```

3. 编写一个Dao接口来操作实体类对应的数据表（Repository）

```java
// 继承JpaRepository来完成对数据库的操作
// 泛型1：需要操作的实体类   泛型2：实体类主键类型
public interface UserRepository extends JpaRepository<User,Integer> {
}
```

4. 再加上配置JpaProperties

```yaml
spring:  
 jpa:
    hibernate:
      # 更新或者创建数据表结构
      ddl-auto: update
    # 控制台显示SQL
    show-sql: true
```

启动应用程序，可见控制台输出：

```
Hibernate: create table tbl_user (id integer not null auto_increment, email varchar(255), last_name varchar(50), primary key (id)) engine=InnoDB
```

创建了新的数据表

#### 5.4.3 增删改查

创建Controller：

```java
@RestController
public class UserController {

    @Autowired
    UserRepository userRepository;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){
        return userRepository.findById(id).orElse(null);
    }

    @GetMapping("/user")
    public void insertUser(User user){
        userRepository.save(user);
        System.out.println(user);
    }
}

```

测试：

```
URL：http://localhost:8080/user/1
结果：空

URL：http://localhost:8080/user?lastName=zhangsan&email=zhangsan@123.com
控制台：
Hibernate: insert into tbl_user (email, last_name) values (?, ?)
User{id=1, lastName='zhangsan', email='zhangsan@123.com'}

URL：http://localhost:8080/user/1
结果：{"id":1,"lastName":"zhangsan","email":"zhangsan@123.com"}
```

## 7. 启动配置原理

**几个重要的事件回调机制：**

配置在META-INF/spring.factories

* **ApplicationContextInitializer**

* **SpringApplicationRunListener**

只需要放在ioc容器中

* **ApplicationRunner**

* **CommandLineRunner**

### 7.1 创建SpringApplication对象

```java
@SpringBootApplication
public class Springboot11Application {
	public static void main(String[] args) {
        // 1.首先在这里调用run方法
		SpringApplication.run(Springboot11Application.class, args);
	}
}

public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    // 2.然后在run方法里继续调用run
    return run(new Class<?>[] { primarySource }, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    // 3. 在这里创建的一个行的SpringApplication对象，然后运行了run
    return new SpringApplication(primarySources).run(args);
}

// 4. 通过SpringApplication的构造函数创建SpringApplication对象
@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // 保存主配置类
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 判断当前是否一个web应用
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 从多个配置类中找到有main方法的主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}

private Class<?> deduceMainApplicationClass() {
    try {
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) {
            // 判断传入的类中那个类有main方法
            if ("main".equals(stackTraceElement.getMethodName())) {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    }
    catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
    return null;
}
```

> ```java
> // 从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
> public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
>     this.initializers = new ArrayList<>(initializers);
> }
> ```
>
> ![捕获](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314025.PNG)
>
> ```java
> public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
>     this.listeners = new ArrayList<>(listeners);
> }
> ```
>
> ![Listeners](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314377.PNG)

### 7.2 运行run方法

```java
// 1.在创建完SpringApplication后，运行其run方法
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}

// 2.进入run方法了
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
	// 声明一个空的IOC容器
    ConfigurableApplicationContext context = null;
    FailureAnalyzers analyzers = null;
	// awt相关配置
    configureHeadlessProperty();
    // 获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 回调所有的获取SpringApplicationRunListener.starting()方法
    listeners.starting();
    try {
        // 封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        // 准备环境
        // 创建环境完成后回调SpringApplicationRunListener.environmentPrepared();表示环境准备完成
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                                                                 applicationArguments);
        // 控制图台中打印spring图标
        Banner printedBanner = printBanner(environment);

        // 创建ApplicationContext；决定创建web的ioc还是普通的ioc
        context = createApplicationContext();
        // 异常相关
        analyzers = new FailureAnalyzers(context);
        
        // 准备上下文环境中：将environment保存到ioc中；且applyInitializers()；
        //	  而applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的initialize方法
        //    回调所有的SpringApplicationRunListener的contextPrepared()；
        prepareContext(context, environment, listeners, applicationArguments,
                       printedBanner);
        // prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）；

        // 刷新容器；ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）
        // 扫描，创建，加载所有组件的地方；（配置类，组件，自动配置）
        refreshContext(context);
        // 从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
        // ApplicationRunner先回调，CommandLineRunner再回调
        afterRefresh(context, applicationArguments);
        // 所有的SpringApplicationRunListener回调finished方法
        listeners.finished(context, null);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        // 整个SpringBoot应用启动完成以后返回启动的ioc容器；
        return context;
    }
    catch (Throwable ex) {
        handleRunFailure(context, listeners, analyzers, ex);
        throw new IllegalStateException(ex);
    }
}
```

### 7.3 事件监听机制

配置在META-INF/spring.factories

**ApplicationContextInitializer**

```java
public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("ApplicationContextInitializer...initialize..."+applicationContext);
    }
}

```

**SpringApplicationRunListener**

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {

    // 必须有的构造器
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args){

    }

    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting...");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared.."+o);
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared...");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextLoaded...");
    }

    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener...finished...");
    }
}
```

配置（resources/META-INF/spring.factories）

```properties
org.springframework.context.ApplicationContextInitializer=\
cn.xyc.springboot11.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
cn.xyc.springboot11.listener.HelloSpringApplicationRunListener
```

只需要放在ioc容器中

**ApplicationRunner**

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run....");
    }
}
```

**CommandLineRunner**

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
    }
}
```

## 8. 自定义starter

**starter：**

1. 这个场景需要使用到的依赖是什么？

2. 如何编写自动配置

   ```properties
   @Configuration       // 指定这个类是一个配置类
   @ConditionalOnXXX    // 在指定条件成立的情况下自动配置类生效
   @AutoConfigureAfter  // 指定自动配置类的顺序
   @Bean                // 给容器中添加组件
   
   @ConfigurationPropertie         // 结合相关xxxProperties类来绑定相关的配置
   @EnableConfigurationProperties  // 让xxxProperties生效加入到容器中
   
   // 自动配置类要能加载
   // 将需要启动就加载的自动配置类，配置在META-INF/spring.factories
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
   ```

3. 模式：
   1. 启动器只用来做依赖导入；
   2. 专门来写一个自动配置模块；
   3. 启动器依赖自动配置；别人只需要引入启动器（starter）
   4. 命名：自定义启动器名-spring-boot-starter，如：mybatis-spring-boot-starter；

4. 步骤：

1）创建一个新的Maven工程作为启动器模块`atguigu-spring-boot-starter`，只用来做依赖导入，其配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--启动器-->
    <dependencies>
        <!--引入自动配置模块-->
        <dependency>
            <groupId>com.atguigu.starter</groupId>
            <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

2）创建一个springboot工程作为自动配置模块`atguigu-spring-boot-starter-autoconfigurer`，启动器依赖这个自动配置，其pom.xml文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>atguigu-spring-boot-starter-autoconfigurer</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>

        <!--引入spring-boot-starter；所有starter的基本配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

    </dependencies>
</project>
```

创建一个properties配置文件类，绑定配置文件中响应的配置，这些属性就是可以被覆盖配置的，如下：

```java
package com.atguigu.starter;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "atguigu.hello")
public class HelloProperties {

    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

创建一个HelloService类，后续通过HelloServiceAutoConfiguration中@Bean的注解将该类注入到容器中：

```java
package com.atguigu.starter;

public class HelloService {

    HelloProperties helloProperties;

    public HelloProperties getHelloProperties() {
        return helloProperties;
    }

    public void setHelloProperties(HelloProperties helloProperties) {
        this.helloProperties = helloProperties;
    }

    public String sayHellAtguigu(String name){
        return helloProperties.getPrefix()+"-" +name + helloProperties.getSuffix();
    }
}
```

自动配置类：

```java
package com.atguigu.starter;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration  // 配置类
@ConditionalOnWebApplication // web应用才生效
@EnableConfigurationProperties(HelloProperties.class)
public class HelloServiceAutoConfiguration {

    @Autowired
    HelloProperties helloProperties;
    
    // 把helloService组件注入到容器中
    @Bean
    public HelloService helloService(){
        HelloService service = new HelloService();
        service.setHelloProperties(helloProperties);
        return service;
    }
}
```

类路径下添加文件：resources/MATE-INFO/spring.factories

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.atguigu.starter.HelloServiceAutoConfiguration
```

3）将两个配置包`atguigu-spring-boot-starter`，`atguigu-spring-boot-starter-autoconfigurer`都安装到仓库中，通过Maven的install命令

4）新创建的springboot工程就可以在pom文件中添加自定义启动器的坐标了

```xml
<!--引入自定义的starter-->
<dependency>
    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

5）写一个Contorller进行helloService的注入测试

```java
@RestContorller
public class HelloController{
	@Autowired
    HelloService helloSerivce;
    
    @GetMapping("/hello")
    public String hello(){
        return helloSerivce.sayHellAtguigu("haha");
    }
}
```

在配置文件中修改prefix与suffix

```properties
atguigu.hello.perfix=ATGUIGU
atguigu.hello.suffix=HELLO WORLD
```

访问：loaclhost:8080/hello

页面结果：ATGUIGU-hahaHELLO WORLD

