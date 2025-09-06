# Java：ThreadLocal学习笔记

> 看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444)** 的课程 [java基础教程由浅入深全面解析threadlocal](https://www.bilibili.com/video/BV1N741127FH) 后做的学习笔记
>

### 内容

1. ThreadLocal 介绍
2. 运用场景-事务案例
3. ThreadLocal 的内部结构
4. ThreadLocal 的核心方法源码
5. ThreadLocalMap 源码分析

## 1. ThreadLocal介绍

### 1.1 官方介绍

```java
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 *
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 * <pre>
 * import java.util.concurrent.atomic.AtomicInteger;
 *
 * public class ThreadId {
 *     // Atomic integer containing the next thread ID to be assigned
 *     private static final AtomicInteger nextId = new AtomicInteger(0);
 *
 *     // Thread local variable containing each thread's ID
 *     private static final ThreadLocal&lt;Integer&gt; threadId =
 *         new ThreadLocal&lt;Integer&gt;() {
 *             &#64;Override protected Integer initialValue() {
 *                 return nextId.getAndIncrement();
 *         }
 *     };
 *
 *     // Returns the current thread's unique ID, assigning it if necessary
 *     public static int get() {
 *         return threadId.get();
 *     }
 * }
 * </pre>
 * <p>Each thread holds an implicit reference to its copy of a thread-local
 * variable as long as the thread is alive and the {@code ThreadLocal}
 * instance is accessible; after a thread goes away, all of its copies of
 * thread-local instances are subject to garbage collection (unless other
 * references to these copies exist).
 *
 * @author  Josh Bloch and Doug Lea
 * @since   1.2
 */
public class ThreadLocal<T> {
    ...
}
```

从Java官方文档中的描述：ThreadLocal 类用来提供线程内部的局部变量。这种变量在多线程环境下访问（通过 get 和 set 方法访问）时能保证各个线程的变量相对独立于其他线程内的变量。ThreadLocal 实例通常来说都是 private static 类型的，用于关联线程和线程上下文。

我们可以得知 ThreadLocal 的作用是：提供线程内的局部变量，不同的线程之间不会相互干扰，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或组件之间一些公共变量传递的复杂度。

**总结:**

1. 线程并发: 在多线程并发的场景下
2. 传递数据: 我们可以通过 ThreadLocal 在同一线程，不同组件中传递公共变量
3. 线程隔离: 每个线程的变量都是独立的，不会相互影响

### 1.2 基本使用

#### 1.2.1 常用方法

 在使用之前,我们先来认识几个 ThreadLocal 的常用方法

| 方法声明                    | 描述                       |
| --------------------------- | -------------------------- |
| `ThreadLocal()`             | 创建ThreadLocal对象        |
| `public void set( T value)` | 设置当前线程绑定的局部变量 |
| `public T get()`            | 获取当前线程绑定的局部变量 |
| `public void remove()`      | 移除当前线程绑定的局部变量 |

#### 1.2.2 使用案例

我们来看下面这个案例

```java
/**
 * 需求：线程隔离
 *      在多线程并发的场景下，每个线程中的变量都是相互独立
 *      线程A： 设置（变量1）   获取（变量1）
 *      线程B： 设置（变量2）   获取（变量2）
 */
public class Demo01 {

    private String content;

    private String getContent() {
        return content;
    }

    private void setContent(String content) {
        this.content = content;
    }

    public static void main(String[] args) {
        Demo01 demo = new Demo01();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                // 每个线程：存一个变量，过一会儿取出这个变量
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------");
                System.out.println(Thread.currentThread().getName() + "--->" + demo.getContent());
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```

打印结果:

```java
-----------------------
-----------------------
线程1--->线程1的数据
线程0--->线程1的数据
-----------------------
线程2--->线程3的数据
-----------------------
线程3--->线程3的数据
-----------------------
线程4--->线程4的数据
```

从结果可以看出多个线程在访问同一个变量的时候出现的异常，线程间的数据没有隔离。下面我们来看下采用 ThreadLocal 的方式来解决这个问题的例子。

```java
public class Demo01 {

    // 创建一个 ThreadLocal 对象
    ThreadLocal<String> tl = new ThreadLocal<>();

    // private String content;
    private String getContent() {
        // return content;
        return tl.get();
    }

    private void setContent(String content) {
        // this.content = content;
        // 将变量绑定到ThreadLocal中
        tl.set(content);
    }

    public static void main(String[] args) {
        Demo01 demo = new Demo01();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                // 每个线程：存一个变量，过一会儿取出这个变量
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------");
                System.out.println(Thread.currentThread().getName() + "--->" + demo.getContent());
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```

打印结果:

```java
-----------------------
线程0--->线程0的数据
-----------------------
线程1--->线程1的数据
-----------------------
线程2--->线程2的数据
-----------------------
线程3--->线程3的数据
-----------------------
线程4--->线程4的数据
```

从结果来看，这样很好的解决了多线程之间数据隔离的问题，十分方便。

### 1.3 ThreadLocal 类与 synchronized 关键字

#### 1.3.1 synchronized 同步方式

首先上述例子中我们完全可以通过加锁来实现这个功能。我们首先来看一下用 synchronized 代码块实现的效果:

```java
public class Demo02 {

    private String content;

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public static void main(String[] args) {
        Demo02 demo02 = new Demo02();

        for (int i = 0; i < 5; i++) {
            Thread t = new Thread(() -> {
                synchronized (Demo02.class){
                    demo02.setContent(Thread.currentThread().getName() + "的数据");
                    System.out.println("-------------------------------------");
                    String content = demo02.getContent();
                    System.out.println(Thread.currentThread().getName() + "--->" + content);
                }
            });
            t.setName("线程" + i);
            t.start();
        }
    }
}
```

打印结果:

```java
-------------------------------------
线程0--->线程0的数据
-------------------------------------
线程3--->线程3的数据
-------------------------------------
线程2--->线程2的数据
-------------------------------------
线程1--->线程1的数据
-------------------------------------
线程4--->线程4的数据
```

从结果可以发现, 加锁确实可以解决这个问题，但是在这里我们**强调的是线程数据隔离的问题，并不是多线程共享数据的问题**, 在这个案例中使用 synchronized 关键字是不合适的。

#### 1.3.2 ThreadLocal 与 synchronized 的区别

虽然 ThreadLocal 模式与 synchronized 关键字都用于处理多线程并发访问变量的问题, 不过两者处理问题的角度和思路不同。

|        | synchronized                                                 | ThreadLocal                                                  |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理   | 同步机制采用’**以时间换空间**’的方式, **只提供了一份变量,让不同的线程排队访问** | ThreadLocal采用’**以空间换时间**’的方式, **为每一个线程都提供了一份变量的副本**,从而实现同时访问而相不干扰 |
| 侧重点 | 多个线程之间访问资源的**同步性**                             | 多线程中让每个线程之间的数据**相互隔离**                     |

总结： 在刚刚的案例中，虽然使用 ThreadLocal 和 synchronized 都能解决问题,但是**使用 ThreadLocal 更为合适,因为这样可以使程序拥有更高的并发性。**

## 2. 运用场景-事务案例

通过以上的介绍，我们已经基本了解 ThreadLocal 的特点。但是它具体的应用是在哪里呢？ 现在让我们一起来看一个 **ThreadLocal 的经典运用场景： 事务**。

### 2.1 转账案例

#### 2.1.1 场景构建

 这里我们先构建一个简单的转账场景： 有一个数据表 account，里面有两个用户 Jack 和 Rose，用户 Jack 给用户 Rose 转账。

 案例的实现就简单的用 mysql 数据库，JDBC 和 C3P0 框架实现。以下是详细代码 ：

1. 项目结构

![account项目结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603468.PNG)

2. 数据准备

```sql
-- 使用数据库
use test;
-- 创建一张账户表
create table account(
	id int primary key auto_increment,
	name varchar(20),
	money double
);
-- 初始化数据
insert into account values(null, 'Jack', 1000);
insert into account values(null, 'Rose', 1000);
```

3. C3P0配置文件和工具类

```xml
<c3p0-config>
    <!-- 使用默认的配置读取连接池对象 -->
    <default-config>
        <!-- 连接参数 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/db1</property>
        <property name="user">root</property>
        <property name="password">root</property>

        <!-- 连接池参数 -->
        <property name="initialPoolSize">5</property>
        <property name="maxPoolSize">10</property>
        <property name="checkoutTimeout">3000</property>
    </default-config>
</c3p0-config>
```

4. 工具类 ： JdbcUtils

```java
package cn.xyc.transfer.utils;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.sql.Connection;
import java.sql.SQLException;

public class JdbcUtils {
    // c3p0 数据库连接池对象属性
    private static final ComboPooledDataSource ds = new ComboPooledDataSource();
    // 获取连接
    public static Connection getConnection() throws SQLException{
        return ds.getConnection();
    }

    //释放资源
    public static void release(AutoCloseable... ios){
        for(AutoCloseable io: ios){
            if(io != null){
                try {
                    io.close();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }

    // 事务提交并释放连接
    public static void commitAndClose(Connection conn) {
        try {
            if(conn != null){
                //提交事务
                conn.commit();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // 事物回滚并释放连接
    public static void rollbackAndClose(Connection conn) {
        try {
            if(conn != null){
                //回滚事务
                conn.rollback();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

5. dao层代码 ： AccountDao

```java
package cn.xyc.transfer.dao;

import cn.xyc.transfer.utils.JdbcUtils;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class AccountDao {

    // 转钱
    public void out(String outUser, int money) throws SQLException{
        String sql = "update account set money = money - ? where name = ?";

        Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1, money);
        pstm.setString(2, outUser);
        pstm.executeUpdate();

        JdbcUtils.release();
    }

    // 收钱
    public void in(String inUser, int money) throws SQLException {
        String sql = "update account set money = money + ? where name = ?";

        Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1,money);
        pstm.setString(2,inUser);
        pstm.executeUpdate();

        JdbcUtils.release(pstm,conn);
    }
}
```

6. service层代码 ： AccountService

```java
package cn.xyc.transfer.service;

import cn.xyc.transfer.dao.AccountDao;

import java.sql.SQLException;

public class AccountService {

    public boolean transfer(String outUser, String inUser, int money){
        AccountDao dao = new AccountDao();
        // 转出钱
        try {
            dao.out(outUser, money);
            // 收钱
            dao.in(inUser, money);
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
}
```

7. web层代码 ： AccountWeb

   > 这里用 main 来模拟了

```java
package cn.xyc.transfer.web;

import cn.xyc.transfer.service.AccountService;

public class AccountWeb {

    public static void main(String[] args) {
        // 模拟数据 : Jack 给 Rose 转账 100
        String outUser = "Jack";
        String inUser = "Rose";
        int money = 100;

        AccountService as = new AccountService();
        boolean result = as.transfer(outUser, inUser, money);

        if (result == false) {
            System.out.println("转账失败!");
        } else {
            System.out.println("转账成功!");
        }
    }
}
```

#### 2.1.2 引入事务

案例中的转账涉及两个DML操作： 一个转出，一个转入。这些操作是需要具备原子性的，不可分割。不然就有可能出现数据修改异常情况。

```java
public class AccountService {
    public boolean transfer(String outUser, String inUser, int money) {
        AccountDao ad = new AccountDao();
        try {
            // 转出
            dao.out(outUser, money);
            // 算数运算异常：模拟转账过程中的异常
            int i = 1/0;
            // 转入
            dao.in(inUser, money);
        } catch (Exception e) {  // 异常提升
            e.printStackTrace();
            return false;
        }
        return true;
    }
}
```

 输出：出现了异常

```java
java.lang.ArithmeticException: / by zero
	at cn.xyc.transfer.service.AccountService.transfer(AccountService.java:24)
	at cn.xyc.transfer.web.AccountWeb.main(AccountWeb.java:14)
转账失败!
```

而数据库中的数据已经不能保存原子性了：

```
// 原先数据
id       name       money
1		 Jack		1000
2		 Rose		1000

// 发送异常后的数据
id       name       money
1		 Jack		 900
2		 Rose		1000
```

所以这里就需要操作事务，来保证转出和转入操作具备原子性，要么同时成功，要么同时失败。

**（1） JDBC中关于事务的操作的api**

| Connection接口的方法      | 作用                         |
| ------------------------- | ---------------------------- |
| void setAutoCommit(false) | 禁用事务自动提交（改为手动） |
| void commit();            | 提交事务                     |
| void rollback();          | 回滚事务                     |

使用上述API，开始事务后的代码如下：

```java
public boolean transfer(String outUser, String inUser, int money){
    AccountDao dao = new AccountDao();
    Connection connection = null;
    try {
        // 1.开启事务
        connection = JdbcUtils.getConnection();
        connection.setAutoCommit(false);
        // 转出
        dao.out(outUser, money);
        // 算数运算异常：模拟转账过程中的异常
        int i = 1/0;
        // 转入
        dao.in(inUser, money);

        // 2.成功提交
        // connection.commit();
        // connection.close();
        JdbcUtils.commitAndClose(connection);
    } catch (Exception e) {  // 异常提升
        e.printStackTrace();
        // 3.异常回滚
        // connection.rollback();
        // connection.close();
        JdbcUtils.rollbackAndClose(connection);
        return false;
    }
    return true;
}
```

上述代码是还是不能保证数据库的原子性的，因为 Connection 的对象不一致。

**（2） 开启事务的注意点:**

1. 为了保证所有的操作在一个事务中,案例中使用的连接（connection）必须是同一个: service层开启事务的connection需要跟dao层访问数据库的connection保持一致；

2. 而在线程并发情况下, 每个线程只能操作各自的 connection。

### 2.2 常规解决方案

#### 2.2.1 常规方案的实现

基于上面给出的前提， 大家通常想到的解决方案是 ：

1. **传参：**从service层将connection对象向dao层传递

2. **加锁**

以下是service代码实现修改的部分：

   ```java
public class AccountService {

    public boolean transfer(String outUser, String inUser, int money){
        AccountDao dao = new AccountDao();
        Connection connection = null;
        try {
            // 加锁保证线程并发情况下，每个线程只能操作各自的 connection
            synchronized (AccountService.class){
                // 1.开启事务
                connection = JdbcUtils.getConnection();
                connection.setAutoCommit(false);
                dao.out(outUser, money, connection);
                // 算数运算异常：模拟转账过程中的异常
                int i = 1/0;
                dao.in(inUser, money, connection);

                // 2.成功提交
                JdbcUtils.commitAndClose(connection);
            }

        } catch (Exception e) {  // 异常提升
            e.printStackTrace();
            // 3.异常回滚
            JdbcUtils.rollbackAndClose(connection);
            return false;
        }
        return true;
    }
}
   ```

AccountDao 类：

1. 方法添加参数Connection；
2. 不能从连接池中获取Connection连接了；
3. Connection连接不能在Dao层中释放了，而是在Service层中提交

```java
public class AccountDao {
    // 转钱
    public void out(String outUser, int money, Connection conn) throws SQLException{
        String sql = "update account set money = money - ? where name = ?";

        // Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1, money);
        pstm.setString(2, outUser);
        pstm.executeUpdate();
        // JdbcUtils.release();
    }

    // 收钱
    public void in(String inUser, int money, Connection conn) throws SQLException {
        String sql = "update account set money = money + ? where name = ?";

        // Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1,money);
        pstm.setString(2,inUser);
        pstm.executeUpdate();
        // JdbcUtils.release(pstm,conn);
    }
}
```

#### 2.2.2 常规方案的弊端

上述方式我们看到的确按要求解决了问题，但是仔细观察，会发现这样实现的弊端：

1. 直接从 service 层传递 connection 到 dao 层, 造成代码耦合度提高；
2. 加锁会造成线程失去并发性，程序性能降低。

### 2.3 ThreadLocal 解决方案

#### 2.3.1 ThreadLocal 方案的实现

像这种需要在项目中进行**数据传递**和**线程隔离**的场景，我们不妨用 ThreadLocal 来解决：

（1） 工具类的修改： 加入 ThreadLocal

```java
public class JdbcUtils {
    // c3p0 数据库连接池对象属性
    private static final ComboPooledDataSource ds = new ComboPooledDataSource();
    // ThreadLoacal对象
    private static final ThreadLocal<Connection> tl = new ThreadLocal<>();

    /**
     * 获取连接
     * 原本：直接从连接池中获取连接
     * 现在：
     *  1. 直接获取当前线程绑定的连接对象
     *  2. 若连接对象是空的，则再从连接池中获取连接，并将连接对象与当前线程绑定
     */
    public static Connection getConnection() throws SQLException{
        Connection connection = tl.get();
        if (connection==null){
            // 连接对象为空，连接池中获取连接
            connection = ds.getConnection();
            // 将连接对象与当前线程绑定
            tl.set(connection);
        }
        return connection;
    }
    
    // 事务提交并释放连接
    public static void commitAndClose(Connection conn) {
        try {
            if(conn != null){
                //提交事务
                conn.commit();
                // 解绑当前线程绑定的连接对象
                tl.remove();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // 事物回滚并释放连接
    public static void rollbackAndClose(Connection conn) {
        try {
            if(conn != null){
                //回滚事务
                conn.rollback();
                // 解绑当前线程绑定的连接对象
                tl.remove();
                //释放连接
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

（2） AccountService类的修改：不需要传递connection对象

```java
public class AccountService {

    public boolean transfer(String outUser, String inUser, int money){
        AccountDao dao = new AccountDao();
        Connection connection = null;
        try {
            // 1.开启事务
            connection = JdbcUtils.getConnection();
            connection.setAutoCommit(false);
            dao.out(outUser, money);
            // 算数运算异常：模拟转账过程中的异常
            int i = 1/0;
            dao.in(inUser, money);

            // 2.成功提交
            JdbcUtils.commitAndClose(connection);
        } catch (Exception e) {  // 异常提升
            e.printStackTrace();
            // 3.异常回滚
            JdbcUtils.rollbackAndClose(connection);
            return false;
        }
        return true;
    }
}
```

（3） AccountDao类的修改

```java
public class AccountDao {

    // 转钱
    public void out(String outUser, int money) throws SQLException{
        String sql = "update account set money = money - ? where name = ?";

        Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1, money);
        pstm.setString(2, outUser);
        pstm.executeUpdate();
    }

    // 收钱
    public void in(String inUser, int money) throws SQLException {
        String sql = "update account set money = money + ? where name = ?";

        Connection conn = JdbcUtils.getConnection();
        PreparedStatement pstm = conn.prepareStatement(sql);
        pstm.setInt(1,money);
        pstm.setString(2,inUser);
        pstm.executeUpdate();
    }
}
```

#### 2.3.2 ThreadLocal方案的好处

从上述的案例中我们可以看到， 在一些特定场景下，ThreadLocal方案有两个突出的优势：

1. **传递数据** ： 保存每个线程绑定的数据，在需要的地方可以直接获取, 避免参数直接传递带来的代码耦合问题
2. **线程隔离** ： 各线程之间的数据相互隔离却又具备并发性，避免同步方式带来的性能损失

## 3. ThreadLocal 的内部结构

通过以上的学习，我们对 ThreadLocal 的作用有了一定的认识。现在我们一起来看一下 ThreadLocal 的内部结构，探究它能够实现线程数据隔离的原理。

### 3.1 常见的误解

通常，如果我们不去看源代码的话，我猜 ThreadLocal 是这样子设计的：每个 ThreadLocal 类都创建一个 Map，然后用线程的 ID：threadID 作为 Map 的 key，要存储的局部变量作为 Map 的 value，这样就能达到各个线程的局部变量隔离的效果。这是最简单的设计方法，JDK最早期的 ThreadLocal 就是这样设计的。

![早期ThreadLocal设计](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603556.PNG)

### 3.2 核心结构

但是，JDK 后面优化了设计方案，现时JDK8  ThreadLocal 的设计是：每个 Thread 维护一个 ThreadLocalMap 哈希表，这个哈希表的 key 是 ThreadLocal 实例本身， value 才是真正要存储的值 Object。

1. 每个 Thread 线程内部都有一个 Map (ThreadLocalMap)
2. Map 里面存储 ThreadLocal 对象（key）和线程的变量副本（value）
3. Thread 内部的 Map 是由 ThreadLocal 维护的，由 ThreadLocal 负责向 map 获取和设置线程的变量值。
4. 对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。

![现在ThreadLocal设计](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603244.PNG)

### 3.3 这样设计的好处

这个设计与我们一开始说的设计刚好相反，这样设计有如下两个优势：

1. 这样设计之后每个`Map`存储的`Entry`数量就会变少，因为之前的存储数量由`Thread`的数量决定，现在是由`ThreadLocal`的数量决定，实际开发中，`ThreadLocal`的数量往往少于`Thread`的数量；

2. 当`Thread`销毁之后，对应的`ThreadLocalMap`也会随之销毁，能减少内存的使用。

## 4. ThreadLocal的核心方法源码

基于 ThreadLocal 的内部结构，我们继续探究一下 ThreadLocal 的核心方法源码，更深入的了解其操作原理。

除了构造之外， ThreadLocal 对外暴露的方法有以下4个：

| 方法声明                     | 描述                         |
| ---------------------------- | ---------------------------- |
| `protected T initialValue()` | 返回当前线程局部变量的初始值 |
| `public void set(T value)`   | 设置当前线程绑定的局部变量   |
| `public T get()`             | 获取当前线程绑定的局部变量   |
| `public void remove()`       | 移除当前线程绑定的局部变量   |

其实 get，set 和 remove 逻辑是比较相似的，我们要研究清楚其中一个，其他也就明白了。

### 4.1 set方法

**（1 ) 源码和对应的中文注释**

```java
/**
  * 设置当前线程对应的ThreadLocal的值
  *
  * @param value 将要保存在当前线程对应的ThreadLocal的值
  */
public void set(T value) {
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取此线程对象中维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    // 如果此map存在
    if (map != null)
        // 存在则调用map.set设置此实体entry
        map.set(this, value);
    else
        // 1）当前线程Thread 不存在ThreadLocalMap对象
        // 2）则调用createMap进行ThreadLocalMap对象的初始化
        // 3）并将此实体entry作为第一个值存放至ThreadLocalMap中
        createMap(t, value);
}

/**
  * 获取当前线程Thread对应维护的ThreadLocalMap 
  * 
  * @param  t the current thread 当前线程
  * @return the map 对应维护的ThreadLocalMap 
  */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

/**
  * 创建当前线程Thread对应维护的ThreadLocalMap 
  *
  * @param t 当前线程
  * @param firstValue 存放到map中第一个entry的值
  */
void createMap(Thread t, T firstValue) {
    // 这里的this是调用此方法的threadLocal
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

**（2 ) 代码执行流程**

 A. 首先获取当前线程，并根据当前线程获取一个Map

 B. 如果获取的Map不为空，则将参数设置到Map中（当前ThreadLocal的引用作为key）

 C. 如果Map为空，则给该线程创建 Map，并设置初始值

### 4.2 get方法

**（1 ) 源码和对应的中文注释**

```java
/**
  * 返回当前线程中保存ThreadLocal的值
  * 如果当前线程没有此ThreadLocal变量，
  * 则它会通过调用{@link #initialValue} 方法进行初始化值
  *
  * @return 返回当前线程对应此ThreadLocal的值
  */
public T get() {
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取此线程对象中维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    // 如果此map存在
    if (map != null) {
        // 以当前的ThreadLocal为key，调用getEntry获取对应的存储实体e
        ThreadLocalMap.Entry e = map.getEntry(this);
        // 找到对应的存储实体 e 
        if (e != null) {
            @SuppressWarnings("unchecked")
            // 获取存储实体 e 对应的 value值
            // 即为我们想要的当前线程对应此ThreadLocal的值
            T result = (T)e.value;
            return result;
        }
    }
    // 初始化：有两种情况执行下述代码
    // 情况1：如果map不存在，则证明此线程没有维护的ThreadLocalMap对象
    // 情况2：map存在，但是没有与当前ThreadLocal关联的entry
    return setInitialValue();
}

/**
  * 初始化：
  *
  * @return the initial value 初始化后的值
  */
private T setInitialValue() {
    // 调用initialValue获取初始化的值
    // 此方法可以被子类重写，如果不重写默认返回null
    T value = initialValue();
    // 获取当前线程对象
    Thread t = Thread.currentThread();
    // 获取此线程对象中维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    // 如果此map存在
    if (map != null)
        // 存在则调用map.set设置此实体entry
        map.set(this, value);
    else
        // 1）当前线程Thread 不存在ThreadLocalMap对象
        // 2）则调用createMap进行ThreadLocalMap对象的初始化
        // 3）并将此实体entry作为第一个值存放至ThreadLocalMap中
        createMap(t, value);
    // 返回设置的值value
    return value;
}
```

**（2 ) 代码执行流程**

1. 首先获取当前线程

2. 根据当前线程获取一个Map

3. 如果获取的Map不为空，则在Map中以ThreadLocal的引用作为key来在Map中获取对应的value e，否则转到5
4. 如果e不为null，则返回e.value，否则转到5
5. Map为空或者e为空，则通过initialValue函数获取初始值value，然后用ThreadLocal的引用和value作为firstKey和firstValue创建一个新的Map

总结: **先获取当前线程的 ThreadLocalMap 变量，如果存在则返回值，不存在则创建并返回初始值。**

### 4.3 remove方法

**（1 ) 源码和对应的中文注释**

```java
 /**
   * 删除当前线程中保存的ThreadLocal对应的实体entry
   */
public void remove() {
    // 获取当前线程对象中维护的ThreadLocalMap对象
    ThreadLocalMap m = getMap(Thread.currentThread());
    // 如果此map存在
    if (m != null)
       // 存在则调用map.remove
       // 以当前ThreadLocal为key删除对应的实体entry
       m.remove(this);
}
```

**（2 ) 代码执行流程**

1. 首先获取当前线程，并根据当前线程获取一个Map

2. 如果获取的Map不为空，则移除当前ThreadLocal对象对应的entry

### **4.4 initialValue方法**

```java
/**
  * 返回当前线程对应的ThreadLocal的初始值
  
  * 此方法的第一次调用发生在，当线程通过{@link #get}方法访问此线程的ThreadLocal值时
  * 除非线程先调用了 {@link #set}方法，在这种情况下，
  * {@code initialValue} 才不会被这个线程调用。
  * 通常情况下，每个线程最多调用一次这个方法。
  *
  * <p>这个方法仅仅简单的返回null {@code null};
  * 如果程序员想ThreadLocal线程局部变量有一个除null以外的初始值，
  * 必须通过子类继承{@code ThreadLocal} 的方式去重写此方法
  * 通常, 可以通过匿名内部类的方式实现
  *
  * @return 当前ThreadLocal的初始值
  */
protected T initialValue() {
    return null;
}
```

 此方法的作用是返回该线程局部变量的初始值。

（1） 这个方法是一个延迟调用方法，从上面的代码我们得知，在set方法还未调用而先调用了get方法时才执行，并且仅执行1次。

（2）这个方法缺省实现直接返回一个`null`。

（3）如果想要一个除null之外的初始值，可以重写此方法。（备注： 该方法是一个`protected`的方法，显然是为了让子类覆盖而设计的）

## 5. ThreadLocalMap 源码分析

### 5.1 基本结构

ThreadLocalMap 是 ThreadLocal 的静态内部类，没有实现 Map 接口，用独立的方式实现了 Map 的功能，其内部的 Entry 也是独立实现。

![ThreadLocalMap基本结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603881.PNG)

**（1）成员变量**

```java
/**
 * 初始容量 —— 必须是2的整次幂
 */
private static final int INITIAL_CAPACITY = 16;

/**
 * 存放数据的table，Entry类的定义在下面分析
 * 同样，数组长度必须是2的冥。
 */
private Entry[] table;

/**
 * 数组里面 entrys 的个数，可以用于判断 table 当前使用量是否超过负因子。
 */
private int size = 0;

/**
 * 进行扩容的阈值，表使用量大于它的时候进行扩容。
 */
private int threshold; // Default to 0

/**
 * 阈值设置为长度的2/3
 */
private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```

* 跟 HashMap 类似，INITIAL_CAPACITY 代表这个 Map 的初始容量；

* table 是一个 Entry 类型的数组，用于存储数据；

* size代表表中的存储数目；

* threshold代表需要扩容时对应size的阈值。

**（2）存储结构 - Entry**

```java
/**
  * Entry继承 WeakReference，并且用ThreadLocal作为key，
  * 如果key为null(entry.get()==null)，意味着key不再被引用，
  * 因此这时候entry也可以从table中删除
  */
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```

在 ThreadLocalMap 中，也是用 Entry 来保存 K-V 结构数据的。但是 Entry 中 key 只能是 ThreadLocal 对象，这点被 Entry 的构造方法已经限定死了；

另外，Entry 继承 WeakReference，也就是 key（ThreadLocal）是弱引用，其目的是将 ThreadLocal 对象的生命周期和线程的生命周期解绑，可以使得ThreadLocal 在没有其他强引用的时候被回收掉，这样可以避免因为线程得不到销毁导致 ThreadLocal 对象无法被回收的情况。

### 5.2 弱引用和内存泄漏

有些程序员在使用 ThreadLocal 的过程中会发现有内存泄漏的情况发生，就猜测这个内存泄漏跟 Entry 中使用了弱引用的 key 有关系。这个理解其实是不对的。

我们先来回顾这个问题中涉及的几个名词概念，再来分析问题。

**（1）内存泄漏相关概念**

**Memory overflow**：内存溢出，没有足够的内存提供申请者使用。

**Memory leak**：内存泄漏是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果，内存泄漏的堆积终将导致内存溢出。

**（2）弱引用相关概念**

Java中的引用有4种类型: 强、软、弱、虚。当前这个问题主要涉及到强引用和弱引用:

**强引用( “Strong” Reference)** , 就是我们最常见的普通对象引用,只要还有强引用指向一个对象,就能表明对象还“活着”,垃圾回收器就不会回收这种对象。

**弱引用( WeakReference)** ,垃圾回收器一旦发现了只具有弱引用的对象,不管当前内存空间足够与否,都会回收它的内存。

**（3）如果key使用强引用**

假设 ThreadLocalMap 中的 key 使用了强引用,那么会出现内存泄漏吗?

此时 ThreadLocal 的内存图(实线表示强引用)如下:

![若key使用强引用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603171.PNG)

1. 假设在业务代码中使用完 ThreadLocal，ThreadLocal Ref 被回收了；
2. 但是因为 ThreadLocalMap 的 Entry 强引用 ThreadLocal，造成 ThreadLocal 无法被回收；
3. 在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，始终有强引用链`Current threadRef->currentThread->threadLocalMap->entry`，Entry 就不会被回收（Entry 中包括了 ThreadLocal 实例和 value），导致 Entry 内存泄露。

也就是说，ThreadLocalMap 中的 key 使用了强引用，是无法完全避免内存泄露的。

**（4）如果key使用弱引用**

那么 ThreadLocalMap 中的 key 使用了强引用,那么会出现内存泄漏吗?

![若key使用弱引用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603642.PNG)

1. 同样假设在业务代码中使用完 ThreadLocal，ThreadLocal Ref 被回收了；
2. 由于 ThreadLocalMap 只持有 ThreadLocal 的弱引用，没有任何强引用指向 ThreadLocal 实例，所以 ThreadLocal 就可以顺利被GC回收了，此时 Entry 中的key=null；
3. 但是在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，也存在有强引用链`Current threadRef->currentThread->threadLocalMap->entry->value`，value 不会被回收，而这块value永远不会被访问到了，导致value内存泄露。

也就是说，ThreadLocalMap 中的 key 使用了弱引用，也可能内存泄露。

**（5）出现内存泄露的原因**

比较以上两种情况，我们发现，内存泄露的发生跟 ThreadLocalMap 中的 key 是否使用弱引用是没有关系的，那么内存泄露真正的原因是什么呢？

又可以发现，以上两种内存泄露的情况中，都有两个前提：

1. 没有手动删除这个Entry；
2. CurrentThread 依然在运行。

第1点很好理解，只要在使用完 ThreadLocal，调用其 remove 方法删除对应的 Entry，就能避免内存泄露；

第2点稍微复杂一点，由于 ThreadLocalMap 是 Thread 的一个属性，被当前线程所引用，所以它的生命周期跟 Thread 一样长。那么在使用完 ThreadLocal 后，如果当前 Thread 也随之结束，ThreadLocalMap自然也会被GC回收，从根源上避免了内存泄露。

对于第2点，实现使用完 ThreadLocal，将当前的 Thread 结束不好控制，特别是使用线程池的时候，线程结束是不会被销毁的。

综上，ThreadLocal 内存泄露的根源是：**由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄露。**

也就是说,只要记得在使用完 ThreadLocal 及时的调用 remove ,无论 key 是强引用还是弱引用都不会有内存泄露问题。

**（6）那么为什么key要用弱引用呢?**

事实上，在 ThreadLocalMap 中的 set/getEntry 方法中,会对 key 为 null (也即是 ThreadLocal 为 null )进行判断,如果为 null 的话,那么是会对 value 置为 null 的。

这就意味着使用完 ThreadLocal ，CurrentThread 依然运行的前提下，就算忘记调用 remove 方法，**弱引用比强引用可以多一层保障**：弱引用的 ThreadLocal 会被回收，对应的 value 在下一次 ThreadLocalMap 调用 set，get，remove 中的任一方法的时候会被清除，从而避免内存泄漏。

### 5.3 hash冲突的解决

ThreadLocal 使用的是自定义的 ThreadLocalMap，接下来我们来探究一下 ThreadLocalMap 的 hash 冲突解决方式。

**（1） 先回顾ThreadLocal的set() 方法**

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocal.ThreadLocalMap map = getMap(t);
    if (map != null)
        // 调用ThreadLocalMap的set方法
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocal.ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    // 调用ThreadLocalMap的构造方法
    t.threadLocals = new ThreadLocal.ThreadLocalMap(this, firstValue);
}
```

这个方法之前分析过，其作用是设置当前线程绑定的局部变量：

1. 首先获取当前线程，并根据当前线程获取一个Map；
2. 如果获取的Map不为空，则将参数设置到Map中（当前ThreadLocal的引用作为key）
3. 如果Map为空，则给该线程创建Map，并设置初始值。

这段代码有两个地方分别涉及到ThreadLocalMap的两个方法，对其进行分析

**（2）构造函数`ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue)`**

```java
/**
  * firstKey:本ThreadLocal实例(this)
  * firstValue:要保存的线程本地变量
  */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    // 初始化table
    table = new ThreadLocal.ThreadLocalMap.Entry[INITIAL_CAPACITY];
    // 计算索引
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    // 设置值
    table[i] = new ThreadLocal.ThreadLocalMap.Entry(firstKey, firstValue);
    size = 1;
    // 设置阈值--INITIAL_CAPACITY的2/3
    setThreshold(INITIAL_CAPACITY);
}
```

构造函数首先创建一个长度为16的Entry数组，然后计算出 firstKey 的索引，然后存储到 table 中，并设置 size 和 Threshold

**重点分析：**`int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);`

* 关于`firstKey.threadLocalHashCodec`，firstKey 是 ThreadLocal 类的实例，threadLocalHashCodec 为其成员属性

  ```java
  private final int threadLocalHashCode = nextHashCode();
  
  private static int nextHashCode() {
      return nextHashCode.getAndAdd(HASH_INCREMENT);
  }
  // AtomicInteger是一个提供原子操作的Integer类，通过线程安全的方式操作加减，适合高并发情况下使用
  private static nextHashCode =  new AtomicInteger();
  
  private static final int HASH_INCREMENT = 0x61c88647;
  ```

  这里定义了一个AtomicInteger类型，每次获取当前值并加上HASH_INCREMENT，`HASH_INCREMENT = 0x61c88647`,这个值和斐波那契散列有关（黄金分割数），其主要目的就是为了让哈希码能均匀的分布在2的n次方的数组里, 也就是`Entry[] table`中，这样做可以尽量避免hash冲突。

- 关于`& (INITIAL_CAPACITY - 1)`

  计算hash的时候里面采用了`hashCode&(size-1)`的算法，这相当于取模运算`hashCode&size`的一个更高效的实现。正是因为这种算法，我们要求size必须是2的整次幂，这也能保证在索引不越界的前提下，使得hash发生冲突的次数减小。

**（3） ThreadLocalMap中的set()**

```java
private void set(ThreadLocal<?> key, Object value) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    // 计算索引(重点代码，上述已经分析过了)
    int i = key.threadLocalHashCode & (len-1);

    /**
      * 使用线性探测法查找元素（重点）
      */
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // ThreadLocal 对应的key存在，直接覆盖之前的值
        if (k == key) {
            e.value = value;
            return;
        }
        // 如果key为null，但是值不为空，说明之前的ThreadLocal对象已经被回收了
        // 当前数组中的Entry是一个陈旧(stale)的元素
        if (k == null) {
            // 用新元素替换陈旧的元素，这个方法进行了不少的垃圾清理动作，防止内存泄露
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // ThreadLocal对应的key不存在并且没有找到陈旧的元素，则在空元素的位置创建一个新的Entry
    tab[i] = new Entry(key, value);
    int sz = ++size;
    /**
   	 * cleanSomeSlots用于清除那些e.get()==null的元素
   	 * 这种数据key关联的对象已经被回收，所以这个Entry(table[index])可以被置为null.
   	 * 如果没有清除任何entry，并且当前使用量达到了负载因子所定义(长度的2/3)
   	 * 那么进行rehash(执行一次全表的扫描清理工作)
   	 */
    if(!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
    
/**
 * 获取环形数组的下一个索引
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * 获取环形数组的上一个索引
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

代码执行流程：

1. 首先还是根据key计算出索引 i ，然后查找 i 位置上的Entry；
2. 若是Entry已经存在并且key等于传入的key，那么这时候直接给这个Entry赋新的value值；
3. 若是Entry存在，但是key为null，则调用replaceStaleEntry来更换这个key为空的Entry；
4. 不断循环检测，直到遇到为null的地方，这时候要是还没在循环过程中return，那么就在这个null的位置新建一个Entry，并且插入，同时Size增加1；
5. 最后调用cleanSomeSlots，清理key为null的Entry，最后返回是否清理了Entry，接下来再判断sz是否大于等于threshold达到了rehash条件，达到的话就会调用rehash函数执行一次全表的扫描清理。

**重点分析：**ThreadLocalMap使用**线性探测法**来解决哈希冲突

该方法一次探测下一个地址，直到有空的地址后插入，若整个空间都找不到空余的地址，则产生溢出。

举个例子：假设当前table长度为16，也就是说如果计算出来key的hash值为14，如果table[14]上已经有值，并且其key与当前key不一致，那么就发生了hash冲突，这个时候将14加1得到15，取table[15]进行判断，这个时候如果还是冲突会回到0（nextIndex），取table[0]，以此类推，直到可以插入。

按照上面的描述，可以把table看成一个环形数组。