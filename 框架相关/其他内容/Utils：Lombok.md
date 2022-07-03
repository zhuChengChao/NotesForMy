# Utils：Lombok小记

> 简单的记录一下工作中较为常用的Lombok包，及其中所常用的一些注解。
>
> > 最近加班是真的加麻了:sob:...周末就直接开躺，确实是好久都没学习没沉淀了，之后还是得每周抽空来学点东西沉淀一下:sweat_smile:

日常开发中，Lombok已经成为了不可获取的一个工具包，很多繁琐的get/set/构造函数，只需要一个注解就可以轻松解决，使整个类看上去仅有其成员属性，看上去十分清晰。

```xml
<!--lomboky依赖-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
    <scope>provided</scope>  <!--只在编译阶段生效，不需要打入包中-->
     <!-- Lombok在编译期将带Lombok注解的Java文件正确编译为完整的Class文件。-->
</dependency>
```

## 0. 概述

> 使用lombok需要安装IDEA的插件，不然代码会爆红，不过最新的IDEA默认已经集成了这个插件。
>
> 官网地址：https://projectlombok.org/

![Lombok注解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261426789.png)

**常用注解：**

**@Getter/@Setter：**

* 注解在类上，给类中所有的字段添加上getter/setter方法；
* 注解在类的字段上，仅对该字段自动生成getter/setter方法；

**@xxxConstructor：**

* **@NoArgsConstructor**：注解在类上，生成无参的构造方法；
* **@RequiredArgsConstructor**：注解在类上，为类中需要特殊处理的字段生成构造方法，如final和被@NonNull注解的字段；
* **@AllArgsConstructor**：注解在类上，生成包含类中所有字段的构造方法。

**@ToString**：注解在类上，给类自动添加上toString方法，还可以通过of属性限定显示某些字段，通过exclude属性排除某些字段。

**@Data**：作用于类上，是注解 @ToString & @EqualsAndHashCode & @Getter & @Setter & @RequiredArgsConstructor的集合；

**@Builder：**作用于类上，将类转变为建造者模式；

**@Log：**作用于类上，生成日志变量。针对不同的日志实现产品，有不同的注解，@Log & @Slf4j & @Log4j & @Log4j2 & @XSlf4j & @CommonsLog

**@NonNull**：主要作用于成员变量和参数中，标识不能为空，否则抛出空指针异常。

## 2. 常用注解

[Lombok插件使用详解及原理 ](https://www.cnblogs.com/jing99/p/13785684.html)

### 2.1 变量相关注解

* **val & var**：自动推断类型；
* **NonNul：**
  * 在方法参数上：Lombok将在方法/构造函数体的开头插入一个空检查，并将参数的名称作为消息引发一个NullPointerException；
  * 在字段上：任何为该字段赋值的生成方法也将生成这些空检查。

### 2.2 实体类相关

**@Getter & @Setter**：取代实体类中的get和set方法，可以在任何字段上使用@Getter或@Setter，lombok会自动生成默认的getter / setter；如果在类上使用，则所有字段生成getter / setter；

> 关于方法命名：
>
> * Getter方法：
>
>   * 普通属性：默认情况下方法名为**get+字段名**，字段名以驼峰连接；
>
>   * boolean类型：**is+字段名**，字段名以驼峰连接，**但Boolean类型是get开头**；
>
>     > 不建议使用is开头的字段命名，会产生混淆：
>
> * Setter方法：
>
>   * Setter不受boolean影响，直接**set+字段名**，字段名以驼峰连接

**@xxxConstructor：**

* **@NoArgsConstructor**：注解在类上，生成无参的构造方法；
* **@RequiredArgsConstructor**：注解在类上，为类中需要特殊处理的字段生成构造方法，如final和被@NonNull注解的字段，如果指定staticName = "of"参数，还会生成一个返回类对象的静态工厂方法；
* **@AllArgsConstructor**：注解在类上，生成包含类中所有字段的构造方法。

**@ToString**：注解在类上，给类自动添加上toString方法，还可以通过of属性限定显示某些字段，通过exclude属性排除某些字段。

**@EqualsAndHashCode**：作用于类，覆盖默认的equals和hashCode，在默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals/canEqual和hasCode，也能通过exclude注解来排除一些属性。

**@Data**：作用于类上，是注解 @ToString & @EqualsAndHashCode & @Getter & @Setter & @RequiredArgsConstructor的集合；

> 通常 @Data 会加在一个值可以被更新的对象上，像是日常使用的 DTO 们，或是 JPA 里的 Entity 们，就很适合加上 @Data 注解，也就是 **@Data for mutable class**。

**@Builder**：通过内部类和一个全参构造器来实现构造者模式；

* 注意：虽然只要加上 @Builder 注解，我们就能够用流式写法快速设定对象的值，但是 setter 还是必须要写不能省略的，因为 Spring 或是其他框架有很多地方都会用到对象的 getter/setter 对他们取值/赋值，所以通常是 @Data 和 @Builder 会一起用在同个类上，既方便我们流式写代码，也方便框架做事；

* **由于Builder会生成一个全参构造器**，导致默认的无参构造器失效，所以类采用@Builder注解后无法new出来。完美的避免方式：加上@AllArgsConstructor、@NoArgsConstructor后就可以同时使用new和构造者方式实例化对象了。

  > ```java
  > // 类定义
  > @Data
  > @Builder
  > public class BuilderDemo {
  > 
  >     private String name;
  >     private Integer age;
  >     private Innerclass innerclass;
  > 
  >     @Data
  >     @Builder
  >     public static class Innerclass {
  >         private String field;
  >     }
  > 
  > }
  > 
  > // 使用
  > @Test
  > public void test(){
  > 
  >     BuilderDemo.Innerclass innerclass =
  >         BuilderDemo.Innerclass.builder().field("filed").build();
  >     BuilderDemo builderDemo =
  >         BuilderDemo.builder()
  >         .age(20).name("zhangsan")
  >         .innerclass(innerclass).build();
  > }
  > ```
  >
  > 

**@Getter(lazy=true)**：可以替代经典的Double Check Lock样板代码

> @Accessors：使用在类或字段上，目的是修改getter和setter方法的内容
>
> @Value：也是整合包，但是他会把所有的变量都设成 final 的，其他的就跟 @Data 一样，等于同时加了以下注解：@ToString & @EqualsAndHashCode & @AllArgsConstructor & @FieldDefaults & @Getter，所有字段由private和final修饰，不会产生setter方法。类本身也是由final修饰。

### 2.3 工具类注解

**@Cleanup**：自动资源管理，自动调用关闭资源方法，默认是调用close()方法，在finally块中，只有在给定资源不是null的情况下才会调用清理方法。

**@日志记录：**作用于类上，生成日志变量。针对不同的日志实现产品，有不同的注解

* 作用于类上会自动生成该类的 log 静态常量，要打日志就可以直接打，不用再手动 new log 静态常量了；

* lombok 提供@Log、@Slf4j、@Log4j、@Log4j2、@XSlf4j、@CommonsLog日志框架的变种注解，他们都是帮我们创建一个静态常量 log，只是使用的库不一样而已；

* SpringBoot默认支持的就是 **slf4j + logback** 的日志框架，所以也不用再多做啥设定，直接就可以用在 SpringBoot project 上；因此 log 系列注解最常用的就是 @Slf4j

  > ```java
  > @Log4j //对应的log语句如下
  > private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);
  > 
  > // 在添加了该注解后，直接在代码里可以使用log.info/warn/error(xxx)来输出日志信息
  > ```

> **@SneakyThrows**：和抛出异常相关；
>
> **@Synchronized**：@Synchronized是synchronized方法修饰符的更安全的变体。

## 3. 实现原理

Lombok使用的过程中，只需要添加相应的注解，无需再为此写任何代码；但是自动生成的代码到底是如何产生的呢？

核心之处就是对于注解的解析上。JDK5引入了注解的同时，也提供了两种解析方式。

1. **运行时解析**：运行时能够解析的注解，必须将@Retention设置为RUNTIME，这样就可以通过反射拿到该注解。java.lang.reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。

2. 编译时解析，编译时解析有两种机制，分别简单描述下：

   1. Annotation Processing Tool：apt自JDK5产生，JDK7已标记为过期，不推荐使用，JDK8中已彻底删除，自JDK6开始，可以使用Pluggable Annotation Processing API来替换它，apt被替换主要有2点原因：api都在com.sun.mirror非标准包下；没有集成到javac中，需要额外运行。

   2. Pluggable Annotation Processing API：**JSR 269**自JDK6加入，作为apt的替代方案，它解决了apt的两个问题，javac在执行的时候会调用实现了该API的程序，这样我们就可以对编译器做一些增强，这时javac执行的过程如下： 

      ![javac执行过程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261556538.png)

自从Java 6起，**javac就支持“JSR 269 Pluggable Annotation Processing API”**规范，只要程序实现了该API，就能在javac运行的时候得到调用。

Lombok就是一个实现了"JSR 269 API"的程序；在使用javac的过程中，它产生作用的具体流程如下：

1. javac对源代码进行分析，生成一棵抽象语法树(AST)

2. javac编译过程中调用实现了JSR 269的Lombok程序

3. 此时Lombok就对第一步骤得到的AST进行处理，找到Lombok注解所在类对应的语法树(AST)，然后修改该语法树(AST)，增加Lombok注解定义的相应树节点

4. javac使用修改后的抽象语法树(AST)生成字节码文件

## 4. 参考文件

https://projectlombok.org/features/all

https://www.jianshu.com/p/2543c71a8e45

https://www.cnblogs.com/jing99/p/13785684.html