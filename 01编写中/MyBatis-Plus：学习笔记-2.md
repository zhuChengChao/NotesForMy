# MyBatis-Plus：学习笔记-2

> 工作中看使用的MyBatis-Plus，虽然说日常开发起来也没遇到啥问题，但还是抽空来系统的学习一下。
>
> [Mybatis-plus课程地址](https://www.bilibili.com/video/BV1rE41197jR)

## 1. ActiveRecord  

ActiveRecord（简称AR）一直广受动态语言（ PHP、Ruby 等）的喜爱，而 Java 作为准静态语言，对于 ActiveRecord 往往只能感叹其优雅，所以我们也在 AR 道路上进行了一定的探索，喜欢大家能够喜欢。  

> **什么是ActiveRecord？**
>
> ActiveRecord也属于ORM（对象关系映射）层，由Rails最早提出，遵循标准的ORM模型：**表映射到记录，记录映射到对象，字段映射到对象属性**。配合遵循的命名和配置惯例，能够很大程度的快速实现模型的操作，而且简洁易懂。
>
> ActiveRecord的主要思想是：
>
> * 每一个数据库表对应创建一个类，类的每一个对象实例对应于数据库中表的一行记录；通常表的每个字段在类中都有相应的Field；
> * **ActiveRecord同时负责把自己持久化**，在ActiveRecord中封装了对数据库的访问，即CURD;
> * ActiveRecord是一种领域模型(Domain Model)，封装了部分业务逻辑；  

### 1.1 开启AR之旅

在MP中，开启AR非常简单，只需要将实体对象继承Model即可。

```java
package cn.xyc.mp.springboot.pojo;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("tb_user")
public class User extends Model<User>{
    @TableId(type = IdType.AUTO)
    private Long id;
    private String userName;
    private String password;
    private String name;
    private Integer age;
    private String email;
}
```

### 1.2 根据主键查询

> 可以看到的是，这里不需要再注入mapper了`@Autowired private UserMapper userMapper;`
>
> 不过需要注意的是，虽然没有注入mapper，但mapper还是不能删除的

```java
package cn.xyc.mp.springboot.test;


import cn.xyc.mp.springboot.pojo.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ActiveRecordTest {

    @Test
    public void testARSelect() {
        User user = new User();
        user.setId(2L);
        System.out.println(user.selectById());
    }
}

// 对应的sql：
// Preparing: SELECT id,user_name,password,name,age,email FROM tb_user WHERE id=?
// Parameters: 2(Long)
```

### 1.3 新增数据

```java
@Test
public void testARInsert() {
    User user = new User();
    user.setName("钱八");
    user.setAge(30);
    user.setPassword("123456");
    user.setUserName("qianba");
    user.setEmail("qianba@itcast.cn");
    boolean insert = user.insert();
    System.out.println(insert);
}

// 对应的sql：
// Preparing: INSERT INTO tb_user ( user_name, password, name, age, email ) VALUES ( ?, ?, ?, ?, ? ) 
// Parameters: qianba(String), 123456(String), 钱八(String), 30(Integer), qianba@itcast.cn(String)
```

### 1.4 更新操作

```java
@Test
public void testARUpdate() {
    User user = new User();
    user.setId(8L);
    user.setAge(35);
    boolean update = user.updateById();
    System.out.println(update);
}

// 对应的sql：
// Preparing: UPDATE tb_user SET age=? WHERE id=? 
// Parameters: 35(Integer), 8(Long)
```

### 1.5 删除操作

```java
@Test
public void testARDelete() {
    User user = new User();
    // user.setId(8L);
    // boolean delete = user.deleteById();
    boolean delete = user.deleteById(8L);
    System.out.println(delete);
}

// 对应的sql：
// Preparing: DELETE FROM tb_user WHERE id=? 
// Parameters: 8(Long)
```

### 1.6 根据条件查询

```java
@Test
public void testARQuery() {
    User user = new User();
    QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
    userQueryWrapper.le("age","20");
    List<User> users = user.selectList(userQueryWrapper);
    for (User user1 : users) {
        System.out.println(user1);
    }
}
// 对应的输出内容
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectList]-[DEBUG] ==>  Preparing: SELECT id,user_name,password,name,age,email FROM tb_user WHERE age <= ? 
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectList]-[DEBUG] ==> Parameters: 20(String)
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectList]-[DEBUG] <==      Total: 2
[main] [org.mybatis.spring.SqlSessionUtils]-[DEBUG] Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@53667cbe]
User(id=1, userName=zhangsan, password=123456, name=张三, age=18, email=test1@itcast.cn)
User(id=2, userName=lisi, password=123456, name=李四, age=20, email=test2@itcast.cn)
```

## 2. Oracle 主键Sequence

> 没用过Oracle，这部分内容略

## 3. 插件

### 3.1 mybatis的插件机制

MyBatis 允许你在已映射语句执行过程中的**某一点进行拦截调用**。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

1. Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
2. ParameterHandler (getParameterObject, setParameters)
3. ResultSetHandler (handleResultSets, handleOutputParameters)
4. StatementHandler (prepare, parameterize, batch, update, query)

我们看到了可以拦截Executor接口的部分方法，比如update，query，commit，rollback等方法，还有其他接口的一些方法等。

**总体概括为：**

1. 拦截执行器的方法→Executor 
2. 拦截参数的处理→ParameterHandler 
3. 拦截结果集的处理→ResultSetHandler 
4. 拦截Sql语法构建的处理→StatementHandler

**拦截器示例：**

```java
package cn.xyc.mp.springboot.plugins;

import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;

import java.util.Properties;

@Intercepts({
        @Signature(
                type = Executor.class,  // 指定拦截的类型
                method = "update",      // 指定Executor中的类型
                args = {MappedStatement.class,Object.class}
        )
})
public class MyInterceptor implements Interceptor{


    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 拦截方法，具体业务逻辑编写的位置
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        // 创建target对象的代理对象,目的是将当前拦截器加入到该对象中
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        // 属性设置，等价于在配置文件中<plugin></plugin>内部添加属性
    }
}
```

**注入方式1：注入到Spring容器**

```java
package cn.xyc.mp.springboot.config;

import cn.xyc.mp.springboot.plugins.MyInterceptor;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("cn.xyc.mp.springboot.mapper") // 设置mapper接口的扫描包
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor(){
        return new PaginationInterceptor();
    }

    /**
     * 自定义拦截器
     */
    @Bean
    public MyInterceptor myInterceptor(){
        return new MyInterceptor();
    }
}
```

**注入方式2：或者通过xml配置，mybatis-config.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

<plugins>
    <!--注入分页插件-->
    <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor"></plugin>
    <!--注入自定义插件-->
    <plugin interceptor="cn.xyc.mp.springboot.plugins.MyInterceptor"></plugin>
</plugins>

</configuration>
```

**测试：**在自定义的拦截器方法上打上断点

> ![自定义拦截器打上断点](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091518067.png)

执行select类型的方法：

```java
@Test
public void testARSelect() {
    User user = new User();
    user.setId(2L);
    System.out.println(user.selectById());
}
```

> 只会进入plugin方法中：
>
> * CachingExecutor
>
>   ![拦截器1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091524978.png)
>
> * MybatisDefaultParameterHandler
>
>   ![拦截器2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091524576.png)
>
> * DefaultResultSetHandler
>
>   ![拦截器3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091524282.png)
>
> * ...RoutingStatementHandler
>
>   ![拦截器4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091524772.png)

执行update类型方法：

```java
@Test
public void testARUpdate() {
    User user = new User();
    user.setId(8L);
    user.setAge(35);
    boolean update = user.updateById();
    System.out.println(update);
}
```

> 在执行update方法时，会进入intercept断点，其他和select类型方法类似
>
> * CachingExecutor
> * ![拦截器intercept方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091524292.png)
> * MybatisDefaultParameterHandler
> * DefaultResultSetHandler
> * ...RoutingStatementHandler

### 3.2 执行分析插件

在MP中提供了对SQL执行的分析的插件，可用作阻断全表更新、删除的操作。

> 注意：该插件仅适用于开发环境，不适用于生产环境。  

SpringBoot配置：

```java
/**
 * sql分析插件
 * @return
 */
@Bean
public SqlExplainInterceptor sqlExplainInterceptor(){
    SqlExplainInterceptor sqlExplainInterceptor = new SqlExplainInterceptor();
    List<ISqlParser> sqlParserList = new ArrayList<>();
    // 攻击 SQL 阻断解析器、加入解析链
    sqlParserList.add(new BlockAttackSqlParser());
    sqlExplainInterceptor.setSqlParserList(sqlParserList);
    return sqlExplainInterceptor;
}
```

测试：

```java
package cn.xyc.mp.springboot.test;


import cn.xyc.mp.springboot.mapper.UserMapper;
import cn.xyc.mp.springboot.pojo.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest02 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testUpdate(){
        User user = new User();
        user.setAge(20);
        // 进行全表的更新
        int result = this.userMapper.update(user, null);
        System.out.println("result = " + result);
    }
}
```

执行结果：

![sql执行拦截](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091535151.png)

> 可以看到，当执行全表更新时，会抛出异常，这样有效防止了一些误操作。  

### 3.3 性能分析插件

性能分析拦截器，用于输出每条 SQL 语句及其执行时间，可以设置最大执行时间，超过时间会抛出异常。

> 注意：该插件只用于开发环境，不建议生产环境使用

配置：(这次配置在配置文件中，mybaits-config.xml)

```xml
<plugins>
    <!-- SQL 执行性能分析，开发环境使用，线上不推荐。 maxTime 指的是 sql 最大执行时长 -->
    <plugin
            interceptor="com.baomidou.mybatisplus.extension.plugins.PerformanceInterceptor">
        <property name="maxTime" value="100" />
        <!--SQL是否格式化 默认false-->
        <property name="format" value="true" />
    </plugin>
</plugins>
```

随便找个select的测试方法，执行输出如下：

```mysql
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectById]-[DEBUG] ==>  Preparing: SELECT id,user_name,password,name,age,email FROM tb_user WHERE id=? 
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectById]-[DEBUG] ==> Parameters: 2(Long)
[main] [cn.xyc.mp.springboot.mapper.UserMapper.selectById]-[DEBUG] <==      Total: 1

Time：9 ms - ID：cn.xyc.mp.springboot.mapper.UserMapper.selectById
Execute SQL：
    SELECT
        id,
        user_name,
        password,
        name,
        age,
        email 
    FROM
        tb_user 
    WHERE
        id=2 
```

将上述最大执行时间设置为1ms：`<property name="maxTime" value="1" />`

![性能分析插件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207091542837.png)

### 3.4 乐观锁插件

#### 3.4.1 主要适用场景

**意图：**当要更新一条记录的时候，希望这条记录没有被别人更新

**乐观锁实现方式：**

* 取出记录时，获取当前version
* 更新时，带上这个version
* 执行更新时， `set version = newVersion where version = oldVersion`
* 如果version不对，就更新失败

#### 3.4.2 插件配置

spring xml：

```xml
<bean class="com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor"/>
```

spring boot：

```java
/**
 * 乐观锁插件
 * @return
 */
@Bean
public OptimisticLockerInterceptor optimisticLockerInterceptor() {
    return new OptimisticLockerInterceptor();
}
```

#### 3.4.3 注解实体字段

需要为实体字段添加`@Version`注解。

第一步：为表添加version字段，并且设置初始值为1

```mysql
ALTER TABLE `tb_user`
ADD COLUMN `version` int(10) NULL AFTER `email`;

UPDATE `tb_user` SET `version`='1';
```

第二步：为User实体对象添加version字段，并且添加@Version注解

```java
@Version
private Integer version;
```

#### 3.4.4 测试

测试用例：  

```java
@Test
public void testUpdateVersion(){
    User user = new User();
    user.setAge(30);
    user.setId(2L);
    user.setVersion(1); // 获取到version为1
    int result = this.userMapper.updateById(user);
    System.out.println("result = " + result);
}
```

执行输出：

```mysql
Execute SQL：
    UPDATE
        tb_user 
    SET
        age=30,
        version=2 
    WHERE
        id=2 
        AND version=1
```

可以看到，更新的条件中有version条件，并且更新的version为2。

如果再次执行，更新则不成功。这样就避免了多人同时更新时导致数据的不一致  

#### 3.4.5、特别说明

* **支持的数据类型只有：int, Integer, long, Long, Date, Timestamp, LocalDateTime**
* 整数类型下 `newVersion = oldVersion + 1`
* newVersion 会回写到 entity 中
* 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
* 在 `update(entity, wrapper)` 方法下,  **wrapper 不能复用**

## 4. SQL注入器

我们已经知道，在MP中，通过AbstractSqlInjector将BaseMapper中的方法注入到了Mybatis容器，这样这些方法才可以正常执行。

那么，如果我们需要**扩充BaseMapper中的**方法，又该如何实现呢？

**编写MyBaseMapper**

```java
package cn.xyc.mp.springboot.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;

import java.util.List;

public interface MyBaseMapper<T> extends BaseMapper<T> {
    
    List<T> findAll();
}
```

其他的Mapper都可以继承该Mapper，这样实现了统一的扩展，如：

```java
package cn.xyc.mp.springboot.mapper;

import cn.xyc.mp.springboot.pojo.User;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;

public interface UserMapper extends MyBaseMapper<User> {

    User findById(Long id);
    
    // 这里就包含了MyBaseMapper中定义的方法findAll
}
```

**编写FindAll**

```java
package cn.xyc.mp.springboot.injectors;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.injector.DefaultSqlInjector;

import java.util.List;

public class MySqlInjector extends DefaultSqlInjector /*extends AbstractSqlInjector*/ {

    @Override
    public List<AbstractMethod> getMethodList() {

        List<AbstractMethod> methodList = super.getMethodList();
        // 再扩充自定义的方法
        methodList.add(new FIndAll());
        return methodList;
    }
}
```

**编写MySqlInjector**

如果直接继承AbstractSqlInjector的话，原有的BaseMapper中的方法将失效，所以我们选择继承DefaultSqlInjector进行扩展。 

> 直接继承AbstractSqlInjector：
>
> ```java
> package cn.xyc.mp.springboot.injectors;
> 
> import com.baomidou.mybatisplus.core.injector.AbstractMethod;
> import com.baomidou.mybatisplus.core.injector.AbstractSqlInjector;
> 
> import java.util.ArrayList;
> import java.util.List;
> 
> public class MySqlInjector extends AbstractSqlInjector {
> 
>     @Override
>     public List<AbstractMethod> getMethodList() {
>         
>         List<AbstractMethod> list = new ArrayList<>();
>         list.add(new FIndAll());
>         return list;
>     }
> }
> ```

```java
package cn.xyc.mp.springboot.injectors;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.injector.DefaultSqlInjector;

import java.util.List;

public class MySqlInjector extends DefaultSqlInjector /*extends AbstractSqlInjector*/ {

    @Override
    public List<AbstractMethod> getMethodList() {

        List<AbstractMethod> methodList = super.getMethodList();
        // 再扩充自定义的方法
        methodList.add(new FIndAll());
        return methodList;
    }
}
```

**注册到Spring容器**

```java
/**
* 自定义SQL注入器
*/
@Bean
public MySqlInjector mySqlInjector(){
    return new MySqlInjector();
}
```

**测试**

```java
@Test
public void testFindAll(){
    List<User> users = this.userMapper.findAll();
    for (User user : users) {
        System.out.println(user);
    }
}
```

输出的SQL：

```java
[main] [cn.xyc.mp.springboot.mapper.UserMapper.findAll]-[DEBUG] ==>  Preparing: select * from tb_user 
[main] [cn.xyc.mp.springboot.mapper.UserMapper.findAll]-[DEBUG] ==> Parameters: 
[main] [cn.xyc.mp.springboot.mapper.UserMapper.findAll]-[DEBUG] <==      Total: 5
```

> 这里如果是直接继承AbstractSqlInjector，在默认的BaseMapper中的方法都会丢失：、
>
> ```java
> @Test
> public void testSelect() {
>     List<User> userList = userMapper.selectList(null);
>     for (User user : userList) {
>         System.out.println(user);
>     }
> }
> 
> // 报错：org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): cn.xyc.mp.springboot.mapper.UserMapper.selectList
> ```

## 5. 自动填充功能

有些时候我们可能会有这样的需求，插入或者更新数据时，希望有些字段可以自动填充数据，比如密码、version等。

在MP中提供了这样的功能，可以实现自动填充。

**添加@TableField注解**

```java
@TableField(select = false, fill = FieldFill.INSERT)
private String password;
```

为password添加自动填充功能，在新增数据时有效。

FieldFill提供了多种模式选择：

```java
public enum FieldFill {
    /**
     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入时填充字段
     */
    INSERT,
    /**
     * 更新时填充字段
     */
    UPDATE,
    /**
     * 插入和更新时填充字段
     */
    INSERT_UPDATE
}
```

编写 MyMetaObjectHandler：

```java
package cn.xyc.mp.springboot.handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler{

    /**
     * 在插入数据时填充
     * @param metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        
        // 先获取到password的值，为空处理，不为空不处理
        Object password = getFieldValByName("password", metaObject);
        if(null == password){
            // 字段为空，可以进行填充
            setFieldValByName("password", "123456", metaObject);
        }
    }

    /**
     * 在更新数据时填充
     * @param metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {

    }
}
```

测试：

```java
@Test
public void testAutoInsert(){
    User user = new User();
    user.setName("钱八");
    user.setUserName("qianba");
    user.setAge(30);
    user.setEmail("qianba@itast.cn");
    user.setVersion(1);
    int result = this.userMapper.insert(user);
    System.out.println("result = " + result);
}
```

执行输出：可以看到没有设置密码，却还是被自动设置了值

```mysql
Execute SQL：
    INSERT 
    INTO
        tb_user
        ( user_name, password, name, age, email, version ) 
    VALUES
        ( 'qianba', '123456', '钱八', 30, 'qianba@itast.cn', 1 )
```

## 6. 逻辑删除

开发系统时，有时候在实现功能时，删除操作需要实现逻辑删除，所谓逻辑删除就是将数据标记为删除，而并非真正的物理删除（非DELETE操作），查询时需要携带状态条件，确保被标记的数据不被查询到。这样做的目的就是避免数据被真正的删除。  

MP就提供了这样的功能，方便我们使用。

**修改表结构**

为tb_user表增加deleted字段，用于表示数据是否被删除，1代表删除，0代表未删除：

```mysql
ALTER TABLE 
	`tb_user` 
ADD COLUMN `deleted` int(1) NULL DEFAULT 0 COMMENT '1代表删除，0代表未删除' 
AFTER `version`;
```

同时，也修改User实体，增加deleted属性并且添加`@TableLogic`注解：

```java
@TableLogic
private Integer deleted;
```

**配置**：application.properties

```properties
# 逻辑已删除值(默认为 1)
mybatis-plus.global-config.db-config.logic-delete-value=1
# 逻辑未删除值(默认为 0)
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

**测试：**

```java
@Test
public void testDeleteById(){
    this.userMapper.deleteById(9L);
}
```

输出如下：

```mysql
Execute SQL：
    UPDATE
        tb_user 
    SET
        deleted=1 
    WHERE
        id=9 
        AND deleted=0
```

> 执行了 delete，从sql来看执行的是update，标记了逻辑删除

数据查找：

```java
@Test
public void testSelect(){
    User result = this.userMapper.selectById(8L);
    System.out.println("result = " + result);
}
```

执行输出：

```mysql
Time：9 ms - ID：cn.xyc.mp.springboot.mapper.UserMapper.selectById
Execute SQL：
    SELECT
        id,
        user_name,
        name,
        age,
        email,
        version,
        deleted 
    FROM
        tb_user 
    WHERE
        id=8 
        AND deleted=0
        
-- result = null
```

可见，已经实现了逻辑删除。

## 7. 通用枚举

解决了繁琐的配置，让 mybatis 优雅的使用枚举属性！