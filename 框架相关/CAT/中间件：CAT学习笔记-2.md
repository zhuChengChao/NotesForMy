# 中间件：CAT学习笔记-2

> 实习过程中了解到存在调用链监控这种好用的工具，就想稍微系统的学习一下，发现也有相应的视频课程：[Java进阶教程Cat入门](https://www.bilibili.com/video/BV1m64y127f3)，就顺道看了一波，跟着课程内容做了下学习笔记，此为 2/3 篇。

# 3. CAT高级

## 3.1 框架集成

### 3.1.1 dubbo

#### 3.1.1.1 制作 cat-dubbo 插件

> 使用 idea 打开 cat 源码，找到 integration 目录，右键点击如下：...
>
> 然后使用 install 命令将插件安装到本地仓库：...
>
> 笑死...这个包的依赖太麻烦了，还不如直接下 jar 包，然而根本下不到...吐血

接下来我们就可以使用如下依赖自动引入dubbo插件了。

```xml
<dependency>
    <groupId>net.dubboclub</groupId>
    <artifactId>cat-monitor</artifactId>
    <version>0.0.6</version>
</dependency>
```

#### 3.1.1.2 服务提供方

pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itcast</groupId>
    <artifactId>dubbo-provider-cat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>dubbo-provider-cat</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--添加springboot和dubbo集成配置-->
        <dependency>
            <groupId>com.alibaba.spring.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>

        <dependency>
            <groupId>net.dubboclub</groupId>
            <artifactId>cat-monitor</artifactId>
            <version>0.0.6</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

> 这里直接使用了`dubbo-spring-boot-starter`这一 dubbo 与 spring-boot 集成的组件。
>
> 官方文档地址：https://github.com/alibaba/dubbo-spring-boot-starter/blob/master/README_zh.md

application.properties:

```properties
# tomcat启动的端口
server.port=7072
# 应用名
spring.application.name=dubbo_provider_cat
# 服务提供方
spring.dubbo.server=true  
# 为了简化环境搭建，采用了本地直接调用的方式，所以将注册中心写成N/A表示不注册到注册中心
spring.dubbo.registry=N/A
```

HelloService接口：简化项目的开发，将HelloService接口在消费方和提供方都编写一份

```java
package com.itcast.api;

public interface HelloService {
    public String hello();
}
```

HelloServiceImpl实现类：

```java
package com.itcast.dubboprovidercat.service;

import com.alibaba.dubbo.config.annotation.Service;
import com.itcast.api.HelloService;
import org.springframework.stereotype.Component;

@Component
@Service(interfaceClass = HelloService.class)  // 用的不是spring的注解，而是dubbo的注解
public class HelloServiceImpl implements HelloService {

    public String hello() {
        return "hello cat";
    }
}
```

DubboProviderCatApplication 启动类:

```java
package com.itcast.dubboprovidercat;

import com.alibaba.dubbo.spring.boot.annotation.EnableDubboConfiguration;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubboConfiguration  // 需要添加@EnableDubboConfiguration注解
public class DubboProviderCatApplication {

	public static void main(String[] args) {
		SpringApplication.run(DubboProviderCatApplication.class, args);
	}
}
```

添加 CAT 应用名，添加文件：resources/MATE-INF/app.properties，内容为：`app.name=dubbo-provider-cat`

#### 3.1.1.3 服务消费方

pom文件：和服务提供方的基本一样

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itcast</groupId>
    <artifactId>dubbo-consumer-cat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>dubbo-consumer-cat</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--添加springboot和dubbo集成配置-->
        <dependency>
            <groupId>com.alibaba.spring.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>

        <dependency>
            <groupId>net.dubboclub</groupId>
            <artifactId>cat-monitor</artifactId>
            <version>0.0.6</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

application.properties:

```properties
# tomcat启动的端口
server.port=7074
# 应用名
spring.application.name=dubbo_consumer_cat
```

HelloService接口：简化项目的开发，将HelloService接口在消费方和提供方都编写一份

```java
package com.itcast.api;

public interface HelloService {
    public String hello();
}
```

TestController：

```java
package com.itcast.dubboconsumercat.controller;

import com.alibaba.dubbo.config.annotation.Reference;
import com.itcast.api.HelloService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    
    // 因为采用直连而非从注册中心获取服务地址的方式，在@Reference注解中声明了 URL
    @Reference(url = "dubbo://127.0.0.1:20880")
    private HelloService helloService;

    @GetMapping("/hello")
    public String hello(){
        return helloService.hello();
    }
}
```

DubboConsumerCatApplication 启动类:

```java
package com.itcast.dubboconsumercat;

import com.alibaba.dubbo.spring.boot.annotation.EnableDubboConfiguration;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubboConfiguration  // 需要添加@EnableDubboConfiguration注解
public class DubboConsumerCatApplication {

	public static void main(String[] args) {
		SpringApplication.run(DubboConsumerCatApplication.class, args);
	}
}
```

#### 3.1.1.4 测试

按照如下顺序启动相关应用：

1. 启动DubboProviderCatApplication

2. 启动DubboConsumerCatApplication

3. 访问地址：http://localhost:7074/hello

4. 查看cat页面

![cat-transaction](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261407412.png)

如图所示 dubbo 的调用已经被正确显示在 transaction 报表中。点击 log view 查看详细的调用。

![dubbo-logview](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261407089.png)

如图所示，调用的日志已经被成功打印。

>dubbo插件的日志打印内容显示的并不是十分的良好，如果在企业中应用，可以基于dubbo插件进行二次开发。

### 3.1.2 mybatis

#### 3.1.2.1 创建SpringBoot和mybatis的集成项目

表结构：

```sql
CREATE TABLE `t_user` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    `password` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicde_ci
```

pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itcast</groupId>
    <artifactId>mybatis-cat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>mybatis-cat</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
            <version>5.1.27</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.dianping.cat</groupId>
            <artifactId>cat-client</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

application.yml配置文件：

```yaml
# datasource
spring:
  datasource:
    url: jdbc:mysql:///springboot?serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

# mybatis
mybatis:
  # mapper映射文件路径
  mapper-locations: classpath:mapper/*Mapper.xml  
  # 实体类的别名
  type-aliases-package: com.itcast.mybatiscat.entity
  
# tomcat的接口
server:
  port: 7079
```

编写dao层：

```java
package com.itcast.mybatiscat.dao;

import com.itcast.mybatiscat.entity.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

import java.util.List;

@Mapper
@Repository
public interface UserXmlMapper {

    public List<User> findAll();
}
```

resources/mapper/userMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itcast.mybatiscat.dao.UserXmlMapper">
    <select id="findAll" resultType="user">
        select * from t_user1
    </select>
</mapper>
```

启动类：

```java
package com.itcast.mybatiscat;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MybatisCatApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisCatApplication.class, args);
    }

}
```

编写Controller进行测试：

```java
package com.itcast.mybatiscat.controller;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Event;
import com.dianping.cat.message.Transaction;
import com.itcast.mybatiscat.dao.UserXmlMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MybatisController {

    @Autowired
    private UserXmlMapper userXmlMapper;

    @RequestMapping("mybatis")
    public String test(){
        return  userXmlMapper.findAll().toString();
    }
}
```

#### 3.1.2.2 集成cat-mybatis插件：

将以下文件放置到项目中，该文件来源于 cat 的官方代码，通过拦截器的方式

```java
package com.itcast.mybatiscat.cat;

import com.alibaba.druid.pool.DruidDataSource;
import com.dianping.cat.Cat;
import com.dianping.cat.message.Message;
import com.dianping.cat.message.Transaction;
import org.apache.ibatis.datasource.pooled.PooledDataSource;
import org.apache.ibatis.datasource.unpooled.UnpooledDataSource;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.*;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;
import org.apache.ibatis.type.TypeHandlerRegistry;

import javax.sql.DataSource;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.text.DateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


/**
 *  1.Cat-Mybatis plugin:  Rewrite on the version of Steven;
 *  2.Support DruidDataSource, PooledDataSource(mybatis Self-contained data source);
 *  @author zhanzehui(west_20@163.com)
 */
@Intercepts({
    @Signature(method = "query", type = Executor.class, args = {
        MappedStatement.class, Object.class, RowBounds.class,
        ResultHandler.class }),
    @Signature(method = "update", type = Executor.class, args = { MappedStatement.class, Object.class })
})
public class CatMybatisPlugin implements Interceptor {

    private static final Pattern PARAMETER_PATTERN = Pattern.compile("\\?");
    private static final String MYSQL_DEFAULT_URL = "jdbc:mysql://UUUUUKnown:3306/%s?useUnicode=true";
    private Executor target;

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        MappedStatement mappedStatement = this.getStatement(invocation);
        String          methodName      = this.getMethodName(mappedStatement);
        Transaction t = Cat.newTransaction("SQL", methodName);

        String sql = this.getSql(invocation,mappedStatement);
        SqlCommandType sqlCommandType = mappedStatement.getSqlCommandType();
        Cat.logEvent("SQL.Method", sqlCommandType.name().toLowerCase(), Message.SUCCESS, sql);

        String url = this.getSQLDatabaseUrlByStatement(mappedStatement);
        Cat.logEvent("SQL.Database", url);

        return doFinish(invocation,t);
    }

    private MappedStatement getStatement(Invocation invocation) {
        return (MappedStatement)invocation.getArgs()[0];
    }

    private String getMethodName(MappedStatement mappedStatement) {
        String[] strArr = mappedStatement.getId().split("\\.");
        String methodName = strArr[strArr.length - 2] + "." + strArr[strArr.length - 1];

        return methodName;
    }

    private String getSql(Invocation invocation, MappedStatement mappedStatement) {
        Object parameter = null;
        if(invocation.getArgs().length > 1){
            parameter = invocation.getArgs()[1];
        }

        BoundSql boundSql = mappedStatement.getBoundSql(parameter);
        Configuration configuration = mappedStatement.getConfiguration();
        String sql = sqlResolve(configuration, boundSql);

        return sql;
    }

    private Object doFinish(Invocation invocation,Transaction t) throws InvocationTargetException, IllegalAccessException {
        Object returnObj = null;
        try {
            returnObj = invocation.proceed();
            t.setStatus(Transaction.SUCCESS);
        } catch (Exception e) {
            Cat.logError(e);
            throw e;
        } finally {
            t.complete();
        }

        return returnObj;
    }


    private String getSQLDatabaseUrlByStatement(MappedStatement mappedStatement) {
        String url = null;
        DataSource dataSource = null;
        try {
            Configuration configuration = mappedStatement.getConfiguration();
            Environment environment = configuration.getEnvironment();
            dataSource = environment.getDataSource();

            url = switchDataSource(dataSource);

            return url;
        } catch (NoSuchFieldException|IllegalAccessException|NullPointerException e) {
            Cat.logError(e);
        }

        Cat.logError(new Exception("UnSupport type of DataSource : "+dataSource.getClass().toString()));
        return MYSQL_DEFAULT_URL;
    }

    private String switchDataSource(DataSource dataSource) throws NoSuchFieldException, IllegalAccessException {
        String url = null;

        if(dataSource instanceof DruidDataSource) {
            url = ((DruidDataSource) dataSource).getUrl();
        }else if(dataSource instanceof PooledDataSource) {
            Field dataSource1 = dataSource.getClass().getDeclaredField("dataSource");
            dataSource1.setAccessible(true);
            UnpooledDataSource dataSource2 = (UnpooledDataSource)dataSource1.get(dataSource);
            url =dataSource2.getUrl();
        }else {
            //other dataSource expand
        }

        return url;
    }

    public String sqlResolve(Configuration configuration, BoundSql boundSql) {
        Object parameterObject = boundSql.getParameterObject();
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        String sql = boundSql.getSql().replaceAll("[\\s]+", " ");
        if (parameterMappings.size() > 0 && parameterObject != null) {
            TypeHandlerRegistry typeHandlerRegistry = configuration.getTypeHandlerRegistry();
            if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                sql = sql.replaceFirst("\\?", Matcher.quoteReplacement(resolveParameterValue(parameterObject)));

            } else {
                MetaObject metaObject = configuration.newMetaObject(parameterObject);
                Matcher matcher = PARAMETER_PATTERN.matcher(sql);
                StringBuffer sqlBuffer = new StringBuffer();
                for (ParameterMapping parameterMapping : parameterMappings) {
                    String propertyName = parameterMapping.getProperty();
                    Object obj = null;
                    if (metaObject.hasGetter(propertyName)) {
                        obj = metaObject.getValue(propertyName);
                    } else if (boundSql.hasAdditionalParameter(propertyName)) {
                        obj = boundSql.getAdditionalParameter(propertyName);
                    }
                    if (matcher.find()) {
                        matcher.appendReplacement(sqlBuffer, Matcher.quoteReplacement(resolveParameterValue(obj)));
                    }
                }
                matcher.appendTail(sqlBuffer);
                sql = sqlBuffer.toString();
            }
        }
        return sql;
    }

    private String resolveParameterValue(Object obj) {
        String value = null;
        if (obj instanceof String) {
            value = "'" + obj.toString() + "'";
        } else if (obj instanceof Date) {
            DateFormat formatter = DateFormat.getDateTimeInstance(DateFormat.DEFAULT, DateFormat.DEFAULT, Locale.CHINA);
            value = "'" + formatter.format((Date) obj) + "'";
        } else {
            if (obj != null) {
                value = obj.toString();
            } else {
                value = "";
            }

        }
        return value;
    }

    @Override
    public Object plugin(Object target) {
        if (target instanceof Executor) {
            this.target = (Executor) target;
            return Plugin.wrap(target, this);
        }
        return target;
    }

    @Override
    public void setProperties(Properties properties) {
    }

}
```

>将此文件和所有其他 cat 插件一同打包放到私有仓库上是一种更好的选择。

编写 mybatis-config.xml 配置文件，将文件放置在 resources/mybatis/ 文件夹下:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <plugins>
        <plugin interceptor="com.itcast.mybatiscat.cat.CatMybatisPlugin"></plugin>
    </plugins>
</configuration>
```

修改 application.yml 文件：

```yaml
# mybatis
mybatis:
  # mapper映射文件路径
  mapper-locations: classpath:mapper/*Mapper.xml 
  # 实体类的别名
  type-aliases-package: com.itcast.mybatiscat.entity
  # config-location:  指定mybatis的核心配置文件
  config-location: classpath:mybatis/mybatis-config.xml
```

#### 3.1.2.3 测试

访问接口http://localhost:7079/mybatis，然后如果我们将sql语句修改为错误的语句，再次访问接口，如下图所示：

![mybatis1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261407165.png)

已经能够显示出有部分语句执行错误，如果要查看具体的错误，点击Log View查看：

![mybatis2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261407855.png)

图中不止能看到具体的 sql 语句，也可以看到报错的堆栈信息。

### 3.1.3 日志框架

CAT 集成日志框架的思路大体上都类似，所以课程中采用 SpringBoot 默认的 logback 日志框架来进行讲解，如果使用了 log4j、log4j2 处理方式也是类似的。

#### 3.1.3.1 搭建环境

搭建Spring Boot初始化环境，只需要添加Web起步依赖，然后修改 pom.xml 文件，添加 CAT 客户端依赖:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itcast</groupId>
    <artifactId>logback-cat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>logback-cat</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
		<!-- 添加 CAT 客户端依赖 -->
        <dependency>
            <groupId>com.dianping.cat</groupId>
            <artifactId>cat-client</artifactId>
            <version>3.0.0</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

删除 application.properties，使用 application.yml：

```yaml
# 日志的配置
logging:
  level:
    root: info
  path: ./logs
  config: classpath:logback-spring.xml
# tomcat的接口
server:
  port: 7080
```

编写配置文件 logback-spring.xml，放在resources目录下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <!-- 属性文件:在properties文件中找到对应的配置项 -->
    <springProperty scope="context" name="logging.path" source="logging.path"/>
    <contextName>cat</contextName>
    
    <!-- 控制台日志 -->
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出（配色）：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度，%msg：日志消息，%n是换行符-->
            <pattern>
                %yellow(%d{yyyy-MM-dd HH:mm:ss}) %red([%thread]) %highlight(%-5level) %cyan(%logger{50}) - %magenta(%msg) %n
            </pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--根据日志级别分离日志，分别输出到不同的文件-->
    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--按时间保存日志 修改格式可以按小时、按天、月来保存-->
            <fileNamePattern>${logging.path}/cat.info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--保存时长-->
            <MaxHistory>90</MaxHistory>
            <!--文件大小-->
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>${logging.path}/cat.error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <MaxHistory>90</MaxHistory>
        </rollingPolicy>
    </appender>
    
    <root level="info">
        <appender-ref ref="consoleLog"/>
        <appender-ref ref="fileInfoLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>
</configuration>
```

编写Controller接口：

```java
package com.itcast.logbackcat.controller;

import com.dianping.cat.Cat;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LogbackController {

    private Logger log = LoggerFactory.getLogger(LogbackController.class);

    @RequestMapping("logback")
    public String test(){
        log.info("cat info");
        try {
            int i = 1/0;
        }catch (Exception e){
            log.error("cat error",e);
        }
        return "logback";
    }
}
```

#### 3.1.3.2 集成CAT

创建 CatLogbackAppender 类，放置在 CAT 目录下：

```java
package com.itcast.logbackcat.cat;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.classic.spi.ThrowableProxy;
import ch.qos.logback.core.AppenderBase;
import ch.qos.logback.core.LogbackException;
import com.dianping.cat.Cat;

import java.io.PrintWriter;
import java.io.StringWriter;

public class CatLogbackAppender extends AppenderBase<ILoggingEvent> {

	@Override
	protected void append(ILoggingEvent event) {
		try {
			boolean isTraceMode = Cat.getManager().isTraceMode();
			Level level = event.getLevel();
			if (level.isGreaterOrEqual(Level.ERROR)) {
				logError(event);
			} else if (isTraceMode) {
				logTrace(event);
			}
		} catch (Exception ex) {
			throw new LogbackException(event.getFormattedMessage(), ex);
		}
	}

	private void logError(ILoggingEvent event) {
		ThrowableProxy info = (ThrowableProxy) event.getThrowableProxy();
		if (info != null) {
			Throwable exception = info.getThrowable();

			Object message = event.getFormattedMessage();
			if (message != null) {
				Cat.logError(String.valueOf(message), exception);
			} else {
				Cat.logError(exception);
			}
		}
	}

	private void logTrace(ILoggingEvent event) {
		String type = "Logback";
		String name = event.getLevel().toString();
		Object message = event.getFormattedMessage();
		String data;
		if (message instanceof Throwable) {
			data = buildExceptionStack((Throwable) message);
		} else {
			data = event.getFormattedMessage().toString();
		}

		ThrowableProxy info = (ThrowableProxy) event.getThrowableProxy();
		if (info != null) {
			data = data + '\n' + buildExceptionStack(info.getThrowable());
		}

		Cat.logTrace(type, name, "0", data);
	}

	private String buildExceptionStack(Throwable exception) {
		if (exception != null) {
			StringWriter writer = new StringWriter(2048);
			exception.printStackTrace(new PrintWriter(writer));
			return writer.toString();
		} else {
			return "";
		}
	}
}
```

修改 logback-spring.xml 配置文件：

```xml
<configuration>
    
    ...
    <appender name="CatAppender" class="com.itcast.logbackcat.cat.CatLogbackAppender"></appender>
    <root level="info">
        <appender-ref ref="consoleLog"/>
        <appender-ref ref="fileInfoLog"/>
        <appender-ref ref="fileErrorLog"/>
        <!-- 添加了这个 -->
        <appender-ref ref="CatAppender" />
    </root>
</configuration>
```

修改Controller接口：

```java
@RequestMapping("logback")
public String test(){
    Cat.getManager().setTraceMode(true);
    log.info("cat info");
    try {
        int i = 1/0;
    }catch (Exception e){
        log.error("cat error",e);
    }
    return  "logback";
}
```

#### 3.1.3.3 测试

启动项目，访问 http://localhost:7080/logback

访问 CAT 控制台，可以看到 Problem 报表中已经出现了一个 error，点击SampleLinks查看详细信息

![logback-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408022.png)

Cat列出的信息还是相对详细的，有 INFO 级别的日志与 ERROR 级别的日志，其中 ERROR 级别的日志显示出了所有的堆栈信息方便分析问题。

![logback-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408316.png) 

### 3.1.4 SpringBoot

#### 3.1.4.1 搭建环境

SpringBoot 的集成方式相对比较简单，我们使用已经搭建完的 Mybatis 框架来进行测试：

添加如下配置类到 config 包中

```java
package com.itcast.mybatiscat.config;

import com.dianping.cat.servlet.CatFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CatFilterConfigure {
    @Bean
    public FilterRegistrationBean catFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        CatFilter filter = new CatFilter();
        registration.setFilter(filter);
        registration.addUrlPatterns("/*");
        registration.setName("cat-filter");
        registration.setOrder(1);
        return registration;
    }
}
```

#### 3.1.4.2 测试

访问地址 http://localhost:7079/mybatis

![mybatis3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408356.png)

图中的调用先经过了 Controller，所以打印出了相关信息：

-  /mybatis 接口地址
-  URL.Server 服务器、浏览器等相关信息
-  URL.Method 调用方法(GET、POST等)和 URL

### 3.1.5 Spring AOP

使用 Spring AOP 技术可以简化我们的埋点操作，通过添加统一注解的方式，使得指定方法被能被 CAT 监控起来。

#### 3.1.5.1 搭建环境

创建基于 SpringBoot 的 springaop-cat 项目。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itcast</groupId>
    <artifactId>springaop-cat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springaop-cat</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 使用AOP必须添加的依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <!-- 添加cat客户端的依赖 -->
        <dependency>
            <groupId>com.dianping.cat</groupId>
            <artifactId>cat-client</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

创建 AOP 接口：

```java
package com.itcast.springaopcat.aop;

import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

@Retention(RUNTIME)
@Target(ElementType.METHOD)
public @interface CatAnnotation {
}
```

创建 AOP 处理类：

```java
package com.itcast.springaopcat.aop;

import java.lang.reflect.Method;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Transaction;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class CatAopService {

    @Around(value = "@annotation(CatAnnotation)")
    public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
        MethodSignature joinPointObject = (MethodSignature) pjp.getSignature();
        Method method = joinPointObject.getMethod();

        Transaction t = Cat.newTransaction("method", method.getName());

        try {
            Object res = pjp.proceed();
            t.setSuccessStatus();
            return res;
        } catch (Throwable e) {
            t.setStatus(e);
            Cat.logError(e);
            throw e;
        } finally {
            t.complete();
        }
    }
}
```

创建 controller：在 aop1 上添加注解 @CatAnnotation，这样 aop1 就能被CAT监控起来。

```java
package com.itcast.springaopcat.controller;

import com.dianping.cat.Cat;
import com.itcast.springaopcat.aop.CatAnnotation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AopController {

    @RequestMapping("aop")
    @CatAnnotation
    public String aop1(){
        return  "aop";
    }
}
```

#### 3.1.5.2 测试

访问接口http://localhost:7090/aop

查看cat的transaction报表可以看到：

![aop1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408823.png)

加上CatAnnotation注解的方法被调用后，生成了1次调用记录。点击Log View显示详细信息：

![aop2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408028.png)

上图中显示了当前调用的方法名aop1以及时间4.52ms，证明spring-aop注解方式与CAT集成成功。

### 3.1.6 Spring MVC

Spring MVC 的集成方式，官方提供的是使用 AOP 来进行集成，源码如下：

AOP 接口

```JAVA
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface CatTransaction {
    String type() default "Handler";  //"URL MVC Service SQL" is reserved for Cat Transaction Type
    String name() default "";
}
```

AOP处理代码：

```java
@Around("@annotation(catTransaction)")
public Object catTransactionProcess(ProceedingJoinPoint pjp, CatTransaction catTransaction) throws Throwable {
    String transName = pjp.getSignature().getDeclaringType().getSimpleName() + "." + pjp.getSignature().getName();
    if(StringUtils.isNotBlank(catTransaction.name())){
        transName = catTransaction.name();
    }
    Transaction t = Cat.newTransaction(catTransaction.type(), transName);
    try {
        Object result = pjp.proceed();
        t.setStatus(Transaction.SUCCESS);
        return result;
    } catch (Throwable e) {
        t.setStatus(e);
        throw e;
    }finally{
        t.complete();
    }
}
```

因这部分与 Spring AOP 处理方式基本你一样，如需集成请详见Spring AOP。

## 3.2 告警配置

CAT 提供给我们完善的告警功能。合理、灵活的监控规则可以帮助更快、更精确的发现业务线上故障。

### 3.2.1 告警通用配置

#### 告警服务器配置

**只有配置为告警服务器的机器，才会执行告警逻辑；只有配置为发送服务器的机器，才会发送告警。**

进入功能 全局系统配置-服务端配置，修改服务器类型，对告警服务器增加`<property name="alarm-machine" value="true"/>`配置、以及`<property name="send-machine" value="true"/>`配置。如下图所示：

![alert1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408433.png)

#### 告警策略

告警策略：配置某种告警类型、某个项目、某个错误级别，对应的告警发送渠道，以及暂停时间。

![alert2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408221.png)

举例：下述配置示例，说明对于Transaction告警，当告警项目名为 demo_project：

- 当告警级别为 error 时，发送渠道为邮件、短信、微信，连续告警之间的间隔为5分钟
- 当告警级别为 warning 时，发送渠道为邮件、微信，连续告警之间的间隔为10分钟

##### 配置示例

```xml
<alert-policy>
	<type id="Transaction">
          <group id="default">
             <level id="error" send="mail,weixin" suspendMinute="5"/>
             <level id="warning" send="mail,weixin" suspendMinute="5"/>
          </group>
          <group id="demo-project">
             <level id="error" send="mail,weixin,sms" suspendMinute="5"/>
             <level id="warning" send="mail,weixin" suspendMinute="10"/>
          </group>
    </type>
</alert-policy>
```

##### 配置说明：

- type 标签：告警的类型，可选：Transaction、Event、Business、Heartbeat
- group id 属性：group可以为default，代表默认，即所有项目；也可以为项目名，代表某个项目的策略，**此时 default 策略不会生效**
- level id 属性：错误级别，分为warning代表警告、error代表错误
- level send属性：告警渠道，分为mail-邮箱、weixin-微信、sms-短信
- level suspendMinute属性：连续告警的暂停时间

#### 告警接收人

告警接收人，为告警所属项目的联系人：

- 项目组邮件：项目负责人邮件，或项目组产品线邮件，多个邮箱由英文逗号分割，不要留有空格，作为发送告警邮件、微信的依据
- 项目组号码：项目负责人手机号；多个号码由英文逗号分隔，不要留有空格，作为发送告警短信的依据

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408274.png)

#### 告警服务端

告警发送中心的配置。

告警发送中心：提供发送短信、邮件、微信功能，且提供 Http API 的服务

CAT 在生成告警后，调用告警发送中心的 Http 接口发送告警。**CAT自身并不集成告警发送中心，请自己搭建告警发送中心**。

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408557.png)

##### 配置示例

```xml
<sender-config>
    <sender id="mail" url="http://test/" type="post" successCode="200" batchSend="true">
        <par id="type=1500"/>
        <par id="key=title,body"/>
        <par id="re=test@test.com"/>
        <par id="to=${receiver}"/>
        <par id="value=${title},${content}"/>
    </sender>
    <sender id="weixin" url="http://test/" type="post" successCode="success" batchSend="true">
        <par id="domain=${domain}"/>
        <par id="email=${receiver}"/>
        <par id="title=${title}"/>
        <par id="content=${content}"/>
        <par id="type=${type}"/>
    </sender>
    <sender id="sms" url="http://test/" type="post" successCode="200" batchSend="false">
        <par id="jsonm={type:808,mobile:'${receiver}',pair:{body='${content}'}}"/>
    </sender>
</sender-config>
```

##### 配置说明：

- sender id属性：告警的类型，可选mail、sms、weixin
- sender url属性：告警中心的URL
- sender batchSend属性：是否支持批量发送告警信息
- par：告警中心所需的 Http 参数。`${argument}` 代表构建告警对象时，附带的动态参数；此处需要根据告警发送中心的需求，将动态参数加入到代码 AlertEntity 中的 m_paras

### 3.2.2 告警规则

**目前 CAT 的监控规则有五个要素**

- **告警时间段**：同一项业务指标在每天不同的时段可能有不同的趋势。设定该项，可让CAT在每天不同的时间段执行不同的监控规则。注意：告警时间段，不是监控数据的时间段，**只是告警从这一刻开始进行检查数据**；

- **规则组合**：在一个时间段中，可能指标触发了多个监控规则中的一个规则就要发出警报，也有可能指标要同时触发了多个监控规则才需要发出警报。

- **监控规则类型**：通过以下六种类型对指标进行监控：最大值、最小值、波动上升百分比、波动下降百分比、总和最大值、总和最小值

- **监控最近分钟数**：设定时间后（单位为分钟），当指标在设定的最近的时间长度内连续触发了监控规则，才会发出警报。比如最近分钟数为3，表明连续三分钟的数据都满足条件才告警。如果分钟数为1，表示最近的一分钟满足条件就告警

- **规则与被监控指标的匹配**：监控规则可以按照名称、正则表达式与监控的对象（指标）进行匹配


##### 子条件类型：

有六种类型。子条件的内容为对应的阈值，请注意阈值只能由数字组成，当阈值表达百分比时，不能在最后加上百分号。

六种类型如下：

| 类型                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| MaxVal 最大值（当前值）             | 当前实际值最大值，比如检查最近3分钟数据，3分钟数据会有3个value，是表示3个值都必须同时>=设定值 |
| MinVal 最小值（当前值）             | 当前实际值最小值，比如检查最近3分钟数据，3分钟数据会有3个value，是表示3个值都必须同时比<=设定值 |
| FluAscPer 波动上升百分比（当前值）  | 波动百分比最大值。即当前最后（N）分钟值比监控周期内其它分钟值（M-N个）的增加百分比都>=设定的百分比时触发警报；<br />比如检查最近10分钟数据，触发个数为3；10分钟内数据会算出7个百分比数据，是表示最后3分钟值分别相比前面7分钟值，3组7次比较的上升波动百分比全部>=配置阈值。比如下降50%，阈值填写50。 |
| FluDescPer 波动下降百分比（当前值） | 波动百分比最小值。当前最后（N）分钟值比监控周期内其它（M-N个）分钟值的减少百分比都大于设定的百分比时触发警报；<br />比如检查最近10分钟数据，触发个数为3；10分钟数据会算出7个百分比数据，是表示最后3分钟值分别相比前面7分钟值，3组7次比较的下降波动百分比全部>=配置阈值。比如下降50%，阈值填写50。 |
| SumMaxVal 总和最大值（当前值）      | 当前值总和最大值，比如检查最近3分钟数据，表示3分钟内的总和>=设定值就告警。 |
| SumMinVal 总和最小值（当前值）      | 当前值总和最小值，比如检查最近3分钟数据，表示3分钟内的总和<=设定值就告警。 |

#### Transaction告警

对 Transaction 的告警，支持的指标有次数、延时、失败率；

监控周期：一分钟

##### 配置图示

如下图所示，配置了 springboot-cat 项目的 Transaction 监控规则。

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408129.png)

##### 配置说明

- 项目名：要监控的项目名

- Tyep：被监控transaction的type

- Name：被监控transaction的name；如果为All，代表全部name

- 监控指标：次数、延时、失败率

- 告警规则：详情见**告警规则**部分


#### Event告警

对Event的个数进行告警；

监控周期：一分钟

##### 配置图示

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408027.png)

##### 配置说明

- 项目名：要监控的项目名
- type：被监控event的type
- name：被监控event的name；如果为All，代表全部name
- 告警规则：详情见**告警规则**部分

#### 心跳告警

心跳告警是对服务器当前状态的监控，如监控系统负载、GC数量等信息；

监控周期：一分钟

##### 配置图示

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408725.png)

##### 配置说明

- 项目名：要监控的项目名
- 指标：被监控的心跳指标名称；心跳告警是由两级匹配的：首先匹配项目，然后按照指标匹配
- 告警规则：详情见**告警规则**部分

#### 异常告警

对异常的个数进行告警；

监控周期：一分钟

##### 配置图示

![alert3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408591.png)

##### 配置说明

- 项目名：要监控的项目名
- 异常名称：被监控异常名称；当设置为“Total”时，是针对当前项目组所有异常总数阈值进行设置；当设置为特定异常名称时，针对当前项目组所有同名的异常阈值进行设定
- warning阈值：到达该阈值，发送warning级别告警；当异常数小于该阈值时，不做任何警报
- error阈值：到达该阈值，发送error级别告警
- 总数大于Warning阈值，小于Error阈值，进行Warning级别告警；大于Error阈值，进行Error级别告警

### 3.2.3 告警接口编写

编写 controller 接口：

```java
package com.itcast.springbootcat;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Event;
import com.dianping.cat.message.Transaction;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import java.util.Map;

@RestController
public class AlertController {

    @RequestMapping(value = "/alert/msg")
    public String sendAlert(@RequestParam String to, @RequestParam String value) {
        System.out.println("告警了" + to);
        // 后续就对接自己的一些接口服务
        return "200";
    }
}
```

修改告警服务端的配置，填写接口地址，以邮件为例：

##### 配置示例

```xml
<sender id="mail" url="http://localhost:8085/alert/msg" type="post" successCode="200" batchSend="true">
    <par id="type=1500"/>
    <par id="key=title,body"/>
    <par id="re=test@test.com"/>
    <par id="to=${receiver}"/>
    <par id="value=${title},${content}"/>
</sender>
```

测试结果，输出内容如下：

```properties
# 这就是上面配置的告警人员信息
告警了testUser1@test.com,testUser2@test.com
```

