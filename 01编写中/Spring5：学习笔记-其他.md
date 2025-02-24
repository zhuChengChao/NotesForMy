# Spring5：学习笔记

[课程地址](https://www.bilibili.com/video/BV1P44y1N7QG/?p=156)

## 其他内容

### 43. FactoryBean

**演示代码**

cn.xyc.a43.A43

```java
/**
 * @author xiaochao
 * @date 2025/2/16 18:29
 */
@ComponentScan
public class A43 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A43.class);

        Bean1 bean1 = (Bean1) context.getBean("bean1");
        Bean1 bean2 = (Bean1) context.getBean("bean1");
        Bean1 bean3 = (Bean1) context.getBean("bean1");
        System.out.println(bean1);
        System.out.println(bean2);
        System.out.println(bean3);

        System.out.println(context.getBean(Bean1.class));

        System.out.println(context.getBean(Bean1FactoryBean.class));
        System.out.println(context.getBean("&bean1"));

        context.close();

        /*
            学到了什么: 一个在 Spring 发展阶段中重要, 但目前已经很鸡肋的接口 FactoryBean 的使用要点
            说它鸡肋有两点:
                1. 它的作用是用制造创建过程较为复杂的产品, 如 SqlSessionFactory, 但 @Bean 已具备等价功能
                2. 使用上较为古怪, 一不留神就会用错
                    a. 被 FactoryBean 创建的产品
                        - 会认为创建、依赖注入、Aware 接口回调、前初始化这些都是 FactoryBean 的职责, 这些流程都不会走
                        - 唯有后初始化的流程会走, 也就是产品可以被代理增强
                        - 单例的产品不会存储于 BeanFactory 的 singletonObjects 成员中, 而是另一个 factoryBeanObjectCache 成员中
                    b. 按名字去获取时, 拿到的是产品对象, 名字前面加 & 获取的是工厂对象
            就说恶心不?

            但目前此接口的实现仍被大量使用, 想被全面废弃很难
         */
    }
}
```

cn.xyc.a43.Bean1 && cn.xyc.a43.Bean2

```
/**
 * @author xiaochao
 * @date 2025/2/16 18:29
 */
public class Bean1 implements BeanFactoryAware {
    private static final Logger log = LoggerFactory.getLogger(Bean1.class);

    private Bean2 bean2;

    @Autowired
    public void setBean2(Bean2 bean2) {
        log.debug("setBean2({})", bean2);
        this.bean2 = bean2;
    }

    public Bean2 getBean2() {
        return bean2;
    }

    @PostConstruct
    public void init() {
        log.debug("init");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        log.debug("setBeanFactory({})", beanFactory);
    }
}

/**
 * @author xiaochao
 * @date 2025/2/16 18:29
 */
@Component
public class Bean2 {
}
```

cn.xyc.a43.Bean1FactoryBean

```java
/**
 * @author xiaochao
 * @date 2025/2/16 18:30
 */
@Component("bean1")
public class Bean1FactoryBean implements FactoryBean<Bean1> {

    private static final Logger log = LoggerFactory.getLogger(Bean1FactoryBean.class);

    @Override
    public Bean1 getObject() throws Exception {
        Bean1 bean1 = new Bean1();
        log.debug("create bean: {}", bean1);
        return bean1;
    }

    @Override
    public Class<?> getObjectType() {
        // 决定了根据【类型】获取或依赖注入能否成功
        return Bean1.class;
    }

    @Override
    public boolean isSingleton() {
        // 决定了 getObject() 方法被调用一次还是多次
        return true;
    }
}
```

cn.xyc.a43.Bean1PostProcessor

```java
/**
 * @author xiaochao
 * @date 2025/2/16 18:32
 */
@Component
public class Bean1PostProcessor implements BeanPostProcessor {
    private static final Logger log = LoggerFactory.getLogger(Bean1PostProcessor.class);

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("bean1") && bean instanceof Bean1) {
            log.debug("before [{}] init", beanName);
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("bean1") && bean instanceof Bean1) {
            log.debug("after [{}] init", beanName);
        }
        return bean;
    }
}
```

**收获**

它的作用是用制造创建过程较为复杂的产品, 如 SqlSessionFactory, 但 @Bean 已具备等价功能

使用上较为古怪, 一不留神就会用错

1. 被 FactoryBean 创建的产品
   * 会认为创建、依赖注入、Aware 接口回调、前初始化这些都是 FactoryBean 的职责, 这些流程都不会走
   * 唯有后初始化的流程会走, 也就是产品可以被代理增强
   * 单例的产品不会存储于 BeanFactory 的 singletonObjects 成员中, 而是另一个 factoryBeanObjectCache 成员中
2. 按名字去获取时, 拿到的是产品对象, 名字前面加 & 获取的是工厂对象

### 44. @Indexed 原理

代码参考

```java
/**
 * 做这个试验前, 先在 target/classes 创建 META-INF/spring.components, 内容为
 *
 *     com.itheima.a44.Bean1=org.springframework.stereotype.Component
 *     com.itheima.a44.Bean2=org.springframework.stereotype.Component
 *
 *     做完实现建议删除, 避免影响其它组件扫描的结果
 *
 *     真实项目中, 这个步骤可以自动完成, 加入以下依赖
 *     <dependency>
 *         <groupId>org.springframework</groupId>
 *         <artifactId>spring-context-indexer</artifactId>
 *         <optional>true</optional>
 *     </dependency>
 *
 * @author xiaochao
 * @date 2025/2/16 18:39
 */
public class A44 {
    public static void main(String[] args) throws IOException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        // 组件扫描的核心类
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(beanFactory);

        scanner.scan(A44.class.getPackage().getName());

        for (String name : beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }

        /*
            学到了什么
                a. @Indexed 的原理, 在编译时就根据 @Indexed 生成 META-INF/spring.components 文件
                扫描时
                1. 如果发现 META-INF/spring.components 存在, 以它为准加载 bean definition
                2. 否则, 会遍历包下所有 class 资源 (包括 jar 内的)
         */
    }
}
```

**收获**

在编译时就根据 @Indexed 生成 META-INF/spring.components 文件

扫描时

* 如果发现 META-INF/spring.components 存在, 以它为准加载 bean definition
* 否则, 会遍历包下所有 class 资源 (包括 jar 内的)

解决的问题，在编译期就找到 @Component 组件，节省运行期间扫描 @Component 的时间

### 45. 代理进一步理解

**演示代码**

cn.xyc.a45.A45

```java
@SpringBootApplication
public class A45 {

    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext context = SpringApplication.run(A45.class, args);

        Bean1 proxy = context.getBean(Bean1.class);
        /*
            1.演示 spring 代理的设计特点
                依赖注入和初始化影响的是原始对象
                代理与目标是两个对象，二者成员变量并不共用数据
         */
        showProxyAndTarget(proxy);
        System.out.println(">>>>>>>>>>>>>>>>>>>");
        System.out.println(proxy.getBean2());
        System.out.println(proxy.isInitialized());

        /*
            2.演示 static 方法、final 方法、private 方法均无法增强
         */

        proxy.m1();
        proxy.m2();
        proxy.m3();
        Method m4 = Bean1.class.getDeclaredMethod("m4");
        m4.setAccessible(true);
        m4.invoke(proxy);

        context.close();
    }

    public static void showProxyAndTarget(Bean1 proxy) throws Exception {
        System.out.println(">>>>> 代理中的成员变量");
        System.out.println("\tinitialized=" + proxy.initialized);
        System.out.println("\tbean2=" + proxy.bean2);

        if (proxy instanceof Advised) {
            Advised advised = (Advised)proxy;
            System.out.println(">>>>> 目标中的成员变量");
            Bean1 target = (Bean1)advised.getTargetSource().getTarget();
            System.out.println("\tinitialized=" + target.initialized);
            System.out.println("\tbean2=" + target.bean2);
        }
    }
}
```

cn.xyc.a45.Bean1 && cn.xyc.a45.Bean2

```java
/**
 * @author xiaochao
 * @date 2025/2/16 19:23
 */
@Component
public class Bean1 {
    private static final Logger log = LoggerFactory.getLogger(Bean1.class);

    protected Bean2 bean2;

    protected boolean initialized;

    @Autowired
    public void setBean2(Bean2 bean2) {
        log.debug("setBean2(Bean2 bean2)");
        this.bean2 = bean2;
    }

    @PostConstruct
    public void init() {
        log.debug("init");
        initialized = true;
    }

    public Bean2 getBean2() {
        log.debug("getBean2()");
        return bean2;
    }

    public boolean isInitialized() {
        log.debug("isInitialized()");
        return initialized;
    }

    public void m1() {
        System.out.println("m1() public 成员方法");
    }

    final public void m2() {
        System.out.println("m2() final 方法");
    }

    static public void m3() {
        System.out.println("m3() static 方法");
    }

    private void m4() {
        System.out.println("m4() private 方法");
    }
}

/**
 * @author xiaochao
 * @date 2025/2/16 19:23
 */
@Component
public class Bean2 {
}
```

cn.xyc.a45.MyAspect

```java
/**
 * @author xiaochao
 * @date 2025/2/16 19:24
 */
@Aspect
@Component
public class MyAspect {

    @Before("execution(* cn.xyc.a45.Bean1.*(..))")
    public void before() {
        System.out.print("before 增强");
    }
}
```

**spring 代理的设计特点**

* 依赖注入和初始化影响的是原始对象

  因此 cglib 不能用 MethodProxy.invokeSuper()

* 代理与目标是两个对象，二者成员变量并不共用数据

* static 方法、final 方法、private 方法均无法增强，因为代理的增强是基于方法重写实现

### 46) @Value 装配底层

> **按类型装配的步骤**
>
> 1. 查看需要的类型是否为 Optional，是，则进行封装（非延迟），否则向下走
> 2. 查看需要的类型是否为 ObjectFactory 或 ObjectProvider，是，则进行封装（延迟），否则向下走
> 3. 查看需要的类型（成员或参数）上是否用 @Lazy 修饰，是，则返回代理，否则向下走
> 4. 解析 @Value 的值
>    1. 如果需要的值是字符串，先解析 ${ }，再解析 #{ }
>    2. 不是字符串，需要用 TypeConverter 转换
> 5. 看需要的类型是否为 Stream、Array、Collection、Map，是，则按集合处理，否则向下走
> 6. 在 BeanFactory 的 resolvableDependencies 中找有没有类型合适的对象注入，没有向下走
> 7. 在 BeanFactory 及父工厂中找类型匹配的 bean 进行筛选，筛选时会考虑 @Qualifier 及泛型
> 8. 结果个数为 0 抛出 NoSuchBeanDefinitionException 异常 
> 9. 如果结果 > 1，再根据 @Primary 进行筛选
> 10. 如果结果仍 > 1，再根据成员名或变量名进行筛选
> 11. 结果仍 > 1，抛出 NoUniqueBeanDefinitionException 异常

**代码参考**

```java
/**
 * 本章节作为第四讲的延续
 *
 * @author xiaochao
 * @date 2025/2/16 19:30
 */
@Configuration
public class A46 {

    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A46.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();

        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);

        System.out.println("--------test1--------");
        test1(context, resolver, Bean1.class.getDeclaredField("home"));
        System.out.println("--------test2--------");
        test2(context, resolver, Bean1.class.getDeclaredField("age"));
        System.out.println("--------test3--------");
        test3(context, resolver, Bean2.class.getDeclaredField("bean3"));
        System.out.println("--------test4--------");
        test3(context, resolver, Bean4.class.getDeclaredField("value"));

        context.close();
    }

    private static void test1(AnnotationConfigApplicationContext context,
        ContextAnnotationAutowireCandidateResolver resolver, Field field) {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        // 获取 @Value 的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);

        // 解析 ${}
        value = context.getEnvironment().resolvePlaceholders(value);
        System.out.println(value);
    }

    private static void test2(AnnotationConfigApplicationContext context,
        ContextAnnotationAutowireCandidateResolver resolver, Field field) {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        // 获取 @Value 的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);

        // 解析 ${}
        value = context.getEnvironment().resolvePlaceholders(value);
        System.out.println(value);
        System.out.println(value.getClass());
        Object age = context.getBeanFactory().getTypeConverter().convertIfNecessary(value, dd1.getDependencyType());
        System.out.println(age.getClass());
    }

    private static void test3(AnnotationConfigApplicationContext context,
        ContextAnnotationAutowireCandidateResolver resolver, Field field) {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        // 获取 @Value 的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);

        // 解析 ${}
        value = context.getEnvironment().resolvePlaceholders(value);
        System.out.println(value);
        System.out.println(value.getClass());

        // 解析 #{} @bean3
        Object bean3 = context.getBeanFactory().getBeanExpressionResolver()
            .evaluate(value, new BeanExpressionContext(context.getBeanFactory(), null));

        // 类型转换
        Object result = context.getBeanFactory().getTypeConverter().convertIfNecessary(bean3, dd1.getDependencyType());
        System.out.println(result);
    }

    public class Bean1 {
        @Value("${JAVA_HOME}")
        private String home;
        @Value("18")
        private int age;
    }

    public class Bean2 {
        @Value("#{@bean3}") // SpringEL表达式  #{SpEL}
        private Bean3 bean3;
    }

    @Component("bean3")
    public class Bean3 {
    }

    static class Bean4 {
        @Value("#{'hello, ' + '${JAVA_HOME}'}")
        private String value;
    }
}
```

**收获**

1. ContextAnnotationAutowireCandidateResolver 作用之一，获取 @Value 的值
2. 了解 ${ } 对应的解析器
3. 了解 #{ } 对应的解析器
4. TypeConvert 的一项体现

### 47. @Autowired 装配底层

##### 演示代码-1

```java
@Configuration
public class A47_1 {

    public static void main(String[] args) throws NoSuchFieldException, NoSuchMethodException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_1.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        // 1. 根据成员变量的类型注入
        DependencyDescriptor dd1 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean2"), false);
        System.out.println(beanFactory.doResolveDependency(dd1, "bean1", null, null));
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        // 2. 根据参数的类型注入
        Method setBean2 = Bean1.class.getDeclaredMethod("setBean2", Bean2.class);
        DependencyDescriptor dd2 = new DependencyDescriptor(new MethodParameter(setBean2, 0), false);
        System.out.println(beanFactory.doResolveDependency(dd2, "bean1", null, null));
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        // 3. 结果包装为 Optional<Bean2>
        DependencyDescriptor dd3 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean3"), false);
        if (dd3.getDependencyType() == Optional.class) {
            dd3.increaseNestingLevel();
            Object result = beanFactory.doResolveDependency(dd3, "bean1", null, null);
            System.out.println(Optional.ofNullable(result));
        }
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        // 4. 结果包装为 ObjectProvider,ObjectFactory
        DependencyDescriptor dd4 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean4"), false);
        if (dd4.getDependencyType() == ObjectFactory.class) {
            dd4.increaseNestingLevel();
            ObjectFactory objectFactory = new ObjectFactory() {
                @Override
                public Object getObject() throws BeansException {
                    return beanFactory.doResolveDependency(dd4, "bean1", null, null);
                }
            };
            System.out.println(objectFactory.getObject());
        }
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        // 5. 对 @Lazy 的处理
        DependencyDescriptor dd5 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean2"), false);
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        Object proxy = resolver.getLazyResolutionProxyIfNecessary(dd5, "bean1");
        System.out.println(proxy);
        System.out.println(proxy.getClass());
        /*
            学到了什么
                1. Optional 及 ObjectFactory 对于内嵌类型的处理, 源码参考 ResolvableType
                2. ObjectFactory 懒惰的思想
                3. @Lazy 懒惰的思想
            下一节, 继续学习 doResolveDependency 内部处理
         */
    }

    static class Bean1 {
        @Autowired
        @Lazy
        private Bean2 bean2;

        @Autowired
        public void setBean2(Bean2 bean2) {
            this.bean2 = bean2;
        }

        @Autowired
        private Optional<Bean2> bean3;
        @Autowired
        private ObjectFactory<Bean2> bean4;
    }

    @Component("bean2")
    static class Bean2 {
    }
}
```

##### 演示代码-2

```java
@Configuration
public class A47_2 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_2.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>> 1. 数组类型");
        testArray(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>> 2. List 类型");
        testList(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>> 3. applicationContext");
        testApplicationContext(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>> 4. 泛型");
        testGeneric(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>> 5. @Qualifier");
        testQualifier(beanFactory);
        /*
            学到了什么
                1. 如何获取数组元素类型
                2. Spring 如何获取泛型中的类型
                3. 特殊对象的处理, 如 ApplicationContext, 并注意 Map 取值时的类型匹配问题 (另见  TestMap)
                4. 谁来进行泛型匹配 (另见 TestGeneric)
                5. 谁来处理 @Qualifier
                6. 刚开始都只是按名字处理, 等候选者确定了, 才会创建实例
         */
    }

    private static void testQualifier(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd5 = new DependencyDescriptor(Target.class.getDeclaredField("service"), true);
        Class<?> type = dd5.getDependencyType();
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        for (String name : BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type)) {
            BeanDefinition bd = beanFactory.getMergedBeanDefinition(name);
            // dd5是加了@Qualifier("service2")注解的
            if (resolver.isAutowireCandidate(new BeanDefinitionHolder(bd, name), dd5)) {
                System.out.println(name);
                System.out.println(dd5.resolveCandidate(name, type, beanFactory));
            }
        }
    }

    private static void testGeneric(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd4 = new DependencyDescriptor(Target.class.getDeclaredField("dao"), true);
        Class<?> type = dd4.getDependencyType();
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        for (String name : BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type)) {
            BeanDefinition bd = beanFactory.getMergedBeanDefinition(name);
            // 对比 BeanDefinition 与 DependencyDescriptor 的泛型是否匹配
            if (resolver.isAutowireCandidate(new BeanDefinitionHolder(bd, name), dd4)) {
                System.out.println(name);
                System.out.println(dd4.resolveCandidate(name, type, beanFactory));
            }
        }
    }

    private static void testApplicationContext(DefaultListableBeanFactory beanFactory)
        throws NoSuchFieldException, IllegalAccessException {
        DependencyDescriptor dd3 = new DependencyDescriptor(Target.class.getDeclaredField("applicationContext"), true);

        Field resolvableDependencies = DefaultListableBeanFactory.class.getDeclaredField("resolvableDependencies");
        resolvableDependencies.setAccessible(true);
        Map<Class<?>, Object> dependencies = (Map<Class<?>, Object>)resolvableDependencies.get(beanFactory);
        /*dependencies.forEach((k, v) -> {
            System.out.println("key:" + k + " value: " + v);
        });*/
        for (Map.Entry<Class<?>, Object> entry : dependencies.entrySet()) {
            // isAssignableFrom表示，dd3.getDependencyType()类型是否能赋值给entry.getKey()
            if (entry.getKey().isAssignableFrom(dd3.getDependencyType())) {
                System.out.println(entry.getValue());
                break;
            }
        }
    }

    private static void testList(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd2 = new DependencyDescriptor(Target.class.getDeclaredField("serviceList"), true);
        if (dd2.getDependencyType() == List.class) {
            Class<?> resolve = dd2.getResolvableType().getGeneric().resolve();
            System.out.println(resolve);
            List<Object> list = new ArrayList<>();
            String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, resolve);
            for (String name : names) {
                Object bean = dd2.resolveCandidate(name, resolve, beanFactory);
                list.add(bean);
            }
            System.out.println(list);
        }
    }
    private static void testArray(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd1 = new DependencyDescriptor(Target.class.getDeclaredField("serviceArray"), true);
        if (dd1.getDependencyType().isArray()) {
            Class<?> componentType = dd1.getDependencyType().getComponentType();
            System.out.println(componentType);
            String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, componentType);
            List<Object> beans = new ArrayList<>();
            for (String name : names) {
                System.out.println(name);
                Object bean = dd1.resolveCandidate(name, componentType, beanFactory);
                beans.add(bean);
            }
            Object array = beanFactory.getTypeConverter().convertIfNecessary(beans, dd1.getDependencyType());
            System.out.println(array);
        }
    }

    static class Target {
        @Autowired
        private Service[] serviceArray;
        @Autowired
        private List<Service> serviceList;
        @Autowired
        private ConfigurableApplicationContext applicationContext;
        @Autowired
        private Dao<Teacher> dao;
        @Autowired
        @Qualifier("service2")
        private Service service;
    }

    interface Dao<T> {}

    @Component("dao1") static class Dao1 implements Dao<Student> {}
    @Component("dao2") static class Dao2 implements Dao<Teacher> {}

    static class Student {}

    static class Teacher {}

    interface Service {}

    @Component("service1")
    static class Service1 implements Service {}

    @Component("service2")
    static class Service2 implements Service {}

    @Component("service3")
    static class Service3 implements Service {}
}
```

##### 演示代码-3

```java
@Configuration
public class A47_3 {
    public static void main(String[] args) throws NoSuchFieldException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_3.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        testPrimary(beanFactory);
        testDefault(beanFactory);

        /*
            学到了什么
                1. @Primary 的处理, 其中 @Primary 会在 @Bean 解析或组件扫描时被解析 (另见 TestPrimary)
                2. 最后的防线, 通过属性或参数名匹配
         */
    }

    private static void testDefault(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd = new DependencyDescriptor(Target2.class.getDeclaredField("service3"), false);
        Class<?> type = dd.getDependencyType();
        for (String name : BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type)) {
            if (name.equals(dd.getDependencyName())) {
                System.out.println(name);
            }
        }
    }

    private static void testPrimary(DefaultListableBeanFactory beanFactory) throws NoSuchFieldException {
        DependencyDescriptor dd = new DependencyDescriptor(Target1.class.getDeclaredField("service"), false);
        Class<?> type = dd.getDependencyType();
        for (String name : BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type)) {
            if (beanFactory.getMergedBeanDefinition(name).isPrimary()) {
                System.out.println(name);
            }
        }
    }

    static class Target1 {
        @Autowired
        private Service service;
    }

    static class Target2 {
        @Autowired
        private Service service3;
    }

    interface Service {

    }

    @Component("service1")
    static class Service1 implements Service {

    }

    @Component("service2")
    @Primary
    static class Service2 implements Service {

    }

    @Component("service3")
    static class Service3 implements Service {

    }
}
```

##### 收获

1. @Autowired 本质上是根据成员变量或方法参数的类型进行装配

   ```java
   @Autowired private Bean2 bean2;
   
   @Autowired
   public void setBean2(Bean2 bean2) { this.bean2 = bean2;}
   ```

2. 如果待装配类型是 Optional，需要根据 Optional 泛型找到 bean，再封装为 Optional 对象装配

   ```java
   @Autowired private Optional<Bean2> bean3;
   ```

3. 如果待装配的类型是 ObjectFactory，需要根据 ObjectFactory 泛型创建 ObjectFactory 对象装配
   
   此方法可以延迟 bean 的获取
   
   ```java
   @Autowired private ObjectFactory<Bean2> bean4;
   ```
   
4. 如果待装配的成员变量或方法参数上用 @Lazy 标注，会创建代理对象装配
   
   此方法可以延迟真实 bean 的获取
   
   被装配的代理不作为 bean
   
   ```java
   @Autowired @Lazy private Bean2 bean2;
   ```
   
5. 如果待装配类型是数组，需要获取数组元素类型，根据此类型找到多个 bean 进行装配

   ```java
   @Autowired private Service[] serviceArray;
   ```

6. 如果待装配类型是 Collection 或其子接口，需要获取 Collection 泛型，根据此类型找到多个 bean

   ```java
   @Autowired private List<Service> serviceList;
   ```

7. 如果待装配类型是 ApplicationContext 等特殊类型
   
   会在 BeanFactory 的 resolvableDependencies 成员按类型查找装配
   
   resolvableDependencies 是 map 集合，key 是特殊类型，value 是其对应对象
   
   不能直接根据 key 进行查找，而是用 isAssignableFrom 逐一尝试右边类型是否可以被赋值给左边的 key 类型
   
   ```java
   @Autowired private ConfigurableApplicationContext applicationContext;
   ```
   
8. 如果待装配类型有泛型参数
   
   需要利用 ContextAnnotationAutowireCandidateResolver 按泛型参数类型筛选
   
   ```java
   @Autowired private Dao<Teacher> dao;
   ```
   
9. 如果待装配类型有 @Qualifier
   
   需要利用 ContextAnnotationAutowireCandidateResolver 按注解提供的 bean 名称筛选
   
   ```java
   @Autowired @Qualifier("service2") private Service service;
   ```
   
10. 有 @Primary 标注的 @Component 或 @Bean 的处理

    ```java
    @Component("service2")
    @Primary
    static class Service2 implements Service {}
    ```

11. 与成员变量名或方法参数名同名 bean 的处理

    ```java
    static class Target2 {
        @Autowired private Service service3;
    }
    ```

### 48. 事件监听器

#### ApplicationListener

根据接口泛型确定事件类型

```java
/**
 * @author xiaochao
 * @date 2025/2/23 21:15
 */
@Configuration
public class A48_1 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A48_1.class);
        context.getBean(MyService.class).doBusiness();
        context.close();
    }

    @Component
    static class MyService {
        private static final Logger log = LoggerFactory.getLogger(MyService.class);
        @Autowired
        private ApplicationEventPublisher publisher; // applicationContext

        public void doBusiness() {
            log.debug("主线业务");
            // 主线业务完成后需要做一些支线业务，下面是问题代码
            publisher.publishEvent(new MyEvent("MyService.doBusiness()"));
        }
    }

    static class MyEvent extends ApplicationEvent {
        public MyEvent(Object source) {
            super(source);
        }
    }

    @Component
    static class SmsApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(SmsApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送短信");
        }
    }

    @Component
    static class EmailApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(EmailApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送邮件");
        }
    }
}
```

```java
/**
 * @author xiaochao
 * @date 2025/2/23 21:15
 */
@Configuration
public class A48_1 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A48_1.class);
        context.getBean(MyService.class).doBusiness();
        context.close();
    }

    @Component
    static class MyService {
        private static final Logger log = LoggerFactory.getLogger(MyService.class);
        @Autowired
        private ApplicationEventPublisher publisher; // applicationContext

        public void doBusiness() {
            log.debug("主线业务");
            // 主线业务完成后需要做一些支线业务，下面是问题代码
            publisher.publishEvent(new MyEvent("MyService.doBusiness()"));
        }
    }

    static class MyEvent extends ApplicationEvent {
        public MyEvent(Object source) {
            super(source);
        }
    }

    @Component
    static class SmsApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(SmsApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送短信");
        }
    }

    @Component
    static class EmailApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(EmailApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送邮件");
        }
    }
}
```

#### @EventListener

```java
@Configuration
public class A48_2 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A48_2.class);
        context.getBean(MyService.class).doBusiness();
        context.close();
    }

    static class MyEvent extends ApplicationEvent {
        public MyEvent(Object source) {
            super(source);
        }
    }

    @Component
    static class MyService {
        private static final Logger log = LoggerFactory.getLogger(MyService.class);
        @Autowired
        private ApplicationEventPublisher publisher; // applicationContext
        public void doBusiness() {
            log.debug("主线业务");
            // 主线业务完成后需要做一些支线业务
            publisher.publishEvent(new MyEvent("MyService.doBusiness()"));
        }
    }

    @Component
    static class SmsService {
        private static final Logger log = LoggerFactory.getLogger(SmsService.class);
        @EventListener
        public void listener(MyEvent myEvent) {
            log.debug("发送短信");
        }
    }

    @Component
    static class EmailService {
        private static final Logger log = LoggerFactory.getLogger(EmailService.class);
        @EventListener
        public void listener(MyEvent myEvent) {
            log.debug("发送邮件");
        }
    }

    @Bean
    public ThreadPoolTaskExecutor executor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        return executor;
    }

    @Bean
    public SimpleApplicationEventMulticaster applicationEventMulticaster(ThreadPoolTaskExecutor executor) {
        SimpleApplicationEventMulticaster multicaster = new SimpleApplicationEventMulticaster();
        multicaster.setTaskExecutor(executor);
        return multicaster;
    }
}
```

#### @EventListener 原理

```java
@Configuration
public class A48_3 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A48_3.class);
        context.getBean(MyService.class).doBusiness();
        context.close();
    }

    @Bean
    public SmartInitializingSingleton smartInitializingSingleton(ConfigurableApplicationContext context) {

        return new SmartInitializingSingleton() {
            @Override
            public void afterSingletonsInstantiated() {
                for (String name : context.getBeanDefinitionNames()) {
                    Object bean = context.getBean(name);
                    for (Method method : bean.getClass().getMethods()) {
                        if (method.isAnnotationPresent(MyListener.class)) {
                            ApplicationListener<MyEvent> applicationListener = new ApplicationListener() {
                                @Override
                                public void onApplicationEvent(ApplicationEvent event) {
                                    System.out.println(event);
                                    Class<?> eventType = method.getParameterTypes()[0];
                                    if (eventType.isAssignableFrom(event.getClass())) {
                                        try {
                                            method.invoke(bean, event);
                                        } catch (Exception e) {
                                            e.printStackTrace();
                                        }
                                    }
                                }
                            };
                            context.addApplicationListener(applicationListener);
                        }
                    }
                }
            }
        };
    }

    @Component
    static class MyService {
        private static final Logger log = LoggerFactory.getLogger(MyService.class);
        @Autowired
        private ApplicationEventPublisher publisher; // applicationContext

        public void doBusiness() {
            log.debug("主线业务");
            // 主线业务完成后需要做一些支线业务，下面是问题代码
            publisher.publishEvent(new MyEvent("MyService.doBusiness()"));
        }
    }

    @Component
    static class SmsService {
        private static final Logger log = LoggerFactory.getLogger(SmsService.class);

        @MyListener
        public void listener(MyEvent myEvent) {
            log.debug("发送短信");
        }
    }

    @Component
    static class EmailService {
        private static final Logger log = LoggerFactory.getLogger(EmailService.class);

        @MyListener
        public void listener(MyEvent myEvent) {
            log.debug("发送邮件");
        }
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @interface MyListener {
    }

    static class MyEvent extends ApplicationEvent {
        public MyEvent(Object source) {
            super(source);
        }
    }

}
```

#### 收获

事件监听器的两种方式

1）实现 ApplicationListener 接口

* 根据接口泛型确定事件类型

2）@EventListener 标注监听方法

* 根据监听器方法参数确定事件类型
* 解析时机：在 SmartInitializingSingleton（所有单例初始化完成后），解析每个单例 bean

### 49. 事件发布器

**事件发布器模拟实现**

```java
@Configuration
public class A49 {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A49.class);
        context.getBean(MyService.class).doBusiness();
        context.close();
    }

    static class MyEvent extends ApplicationEvent {
        public MyEvent(Object source) {
            super(source);
        }
    }

    @Component
    static class MyService {
        private static final Logger log = LoggerFactory.getLogger(MyService.class);
        @Autowired
        private ApplicationEventPublisher publisher; // applicationContext

        public void doBusiness() {
            log.debug("主线业务");
            // 主线业务完成后需要做一些支线业务，下面是问题代码
            publisher.publishEvent(new MyEvent("MyService.doBusiness()"));
        }
    }

    @Component
    static class SmsApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(SmsApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送短信");
        }
    }

    @Component
    static class EmailApplicationListener implements ApplicationListener<MyEvent> {
        private static final Logger log = LoggerFactory.getLogger(EmailApplicationListener.class);

        @Override
        public void onApplicationEvent(MyEvent event) {
            log.debug("发送邮件");
        }
    }

    @Bean
    public ThreadPoolTaskExecutor executor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        return executor;
    }

    /**
     * 事件发布器主要关注的方法
     *
     * @param context
     * @param executor
     * @return
     */
    @Bean
    public ApplicationEventMulticaster applicationEventMulticaster(ConfigurableApplicationContext context,
        ThreadPoolTaskExecutor executor) {
        return new AbstractApplicationEventMulticaster() {
            private List<GenericApplicationListener> listeners = new ArrayList<>();

            /**
             * 收集监听器
             * @param name
             */
            @Override
            public void addApplicationListenerBean(String name) {
                ApplicationListener listener = context.getBean(name, ApplicationListener.class);
                System.out.println(listener);
                // 获取类对应的接口的泛型类型，即：获取该监听器支持的事件类型
                ResolvableType type = ResolvableType.forClass(listener.getClass()).getInterfaces()[0].getGeneric();
                System.out.println(type);

                // 将原始的 listener 封装为支持事件类型检查的 listener
                GenericApplicationListener genericApplicationListener = new GenericApplicationListener() {
                    // 是否支持某事件类型                真实的事件类型
                    public boolean supportsEventType(ResolvableType eventType) {
                        return type.isAssignableFrom(eventType);
                    }

                    public void onApplicationEvent(ApplicationEvent event) {
                        executor.submit(() -> listener.onApplicationEvent(event));
                    }
                };

                listeners.add(genericApplicationListener);
            }

            /**
             * 发布事件
             * @param event
             * @param eventType
             */
            @Override
            public void multicastEvent(ApplicationEvent event, ResolvableType eventType) {
                for (GenericApplicationListener listener : listeners) {
                    if (listener.supportsEventType(ResolvableType.forClass(event.getClass()))) {
                        listener.onApplicationEvent(event);
                    }
                }
            }
        };
    }

    abstract static class AbstractApplicationEventMulticaster implements ApplicationEventMulticaster {

        @Override
        public void addApplicationListener(ApplicationListener<?> listener) {

        }

        @Override
        public void addApplicationListenerBean(String listenerBeanName) {

        }

        @Override
        public void removeApplicationListener(ApplicationListener<?> listener) {

        }

        @Override
        public void removeApplicationListenerBean(String listenerBeanName) {

        }

        @Override
        public void removeApplicationListeners(Predicate<ApplicationListener<?>> predicate) {

        }

        @Override
        public void removeApplicationListenerBeans(Predicate<String> predicate) {

        }

        @Override
        public void removeAllListeners() {

        }

        @Override
        public void multicastEvent(ApplicationEvent event) {

        }

        @Override
        public void multicastEvent(ApplicationEvent event, ResolvableType eventType) {

        }
    }
}
```

**收获**

addApplicationListenerBean 负责收集容器中的监听器
* 监听器会统一转换为 GenericApplicationListener 对象，以支持判断事件类型

multicastEvent 遍历监听器集合，发布事件
* 发布前先通过 GenericApplicationListener.supportsEventType 判断支持该事件类型才发事件
* 可以利用线程池进行异步发事件优化

如果发送的事件对象不是 ApplicationEvent 类型，Spring 会把它包装为 PayloadApplicationEvent 并用泛型技术解析事件对象的原始类型
* 视频中未讲解

