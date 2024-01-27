# Spring5：学习笔记

> 最为应用的依赖容器Spring在工作中可谓用的十分频繁，之前学的很浅，最近有空就抽个周末的时间每天学习一点点，一下是原学习视频地址链接及代码链接：
>
> * [学习视频地址](https://www.bilibili.com/video/BV1P44y1N7QG)
> * [代码地址]()

## 01. 容器接口

`BeanFactory` 接口，典型功能有：`getBean`

`ApplicationContext` 接口，是 `BeanFactory` 的子接口。它扩展了 `BeanFactory` 接口的功能，如：

* 国际化
* 通配符方式获取一组 Resource 资源
* 整合 Environment 环境（能通过它获取各种来源的配置信息）
* 事件发布与监听，实现组件之间的解耦

可以看到，这里都是 `BeanFactory` 提供的基本功能，`ApplicationContext` 中的扩展功能都没有用到。

### 1.1 BeanFactory 与 ApplicationContext 的区别

首先的话创建一个SpringBoot的应用：

```java
package cn.xyc;


import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * BeanFactory 与 ApplicationContext 的区别
 */
@Slf4j
@SpringBootApplication
public class Class01Application {

    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(Class01Application.class, args);
    }
}
```

可以看到在`SpringApplication.run`后，我们获取到了一个容器`ConfigurableApplicationContext`，查看其类图：

![ConfigurableApplicationContext类关系图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231334640.png)

可以看到 `ConfigurableApplicationContext` 继承自 `ApplicationContext`，而`ApplicationContext`又继承自 `BeanFactory`，**那到底什么是 `BeanFactory`？**

1. 它是 `ApplicationContext` 的父接口；
2. 它才是 Spring 的核心容器, 主要的 `ApplicationContext` 实现都 **组合** 了它的功能

利用`ApplicationContext`从spring容器中获取bean：`context.getBean("ClassName");` 实则是调用了`BeanFactory`中的方法，`BeanFactory`的类结构如下：

![BeanFactory中的方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231357790.png)

在 Debug 模式下查看下 `ConfigurableApplicationContext` 中的内容（这里看到的是`ConfigurableApplicationContext` 的实现类 `AnnotationConfigServletWebApplicationContext`）：

![context类Debug](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231359580.png)

可以看到其中包含了这个 `BeanFactory` 的子类 `DefaultListableBeanFactory`，下面有类图。

### 1.2 BeanFactory的功能

`BeanFactory`类结构：

![BeanFactory中的方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231357790.png)

**BeanFactory 能干点啥？**

1. 表面上只有 `getBean`；
2. 实际上控制反转、基本的依赖注入、直至 Bean 的生命周期的各种功能，都由它的实现类提供；

> 其实现类：`DefaultListableBeanFactory`，后续分析
>
> ![DefaultListableBeanFactory类关系图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231406698.png)

暂且不细看`DefaultListableBeanFactory`，查看类`DefaultSingletonBeanRegistry`中的成员变量：

![DefaultSingletonBeanRegistry中的成员变量](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231444176.png)

```java
// 写个DEMO查看一下：拿出DefaultSingletonBeanRegistry类中的单例对象
Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
singletonObjects.setAccessible(true);
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
map.forEach((k, v)->{
    System.out.println(k + "=" + v);
});
```

> 上面输出的类过多了，这里通过自己往Spring容器中放入对象来观察：
>
> ```java
> // 类1：Component1
> package cn.xyc;
> 
> import org.slf4j.Logger;
> import org.slf4j.LoggerFactory;
> import org.springframework.stereotype.Component;
> 
> @Component
> public class Component1 {
> 
>     private static final Logger log = LoggerFactory.getLogger(Component1.class);
> 
> }
> 
> // 类2：Component2
> package cn.xyc;
> 
> import org.slf4j.Logger;
> import org.slf4j.LoggerFactory;
> import org.springframework.stereotype.Component;
> 
> @Component
> public class Component2 {
> 
>     private static final Logger log = LoggerFactory.getLogger(Component2.class);
> 
> }
> 
> // main函数中找出这两个我们注入进去的类：
> Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
> singletonObjects.setAccessible(true);
> ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
> Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
> 
> map.entrySet().stream().filter(e -> e.getKey().startsWith("component"))
>     .forEach(e-> System.out.println(e.getKey() + "=" + e.getValue()));
> 
> // 输出结果：
> // component1=cn.xyc.Component1@45f24169
> // component2=cn.xyc.Component2@6ad5923a
> ```

### 1.3 ApplicationContext接口

![ApplicationContext接口](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231420638.png)

**ApplicationContext 比 BeanFactory 多点啥？**ApplicationContext 多实现了四个接口：

* `MessageSource`: 国际化功能，支持多种语言

  > resource文件夹下搞点配置文件：
  >
  > * `resource/messages.propertes`，添加内容：空
  > * `resource/message_en.properties`，添加内容：`hi=hello`
  > * `resource/messages_zh.properties`，添加内容：`hi=你好`
  > * `resource/messages_ja.properties`，添加内容：`hi=こんにちは`
  >
  > ```java
  > System.out.println(context.getMessage("hi", null, Locale.CHINA));  // 输出：你好
  > System.out.println(context.getMessage("hi", null, Locale.ENGLISH));  // 输出：Hello
  > System.out.println(context.getMessage("hi", null, Locale.JAPANESE));  // 输出：こんにちは
  > ```

* `ResourcePatternResolver`: 通配符匹配资源路径

  ```java
  // classpath:application.properties路径下的文件
  Resource[] resources = context.getResources("classpath:application.properties");
  for (Resource resource : resources) {
      System.out.println(resource);
  }
  // 输出如下：class path resource [application.properties]
  Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
  for (Resource resource : resources) {
      System.out.println(resource);
  }
  // 输出如下：
  // URL [file:/C:/Users/ZhuCC/Desktop/Spring/SpringCode/spring_class_01/target/classes/META-INF/spring.factories]
  // URL [jar:file:/C:/Software/Maven/repository/org/springframework/boot/spring-boot/2.6.4/spring-boot-2.6.4.jar!/META-INF/spring.factories]
  // URL [jar:file:/C:/Software/Maven/repository/org/springframework/boot/spring-boot-autoconfigure/2.6.4/spring-boot-autoconfigure-2.6.4.jar!/META-INF/spring.factories]
  // URL [jar:file:/C:/Software/Maven/repository/org/springframework/spring-beans/5.3.16/spring-beans-5.3.16.jar!/META-INF/spring.factories]
  ```

  > 对上述`classpath*`加`*`与`不加*`的说明，不加就不会去jar包里搜索，机会搜索你工程目录下的内容，即不加的话仅输出：`class path resource [META-INF/spring.factories]`

* `EnvironmentCapable`: 环境信息，系统环境变量，`*.properties`、`*.application.yml` 等配置文件中的值

  ```java
  // 拿系统变量的信息
  System.out.println(context.getEnvironment().getProperty("java_home"));
  // 输出：C:\Software\Java\jdk1.8.0_241
  // 拿配置文件中的信息
  System.out.println(context.getEnvironment().getProperty("server.port"));
  // 输出：8080
  ```

* `ApplicationEventPublisher`: 发布事件对象，**用于解耦**

  * **案例1：**

  ```java
  context.publishEvent(new UserRegisteredEvent(context));
  // 启动后输出：
  // [DEBUG] 15:17:41.451 [main] cn.xyc.Component2                   - 收到事件：cn.xyc.UserRegisteredEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@3c947bc5, started on Sat Jul 23 15:17:40 CST 2022] 
  ```

  > 事件来源，需要继承`ApplicationEvent`
  >
  > ```java
  > package cn.xyc;
  > 
  > import org.springframework.context.ApplicationEvent;
  > 
  > public class UserRegisteredEvent extends ApplicationEvent {
  >     public UserRegisteredEvent(Object source) {
  >         super(source);
  >     }
  > }
  > ```
  >
  > 监听器：任何一个由Spring管理的组件都可以作为监听器，但需要定义一个处理事件的方法，参数类型为**用户注册事**件类的对象，方法头上需要加上`@EventListener`注解，这里选择`Component2`作为监听器，
  >
  > ```java
  > package cn.xyc;
  > 
  > import org.slf4j.Logger;
  > import org.slf4j.LoggerFactory;
  > import org.springframework.context.event.EventListener;
  > import org.springframework.stereotype.Component;
  > 
  > @Component
  > public class Component2 {
  > 
  >     private static final Logger log = LoggerFactory.getLogger(Component2.class);
  > 
  >     @EventListener
  >     public void listener(UserRegisteredEvent event) {
  >         log.debug("收到事件：{}", event);
  >     }
  > }
  > ```

  * 案例2：

  ```java
  context.getBean(Component1.class).register();
  ```

  > ```java
  > // 发布者
  > package cn.xyc;
  > 
  > import org.slf4j.Logger;
  > import org.slf4j.LoggerFactory;
  > import org.springframework.beans.factory.annotation.Autowired;
  > import org.springframework.context.ApplicationEventPublisher;
  > import org.springframework.stereotype.Component;
  > 
  > @Component
  > public class Component1 {
  > 
  >     private static final Logger log = LoggerFactory.getLogger(Component1.class);
  >     
  >     @Autowired
  >     private ApplicationEventPublisher publisher;
  > 
  >     public void register() {
  >         log.debug("用户注册");
  >         publisher.publishEvent(new UserRegisteredEvent(this));
  >     }
  > }
  > 
  > // 监听者
  > @Component
  > public class Component2 {
  > 
  >     private static final Logger log = LoggerFactory.getLogger(Component2.class);
  > 
  >     @EventListener
  >     public void listener(UserRegisteredEvent event) {
  >         log.debug("收到事件：{}", event);
  >     }
  > }
  > ```
  >
  > 日志输出：
  >
  > ```java
  > [DEBUG] 15:23:29.811 [main] cn.xyc.Component1                   - 用户注册 
  > [DEBUG] 15:23:29.811 [main] cn.xyc.Component2                   - 收到事件：cn.xyc.UserRegisteredEvent[source=cn.xyc.Component1@1dab9dd6] 
  > ```

### 1.4 总结

**这节学到了什么？**

1. `BeanFactory` 与 `ApplicationContext` 并不仅仅是简单接口继承的关系，`ApplicationContext` 组合并扩展了 `BeanFactory` 的功能（扩展实现）
2. 又新学一种代码之间解耦途径

**练习**：完成用户注册与发送短信之间的解耦，用事件方式、和 AOP 方式分别实现

## 02. 容器实现

Spring 的发展历史较为悠久，因此很多资料还在讲解它较旧的实现，这里出于怀旧的原因，把它们都列出来，供参考

* `DefaultListableBeanFactory`，是 BeanFactory 最重要的实现，像**控制反转**和**依赖注入**功能，都是它来实现
* `ClassPathXmlApplicationContext`，从类路径查找 XML 配置文件，创建容器（旧）
* `FileSystemXmlApplicationContext`，从磁盘路径查找 XML 配置文件，创建容器（旧）
* `XmlWebApplicationContext`，传统 SSM 整合时，基于 XML 配置文件的容器（旧）
* `AnnotationConfigWebApplicationContext`，传统 SSM 整合时，基于 java 配置类的容器（旧）
* `AnnotationConfigApplicationContext`，SpringBoot 中非 web 环境容器（新）
* `AnnotationConfigServletWebServerApplicationContext`，SpringBoot 中 servlet web 环境容器（新）
* `AnnotationConfigReactiveWebServerApplicationContext`，SpringBoot 中 reactive web 环境容器（新）

另外要注意的是，后面这些带有 `ApplicationContext` 的类都是 `ApplicationContext` 接口的实现，但它们是**组合**了 `DefaultListableBeanFactory` 的功能，并非继承而来

### 2.1 BeanFactory实现的特点

![DefaultListableBeanFactory类关系图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207231545980.png)

`DefaultListableBeanFactory`类作为`BeanFactory`最为重要的实现，先看个例子：

```java
package cn.xyc;

import lombok.Getter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class TestBeanFactory {

    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        // bean 的定义（class, scope, 初始化, 销毁）
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
        beanFactory.registerBeanDefinition("config", beanDefinition);
		// 输出现在定义的bean，仅输出：config
        for (String name : beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
    }

    @Configuration
    static class Config {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }
    }

    static class Bean1 {
        private static final Logger log = LoggerFactory.getLogger(Bean1.class);

        public Bean1() {
            log.debug("构造 Bean1()");
        }

        @Autowired
        @Getter
        private Bean2 bean2;

    }

    static class Bean2 {
        private static final Logger log = LoggerFactory.getLogger(Bean2.class);

        public Bean2() {
            log.debug("构造 Bean2()");
        }
    }
}
```

上述输出仅输出：`config`，可见此时容器中仅有一个bean对象，可见`@Configuration`和`@Bean`注解并没有被解析，说明类`DefaultListableBeanFactory`的功能并不完整，**即`@Configuration`和`@Bean`注解对BeanFactory而言为拓展功能；**

继续添加内容：

```java
public static void main(String[] args) {
    DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
    // bean 的定义（class, scope, 初始化, 销毁）
    AbstractBeanDefinition beanDefinition =
        BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
    beanFactory.registerBeanDefinition("config", beanDefinition);

    // 给 BeanFactory 添加一些常用的后处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);

    for (String name : beanFactory.getBeanDefinitionNames()) {
        System.out.println(name);
    }
}
```

> 再次执行，可见此时容器中bean多了：
>
> ```java
> config
> // BeanFactory后处理器，用于解析@Configuration，@Bean
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> // bean后处理器，用于解析@Autowired注解
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> // bean后处理器，用于解析@Resource注解
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> ```

继续在后面添加内容：

```java
// BeanFactory 后处理器主要功能，补充了一些 bean 定义
beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(beanFactory::addBeanPostProcessor);
```

> 再次执行，
>
> ```java
> config
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> bean1  // 注解起作用，所定义的Bean都被注入了
> bean2
> ```

继续添加，在注入了bean1和bean2后，可以使用bean1&bean2了，如下：

```java
System.out.println(beanFactory.getBean(Bean1.class).getBean2());
// [DEBUG] 15:56:24.125 [main] cn.xyc.TestBeanFactory$Bean1        - 构造 Bean1() 
// null
```

上面可知，bean1被成功构造，但bean2却没有被`@Autowired`，**可知`@Autowired`对于BeanFactory而言也是拓展功能**；

继续添加后处理器：

```java
// Bean 后处理器, 针对 bean 的生命周期的各个阶段提供扩展, 例如 @Autowired @Resource ...
beanFactory.getBeansOfType(BeanPostProcessor.class).values().stream()
    .sorted(beanFactory.getDependencyComparator())
    .forEach(beanPostProcessor -> {
        System.out.println(">>>>" + beanPostProcessor);
        beanFactory.addBeanPostProcessor(beanPostProcessor);
    });
```

> 再次执行，
>
> ```java
> [DEBUG] 16:03:51.816 [main] cn.xyc.TestBeanFactory$Bean1        - 构造 Bean1() 
> [DEBUG] 16:03:51.826 [main] cn.xyc.TestBeanFactory$Bean2        - 构造 Bean2() 
> ```

关于Bean的创建时机，从上述输出来看，Bean对象都是在用的时候被创建的，但对于单例对象可以选择先创建：

```java
// 准备好所有单例
beanFactory.preInstantiateSingletons(); 
System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> ");
System.out.println(beanFactory.getBean(Bean1.class).getInter());
```

> 输出如下：
>
> ```java
> [DEBUG] 16:07:20.584 [main] cn.xyc.TestBeanFactory$Bean1        - 构造 Bean1() 
> [DEBUG] 16:07:20.594 [main] cn.xyc.TestBeanFactory$Bean2        - 构造 Bean2() 
> >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
> // 如果不加beanFactory.preInstantiateSingletons()的话，>>>>>>>>>会输出在构造函数前
> ```

**总结：**

1. beanFactory 不会做的事

   1. 不会主动调用 BeanFactory 后处理器
   2. 不会主动添加 Bean 后处理器
   3. 不会主动初始化单例
   4. 不会解析beanFactory 还不会解析 `${ }` 与 `#{ }`

2. bean 后处理器会有排序的逻辑

   > 重新定义下类内容：
   >
   > ```java
   > @Configuration
   > static class Config {
   >     @Bean
   >     public Bean1 bean1() {
   >         return new Bean1();
   >     }
   > 
   >     @Bean
   >     public Bean2 bean2() {
   >         return new Bean2();
   >     }
   > 
   >     @Bean
   >     public Bean3 bean3() {
   >         return new Bean3();
   >     }
   > 
   >     @Bean
   >     public Bean4 bean4() {
   >         return new Bean4();
   >     }
   > }
   > 
   > static class Bean1 {
   >     private static final Logger log = LoggerFactory.getLogger(Bean1.class);
   > 
   >     public Bean1() {
   >         log.debug("构造 Bean1()");
   >     }
   > 
   >     @Autowired
   >     @Getter
   >     private Bean2 bean2;
   > 
   >     @Autowired
   >     @Getter
   >     private Inter inter;
   > }
   > 
   > static class Bean2 {
   >     private static final Logger log = LoggerFactory.getLogger(Bean2.class);
   > 
   >     public Bean2() {
   >         log.debug("构造 Bean2()");
   >     }
   > }
   > 
   > interface Inter { }
   > 
   > static class Bean3 implements Inter { }
   > 
   > static class Bean4 implements Inter { }
   > ```
   >
   > 情景1：是否能注入成功？**不能！Bean3&Bean4都实现了Inter，因此`@Autowired`无法区分**
   >
   > ```java
   > // class:Bean1
   > @Autowired
   > private Inter inter;
   > ```
   >
   > 情景2：是否能注入成功？**可以注入成功**
   >
   > ```java
   > // class:Bean1
   > @Autowired
   > private Inter bean3;
   > // 验证下：System.out.println(beanFactory.getBean(Bean1.class).getInter());
   > // cn.xyc.TestBeanFactory$Bean3@9225652
   > ```
   >
   > 情景3：`@Resource`注解，基本和`@Autowired`相同
   >
   > ```java
   > @Resource(name = "bean4")
   > private Inter bean3;
   > 
   > // 验证下：System.out.println(beanFactory.getBean(Bean1.class).getInter());
   > // cn.xyc.TestBeanFactory$Bean4@6a28ffa4
   > ```
   >
   > 情景4：同时加上`@Autowired`注解和`@Resource`注解
   >
   > ```java
   > @Autowired  // 应该会匹配到bena3
   > @Resource(name = "bean4")  // 应该会匹配到bena4
   > private Inter bean3;
   > // 输出：cn.xyc.TestBeanFactory$Bean3@1700915
   > ```
   >
   > 那情景4中为什么注入的是Bean3呢？**这就涉及到了Bean后处理器加入的顺序**：
   >
   > ```java
   > // BeanFactory 的加入顺序如下
   > beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(beanPostProcessor -> {
   >     System.out.println(beanPostProcessor);
   >     beanFactory.addBeanPostProcessor(beanPostProcessor);
   > });
   > 
   > // 输出如下：可以看到是Autowired后处理器先加进来的
   > // org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@7ee955a8
   > // org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@1677d1
   > ```
   >
   > 控制注入的先后：
   >
   > ```java
   > beanFactory.getBeansOfType(BeanPostProcessor.class).values().stream()
   >     .sorted(beanFactory.getDependencyComparator())
   >     .forEach(beanPostProcessor -> {
   >         System.out.println(beanPostProcessor);
   >         beanFactory.addBeanPostProcessor(beanPostProcessor);
   >     });
   > 
   > // 输出：这回是CommonAnnotationBean先被注入，可以看到@Resource注解起作用了，注入的是Bean4对象
   > // org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@730d2164
   > // org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@24959ca4
   > // cn.xyc.TestBeanFactory$Bean4@2a7ed1f
   > ```
   >
   > 这里关于如何排序的内容不再展开：
   >
   > ```java
   > System.out.println("Common:" + (Ordered.LOWEST_PRECEDENCE - 3));
   > System.out.println("Autowired:" + (Ordered.LOWEST_PRECEDENCE - 2));
   > ```

### 2.2 常见ApplicationContext接口

`ClassPathXmlApplicationContext`，从类路径查找 XML 配置文件，创建容器

* 创建配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!-- 控制反转, 让 bean1 被 Spring 容器管理 -->
      <bean id="bean1" class="cn.xyc.Class02Application.Bean1"/>
  
      <!-- 控制反转, 让 bean2 被 Spring 容器管理 -->
      <bean id="bean2" class="cn.xyc.Class02Application.Bean2">
          <!-- 依赖注入, 建立与 bean1 的依赖关系 -->
          <property name="bean1" ref="bean1"/>
      </bean>
  </beans>
  ```

* Bean类定义：

  ```java
  static class Bean1 { }
  
  static class Bean2 {
      @Setter
      @Getter
      private Bean1 bean1;
  }
  ```

* 测试函数：

  ```java
  package cn.xyc;
  
  import lombok.Getter;
  import lombok.Setter;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  
  /**
   * 常见 ApplicationContext 实现
   */
  public class Class02Application {
  
      private static final Logger log = LoggerFactory.getLogger(Class02Application.class);
  
      public static void main(String[] args) {
          testClassPathXmlApplicationContext();
      }
  
      // 较为经典的容器, 基于 classpath 下 xml 格式的配置文件来创建
      private static void testClassPathXmlApplicationContext() {
          ClassPathXmlApplicationContext context =
                  new ClassPathXmlApplicationContext("Class02Applicaiton.xml");
  
          for (String name : context.getBeanDefinitionNames()) {
              System.out.println(name);
          }
  
          System.out.println(context.getBean(Bean2.class).getBean1());
      }
  }
  
  // 输出如下：
  // org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@1677d1
  // org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@48fa0f47
  // [DEBUG] 16:46:05.111 [main] cn.xyc.TestBeanFactory$Bean1        - 构造 Bean1() 
  // [DEBUG] 16:46:05.123 [main] cn.xyc.TestBeanFactory$Bean2        - 构造 Bean2() 
  // cn.xyc.TestBeanFactory$Bean4@21de60b4
  ```

* `ClassPathXmlApplicationContext`内部干了啥？

  ```java
  DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
  System.out.println("读取之前...");
  for (String name : beanFactory.getBeanDefinitionNames()) {
      System.out.println(name);
  }
  System.out.println("读取之后...");
  XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
  reader.loadBeanDefinitions(new ClassPathResource("Class02Applicaiton.xml"));
  for (String name : beanFactory.getBeanDefinitionNames()) {
      System.out.println(name);
  }
  
  // 输出内容：
  // 读取之前...  (可以看到这里没有内容)
  // 读取之后...  (可以看到在读取之后存在内容了)
  // bean1
  // bean2
  ```

`FileSystemXmlApplicationContext`，从磁盘路径查找 XML 配置文件，创建容器（旧）

* 测试代码

  ```java
  // 基于磁盘路径下 xml 格式的配置文件来创建
  private static void testFileSystemXmlApplicationContext() {
      FileSystemXmlApplicationContext context =
          new FileSystemXmlApplicationContext(
          "src\\main\\resources\\Class02Applicaiton.xml");
      for (String name : context.getBeanDefinitionNames()) {
          System.out.println(name);
      }
  
      System.out.println(context.getBean(Bean2.class).getBean1());
  }
  ```

* `FileSystemXmlApplicationContext`内部干了啥？

  ```java
  DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
  System.out.println("读取之前...");
  for (String name : beanFactory.getBeanDefinitionNames()) {
      System.out.println(name);
  }
  System.out.println("读取之后...");
  XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
  reader.loadBeanDefinitions(new FileSystemResource("src\\main\\resources\\Class02Applicaiton.xml"));
  for (String name : beanFactory.getBeanDefinitionNames()) {
      System.out.println(name);
  }
  ```

  > 这里注意运行时工作目录的设置

`AnnotationConfigApplicationContext`，SpringBoot 中非 web 环境容器

* 测试代码

  ```java
  // 较为经典的容器, 基于 java 配置类来创建
  private static void testAnnotationConfigApplicationContext() {
      AnnotationConfigApplicationContext context =
          new AnnotationConfigApplicationContext(Config.class);
  
      for (String name : context.getBeanDefinitionNames()) {
          System.out.println(name);
      }
  
      System.out.println(context.getBean(Bean2.class).getBean1());
  }
  
  @Configuration
  static class Config {
      @Bean
      public Bean1 bean1() {
          return new Bean1();
      }
  
      @Bean
      public Bean2 bean2(Bean1 bean1) {
          Bean2 bean2 = new Bean2();
          bean2.setBean1(bean1);
          return bean2;
      }
  }
  
  // 输出内容：
  // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
  // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
  // org.springframework.context.annotation.internalCommonAnnotationProcessor
  // org.springframework.context.event.internalEventListenerProcessor
  // org.springframework.context.event.internalEventListenerFactory
  // class02Application.Config
  // bean1
  // bean2
  // cn.xyc.Class02Application$Bean1@368f2016
  ```

  > 可以看到容器中默认加了很多的处理器，其实这些就等价于在xml配置文件中的`<context:annotation-config/>`，如下：在配置文件中加入`<context:annotation-config/>`，再次执行方法`testClassPathXmlApplicationContext();`，输出如下：
  >
  > ```java
  > bean1
  > bean2
  > org.springframework.context.annotation.internalConfigurationAnnotationProcessor
  > org.springframework.context.annotation.internalAutowiredAnnotationProcessor
  > org.springframework.context.annotation.internalCommonAnnotationProcessor
  > org.springframework.context.event.internalEventListenerProcessor
  > org.springframework.context.event.internalEventListenerFactory
  > cn.xyc.Class02Application$Bean1@1f3f4916
  > ```
  >
  > 对比之前没有加这个配置项的输出，可以明显发现这里多加了几个处理器。

`AnnotationConfigServletWebServerApplicationContext`，SpringBoot 中 servlet web 环境容器（新）

* 测试代码

  ```java
  // 较为经典的容器, 基于 java 配置类来创建, 用于 web 环境
  private static void testAnnotationConfigServletWebServerApplicationContext() {
      AnnotationConfigServletWebServerApplicationContext context =
          new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
      for (String name : context.getBeanDefinitionNames()) {
          System.out.println(name);
      }
  }
  
  @Configuration
  static class WebConfig {
      @Bean
      public ServletWebServerFactory servletWebServerFactory(){
          return new TomcatServletWebServerFactory();
      }
      @Bean
      public DispatcherServlet dispatcherServlet() {
          return new DispatcherServlet();
      }
      @Bean
      public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet) {
          return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
      }
      @Bean("/hello")
      public Controller controller1() {
          return (request, response) -> {
              response.getWriter().print("hello");
              return null;
          };
      }
  }
  
  @Configuration
  static class Config {
      @Bean
      public Bean1 bean1() {
          return new Bean1();
      }
  
      @Bean
      public Bean2 bean2(Bean1 bean1) {
          Bean2 bean2 = new Bean2();
          bean2.setBean1(bean1);
          return bean2;
      }
  }
  ```

**总结：**

1. 常见的 `ApplicationContext` 容器实现；
2. 内嵌容器、`DispatcherServlet` 的创建方法、作用

## 03. Bean的生命周期

一个受 Spring 管理的 bean，生命周期主要阶段有

1. 创建：根据 bean 的构造方法或者工厂方法来创建 bean 实例对象；
2. 依赖注入：根据 @Autowired，@Value 或其它一些手段，为 bean 的成员变量填充值、建立关系
3. 初始化：回调各种 Aware 接口，调用对象的各种初始化方法
4. 销毁：在容器关闭时，会销毁所有单例对象（即调用它们的销毁方法），prototype 对象也能够销毁，不过需要容器这边主动调用

> 一些资料会提到，生命周期中还有一类 bean 后处理器：BeanPostProcessor，会在 bean 的初始化的前后，提供一些扩展逻辑。但这种说法是不完整的，见下方内容；

### 3.1 Bean的生命周期

准备代码：

```java
package cn.xyc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class Class03Application {

    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(Class03Application.class, args);
        // 关闭容器
        context.close();
    }
}

```

准备一个放入容器中的类：

```java
package cn.xyc;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Slf4j
@Component
public class LifeCycleBean {

    public LifeCycleBean() {
        log.debug("构造函数");
    }

    @Autowired
    public void autowire(@Value("${JAVA_HOME}") String home) {
        log.debug("依赖注入: {}", home);
    }

    @PostConstruct
    public void init() {
        log.debug("初始化");
    }

    @PreDestroy
    public void destroy() {
        log.debug("销毁");
    }
}
```

验证执行顺序：

```java
[DEBUG] 16:38:04.322 [main] cn.xyc.LifeCycleBean                - 构造函数 
[DEBUG] 16:38:04.327 [main] cn.xyc.LifeCycleBean                - 依赖注入: C:\Software\Java\jdk1.8.0_241 
[DEBUG] 16:38:04.328 [main] cn.xyc.LifeCycleBean                - 初始化 
[DEBUG] 16:38:04.629 [main] cn.xyc.LifeCycleBean                - 销毁
```

bean的后处理器：

```java
package cn.xyc;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {

    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")){
            log.debug("<<<<<< 销毁之前执行, 如 @PreDestroy");
        }
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")){
            log.debug("<<<<<< 实例化之前执行, 这里返回的对象会替换掉原本的 bean");
        }
        // 返回不为null，会替换原来的bean
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.debug("<<<<<< 实例化之后执行, 这里如果返回 false 会跳过依赖注入阶段");
            // return false;
        }
        // 返回true，会继续执行后续依赖注入
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.debug("<<<<<< 依赖注入阶段执行, 如 @Autowired、@Value、@Resource");
        }
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.debug("<<<<<< 初始化之前执行, 这里返回的对象会替换掉原本的 bean, 如 @PostConstruct、@ConfigurationProperties");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.debug("<<<<<< 初始化之后执行, 这里返回的对象会替换掉原本的 bean, 如代理增强");
        }
        return bean;
    }
}
```

> 其中：`InstantiationAwareBeanPostProcessor`，`DestructionAwareBeanPostProcessor`，都继承自：`BeanPostProcessor`

运行main方法，输出如下：

```java
[DEBUG] 16:49:42.828 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 实例化之前执行, 这里返回的对象会替换掉原本的 bean 
[DEBUG] 16:49:42.829 [main] cn.xyc.LifeCycleBean                - 构造函数 
[DEBUG] 16:49:42.831 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 实例化之后执行, 这里如果返回 false 会跳过依赖注入阶段 
[DEBUG] 16:49:42.831 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 依赖注入阶段执行, 如 @Autowired、@Value、@Resource 
[DEBUG] 16:49:42.833 [main] cn.xyc.LifeCycleBean                - 依赖注入: C:\Software\Java\jdk1.8.0_241 
[DEBUG] 16:49:42.834 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 初始化之前执行, 这里返回的对象会替换掉原本的 bean, 如 @PostConstruct、@ConfigurationProperties 
[DEBUG] 16:49:42.834 [main] cn.xyc.LifeCycleBean                - 初始化 
[DEBUG] 16:49:42.834 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 初始化之后执行, 这里返回的对象会替换掉原本的 bean, 如代理增强 
[DEBUG] 16:49:43.075 [main] cn.xyc.MyBeanPostProcessor          - <<<<<< 销毁之前执行, 如 @PreDestroy 
[DEBUG] 16:49:43.076 [main] cn.xyc.LifeCycleBean                - 销毁 
```

### 3.2 模版设计模式

**提高现有代码的扩展能力。**

初始实现：实现Bean工厂

```java
package cn.xyc;

public class TestMethodTemplate {

    public static void main(String[] args) {
        MyBeanFactory beanFactory = new MyBeanFactory();
        beanFactory.getBean();
    }

    // 模板方法  Template Method Pattern
    static class MyBeanFactory {
        public Object getBean() {
            Object bean = new Object();
            System.out.println("构造 " + bean);
            System.out.println("依赖注入 " + bean); // @Autowired, @Resource
            System.out.println("初始化 " + bean);
            return bean;
        }
    }
}
```

问题：扩展性比较弱；优化如下，先来实现一个接口：

```java
static interface BeanPostProcessor {
    // 对依赖注入阶段的扩展
    public void inject(Object bean); 
}
```

修改原先代码：对固定不变的内容写在方法中（**静态部分**），对有变化的内容改成接口（**动态部分**）：

```java
    // 模板方法  Template Method Pattern
    static class MyBeanFactory {
        public Object getBean() {
            Object bean = new Object();
            System.out.println("构造 " + bean);
            System.out.println("依赖注入 " + bean); // @Autowired, @Resource
            for (BeanPostProcessor processor : processors) {
                processor.inject(bean);
            }
            System.out.println("初始化 " + bean);
            return bean;
        }

        private List<BeanPostProcessor> processors = new ArrayList<>();

        public void addBeanPostProcessor(BeanPostProcessor processor) {
            processors.add(processor);
        }
    }
```

修改main方法：

```java
public static void main(String[] args) {
    MyBeanFactory beanFactory = new MyBeanFactory();
    beanFactory.addBeanPostProcessor(processer -> System.out.println("解析 @Autowired"));
    beanFactory.addBeanPostProcessor(processer -> System.out.println("解析 @Resource"));
    beanFactory.getBean();
}
```

执行输出如下：

```java
构造 java.lang.Object@4783da3f
依赖注入 java.lang.Object@4783da3f
解析 @Autowired
解析 @Resource
初始化 java.lang.Object@4783da3f
```

此时代码的扩展性就较强，当需要加扩展功能时，`getBean`方法无需改动。

## 04. Bean后处理器

### 4.1 Bean后处理器作用

为Bean生命周期各个阶段提供扩展

测试代码构建：

```java
package cn.xyc;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.support.GenericApplicationContext;

public class Class04Application {

    public static void main(String[] args) {

        // GenericApplicationContext 是一个【干净】的容器
        GenericApplicationContext context = new GenericApplicationContext();

        // 用原始方法注册三个 bean
        context.registerBean("bean1", Bean1.class);
        context.registerBean("bean2", Bean2.class);
        context.registerBean("bean3", Bean3.class);

        // 初始化容器
        context.refresh(); // 执行beanFactory后处理器, 添加bean后处理器, 初始化所有单例

        // 销毁容器
        context.close();
    }
}

// Bean1.class，这里加了很多的打印信息
package cn.xyc;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.annotation.Resource;

@Slf4j
public class Bean1 {

    private Bean2 bean2;

    @Autowired
    public void setBean2(Bean2 bean2) {
        log.debug("@Autowired 生效: {}", bean2);
        this.bean2 = bean2;
    }

    @Autowired
    private Bean3 bean3;

    @Resource
    public void setBean3(Bean3 bean3) {
        log.debug("@Resource 生效: {}", bean3);
        this.bean3 = bean3;
    }

    private String home;

    @Autowired
    public void setHome(@Value("${JAVA_HOME}") String home) {
        log.debug("@Value 生效: {}", home);
        this.home = home;
    }

    @PostConstruct
    public void init() {
        log.debug("@PostConstruct 生效");
    }

    @PreDestroy
    public void destroy() {
        log.debug("@PreDestroy 生效");
    }

    @Override
    public String toString() {
        return "Bean1{" +
               "bean2=" + bean2 +
               ", bean3=" + bean3 +
               ", home='" + home + '\'' +
               '}';
    }
}

// Bean2.class
package cn.xyc;

public class Bean2 {
}

// Bean3.class
package cn.xyc;

public class Bean3 {
}
```

执行main方法后，并无Bean被注入的输出内容；继续往context中加入内容：

```java
context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
// 在依赖注入阶段解析@Autowired @Value
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
```

执行main方法，输出如下：

```java
[DEBUG] 18:00:40.209 [main] cn.xyc.Bean1     - @Autowired 生效: cn.xyc.Bean2@7fe8ea47 
[DEBUG] 18:00:40.221 [main] cn.xyc.Bean1     - @Value 生效: C:\Software\Java\jdk1.8.0_241 
```

继续添加后处理器：

```java
// 在依赖注入前解析@Resource 初始化前解析@PostConstruct 销毁前解析@PreDestroy
context.registerBean(CommonAnnotationBeanPostProcessor.class);
```

执行main方法，输出如下：

```java
[DEBUG] 18:01:05.439 [main] cn.xyc.Bean1       - @Resource 生效: cn.xyc.Bean3@3b2c72c2 
[DEBUG] 18:01:05.446 [main] cn.xyc.Bean1       - @Autowired 生效: cn.xyc.Bean2@145eaa29 
[DEBUG] 18:01:05.455 [main] cn.xyc.Bean1       - @Value 生效: C:\Software\Java\jdk1.8.0_241 
[DEBUG] 18:01:05.455 [main] cn.xyc.Bean1       - @PostConstruct 生效 
[DEBUG] 18:01:05.460 [main] cn.xyc.Bean1       - @PreDestroy 生效 
```

> 看下上面的输出顺序：`@Resource` 先生效再生效 `@Autowired`，因为这里`CommonAnnotationBeanPostProcessor` 排在 `AutowiredAnnotationBeanPostProcessor` 前面

再看下 SpringBoot 中的后处理器：添加`Bean4.class`

```java
/*
    java.home=
    java.version=
 */
@Data
@ConfigurationProperties(prefix = "java")  // 前缀匹配
public class Bean4 {
    // java.home
    private String home;
	// java.version
    private String version;
}
```

让容器中放入 `Bean4.class`，放入后获取下是否注入：

```java
context.registerBean("bean4", Bean4.class);
// ...
System.out.println(context.getBean(Bean1.class));
// 输出如下：Bean4(home=null, version=null)
```

加上后处理器，注册，再次测试，完整代码如下：

```java
// GenericApplicationContext 是一个【干净】的容器
GenericApplicationContext context = new GenericApplicationContext();

// 用原始方法注册三个 bean
context.registerBean("bean1", Bean1.class);
context.registerBean("bean2", Bean2.class);
context.registerBean("bean3", Bean3.class);
context.registerBean("bean4", Bean4.class);

context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
// 在依赖注入阶段解析@Autowired @Value
context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
// 在依赖注入前解析@Resource 初始化前解析@PostConstruct 销毁前解析@PreDestroy
context.registerBean(CommonAnnotationBeanPostProcessor.class);
// Springboot的一个后处理器，使得注解@ConfigurationProperties注解起作用，在初始化前解析
ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());

// 初始化容器
context.refresh(); // 执行beanFactory后处理器, 添加bean后处理器, 初始化所有单例
// 判断Springboot的后处理器是否生效
System.out.println(context.getBean(Bean4.class));
// 输出如下：Bean4(home=C:\Software\Java\jdk1.8.0_241\jre, version=1.8.0_241)

// 销毁容器
context.close();
```

### 4.2 常见的后处理器



