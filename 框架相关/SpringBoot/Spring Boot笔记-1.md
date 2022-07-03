# SpringBoot：学习笔记-1

> 虽然说之前已经大致过了 SpringBoot，但也仅限于了解层面，这里打算稍微再系统些的学习一下，于是找了 bilibili 上播放量较高的雷丰阳老师的课程进行学习，课程连接：[SpringBoot](https://www.bilibili.com/video/BV1Et411Y7tQ?from=search&seid=11084747628285566436)，在学习过程中顺带做个学习笔记。
>
> SpringBoot 学习笔记总共分为5篇，此为 1/5 篇。

## 1. Spring Boot 入门

### 1.1 Spring Boot简介

**Spring Boot 简介：**

简化Spring应用开发的一个框架；

整个Spring技术栈的一个大整合；

J2EE开发的一站式解决方案；

**微服务概述：**

2014，martin fowler 提出了微服务：架构风格（服务微化）

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

单体应用：ALL IN ONE

微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

[详细参照微服务文档](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa)

**环境准备：（环境约束）**

* `–jdk1.8`：Spring Boot 推荐 `jdk1.7` 及以上；`java version "1.8.0_112"`

* `–maven3.x`：maven 3.3以上版本；`Apache Maven 3.3.9`

* `–IntelliJIDEA2017`：`IntelliJ IDEA 2017.2.2 x64`、`STS`

* `–SpringBoot 1.5.9.RELEASE`：1.5.9；

**Maven设置**

给 maven 的 settings.xml 配置文件的 profiles 标签添加

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

**IDEA设置，即整合maven进来**

### 1.2 HelloWorld程序

一个功能：浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；

**创建一个maven工程**

**导入SpringBoot相关的依赖**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**编写一个主程序**

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {
        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class, args);
    }
}
```

**编写相关的 Controller&Service**

```java
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

**运行主程序测试**：运行主函数main即可启动SpringBoot

**简化部署**

```xml
<!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

将这个应用打成jar包，直接使用 `java -jar` 的命令进行执行，步骤：

1. 进入jar包的目录，打开控制台；

2. 通过运行命令`java -jar XXX.jar`即可部署

### 1.3 Hello World探究

#### 1.3.1 POM文件

**父项目**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
```

他的父项目是：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

他来真正管理Spring Boot应用里面的所有依赖版本；

```xml
<!-- 在spring-boot-dependencies对jar包进行了版本管理 -->
<properties>
    <activemq.version>5.15.13</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.82</appengine-sdk.version>
    <artemis.version>2.12.0</artemis.version>
    ...
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-amqp</artifactId>
            <version>${activemq.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-blueprint</artifactId>
            <version>${activemq.version}</version>
        </dependency>
    </dependencies>
    ...
</dependencyManagement>
```

这是Spring Boot的版本仲裁中心；

以后我们导入依赖默认是不需要写版本；

> 当然没有在dependencies里面管理的依赖自然需要声明版本号

**启动器**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter-xxx**：spring-boot场景启动器；

**spring-boot-starter-web**：帮我们导入了web模块正常运行所依赖的组件；

Spring Boot 将所有的功能场景都抽取出来，做成一个个的 starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。

#### 1.3.2 主程序类/主入口类

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {
        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class, args);
    }
}
```

##### 1.3.2.1 @SpringBootApplication

**@SpringBootApplication**：Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

```java
// SpringBootApplication注解源码：
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    //..
}
```

##### 1.3.2.2 @SpringBootConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

**@SpringBootConfiguration**：Spring Boot的配置类； 标注在某个类上，表示这是一个Spring Boot的配置类；

* **@Configuration**：说明这是一个配置类 ，配置类就是对应Spring的xml配置文件
  * **@Component**：说明这是Spring容器中的一个组件

##### 1.3.2.3 @EnableAutoConfiguration

**@EnableAutoConfiguration**：开启自动配置功能；

以前我们需要配置的东西，Spring Boot帮我们自动配置；**@EnableAutoConfiguration**告诉SpringBoot开启自动配置功能；这样自动配置才能生效；@EnableAutoConfiguration上的注解如下：

```java
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    //..
}
```

1. **注解@AutoConfigurationPackage说明：**

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```

**@AutoConfigurationPackage**：自动配置包

* **@Import(AutoConfigurationPackages.Registrar.class)：**
* Spring的底层注解**@Import**，给容器中导入一个组件；导入的组件为`AutoConfigurationPackages.Registrar.class`；

> 进入`AutoConfigurationPackages.Registrar.class`
>
> ```java
> public abstract class AutoConfigurationPackages {
>        // ...
>        @Order(Ordered.HIGHEST_PRECEDENCE)
>        static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
> 
>            @Override
>            public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
>                // 注册一些信息，给容器中导入组件
>                register(registry, new PackageImport(metadata).getPackageName());
>            }
>    
>            // ...
>        }
>        // ...
>    }
> ```
> 
>对上述代码进行Debug查看：
> 
>`new PackageImport(metadata).getPackageName()`
> 
>* metadata：注解的元信息，如下
> 
>  ![metadata](Spring Boot笔记-1/metadata.PNG)
> 
>* 通过对`new PackageImport(metadata).getPackageName()`右键计算，得出如下：
> 
>  ![对包名进行计算](Spring Boot笔记-1/对包名进行计算.PNG)

`@EnableAutoConfiguration注解的注解@AutoConfigurationPackage`**作用总结：将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；**

2. **注解@Import(EnableAutoConfigurationImportSelector.class)说明：**

分析完@EnableAutoConfiguration上第一个注解，继续对第二个注解进行分析：

**@Import(EnableAutoConfigurationImportSelector.class)；**

其中**EnableAutoConfigurationImportSelector**：导入哪些组件的选择器；

进入源码进行分析：

```java
// 1. 进入EnableAutoConfigurationImportSelector类中，其继承自AutoConfigurationImportSelector
public class EnableAutoConfigurationImportSelector
		extends AutoConfigurationImportSelector {
	// ...
}

// 2. 进入AutoConfigurationImportSelector
public class AutoConfigurationImportSelector
    implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
	
    // 这个方法将所有需要导入的组件以全类名的方式返回(String数组)；这些组件就会被添加到容器中；
    @Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		try { 
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
			AnnotationAttributes attributes = getAttributes(annotationMetadata);
            // 3. 从这里获取了相应的候选配置文件信息(关键方法)
			List<String> configurations = getCandidateConfigurations(
                annotationMetadata, attributes);
			configurations = removeDuplicates(configurations);
			configurations = sort(configurations, autoConfigurationMetadata);
			Set<String> exclusions = getExclusions(annotationMetadata, attributes);
			checkExcludedClasses(configurations, exclusions);
			configurations.removeAll(exclusions);
			configurations = filter(configurations, autoConfigurationMetadata);
			fireAutoConfigurationImportEvents(configurations, exclusions);
			return configurations.toArray(new String[configurations.size()]);
		}
		catch (IOException ex) {
			throw new IllegalStateException(ex);
		}
	}

    // 3. 从这里获取了相应的候选配置类信息
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
                                                      AnnotationAttributes attributes) {
        // 4. 主要是SpringFactoriesLoader.loadFactoryNames方法(关键方法)
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
            getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                        "No auto configuration classes found in META-INF/spring.factories. If you " + 
                        "are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
}

// 5. 查看SpringFactoriesLoader.loadFactoryNames方法
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    try {
        // 其中：FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
        Enumeration<URL> urls = (
            classLoader != null ?
            classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
            ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        List<String> result = new ArrayList<String>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            result.addAll(
                Arrays.asList(
                    StringUtils.commaDelimitedListToStringArray(factoryClassNames)
                ));
        }
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
                                           "] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}

// 5.1 SpringFactoriesLoader.loadFactoryNames参数1：
// value:interface org.springframework.boot.autoconfigure.EnableAutoConfiguration
// 在META-INF/spring.factories文件下就有这个：
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}

// 5.2 SpringFactoriesLoader.loadFactoryNames参数2：
protected ClassLoader getBeanClassLoader() {
    return this.beanClassLoader;
}
```

对上述代码打上断点，如下：

![configurations导入自动配置类](Spring Boot笔记-1/configurations导入自动配置类.PNG)

可见该注解会给容器中导入非常多的自动配置类（**xxxAutoConfiguration**）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

`SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader);`

**总结：Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都帮我们；**

J2EE的整体整合解决方案和自动配置都在 `spring-boot-autoconfigure-x.x.x.RELEASE.jar`；

### 1.4 Spring Initializer快速创建Spring Boot项目

**IDEA：使用 Spring Initializer快速创建项目**

IDEA都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

选择我们需要的模块；向导会联网创建Spring Boot项目；

默认生成的Spring Boot项目；

- 主程序已经生成好了，我们只需要我们自己的逻辑
- **resources文件夹中目录结构**
  
  - static：保存所有的静态资源，如： js/css/images；
  
  - templates：保存所有的模板页面；
  
    > **Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面；**
    >
    > 可以使用模板引擎（freemarker、thymeleaf）；
  
  - application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；

**STS使用 Spring Starter Project快速创建项目，过程略**

## 2. 配置文件

### 2.1 配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

* application.properties

* application.yml

配置文件的作用：**修改SpringBoot自动配置的默认值**；SpringBoot在底层都给我们自动配置好；

YAML（YAML Ain't Markup Language）

* YAML  A Markup Language：是一个标记语言

* YAML   isn't Markup Language：不是一个标记语言；

标记语言：

* 以前的配置文件；大多都使用的是**xxxx.xml**文件；

* YAML：**以数据为中心**，比json、xml等更适合做配置文件；

* YAML：配置例子

  ```yml
  server:
    port: 8081
  ```

* XML：

  ```xml
  <server>
  	<port>8081</port>
  </server>
  ```

### 2.2 YAML语法

#### 2.2.1 基本语法

`k: v`：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
    port: 8081
    path: /hello
```

属性和值也是大小写敏感。

#### 2.2.2 值的写法

**字面量：普通的值（数字，字符串，布尔）**

`k: v`：字面直接来写；

* 字符串默认不用加上单引号或者双引号；

* `""`：双引号：会转义特殊字符，特殊字符会作为本身想表示的意思

  例如：`name:   "zhangsan \n lisi"`：输出；zhangsan 换行  lisi

* `''`：单引号：不会转义字符串里面的特殊字符，特殊字符最终只是一个普通的字符串数据

  例如：`name:   ‘zhangsan \n lisi’`：输出；zhangsan \n  lisi

**对象、Map（属性和值）（键值对）：**

`k: v`：在下一行来写对象的属性和值的关系；注意缩进

对象还是`k: v`的方式

```yaml
friends:
	lastName: zhangsan
	age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan, age: 18}
```

**数组（List、Set）：**

用 `-` 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat, dog, pig]
```

### 2.3 配置文件值注入

#### 2.3.1 yaml配置文件注入

**配置文件**：`src/main/resources/appliaction.yml`

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

**javaBean**：`src/main/java/cn/xyc/springboot/bean/Person.class`

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 * 但是只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能，
 * 因此需要通过@Component将类添加到容器中
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
	
    // Getter and Setting and toString
}
```

> 添加了@ConfigurationProperties注解后，会有如下提示：
>
> Spring Boot Configuration Annotation Processor not found in class path
>
> 解决如下：
>
> ![ConfigurationProperties问题](Spring Boot笔记-1/ConfigurationProperties问题.PNG)
>
> 我们可以导入配置文件处理器，以后编写配置就有提示了
>
> ```xml
> <!--导入配置文件处理器，配置文件进行绑定就会有提示-->
> <dependency>
>        <groupId>org.springframework.boot</groupId>
>        <artifactId>spring-boot-configuration-processor</artifactId>
>        <optional>true</optional>
> </dependency>
> ```
>
> 添加后需要重启一下springboot应用

**测试：**`src\test\java\cn\xyc\springboot`

```java
/**
 * Spring Boot的单元测试
 * 可以在测试期间很方便的类似编码一样进行自动注入等容器的功能
 */
@SpringBootTest
class Springboot01ApplicationTests {

    // 自动注入
	@Autowired
	Person person;

	@Test
	void contextLoads() {
		System.out.println(person);
	}
}
```

**控制台输出：**

```java
Person{lastName='hello', age=18, boss=false, birth=Tue Dec 12 00:00:00 CST 2017, maps={k1=v1, k2=12}, lists=[lisi, zhaoliu], dog=Dog{name='小狗', age=12}}
```

#### 2.3.2 properties配置文件注入

**配置文件**：`src/main/resources/appliaction.properties`

```properties
# 配置person的值
person.age=18
person.birth=2017/12/15
person.boss=false
person.last-name=张三
person.maps.k1=v1
person.maps.k2=v2
person.lists=a,b,c
person.dog.name=小狗
person.dog.age=15
```

**javaBean**与**测试**类同上，输出：

```java
Person{lastName='ï¿½ï¿½ï¿½ï¿½', age=18, boss=false, birth=Fri Dec 15 00:00:00 CST 2017, maps={k1=v1, k2=v2}, lists=[a, b, c], dog=Dog{name='Ð¡ï¿½ï¿½', age=15}}
```

**问题：properties配置文件在idea中默认utf-8可能会乱码**

解决如下：

![idea配置乱码](Spring Boot笔记-1/idea配置乱码.png)

修改之后输出正常了

#### 2.3.3 通过@Value完成注入

配置文件不变，**JavaBean**修改如下：

```java
@Component
// @ConfigurationProperties(prefix = "person")
public class Person {

    /**
     * 之前spring配bean的方式：
     * <bean class="Person">
     *     <property name="lastName" value="字面量/${key} key是从环境变量、配置文件中获取值/#{SpEL}"></property>
     * </bean>
     */
    @Value("${person.last-name}")
    private String lastName;
    @Value("#{10+20}")  // 通过SpEL表达式
    private Integer age;
    @Value("false")
    private Boolean boss;
    @Value("2020/10/31")
    private Date birth;
}
```

测试类不变，输出结果：

```java
Person{lastName='张三', age=30, boss=false, birth=Sat Oct 31 00:00:00 CST 2020, maps=null, lists=null, dog=null}
```

#### 2.3.3 @Value和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties                 | @Value                    |
| -------------------- | ---------------------------------------- | ------------------------- |
| 功能                 | 批量注入配置文件中的属性，指定prefix即可 | 需要一个个指定            |
| 松散绑定（松散语法） | 支持                                     | 不支持                    |
| SpEL，即#{}          | 不支持                                   | 支持                      |
| JSR303数据校验       | 支持                                     | 不支持                    |
| 复杂类型封装         | 支持                                     | 不支持，比如Map/List/对象 |

**功能说明：**配置文件yml还是properties他们都能获取到值；

* 如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

* 如果说，我们专门编写了一个JavaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

**松散绑定（Relaxed Binding）说明：**

在配置文件中把javabean中的fristName数据的注入key修改为

- person.firstName
- person.frist-name
- person.first_name

都等价，都可以完成属性值的注入，这就是松散绑定

**JSR303数据校验**，配置文件注入值数据校验：

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated  // JSR303数据校验所需注解
public class Person {
    
    @Email  // 添加该注解后，lastName必须是邮箱格式
    private String lastName;
    private Integer age;
    private Boolean boss;

    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

#### 2.3.4 @PropertySource&@ImportResource&@Bean

##### 2.3.4.1 @PropertySource

**@PropertySource**：加载指定的配置文件；

在resources目录下创建一个新的 `person.properties` 文件

```properties
# person.properties文件
# 配置person的值
person.properties.age=18
person.properties.birth=2017/12/15
person.properties.boss=false
person.properties.last-name=王五
person.properties.maps.k1=v1
person.properties.maps.k2=v2
person.properties.lists=a,b,c
person.properties.dog.name=小狗
person.properties.dog.age=15
```

在JavaBean中修改如下：

```java
/**
 *  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值；
 *  @PropertySource 加载指定的配置文件
 */
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    // @Value("#{11*2}")
    private Integer age;
    // @Value("true")
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

##### 2.3.4.2 @ImportResource

**@ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来，需要用到@ImportResource注解，流程如下：

1. 创建在resources文件下spring的配置文件beans.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 配置一个bean -->
       <bean id="helloService" class="cn.xyc.springboot01.service.HelloService"></bean>
   </beans>
   ```

   > 已经不推荐了，见下方说明

2. 创建相应的类`cn.xyc.springboot01.service.HelloService`，需要将这个类交给spring进行管理；

   > 进行测试
   >
   > ```java
   > @SpringBootTest
   > class Springboot01ApplicationTests {
   > 
   > 	@Autowired
   > 	ApplicationContext ioc;
   > 
   > 	@Test
   > 	void testHelloService(){
   > 		boolean b = ioc.containsBean("helloService");
   > 		System.out.println(b);  // false 即helloService没有被spring进行管理
   > 	}
   > }
   > ```

3. 想要自己编写的配置文件生效，需要将**@ImportResource**标注在一个配置类上

   ```java
   @ImportResource(locations = {"classpath:beans.xml"})
   @SpringBootApplication
   public class Springboot01Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Springboot01Application.class, args);
   	}
   
   }
   ```

   此时测试发现，spring管理了这个bean：helloService

##### 2.3.4.3 @Bean

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类**@Configuration**，就是来替代之前的Spring配置文件

2、使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 * 在配置文件中用<bean><bean/>标签添加组件
 */
@Configuration
public class MyAppConfig {

    // 将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

### 2.4 配置文件占位符

#### 2.4.1 随机数

RandomValuePropertySource：配置文件中可以使用的随机数

```java
${random.value}
${random.int}
${random.long}
${random.int(10)}
${random.int[1024,65536]}
```

#### 2.4.2 属性配置占位符

可以在配置文件中引用前面配置过的属性

`${app.name:默认值}`来指定找不到属性时的默认值

```properties
# 修改application.properties文件为
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```

测试输出结果为：

```java
Person{lastName='张三df13ccbc-fa58-4d26-a0c7-45f06811347a', age=-897138853, boss=false, birth=Fri Dec 15 00:00:00 CST 2017, maps={k1=v1, k2=14}, lists=[a, b, c], dog=Dog{name='hello_dog', age=15}}
```

### 2.5 Profile

profiles是spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境

#### 2.5.1 多Profile文件

我们在主配置文件编写的时候，文件名可以是`application-{profile}.properties/yml`

例如可以定义多个properties文件：

```properties
# application.properties文件
server.port=8081

# application-dev.properties文件
server.port=8082

# application-prod.properties文件
server.port=80
```

默认启动时是使用application.properties的配置；

#### 2.5.2 yml支持多文档块方式

```yml
# 文档块1
server:
  port: 8081
---
# 文档块2
server:
  port: 8083
spring:
  profiles: dev
---
# 文档块3
server:
  port: 8084
spring:
  profiles: prod  # 指定属于哪个环境
```

#### 2.5.3 激活指定profile

1. 对于properties配置文件，在application.properties配置文件中指定`spring.profiles.active=xxx`，则激活对应的 `application-{xxx}.properties`

2. 对于 yml 配置文件，通过如下激活：

   ```yml
   spring:
     profiles:
       active: prod	# 激活prod环境
   ```

3. 通过命令行：`java -jar xxx.jar --spring.profiles.active=dev;`可以直接在测试的时候，配置传入命令行参数

4. 虚拟机参数`VM options`：`-Dspring.profiles.active=dev`

### 2.6 配置文件加载位置

springboot 启动会扫描以下位置的 `application.properties/application.yml` 文件作为Spring boot的默认配置文件

1. `–file:./config/ `
2. `–file:./`
3. `–classpath:/config/`
4. `–classpath:/`

> 注意：1,2两点我的springboot版本可能过高，导致无效！
>

优先级由高到底，**高优先级的配置会覆盖低优先级的配置**；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

> 互补配置说明：
>
> 1. 在`–classpath:/config/`路径下配置：`server.port=8081`
> 2. 在`–classpath:/`路径下配置：`server.servlet.context-path=/boot`
>
> 则上述两个配置属性就形成了互补配置，访问时：http://localhost:8081/boot/hello

还可以通过 `spring.config.location` 来改变默认的配置文件位置

**项目打包好以后，我们可以使用命令行参数的形式`--spring.config.location`，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

`java -jar XXX.jar --spring.config.location=./application.properties`

### 2.7 外部配置加载顺序

**SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

1. **命令行参数**

   所有的配置都可以在命令行上进行指定

   `java -jar xxx.jar --server.port=8087  --server.servlet.context-path=/boot`

   多个配置用空格分开：`--配置项=值 --配置项=值`

2. 来自 `java:comp/env` 的JNDI属性

3. Java 系统属性（`System.getProperties()`）

4. 操作系统环境变量

5. RandomValuePropertySource配置的random.*属性值

6. **jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

7. **jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

8. **jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

9. **jar包内部的application.properties或application.yml(不带spring.profile)配置文件**

> 对6-9点总结：
>
> 1. **由jar包外向jar包内进行寻找**
> 2. **优先加载带profile**
> 3. **再来加载不带profile**

10. @Configuration注解类上的@PropertySource

11. 通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源：[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)

### 2.8 自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties)

#### 2.8.1 自动配置原理

> 注意：这里我按照了spring-boot-2.3.5.RELEASE版本分析了一下，上面的是1.x.x的分析，但是内容大致相同

1. SpringBoot启动的时候加载主配置类**@SpringBootApplication**，开启了自动配置功能 **@EnableAutoConfiguration**

   > ```java
   > @SpringBootConfiguration  // 前面分析了
   > @EnableAutoConfiguration  // 前面也分析了，这里再分析一下
   > @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   > 		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
   > public @interface SpringBootApplication {
   >     // ...
   > }
   > ```

2. **@EnableAutoConfiguration 作用：**

   > ```java
   > @AutoConfigurationPackage  // 这个的分析之前已经分析过了
   > @Import(AutoConfigurationImportSelector.class)
   > public @interface EnableAutoConfiguration {
   >        // ...
   > }
   > ```

   1. 利用`@Import(AutoConfigurationImportSelector.class)`给容器中导入一些组件

   ```java
   // AutoConfigurationImportSelector类下的selectImports方法
   @Override
   public String[] selectImports(AnnotationMetadata annotationMetadata) {
       if (!isEnabled(annotationMetadata)) {
           return NO_IMPORTS;
       }
       // 关键加载
       AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
       return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
   }
   ```

   2. 可以查看`getAutoConfigurationEntry`方法的内容；

   ```java
   protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
       if (!isEnabled(annotationMetadata)) {
           return EMPTY_ENTRY;
       }
       AnnotationAttributes attributes = getAttributes(annotationMetadata);
       // 关键方法 获取候选的配置
       List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
       configurations = removeDuplicates(configurations);
       Set<String> exclusions = getExclusions(annotationMetadata, attributes);
       checkExcludedClasses(configurations, exclusions);
       configurations.removeAll(exclusions);
       configurations = filter(configurations, autoConfigurationMetadata);
       fireAutoConfigurationImportEvents(configurations, exclusions);
       return new AutoConfigurationEntry(configurations, exclusions);
   }
   ```

   3. `List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);`获取候选的配置

   ```java
   protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
       // 关键方法
       List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
           getSpringFactoriesLoaderFactoryClass(),
           getBeanClassLoader());
       Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                       + "are using a custom packaging, make sure that file is correct.");
       return configurations;
   }
   ```

   4. `SpringFactoriesLoader.loadFactoryNames()`

   ```java
   public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
       String factoryTypeName = factoryType.getName();
       return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
   }
   
   // loadSpringFactories函数
   private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
       MultiValueMap<String, String> result = cache.get(classLoader);
       if (result != null) {
           return result;
       }
   
       try {
           // 扫描所有jar包类路径下META-INF/spring.factories
           Enumeration<URL> urls = (
               classLoader != null ?
               classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
               ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
           result = new LinkedMultiValueMap<>();
           while (urls.hasMoreElements()) {
               URL url = urls.nextElement();
               UrlResource resource = new UrlResource(url);
               // 把扫描到的这些文件的内容包装成properties对象
               Properties properties = PropertiesLoaderUtils.loadProperties(resource);
               for (Map.Entry<?, ?> entry : properties.entrySet()) {
                   String factoryTypeName = ((String) entry.getKey()).trim();
                   for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                       result.add(factoryTypeName, factoryImplementationName.trim());
                   }
               }
           }
           cache.put(classLoader, result);
           return result;
       }
       catch (IOException ex) {
           throw new IllegalArgumentException("Unable to load factories from location [" +
                                              FACTORIES_RESOURCE_LOCATION + "]", ex);
       }
   }
   ```

   其中：`FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"`

   因此该函数的作用：扫描所有jar包类路径下的`META-INF/spring.factories`

   把扫描到的这些文件的内容包装成properties对象

   从properties中获取到`EnableAutoConfiguration.class`类（类名）对应的值，然后把他们添加在容器中

   **总结：将类路径下META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；**

   ```properties
   # 例如在导入的依赖jar包中：
   # org.springframework.boot.autoconfigure:2.2.5.RELEASE.jar下有：/MATE-INF/spring.fatories
   # spring.fatories 配置文件中有如下内容：
   
   # Auto Configure,格式：key = value，这个key就是xxx..EnableAutoConfiguration
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
   org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
   org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
   org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
   org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
   org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
   org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
   org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
   org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
   org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
   org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
   org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
   org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
   org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
   org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
   org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
   org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
   org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
   org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
   org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
   org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
   org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
   org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
   org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
   org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
   org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
   org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
   org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
   org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
   org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
   org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
   ```

   每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

3. 每一个自动配置类进行自动配置功能；

   以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；

   ```java
   // 表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
   @Configuration(proxyBeanMethods = false)  
   // 启动指定类的ConfigurationProperties功能；
   // 将配置文件中对应的值和HttpProperties绑定起来；
   // 并把HttpProperties加入到ioc容器中
   @EnableConfigurationProperties(HttpProperties.class)
   // Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效;
   // 判断当前应用是否是web应用，如果是，当前配置类生效
   @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
   // 判断当前项目有没有这个类CharacterEncodingFilter
   // 这是SpringMVC中进行乱码解决的过滤器；
   @ConditionalOnClass(CharacterEncodingFilter.class) 
   // 判断配置文件中是否存在某个配置 spring.http.encoding.enabled；
   // matchIfMissing = true：说明我们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的；
   @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
   public class HttpEncodingAutoConfiguration {
     
     	// 他已经和SpringBoot的配置文件映射了
     	private final HttpProperties.Encoding properties;
     
       // 只有一个有参构造器的情况下，参数的值就会从容器中拿
       // 因为在@EnableConfigurationProperties(HttpProperties.class)中已经加入到IOC容器中了
   	public HttpEncodingAutoConfiguration(HttpProperties properties) {
   		this.properties = properties.getEncoding();
   	}
     
   	// 给容器中添加一个组件，这个组件的某些值需要从properties中获取
       @Bean   
       // 判断容器没有这个组件
   	@ConditionalOnMissingBean
       public CharacterEncodingFilter characterEncodingFilter() {
           CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
           filter.setEncoding(this.properties.getCharset().name());
           filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
           filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
           return filter;
       }
   }
   ```

   根据当前不同的条件判断，决定这个配置类是否生效；

   一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的xxxxProperties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

4. 所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类

   ```java
   // 从配置文件中获取指定的值和bean的属性进行绑定
   @ConfigurationProperties(prefix = "spring.http.encoding")  
   public class HttpEncodingProperties {
       public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
       private Charset charset = DEFAULT_CHARSET;
       private Boolean force;
       private Boolean forceRequest;
       private Boolean forceResponse;
       // ... 
   }
   ```

   上面这些xxxxProperties中的属性都是可以在application.properties中指定的

   ```properties
   spring.http.encoding.enabled=true
   spring.http.encoding.force=true
   spring.http.encoding.charset=UTF-8
   spring.http.encoding.force-request=true
   spring.http.encoding.force-response=true
   ```

**总结一波：**

1. **SpringBoot启动会加载大量的自动配置类xxxAutoConfiguration**
2. **我们看我们需要的功能SpringBoot是否默认写好了自动配置类；**
3. **我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）**

4. **给容器中自动配置类添加组件的时候，会从xxxproperties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**

> xxxxAutoConfigurartion：自动配置类；给容器中添加组件；
>
> xxxxProperties:封装配置文件中相关属性；

#### 2.8.2 细节

##### 2.8.2.1 @Conditional派生注解

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean                               |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean                             |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效；**

> 例如AopAutoConfiguration
>
> ```java
> @Configuration
> @ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class })
> @ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
> public class AopAutoConfiguration {
> 	// ...
> }
> ```
>
> 在没有导入AOP相应的包时就不会生效
>
> ```java
> import org.aspectj.lang.annotation.Aspect;
> import org.aspectj.lang.reflect.Advice;
> ```

##### 2.8.2.2 判断自动配置类生效

我们怎么知道哪些自动配置类生效？

**我们可以通过启用debug=true属性（在配置文件中添加即可）；来让控制台打印自动配置报告**；

这样我们就可以很方便的知道哪些自动配置类生效；

```java
============================
CONDITIONS EVALUATION REPORT
============================


Positive matches:  // 自动配置类启用的
-----------------

   AopAutoConfiguration matched:
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)     
          
   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet' (OnClassCondition)
      - found 'session' scope (OnWebApplicationCondition)
   
   // ...
          
Negative matches:  // 没有启用，没有匹配成功的自动配置类
-----------------
   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
             
   // ...
```

