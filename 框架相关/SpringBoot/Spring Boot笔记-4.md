# SpringBoot：学习笔记-4

> 虽然说之前已经大致过了 SpringBoot，但也仅限于了解层面，这里打算稍微再系统些的学习一下，于是找了 bilibili 上播放量较高的雷丰阳老师的课程进行学习，课程连接：[SpringBoot](https://www.bilibili.com/video/BV1Et411Y7tQ?from=search&seid=11084747628285566436)，在学习过程中顺带做个学习笔记。
>
> SpringBoot 学习笔记总共分为5篇，此为 4/5 篇。

## Spring Boot与缓存

### 1. JSR107

Java Caching定义了5个核心接口

* **CachingProvider**：缓存提供者

  定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。

* **CacheManager**：缓存管理器

  定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。

* **Cache**：缓存

  一个类似**Map**的数据结构并**临时存储以Key为索引**的值。一个Cache仅被一个CacheManager所拥有。

* **Entry**

  一个存储在Cache中的key-value对。

* **Expiry**

  每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

关系示意图：

![jsr107示意图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314017.png)

  ### 2. Spring缓存抽象

Spring从3.1开始定义了`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`接口来**统一**不同的缓存技术；**并支持使用JCache（JSR-107）**注解简化我们开发；

Cache接口有以下功能：

* 为缓存的组件规范定义，包含缓存的各种操作集合；

  * Spring提供了各种xxxCache的实现；
    
    如RedisCache，EhCacheCache , ConcurrentMapCache等；

  ![Spring缓存抽象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314814.png)

### 3. 重要缓存注解及概念

| 接口/注解      | 作用                                                         |
| :------------- | ------------------------------------------------------------ |
| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                        |
| @Cacheable     | 根据方法的请求参数对其结果进行缓存                           |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又希望结果被保存，即更新缓存                 |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

#### 3.1 @Cacheable/@CachePut/@CacheEvict 主要参数

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {

	@AliasFor("cacheNames")
	String[] value() default {};
	@AliasFor("value")
	String[] cacheNames() default {};
	String key() default "";
	String keyGenerator() default "";
	String cacheManager() default "";
	String cacheResolver() default "";
	String condition() default "";
	String unless() default "";
	boolean sync() default false;
}
```

* cacheNames/value：缓存组件名称，字符串/字符数组形式；

  作用：CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；

  如：`@Cacheable(value=”mycache”)` 或者 `@Cacheable(value={”cache1”,”cache2”}`；

* key：缓存数据使用的key，需要按照SpEL表达式编写，如果不指定则按照方法所有参数进行组合；

  如：`@Cacheable(value=”testcache”, key=”#userName”)`

* keyGenerator：key的生成器；可以自己指定key的生成器的组件id

  注意：key与keyGenerator：二选一使用;

* cacheManager/cacheResolver：指定缓存管理器/指定缓存解析器

* condition：缓存条件，使用SpEL编写，在调用方法之前之后都能判断；

  如：`@Cacheable(value=”testcache”,condition=”#userName.length()>2”)`

* unless（@CachePut、@Cacheable）：当unless指定的条件为true，方法的返回值就不会被缓存，恰好与condition相反，只在方法执行之后判断；

  如：`@Cacheable(value=”testcache”, unless=”#result ==null”)`

* beforeInvocation（@CacheEvict）：是否在执行前清空缓存，默认为false，false情况下方法执行异常则不会清空；

  如：`@CachEvict(value=”testcache”，beforeInvocation=true)`

* allEntries（@CacheEvict）：是否清空所有缓存内容，默认为false；

  如：`@CachEvict(value=”testcache”,allEntries=true)`

* sync：是否使用异步模式

> 使用示例可参考第4.3节内容

#### 3.2 缓存可用的SpEL表达式

| 名字          | 位置               | 描述                                                         | 示例                 |
| ------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method        | root object        | 当前被调用的方法                                             | #root.method.name    |
| target        | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | root object        | 当前方法调用使用的缓存列表<br />如@Cacheable(value={"cache1", "cache2"})，则有两个cache | #root.caches[0].name |
| argument name | evaluation context | 方法参数的名字.<br />可以直接#参数名 ，也可以使用#p0或#a0的形式，0代表参数的索引 | #iban、#a0、#p0      |
| result        | evaluation context | 方法执行后的返回值，仅当方法执行之后的判断有效<br />如‘unless’ ， @CachePut、@CacheEvict’的表达式beforeInvocation=false | #result              |

### 4. 缓存使用

#### 4.1 搭建实验环境

1. 创建出department和employee表

   ```sql
   -- ----------------------------
   -- Table structure for department
   -- ----------------------------
   DROP TABLE IF EXISTS `department`;
   CREATE TABLE `department` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `departmentName` varchar(255) DEFAULT NULL,
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   
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
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   ```

2. 创建javaBean封装数据

   ```java
   public class Department {
       private Integer id;
       private String departmentName;
   }
   
   public class Employee {
   
       private Integer id;
       private String lastName;
       private String email;
       //1 male, 0 female
       private Integer gender;
       private Integer dId;
   }
   ```

3. 整合MyBatis操作数据库

   1. 配置数据源信息

   ```properties
   # 开启debug
   debug=true
   # 数据库配置
   spring.datasource.username=root
   spring.datasource.password=root
   spring.datasource.url=jdbc:mysql://localhost:3306/db1?serverTimezone=GMT
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   
   # 开启驼峰命名法(否则部分字段封装不了)
   mybatis.configuration.map-underscore-to-camel-case=true
   # 打印sql
   logging.level.cn.edu.ustc.springboot.mapper=debug
   ```

   使用注解版的MyBatis：在启动类上添加注解@MapperScan指定需要扫描的mapper接口所在的包，如下：

   ```java
   @MapperScan("cn.xyc.springbootcache.mapper")
   @SpringBootApplication
   public class SpringbootCacheApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(SpringbootCacheApplication.class, args);
   	}
   }
   ```

4. 创建两个mapper

   ```java
   public interface DepartmentMapper {
   
       @Select("select * from department where id=#{id}")
       public Department getDeptById(Integer id);
   
       @Delete("delete from department where id=#{id}")
       public int deleteDeptById(Integer id);
   
       @Options(useGeneratedKeys = true,keyProperty = "id")
       @Insert("insert into department(departmentName) values(#{departmentName})")
       public int insertDept(Department department);
   
       @Update("update department set departmentName=#{departmentName} where id=#{id}")
       public int updateDept(Department department);
   }
   
   public interface EmployeeMapper {
   
       @Select("select * from employee where id=#{id}")
       public Employee getEmpById(Integer id);
   
       @Update("update employee set lastName=#{lastName}, email=#{email}, gender=#{gender}, d_id=#{dId} where id=#{id}")
       public void updateEmp(Employee employee);
   
   
       @Delete("delete from employee where id=#{id}")
       public void deleteEmp(Integer id);
   
       @Insert("insert into employee (lastName, email, gender, d_id) values (#{lastName}, #{email}, #{gender}, #{dId})")
       public void insertEmp(Employee employee);
   }
   ```

5. 在SpringbootCacheApplicationTests中测试配置是否成功

   ```java
   @SpringBootTest
   class SpringbootCacheApplicationTests {
   
   	@Autowired
   	EmployeeMapper employeeMapper;
   
   	@Test
   	void contextLoads() {
   		Employee emp1 = employeeMapper.getEmpById(1);
   		System.out.println(emp1);
   	}
   }
   ```

   输出：

   ```
   Employee{id=1, lastName='zhangsan', email='zhangsan@qq.com', gender=1, dId=1}
   ```

   **目前没有使用缓存技术，则每一次调用都需要去查询数据库**

#### 4.2 基本使用步骤

1. 引入spring-boot-starter-cache模块

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

2. 在主配置类上标注：@EnableCaching开启缓存

3. 使用缓存注解：如@Cacheable：可缓存的；@CacheEvict：缓存清除；@CachePut：缓存更新

#### 4.3 缓存使用

##### 4.3.1 @Cacheable注解使用

```java
@Service
public class EmployeeService {
    @Autowired
    private EmployeeMapper employeeMapper;

    // 员工查询方法
    @Cacheable(value={"emp"},
            key = "#id+#root.methodName+#root.caches[0].name",
            condition = "#a0>1",
            unless = "#p0==2"
    )
    public Employee getEmpById(Integer id) {
        System.out.println("查询员工："+id);
        return employeeMapper.getEmpById(id);
    }
}
```

> 对于@Cacheable中属性的进一步说明，结合3.2节内容：

**对key的说明：**

```java
// 不加，默认就是参数的值
@Cacheable(value={"emp"}}
// 此时key的值即是id的值
           
@Cacheable(value = "emp", key = "#root.methodName + '[' + #id + ']'")
// 此时key的结果：getEmpById[1]  
```

**对KeyGenerator的说明：**

```java
@Configuration  // 配置类
public class MyCacheConfig {
    @Bean("myKeyGenerator")  // 加在容器中
    public KeyGenerator myKeyGenerator() {
        return new KeyGenerator(){
            @Override
            public Object generate(Object target, Method method, Object... params) {
                // 方法名[参数表]
                return method.getName()+"["+ Arrays.asList(params).toString()+target+"]";
            }
        };
    }
}
```

使用时在注解属性内指定KeyGenerator="myKeyGenerator"

```java
@Cacheable(value = "emp", keyGenerator = "myKeyGenerator")
// 结果：getEmpById[[1]cn.xyc.springbootcache.service.EmployeeService@554a6ced]
```

**对condition属性说明：**

```java
// 在第一个参数的值大于1时才进行缓存
@Cacheable(value = "emp", condition = "#a0>1")
```

**对unless属性说明：**

```java
// 在第一个参数的值为2时不进行缓存
@Cacheable(value = "emp", unless = "#a0 == 2")
```

##### 4.3.2 @CachePut注解使用

@CachePut：即调用方法，又更新缓存数据

运行时机：

1. 先调用目标方法；
2. 将目标方法的结果缓存起来

```java
@Service
public class EmployeeService {
    @Autowired
    private EmployeeMapper employeeMapper;
	
    // 员工更新方法
    // @CachePut(value = {"emp"}, key = "#employee.id" )
    @CachePut(value = {"emp"}, key = "#result.id" )
    public Employee updateEmp(Employee employee) {
        System.out.println("更新员工"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
}
```

测试：

1. 查询1号员工，因为开启了缓存，因此只会查询一次，将查询结果放在缓存中；
2. 以后查询还是之前的结果：`{"id":1,"lastName":"zhangsan","email":"zhagnsan@163.com","gender":1,"dId":1}`；
3. 更新1好员工`http://localhost:8080/emp?id=1&lastName=zhangsan&gender=0`，并将方法的返回值放回缓存中；
4. 再查询1号员工：`{"id":1,"lastName":"zhangsan","email":"zhagnsan@163.com","gender":1,"dId":1}`，发现数据没有变化，还是之前的数据；

为什么呢？**这是由于key的不同**，而导致更新的缓存不同，因此想要实现同步更新，则需要保证key一致即可。

##### 4.3.3 @CacheEvict注解使用

@CacheEvict：清除缓存

```java
@Service
public class EmployeeService {
    @Autowired
    private EmployeeMapper employeeMapper;
	
    // 删除员工的方法
    @CacheEvict(value = {"emp"}, 
                key = "#id", 			 // 删除缓存对应的key
            	allEntries = true,  	 // 删掉缓存中的所有数据，默认false
                beforeInvocation = true  // 缓存的清除是否在方法之前执行；默认为false，表示方法执行之后再清除缓存，当方法出现异常的情况下不会清空缓存
    )
    public Integer delEmp(Integer id){
        System.out.println("删除员工："+id);
        employeeMapper.deleteEmp(id);
        return id;
    }
}
```

##### 4.3.4 @CacheConfig&@Caching

**@CacheConfig**：抽取缓存的公共配置

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CacheConfig {
    String[] cacheNames() default {};
    String keyGenerator() default "";
    String cacheManager() default "";
    String cacheResolver() default "";
}
```

标注在类上，用于抽取@Cacheable的公共属性

由于一个类中可能会使用多次@Cacheable等注解，所以各项属性可以抽取到@CacheConfig

例如：`@CacheConfig(cacheNames = "emp")`，这样就可以注释掉类中@Cacheable中value=“emp”的属性了

**@Caching**：组合多个注解

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {

	Cacheable[] cacheable() default {};

	CachePut[] put() default {};

	CacheEvict[] evict() default {};

}
```

组合使用@Cacheable、@CachePut、@CacheEvict

```java
// service
@Caching(
    cacheable = {
        @Cacheable(value="emp", key = "#lastName")
    },
    put = {
        @CachePut(value="emp", key = "#result.id"),
        @CachePut(value="emp", key = "#result.email")
    }
)
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}

// controller
@GetMapping("/emp/lastname/{lastName}")
public Employee getEmpByLastName(String lastName){
    return employeeService.getEmpByLastName(lastName);
}

// mapper
@Select("select * from employee where lastName=#{lastName}")
Employee getEmpByLastName(String lastName);
```

在调用controller后，在同名emp.id查询数据库时，已经可以从缓存中进行查找了；

同理通过email查询也不需要查询数据库了，可以从缓存中进行查找；

但是，通过lastName进行查询还是会查询数据库，因为@CachePut的存在，使得方法`getEmpByLastName`必定会被调用，即会走一遍数据库查询。

#### 4.4 工作原理

1. 自动配置类：缓存的自动配置类`CacheAutoConfiguration`

   ```java
   @Configuration
   @ConditionalOnClass(CacheManager.class)
   @ConditionalOnBean(CacheAspectSupport.class)
   @ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
   @EnableConfigurationProperties(CacheProperties.class)
   @AutoConfigureBefore(HibernateJpaAutoConfiguration.class)
   @AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
   		RedisAutoConfiguration.class })
   @Import(CacheConfigurationImportSelector.class)
   public class CacheAutoConfiguration {
   
   }
   ```

2. 通过`@Import(CacheConfigurationImportSelector.class)`，该类的`selectImports()`方法添加了许多配置类，其中`SimpleCacheConfiguration`默认生效

   ```java
   static class CacheConfigurationImportSelector implements ImportSelector {
       @Override
       public String[] selectImports(AnnotationMetadata importingClassMetadata) {
           CacheType[] types = CacheType.values();
           String[] imports = new String[types.length];
           for (int i = 0; i < types.length; i++) {
               // 向容器中导入多个配置类
               imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
           }
           return imports;
       }
   }
   ```

3. 通过`CacheConfigurationImportSelector`类中的`selectImports`方法，导入的组件有：`org.springframework.boot.autoconfigure.cache.xxxConfigureation`

   * GenericCacheConfiguration
   * JCacheCacheConfiguration
   * EhCacheCacheConfiguration
   * HazelcastCacheConfiguration
   * InfinispanCacheConfiguration
   * CouchbaseCacheConfiguration
   * RedisCacheConfiguration
   * CaffeineCacheConfiguration
   * GuavaCacheConfiguration
   * **SimpleCacheConfiguration：默认生效**
   * NoOpCacheConfiguration

4. 这些判断类的生效规则：

   例如对于`GenericCacheConfiguration`类

   ```java
   @Configuration
   @ConditionalOnBean(Cache.class)  // 容器中有Cache
   @ConditionalOnMissingBean(CacheManager.class)  // 容器中没有CacheManager组件
   @Conditional(CacheCondition.class)
   class GenericCacheConfiguration {
   
       private final CacheManagerCustomizers customizers;
   
       GenericCacheConfiguration(CacheManagerCustomizers customizers) {
           this.customizers = customizers;
       }
   
       @Bean
       public SimpleCacheManager cacheManager(Collection<Cache> caches) {
           SimpleCacheManager cacheManager = new SimpleCacheManager();
           cacheManager.setCaches(caches);
           return this.customizers.customize(cacheManager);
       }
   }
   ```

   对于默认生效的`SimpleCacheConfiguration`：

   ```java
   @Configuration
   @ConditionalOnMissingBean(CacheManager.class)
   @Conditional(CacheCondition.class)
   class SimpleCacheConfiguration {
   
   	private final CacheProperties cacheProperties;
   
   	private final CacheManagerCustomizers customizerInvoker;
   
   	SimpleCacheConfiguration(CacheProperties cacheProperties,
   			CacheManagerCustomizers customizerInvoker) {
   		this.cacheProperties = cacheProperties;
   		this.customizerInvoker = customizerInvoker;
   	}
   
       // 向容器中导入ConcurrentMapCacheManager
   	@Bean
   	public ConcurrentMapCacheManager cacheManager() {
           // 新建一个缓存管理器
   		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
   		List<String> cacheNames = this.cacheProperties.getCacheNames();
   		if (!cacheNames.isEmpty()) {
   			cacheManager.setCacheNames(cacheNames);
   		}
   		return this.customizerInvoker.customize(cacheManager);
   	}
   }
   ```

   即：**SimpleCacheConfiguration向容器中注册了ConcurrentMapCacheManager**；

5. 而`ConcurrentMapCacheManager`用来管理缓存，其可以获取和创建`ConcurrentMapCache`类型的缓存组件，而`ConcurrentMapCache`将数据保存在`ConcurrentMap`中

   ```java
   public class ConcurrentMapCacheManager implements CacheManager, BeanClassLoaderAware {
       // 缓存的map
       private final ConcurrentMap<String, Cache> cacheMap = new ConcurrentHashMap<String, Cache>(16);
   
       // 获取缓存
       @Override
       public Cache getCache(String name) {
           Cache cache = this.cacheMap.get(name);
           if (cache == null && this.dynamic) {
               synchronized (this.cacheMap) {
                   cache = this.cacheMap.get(name);
                   if (cache == null) {
                       // 当无缓存时，创建createConcurrentMapCache类型的缓存
                       cache = createConcurrentMapCache(name);
                       this.cacheMap.put(name, cache);
                   }
               }
           }
           return cache;
       }
   
       protected Cache createConcurrentMapCache(String name) {
           SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
           return new ConcurrentMapCache(name, new ConcurrentHashMap<Object, Object>(256),
                                         isAllowNullValues(), actualSerialization);
   
       }
   }
   ```

   对于`createConcurrentMapCache`类型的缓存组件：

   ```java
   public class ConcurrentMapCache extends AbstractValueAdaptingCache {
   	// store:将数据保存在ConcurrentMap中
       private final ConcurrentMap<Object, Object> store;
       
       // 查询缓存
       @Override
       protected Object lookup(Object key) {
           return this.store.get(key);
       }
       
       // 放入缓存
       @Override
       public void put(Object key, Object value) {
           this.store.put(key, toStoreValue(value));
       }
   }
   ```

   即，`ConcurrentMapCacheManager`使用`ConcurrentMap`以k-v的方式存储缓存缓存，

> 进一步说明：通过Debug发现
>
> 1、在@Cacheable标注方法执行前，会执行CacheAspectSupport类的execute方法，在该方法中会以一定的规则生成key，并尝试在缓存中通过该key获取值，若通过key获取到值则直接返回，不用执行@Cacheable标注方法，否则执行该方法获得返回值。
>
> ```java
> // CacheAspectSupport类中的execute方法：
> @Nullable
> protected Object execute(CacheOperationInvoker invoker, Object target, Method method, Object[] args) {
>     // 在执行@Cacheable标注的方法前执行此方法
>     // Check whether aspect is enabled (to cope with cases where the AJ is pulled in automatically)
>     if (this.initialized) {
>         Class<?> targetClass = getTargetClass(target);
>         CacheOperationSource cacheOperationSource = getCacheOperationSource();
>         if (cacheOperationSource != null) {
>             Collection<CacheOperation> operations = cacheOperationSource.getCacheOperations(method, targetClass);
>             if (!CollectionUtils.isEmpty(operations)) {
>                 // 在类中的execute方法再跳转至execute的重载方法
>                 return execute(invoker, method,
>                                new CacheOperationContexts(operations, method, args, target, targetClass));
>             }
>         }
>     }
>     return invoker.invoke();
> }
> ```
>
> 2、默认情况下使用SimpleKeyGenerator生成key
>
> ```java
> public class SimpleKeyGenerator implements KeyGenerator {
> 
>     @Override
>     public Object generate(Object target, Method method, Object... params) {
>         return generateKey(params);
>     }
> 
>     /**
> 	 * Generate a key based on the specified parameters.
> 	 */
>     public static Object generateKey(Object... params) {
>         if (params.length == 0) {
>             return SimpleKey.EMPTY;
>         }
>         if (params.length == 1) {
>             Object param = params[0];
>             if (param != null && !param.getClass().isArray()) {
>                 return param;
>             }
>         }
>         return new SimpleKey(params);
>     }
> }
> ```

#### 4.5 工作流程

下面以添加了`@Cacheable`注解的方法运行流程为例，说明`ConcurrentMapCacheManager`的作用：

1. 方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；（CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建,并以cacheNames-cache对放入ConcurrentMap。
   
2. 去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
   

SimpleKeyGenerator生成key的默认策略；

* 如果没有参数；key=new SimpleKey()；
   * 如果有一个参数：key=参数的值
   * 如果有多个参数：key=new SimpleKey(params)；
   
3. 没有查到缓存就调用目标方法；

4. 将目标方法返回的结果，放进缓存中

**总结：**

@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据。

**核心：**

1. 使用CacheManager(默认是：ConcurrentMapCacheManager)按照名字得到Cache（ConcurrentMapCache）组件
2. key使用keyGenerator生成的，默认是SimpleKeyGenerator

### 5. Redis与缓存

#### 5.1 环境搭建

1. 安装redis

2. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

3. 在spring.properties指定Redis服务器地址

   ```properties
   #redis服务器主机地址
   spring.redis.host=127.0.0.1
   ```

4. 测试redis

   `RedisTemplate<Object, Object> redisTemplate`：操作k-v都为对象

   `StringRedisTemplate`k-v都为字符串的值

   ```java
   @Autowired
   RedisTemplate redisTemplate;  // k-v都是对象
   
   @Autowired
   StringRedisTemplate stringRedisTemplate;  // 专门操作字符串使用
   
   /**
     * Redis常见的五大数据类型
     *  String（字符串）、List（列表）、Set（集合）、Hash（散列）、ZSet（有序集合）
     *  1. String: stringRedisTemplate.opsForValue()
     *  2. List: stringRedisTemplate.opsForList()
     *  3. Set: stringRedisTemplate.opsForSet()
     *  4. Hash: stringRedisTemplate.opsForHash()
     *  5. ZSet: stringRedisTemplate.opsForZSet()
     */
   @Test
   void testRedis(){
       // 给redis保存了一个数据
       stringRedisTemplate.opsForValue().append("msg", "hello");
       // 从redis中读出一个数据
       String msg = stringRedisTemplate.opsForValue().get("msg");
       System.out.println(msg);
   
       // 列表测试
       stringRedisTemplate.opsForList().leftPush("mylist", "1");
       stringRedisTemplate.opsForList().leftPush("mylist", "2");
       stringRedisTemplate.opsForList().leftPush("mylist", "3");
   }
   
   // 测试保存对象
   @Test
   void testRedisSaveObj(){
       Employee emp1 = employeeMapper.getEmpById(1);
       // 默认如果保存对象，会使用jdk序列化机制，因此对于Employee需要加上Serializable
       redisTemplate.opsForValue().set("emp1", emp1);
   	// 因此保存结果为：
       // string：\xAC\xED\x00\x05t\x00\x04emp1
       // value:\xAC\xED\x00...
   }
   
   // 1. 若想用json的格式保存数据
   //    (1) 自己将对象转为json格式
   //    (2) 使用redisTemplate默认的序列化规则;改变默认的序列化规则
   // 修改redis默认的序列化规则：
   @Configuration
   public class MyRedisConfig {
   
       @Bean
       public RedisTemplate<String, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory)
               throws UnknownHostException {
           RedisTemplate<String, Employee> template = new RedisTemplate<>();
           template.setConnectionFactory(redisConnectionFactory);
           // 设置需要的序列化器
           Jackson2JsonRedisSerializer<Employee> serializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
           template.setDefaultSerializer(serializer);
           return template;
       }
   }
   
   // 测试：
   @Autowired
   RedisTemplate<String, Employee> empRedisTemplate;
   
   @Test
   void testRedisSaveEmp(){
       Employee emp1 = employeeMapper.getEmpById(1);
       empRedisTemplate.opsForValue().set("emp1", emp1);
   }
   // 结果：
   // string: "emp1"
   // values: 	
   // 			{
   // 			  "id": 1,
   // 			  "lastName": "zhangsan",
   // 			  "email": "zhangsan@163.com",
   // 			  "gender": 1,
   // 			  "dId": 1
   // 			}
   ```


> redisTemplate默认的序列化规则：
   >
   > ```java
   > // 在redisTemplate类中有如下代码：
   > private boolean enableDefaultSerializer = true;
   > @Nullable
   > private RedisSerializer<?> defaultSerializer;
   > @Nullable
   > private ClassLoader classLoader;
   > @Nullable
   > private RedisSerializer keySerializer = null;
   > @Nullable
   > private RedisSerializer valueSerializer = null;
   > @Nullable
   > private RedisSerializer hashKeySerializer = null;
   > @Nullable
   > private RedisSerializer hashValueSerializer = null;
   > private RedisSerializer<String> stringSerializer = RedisSerializer.string();
   > 
   > // 默认使用的就是JDK的序列化器
   > this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
   > ```

上述涉及到的两个配置类`RedisTemplate`与`StringRedisTemplate`，通过`RedisAutoConfiguration`配置类注入容器中：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
}
```

#### 5.2 Redis缓存使用

1. 在导入redis的starter依赖后`RedisCacheConfiguration`类就会自动生效，创建`RedisCacheManager`；

2. `RedisCacheManager`中创建`RedisCache`作为缓存组件，来缓存数据；

3. 上述都是springboot帮我们自动配置好的，直接进行测试；此时不需要修改任何代码，因为添加了redis启动器，替换了之前的`SimpleCacheConfiguration`，因此此时使用的都是redis的缓存；

4. 访问`http://localhost:8080/emp/1`，成功后，可以在redis中看到添加了新的缓存内容；

   但是默认情况下是以**jdk序列化数据**存在redis中，如下：

   ```java
   String: "emp::1"
   Value:
   \xAC\xED\x00\x05sr\x00...
   ```

   要想让对象以**json形式**存储在redis中，需要自定义`RedisCacheManager`，使用`GenericJackson2JsonRedisSerializer`类对value进行序列化，同时要缓存的对象的类要实现Serializable接口；

   ```java
   @Configuration
   public class MyRedisConfig {
       @Bean
       RedisCacheManager cacheManager(RedisConnectionFactory factory){
           // 创建默认RedisCacheWriter
           RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(factory);
   
           // 创建默认RedisCacheConfiguration并使用GenericJackson2JsonRedisSerializer构造的SerializationPair对value进行转换
           // 创建GenericJackson2JsonRedisSerializer的json序列化器
           GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
           // 使用json序列化器构造出对转换Object类型的SerializationPair序列化对
           RedisSerializationContext.SerializationPair<Object> serializationPair = RedisSerializationContext.SerializationPair.fromSerializer(jsonRedisSerializer);
           // 将可以把Object转换为json的SerializationPair传入RedisCacheConfiguration
           // 使得RedisCacheConfiguration在转换value时使用定制序列化器
           RedisCacheConfiguration cacheConfiguration=RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(serializationPair);
   
           RedisCacheManager cacheManager = new RedisCacheManager(cacheWriter,cacheConfiguration);
           return cacheManager;
       }
   }
   ```

   最终序列化数据如下：

   ```json
   key: "emp::3"
   
   value:
   {
     "@class": "cn.xyc.springbootcache.bean.Employee",
     "id": 1,
     "lastName": "zhangsan",
     "email": "zhangsan@163.com",
     "gender": 1,
     "dId": 1
   }
   ```

   **注意**，这里必须用**GenericJackson2JsonRedisSerializer**进行value的序列化解析，如果使用Jackson2JsonRedisSerializer，序列化的json没有` "@class": "cn.edu.ustc.springboot.bean.Employee"`,在读取缓存时会报类型转换异常。

## Spring boot与任务

### 1. 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，springboot中可以用异步任务解决。

**两个注解：**

`@Async`：在需要异步执行的方法上标注注解

`@EnableAsync`：在主类上标注开启异步任务支持

开启异步任务后，当controller层调用该方法会直接返回结果，该任务异步执行

```java
@Service
public class AsyncService {
    @Async  // 标明这是一个异步任务
    public void sayHello() {
        try {
            Thread.sleep(3000);
            System.out.println("hello async task!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、TaskScheduler 接口。

**两个注解：**

`@EnableScheduling`：标注在主类，开启对定时任务支持

`@Scheduled`：标注在执行的方法上，并制定cron属性

```java
@Service
public class ScheduleService {
    @Scheduled(cron = "0,1,2,3,4,5,30,50 * * * * 0-7")
    public void schedule() {
        System.out.println("I am executing..");
    }
}
```

**cron表达式**：

 顺序如下：second, minute, hour, day of month, month, day of week.

示例：

`0 0/5 14,18 * * ?` 	每天14点整，和18点整，每隔5分钟执行一次

`  0 15 10 ? * 1-6` 		每个月的周一至周六10:15分执行一次

`0 0 2 ? * 6L`				每个月的最后一个周六凌晨2点执行一次

`0 0 2 LW * ?`				每个月的最后一个工作日凌晨2点执行一次

`0 0 2-4 ? * 1#1`		  每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；

| **字段** | **允许值**             | **允许的特殊字符** |
| -------- | ---------------------- | ------------------ |
| 秒       | 0-59                   | , -  * /           |
| 分       | 0-59                   | , -  * /           |
| 小时     | 0-23                   | , -  * /           |
| 日期     | 1-31                   | , -  * ? / L W C   |
| 月份     | 1-12                   | , -  * /           |
| 星期     | 0-7或SUN-SAT  0,7是SUN | , -  * ? / L C #   |

| **特殊字符** | **代表含义**               |
| ------------ | -------------------------- |
| ,            | 枚举                       |
| -            | 区间                       |
| *            | 任意                       |
| /            | 步长                       |
| ?            | 日/星期冲突匹配            |
| L            | 最后                       |
| W            | 工作日                     |
| C            | 和calendar联系后计算过的值 |
| #            | 星期，4#2，第2个星期四     |

### 3. 邮件任务

> 添加启动类：spring-boot-starter-web

springboot自动配置包中`MailSenderAutoConfiguration`通过`@Import`注解向容器中导入了`MailSenderJndiConfiguration`,而`MailSenderJndiConfiguration`向容器中导入了`JavaMailSenderImpl`类，我们可以使用该类发送邮件

**配置文件**

```properties
spring.mail.username=邮箱用户名
spring.mail.password=邮箱密码或授权码
spring.mail.host=smtp.example.com 
```

**自动注入**

```java
@Autowired
private JavaMailSenderImpl javaMailSender;
```

#### 3.1 简单邮件发送

```java
SimpleMailMessage message = new SimpleMailMessage();
// 设置主题和内容
message.setSubject("今天开会");
message.setText("物质楼555开会，不要迟到");
// 设置发送方和接收方
message.setFrom("xxx@163.com");
message.setTo("xxx@qq.com");

javaMailSender.send(message);
```

#### 3.2 复杂邮件发送

带有附件或html页面的邮件

**两个设置**

`new MimeMessageHelper(message,true)`	设置multipart=true，开启对内联元素和附件的支持

`helper.setText("xxxx",true)`	html=ture，设置content type=text/html，默认为text/plain

```java
MimeMessage message = javaMailSender.createMimeMessage();
// multipart=true
// 开启对内联元素和附件的支持
MimeMessageHelper helper = new MimeMessageHelper(message,true);

helper.setSubject("今天开会");
// html=ture
// 设置content type=text/html，默认为text/plain
helper.setText("<b style='color:red'>物质楼555开会，不要迟到</b>",true);

helper.setFrom("hongshengmo@163.com");
helper.setTo("1043245239@qq.com");
/ /设置附件
helper.addAttachment("2.png",new File("D:\\Works\\Note\\images\\图片2.png"));
helper.addAttachment("3.png",new File("D:\\Works\\Note\\images\\图片3.png"));
javaMailSender.send(message);
```

## Spring boot与安全

### 1. 安全

应用程序的两个主要区域是“认证”和“授权”（或者访问控制），这两个主要区域是Spring Security的两个目标。身份验证意味着**确认您自己的身份**，而授权意味着**授予对系统的访问权限**

**认证**

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。系统确定您是否就是您所说的使用凭据。在公共和专用网络中，系统通过登录密码验证用户身份。身份验证通常通过用户名和密码完成，

**授权**

另一方面，授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。简单来说，授权决定了您访问系统的能力以及达到的程度。验证成功后，系统验证您的身份后，即可授权您访问系统资源。

### 2. Spring Security

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入`spring-boot-starter-security`模块，进行少量的配置，即可实现强大的安全管理。

**WebSecurityConfigurerAdapter：自定义Security策略**

通过在配置类中继承该类重写`configure(HttpSecurity http)`方法来实现自定义策略

**@EnableWebSecurity：开启WebSecurity模式**

在配置类上标注`@EnableWebSecurity`开启WebSecurity模式

### 3. Springboot整合security

#### 1. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

导入spring security的包之后，默认情况所有应用访问认证授权，默认用户名user，密码为随机生成的uuid，启动时打印在控制台

#### 2. 登录/注销

```java
@EnableWebSecurity  // 开启WebSecurity模式
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 目录允许所有人访问，其他目录都需要对应角色
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("VIP1")
                .antMatchers("/level2/**").hasRole("VIP2")
                .antMatchers("/level3/**").hasRole("VIP3");
        
        // 开启自动配置的登陆功能，效果，如果没有登陆，没有权限就会来到登陆页面
        //	/login来到登陆页
        //	重定向到/login?error表示登陆失败
        http.formLogin();
        
        // 开启自动配置的注销功能
        // 向/logout发送post请求表示注销
        // http.logout();
        // 注销成功后回到首页
        http.logout().logoutSuccessUrl("/");
    }
    
    // 定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        // super.configure(auth);
        auth.inMemoryAuthentication()
            .withUser("zhangsan").password("123456").roles("VIP1", "VIP2")
            .and()
            .withUser("lisi").password("123456").roles("VIP2", "VIP3")
            .and()
            .withUser("wangwu").password("123456").roles("VIP1", "VIP3");
    }
}
```

此时除了主页，点击其他的页面都会自动跳转到security自动生成的登录页面，`/login`来到登陆页，重定向到`/login?error`表示登陆失败；

` http.logout()`开启自动配置的注销功能,向`/logout`发送post请求表示注销，在欢迎页加上注销表单，默认注销后自动跳转到登录页面，若想改变转发路径，可以通过`http.logout().logoutSuccessUrl(url)`设置路径

```html
<form th:action="@{/logout}" method="post">
   <input type="submit" value="注销">
</form>
```

#### 3. 定义认证规则

为了保证密码能安全存储，springboot内置`PasswordEncoder`对密码进行转码，默认密码编码器为`DelegatingPasswordEncoder`。在定义认证规则时，我们需要使用`PasswordEncoder`将密码转码，由于`withDefaultPasswordEncoder()`并非安全已被弃用，因此仅在测试中使用。

```java
@Bean
public UserDetailsService users() {
    //使用默认的PasswordEncoder
    User.UserBuilder builder = User.withDefaultPasswordEncoder();
    //定义账户用户名、密码、权限
    UserDetails user1 = builder.username("zhangsan")
        .password("123456")
        .roles("VIP1", "VIP2")
        .build();
    UserDetails user2 = builder.username("lisi")
        .password("123456")
        .roles("VIP3", "VIP2")
        .build();
    UserDetails user3 = builder.username("wangwu")
        .password("123456")
        .roles("VIP1", "VIP3")
        .build();
    // 使用内存保存用户信息
    return new InMemoryUserDetailsManager(user1,user2,user3);
}
```

#### 4.自定义欢迎页

**导入依赖**

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

**引入命名空间**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
     xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

**根据是否登录显示游客或用户信息**

```html
<!-- 未登录显示此div -->
<div sec:authorize="!isAuthenticated()">
    <h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
</div>
<!-- 登录显示此div -->
<div sec:authorize="isAuthenticated()">
    <!-- 显示用户名 -->
    <h2>尊敬的<span th:text="${#authentication.name}"></span>,您好！您的角色有：
        <!-- 显示用户角色 -->
        <span th:text="${#authentication.authorities}"></span></h2>
    <form th:action="@{/logout}" method="post">
        <input type="submit" value="注销">
    </form>
</div>
```

**根据角色类型显示信息**

```html
<!-- 具有VIP1的角色显示以下div -->
<div sec:authorize="hasRole('VIP1')">
    <h3>普通武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level1/1}">罗汉拳</a></li>
        <li><a th:href="@{/level1/2}">武当长拳</a></li>
        <li><a th:href="@{/level1/3}">全真剑法</a></li>
    </ul>
</div>
```

#### 5. 自定义登录页/记住我

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // ...
    // 来到自己定制登录页
    http.formLogin()
        .usernameParameter("user")  // 表单用户名name
        .passwordParameter("pwd")   // 表单密码name
        .loginPage("/userlogin");   // 定制登陆页路径
    // ...

    // 开启记住我
    // 登录成功后，将cookie发送给浏览器，以后访问带上这个cookie，只要通过检查就可以免登陆
    // 点击注销会删除这个cookie
    // http.rememberMe();
    http.rememberMe()
        .rememberMeParameter("rem");  // 设置表单记住我name值
}
```

通过`loginPage(url)`设置登录页路径后，在定制的登录页发送post请求url即为登录请求，并设置表单的`name`属性都为对应值；

通过勾选`记住我`，session退出后依然能通过`cookie`保存用户信息，下次免登陆

```html
<form th:action="@{/userlogin}" method="post">
   用户名:<input name="user"/><br>
   密码:<input name="pwd"><br/>
   <input type="checkbox" name="rem">记住我<br>
   <input type="submit" value="登陆">
</form>
```

## Spring Boot与监控管理

通过引入spring-boot-starter-actuator，可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息等

### 1. Actuator监控管理

**导入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

浏览器打开链接`http://localhost:8080/actuator/`,可以看到所有支持的连接，响应如下,默认只支持这些端点

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/actuator",
            "templated": false
        },
        "health": {
            "href": "http://localhost:8080/actuator/health",
            "templated": false
        },
        "health-path": {
            "href": "http://localhost:8080/actuator/health/{*path}",
            "templated": true
        },
        "info": {
            "href": "http://localhost:8080/actuator/info",
            "templated": false
        }
    }
}
```

如果要看到所有支持的状态查询，需要配置

```properties
management.endpoints.web.exposure.include=*
```

bean加载情况`http://localhost:8080/actuator/beans`,显示了容器中各类各项属性

```json
{
    contexts: {
        application: {
            beans: {
                endpointCachingOperationInvokerAdvisor: {
                    aliases: [ ],
                    scope: "singleton",
                    type: "org.springframework.boot.actuate.endpoint.invoker.cache.CachingOperationInvokerAdvisor",
                    resource: "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/EndpointAutoConfiguration.class]",
                    dependencies: [
                        "environment"
                    ]
                },
            }
        }
    }
}
```

### 2. 监控和管理端点

| 端点名      | 描述                        |
| ----------- | --------------------------- |
| autoconfig  | 所有自动配置信息            |
| auditevents | 审计事件                    |
| beans       | 所有Bean的信息              |
| configprops | 所有配置属性                |
| dump        | 线程状态信息                |
| env         | 当前环境信息                |
| health      | 应用健康状况                |
| info        | 当前应用信息                |
| metrics     | 应用的各项指标              |
| mappings    | 应用@RequestMapping映射路径 |
| shutdown    | 关闭当前应用（默认关闭）    |
| trace       | 追踪信息（最新的http请求）  |

### 3. 端点配置

默认情况下，除shutdown以外的所有端点均已启用。要配置单个端点的启用，请使用`management.endpoint.<id>.enabled`属性。以下示例启用`shutdown`端点：

```properties
management.endpoint.shutdown.enabled=true
```

另外可以通过`management.endpoints.enabled-by-default`来修改全局端口默认配置,以下示例启用info端点并禁用所有其他端点：

```properties
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

修改路径

```properties
# 修改根目录路径
management.endpoints.web.base-path=/management
# 修改/health路径
management.endpoints.web.path-mapping.health=healthcheck
```
