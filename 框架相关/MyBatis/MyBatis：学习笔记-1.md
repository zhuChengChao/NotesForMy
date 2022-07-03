# MyBatis：学习笔记-1

> 后端学习ING，在学完了Java语法部分，JavaWeb知识后，开始一套组合拳来带走经典框架SSM。
>
> 此文为第一拳：MyBatis学习笔记的 **1/4**。

## 1. 框架概述

### 1.1.1 什么是框架

框架（Framework）是整个或部分系统的可重用设计，表现为一组抽象构件及构件实例间交互的方法；另一种定义认为，框架是可被应用开发者定制的应用骨架。前者是从应用方面，后者是从目的方面给出的定义。

简而言之，框架其实就是某种应用的半成品，就是一组组件，供你选用完成你自己的系统。简单说就是使用别人搭好的舞台，你来做表演。而且，框架一般是成熟的，不断升级的软件。  

### 1.1.2 框架要解决的问题

框架要解决的最重要的一个问题是技术整合的问题，在 J2EE 的框架中，有着各种各样的技术，不同的软件企业需要从 J2EE 中选择不同的技术，这就使得软件企业最终的应用依赖于这些技术，技术自身的复杂性和技术的风险性将会直接对应用造成冲击。而应用是软件企业的核心，是竞争力的关键所在，因此应该将应用自身的设计和具体的实现技术解耦。这样，软件企业的研发将集中在应用的设计上，而不是具体的技术实现，技术实现是应用的底层支撑，它不应该直接对应用产生影响。

**框架一般处在低层应用平台（如 J2EE）和高层业务逻辑之间的中间层。**  

### 1.1.3 软件开发的分层重要性

框架的重要性在于它实现了部分功能，并且能够很好的将低层应用平台和高层业务逻辑进行了缓和。为了实现软件工程中的“**高内聚、低耦合**”。把问题划分开来各个解决，易于控制，易于延展，易于分配资源。 

常见的 MVC 软件设计思想就是很好的分层思想。  

![MVC设计思想](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252255475.PNG)

通过分层更好的实现了各个部分的职责，在每一层将再细化出不同的框架，分别解决各层关注的问题。  

### 1.1.4 分层开发下的常见框架

常见的 JavaEE 开发框架：

1. 解决数据的持久化问题的框架：MyBatis，Hibernate，Spring Data；
2. 解决 WEB 层问题的 MVC 框架：SpringMVC；
3. 解决技术整合问题的框架：Spring；

### 1.1.5 MyBatis框架概述

MyBatis 是一个优秀的基于 java 的持久层框架，它内部封装了 JDBC，**使开发者只需要关注 SQL语句本身**，而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。

MyBatis 通过 XML 或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 SQL的动态参数进行映射生成最终执行的 SQL语句，最后由 MyBatis 框架执行 SQL 并将结果映射为 java 对象并返回。

采用 ORM（Object Relation Model） 思想解决了实体和数据库映射的问题，对 JDBC 进行了封装，屏蔽了 JDBC API 底层访问细节，使我们不用与 JDBC API  打交道，就可以完成对数据库的持久化操作。

为了能够更好掌握框架运行的内部过程，并且有更好的体验，下面将从自定义 MyBatis 框架开始来学习框架。体验框架从无到有的过程体验，也能够很好的综合前面阶段所学的基础。  

**简单的说：MyBatis就是把数据库表和实体类及实体类的属性对应起来，让我们可以操作实体类就实现操作数据库表。**

## 1.2 JDBC编程的分析

### 1.2.1 JDBC程序的回顾

```java
public static void main(String[] args) {
    
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;
    
    try {
        //加载数据库驱动
        Class.forName("com.mysql.jdbc.Driver");
        //通过驱动管理类获取数据库链接
        connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8","root", "root");
        //定义 sql 语句 ?表示占位符
        String sql = "select * from user where username = ?";
        //获取预处理 statement
        preparedStatement = connection.prepareStatement(sql);
        //设置参数，第一个参数为 sql 语句中参数的序号（从 1 开始），第二个参数为设置的参数值
        preparedStatement.setString(1, "王五");
        //向数据库发出 sql 执行查询，查询出结果集
        resultSet = preparedStatement.executeQuery();
        //遍历查询结果集
        while(resultSet.next()){
            System.out.println(resultSet.getString("id")+" "+resultSet.getString("username"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    }finally{
        //释放资源
        if(resultSet!=null){
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(preparedStatement!=null){
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(connection!=null){
            try {
                connection.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}
```

上边使用 JDBC 的原始方法（未经封装）实现了查询数据库表记录的操作。

### 1.2.2 JDBC问题分析

1. 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
2. SQL语句在代码中硬编码，造成代码不易维护，实际应用 SQL 变化的可能较大，SQL变动需要改变java代码。
3. 使用 preparedStatement 向占有位符号传参数存在硬编码，因为SQL语句的 where 条件不一定，可能多也可能少，修改SQL还要修改代码，系统不易维护。
4. 对结果集解析存在硬编码（查询列名），SQL变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成 POJO 对象解析比较方便。  

## 2. MyBatis框架快速入门

### 2.1 MyBatis框架开发准备

这里使用Maven，仅需在建立工程的 pom.xml 导包即可：

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>x.x.x</version>
</dependency>
```

### 2.2 搭建MyBatis开发环境

创建 mybatis01 的工程，工程信息如下：

```
Groupid:cn.xyc
ArtifactId:mybatis01
Packing:jar
```

#### 2.2.2 添加MyBatis坐标

在 pom.xml 文件中添加 MyBatis的坐标，如下：  

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
        <scope>test</scope>
    </dependency>
    <!--mysql的驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
        <scope>runtime</scope>
    </dependency>
    <!-- 日志记录 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.12</version>
    </dependency>
</dependencies>
```

#### 2.2.3 编写User实体类

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    
    // 省略 get/set/toString方法    
}
```

#### 2.2.4 编写持久层接口IUserDao

IUserDao 接口就是持久层接口（也可以写成 UserDao 或者 UserMapper），具体代码如下：  

```java
public interface IUserDao {
    
    // 查询所有用户
    List<User> findAll();
}
```

#### 2.2.5 编写持久层接口的映射文件

编写持久层接口的映射文件：IUserDao.xml  

**要求：**

1. 创建位置：**必须和持久层接口在相同的包中**。
2. 名称：必须以持久层接口名称命名文件名，扩展名是 `.xml` 

例如此处：

1. IUserDao接口的位置为：`main/java/cn/xyc/dao/IUserDao`
2. IUserDao.xml接口的位置为：`main/resources/cn/xyc/dao/IUserDao.xml`

**文件内容如下**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.xyc.dao.IUserDao">
    <!-- 配置查询所有操作 -->
    <select id="findAll" resultType="cn.xyc.domain.User">
        select * from user
    </select>
</mapper>
```

#### 2.2.6 编写 SqlMapConfig.xml 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- mybatis的主配置文件 -->
<configuration>

    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <mapper resource="cn/xyc/dao/IUserDao.xml"/>
    </mappers>

</configuration>
```

#### 2.2.7 编写测试类

```java
public class MybatisTest {

    public static void main(String[] args) throws Exception {

        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //4.使用SqlSession创建Dao接口的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
        //6.释放资源
        session.close();
        in.close();

    }
}
```

#### 2.2.7 小结

通过快速入门示例，可以发现使用 mybatis 是非常容易的一件事情，因为只需要编写 Dao 接口并且按照 mybatis 要求编写两个配置文件，就可以实现功能。远比之前的 jdbc 方便多了。

> 在使用注解之后，将变得更为简单，只需要编写一个 mybatis 配置文件就够了。

但是，这里面包含了许多细节，比如为什么会有工厂对象（SqlSessionFactory），为什么有了工厂之后还要有构建者对象（SqlSessionFactoryBuilder），为什么 IUserDao.xml 在创建时有位置和文件名的要求等等。

这些问题在自定义 mybatis 框架的部分，通过层层剥离的方式进行解释。

> 注： 自定义 Mybatis 框架，是为能更好了解 mybatis 内部是怎么执行的，在以后的开发中能更好的使用 mybatis 框架，同时对它的设计理念（设计模式）有一个认识。  

### 2.4 基于注解的MyBatis使用

#### 2.4.1 在持久层接口中添加注解

```java
public interface IUserDao {

    // 查询所有操作
    @Select("select * from user")
    List<User> findAll();
}
```

#### 2.4.2 修改 SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- mybatis的主配置文件 -->
<configuration>

    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <!--<mappers>-->
        <!--<mapper resource="cn/xyc/dao/IUserDao.xml"/>-->
    <!--</mappers>-->

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件
        如果是用注解来配置的话，此处应该使用class属性指定被注解的dao全限定类名-->
    <mappers>
        <mapper class="cn.xyc.dao.IUserDao"/>
    </mappers>

</configuration>
```

#### 2.4.3 注意事项

在使用基于注解的 Mybatis 配置时，请移除 xml 的映射配置（IUserDao.xml）。  

## 3. 自定义MyBatis框架

### 3.1 自定义MyBatis框架分析

**涉及知识点介绍**：

本章将使用前面所学的基础知识来构建一个属于自己的持久层框架，将会涉及到的一些知识点：工厂模式（Factory 工厂模式）、构造者模式（Builder 模式）、代理模式、反射、自定义注解、注解的反射、xml 解析、数据库元数据、元数据的反射等。  

**分析流程：**

![自定义Mybatis分析](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252255232.png)

### 3.2 前期准备

#### 3.2.1 创建工程&引入坐标

**创建工程**

```
Groupid:cn.xyc
ArtifactId:mybatis02
Packing:jar
```

**引入相关坐标**

```xml
<dependencies>
    <!-- 日志坐标 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.12</version>
    </dependency>
    <!-- 解析 xml 的 dom4j -->
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <!-- mysql 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
    </dependency>
    <!-- dom4j 的依赖包 jaxen -->
    <dependency>
        <groupId>jaxen</groupId>
        <artifactId>jaxen</artifactId>
        <version>1.1.6</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
</dependencies>
```

#### 3.2.2 引入工具类到项目中

* **用于解析配置文件的：XMLConfigBuilder**

```java
public class XMLConfigBuilder {
    
    /**
     * 解析主配置文件，把里面的内容填充到DefaultSqlSession所需要的地方
     * 使用的技术：dom4j+xpath
     */
    public static Configuration loadConfiguration(InputStream config){
        
        try{
            //定义封装连接信息的配置对象（mybatis的配置对象）
            Configuration cfg = new Configuration();

            //1.获取SAXReader对象
            SAXReader reader = new SAXReader();
            //2.根据字节输入流获取Document对象
            Document document = reader.read(config);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.使用xpath中选择指定节点的方式，获取所有property节点
            List<Element> propertyElements = root.selectNodes("//property");
            //5.遍历节点
            for(Element propertyElement : propertyElements){
                //判断节点是连接数据库的哪部分信息
                //取出name属性的值
                String name = propertyElement.attributeValue("name");
                if("driver".equals(name)){
                    //表示驱动
                    //获取property标签value属性的值
                    String driver = propertyElement.attributeValue("value");
                    cfg.setDriver(driver);
                }
                if("url".equals(name)){
                    //表示连接字符串
                    //获取property标签value属性的值
                    String url = propertyElement.attributeValue("value");
                    cfg.setUrl(url);
                }
                if("username".equals(name)){
                    //表示用户名
                    //获取property标签value属性的值
                    String username = propertyElement.attributeValue("value");
                    cfg.setUsername(username);
                }
                if("password".equals(name)){
                    //表示密码
                    //获取property标签value属性的值
                    String password = propertyElement.attributeValue("value");
                    cfg.setPassword(password);
                }
            }
            //取出mappers中的所有mapper标签，判断他们使用了resource还是class属性
            List<Element> mapperElements = root.selectNodes("//mappers/mapper");
            //遍历集合
            for(Element mapperElement : mapperElements){
                //判断mapperElement使用的是哪个属性
                Attribute attribute = mapperElement.attribute("resource");
                if(attribute != null){
                    System.out.println("使用的是XML");
                    //表示有resource属性，用的是XML
                    //取出属性的值
                    String mapperPath = attribute.getValue();//获取属性的值"com/itheima/dao/IUserDao.xml"
                    //把映射配置文件的内容获取出来，封装成一个map
                    Map<String,Mapper> mappers = loadMapperConfiguration(mapperPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }else{
                    System.out.println("使用的是注解");
                    //表示没有resource属性，用的是注解
                    //获取class属性的值
                    String daoClassPath = mapperElement.attributeValue("class");
                    //根据daoClassPath获取封装的必要信息
                    Map<String,Mapper> mappers = loadMapperAnnotation(daoClassPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }
            }
            //返回Configuration
            return cfg;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            try {
                config.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }

    /**
     * 根据传入的参数，解析XML，并且封装到Map中
     * @param mapperPath    映射配置文件的位置
     * @return  map中包含了获取的唯一标识（key是由dao的全限定类名和方法名组成）
     *     以及执行所需的必要信息（value是一个Mapper对象，里面存放的是执行的SQL语句和要封装的实体类全限定类名）
     */
    private static Map<String,Mapper> loadMapperConfiguration(String mapperPath)throws IOException {
        
        InputStream in = null;
        
        try{
            //定义返回值对象
            Map<String,Mapper> mappers = new HashMap<String,Mapper>();
            //1.根据路径获取字节输入流
            in = Resources.getResourceAsStream(mapperPath);
            //2.根据字节输入流获取Document对象
            SAXReader reader = new SAXReader();
            Document document = reader.read(in);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.获取根节点的namespace属性取值
            String namespace = root.attributeValue("namespace");//是组成map中key的部分
            //5.获取所有的select节点
            List<Element> selectElements = root.selectNodes("//select");
            //6.遍历select节点集合
            for(Element selectElement : selectElements){
                //取出id属性的值      组成map中key的部分
                String id = selectElement.attributeValue("id");
                //取出resultType属性的值  组成map中value的部分
                String resultType = selectElement.attributeValue("resultType");
                //取出文本内容            组成map中value的部分
                String queryString = selectElement.getText();
                //创建Key
                String key = namespace+"."+id;
                //创建Value
                Mapper mapper = new Mapper();
                mapper.setQueryString(queryString);
                mapper.setResultType(resultType);
                //把key和value存入mappers中
                mappers.put(key,mapper);
            }
            return mappers;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            in.close();
        }
    }

    /**
     * 根据传入的参数，得到dao中所有被select注解标注的方法。
     * 根据方法名称和类名，以及方法上注解value属性的值，组成Mapper的必要信息
     * @param daoClassPath
     * @return
     */
    private static Map<String,Mapper> loadMapperAnnotation(String daoClassPath)throws Exception{
        //定义返回值对象
        Map<String,Mapper> mappers = new HashMap<String, Mapper>();

        //1.得到dao接口的字节码对象
        Class daoClass = Class.forName(daoClassPath);
        //2.得到dao接口中的方法数组
        Method[] methods = daoClass.getMethods();
        //3.遍历Method数组
        for(Method method : methods){
            //取出每一个方法，判断是否有select注解
            boolean isAnnotated = method.isAnnotationPresent(Select.class);
            if(isAnnotated){
                //创建Mapper对象
                Mapper mapper = new Mapper();
                //取出注解的value属性值
                Select selectAnno = method.getAnnotation(Select.class);
                String queryString = selectAnno.value();
                mapper.setQueryString(queryString);
                //获取当前方法的返回值，还要求必须带有泛型信息
                Type type = method.getGenericReturnType();//List<User>
                //判断type是不是参数化的类型
                if(type instanceof ParameterizedType){
                    //强转
                    ParameterizedType ptype = (ParameterizedType)type;
                    //得到参数化类型中的实际类型参数
                    Type[] types = ptype.getActualTypeArguments();
                    //取出第一个
                    Class domainClass = (Class)types[0];
                    //获取domainClass的类名
                    String resultType = domainClass.getName();
                    //给Mapper赋值
                    mapper.setResultType(resultType);
                }
                //组装key的信息
                //获取方法的名称
                String methodName = method.getName();
                String className = method.getDeclaringClass().getName();
                String key = className+"."+methodName;
                //给map赋值
                mappers.put(key,mapper);
            }
        }
        return mappers;
    }
}
```

* **负责执行SQL语句，并且封装结果集：Executor**

```java
public class Executor {

    public <E> List<E> selectList(Mapper mapper, Connection conn) {
        PreparedStatement pstm = null;
        ResultSet rs = null;
        try {
            //1.取出mapper中的数据
            String queryString = mapper.getQueryString();//select * from user
            String resultType = mapper.getResultType();//cn.xyc.domain.User
            Class domainClass = Class.forName(resultType);
            //2.获取PreparedStatement对象
            pstm = conn.prepareStatement(queryString);
            //3.执行SQL语句，获取结果集
            rs = pstm.executeQuery();
            //4.封装结果集
            List<E> list = new ArrayList<E>();//定义返回值
            while(rs.next()) {
                //实例化要封装的实体类对象
                E obj = (E)domainClass.newInstance();

                //取出结果集的元信息：ResultSetMetaData
                ResultSetMetaData rsmd = rs.getMetaData();
                //取出总列数
                int columnCount = rsmd.getColumnCount();
                //遍历总列数
                for (int i = 1; i <= columnCount; i++) {
                    //获取每列的名称，列名的序号是从1开始的
                    String columnName = rsmd.getColumnName(i);
                    //根据得到列名，获取每列的值
                    Object columnValue = rs.getObject(columnName);
                    //给obj赋值：使用Java内省机制（借助PropertyDescriptor实现属性的封装）
                    PropertyDescriptor pd = new PropertyDescriptor(columnName,domainClass);//要求：实体类的属性和数据库表的列名保持一种
                    //获取它的写入方法
                    Method writeMethod = pd.getWriteMethod();
                    //把获取的列的值，给对象赋值
                    writeMethod.invoke(obj,columnValue);
                }
                //把赋好值的对象加入到集合中
                list.add(obj);
            }
            return list;
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            release(pstm,rs);
        }
    }

    private void release(PreparedStatement pstm,ResultSet rs){
        if(rs != null){
            try {
                rs.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }

        if(pstm != null){
            try {
                pstm.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

* **数据源的工具类：DataSourceUtil**

```java
public class DataSourceUtil {

    /**
     * 用于获取一个连接
     * @param cfg
     * @return
     */
    public static Connection getConnection(Configuration cfg){
        try {
            Class.forName(cfg.getDriver());
            return DriverManager.getConnection(cfg.getUrl(), cfg.getUsername(), cfg.getPassword());
        }catch(Exception e){
            throw new RuntimeException(e);
        }
    }
}
```

#### 3.2.3 编写SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" ></property>
                <property name="url" value="jdbc:mysql:///db1" ></property>
                <property name="username" value="root"></property>
                <property name="password" value="root"></property>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

> 注：此处此处使用的是 mybatis 的配置文件，但是由于此时没有使用 mybatis 的 jar 包，所以要把配置文件的约束删掉否则会报错

#### 3.2.4 编写读取配置文件类

```java
public class Resources {

    /**
      * 用于加载 xml 文件，并且得到一个流对象
      * @param xmlPath
      * @return
      * 在实际开发中读取配置文件:
      * 第一：使用类加载器。但是有要求： a 文件不能过大。 b 文件必须在类路径下(classpath)
      * 第二：使用 ServletContext 的 getRealPath()
      */
    public static InputStream getResourceAsStream(String filePath){
        return Resources.class.getClassLoader().getResourceAsStream(filePath);
    }
}
```

#### 3.2.5 编写Mapper类

```java
public class Mapper {

    private String queryString;//SQL
    private String resultType;//实体类的全限定类名

    public String getQueryString() {
        return queryString;
    }

    public void setQueryString(String queryString) {
        this.queryString = queryString;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }
}
```

#### 3.2.6 编写Configuration配置类

```java
/**
  * 核心配置类
  * 1.数据库信息
  * 2.sql 的 map 集合
  */
public class Configuration {

    private String driver;
    private String url;
    private String username;
    private String password;

    private Map<String,Mapper> mappers = new HashMap<String,Mapper>();

    public Map<String, Mapper> getMappers() {
        return mappers;
    }

    public void setMappers(Map<String, Mapper> mappers) {
        this.mappers.putAll(mappers);//此处需要使用追加的方式
    }

    public String getDriver() {
        return driver;
    }

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

#### 3.2.7 编写User实体类

```java
public class User implements Serializable {
    private int id;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址
    //省略 getter & setter & toString
}
```

### 3.3 基于XML的自定义MyBatis框架

#### 3.3.1 编写持久层接口和 IUserDao.xml

* 用户的持久层接口

```java
public interface IUserDao {

    // 查询所有操作
    @Select("select * from user")
    List<User> findAll();
}
```

* IUserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mapper namespace="cn.xyc.dao.IUserDao">
    <!-- 配置查询所有操作 -->
    <select id="findAll" resultType="cn.xyc.domain.User">
        select * from user
    </select>
</mapper>
```

> 此处使用的也是 mybatis 的配置文件，所以也要把约束删除了  

#### 3.3.2 编写构建者类

```java
public class SqlSessionFactoryBuilder {
    /**
      * 根据传入的流，实现对 SqlSessionFactory 的创建
      * @param in 它就是 SqlMapConfig.xml 的配置以及里面包含的 IUserDao.xml 的配置
      * @return
      */
    public SqlSessionFactory build(InputStream in) {
        DefaultSqlSessionFactory factory = new DefaultSqlSessionFactory();
        //给 factory 中 config 赋值
        factory.setConfig(in);
        return factory;
    }
}
```

#### 3.3.3 编写SqlSessionFactory接口和实现类

```java
public interface SqlSessionFactory {
    /**
      * 创建一个新的 SqlSession 对象
      * @return
      */
    SqlSession openSession();
}

public class DefaultSqlSessionFactory implements SqlSessionFactory {
    private InputStream config = null;
    
    public void setConfig(InputStream config) {
        this.config = config;
    }
    
    @Override
    public SqlSession openSession() {
        DefaultSqlSession session = new DefaultSqlSession();
        //调用工具类解析 xml 文件
        XMLConfigBuilder.loadConfiguration(session, config);
        return session;
    }
}
```

#### 3.3.4 编写SqlSession接口和实现类

* 自定义Mybatis中和数据库交互的核心类，它里面可以创建dao接口的代理对象

```java
public interface SqlSession {

    /**
     * 根据参数创建一个代理对象
     * @param daoInterfaceClass dao的接口字节码
     * @param <T>
     * @return
     */
    <T> T getMapper(Class<T> daoInterfaceClass);

    /**
     * 释放资源
     */
    void close();
}
```

* SqlSession 的具体实现  

```java
public class DefaultSqlSession implements SqlSession {
	
    //核心配置对象
    private Configuration cfg;
    //连接对象
    private Connection connection;

    public DefaultSqlSession(Configuration cfg){
        this.cfg = cfg;
        //调用 DataSourceUtils 工具类获取连接
        connection = DataSourceUtil.getConnection(cfg);
    }

    /**
     * 用于创建代理对象(动态代理)
     * @param daoInterfaceClass dao的接口字节码
     * @param <T>
     * @return
     */
    @Override
    public <T> T getMapper(Class<T> daoInterfaceClass) {
        return (T) Proxy.newProxyInstance(daoInterfaceClass.getClassLoader(),
                new Class[]{daoInterfaceClass},new MapperProxy(cfg.getMappers(),connection));
    }

    /**
     * 用于释放资源
     */
    @Override
    public void close() {
        if(connection != null) {
            try {
                connection.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    //查询所有方法
    public <E> List<E> selectList(String statement){
        Mapper mapper = cfg.getMappers().get(statement);
        return new Executor().selectList(mapper,conn);
    }
}
```

#### 3.3.5 编写用于创建Dao接口代理对象的类

```java
public class MapperProxyFactory implements InvocationHandler {
    
    private Map<String,Mapper> mappers;
    private Connection conn;
    
    public MapperProxyFactory(Map<String, Mapper> mappers,Connection conn) {
        this.mappers = mappers;
        this.conn = conn;
    }
    
    /**
      * 对当前正在执行的方法进行增强
      * 取出当前执行的方法名称
      * 取出当前执行的方法所在类
      * 拼接成 key
      * 去 Map 中获取 Value（Mapper)
      * 使用工具类 Executor 的 selectList 方法
      */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
    {
        //1.取出方法名
        String methodName = method.getName();
        //2.取出方法所在类名
        String className = method.getDeclaringClass().getName();
        //3.拼接成 Key
        String key = className+"."+methodName;
        //4.使用 key 取出 mapper
        Mapper mapper = mappers.get(key);
        if(mapper == null) {
            throw new IllegalArgumentException("传入的参数有误，无法获取执行的必要条件");
        }
        //5.创建 Executor 对象
        Executor executor = new Executor();
        return executor.selectList(mapper, conn);
    }
}
```

#### 3.3.6 运行测试类

```java
public class MybatisTest {
    public static void main(String[] args)throws Exception {
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建 SqlSessionFactory 的构建者对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //3.使用构建者创建工厂对象 SqlSessionFactory
        SqlSessionFactory factory = builder.build(in);
        //4.使用 SqlSessionFactory 生产 SqlSession 对象
        SqlSession session = factory.openSession();
        //5.使用 SqlSession 创建 dao 接口的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
        //6.使用代理对象执行查询所有方法
        List<User> users = userDao.findAll();
        for(User user : users) {
            System.out.println(user);
        }
        //7.释放资源
        session.close();
        in.close();
    }
}
```

### 3.4 基于注解方式定义MyBatis框架

#### 3.4.1 自定义@Select注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Select {
    String value();
}
```

#### 3.4.2 修改持久层接口

```java
public interface IUserDao {
    
    // 查询所有用户
    @Select("select * from user")
    List<User> findAll();
}
```

#### 3.4.3 修改 SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 配置 mybatis 的环境 -->
    <environments default="mysql">
        <!-- 配置 mysql 的环境 -->
        <environment id="mysql">
            <!-- 配置事务的类型 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置连接数据库的信息：用的是数据源(连接池) -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 告知 mybatis 映射配置的位置 -->
    <mappers>
        <mapper class="cn.xyc.dao.IUserDao"/>
    </mappers>
</configuration>
```

### 3.5 自定义MyBatis的设计模式说明

1. 工厂模式（SqlSessionFactory）
2. 代理模式（MapperProxyFactory）
3. 构建者模式（SqlSessionFactoryBuilder）

> 设计模式后续再单独学习...

### 3.6 总结

![自定义mybatis开发流程图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252255380.png)