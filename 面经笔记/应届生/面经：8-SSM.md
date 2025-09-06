# 面经：8-SSM

> 个人整理 :muscle: 个人专用 :muscle: 暑期实习 :muscle: 秋招 :muscle: 后端开发 :muscle: 八股文 :no_mouth:

## Spring

### 使用 Spring 框架的好处

1. 轻量：Spring 是轻量的，基本的版本大约 2MB；

2. **控制反转(IOC)**：Spring 通过控制反转实现了松散耦合，通过 Spring 提供的 IOC 容器，可以将对象间的依赖关系交由 Spring 进行控制，避免硬编码造成的过度程序耦合；

3. **面向切面的编程(AOP)**：Spring 支持面向切面的编程，并且把应用业务逻辑和系统服务分开；

4. **容器**：Spring 包含并管理应用中对象的生命周期和配置；

5. MVC 框架：Spring 的 Web 框架是个精心设计的框架，是 Web 框架的一个很好的替代品；

   > 更有：Spring 可以降低各种框架的使用难度，提供了对各种优秀框架（ Struts、 Hibernate、 Hessian、 Quartz等）的直接支持。  

6. **事务管理**：Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全局事务（JTA， Java Transaction Manager）；

   > 更有：Spring提供了声明式事务的支持，直接可以通过声明的方式进行事物管理，提高开发效率和质量。  

7. 异常处理：Spring 提供方便的 API 把具体技术相关的异常（比如由 JDBC，Hibernate or JDO 抛出的）转化为一致的 unchecked 异常。

8. 方便程序的测试：可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情  

### 什么是 AOP :exclamation::exclamation::exclamation:

AOP（Aspect-Oriented Programming，面向切面编程），可以说是 OOP（Object-Oriented Programing，面向对象编程）的补充和完善。

> OOP 引入封装、继承和多态等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。

**当我们需要为分散的对象引入公共行为的时候，OOP 则显得无能为力。也就是说，OOP 允许你定义从上到下的关系，但并不适合定义从左到右的关系**。

> 例如日志功能：日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如：安全性、异常处理和透明的持续性也是如此。

**这种散布在各处的无关的代码被称为横切（cross-cutting）代码，在 OOP 设计中，它导致了大量代码的重复，而不利于各个模块的重用。**

而 AOP 技术则恰恰相反，它**利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。**

所谓“方面”，简单地说，就是**将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性**。

AOP 代表的是一个**横向的关系**，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向切面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。

使用“横切”技术，AOP 把软件系统分为两个部分：**核心关注点和横切关注点**。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，它们经常发生在核心关注点的多处，而各处都基本相似。比如：权限认证、日志、事务处理。**AOP 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。**

### AOP 的代理有哪几种方式 :exclamation::exclamation:

AOP 思想的实现一般都是基于**代理模式** ，AOP的代理主要分为**静态代理和动态代理**；

**静态代理**的代表为 AspectJ，其是静态代理的增强；所谓静态代理，就是AOP框架会在**编译阶段**生成AOP代理类，因此也称为**编译时增强**，他会在编译阶段将 AspectJ(切面) 织入到 Java 字节码中，运行的时候就是增强之后的 AOP 对象。

**Spring AOP使用的是动态代理**，所谓的动态代理就是说 AOP 框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个 AOP 对象，这个 AOP 对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

在 Java 中一般采用 JDK 动态代理模式，但是我们都知道，**JDK 动态代理模式要求被代理的对象必须实现一个接口**。因此，Spring AOP 会按照下面两种情况进行切换：

1. 如果目标对象的实现类实现了接口，Spring AOP 将会采用 **JDK 动态代理**来生成 AOP 代理类；
2. 如果目标对象的实现类没有实现接口，Spring AOP 将会采用 **CGLIB** 来生成 AOP 代理类。

不过这个选择过程对开发者完全透明、开发者也无需关心。

静态代理与动态代理**区别在于生成AOP代理对象的时机不同**，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

### 实现 JDK 动态代理 :exclamation:

**Java领域中，常用的动态代理实现方式有两种**

* JDK 原生动态代理：利用**JDK反射机制**生成代理;

  > 动态代理类和被代理类必须继承同一个接口：之所以实现相同的接口，目的是为了**尽可能保证代理对象内部结构和目标对象一致**，这样我们对代理对象的操作最终都可以转移到目标对象身上，代理对象只需专注增强代码的编写。

* 另外一种是使用**CGLIB（Code Generation Library）代理**

  > 由于 JDK动态代理存在限制：只能为接口创建代理实例，而**对于没有实现接口的类，就可以通过 CGLIB 来实现动态代理实例**
  >
  > **首先需要引入CGLIB相关Jar包**：其实针对类实现代理，主要**对指定的类生成一个子类**，覆盖其中的方法，添加额外功能，因为是继承，所以该类方法不能用 final 来声明

**JDK原生动态代理：动态代理类和被代理类必须继承同一个接口。动态代理只能对接口中声明的方法进行代理。**

1. **java.lang.reflect 包中的 Proxy 类中的 newProxyInstance 方法：**

   ```java
   // 来创建动态代理的 class 对象和实例
   public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
   ```

   其中的参数含义如下：

   1. loader：被代理的类的类加载器；
   2. interfaces：被代理类的接口数组；
   3. invocationHandler：一个 InvocationHandler 对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个 InvocationHandler 对象上

   该方法会返回一个被修改过的类的实例，从而可以自由的调用该实例的方法。

   通过 `Proxy.newProxyInstance` 创建的代理对象是在 JVM 运行时动态生成的一个对象，它并不是 InvocationHandler 类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象。

2. **java.lang.reflect 包中的 InvocationHandler 接口：**

   ```java
   public interface InvocationHandler { 
   	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable; 
   }
   ```

   对于被代理的类的操作都会由该接口中的 invoke 方法实现，其中的参数的含义分别是：

   1. proxy：被代理的类的实例；
   2. method：调用被代理的类的方法；
   3. args：该方法需要的参数。

   每一个动态代理实例都有一个关联的 InvocationHandler。通过代理实例调用方法，方法调用请求会被转发给InvocationHandler的invoke方法，而在调用invoke方法的前后去做一些额外的事情，从而实现动态代理。

**CGLIB（Code Generation Library）代理**

1. 导入 jar 包：`asm-5.2.jar`，`cglib-3.2.5.jar`
2. CGLIB 主要是**针对指定的类生成一个子类**，从而实现增强操作

> 关于 CGLIB 可见 [3-Java基础：实现动态代理]()

### AOP 的基本概念：切面、连接点、切入点等

1. 切面（Aspect）：被抽取的公共模块，可能会横切多个对象；是切入点和通知(引介)的结合。

   > 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

2. **连接点（Joinpoint）**：程序执行过程中的某一行为，即**业务层接口中的方法**。

3. 通知（Advice）：“切面”对于某个“连接点”所产生的动作。

   > 通知的类型： 前置通知,后置通知,异常通知,最终通知,环绕通知 
   >
   > 许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链

4. Introduction(引介):  引介是一种特殊的通知，在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些 Method 或 Field。  

5. **切入点（Pointcut）**：所谓切入点是指我们要对**哪些连接点进行拦截**的定义；通过切入点表达式，指定拦截的方法，比如指定拦截`add*/search*/...`。

6. 目标对象（Target Object）：被一个或者多个切面所通知的对象、被通知的对象(Advise Object) 、被代理的对象（Proxied Object）。

7. 织入（Weaving）：把增强应用到目标对象来创建新的代理对象的过程

   > Spring采用动态代理织入，而 AspectJ 采用编译器织入和类装载期织入

8. AOP 代理（AOP Proxy）：在 Spring AOP 中有两种代理方式，JDK 动态代理和 CGLIB 代理。

**切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键**，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的环绕(Around)通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

### 通知类型（Advice）型有哪些

> 顺序好像有问题的...待完善

1. **前置通知**（Before advice）：在某连接点（JoinPoint）之前执行的通知，但这个通知不能阻止连接点前的执行。

   > ApplicationContext 中在 `<aop:aspect>` 里面使用 `<aop:before>` 元素进行声明；

2. **后置通知**（After advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

   > ApplicationContext 中在 `<aop:aspect>` 里面使用 `<aop:after>` 元素进行声明。

3. **返回后通知**（After return advice ：在某连接点正常完成后执行的通知，不包括抛出异常的情况。

   > ApplicationContext 中在 `<aop:aspect>` 里面使用 `<after-returning>` 元素进行声明。

4. **环绕通知**（Around advice）：包围一个连接点的通知，类似 Web 中 Servlet 规范中的 Filter 的 doFilter 方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行。

   > ApplicationContext 中在 `<aop:aspect>` 里面使用 `<aop:around>` 元素进行声明。

5. **抛出异常后通知**（After throwing advice）：在方法抛出异常退出时执行的通知。

   > ApplicationContext 中在 `<aop:aspect>` 里面使用 `<aop:after-throwing>` 元素进行声明。

> 例如：
>
> ```xml
> <!-- 通知类通过bean进行管理 -->
> <bean id="txManager" class="com.itheima.utils.TransactionManager">
>        <property name="dbAssit" ref="dbAssit"></property>
> </bean>
> 
> <!-- 使用aop:config声明aop配置 -->
> <aop:config>
>        <!-- 配置切入点表达式 -->
>        <aop:pointcut expression="execution(* com.itheima.service.impl.*.*(..))" id="pt1"/>
>        <!-- 配置切面  ref:引用配置好的通知类bean的id -->
>        <aop:aspect id="txAdvice" ref="txManager">
>            <!-- 配置对应的通知类型：配置环绕通知 -->
>            <aop:around method="transactionAround" pointcut-ref="pt1"/>
>        </aop:aspect>
> </aop:config>
> ```

### 谈谈你对 IOC 的理解？:exclamation::exclamation::exclamation:

IOC 是 Inversion of Control 的缩写，多数书籍翻译成“控制反转”。

**IOC 理论提出的观点大体是这样的：借助于“第三方”来实现具有依赖关系的对象之间的解耦**。

> 如下图：
>
> ![ioc1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811855.jpg)
>
> 由于引进了中间位置的“第三方”，也就是 **IOC 容器**，使得 A、B、C、D 这 4 个对象没有了耦合关系，齿轮之间的传动全部依靠“第三方”了，全部对象的控制权全部上缴给“第三方”IOC 容器，所以，IOC 容器成了整个系统的关键核心，它起到了一种类似“粘合剂”的作用，把系统中的所有对象粘合在一起发挥作用，如果没有这个“粘合剂”，对象与对象之间会彼此失去联系，这就是有人把 IOC 容器比喻成“粘合剂”的由来。　　
>
> 把上图中间的 IOC 容器拿掉，然后再来看看这套系统：
>
> ![ioc2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811224.jpg)
>
> 现在看到的画面，就是我们要实现整个系统所需要完成的全部内容。这时候，A、B、C、D这 4 个对象之间已经没有了耦合关系，彼此毫无联系，这样的话，**当你在实现 A 的时候，根本无须再去考虑 B、C 和 D了**，对象之间的依赖关系已经降低到了最低程度。所以，如果真能实现 IOC 容器，对于系统开发而言，这将是一件多么美好的事情，参与开发的每一成员只要实现自己的类就可以了，跟别人没有任何关系！

我们再来看看，控制反转(IOC)到底为什么要起这么个名字？**我们来对比一下**：

* 软件系统在没有引入 IOC 容器之前，对象 A 依赖于对象 B，那么对象 A 在初始化或者运行到某一点的时候，自己**必须主动去创建对象 B 或者使用已经创建的对象 B**。无论是创建还是使用对象 B，控制权都在自己手上。

* 软件系统在引入 IOC 容器之后，这种情形就完全改变了，由于 IOC 容器的加入，对象 A 与对象 B 之间失去了直接联系，所以，当对象 A 运行到需要对象 B 的时候，**IOC 容器会主动创建一个对象 B 注入到对象 A 需要的地方**。

通过前后的对比，我们不难看出来：对象 A 获得依赖对象 B 的过程，**由主动行为变为了被动行为，控制权颠倒过来了，这就是“控制反转”这个名称的由来。**

**总结：**

**控制反转把创建对象的权利交给Spring框架**，是框架的重要特征，其消减了程序之间的耦关系；

最直观的表现就是：IOC让对象的创建**不用去new了**，可以由spring自动生产，使用java的反射机制，根据配置文件在运行时**动态的去创建对象以及管理对象**，并调用对象的方法。

控制反转中包括了：**依赖注入（Dependency Injection，DI）**和**依赖查找（Dependency Lookup）**。

### BeanFactory 和 ApplicationContext 的区别

> spring 中工厂的类结构图 
>
> ![spring 中工厂的类结构图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811837.PNG)

BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。

而 BeanFactory 才是 Spring 容器中的顶层接口。ApplicationContext 是它的子接口。

**BeanFactory 和 ApplicationContext 的区别：**创建对象的时间点不一样

- ApplicationContext：只要一读取配置文件，默认情况下就会创建对象。
- BeanFactory：什么使用什么时候创建对象，`getBean()`

### ApplicationContext 通常的实现

1. FileSystemXmlApplicationContext：此容器从一个 XML 文件中加载 beans 的定义，其是**从磁盘路径上加载配置文件**，配置文件可以在磁盘的任意位置；
2. ClassPathXmlApplicationContext：此容器也从一个 XML 文件中加载 beans 的定义，其是从**类的根路径下加载配置文件**；
3. AnnotationConfigApplicationContext ：当我们使用**注解配置容器对象**时，需要使用此类来创建 spring 容器。它用来读取注解；
4. WebXmlApplicationContext：此容器加载一个 XML 文件，此文件定义了一个 **Web 应用**的所有 bean。

### Bean 的生命周期 :exclamation::exclamation::exclamation:

> 这部分内容真不太理解

> 在传统的 Java 应用中，bean 的生命周期很简单，使用 Java 关键字 new 进行 Bean 的实例化，然后该 Bean 就能够使用了。一旦 Bean 不再被使用，则由 Java 自动进行垃圾回收。
>
> :dog2: 也不是很简单的，涉及到垃圾回收等相关内容

相比之下，Spring 管理 Bean 的生命周期就复杂多了，正确理解 Bean 的生命周期非常重要，因为 Spring 对 Bean 的管理可扩展性非常强，下面展示了一个 Bean 的构造过程：

![Bean构造过程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811960.jpg)

1. **Spring 启动，查找并加载需要被 Spring 管理的 Bean，进行 Bean 的实例化；**

2. **Bean 实例化后，对 Bean 中的属性完成依赖注入；**

3. 如果 Bean 实现了 BeanNameAware 接口的话，Spring 将 Bean 的 Id 传递给 `setBeanName(String name)` 方法；

4. 如果 Bean 实现了 BeanFactoryAware 接口的话，Spring 将调用 `setBeanFactory()` 方法，将 BeanFactory 容器实例传入；

5. 如果 Bean 实现了 ApplicationContextAware 接口的话，Spring 将调用 Bean 的 `setApplicationContext()` 方法，将 Bean 所在应用上下文引用传入进来；

6. 如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用它们的 `postProcessBeforeInitialization()` 方法，在初始化之前对Bean进行增强；

7. 如果 Bean 实现了 InitializingBean 接口，Spring 将调用它们的 `afterPropertiesSet()` 方法。类似地，如果 Bean 在配置文件中使用 init-method 声明了初始化方法，该方法也会被调用；

8. 如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用它们的 `postProcessAfterInitialization()` 方法，在初始化之后对Bean进行增强；

   > BeanPostProcessor 接口的两个方法 `postProcessBeforeInitialization() & postProcessAfterInitialization()` 分别在Bean的初始化前后对Bean对象提供自己的实例化逻辑

9. 此时，**Bean 已经准备就绪**，可以被应用程序使用了。它们将一直驻留在应用上下文中，直到应用上下文被销毁；

10. 如果 Bean 实现了 DisposableBean 接口，Spring 将调用它的 `destory()` 接口方法，同样，如果 Bean 在配置文件中使用了 destory-method 声明销毁方法，该方法也会被调用。

> 更详细了解见：[Spring 了解Bean的一生(生命周期)](https://xulinjie.blog.csdn.net/article/details/80086950?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-15.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-15.control)

### Bean 的作用域 :exclamation::exclamation:

bean 标签中的 scope 属性，指定对象的作用范围  

1. **singleton** : 唯一 bean 实例，Spring 中的 bean 默认都是单例的；
2. **prototype** : 每次请求都会创建一个新的 bean 实例；
3. request：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效；
4. session : 同一个HTTP Session共享一个Bean，不同Session使用不同Bean，该 bean 仅在当前 HTTP session 内有效；

> global-session：全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring5 已经没有了。Portlet 是能够生成语义代码(例如：HTML)片段的小型 Java Web 插件。
>
> 它们基于 portlet 容器，可以像 servlet 一样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话。
>
> 如果没有 Portlet 环境那么  globalSession 相当于 session

### Spring 中的单例 Bean 的线程安全 :exclamation::exclamation:

**单例 bean 存在线程问题**，主要是因为：当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。常见的有两种解决办法：

1. 在 Bean 对象中尽量避免定义可变的成员变量（不太现实）
2. 在类中定义一个 ThreadLocal 成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### 实例化 Bean 的方式

方式1：使用**默认无参构造函数**

```xml
<!--在默认情况下：
	它会根据默认无参构造函数来创建类对象。如果 bean 中没有默认无参构造函数，将会创建失败。-->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"/>
```

方式2：**Spring管理静态工厂**：使用静态工厂的方法创建对象

```java
// 模拟一个静态工厂，创建业务层实现类
public class StaticFactory {
    public static IAccountService createAccountService(){
        return new AccountServiceImpl();
    }
}
```

```xml
<!-- 此种方式是:
	   使用 StaticFactory 类中的静态方法 createAccountService 创建对象，并存入 spring 容器
	   - id 属性：指定 bean 的 id，用于从容器中获取
	   - class 属性：指定静态工厂的全限定类名
	   - factory-method 属性：指定生产对象的静态方法
-->
<bean id="accountService"
      class="com.itheima.factory.StaticFactory"
      factory-method="createAccountService"></bean>
```

方式3：**Spring 管理实例工厂**——使用实例工厂的方法创建对象  

```java
/**
  * 模拟一个实例工厂，创建业务层实现类
  * 此工厂创建对象，必须现有工厂实例对象，再调用方法
  */
public class InstanceFactory {
    public IAccountService createAccountService(){
        return new AccountServiceImpl();
    }
}
```

```xml
<!-- 此种方式是：
        1. 先把工厂的创建交给 spring 来管理。
        2. 然后在使用工厂的 bean 来调用里面的方法
        	- factory-bean 属性：用于指定实例工厂 bean 的 id。
        	- factory-method 属性：用于指定实例工厂中创建对象的方法。
-->
<bean id="instancFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService"
      factory-bean="instancFactory"
      factory-method="createAccountService"></bean>
```

### 对 Spring 中的事物的理解 :exclamation:

**概述：**

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。**真正的数据库层的事务提交和回滚是通过 binlog 或者 redolog 实现的。**

事务是逻辑上的一组操作，要么都执行，要么都不执行。

**事务特性：**

* 原子性(Atomicity)：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
* 一致性(Consistency)：执行事务前后，数据保持一致；
* 隔离性(Isolation)：并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；
* 持久性(Durability)：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

**Spring 中事物的种类：**

spring支持**编程式事务管理**和**声明式事务管理**两种方式：

- 编程式事务管理使用 TransactionTemplate
- 声明式事务管理建立在 AOP 之上的。其本质是通过 AOP 功能，对方法前后进行拦截，**将事务处理的功能编织到拦截的方法中**，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件(xml)中做相关的事务规则声明或通过`@Transactional`注解的方式，便可以将事务规则应用到业务逻辑中。

**声明式事务管理要优于编程式事务管理**，这正是spring倡导的**非侵入式的开发方式**，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一**不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。**

**Spring 事务管理接口：**

1. `PlatformTransactionManager`：(平台)事务管理器，**提供了常用的操作事物的方法**
   
   1. 获取事物的状态信息：`TransactionStatus getTransaction(TransactionDefinition definition)`
   2. 提交事物：`void commit(TransactionStatus status)`
   3. 回滚事物：`void rollback(TransactionStatus status)`
   
2. `TransactionDefinition`：**事务定义信息**（事务隔离级别、传播行为、超时、只读、回滚规则）；

   > - 获取事物对象名称：`Sring getName()`
   > - 获取事物隔离级别：`int getIsolationLevel()`
   > - 获取事物传播行为：`int getPropagationBehavior()`
   > - 获取事物超时时间：`int getTimeout()`
   > - 获取事物是否只读：`boolean isReadOnly()`

3. `TransactionStatus`：**事务运行状态**；

所谓事务管理，其实就是“**按照给定的事务规则来执行提交或者回滚操作**”。

### Spring 中的事务隔离级别

TransactionDefinition 接口中定义了五个表示隔离级别的常量：

`TransactionDefinition.ISOLATION_DEFAULT`：使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ（可重复读） 隔离级别；Oracle 默认采用的 READ_COMMITTED（读已提交） 隔离级别；

`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读；

`TransactionDefinition.ISOLATION_READ_COMMITTED`：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生；

`TransactionDefinition.ISOLATION_REPEATABLE_READ`：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生；

`TransactionDefinition.ISOLATION_SERIALIZABLE`：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### Spring 中的事物传播行为

事务传播行为是为了**解决业务层方法之间互相调用的事务问题**。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在 TransactionDefinition 定义中包括了如下几个表示传播行为的常量：

**支持当前事务的情况：**

* `TransactionDefinition.PROPAGATION_REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务；
* `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行；
* `TransactionDefinition.PROPAGATION_MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

**不支持当前事务的情况：**

* `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起；
* `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

* `TransactionDefinition.PROPAGATION_NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 `TransactionDefinition.PROPAGATION_REQUIRED`。

### Spring 常用的注入方式 :exclamation:

**依赖注入： Dependency Injection**。 它是 spring 框架核心 IOC 的具体实现。  

我们的程序在编写时， 通过 控制反转 IOC， 把对象的创建交给了 spring，但是代码中不可能出现没有依赖的情况。**IOC 解耦只是降低他们的依赖关系，但不会消除**。 例如：我们的业务层仍会调用持久层的方法。

那这种业务层和持久层的依赖关系， 在使用 spring 之后， 就让 spring 来维护了。

简单的说，就是坐等框架把持久层对象传入业务层，而不用我们自己去获取。  

注入的方式：

1. **构造器依赖注入**：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

2. **Setter 方法注入**：Setter 方法注入是容器通过调用无参构造器或无参 static 工厂方法实例化 bean 之后，调用该 bean 的 Setter 方法，即实现了基于 Setter 的依赖注入。

   > 使用 p 名称空间注入数据（本质还是调用 Setter 方法）；

3. **基于注解的注入**：

   1. @Autowired：只能注入其他bean类型
   2. @Qualifier：在自动按照类型注入的基础上，再按照 Bean 的 id 注入；
   3. @Resource：直接按照Bean的id注入，只能注入其他bean类型；
   4. @Value：注入基本数据类型和String类型数据。

   > @Autowired 与 @Resource区别：
   >
   > - @Autowired 默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）；
   > - @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入；

**最好的解决方案是用构造器参数实现强制依赖，Setter 方法实现可选依赖。**

### Spring 框架中用到了的设计模式

> 这里得好好完善一下

1. **工厂设计模式** : Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象；
2. **代理设计模式** : Spring AOP 功能的实现；
3. **单例设计模式** : Spring 中的 Bean 默认都是单例的；
4. **模板方法模式** : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式；

> 1. 包装器设计模式：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源；
> 2. 观察者模式：Spring 事件驱动模型就是观察者模式很经典的一个应用；
> 3. 适配器模式：Spring AOP 的增强或通知(Advice)使用到了适配器模式、SpringMVC 中也是用到了适配器模式适配 Controller。

### Spring 框架中有哪些不同类型的事件

Spring 提供了以下5种标准的事件：

1. **上下文更新事件（ContextRefreshedEvent）**：在调用ConfigurableApplicationContext 接口中的`refresh()` 方法时被触发。
2. **上下文开始事件（ContextStartedEvent）**：当容器调用ConfigurableApplicationContext的`start()`方法开始/重新开始容器时触发该事件。
3. **上下文停止事件（ContextStoppedEvent）**：当容器调用ConfigurableApplicationContext的`stop()`方法停止容器时触发该事件。
4. **上下文关闭事件（ContextClosedEvent）**：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. **请求处理事件（RequestHandledEvent）**：在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个 ApplicationEvent 被发布以后，bean会自动被通知。

## SpringMVC

### 对 MVC 模式的理解 :exclamation::exclamation:

MVC 是 Model — View — Controler 的简称，它是一种架构模式，它**分离了表现与交互**。

它被分为三个核心部件：模型、视图、控制器。

![mvc](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811718.png)

**Model / 模型**：是程序的主体部分，主要包含业务数据和业务逻辑。在模型层，还会涉及到用户发布的服务，在服务中会根据不同的业务需求，更新业务模型中的数据。

**View / 视图**：是程序呈现给用户的部分，是用户和程序交互的接口，用户会根据具体的业务需求，在 View 视图层输入自己特定的业务数据，并通过界面的事件交互，将对应的输入参数提交给后台控制器进行处理。

**Controller / 控制器**：Controller 是用来处理用户输入数据，以及更新业务模型的部分。控制器中接收了用户与界面交互时传递过来的数据，并根据数据业务逻辑来执行服务的调用和更新业务模型的数据和状态。

### SpringMVC 的工作原理/执行流程 :exclamation::exclamation::exclamation:

**简单来说：客户端发送请求 -> 前端控制器 DispatcherServlet 接受客户端请求 -> 找到处理器映射 HandlerMapping 解析请求对应的 Handler -> HandlerAdapter 会根据 Handler 来调用真正的处理器 Controller 来处理请求，并处理相应的业务逻辑 -> 处理器返回一个模型视图 ModelAndView -> 视图解析器 ViewResolver 进行解析 -> 返回一个视图对象 -> 前端控制器 DispatcherServlet 渲染数据（Model）-> 将得到视图对象返回给用户**。

![spingmvc解析过程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811265.jpg)

上图用于辅助理解，面试时可用下列 8 步描述 SpringMVC 运行流程：

1. 用户向服务器发送请求，请求被 Spring 前端控制Servelt DispatcherServlet 捕获；

2. DispatcherServlet 对请求 URL 进行解析，得到请求资源标识符（URI）。然后根据该 URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以 HandlerExecutionChain 对象的形式返回；

3. DispatcherServlet 根据获得的 Handler，选择一个合适的HandlerAdapter；

   > 附注：如果成功获得 HandlerAdapter 后，此时将开始执行拦截器的 preHandler(...)方法

4. 提取 Request 中的模型数据，填充 Handler 入参，开始执行Handler（Controller)。

   > 在填充 Handler 的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：
   >
   > 1. HttpMessageConveter：将请求消息（如：Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息；
   >
   > 2. 数据转换：对请求消息进行数据转换。如：String 转换成 Integer、Double 等；
   >
   > 3. 数据格式化：对请求消息进行数据格式化。如：将字符串转换成格式化数字或格式化日期等；
   >4. 数据验证：验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中;
   
5. Handler 执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象；

6. 根据返回的 ModelAndView，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的 ViewResolver)返回给DispatcherServlet； 

7. ViewResolver 结合 Model 和 View，来渲染视图；

8. 将渲染结果返回给客户端。

### SpringMVC的优点

1. 可以支持各种视图技术，而不仅仅局限于 JSP；
2. 与 Spring 框架集成（如 IoC 容器、AOP 等）；
3. 清晰的角色分配：前端控制器(DispatcherServlet)，请求到处理器映射（HandlerMapping)，处理器适配器（HandlerAdapter)，视图解析器（ViewResolver）。

4. 支持各种请求资源的映射策略。

### SpringMVC 的核心组件 :exclamation:

**前端控制器 DispatcherServlet**

* 作用：Spring MVC 的入口函数。接收请求，响应结果，**相当于转发器，中央处理器**。有了 DispatcherServlet **减少了其它组件之间的耦合度**。用户请求到达前端控制器，它就相当于 MVC 模式中的 C，DispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet 的存在降低了组件之间的耦合性。

**处理器映射器 HandlerMapping**

* 作用：根据请求的 URL 查找 Handler。HandlerMapping 负责根据用户请求找到 Handler 即处理器 Controller，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**处理器适配器 HandlerAdapter**

* 作用：按照特定规则（HandlerAdapter 要求的规则）去执行 Handler。通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**视图解析器 View resolver**

* 作用：进行视图解析，根据逻辑视图名解析成真正的视图（View ）。View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。

  SpringMVC 框架提供了很多的 View 视图类型，包括：jstlView、freemarkerView、pdfView 等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**视图 View**

* View 是一个接口，实现类支持不同的 View 类型（JSP、freemarker、...）。

> **注意：**
>
> **处理器 Handler（也就是我们平常说的 Controller 控制器）**以及**视图层 View** 都是需要我们自己手动开发的。
>
> 其他的一些组件比如：前端控制器 DispatcherServlet、处理器映射器 HandlerMapping、处理器适配器 HandlerAdapter 等等都是框架提供给我们的，**不需要自己手动开发**。

### SpringMVC 常用的注解有哪些

1. @RequestMapping：用于处理请求 URL 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径；

2. @RequestBody：用于获取请求体内容。 

   1. 直接使用得到是 `key=value&key=value...` 结构的数据；
   2. 实现接收 HTTP 请求的 json 数据，将 json 转换为 Java 对象；

   > get 请求方式不适用

3. @ResponseBody：注解实现将 Controller 方法返回的对象，通过 HttpMessageConverter 接口转换为指定格式的数据如： json、xml 等，通过 Response 响应给客户端

> @PathVaribale、@RequestParam、@RequestHeader、@CookieValue、@ModelAttribute、@SessionAttribute...
>
> 具体见Spring5MVC-01/02

### @RequestMapping 的作用

RequestMapping 是一个用来**处理请求地址映射的注解**，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。RequestMapping 注解有六个属性，下面我们把它分成三类进行说明。

- **value、method：**
  1. value：指定请求的实际地址，指定的地址可以是 URI Template 模式；
  2. method：指定请求的 method 类型， GET、POST、PUT、DELETE 等；

- **consumes、produces：**
  1. consumes：指定处理请求的提交内容类型（Content-Type），例如 application/json、text/html；
  2. produces：指定返回的内容类型，仅当 request 请求头中的（Accept）类型中包含该指定类型才返回；

- **params、header：**
  1. params：指定 request 中必须包含某些参数值时，才让该方法处理。
  2. headers：指定 request 中必须包含某些指定的 header 值，才能让该方法处理请求。

### 如何解决 POST / GET 请求中文乱码问题

1. 解决 POST 请求乱码问题：在 web.xml 中配置一个 CharacterEncodingFilter 过滤器，设置成 utf-8；

   ```xml
   <!-- 配置 springMVC 编码过滤器 -->
   <filter>
       <filter-name>CharacterEncodingFilter</filter-name>
       <filter-class>
           org.springframework.web.filter.CharacterEncodingFilter
       </filter-class>
       <!-- 设置过滤器中的属性值 -->
       <init-param>
           <param-name>encoding</param-name>
           <param-value>UTF-8</param-value>
       </init-param>
       <!-- 启动过滤器 -->
       <init-param>
           <param-name>forceEncoding</param-name>
           <param-value>true</param-value>
       </init-param>
   </filter>
   <!-- 过滤所有请求 -->
   <filter-mapping>
       <filter-name>CharacterEncodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

   > 在 springmvc 的配置文件中可以配置，静态资源不过滤： 
   >
   > ```xml
   > <!-- location 表示路径， mapping 表示文件， **表示该目录下的文件以及子目录的文件 -->
   > <mvc:resources location="/css/" mapping="/css/**"/>
   > <mvc:resources location="/images/" mapping="/images/**"/>
   > <mvc:resources location="/scripts/" mapping="/javascript/**"/>
   > ```

2. GET 请求中文参数出现乱码解决方法有两个：

   （1）修改 tomcat 配置文件添加编码与工程编码一致，如下：

   ```xml
   <ConnectorURIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
   ```

   > 或者：修改 tomcat 的 server.xml 配置文件，如下： 
   >
   > ```xml
   > <Connector port="8080" protocol="HTTP/1.1"
   >             connectionTimeout="20000"
   >             redirectPort="8443" />
   > <!--修改为：-->
   > <Connector connectionTimeout="20000" port="8080"
   > protocol="HTTP/1.1" redirectPort="8443"
   > useBodyEncodingForURI="true"/>
   > ```

   （2）对参数进行重新编码：

   ```java
   String userName = new String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")
   ```

### SpringMVC 的控制器为单例模式存在问题

SpringMVC 的控制器 Controller 是单例模式，所以在多线程访问的时候有线程安全问题；

但是不使用同步，会影响性能；

解决方案：在控制器里面不能写字段。

### SpringMVC 设定重定向和转发

1. 转发：在返回值前面加 "forward:"，譬如：

```java
"forward:user.do?name=method2"
```

2. 重定向：在返回值前面加 "redirect:"，譬如：

```java
"redirect:http://www.zjut.com"
```

### SpringMVC 里面拦截器实现

第一步：**实现 HandlerInterceptor 接口**，接着在接口方法当中，实现处理逻辑

```java
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
        System.out.println("preHandle 拦截器拦截了");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
        throws Exception {
        System.out.println("afterCompletion 方法执行了");
    }
}
```

第二部：然后在 SpringMVC 的**配置文件中配置拦截器**即可

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo1"
              class="com.itheima.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

> 更多拦截器的细节，比如：
>
> 1. 拦截器的放行策略；
> 2. 拦截器的作用路径；
> 3. 多个拦截器的执行顺序；
> 4. 拦截器与过滤器的区别
> 5. ...
>
> 见Spring5MVC-02 拦截器章内容

### SpringMVC 的异常处理

系统的 dao、 service、 controller 出现异常都通过 throws Exception 向上抛出，**最后由 springmvc 前端控制器交由异常处理器进行异常处理**，如下图：

![SpringMVC异常处理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811684.PNG)

**实现步骤：**

1. 编写异常类和错误页面；

   ```java
   public class CustomException extends Exception {
       private String message;
       public CustomException(String message) {
           this.message = message;
       }
       public String getMessage() {
           return message;
       }
   }
   
   // 界面就略了
   ```
   
2. 自定义异常处理器，其中异常处理器需要**实现 HandlerExceptionResolver 接口**；

   ```java
   public class CustomExceptionResolver implements HandlerExceptionResolver {
       // 重写相应方法
       @Override
       public ModelAndView resolveException(HttpServletRequest request,
                                            HttpServletResponse response, 
                                            Object handler, Exception ex) {
           ex.printStackTrace();
           CustomException customException = null;
           // 如果抛出的是系统自定义异常则直接转换
           if(ex instanceof CustomException){
               customException = (CustomException)ex;
           }else{
               // 如果抛出的不是系统自定义异常则重新构造一个系统错误异常。
               customException = new CustomException("系统错误，请与系统管理 员联系！ ");
           }
           // 自定义异常界面
           ModelAndView modelAndView = new ModelAndView();
           modelAndView.addObject("message", customException.getMessage());
           modelAndView.setViewName("error");
           return modelAndView;
       }
   }
   ```
   
3. 配置异常处理器 

   ```xml
   <!-- 配置自定义异常处理器 -->
   <bean id="handlerExceptionResolver"
   class="com.itheima.exception.CustomExceptionResolver"/>
   ```

> 具体流程见：SpringMVC-02

### SpringMVC 和 Struts2 的区别

1. SpringMVC 的入口是一个 **Servlet 即前端控制器（DispatchServlet）**，而 Struts2 入口是一个 **filter 过滤器（StrutsPrepareAndExecuteFilter）**；
2. SpringMVC 是**基于方法开发**（一个 URL 对应一个方法），请求参数传递到方法的形参，可以设计为单例或多例（建议单例），Struts2 是**基于类开发**，传递参数是通过类的属性，只能设计为多例；
3. Struts2 采用值栈存储请求和响应的数据，通过 OGNL 存取数据；**SpringMVC 通过参数解析器**，将 request 请求内容解析，并给方法形参赋值，将数据和视图封装成 ModelAndView 对象，最后又将 ModelAndView 中的模型数据通过 request 域传输到页面。jsp 视图解析器默认使用 jstl。

## MyBatis

### 对 MyBatis 的理解 :exclamation:

1. MyBatis是一个 **半ORM（Object-Relational Mapping 对象关系映射）框架**，它内部封装了 JDBC，**开发时只需要关注 SQL 语句本身**，不需要花费精力去处理加载驱动、创建连接、创建 Statement 等繁杂的过程。程序员直接编写原生态 SQL，可以严格控制 SQL 执行性能，灵活度高。
2. MyBatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO 映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
3. 通过 XML 文件或注解的方式将要执行的各种 Statement 配置起来，并通过 Java 对象和 Statement 中 SQL 的动态参数进行映射生成最终执行的 SQL 语句，最后由 MyBatis 框架执行 SQL 并将结果映射为 Java 对象并返回。（从执行 SQL 到返回 Result 的过程）。

> 知识补充：
>
> - ORM：Object-Relationl Mapping，它的作用是在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的 SQL 语句打交道，只要像平时操作对象一样操作它就可以了 。
>
>   参考：https://blog.csdn.net/u010947534/article/details/90669452
>
> - POJO：Plain Ordinary Java Object，普通Java类，具有一部分getter/setter方法的那种类就可以称作POJO
>
>   参考：https://blog.csdn.net/tiantangdizhibuxiang/article/details/81784873

### MyBaits 的优缺点有哪些？:exclamation:

**优点：**

1. 基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL 写在 XML 里，解除 SQL 与程序代码的耦合，便于统一管理；提供 XML 标签，支持编写动态 SQL 语句，并可重用；
2. 与 JDBC 相比，减少了代码量，消除了 JDBC 大量冗余的代码，不需要手动开关连接；
3. 很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持）；
4. 能够与 Spring 很好的集成；
5. 提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射标签，支持对象关系组件维护。

**缺点：**

1. SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求；
2. SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

### MyBatis 与 Hibernate 的不同 

1. MyBatis 和 Hibernate不同，它不完全是一个 ORM 框架，因为 **MyBatis 需要程序员自己编写 SQL 语句**；Hibernate 对象/关系映射能力强，**数据库无关性好**，对于关系模型要求高的软件，如果用 Hibernate 开发可以节省很多代码，提高效率；
2. MyBatis 直接编写原生态 SQL，可以严格控制 SQL 执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。**但是灵活的前提是 MyBatis 无法做到数据库无关性**，如果需要实现支持多种数据库的软件，则需要自定义多套 SQL 映射文件，工作量大。 

> **为什么说 MyBatis是 半自动ORM 映射工具？它与全自动的区别在哪里？**
>
> Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。
>
> 而 MyBatis 在查询关联对象或关联集合对象时，需要**手动编写 SQL 来完成**，所以，称之为半自动 ORM 映射工具。

### MyBatis 中 #{} 和 ${} 的区别 :exclamation::exclamation:

**#{} 是预编译处理，${} 是字符串替换**

1. MyBatis 在处理 #{} 时，会将 SQL 中的 #{} 替换为 ? 号，调用 PreparedStatement 的 set 方法来赋值；使用 #{} 可以有效的防止 SQL 注入，提高系统安全性；
2. MyBatis 在处理 ${}  时，就是把 ${} 替换成变量的值。
3. 如果 parameterType 传输单个简单类型值时，${} 括号中只能是 value，而#{}并无这个要求

> 通过上述两种方式引入的模糊查询：
>
> 方式一：通过 #{}
>
> ```xml
> <!-- 根据名称模糊查询 -->
> <select id="findByName" resultType="com.itheima.domain.User" parameterType="String">
>     select * from user where username like #{username}
> </select>
> ```
>
> 使用：`List<User> users = userDao.findByName("%王%"); `
>
> 方式二：通过 ${}
>
> ```xml
> <!-- 根据名称模糊查询 -->
> <select id="findByName" resultType="com.itheima.domain.User" parameterType="string">
>     select * from user where username like '%${value}%'
> </select>
> ```
>
> 使用：`List<User> users = userDao.findByName("王"); `

### 实体类的属性名和表中的字段名不一致

**第1种**： 通过在查询的sql语句中**定义字段名的别名**，让字段名的别名和实体类的属性名一致。

```xml
<select id="findAll" resultType="com.itheima.domain.User">
    select id as userId,username as userName,birthday as userBirthday, sex as userSex,address as userAddress from user
</select>
```

**第2种**： 通过`<resultMap>`来映射字段名和实体类属性名的一一对应的关系。

```xml
<resultMap type="com.itheima.domain.User" id="userMap">
    <id column="id" property="userId"/>
    <result column="username" property="userName"/>
    <result column="sex" property="userSex"/>
    <result column="address" property="userAddress"/>
    <result column="birthday" property="userBirthday"/>
</resultMap>

<!-- 
	id 标签：用于指定主键字段
	result 标签：用于指定非主键字段
	column 属性：用于指定数据库列名
	property 属性：用于指定实体类属性名称 
-->

<!-- 配置查询所有操作 -->
<select id="findAll" resultMap="userMap">
    select * from user
</select>
```

### XML映射文件和Dao接口之间的映射关系 :exclamation:

Dao接口即Mapper接口。

* 接口的全限名，就是映射文件（xxxDao.xml）中的 namespace 的值；

* 接口的方法名，就是映射文件中Mapper的Statement的id值；

* 接口方法内的参数，就是传递给sql的参数。

```xml
<!--例如，作为UserDao.xml配置文件 下有：-->
<mapper namespace="com.itheima.dao.IUserDao">
    <!-- 配置查询所有操作 -->
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>

<!-- 对应的Dao接口的方法为：
public interface IUserDao {
	List<User> findAll();
}
-->
```

**Mapper接口是没有实现类的**，当调用接口方法时，**接口全限名+方法名拼接字符串作为key值，可唯一定位一个MapperStatement**。在 MyBatis 中，每一个`<select>、<insert>、<update>、<delete>`标签，都会被解析为一个 MapperStatement 对象。

> 例如：`com.itheima.dao.IUserDao.findAll`，可以唯一找到namespace为`com.itheima.dao.IUserDao`下面 id 为 `findAll` 的 MapperStatement。

**Mapper接口里的方法，是不能重载的，因为是使用 全限名+方法名 的保存和寻找策略。**Mapper 接口的工作原理是 **JDK动态代理**，MyBatis 运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的SQL，然后将SQL执行结果返回。

> **再问：MyBatis的XML映射文件中，不同的XML映射文件，id是否可以重复？**
>
> 不同的XML映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；
>
> 原因就是 `namespace+id` 是作为 `Map<String, MapperStatement>` 的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了 namespace，自然id就可以重复，namespace 不同， namespace+id 自然也就不同。
>
> 但是，在以前的 MyBatis 版本的 namespace 是可选的，不过新版本的 namespace 已经是必须的了。

### MyBatis 分页方式 :exclamation:

MyBatis 使用 RowBounds 对象进行分页，它是**针对 ResultSet 结果集执行的内存分页，而非物理分页**。可以在 SQL 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

**分页插件**的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，**在插件的拦截方法内拦截待执行的 SQL，然后重写 SQL**，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

**MyBatis 有几种分页方式？**

1. 数组分页
2. SQL 分页
3. 拦截器分页
4. RowBounds 分页

### MyBatis 逻辑分页和物理分页的区别

1. 物理分页速度上并不一定快于逻辑分页，逻辑分页速度上也并不一定快于物理分页。
2. **物理分页总是优于逻辑分页**：没有必要将属于数据库端的压力加到应用端来，就算速度上存在优势，然而其它性能上的优点足以弥补这个缺点。

### MyBatis 将 SQL 执行结果封装为目标对象并返回

第一种是使用 `<resultMap>` 标签，逐一定义数据库列名和对象属性名之间的映射关系。

第二种是使用 SQL 列的别名功能，将列的别名书写为对象属性名。

有了列名与属性名的映射关系后，**MyBatis 通过反射创建对象**，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

### 获取自动生成的(主)键值

insert 方法总是返回一个 int 值 ，这个值代表的是插入的行数。

因为 id 是由数据库的自动增长来实现的，因此需要通过以下方式获取自动增长 auto_increment 的值 ：

```xml
<insert id="saveUser" parameterType="USER">
    <!-- 配置保存时获取插入的 id -->
    <selectKey keyColumn="id" keyProperty="id" resultType="int">
        select last_insert_id();
    </selectKey>
    insert into user(username,birthday,sex,address) values(#{username},#{birthday},#{sex},#{address})
</insert>

<!-- 
    keyColumn 为数据库表的列名
    keyProperty 为domain对象中的属性名
-->
```

### MyBatis 实现一对一查询的方式

有**联合查询**和**嵌套查询**

**联合查询**是几个表联合查询，只查询一次, 通过在resultMap里面配置**association节点**配置一对一的类就可以完成；

```xml
<resultMap type="user" id="userMap">
    <id column="id" property="id"></id>
    <result column="username" property="username"/>
    <result column="address" property="address"/>
    <result column="sex" property="sex"/>
    <result column="birthday" property="birthday"/>
    <!-- 
		它是用于指定从表方的引用实体属性的
    -->
    <association property="accounts" javaType="account">
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
    </association>
</resultMap>

<!-- 配置查询所有操作 -->
<select id="findAll" resultMap="userMap">
    select u.*, a.id as aid, a.uid, a.money from user u left outer join account a on u.id =a.uid
</select>
```

**嵌套查询**是先查一个表，根据这个表里面的结果的 外键id，去再另外一个表里面查询数据，也是通过 association配置，但另外一个表的查询通过 select 属性配置：

> **可以实现延迟加载**

```xml
<!-- 一个Dao的映射文件 -->
<mapper namespace="com.itheima.dao.IAccountDao">
    <!-- 建立对应关系 -->
    <resultMap type="account" id="accountMap">
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
        <!-- 它是用于指定从表方的引用实体属性的 -->
        <association property="user" javaType="user"
                     select="com.itheima.dao.IUserDao.findById"
                     column="uid">
        </association>
    </resultMap>
    
    <select id="findAll" resultMap="accountMap">
        select * from account
    </select>
</mapper>

<!-- select： 填写我们要调用的 select 映射的 id
column ： 填写我们要传递给 select 映射的参数 -->

<!-- 另一个Dao的映射文件 -->
<mapper namespace="com.itheima.dao.IUserDao">
    <!-- 根据 id 查询 -->
    <select id="findById" resultType="user" parameterType="int" >
        select * from user where id = #{uid}
    </select>
</mapper>
```

> 具体看Mybatis讲义：3-4

### MyBatis实现一对多查询的方式

有联合查询和嵌套查询。

**联合查询**是几个表联合查询,只查询一次,通过在resultMap里面的collection节点配置一对多的类就可以完成：

```xml
<!--定义 role 表的 ResultMap-->
<resultMap id="roleMap" type="role">
    <id property="roleId" column="rid"></id>
    <result property="roleName" column="role_name"></result>
    <result property="roleDesc" column="role_desc"></result>
    <!-- collection 是用于建立一对多中集合属性的对应关系
    	 ofType 用于指定集合元素的数据类型 -->
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
    select u.*,r.id as rid,r.role_name,r.role_desc from role r left outer join user_role ur on r.id = ur.rid left outer join user u on u.id = ur.uid
</select>
```

**嵌套查询**是先查一个表,根据这个表里面的结果的外键id,去再另外一个表里面查询数据,也是通过配置collection,但另外一个表的查询通过select节点配置：

> **可以实现延迟加载**

```xml
<!-- 一个Dao的映射文件 -->
<resultMap type="user" id="userMap">
    <id column="id" property="id"></id>
    <result column="username" property="username"/>
    <result column="address" property="address"/>
    <result column="sex" property="sex"/>
    <result column="birthday" property="birthday"/>
    <!-- collection 是用于建立一对多中集合属性的对应关系
		 ofType 用于指定集合元素的数据类型
		 select 是用于指定查询账户的唯一标识（账户的 dao 全限定类名加上方法名称）
		 column 是用于指定使用哪个字段的值作为条件查询
	-->
    <collection property="accounts" ofType="account"
                select="com.itheima.dao.IAccountDao.findByUid"
                column="id">
    </collection>
</resultMap>

<!-- 配置查询所有操作 -->
<select id="findAll" resultMap="userMap">
    select * from user
</select>

<!-- 另一个Dao的映射文件 -->
<!-- 根据用户 id 查询账户信息 -->
<select id="findByUid" resultType="account" parameterType="int">
    select * from account where uid = #{uid}
</select>
```

> 具体看MyBatis讲义：3-4

### MyBatis 的延迟加载及原理 :exclamation::exclamation::exclamation:

**MyBatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载**，association 指的就是一对一，collection 指的就是一对多查询。

在MyBatis配置文件中，可以配置是否启用延迟加载`lazyLoadingEnabled=true|false`。

> ```xml
> // SqlMapConfig.xml 中添加相应配置
> <settings>
>        <setting name="lazyLoadingEnabled" value="true"/>
>        <setting name="aggressiveLazyLoading" value="false"/>
> </settings>
> ```

**它的原理是，使用 CGLIB 创建目标对象的代理对象**，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()`，拦截器 `invoke()` 方法发现 `a.getB()` 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 SQL，把 B 查询上来，然后调用 `a.setB(b)`，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()` 方法的调用。这就是延迟加载的基本原理。

### MyBatis 的一级缓存和二级缓存 :exclamation::exclamation::exclamation:

![mybatis缓存](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251811903.PNG)

**一级缓存**：基于 PerpetualCache 的 HashMap 本地缓存，**其存储作用域为 Session**，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存；

**二级缓存**：与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 **Mapper(Namespace)**，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态)，可在它的映射文件中配置 `<cache/>` ；

> 举个例子：
>
> ```xml
> // 1. SqlMapConfig.xml中进行配置
> <settings>
>        <!-- 开启二级缓存的支持 -->
>        <setting name="cacheEnabled" value="true"/>
> </settings>
> 
> // 2. 在对应的Mapper中进行配置
> <mapper namespace="com.itheima.dao.IUserDao">
>        <!-- 开启二级缓存的支持 -->
>        <cache></cache>
> </mapper>
> 
> // 3. 在查询的标签上配置useCache属性 useCache="true"
> <!-- 根据 id 查询 -->
> <select id="findById" resultType="user" parameterType="int" useCache="true">
>        select * from user where id = #{uid}
> </select>
> ```

对于缓存数据更新机制：当某一个作用域(一级缓存 Session / 二级缓存 Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

### MyBatis 的哪些执行器 Executor :exclamation:

​	MyBatis 有 3 种基本的执行器（Executor）：

1. `SimpleExecutor`：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象；
2. `ReuseExecutor`：执行 update 或 select，以 SQL 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map 内，供下一次使用。**简言之，就是重复使用 Statement 对象**；
3. `BatchExecutor`：执行 update（没有 select，JDBC 批处理不支持 select），将所有 SQL 都添加到批处理中（`addBatch()`），等待统一执行（`executeBatch()`），它缓存了多个 Statement 对象，每个Statement对象都是 `addBatch()` 完毕后，等待逐一执行 `executeBatch()` 批处理。与 JDBC 批处理相同。

### MyBatis 动态 SQL :exclamation:

1. MyBatis 动态 SQL 可以让我们在 XML 映射文件内，以标签的形式编写动态 SQL，完成逻辑判断和动态拼接 SQL 的功能；
2. MyBatis 提供了 9 种动态 SQL 标签：trim、where、set、foreach、if、choose、when、otherwise、bind；
3. 执行原理：使用 OGNL 从 SQL 参数对象中计算表达式的值，根据表达式的值动态拼接 SQL，以此来完成动态 SQL 的功能。

### MyBatis 的接口绑定与实现方式

**接口绑定，就是在 MyBatis 中任意定义接口，然后把接口里面的方法和 SQL 语句绑定**, 我们直接调用接口方法就可以，这样比起原来了 SqlSession 提供的方法我们可以有更加灵活的选择和设置。

接口绑定有两种实现方式：

1. 一种是**通过注解绑定**，就是在接口的方法上面加上 @Select、@Update 等注解，里面包含Sql语句来绑定；

2. 另外一种就是**通过 XML 里面写 SQL 来绑定**，在这种情况下，要指定 XML 映射文件里面的 namespace 必须为接口的全路径名。

当 SQL 语句比较简单时候，用注解绑定，当 SQL 语句比较复杂时候，用 XML 绑定，一般用 XML 绑定的比较多。

## Spring Boot

### Spring Boot 概述

Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重xml的配置，提供了各种启动器，在运行过程中自动配置，开发者能快速上手。

### Spring Boot 特点

一方面是Spring具有一定的**缺点**：

1. 复杂的配置：需要配置大量的 xml；
2. 项目依赖管理复杂：在搭建环境时，需要分析导入哪些库；存在版本不兼容，冲突等问题。

另一方面就是Spring Boot的**特点**：

1. 为基于Spring的开发提供更快的入门体验；
2. 开箱即用，没有代码生成，也无需 XML 配置。同时也可以修改默认值来满足特定的需求；
3. 提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等
4. **SpringBoot不是对Spring功能上的增强，而是提供了一种快速使用Spring的方式**

### JavaConfig

JavaConfig 是指基于 Java 配置的 Spring。传统的 Spring 一般都是基本xml配置的，后来 Spring3.0 新增了许多Java Config 的注解，特别是Spring Boot，基本都是清一色的 JavaConfig，有助于避免使用 XML 配置。

> 有如下注解：
>
> `@Configuration`：在类上打上这一标签，表示这个类是配置类
>
> `@ComponentScan`： 相当于 xml 的 `<context:componentscan basepakage>`
>
> `@Bean`：bean 的定义，相当于 xml 的`<bean id="objectMapper" class="org.codehaus.jackson.map.ObjectMapper" /> `
>
> `@EnableWebMvc`： 相当于xml的`<mvc:annotation-driven>`
>
> `@ImportResource`：相当于xml的`<import resource="applicationContext-cache.xml">`
>
> `@PropertySource`：spring 3.1开始引入，它是基于java config的注解，用于读取properties文件
>
> `@Profile`&`@ActiveProfiles`：配置多环境与激活环境

使用 JavaConfig 的优点在于：

1. **面向对象的配置**。由于配置被定义为 JavaConfig 中的类，因此用户可以**充分利用 Java 中的面向对象功能**。一个配置类可以继承另一个，重写它的 @Bean 方法等。
2. **减少或消除 XML 配置**。JavaConfig 为开发人员提供了一种纯 Java 方法来配置与 XML 配置概念相似的 Spring 容器。从技术角度来讲，只使用 JavaConfig 配置类来配置容器是可行的，但实际上很多人认为将 JavaConfig 与 XML 混合匹配是理想的。
3. 类型安全和重构友好。JavaConfig 提供了一种类型安全的方法来配置 Spring 容器。由于 Java 5.0 对泛型的支持，现在可以按类型而不是按名称检索 bean，不需要任何强制转换或基于字符串的查找。

### Spring Boot 配置加载顺序

springboot 启动会扫描以下位置的 application.properties/application.yml 文件作为Spring boot的默认配置文件：

1. –classpath:/config/
2. –classpath:/

SpringBoot 会从上述位置加载配置文件，优先级由高到底，**高优先级的配置会覆盖低优先级的配置，并形成互补配置**；

> 互补配置说明：
>
> 1. 在–classpath:/config/路径下配置：server.port=8081
> 2. 在–classpath:/路径下配置：server.servlet.context-path=/boot
>
> 则上述两个配置属性就形成了互补配置，访问时：http://localhost:8081/boot/hello

同时，**项目打包好以后，我们可以使用命令行参数的形式`--spring.config.location`，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

如：`java -jar XXX.jar --spring.config.location=./application.properties`

另外，SpringBoot也可以从外部进行配置文件的加载：同样高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置，大致有如下：

1. **命令行参数**

   所有的配置都可以在命令行上进行指定

   `java -jar xxx.jar --server.port=8087  --server.servlet.context-path=/boot`

   多个配置用空格分开：`--配置项=值 --配置项=值`

2. Java系统属性（`System.getProperties()`）

3. 操作系统环境变量

4. **jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

5. **jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

6. **jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

7. **jar包内部的application.properties或application.yml(不带spring.profile)配置文件**

> 对4-7点总结：
>
> 1. **由jar包外向jar包内进行寻找**
> 2. **优先加载带profile**
> 3. **再来加载不带profile**

### Spring Boot 的配置文件格式

SpringBoot 是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用application.properties 或者 application.yml（application.yaml）进行配置。

关于 yml 配置文件的优势：

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

但相比 properties 配置文件，YAML 还有一个缺点，就是不支持 @PropertySource 注解导入自定义的 YAML 配置；

> 更多配置文件是使用方式，见：Spring Boot笔记-1：2.3 配置文件注入

### Spring Boot 使用 XML 配置

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置；

进一步说明：

**@ImportResource**：导入 Spring 的配置文件，让配置文件里面的内容生效；

Spring Boot 里面没有 Spring 的配置文件，我们自己编写的配置文件，也不能自动识别；

想让 Spring 的配置文件生效，加载进来，需要用到 @ImportResource 注解，流程如下：

1. 创建在 resources 文件下 spring 的配置文件 beans.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 配置一个bean -->
       <bean id="helloService" class="cn.xyc.springboot01.service.HelloService"></bean>
   </beans>
   ```

2. 创建相应的类 `cn.xyc.springboot01.service.HelloService`，需要将这个类交给spring进行管理；

   ```java
   @SpringBootTest
   class Springboot01ApplicationTests {
   
   	@Autowired
   	ApplicationContext ioc;
   
   	@Test
   	void testHelloService(){
   		boolean b = ioc.containsBean("helloService");
   		System.out.println(b);  // false 即helloService没有被spring进行管理
   	}
   }
   ```

3. 想要自己编写的配置文件生效，需要将**@ImportResource**标注在一个配置类上

   ```java
   @ImportResource(locations = {"classpath:beans.xml"})  // 使xml配置文件生效
   @SpringBootApplication
   public class Springboot01Application {
   	public static void main(String[] args) {
   		SpringApplication.run(Springboot01Application.class, args);
   	}
   }
   ```

   此时测试发现，spring 管理了这个bean：helloService

> 当然SpringBoot目前不推荐这种方式，而是推荐通过`@Bean`注解给容器中添加组件的方式；推荐使用全注解的方式
>
> 1、配置类**@Configuration**，就是来替代之前的Spring配置文件
>
> 2、使用**@Bean**给容器中添加组件
>
> ```java
> /**
>  * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
>  * 在配置文件中用<bean><bean/>标签添加组件
>  */
> @Configuration
> public class MyAppConfig {
> 
>        // 将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
>        @Bean
>        public HelloService helloService(){
>            System.out.println("配置类@Bean给容器中添加组件了...");
>            return new HelloService();
>        }
> }
> ```

### spring boot 核心配置文件 及 bootstrap.properties & application.properties 区别

单纯做 Spring Boot 开发，可能不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时侯。

spring boot 核心的两个配置文件：

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 SpringCloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；
- application (. yml 或者 . properties)： 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。

### Spring Profiles

Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发(dev)中运行时，只有某些 bean 可以加载，而在生产环境(prod)中，某些其他 bean 可以加载。

即：**profiles是spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境**

具体使用：

1. 我们在主配置文件编写的时候，文件名可以是`application-{profile}.properties/yml`

   - 例如可以定义多个properties文件：

   ```properties
   # application.properties文件
   server.port=8081
   
   # application-dev.properties文件
   server.port=8082
   
   # application-prod.properties文件
   server.port=80
   ```

   默认启动时是使用application.properties的配置

   - yml支持多文档块方式

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

   - 通过注解：`@Profile`，任何`@Component`或`@Configuration`都可以标记为`@Profile`以限制其加载

   ```java
   @Configuration
   @Profile("prod")
   public class ProductionConfiguration {
       // ...
   }
   ```

2. 激活指定 profile

   1. 对于 properties 配置文件，在application.properties配置文件中指定`spring.profiles.active=xxx`，则激活对应的`application-{xxx}.properties`

   2. 对于yml配置文件，通过如下激活：

      ```yml
      spring:
        profiles:
          active: prod	# 激活prod环境
      ```

   3. 通过命令行：`java -jar xxx.jar --spring.profiles.active=dev;`可以直接在测试的时候，配置传入命令行参数

   4. 虚拟机参数`VM options`：`-Dspring.profiles.active=dev`

### 在自定义端口上运行 Spring Boot

为了在自定义端口上运行 Spring Boot 应用程序：

1. 修改和server有关的配置（ServerProperties，这个类其实也是实现了EmbeddedServletContainerCustomizer）；

   ```properties
   # 通用的Servlet容器设置
   # server.xxx
   server.port=8081  		   # 修改端口
   server.context-path=/crud  # 修改虚拟路径
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

### Spring Boot 中的 starter 到底是什么:exclamation::exclamation:

starter（启动器）的理念：starter 会把所有用到的依赖都给包含进来，避免了我们自己去引入依赖所带来的麻烦，即**starter是一种对依赖的合成（synthesize）**。

Spring Boot 将所有的功能场景都抽取出来，做成一个个的 starters，只需要在项目里面引入这些 starter 相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。

starter的实现：虽然不同的starter实现起来各有差异，但是他们基本上都会使用到两个相同的内容：**xxxConfigurationProperties**和**xxxAutoConfiguration**。因为Spring Boot坚信“约定大于配置”这一理念，所以我们使用xxxConfigurationProperties来保存我们的配置，并且这些配置都可以有一个默认值，即在我们没有主动覆写原始配置的情况下，默认值就会生效，这在很多情况下是非常有用的。

starter的整体逻辑：

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251812507.png" alt="starter的整体逻辑" style="zoom: 50%;" />

上面的starter依赖的jar和我们自己手动配置的时候依赖的jar并没有什么不同，因此我们可以认为starter其实是把这一些繁琐的配置操作交给了自己，而把简单交给了用户。

**自定义starter：**

首先明确：

1. 这个场景需要使用到的依赖是什么？

2. 如何编写自动配置

   ```java
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

4. 步骤：具体操作步骤见 Spring Boot笔记-3：8.自定义starter

### spring-boot-starter-parent 作用

我们都知道，新创建一个 Spring Boot 项目，默认都是有 parent 的：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
```

这个 parent 就是 spring-boot-starter-parent ，spring-boot-starter-parent 主要有如下作用：

1. 定义了 Java 编译版本。

2. 使用 UTF-8 格式编码。

3. **继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号**

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-dependencies</artifactId>
       <version>1.5.9.RELEASE</version>
       <relativePath>../../spring-boot-dependencies</relativePath>
   </parent>
   ```
   
   这个spring-boot-dependencies真正管理Spring Boot应用里面的所有依赖版本：
   
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

4. 执行打包操作的配置。

5. 自动化的资源过滤。

6. 自动化的插件配置。

7. 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。

### Spring Boot 打成的 jar 和普通的 jar 区别

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 `java -jar xxx.jar` 命令来运行，**这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类**。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 `\BOOT-INF\classes` 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

### 运行 Spring Boot 的方式

1. 直接在IDEA中运行带有`@SpringBootApplication`注解的main方法；

2. 通过maven将应用打成jar包，直接使用java -jar xxx.jar的方式运行；

   ```xml
   <!-- 这个插件，可以将应用打包成一个可执行的jar包 -->
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```
   
3. 通过spring-boot-plugin方式启动

   1. 添加插件配置：

   ```xml
   <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
   </plugin>
   ```

   2. 进入项目的根目录，执行：`mvn spring-boot:run`

### Spring Boot 不需要独立的容器

**SpringBoot默认使用Tomcat作为嵌入式的Servlet容器；**

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <!-- 引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器； -->
</dependency>
```

要切换 Jetty：

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

要切换 Undertow

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

进一步：

**嵌入式Servlet容器**：应用打成可执行的jar包

优点：简单、便携；

缺点：默认不支持JSP、优化定制比较复杂、

> 优化定制：
>
> 1. 使用定制器：ServerProperties、自定义EmbeddedServletContainerCustomizer
> 2. 自己编写嵌入式Servlet容器的创建工厂EmbeddedServletContainerFactory

若要使用**外置的Servlet容器**：外面安装Tomcat，通过war包的方式打包；

### SpringBoot 工程热部署

我们在开发中反复修改类、页面等资源，每次修改后都是需要重新启动才生效，这样每次启动都很麻烦，浪费了大量的时间，我们可以在修改代码后不重启就能生效，在 pom.xml 中添加如下配置就可以实现这样的功能，我们称之为热部署。

```xml
<!--热部署配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

注意：IDEA进行SpringBoot热部署还需要对IDEA进行自动编译的设置/IDEA使用ctrl+F9重新编译实现热部署

### Spring Boot 中实现定时任务

定时任务也是一个常见的需求，Spring Boot 中对于定时任务的支持主要还是来自 Spring 框架。

在 Spring Boot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 @Scheduled 注解，另一个则是使用第三方框架 Quartz。

> 第三方框架 Quartz，没有了解过

使用 Spring 中的 @Scheduled 的方式主要通过 @Scheduled 注解来实现：

`@EnableScheduling`：标注在主类，开启对定时任务支持

`@Scheduled`：标注在执行的方法上，并制定cron属性

> 关于**cron表达式**可见：SpringBoot笔记-4：Spring Boot与任务-2.定时任务

### Spring Boot 中的监视器

通过引入spring-boot-starter-actuator，可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息等

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

更多信息见：Spring Boot笔记-4：Spring Boot与监控管理

### Spring Boot 与安全

应用程序的两个主要区域是“认证”和“授权”（或者访问控制），这两个主要区域是 Spring Security 的两个目标。身份验证意味着**确认您自己的身份**，而授权意味着**授予对系统的访问权限**

**认证**

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。系统确定您是否就是您所说的使用凭据。在公共和专用网络中，系统通过登录密码验证用户身份。身份验证通常通过用户名和密码完成，

**授权**

另一方面，授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。简单来说，授权决定了您访问系统的能力以及达到的程度。验证成功后，系统验证您的身份后，即可授权您访问系统资源。

**Spring Security**：

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入`spring-boot-starter-security`模块，进行少量的配置，即可实现强大的安全管理。

**WebSecurityConfigurerAdapter：自定义Security策略**

通过在配置类中继承该类重写`configure(HttpSecurity http)`方法来实现自定义策略

**@EnableWebSecurity：开启WebSecurity模式**

在配置类上标注`@EnableWebSecurity`开启WebSecurity模式

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

更多信息见：Spring Boot笔记-4：Spring boot与安全

### Spring Boot的自动配置原理 :exclamation::exclamation::exclamation:(待完善)

**@SpringBootApplication**：SpringBoot 应用标注在某个类上说明这个类是 SpringBoot 的主配置类，SpringBoot 就应该运行这个类的 main 方法来启动 SpringBoot 应用；

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个SpringBoot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class, args);
    }
}
```

**@SpringBootApplication**：注解的进一步说明：

```java
@SpringBootConfiguration  // 说明注解1
@EnableAutoConfiguration  // 说明注解2
@ComponentScan(excludeFilters = { 
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    // ...
}
```

1. 说明注解1：**@SpringBootConfiguration**，SpringBoot的配置类； 标注在某个类上，表示这是一个SpringBoot的配置类；

   ```java
   // SpringBootConfiguration注解内容
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Configuration  // 说明这是一个配置类，配置类就是对应Spring的xml配置文件
   public @interface SpringBootConfiguration {
   
   }
   
   // Configuration注解内容
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Component  // 说明这是Spring容器中的一个组件
   public @interface Configuration {
       // ...
   }
   ```
   
2. 说明注解2：**@EnableAutoConfiguration**，开启自动配置功能；

   以前我们需要配置的东西，现在SpringBoot帮我们自动配置；**@EnableAutoConfiguration**告诉SpringBoot开启自动配置功能；这样自动配置才能生效；@EnableAutoConfiguration上的注解如下：

   ```java
   // EnableAutoConfiguration的注解内容
   @AutoConfigurationPackage  // 后续说明1
   @Import(AutoConfigurationImportSelector.class)  // 后续说明
   public @interface EnableAutoConfiguration {
       //..
   }
   ```

   对注解**@AutoConfigurationPackage**说明：自动配置包，

   ```java
   @Import(AutoConfigurationPackages.Registrar.class)
   public @interface AutoConfigurationPackage {
   }
   
   // 上述注解中@Import：Spring的底层注解，给容器中导入一个组件；
   // 导入的组件为:AutoConfigurationPackages.Registrar.class
   // 通过@EnableAutoConfiguration注解下的@AutoConfigurationPackage注解，将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；
   ```

   **后续说明1**：对其进行Debug分析后可以发现其作用是：**总结1：通过@EnableAutoConfiguration注解下的@AutoConfigurationPackage注解，将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；**

   **后续说明2**：注解**@Import(AutoConfigurationImportSelector.class)**说明：

   进入`AutoConfigurationImportSelector`类源码进行分析：

   1. 利用`@Import(AutoConfigurationImportSelector.class)`给容器中导入一些组件

   ```java
   // AutoConfigurationImportSelector 类下的 selectImports方法
   // 这个方法将所有需要导入的组件以全类名的方式返回(String数组)；这些组件就会被添加到容器
   @Override
   public String[] selectImports(AnnotationMetadata annotationMetadata) {
       if (!isEnabled(annotationMetadata)) {
           return NO_IMPORTS;
       }
       // 关键加载方法getAutoConfigurationEntry，后续分析
       AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
       return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
   }
   
   // getAutoConfigurationEntry方法：
   protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
       if (!isEnabled(annotationMetadata)) {
           return EMPTY_ENTRY;
       }
       AnnotationAttributes attributes = getAttributes(annotationMetadata);
       // 从这里获取了相应的候选配置文件信息(关键方法)，后续分析
       List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
       configurations = removeDuplicates(configurations);
       Set<String> exclusions = getExclusions(annotationMetadata, attributes);
       checkExcludedClasses(configurations, exclusions);
       configurations.removeAll(exclusions);
       configurations = filter(configurations, autoConfigurationMetadata);
       fireAutoConfigurationImportEvents(configurations, exclusions);
       return new AutoConfigurationEntry(configurations, exclusions);
   }
   
   // 通过getCandidateConfigurations方法，获取候选的配置
   // 从这里获取了相应的候选配置类信息
   protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
       // 主要是SpringFactoriesLoader.loadFactoryNames方法(关键方法)
       List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                            getBeanClassLoader());
       Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                       + "are using a custom packaging, make sure that file is correct.");
       return configurations;
   }
   
   // SpringFactoriesLoader.loadFactoryNames方法
   // 查看SpringFactoriesLoader.loadFactoryNames方法
   public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
       String factoryTypeName = factoryType.getName();
       // 后续查看pringFactoriesLoader.loadFactoryNames方法
       return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
   }
   // loadSpringFactories函数
   private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
       MultiValueMap<String, String> result = cache.get(classLoader);
       if (result != null) {
           return result;
       }
   
       try {
           // 扫描所有jar包类路径下  META-INF/spring.factories
           Enumeration<URL> urls = (classLoader != null ?
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

   因此该函数的作用：**扫描所有jar包类路径下的META-INF/spring.factories**

   把扫描到的这些文件的内容包装成properties对象

   从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中

   **总结2：SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都帮我们**

   > ```properties
   > # 例如在导入的依赖jar包中：
   > # org.springframework.boot.autoconfigure:2.2.5.RELEASE.jar下有：/MATE-INF/spring.fatories
   > # spring.fatories 配置文件中有如下内容：
   > 
   > # Auto Configure
   > org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   > org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
   > org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
   > org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
   > ```
   >
   > 每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

3. 每一个自动配置类 xxxAutoConfiguration 会进行自动配置功能，对以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；

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
   
   所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类
   
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
   
   ```java
   spring.http.encoding.enabled=true
      spring.http.encoding.force=true
      spring.http.encoding.charset=UTF-8
      spring.http.encoding.force-request=true
      spring.http.encoding.force-response=true
   ```
   

**总结3：**

1. **SpringBoot启动会加载大量的自动配置类xxxAutoConfiguration**
2. **我们看我们需要的功能SpringBoot是否默认写好了自动配置类；**
3. **我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）**
4. **给容器中自动配置类添加组件的时候，会从xxxxProperties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**

> xxxxAutoConfigurartion：自动配置类；给容器中添加组件；
>
> xxxxProperties:封装配置文件中相关属性； 

