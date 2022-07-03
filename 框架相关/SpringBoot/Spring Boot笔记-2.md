# SpringBoot：学习笔记-2

> 虽然说之前已经大致过了 SpringBoot，但也仅限于了解层面，这里打算稍微再系统些的学习一下，于是找了 bilibili 上播放量较高的雷丰阳老师的课程进行学习，课程连接：[SpringBoot](https://www.bilibili.com/video/BV1Et411Y7tQ?from=search&seid=11084747628285566436)，在学习过程中顺带做个学习笔记。
>
> SpringBoot 学习笔记总共分为5篇，此为 2/5 篇。

## 3. 日志

### 3.1 日志框架

#### 3.1.1 小故事

 小张，开发一个大型系统：

1. 起初通过`System.out.println("...")`，将关键数据打印在控制台；过滤信息功能？写在一个文件功能？

2. 框架来记录系统的一些运行时信息，提出日志框架：`zhanglogging.jar`；

3. 扩展高大上的几个功能，异步模式？自动归档？...？提出新框架：`zhanglogging-good.jar`；

4. 将以前框架卸下来再换上新的框架，重新修改之前相关的API，但后来又提出`zhanglogging-prefect.jar`，还要这样操作一遍；

5. 想到了结合JDBC与数据库驱动的关系；
   1. 写了一个统一的接口层，日志门面（日志的一个抽象层），`logging-abstract.jar`；
   2. 给项目中导入具体的日志实现就行了；
   3. 我们之前的日志框架都是实现的抽象层；

#### 3.1.2 市面上的日志框架

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                                   | 日志实现                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ~~JCL（Jakarta  Commons Logging）~~<br />**SLF4J（Simple  Logging Facade for Java）**<br />~~jboss-logging~~ | Log4j  <br />JUL（java.util.logging）<br />Log4j2<br />**Logback** |

左边选一个门面（抽象层）、右边来选一个实现；

选择日志门面：SLF4J；

选择日志实现：Logback；

SpringBoot：底层是Spring框架，Spring框架默认是用JCL；

**而SpringBoot选用SLF4j和logback；**

### 3.2 SLF4J使用

#### 3.2.1 如何在系统中使用SLF4J

[SLF4J官方文档](https://www.slf4j.org)

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是**调用日志抽象层里面的方法**；

给系统里面导入slf4J的jar包和logback的实现jar包

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(HelloWorld.class);
        logger.info("Hello World");
    }
}
```

SLF4J 配合日志实现的使用图示：

![concrete-bindings](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252312634.png)

每一个日志的实现框架都有自己的配置文件，在使用 slf4j 以后，**配置文件还是做成日志实现框架自己本身的配置文件。**

#### 3.2.2 遗留问题

有A系统：

* 使用日志框架：slf4j+logback

* 使用框架：Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、...

**统一日志记录**，即使是别的框架和我一起统一使用slf4j进行输出，如下：

![legacy](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313054.png)

**如何让系统中所有的日志都统一到slf4j；**

1. 将系统中其他日志框架先排除出去；

2. 中间包来替换原有的日志框架：xxx-over-slf4j.jar

3. 我们导入slf4j其他的实现。

### 3.3 SpringBoot日志关系

底层依赖关系展示：

> 在pom.xml文件中右键选择：Diagrams-show Dependences

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

SpringBoot使用它来做日志功能；

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

![springboot的日志](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313073.png)

总结：

1. SpringBoot底层也是使用**slf4j+logback**的方式进行日志记录；

2. SpringBoot也把其他的日志都替换成了slf4j：xxx-over-slf4j.jar；

3. 中间替换包如何实现？如下

   1. 在External Libraries中找到相应的xxx-over-slf4j.jar包；

      ![日志记录替换包](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313540.PNG)

   2. 在代码中进行了替换：

      ```java
      public abstract class LogFactory {
      
          static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";
      	
          // 可以看到这里进行了替换 new 了 SLF4J 的记录器
          static LogFactory logFactory = new SLF4JLogFactory();
      }
      ```

   3. 其他替换操作逻辑也是类似的

4. 如果我们要引入其他框架，则一定要把这个框架的默认日志依赖移除掉

   例如：Spring框架用的是commons-logging，而springboot中则排除了这个依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-core</artifactId>
       <exclusions>
           <exclusion>
               <groupId>commons-logging</groupId>
               <artifactId>commons-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

**总结：SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。**

### 3.4 日志使用

#### 3.4.1 默认配置

SpringBoot默认帮我们配置好了日志，在程序运行时控制台中已经有日志记录输出了。

使用日志记录进行输出：

```java
package cn.xyc.springboot05;

import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class Springboot05ApplicationTests {

	// 记录器
	Logger logger = LoggerFactory.getLogger(getClass());
	@Test
	public void contextLoads() {
		// 日志的级别；
		// 由低到高   trace<debug<info<warn<error
		// 可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
		logger.trace("这是trace日志...");
		logger.debug("这是debug日志...");
		// SpringBoot默认给我们使用的是info级别的;
        // 没有指定级别的就用SpringBoot默认规定的级别,即root级别
		logger.info("这是info日志...");
		logger.warn("这是warn日志...");
		logger.error("这是error日志...");
	}
}
```

**日志文件输出格式说明：**

```
日志输出格式：
	%d表示日期时间，
	%thread表示线程名，
	%-5level：级别从左显示5个字符宽度
	%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
	%msg：日志消息，
	%n是换行符

%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

**SpringBoot修改日志的默认配置**

```properties
logging.level.cn.xyc=trace

# 1. 不指定路径在当前项目下生成springboot.log日志
# logging.file=springboot.log
# 2. 可以指定完整的路径
# logging.file=D:/springboot.log

# logging.path使用
# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用spring.log作为默认文件
logging.path=/spring/log

# 在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

logging.file和logging.path的说明

| logging.file | logging.path | Example  | Description                          |
| ------------ | ------------ | -------- | ------------------------------------ |
| none         | none         | none     | 只在控制台输出                       |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件                 |
| (none)       | 指定目录     | /var/log | 输出到指定目录的默认spring.log文件中 |

#### 3.4.2 指定配置

给类路径下放上每个日志框架自己的配置文件即可；

SpringBoot就不使用他默认配置的了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml与logback-spring.xml区别：

* logback.xml：直接就被日志框架识别了；

* **logback-spring.xml**：日志框架就不直接加载日志的配置项，而是由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

  ```xml
  <springProfile name="staging">
      <!-- configuration to be enabled when the "staging" profile is active -->
    	<!-- 可以指定某段配置只在某个环境下生效 -->
  </springProfile>
  ```

  如：

  ```xml
  <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
      <!--
          日志输出格式：
     			%d表示日期时间，
     			%thread表示线程名，
     			%-5level：级别从左显示5个字符宽度
     			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
     			%msg：日志消息，
     			%n是换行符
      -->
      <layout class="ch.qos.logback.classic.PatternLayout">
          <springProfile name="dev">   <!-- 开发环境的日志输出 -->
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
          </springProfile>
          <springProfile name="!dev">  <!-- 不是开发环境的日志输出 -->
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
          </springProfile>
      </layout>
  </appender>
  ```

  如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误： `no applicable action for [springProfile]`

### 3.5 切换日志框架

若不想用LogBack的日志实现，想更换成Log4J的实现；

可以按照slf4j的日志适配图（3.2.2 遗留问题中的日志适配图），进行相关的切换；

slf4j+log4j的方式：在pom.xml文件中修改如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>  
        <exclusion>  <!-- 排除logback的实现 -->
            <artifactId>logback-classic</artifactId>
            <groupId>ch.qos.logback</groupId>
        </exclusion>
        <exclusion>  <!-- 排除 -->
            <artifactId>log4j-over-slf4j</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>  <!-- 导入Log4J的实现 -->
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

切换为log4j2

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>  <!-- 在原先的依赖中排除spring-boot-starter-logging -->
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

## 4. Web开发

### 4.1 简介

使用SpringBoot；

1. **创建SpringBoot应用，选中我们需要的模块；**
2. **SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来**
3. **自己编写业务代码；**

**自动配置原理**：见之前的分析

这个场景SpringBoot帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？...？

* `xxxxAutoConfiguration`：帮我们给容器中自动配置组件；
* `xxxxProperties`:配置类来封装配置文件的内容；

### 4.2 SpringBoot对静态资源的映射规则

首先，SpringMVC的自动配置类为：`WebMvcAutoConfiguration`

#### 4.2.1 ResourceProperties

在WebMvcAutoConfiguration下有内部类：

```java
// Defined as a nested config to ensure WebMvcConfigurerAdapter is not read when not
// on the classpath
@Configuration
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
    private final ResourceProperties resourceProperties;
}

// @EnableConfigurationProperties中加载的配置类：ResourceProperties.class
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {
    
    // 资源配置类，可以设置和静态资源有关的参数，缓存时间等
    private static final String[] SERVLET_RESOURCE_LOCATIONS = { "/" };

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

	private static final String[] RESOURCE_LOCATIONS;

	static {
		RESOURCE_LOCATIONS = new String[CLASSPATH_RESOURCE_LOCATIONS.length
				+ SERVLET_RESOURCE_LOCATIONS.length];
		System.arraycopy(SERVLET_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS, 0,
				SERVLET_RESOURCE_LOCATIONS.length);
		System.arraycopy(CLASSPATH_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS,
				SERVLET_RESOURCE_LOCATIONS.length, CLASSPATH_RESOURCE_LOCATIONS.length);
	}
    
    // ...
}
```

#### 4.2.2 映射规则-1

在`WebMvcAutoConfiguration`类的内部类`WebMvcAutoConfigurationAdapter`有如下代码：

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
	
    // ...
    Integer cachePeriod = this.resourceProperties.getCachePeriod();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(
            registry.addResourceHandler("/webjars/**")
            .addResourceLocations(
                "classpath:/META-INF/resources/webjars/")
            .setCachePeriod(cachePeriod));
    }
    // ...
}
```

分析上述代码：

1. 所有 `/webjars/**`，都去 `classpath:/META-INF/resources/webjars/` 找资源；

2. 而`webjars`：以jar包的方式引入静态资源；

3. 进入http://www.webjars.org/，选择相应的webjar包；

   ![webjar](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313907.PNG)

4. 在pom.xml文件中导入依赖

   ```xml
   <!--引入jquery的webjar-->
   <dependency>
       <groupId>org.webjars</groupId>
       <artifactId>jquery</artifactId>
       <version>3.5.1</version>
   </dependency>
   ```

5. 查看导入的依赖包结构，如下图，正好对于于上述的路径映射

   ![webjar包结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313415.PNG)

6. 对其进行访问：`localhost:8080/webjars/jquery/3.3.1/jquery.js`就可以放到`jquery.js`文件的内容了

#### 4.2.3 映射规则-2

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {

    // ...
    // 在webMvcProperties中定义了属性：staticPathPattern = "/**"
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(
            registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(
                this.resourceProperties.getStaticLocations())
            .setCachePeriod(cachePeriod));
    }
    // ...
}    

// 对this.resourceProperties.getStaticLocations()查看其定义：
// 在ResourceProperties.java中有如下定义：
// 当然这是可以直接配置的,通过spring.resources.CLASSPATH_RESOURCE_LOCATIONS进行配置覆盖
private static final String[] SERVLET_RESOURCE_LOCATIONS = { "/" };

private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
    "classpath:/META-INF/resources/", "classpath:/resources/",
    "classpath:/static/", "classpath:/public/" };

private static final String[] RESOURCE_LOCATIONS;

static {
    RESOURCE_LOCATIONS = new String[CLASSPATH_RESOURCE_LOCATIONS.length
                                    + SERVLET_RESOURCE_LOCATIONS.length];
    System.arraycopy(SERVLET_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS, 0,
                     SERVLET_RESOURCE_LOCATIONS.length);
    System.arraycopy(CLASSPATH_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS,
                     SERVLET_RESOURCE_LOCATIONS.length, CLASSPATH_RESOURCE_LOCATIONS.length);
}
```

分析上述代码：

1. `/**` 访问当前项目的任何资源，都去（静态资源的文件夹）找映射；

2. 静态资源文件路径有：

   ```java
   "classpath:/META-INF/resources/", 
   "classpath:/resources/",
   "classpath:/static/", 
   "classpath:/public/" 
   "/"：当前项目的根路径
   ```

   例如：`localhost:8080/abc` 去静态资源文件夹里面找abc

#### 4.2.4 其他资源映射

还在在`WebMvcAutoConfiguration`类的内部类`WebMvcAutoConfigurationAdapter`中：

**配置欢迎页映射**

```java
// 配置欢迎页映射
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
    ResourceProperties resourceProperties) {
    return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
                                         this.mvcProperties.getStaticPathPattern());
}

// 在new的WelcomePageHandlerMapping类下有：
// 
// if (welcomePage != null && "/**".equals(staticPathPattern)) {
//     // ..
//     controller.setViewName("forward:index.html");   // 即转发到index.html
// 	   // ..
// }
```

欢迎页：静态资源文件夹下的所有index.html页面；被`/**`映射；

例如：localhost:8080/   找index页面

**配置喜欢的图标**

```java
// 配置喜欢的图标
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
public static class FaviconConfiguration {

    private final ResourceProperties resourceProperties;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        //所有  **/favicon.ico 
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
                                                   faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler
            .setLocations(this.resourceProperties.getFaviconLocations());
        return requestHandler;
    }

}
```

所有的`**/favicon.ico`  都是在静态资源文件下找；

### 4.3 模板引擎

#### 4.3.1 概述

模版引擎有：JSP、Velocity、Freemarker、Thymeleaf

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313148.png)

SpringBoot推荐的Thymeleaf，语法更简单，功能更强大；

#### 4.3.2 引入Thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!-- 切换thymeleaf版本 -->
<properties>
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <!-- 布局功能的支持程序  thymeleaf3主程序 layout2 以上版本 -->
    <!-- thymeleaf2与layout1适配-->
    <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
```

#### 4.3.3 Thymeleaf使用

导入依赖后，有`ThymeleafAutoConfiguration`自动配置类

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration {

}
```

再看相应的配置文件`ThymeleafProperties.class`

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
  	
    // 上面代码可知：只要我们把HTML页面放在`classpath:/templates/xxx.html`
    // thymeleaf就能自动渲染；
}
```

**使用：**

1. 导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2. 使用thymeleaf语法；

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```

3. 控制器代码：

```java
@Controller
// @ResponseBody  这个注解千万不能加，因为这个注解使用流的形式进行返回数据
public class HelloController {
    // 查出一些数据，在页面展示
    @RequestMapping("/success")
    public String success(Map<String, Object> map){
        map.put("hello", "你好");
        return "success";
    }
}
```

#### 4.3.4 语法规则

1. **`th`标签的使用：**

   `th:text`；改变当前元素里面的文本内容；

   `th：任意html属性`；来替换原生属性的值

   ```html
   <div id="div01" class="myDiv" th:id="${hello}" th:class="${hello}" th:text="${hello}">这是显示欢迎信息</div>
   
   <!-- 当用静态页面打开时，显示原先的属性；	当渲染页面时，采用的是th后的属性-->
   ```

![th标签的使用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313488.png)

2. **表达式**

```properties
Simple expressions:（表达式语法）
	1. Variable Expressions: ${...}   获取变量值; 底层是OGNL表达式
		1) 获取对象的属性、调用方法
		2) 使用内置的基本对象：
			#ctx : the context object.
			#vars: the context variables.
		    #locale : the context locale.
		    #request : (only in Web Contexts) the HttpServletRequest object.
		    #response : (only in Web Contexts) the HttpServletResponse object.
		    #session : (only in Web Contexts) the HttpSession object.
		    #servletContext : (only in Web Contexts) the ServletContext object.
		   	使用方式：
		    ${session.foo}
		3) 内置的一些工具对象：
			#execInfo : information about the template being processed.
			#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
			#uris : methods for escaping parts of URLs/URIs
			#conversions : methods for executing the configured conversion service (if any).
			#dates : methods for java.util.Date objects: formatting, component extraction, etc.
			#calendars : analogous to #dates , but for java.util.Calendar objects.
			#numbers : methods for formatting numeric objects.
			#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
			#objects : methods for objects in general.
			#bools : methods for boolean evaluation.
			#arrays : methods for arrays.
			#lists : methods for lists.
			#sets : methods for sets.
			#maps : methods for maps.
			#aggregates : methods for creating aggregates on arrays or collections.
			#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    2. Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
        <div th:object="${session.user}">
            <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
            <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
            <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
        </div>
    
    3. Message Expressions: #{...}：获取国际化内容
    
    4. Link URL Expressions: @{...}：定义URL；
    	@{/order/process(execId=${execId},execType='FAST')}
    
    5. Fragment Expressions: ~{...}：片段引用表达式
    	<div th:insert="~{commons :: main}">...</div>
    	
	6. Literals（字面量）
    	Text literals: 'one text' , 'Another one!' ,…
    	Number literals: 0 , 34 , 3.0 , 12.3 ,…
    	Boolean literals: true , false
    	Null literal: null
    	Literal tokens: one , sometext , main ,…
    	
	7. Text operations:（文本操作）
	    String concatenation: +
	    Literal substitutions: |The name is ${name}|
	    
	8. Arithmetic operations:（数学运算）
	    Binary operators: + , - , * , / , %
	    Minus sign (unary operator): -
	
	9. Boolean operations:（布尔运算）
	    Binary operators: and , or
	    Boolean negation (unary operator): ! , not
	    
	10. Comparisons and equality:（比较运算）
	    Comparators: > , < , >= , <= ( gt , lt , ge , le )
	    Equality operators: == , != ( eq , ne )
	    
	11. Conditional operators:条件运算（三元运算符）
	    If-then: (if) ? (then)
	    If-then-else: (if) ? (then) : (else)
	    Default: (value) ?: (defaultvalue)
	    
	12. Special tokens:
	    No-Operation: _ 
```

3. **演示**

   success.html：

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
       <head>
           <meta charset="UTF-8">
           <title>Title</title>
       </head>
       <body>
           <h1>成功！</h1>
           <!--th:text 将div里面的文本内容设置为 -->
           <div id="div01" class="myDiv" th:id="${hello}" th:class="${hello}" th:text="${hello}">这是显示欢迎信息</div>
           <hr/>
           <div th:text="${hello}"></div>
           <div th:utext="${hello}"></div>
           <hr/>
           <!--th:each每次遍历都会生成当前这个标签：即会产生3个h4标签 -->
           <h4 th:text="${user}" th:each="user:${users}"></h4>
           <hr/>
           <h4>
               <span th:each="user:${users}">[[${user}]]</span>
           </h4>
       </body>
   </html>
   ```

   controller：

   ```java
   @Controller
   public class HelloController {
       // 查出一些数据，在页面展示
       @RequestMapping("/success")
       public String success(Map<String, Object> map){
           map.put("hello", "你好");
           map.put("users", Arrays.asList("zhangsan", "lisi", "wangwu"));
           return "success";
       }
   }
   ```
   

结果：

![thymeleaf模版语法小演示](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313011.PNG)

### 4.4 SpringMVC自动配置

[SpringMVC使用的官方教程：1.5.10.RELEASE](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications)

#### 4.4.1 SpringMVC自动配置功能

**Spring MVC auto-configuration**

> Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

Spring Boot 自动配置好了SpringMVC

> The auto-configuration adds the following features on top of Spring’s defaults:

以下是SpringBoot对SpringMVC的默认配置：

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发/重定向))

  - ContentNegotiatingViewResolver：组合所有的视图解析器的；

  - 如何定制：我们可以自己给容器中添加一个视图解析器，添加完后，ContentNegotiatingViewResolver就会自动的将其组合进来；

    > ```java
    > @SpringBootApplication
    > public class Springboot07Application {
    > 
    > 	public static void main(String[] args) {
    > 		SpringApplication.run(Springboot07Application.class, args);
    > 	}
    > 
    > 	@Bean  // 把自己定义的视图解析器加入Spring容器中
    > 	public ViewResolver myViewResolver(){
    > 		return new MyViewResolver();
    > 	}
    > 
    > 	private static class MyViewResolver implements ViewResolver{
    > 		@Override
    > 		public View resolveViewName(String s, Locale locale) throws Exception {
    > 			return null;
    > 		}
    > 	}
    > }
    > ```
    > 
    >找到DispatcherServlet类中的doDispatch方法，加上断点，找到视图解析器，如下
    > 
    >![ContentNegotiatingViewResolver](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313056.PNG)
    > 
    >可见刚刚自定义的视图解析器被加载进来了
  
- Support for serving static resources, including support for WebJars (see below).

  静态资源文件夹路径,webjars

- Static `index.html` support. 

  静态首页访问

- Custom `Favicon` support (see below).  

  favicon.ico

- 自动注册了 of `Converter`, `GenericConverter`, `Formatter` beans.

  - `Converter`：转换器；  
  
    类型转换使用Converter：`public String hello(User user)`：页面提交的数据转换为User对象中的数据
  
  - `Formatter`  格式化器；  2017.12.17 ===> Date；
  
    ```java
    @Bean
    // 在文件中配置日期格式化的规则
    // 如果配置了这个属性才会把这个DateFormatter注入到容器中
    @ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")
    public Formatter<Date> dateFormatter() {
        // 配置后会加入日期格式化组件
        return new DateFormatter(this.mvcProperties.getDateFormat());
    }
    ```
  
    自己添加的格式化器转换器，我们只需要放在容器中即可，通过方法`addFormatters`

- Support for `HttpMessageConverters` (see below).

  - `HttpMessageConverter`：消息转换器，这是SpringMVC用来转换Http请求和响应的；比如将User转化成Json数据写出；
- `HttpMessageConverters` 是从容器中确定；获取所有的`HttpMessageConverter`；
  
- 自己给容器中添加`HttpMessageConverter`，只需要将自己的组件注册容器中（@Bean,@Component）
  
- Automatic registration of `MessageCodesResolver` (see below).

  定义错误代码生成规则

- Automatic use of a `ConfigurableWebBindingInitializer` bean (see below).

  我们可以配置一个`ConfigurableWebBindingInitializer`来替换默认的；（添加到容器）

  ```
  初始化WebDataBinder，Web数据绑定器
  把请求数据绑定到JavaBean中
  ```

**总结：org.springframework.boot.autoconfigure.web：web的所有自动场景；**

#### 4.4.2 扩展SpringMVC

If you want to keep Spring Boot MVC features, and you just want to add additional [MVC configuration](https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/htmlsingle#mvc) (interceptors, formatters, view controllers etc.) you can add your own `@Configuration` class of type `WebMvcConfigurerAdapter`, but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter` or `ExceptionHandlerExceptionResolver` you can declare a `WebMvcRegistrationsAdapter` instance providing such components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

之前对SpringMVC的配置，创建springmvc.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context-3.2.xsd
      http://www.springframework.org/schema/mvc
      http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">

    <!-- 视图映射 -->
    <mvc:view-controller path="/hello" view-name="success"/>
    
    <!-- 拦截器的配置 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean></bean>
        </mvc:interceptor>
    </mvc:interceptors>
</beans>
```

编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc;

既保留了所有的自动配置，也能用我们扩展的配置；

> 这里spring2.0以上不再使用WebMvcConfigurerAdapter，而是直接实现WebMvcConfigurer
>
> 因为在WebMvcConfigurer里已经对方法有默认实现了，就不需要WebMvcConfigurerAdapter了

```java
// 使用WebMvcConfigurer可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override  // 添加视图控制器
    public void addViewControllers(ViewControllerRegistry registry) {
        // super.addViewControllers(registry);
        // 浏览器发送 /xyc 请求来到 success
        registry.addViewController("/xyc").setViewName("success");
    }
}
```

**原理：**

1. WebMvcAutoConfiguration是SpringMVC的自动配置类

2. 该类中有静态类：WebMvcAutoConfigurationAdapter

3. 该类在做其他自动配置时会导入；@Import(**EnableWebMvcConfiguration.class**)

   ```java
   @Configuration
   public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
       private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
   
       // 从容器中获取所有的WebMvcConfigurer
       @Autowired(required = false)
       public void setConfigurers(List<WebMvcConfigurer> configurers) {
           if (!CollectionUtils.isEmpty(configurers)) {
               this.configurers.addWebMvcConfigurers(configurers);
               // 一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
               // @Override
               // public void addViewControllers(ViewControllerRegistry registry) {
               //    for (WebMvcConfigurer delegate : this.delegates) {
               //       delegate.addViewControllers(registry);
               //   }
           }
       }
   }
   ```

4. 容器中所有的WebMvcConfigurer都会一起起作用；
5. 我们的配置类也会被调用；

**效果：SpringMVC的自动配置和我们的扩展配置都会起作用；**

#### 4.4.3 全面接管SpringMVC；

SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置；

所有的SpringMVC的自动配置都失效了；

**我们需要在配置类中添加@EnableWebMvc即可；**

```java
@EnableWebMvc  // 添加后全面接管SpringMVC
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /atguigu 请求来到 success
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```

**原理：**

为什么@EnableWebMvc自动配置就失效了？

1. @EnableWebMvc的核心

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

3. 重点在这个注解：`@Import(DelegatingWebMvcConfiguration.class)`

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
    // ...
}
```

3. 查看`WebMvcAutoConfiguration`类

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
                     WebMvcConfigurerAdapter.class })
// 容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
                     ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

}
```

4. 而`@EnableWebMvc`将`WebMvcConfigurationSupport`组件导入进来；

5. 导入的`WebMvcConfigurationSupport`只是`SpringMVC`最基本的功能；

### 4.5 如何修改SpringBoot的默认配置

模式：

1. SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（如：ViewResolver）将用户配置的和自己默认的组合起来；
2. 在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置
3. 在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置

### 4.6 RestfulCRUD

首先添加好相应的静态资源文件与相应的类：

![restfulCRUD](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313099.PNG)

#### 4.6.1 默认访问首页

需求：默认会访问到templates中的login.html

方式1：在控制器中添加访问映射

```java
@Controller
public class HelloController {
    @RequestMapping({"/", "/login"})
    public String index(){
        return "login";
    }
}
```

方式2：配置视图解析器

```java
// 使用WebMvcConfigurer可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean // 将组件注册在容器
    public WebMvcConfigurer webMvcConfigurerAdapter(){
        WebMvcConfigurer adapter = new WebMvcConfigurer() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
            }
        };
        return adapter;
    }
}
```

#### 4.6.2 国际化

**1）编写国际化配置文件；**

2）使用ResourceBundleMessageSource管理国际化资源文件

3）在页面使用fmt:message取出国际化内容

**步骤：**

1）编写国际化配置文件，抽取页面需要显示的国际化消息

> 1. 添加相应的配置文件：
>
>    ![国际化1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313542.PNG)
>
> 2. 打开建立的配置文件，选择Resource Bundle视图，在这个视图中添加属性；
>
>    ![国际化2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313722.PNG)
>
> 3. 抽取页面中需要国际化的元素，如下：
>
>    ![国际化3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313251.PNG)

2） SpringBoot自动配置好了管理国际化资源文件的组件；

```java
@ConfigurationProperties(prefix = "spring.messages")
public class MessageSourceAutoConfiguration {

    /**
	 * Comma-separated list of basenames (essentially a fully-qualified classpath
	 * location), each following the ResourceBundle convention with relaxed support for
	 * slash based locations. If it doesn't contain a package qualifier (such as
	 * "org.mypackage"), it will be resolved from the classpath root.
	 */
    private String basename = "messages";  
    // 我们的配置文件可以直接放在类路径下叫messages.properties;

    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        if (StringUtils.hasText(this.basename)) {
            //设置国际化资源文件的基础名（去掉语言国家代码的）
            messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(
                StringUtils.trimAllWhitespace(this.basename)));
        }
        if (this.encoding != null) {
            messageSource.setDefaultEncoding(this.encoding.name());
        }
        messageSource.setFallbackToSystemLocale(this.fallbackToSystemLocale);
        messageSource.setCacheSeconds(this.cacheSeconds);
        messageSource.setAlwaysUseMessageFormat(this.alwaysUseMessageFormat);
        return messageSource;
    }
}
```

由于之前配置的文件放置在了`resources/i18n/...properties`下；

因此，在application.properties文件中，修改默认配置：`spring.messages.basename=i18n.login`

3）去页面获取国际化的值，需要用thymeleaf的语法：

```html
<!DOCTYPE html>
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <meta name="description" content="">
        <meta name="author" content="">
        <title>Signin Template for Bootstrap</title>
        <!-- Bootstrap core CSS -->
        <link href="asserts/css/bootstrap.min.css" th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" rel="stylesheet">
        <!-- Custom styles for this template -->
        <link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
    </head>

    <body class="text-center">
        <form class="form-signin" action="dashboard.html">
            <img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
            <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
            <label class="sr-only" th:text="#{login.username}">Username</label>
            <input type="text" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
            <label class="sr-only" th:text="#{login.password}">Password</label>
            <input type="password" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="">
            <div class="checkbox mb-3">
                <label>
                    <input type="checkbox" value="remember-me"/> [[#{login.remember}]]
                </label>
            </div>
            <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
            <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
            <a class="btn btn-sm">中文</a>
            <a class="btn btn-sm">English</a>
        </form>

    </body>

</html>
```

效果：根据浏览器语言设置的信息切换了国际化。

> 会有出现乱码的情况，是由于Properties文件默认的编码为GBK，参考之前修改的操作

**原理：**

国际化Locale（区域信息对象）；LocaleResolver（获取区域信息对象）；

在WebMvcAutoConfiguration中默认配置了LocaleResolver，如下：

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
    if (this.mvcProperties
        .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }
    AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
    localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
    return localeResolver;
}
```

默认的就是根据浏览器发送的请求头带来的区域信息获取Locale进行国际化

```properties
# 在浏览器发送的Request Headers下有：
Accept-Language: zh-CN, zh;
```

4）需求：点击链接切换国际化，即

先修改一下login.html中的相应标签

```html
<a class="btn btn-sm" th:href="@{/index.html(l='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/index.html(l='en_US')}">English</a>
```

然后自己写一个LocaleResolver：

```java
/**
 * 可以在连接上携带区域信息
 */
public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("l");
        // 获取默认的Locale信息
        Locale locale = Locale.getDefault();
        // 当请求URL中没有携带区域信息时，就使用默认的
        if(!StringUtils.isEmpty(l)){
            String[] split = l.split("_");
            locale = new Locale(split[0],split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}

// 在 MVC的配置类MvcConfig.java中加入如下配置：
@Bean
public LocaleResolver localeResolver(){
    return new MyLocaleResolver();
}
```

#### 4.6.3 登陆

登录的表单：

```html
<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
	...	
</form>
```

Contorller编写：

```java
@Controller
public class LoginController {

    @PostMapping("/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String, Object> map){

        if (!StringUtils.isEmpty(username) && "123456".equals(password)) {
            // 登录成功,则到dashboard.html页面
            // return "dashboard";
            // 修改如下：防止表单重复提交，可以重定向到主页
            return "redirect:/main.html";
        }else{
            map.put("msg", "用户名密码错误");
            return "login";
        }
    }
}
```

> 重复提交说明：由于`return "dashboard"`是转发，因此URL地址还是/user/login，在页面刷新后，会重新提交一遍表单的信息内容，造成表单重复提交

登陆错误消息的显示：

```html
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

为了防止一直访问/user/login导致表单重复提交，在MyMvcConfig中的视图解析器中一个映射，如下：

```java
// 所有配置的 WebMvcConfigurerAdapter 组件都会一起起作用
@Bean // 将组件注册在容器
public WebMvcConfigurer webMvcConfigurerAdapter(){
    WebMvcConfigurer adapter = new WebMvcConfigurer() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/index.html").setViewName("login");
            // 添加一个新的视图映射，访问/main.html时映射到dashboard.html页面
            registry.addViewController("/main.html").setViewName("dashboard");
        }
    };
    return adapter;
}
```

> tips：开发期间模板引擎页面修改以后，要实时生效
>
> 1. 禁用模板引擎的缓存，在application.properties中加入
>
> ```Properties
> # 禁用缓存
> spring.thymeleaf.cache=false 
> ```
>
> 2. 页面修改完成以后在IDEA中`ctrl+f9`：重新编译；

#### 4.6.4 拦截器进行登陆检查

上述在经过视图映射后，会造成一旦访问main.html页面就会跳转到dashboard.html页面，不用登陆也可以进行访问，因此添加一个拦截器：

拦截器代码：

```java
public class LoginHandlerIntercepter implements HandlerInterceptor {
    
    // 目标方法执行之前进行拦截
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if(user == null){
            //未登陆，返回登陆页面
            request.setAttribute("msg","没有权限请先登陆");
            request.getRequestDispatcher("/index.html").forward(request,response);
            return false;
        }else{
            //已登陆，放行请求
            return true;
        }
    }
}
```

修改Controller中的代码，添加一个session域：

```java
@Controller
public class LoginController {

    @PostMapping("/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String, Object> map, HttpSession session){

        if (!StringUtils.isEmpty(username) && "123456".equals(password)) {
            // 登录成功
            // 在session域中添加user的信息，表示用户已登录，不会被拦截器拦截
            session.setAttribute("loginUser", username);
            return "redirect:/main.html";
        }else{
            map.put("msg", "用户名密码错误");
            return "login";
        }
    }
}
```

把拦截器注册到SpringMVC中

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    // 所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean // 将组件注册在容器
    public WebMvcConfigurer webMvcConfigurerAdapter(){
        WebMvcConfigurer adapter = new WebMvcConfigurer() {
            // 略：视图映射器的配置addViewControllers
            
            // 注册拦截器
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                // addPathPatterns("/**")：对所有的资源进行拦截
                // .excludePathPatterns()：对静态资源*.css , *.js等 与 /,/user/login 进行放行
                registry.addInterceptor(new LoginHandlerIntercepter()).addPathPatterns("/**")
                        .excludePathPatterns("/index.html","/","/user/login", "/webjars/**", "/asserts/**");
            }
        };
        return adapter;
    }
}
```

#### 4.6.5 CRUD-员工列表

**要求：**

1）RestfulCRUD：CRUD满足Rest风格；

URI：/资源名称/资源标识

**HTTP请求方式区分对资源CRUD操作**

|      | 普通CRUD（URI来区分操作） | RestfulCRUD          |
| ---- | ------------------------- | -------------------- |
| 查询 | getEmp                    | emp：GET请求         |
| 添加 | addEmp?xxx                | emp：POST请求        |
| 修改 | updateEmp?id=xxx&xxx=xx   | emp/{id}：PUT请求    |
| 删除 | deleteEmp?id=1            | emp/{id}：请求DELETE |

2）实验的请求架构;

| 实验功能                             | 请求URI  | 请求方式 |
| ------------------------------------ | -------- | -------- |
| 查询所有员工                         | emps     | GET      |
| 查询某个员工(来到修改页面)           | emp/{id} | GET      |
| 来到添加页面                         | emp      | GET      |
| 添加员工                             | emp      | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/{id} | GET      |
| 修改员工                             | emp      | PUT      |
| 删除员工                             | emp/{id} | DELETE   |

3）查询员工列表：

1. 实体类 Employee 与 Department

   ```java
   // 实体类：Employee 
   package cn.xyc.springboot07.entities;
   
   import java.util.Date;
   
   public class Employee {
   
       private Integer id;
       private String lastName;
       private String email;
       // 1 male, 0 female
       private Integer gender;
       private Department department;
       private Date birth;
       
       // constructor getting setting toString
   }
   
   // 实体类：Department
   package cn.xyc.springboot07.entities;
   
   public class Department {
   
       private Integer id;
       private String departmentName;
       
   	// constructor getting setting toString
   }
   ```
   
2. Dao层接口

   ```java
   @Repository
   public class EmployeeDao {
   	// 存储员工：假的数据库
   	private static Map<Integer, Employee> employees = null;
   	
   	@Autowired
   	private DepartmentDao departmentDao;
   	
   	static{
   		employees = new HashMap<Integer, Employee>();
   
   		employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1, new Department(101, "D-AA")));
   		employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1, new Department(102, "D-BB")));
   		employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0, new Department(103, "D-CC")));
   		employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0, new Department(104, "D-DD")));
   		employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1, new Department(105, "D-EE")));
   	}
   	
   	private static Integer initId = 1006;
   	
   	public void save(Employee employee){
   		if(employee.getId() == null){
   			employee.setId(initId++);
   		}
   		employee.setDepartment(departmentDao.getDepartment(employee.getDepartment().getId()));
   		employees.put(employee.getId(), employee);
   	}
   
   	//查询所有员工
   	public Collection<Employee> getAll(){
   		return employees.values();
   	}
   	
   	public Employee get(Integer id){
   		return employees.get(id);
   	}
   	
   	public void delete(Integer id){
   		employees.remove(id);
   	}
   }
   
   
   @Repository
   public class DepartmentDao {
   
   	private static Map<Integer, Department> departments = null;
   	
   	static{
   		departments = new HashMap<Integer, Department>();
   		
   		departments.put(101, new Department(101, "D-AA"));
   		departments.put(102, new Department(102, "D-BB"));
   		departments.put(103, new Department(103, "D-CC"));
   		departments.put(104, new Department(104, "D-DD"));
   		departments.put(105, new Department(105, "D-EE"));
   	}
   	
   	public Collection<Department> getDepartments(){
   		return departments.values();
   	}
   	
   	public Department getDepartment(Integer id){
   		return departments.get(id);
   	}
   }
   ```
   
3. 新增查询的控制器

   ```java
   @Controller
   public class EmployeeController {
   
       @Autowired
       EmployeeDao employeeDao;
   
       // 查询所有员工返回列表页面
       @GetMapping("/emps")
       public String list(Model model){
   
           Collection<Employee> employees = employeeDao.getAll();
           // 放在请求域中
           model.addAttribute("emps", employees);
           // thymleaf 默认会拼串
           // classpath:/templates/emp/list.html
           return "emp/list";
       }
   }
   ```

thymeleaf的内容：

> thymeleaf公共页面元素抽取：
>
> ```xml
> 1、抽取公共片段
> <div th:fragment="copy">
>        &copy; 2011 The Good Thymes Virtual Grocery
> </div>
> 
> 2、引入公共片段
> <div th:insert="~{footer :: copy}"></div>
> ~{templatename::selector}：模板名::选择器
> ~{templatename::fragmentname}:模板名::片段名
> 
> 3、默认效果：
> insert的公共片段在div标签中
> 如果使用th:insert等属性进行引入，可以不用写~{}：
> 行内写法可以加上：[[~{}]];[(~{})]；
> ```
>
> 三种引入公共片段的th属性：
>
> **th:insert**：将公共片段整个插入到声明引入的元素中
>
> **th:replace**：将声明引入的元素替换为公共片段
>
> **th:include**：将被引入的片段的内容包含进这个标签中
>
> > ```xml
> > <footer th:fragment="copy">
> >     &copy; 2011 The Good Thymes Virtual Grocery
> > </footer>
> > 
> > 引入方式
> > <div th:insert="footer :: copy"></div>
> > <div th:replace="footer :: copy"></div>
> > <div th:include="footer :: copy"></div>
> > 
> > 效果
> > <div>
> >        <footer>
> >            &copy; 2011 The Good Thymes Virtual Grocery
> >        </footer>
> > </div>
> > 
> > <footer>
> >        &copy; 2011 The Good Thymes Virtual Grocery
> > </footer>
> > 
> > <div>
> >        &copy; 2011 The Good Thymes Vrtual Grocery
> > </div>
> > ```

#### 4.6.6 CRUD-员工添加

添加add.html页面，页面内容略了；

在EmployeeContorller中添加方法：

```java
@Controller
public class EmployeeController {

    @Autowired
    EmployeeDao employeeDao;

    @Autowired
    DepartmentDao departmentDao;

    // 查询所有员工返回列表页面
    // 略...

    // 来到员工添加页面
    @GetMapping("/emp")
    public String toAddPage(Model model){
        // 来到添加页面，查出所有的部门，在页面显示：下拉条中显示
        Collection<Department> departments = departmentDao.getDepartments();
        model.addAttribute("depts", departments);
        return "emp/add";
    }

    // 员工添加
    // SpringMVC自动将请求参数和形参对象的属性一一绑定
    // 但是要求请求参数的名字和JavaBean对象属性名字相同
    @PostMapping("/emp")
    public String addEmp(Employee employee){
        // 添加员工
        employeeDao.save(employee);
        // 来到员工列表页面
        // redirect: 表示重定向到一个地址
        // forward: 表示转发到一个地址
        return "redirect:/emps";
    }

}
```

**存在问题：**

提交的数据格式不对：生日：日期；

2017-12-12；2017/12/12；2017.12.12；

日期的格式化：SpringMVC会将页面提交的值转换为指定的类型；

2017-12-12转为为Date类型； 类型转换，格式化；

而默认日期是按照`/`的方式；

**解决该问题：**

1. 通过查看WebMvcAutoConfiguration的源码，发现在方法dateFormatter中对日期格式进行了定义：

   ```java
   @Bean
   @ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")
   public Formatter<Date> dateFormatter() {
       return new DateFormatter(this.mvcProperties.getDateFormat());
   }
   
   // 其中：this.mvcProperties.getDateFormat()如下
   /**
     * Date format to use (e.g. dd/MM/yyyy).
     */
   private String dateFormat;
   ```

2. 而根据方法上的注解可知，可以在`application.properties`文件中对其进行修改：

   ```properties
   spring.mvc.date-format=yyyy-MM-dd
   ```

   添加完之后，在添加员工时`yyyy/MM/dd`的格式报错了，只有`yyyy-MM-dd`的格式才合法

#### 4.6.7 CRUD-员工修改

同样对于页面的修改略了...

在EmployeeContorller中添加方法：

```java
// 来到修改页面，查出当前员工，在页面回显
@GetMapping("/emp/{id}")
public String toEditPage(@PathVariable("id") Integer id, Model model){
    Employee employee = employeeDao.get(id);
    model.addAttribute("emp", employee);

    // 页面要显示部门列表
    Collection<Department> departments = departmentDao.getDepartments();
    model.addAttribute("depts", departments);

    return "emp/add";
}

// 员工修改
@PutMapping("/emp")
public String updateEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/emps";
}
```

#### 4.6.8 CRUD-员工删除

同样对于页面的修改略了...

在EmployeeContorller中添加方法：

```java
// 员工删除
@DeleteMapping("/emp/{id}")
public String deleteEmployee(@PathVariable("id") Integer id){
    employeeDao.delete(id);
    return "redirect:/emps";
}
```

> 注意，由于该案例是通过delete请求方式来发送请求的，因此在application.properties中配置如下属性：
>
> `spring.mvc.hiddenmethod.filter.enabled=true`

### 4.7 错误处理机制

#### 4.7.1 SpringBoot默认的错误处理机制

SpringBoot的默认默认效果：

1. **浏览器**，返回一个默认的错误页面

   ![默认错误页面](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313666.png)

   查看浏览器发送请求的请求头：

   ![浏览器发送的请求头](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313338.png)

2. **其他客户端**，默认响应一个json数据

![其他客户端相应](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313190.png)

​		其他客户端发送请求的请求头：

![其他客户端发送请求的请求头](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313325.png)

**默认配置的原理：**

SpringBoot有一个自动错误配置类：ErrorMvcAutoConfiguration；

这个类给给容器中添加了以下组件

1. DefaultErrorAttributes：帮我们在页面共享信息；

```java
@Override
public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
    Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
    errorAttributes.put("timestamp", new Date());  // 时间戳
    addStatus(errorAttributes, requestAttributes);  // 状态码
    addErrorDetails(errorAttributes, requestAttributes, includeStackTrace); 
    addPath(errorAttributes, requestAttributes);
    return errorAttributes;
}
```

2. BasicErrorController：处理默认`/error`请求

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    // 产生html类型的数据；浏览器发送的请求来到这个方法处理
    @RequestMapping(produces = "text/html")
    public ModelAndView errorHtml(HttpServletRequest request,
                                  HttpServletResponse response) {
        // 拿到了状态码
        HttpStatus status = getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
            request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());

        // 去哪个页面作为错误页面；包含页面地址和页面内容
        ModelAndView modelAndView = resolveErrorView(request, response, status, model);
        return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
    }

    // 产生json数据，其他客户端来到这个方法处理；
    @RequestMapping
    @ResponseBody    
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> body = getErrorAttributes(request,
                                                      isIncludeStackTrace(request, MediaType.ALL));
        HttpStatus status = getStatus(request);
        return new ResponseEntity<Map<String, Object>>(body, status);
    }
}
```

3. ErrorPageCustomizer：系统出现错误以后来到`/error`请求进行处理；（类似web.xml注册的错误页面规则）

```java
@Bean
public ErrorPageCustomizer errorPageCustomizer() {
    // new了一个ErrorPageCustomizer
    return new ErrorPageCustomizer(this.serverProperties);
}

private static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {

    private final ServerProperties properties;

    protected ErrorPageCustomizer(ServerProperties properties) {
        this.properties = properties;
    }

    // 注册错误页面
    @Override
    public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
        ErrorPage errorPage = new ErrorPage(this.properties.getServletPrefix()
                                            + this.properties.getError().getPath());
        // 上述getPath()见下方说明
        errorPageRegistry.addErrorPages(errorPage);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}

// 其中getPath()的配置如下
// @Value("${error.path:/error}")
// private String path = "/error";
```

4. DefaultErrorViewResolver：

```java
@Override
public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
    ModelAndView modelAndView = resolve(String.valueOf(status), model);
    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
    }
    return modelAndView;
}

private ModelAndView resolve(String viewName, Map<String, Object> model) {
    // 默认SpringBoot可以去找到一个页面？  error/404
    String errorViewName = "error/" + viewName;

    // 模板引擎可以解析这个页面地址就用模板引擎解析
    TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
        .getProvider(errorViewName, this.applicationContext);
    if (provider != null) {
        // 模板引擎可用的情况下返回到errorViewName指定的视图地址
        return new ModelAndView(errorViewName, model);
    }
    // 模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面   error/404.html
    return resolveResource(errorViewName, model);
}
```

**错误相应的步骤：**

1. 一但系统出现4xx或者5xx之类的错误；**ErrorPageCustomizer**就会生效（定制错误的响应规则）；就会来到/error请求；这个/error请求就会被**BasicErrorController**处理；

2. 响应页面；去哪个页面是由**DefaultErrorViewResolver**解析得到的；

```java
// 在BasicErrorController控制器中调用了resolveErrorView
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    // 所有的ErrorViewResolver得到ModelAndView
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

#### 4.7.2 定制错误响应

**如何定制错误的页面；** 	

* 有模板引擎的情况下；error/状态码; 【将错误页面命名为：错误状态码.html 放在模板引擎文件夹里面的error文件夹下】，发生此状态码的错误就会来到对应的页面；
  * 我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）；		
  * 页面能获取的信息：`DefaultErrorAttributes`
    * timestamp：时间戳
    * status：状态码
    * error：错误提示
    * exception：异常对象
    * message：异常消息
    * errors：JSR303数据校验的错误都在这里
* 没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找； 
* 以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

**如何定制错误的json数据；** 

> 首先自定义一个异常类：
>
> ```java
> package cn.xyc.springboot07.exception;
> 
> public class UserNotExistException extends RuntimeException {
>     public UserNotExistException(){
>            super("用户不存在f");
>        }
>    }
> ```
> 
> 方便测试，在HelloContorller中添加抛出异常
>
> ```java
>@ResponseBody
> @RequestMapping("/throw")
> public String testThrowException(@RequestParam("user") String user){
>     if (user.equals("nouser")){
>         throw new UserNotExistException();
>        }
>        return "noException";
>    }
> ```
>    
> 测试：
> 
>* 页面访问：`http://localhost:8080/throw?user=111`  输出：noException
> 
>* 页面访问：`http://localhost:8080/throw?user=nouser`  发生异常！来到了自定义的异常页面
> 
>* 通过postman访问，发送请求：`http://localhost:8080/throw?user=nouser`
> 
>  返回结果：
> 
>  ```json
>   {
>      "timestamp": "2020-11-04T07:10:25.703+00:00",
>       "status": 500,
>       "error": "Internal Server Error",
>       "message": "",
>       "path": "/throw"
>   }
>  ```

1. **自定义异常处理&返回定制json数据；**

```java
@ControllerAdvice
public class MyExceptionHandler {
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}
```

访问时就是自定义的方式，在访问页面或者通过postman发送相应的请求时都会相应Json数据：

```json
{
    "code": "user.notexist",
    "message": "用户不存在f"
}
```

但是**没有自适应效果！**即通过浏览器访问返回页面，通过客户端返回Json数据

2. **转发到/error进行自适应响应效果处理**

```java
// 将MyExceptionHandler修改为
@ExceptionHandler(UserNotExistException.class)
public String handleException(Exception e, HttpServletRequest request){
    Map<String,Object> map = new HashMap<>();
    // 传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
    /**
      * Integer statusCode = (Integer) request .getAttribute("javax.servlet.error.status_code");
      */
    request.setAttribute("javax.servlet.error.status_code",500);
    map.put("code","user.notexist");
    map.put("message",e.getMessage());
    // 转发到/error
    return "forward:/error";
}
```

3. **将我们的定制数据携带出去；**

出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）；

方式1：完全来编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，放在容器中；

**方式2：**页面上能用的数据，或者是json返回能用的数据都是通过**errorAttributes.getErrorAttributes**得到；

​	容器中DefaultErrorAttributes.getErrorAttributes()；默认进行数据处理的；

​	自定义ErrorAttributes

```java
//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company","atguigu");
        return map;
    }
}
```

最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容，

![定制数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313249.png)

### 4.8 配置嵌入式Servlet容器

**SpringBoot默认使用Tomcat作为嵌入式的Servlet容器；**

![默认的servlet容器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252313738.png)



#### 4.8.1 如何定制和修改Servlet容器的相关配置；

1. 修改和server有关的配置（ServerProperties，这个类其实也是实现了EmbeddedServletContainerCustomizer）；

```properties
// 通用的Servlet容器设置
// server.xxx
server.port=8081
server.context-path=/crud
// Tomcat的设置
// server.tomcat.xxx
server.tomcat.uri-encoding=UTF-8
```

2. 编写一个**EmbeddedServletContainerCustomizer**：嵌入式的Servlet容器的定制器；来修改Servlet容器的配置

```java
// 在加了@Configuration的配置类中修改
@Bean  // 一定要将这个定制器加入到容器中
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {

        // 定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}
```

#### 4.8.2 注册Servlet三大组件

Sevlet三大组件：**Servlet、Filter、Listener**

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件。

注册三大组件用以下方式：

1. ServletRegistrationBean

> 首先定义一个Servlet：
>
> ```java
> public class MyServlet extends HttpServlet {
>     @Override
>        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
>            resp.getWriter().write("hello servlet");
>        }
>    }
> ```

```java
// 注册三大组件
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(), "/myServlet");
    return registrationBean;
}
```

2. FilterRegistrationBean

> 定义一个Filter
>
> ```java
> public class MyFilter implements Filter {
>        @Override
>        public void init(FilterConfig filterConfig) throws ServletException {
> 
>        }
> 
>        @Override
>        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
>            System.out.println("MyFilter Process");
>            // 直接放行
>            filterChain.doFilter(servletRequest, servletResponse);
>        }
> 
>        @Override
>        public void destroy() {
> 
>        }
> }
> ```

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
```

3. ServletListenerRegistrationBean

> 定义一个Listener
>
> ```java
> public class MyListener implements ServletContextListener{
> 
>        @Override
>        public void contextInitialized(ServletContextEvent sce) {
>            System.out.println("contextInitialized");
>        }
> 
>        @Override
>        public void contextDestroyed(ServletContextEvent sce) {
>            System.out.println("contextDestroyed");
>        }
> }
> ```

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
```

4. DIspatcherServlet体现了注册三大组件

SpringBoot帮我们自动配置SpringMVC的时候，自动的注册SpringMVC的前端控制器；DIspatcherServlet；

DispatcherServletAutoConfiguration中：

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
    DispatcherServlet dispatcherServlet) {
    ServletRegistrationBean registration = new ServletRegistrationBean(
        dispatcherServlet, this.serverProperties.getServletMapping());
    // 默认拦截： /  所有请求；包静态资源，但是不拦截jsp请求；   
    // /*会拦截jsp
    // 可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
    registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
    registration.setLoadOnStartup(
        this.webMvcProperties.getServlet().getLoadOnStartup());
    if (this.multipartConfig != null) {
        registration.setMultipartConfig(this.multipartConfig);
    }
    return registration;
}
```

#### 4.8.3 替换为其他嵌入式Servlet容器

SpringBoot还能支持其他的Servlet容器；

默认支持：

![springboot支持其他容器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314205.png)

1. Tomcat（默认使用）

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <!-- 引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器； -->
</dependency>
```

2. 要切换 Jetty

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>  <!-- 排除tomcat的容器 -->
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

3. 要切换 Undertow

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>  <!-- 排除tomcat的容器 -->
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

#### 4.8.4 嵌入式Servlet容器自动配置原理

EmbeddedServletContainerAutoConfiguration：嵌入式的Servlet容器自动配置类

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
// 导入BeanPostProcessorsRegistrar：给容器中导入一些组件
// 导入了EmbeddedServletContainerCustomizerBeanPostProcessor：
// 后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作
@Import(BeanPostProcessorsRegistrar.class)
public class EmbeddedServletContainerAutoConfiguration {

    @Configuration
    // 判断当前是否引入了Tomcat依赖；
    @ConditionalOnClass({ Servlet.class, Tomcat.class })
    // 判断当前容器没有用户自己定义EmbeddedServletContainerFactory：嵌入式的Servlet容器工厂；
    // 作用：创建嵌入式的Servlet容器
    @ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
    public static class EmbeddedTomcat {

        @Bean  // 在容器中放入TomCat的容器工厂
        public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
            return new TomcatEmbeddedServletContainerFactory();
        }

    }

    /**
	 * Nested configuration if Jetty is being used.
	 */
    @Configuration
    @ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
                         WebAppContext.class })
    @ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
    public static class EmbeddedJetty {

        @Bean
        public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
            return new JettyEmbeddedServletContainerFactory();
        }

    }

    /**
	 * Nested configuration if Undertow is being used.
	 */
    @Configuration
    @ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
    @ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
    public static class EmbeddedUndertow {

        @Bean
        public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
            return new UndertowEmbeddedServletContainerFactory();
        }

    }
}
```

1. EmbeddedServletContainerFactory：嵌入式Servlet容器工厂

```java
public interface EmbeddedServletContainerFactory {
    // 获取嵌入式的Servlet容器
    EmbeddedServletContainer getEmbeddedServletContainer(
        ServletContextInitializer... initializers);
}
```

![嵌入式Servlet容器工厂](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314380.png)

2. EmbeddedServletContainer：（嵌入式的Servlet容器）

![嵌入式的Servlet容器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314644.png)

3. 以**TomcatEmbeddedServletContainerFactory**为例

```java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
    ServletContextInitializer... initializers) {
    // 创建一个Tomcat
    Tomcat tomcat = new Tomcat();

    // 配置Tomcat的基本环节
    File baseDir = (this.baseDirectory != null ? this.baseDirectory
                    : createTempDir("tomcat"));
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    prepareContext(tomcat.getHost(), initializers);

    // 将配置好的Tomcat传入进去，返回一个EmbeddedServletContainer；并且启动Tomcat服务器
    return getTomcatEmbeddedServletContainer(tomcat);
}
```

4. 我们对嵌入式容器的配置修改是怎么生效？

   1. 方式1：修改ServerProperties
   2. 方式2：EmbeddedServletContainerCustomizer

   **EmbeddedServletContainerCustomizer**：定制器帮我们修改了Servlet容器的配置

   怎么修改的原理？

5. 容器中导入了**EmbeddedServletContainerCustomizerBeanPostProcessor**

```java
// 初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    // 如果当前初始化的是一个ConfigurableEmbeddedServletContainer类型的组件
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
       //
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
    // 获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            // 从容器中获取所有这个类型的组件：EmbeddedServletContainerCustomizer
            // 定制Servlet容器，给容器中可以添加一个EmbeddedServletContainerCustomizer类型的组件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}
```

**步骤：**

1）SpringBoot根据导入的依赖情况，给容器中添加相应的EmbeddedServletContainerFactory，比如：TomcatEmbeddedServletContainerFactory

2）容器中某个组件要创建对象就会惊动后置处理器：EmbeddedServletContainerCustomizerBeanPostProcessor；只要是嵌入式的Servlet容器工厂，后置处理器就工作；

3）后置处理器，从容器中获取所有的**EmbeddedServletContainerCustomizer**，调用定制器的定制方法

#### 4.8.5 嵌入式Servlet容器启动原理

什么时候创建嵌入式的Servlet容器工厂？什么时候获取嵌入式的Servlet容器并启动Tomcat？

**获取嵌入式的Servlet容器工厂：**

1. SpringBoot应用启动运行run方法

2. refreshContext(context)：SpringBoot刷新IOC容器【创建IOC容器对象，并初始化容器，创建容器中的每一个组件】；

   > 如果是web应用创建**AnnotationConfigEmbeddedWebApplicationContext**，否则创建的是：**AnnotationConfigApplicationContext**

3. refresh(context)：**刷新刚才创建好的ioc容器；**

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

4. onRefresh(); web的ioc容器重写了onRefresh方法

5. webioc容器会创建嵌入式的Servlet容器；**createEmbeddedServletContainer**();

6. **获取嵌入式的Servlet容器工厂：**`EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();`

   从ioc容器中获取EmbeddedServletContainerFactory 组件；

   **TomcatEmbeddedServletContainerFactory**创建对象，后置处理器一看是这个对象，就获取所有的定制器来先定制Servlet容器的相关配置；

7. **使用容器工厂获取嵌入式的Servlet容器**：`this.embeddedServletContainer = containerFactory      .getEmbeddedServletContainer(getSelfInitializer());`

8. 嵌入式的Servlet容器创建对象并启动Servlet容器；

**先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来；**

**总结：IOC容器启动创建嵌入式的Servlet容器**

### 4.9 使用外置的Servlet容器

**嵌入式Servlet容器**：应用打成可执行的jar包

优点：简单、便携；

缺点：默认不支持JSP、优化定制比较复杂、

> 使用定制器：ServerProperties、自定义EmbeddedServletContainerCustomizer
>
> 自己编写嵌入式Servlet容器的创建工厂EmbeddedServletContainerFactory

**外置的Servlet容器**：外面安装Tomcat，通过war包的方式打包；

#### 4.9.1 步骤

1. 必须创建一个war项目；

   > 可以利用idea创建好目录结构，具体通过Project Structure进行操作，最终结构如下：
   >
   > ![war项目目录](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314930.PNG)

2. 将本地的服务器整合进IDEA中；

   > 1. 通过Edit Configurations-Run/Debug Configurations进行配置
   >
   > ![tomcat整合进IDEA中](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314275.PNG)
   >
   > ![tomcat整合进IDEA中2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314605.PNG)

2. 将嵌入式的Tomcat指定为provided；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

3. 必须编写一个**SpringBootServletInitializer**的子类，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {
   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       // 传入SpringBoot应用的主程序
      return application.sources(Springboot08Application.class);
   }
}
```

4. 启动服务器就可以使用；

   配置完成后，启动Tomcat服务器，可以成功进入编写的JSP页面了

#### 4.9.2 原理

**jar包**：执行SpringBoot主类的main方法，启动IOC容器，创建嵌入式的Servlet容器；

**war包**：启动服务器，**服务器启动SpringBoot应用**，核心：SpringBootServletInitializer，启动ioc容器；

servlet3.0：规范

> 8.2.4 Shared libraries / runtimes pluggability：

规则：

1. 服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面`ServletContainerInitializer`实例：

2. `ServletContainerInitializer`的实现放在jar包的META-INF/services文件夹下，有一个名为`javax.servlet.ServletContainerInitializer`的文件，内容就是`ServletContainerInitializer`的实现类的全类名

3. 还可以使用`@HandlesTypes`，在应用启动的时候加载我们感兴趣的类；

**流程：**

1. 启动Tomcat服务器；

2. `org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer`：Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer**
3. `SpringServletContainerInitializer`将`@HandlesTypes(WebApplicationInitializer.class)`标注的所有这个类型的类都传入到`onStartup`方法的`Set<Class<?>>`；为这些`WebApplicationInitializer`类型的类创建实例；
4. 每一个`WebApplicationInitializer`都调用自己的`onStartup`；

![WebApplicationInitializer](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252314798.png)

5. 相当于我们的`SpringBootServletInitializer`的类会被创建对象，并执行`onStartup`方法
6. `SpringBootServletInitializer`实例执行`onStartup`的时候会`createRootApplicationContext`；创建容器

```java
protected WebApplicationContext createRootApplicationContext(
    ServletContext servletContext) {
    // 1、创建SpringApplicationBuilder
    SpringApplicationBuilder builder = createSpringApplicationBuilder();
    StandardServletEnvironment environment = new StandardServletEnvironment();
    environment.initPropertySources(servletContext, null);
    builder.environment(environment);
    builder.main(getClass());
    ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
    if (parent != null) {
        this.logger.info("Root context already created (using as parent).");
        servletContext.setAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
        builder.initializers(new ParentContextApplicationContextInitializer(parent));
    }
    builder.initializers(
        new ServletContextApplicationContextInitializer(servletContext));
    builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);

    // 调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
    builder = configure(builder);

    // 使用builder创建一个Spring应用
    SpringApplication application = builder.build();
    if (application.getSources().isEmpty() && AnnotationUtils
        .findAnnotation(getClass(), Configuration.class) != null) {
        application.getSources().add(getClass());
    }
    Assert.state(!application.getSources().isEmpty(),
                 "No SpringApplication sources have been defined. Either override the "
                 + "configure method or add an @Configuration annotation");
    // Ensure error pages are registered
    if (this.registerErrorPageFilter) {
        application.getSources().add(ErrorPageFilterConfiguration.class);
    }
    //启动Spring应用
    return run(application);
}
```

7. Spring的应用就启动并且创建IOC容器

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    FailureAnalyzers analyzers = null;
    configureHeadlessProperty();
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                                                                 applicationArguments);
        Banner printedBanner = printBanner(environment);
        context = createApplicationContext();
        analyzers = new FailureAnalyzers(context);
        prepareContext(context, environment, listeners, applicationArguments,
                       printedBanner);

        //刷新IOC容器
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        listeners.finished(context, null);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        return context;
    }
    catch (Throwable ex) {
        handleRunFailure(context, listeners, analyzers, ex);
        throw new IllegalStateException(ex);
    }
}
```

**总结：启动Servlet容器，再启动SpringBoot应用**

