# MySQL：基础学习笔记-4

> 记录一下 MySQL 基础的一些语法，便于查询，该部分内容主要是参考：bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444)** 的课程而做的笔记。

## 1. JDBC 概述

JDBC 规范定义接口，具体的实现由各大数据库厂商来实现。  

JDBC 是 Java 访问数据库的标准规范，真正怎么操作数据库还需要具体的实现类，也就是数据库驱动。每个数据库厂商根据自家数据库的通信格式编写好自己数据库的驱动。所以我们只需要会调用 JDBC 接口中的方法即可， 数据库驱动由数据库厂商提供。  

使用 JDBC 的好处：

1. 程序员如果要开发访问数据库的程序， 只需要会调用 JDBC 接口中的方法即可， 不用关注类是如何实现的；
2. 使用同一套 Java 代码，进行少量的修改就可以访问其他 JDBC 支持的数据库  

![JDBC概述](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644599.PNG)

**使用 JDBC 开发使用到的包**：

| 会使用到的包 | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| `java.sql`   | 所有与 JDBC 访问数据库相关的接口和类                         |
| `javax.sql`  | 数据库扩展包，提供数据库额外的功能。如：连接池               |
| 数据库的驱动 | 由各大数据库厂商提供，需要额外去下载，是对 JDBC 接口实现的类 |

**JDBC 的核心 API**

| 接口或类              | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| DriverManager 类      | 1. 管理和注册数据库驱动 <br />2. 得到数据库连接对象          |
| Connection 接口       | 一个连接对象，可用于创建 Statement 和 PreparedStatement 对象 |
| Statement 接口        | 一个 SQL 语句对象，用于将 SQL 语句发送给数据库服务器。       |
| PreparedStatemen 接口 | 一个 SQL 语句对象，是 Statement 的子接口                     |
| ResultSet 接口        | 用于封装数据库查询的结果集，返回给客户端 Java 程序           |

**加载和注册驱动**：

`Class.forName(数据库驱动实现类)`：加载和注册数据库驱动，数据库驱动由 MySQL 厂商 `"com.mysql.jdbc.Driver"`

```java
public class JDBCDemo {
    public static void main(String[] args) throws ClassNotFoundException {

        // 抛出类找不到的异常，注册数据库驱动
        Class.forName("com.mysql.jdbc.Driver");
    }
}
```

`com.mysql.jdbc.Driver` 源代码：

```java
package com.mysql.jdbc;

import java.sql.DriverManager;
import java.sql.SQLException;

// Driver 接口，所有数据库厂商必须实现的接口，表示这是一个驱动类。
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

> 从 JDBC3 开始，目前已经普遍使用的版本。可以不用注册驱动而直接使用，即 Class.forName 这句话可以省略。  
>

## 2. DriverManager 类

**作用：**

* 管理和注册驱动；
* 创建数据库的连接；

**方法：**DriverManager 类中的静态方法

1. `Connection getConnection (String url, String user, String password)`：通过连接字符串， 用户名， 密码来得到数据库的连接对象
2. `Connection getConnection (String url, Properties info)`：通过连接字符串， 属性对象来得到连接对象

**使用 JDBC 连接数据库的四个参数**：

| JDBC 连接数据库的四个参数 | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 用户名                    | 登录的用户名                                                 |
| 密码                      | 登录的密码                                                   |
| 连接字符串 URL            | 不同的数据库 URL 是不同的， mysql 的写法： <br />`jdbc:mysql://localhost:3306/数据库[?参数名=参数值]` |
| 驱动类的字符串名          | `com.mysql.jdbc.Driver`                                      |

**连接数据库的 URL 地址格式**：`协议名:子协议://服务器名或IP地址:端口号/数据库名?参数=参数值`

> MySQL 写法 ：`jdbc:mysql://localhost:3306/数据库[?参数名=参数值]`
>
> 当是本地服务器，且端口号是3306时，可以简写为：`jdbc:mysql:///数据库名`
>
> 乱码的处理：可以指定参数: `?characterEncoding=utf8`，即为：`jdbc:mysql://localhost:3306/数据库?characterEncoding=utf8 `

**案例**：得到 MySQL 的数据库连接对象  

```java
// 使用用户名、密码、 URL 得到连接对象
import java.sql.Connection;
import java.sql.DriverManager;

public class JDBCDemo {

    public static void main(String[] args) throws Exception {

        String url = "jdbc:mysql://localhost:3306/db1";
        // 1) 使用用户名、密码、 URL 得到连接对象
        Connection connection = DriverManager.getConnection(url, "root", "root");
        // com.mysql.jdbc.JDBC4Connection@5a10411
        System.out.println(connection);
    }
}

// 使用属性文件和 url 得到连接对象
import java.sql.Connection;
import java.sql.DriverManager;
import java.util.Properties;

public class JDBCDemo {

    public static void main(String[] args) throws Exception {

        // url 连接字符串
        String url = "jdbc:mysql://localhost:3306/db1";
        // 属性对象
        Properties info = new Properties();
        // 把用户名和密码放在 info 对象中
        info.setProperty("user","root");
        info.setProperty("password","root");
        Connection connection = DriverManager.getConnection(url, info);
        // com.mysql.jdbc.JDBC4Connection@5a10411
        System.out.println(connection);
    }
}
```

## 3. Connection接口

**作用**：具体的实现类由数据库的厂商实现，代表一个连接对象

**方法**：`Statement createStatement()`：  创建一条 SQL 语句对象

## 4. Statement 接口

**JDBC 访问数据库的步骤**

![JDBC访问数据库的步骤](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644529.PNG)

1. 注册和加载驱动(可以省略)
2. 获取连接
3. Connection 获取 Statement 对象
4. 使用 Statement 对象执行 SQL 语句
5. 返回结果集
6. 释放资源  

**作用** ：代表一条语句对象，用于发送 SQL 语句给服务器，用于执行静态 SQL 语句并返回它所生成结果的对象。  

**方法**：

| Statement 接口中的方法                | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `int executeUpdate(String sql) `      | 用于发送 DML 语句，增删改的操作，insert、update、 delete<br />参数： SQL 语句 <br />返回值：返回对数据库影响的行数 |
| `ResultSet executeQuery(String sql) ` | 用于发送 DQL 语句，执行查询的操作 select <br />参数： SQL 语句 <br />返回值：查询的结果集 |

**释放资源**：

1. 需要释放的对象： ResultSet 结果集， Statement 语句， Connection 连接
2. 释放原则：先开的后关，后开的先关。 ResultSet--->Statement--->Connection
3. 放在哪个代码块中： finally 块

**案例**：

* 执行 DDL 操作：用 JDBC 在 MySQL 的数据库中创建一张学生表

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCDemo {

    public static void main(String[] args) throws Exception {

        // 1. 创建连接
        Connection conn = null;
        Statement statement = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql:///db1", "root", "root");
            // 2. 通过连接对象得到语句对象
            statement = conn.createStatement();
            // 3. 通过语句对象发送 SQL 语句给服务器
            // 4. 执行 SQL
            statement.executeUpdate(
                    "create table student (id int PRIMARY key auto_increment, " +
                    "name varchar(20) not null, gender boolean, birthday date)"
            );
            // 5. 返回影响行数(DDL 没有返回值)
            System.out.println("创建表成功");
        } catch (SQLException e) {
            e.printStackTrace();
        }
        // 6. 释放资源
        finally {
        // 关闭之前要先判断
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

// 查看数据库：
mysql> desc student;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(20) | NO   |     | NULL    |                |
| gender   | tinyint(1)  | YES  |     | NULL    |                |
| birthday | date        | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
4 rows in set (0.03 sec)
```

* 执行 DML 操作：用 JDBC 在 MySQL 的数据库中创建一张学生表

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class JDBCDemo {

    public static void main(String[] args) throws Exception {
        // 1) 创建连接对象
        Connection connection = DriverManager.getConnection(
                "jdbc:mysql:///db1", "root", "root");
        // 2) 创建 Statement 语句对象
        Statement statement = connection.createStatement();
        // 3) 执行 SQL 语句： executeUpdate(sql)
        int count = 0;
        // 4) 返回影响的行数
        count += statement.executeUpdate("insert into student values(null, '孙悟空', 1, '1993-03-24')");
        count += statement.executeUpdate("insert into student values(null, '白骨精', 0, '1995-03-24')");
        count += statement.executeUpdate("insert into student values(null, '猪八戒', 1, '1903-03-24')");
        count += statement.executeUpdate("insert into student values(null, '嫦娥', 0, '1993-03-11')");
        System.out.println("插入了" + count + "条记录");
        // 5) 释放资源
        statement.close();
        connection.close();
    }
}

// 查看数据库：
mysql> select * from student;
+----+-----------+--------+------------+
| id | name      | gender | birthday   |
+----+-----------+--------+------------+
|  1 | 孙悟空     |      1 | 1993-03-24 |
|  2 | 白骨精     |      0 | 1995-03-24 |
|  3 | 猪八戒     |      1 | 1903-03-24 |
|  4 | 嫦娥       |      0 | 1993-03-11 |
+----+-----------+--------+------------+
4 rows in set (0.00 sec)
```

## 5. ResultSet 接口 

**作用**：封装数据库查询的结果集，对结果集进行遍历，取出每一条记录

**方法**：

| ResultSet 接口中的方法 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `boolean next()`       | 1. 游标向下移动 1 行 <br />2. 返回 boolean 类型，如果还有下一条记录，返回 true，否则返回 false |
| 数据类型 `getXxx()`    | 1. 通过字段名，参数是 String 类型。返回不同的类型 <br />2. 通过列号，参数是整数，从 1 开始。返回不同的类型 |

> 例如有：
>
> * `boolean getBoolean(String columnLabel);`
> * `byte getByte(String columnLabel);`
> * `short getShort(String columnLabel);`
> * ...
> * `String getString(String columnLabel);`

**用数据类型转换表**：

| SQL 类型        | Jdbc 对应方法     | 返回类型                            |
| --------------- | ----------------- | ----------------------------------- |
| BIT(1) bit(n)   | getBoolean()      | boolean                             |
| TINYINT         | getByte()         | byte                                |
| SMALLINT        | getShort()        | short                               |
| INT             | getInt()          | int                                 |
| BIGINT          | getLong()         | long                                |
| CHAR,VARCHAR    | getString()       | String                              |
| Text(Clob) Blob | getClob getBlob() | Clob Blob                           |
| DATE            | getDate()         | java.sql.Date 只代表日期            |
| TIME            | getTime()         | java.sql.Time 只表示时间            |
| TIMESTAMP       | getTimestamp()    | java.sql.Timestamp 同时有日期和时间 |

> `java.sql.Date/Time/Timestamp(时间戳)`，三个共同父类是： `java.util.Date `

**案例**：接上述案例，查询刚刚插入的信息

```java
import java.sql.*;

public class JDBCDemo {

    public static void main(String[] args) throws Exception {
        // 1) 得到连接对象
        Connection connection =
                DriverManager.getConnection("jdbc:mysql://localhost:3306/db1","root","root");
        // 2) 得到语句对象
        Statement statement = connection.createStatement();
        // 3) 执行 SQL 语句得到结果集 ResultSet 对象
        ResultSet rs = statement.executeQuery("select * from student");
        // 4) 循环遍历取出每一条记录
        while(rs.next()) {
            int id = rs.getInt("id");
            String name = rs.getString("name");
            boolean gender = rs.getBoolean("gender");
            Date birthday = rs.getDate("birthday");
            // 5) 输出的控制台上
            System.out.println("编号： " + id + ", 姓名： " + name + ", 性别： " + gender + ", 生日： " + birthday);
        }
        // 6) 释放资源
        rs.close();
        statement.close();
        connection.close();
    }
}

// 输出：
// 编号： 1, 姓名： 孙悟空, 性别： true, 生日： 1993-03-24
// 编号： 2, 姓名： 白骨精, 性别： false, 生日： 1995-03-24
// 编号： 3, 姓名： 猪八戒, 性别： true, 生日： 1903-03-24
// 编号： 4, 姓名： 嫦娥, 性别： false, 生日： 1993-03-11
```

**关于 ResultSet 接口中的注意事项**：

1. 如果光标在第一行之前，使用 `rs.getXX()` 获取列值，报错：Before start of result set
2. 如果光标在最后一行之后，使用 `rs.getXX()` 获取列值，报错：After end of result set
3. 使用完毕以后要关闭结果集 ResultSet，再关闭 Statement，再关闭 Connection  

## 6. 自定义工具类

如果一个功能经常要用到，我们建议把这个功能做成一个工具类，可以在不同的地方重用。

```java
import java.sql.*;

/**
 * 访问数据库的工具类
 */
public class JdbcUtils {

    //可以把几个字符串定义成常量：用户名，密码， URL，驱动类
    private static final String USER = "root";
    private static final String PWD = "root";
    private static final String URL = "jdbc:mysql://localhost:3306/db1";
    private static final String DRIVER= "com.mysql.jdbc.Driver";

    /**
     * 注册驱动
     */
    static {
        try {
            Class.forName(DRIVER);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 得到数据库的连接
     */
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL,USER,PWD);
    }

    /**
     * 关闭所有打开的资源
     */
    public static void close(Connection conn, Statement stmt) {
        if (stmt!=null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn!=null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭所有打开的资源
     */
    public static void close(Connection conn, Statement stmt, ResultSet rs) {
        if (rs!=null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        close(conn, stmt);
    }
}
```

**案例**：

需求：

1. 创建一张用户表，添加几条用户记录，
2. 使用 Statement 字符串拼接的方式实现用户的登录, 用户在控制台上输入用户名和密码  

* 数据库端操作：

```mysql
CREATE TABLE USER (
    id INT PRIMARY KEY AUTO_INCREMENT,
    NAME VARCHAR(20),
    PASSWORD VARCHAR(20)
)

INSERT INTO USER VALUES (NULL,'jack','123'),(NULL,'rose','456');
-- 登录, SQL 中大小写不敏感
SELECT * FROM USER WHERE NAME='JACK' AND PASSWORD='123';
-- 登录失败
SELECT * FROM USER WHERE NAME='JACK' AND PASSWORD='333';
```

* Java 代码

```java
import java.sql.*;
import java.util.Scanner;

public class JDBCDemo {

    //从控制台上输入的用户名和密码
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入用户名： ");
        String name = sc.nextLine();
        System.out.println("请输入密码： ");
        String password = sc.nextLine();
        login(name, password);
    }

    /**
     * 登录的方法
     */
    public static void login(String name, String password) {
        // a) 通过工具类得到连接
        Connection connection = null;
        Statement statement = null;
        ResultSet rs = null;
        try {
            connection = JdbcUtils.getConnection();
            // b) 创建语句对象，使用拼接字符串的方式生成 SQL 语句
            statement = connection.createStatement();
            // c) 查询数据库，如果有记录则表示登录成功，否则登录失败
            String sql = "select * from user where name='" + name + "' and password='" + password + "'";
            System.out.println(sql);
            rs = statement.executeQuery(sql);
            if (rs.next()) {
                System.out.println("登录成功，欢迎您： " + name);
            } else {
                System.out.println("登录失败");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // d) 释放资源
            JdbcUtils.close(connection, statement, rs);
        }
    }   
}

// 控制台：
// 请输入用户名： 
// jack
// 请输入密码： 
// 123
// select * from user where name='jack' and password='123'
// 登录成功，欢迎您： jack
```

### SQL 注入问题

对于上述的案例：

* 当我们输入以下密码，我们发现我们账号和密码都不对竟然登录成功了

  ```java
  请输入用户名：
  newboy
  请输入密码：
  a' or '1'='1
  select * from user where name='newboy' and password='a' or '1'='1'
  登录成功，欢迎您： newboy
  ```

* 问题分析：  

  ```java
  select * from user where name='newboy' and password='a' or '1'='1'
  name='newboy' and password='a' 为假
  '1'='1' 真
  相当于
  select * from user where true; 查询了所有记录
  ```

  我们让用户输入的密码和 SQL 语句进行字符串拼接。用户输入的内容作为了 SQL 语句语法的一部分， 改变了原有 SQL 真正的意义， **以上问题称为 SQL 注入**。

  要解决 SQL 注入就不能让用户输入的密码和我们的 SQL 语句进行简单的字符串拼接。  

## 7. PreparedStatement 接口

**继承结构**：

```java
public interface PreparedStatement extends Statement {}

public interface Statement extends Wrapper, AutoCloseable {}
```

PreparedStatement 是 Statement 接口的子接口，继承于父接口中所有的方法。**它是一个预编译的 SQL 语句**  

**执行原理**：

![PreparedStatement执行原理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251644472.PNG)

1. 因为有预先编译的功能，提高 SQL 的执行效率。
2. 可以有效的防止 SQL 注入的问题，安全性更高。  

**方法**：

> 创建：Connection 创建 PreparedStatement 对象  
>
> `PreparedStatement prepareStatement(String sql)`：指定预编译的 SQL 语句， SQL 语句中使用占位符 ? 创建一个语句对象

| PreparedStatement 接口中的方法 | 描述                                     |
| ------------------------------ | ---------------------------------------- |
| `int executeUpdate()`          | 执行 DML，增删改的操作，返回影响的行数。 |
| `ResultSet executeQuery()`     | 执行 DQL，查询的操作，返回结果集         |

**PreparedSatement 的好处**：

1. `prepareStatement()`会先将 SQL 语句发送给数据库**预编译**。 PreparedStatement 会引用着预编译后的结果。这样可以多次传入不同的参数给 PreparedStatement 对象并执行。减少 SQL 编译次数，提高效率。
2. 安全性更高，没有 SQL 注入的隐患。
3. 提高了程序的可读性  

**使用：**

1. 编写 SQL 语句，未知内容使用 `?` 占位： `"SELECT * FROM user WHERE name=? AND password=?"`;
2. 获得 PreparedStatement 对象
3. 设置实际参数： `setXxx(占位符的位置, 真实的值)`
4. 执行参数化 SQL 语句
5. 关闭资源  

| PreparedStatement 中设置参数的方法             | 描述                                  |
| ---------------------------------------------- | ------------------------------------- |
| `void setDouble(int parameterIndex, double x)` | 将指定参数设置为给定 Java double 值。 |
| `void setFloat(int parameterIndex, float x)`   | 将指定参数设置为给定 Java REAL 值。   |
| `void setInt(int parameterIndex, int x)`       | 将指定参数设置为给定 Java int 值。    |
| `void setLong(int parameterIndex, long x)`     | 将指定参数设置为给定 Java long 值。   |
| `void setObject(int parameterIndex, Object x)` | 使用给定对象设置指定参数的值。        |
| `void setString(int parameterIndex, String x)` | 将指定参数设置为给定 Java String 值。 |

**案例**：

* 对上述有 SQL 注入的案例改进：

```java
import java.sql.*;
import java.util.Scanner;

public class JDBCDemo {

    //从控制台上输入的用户名和密码
    public static void main(String[] args) throws SQLException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入用户名： ");
        String name = sc.nextLine();
        System.out.println("请输入密码： ");
        String password = sc.nextLine();
        login(name, password);
    }

    /**
     * 登录的方法
     */
    private static void login(String name, String password) throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        //写成登录 SQL 语句，没有单引号
        String sql = "select * from user where name=? and password=?";
        //得到语句对象
        PreparedStatement ps = connection.prepareStatement(sql);
        //设置参数
        ps.setString(1, name);
        ps.setString(2,password);
        ResultSet resultSet = ps.executeQuery();
        if (resultSet.next()) {
            System.out.println("登录成功： " + name);
        }
        else {
            System.out.println("登录失败");
        }
        //释放资源,子接口直接给父接口
        JdbcUtils.close(connection,ps,resultSet);
    }
}
```

* 将返回的数据封装成对象

```java
import java.sql.*;
import java.util.Scanner;

public class JDBCDemo {

    //从控制台上输入的用户名和密码
    public static void main(String[] args) throws SQLException {

        //创建学生对象
        Student student = new Student();

        Connection connection = JdbcUtils.getConnection();
        PreparedStatement ps = connection.prepareStatement("select * from student where id=?");
        // 设置参数
        ps.setInt(1,2);
        ResultSet resultSet = ps.executeQuery();
        if (resultSet.next()) {
            // 封装成一个学生对象
            student.setId(resultSet.getInt("id"));
            student.setName(resultSet.getString("name"));
            student.setGender(resultSet.getBoolean("gender"));
            student.setBirthday(resultSet.getDate("birthday"));
        }
        // 释放资源
        JdbcUtils.close(connection,ps,resultSet);
        // 打印数据
        System.out.println(student);
    }
}

// 输出：
// Student{id=2, name='白骨精', gender=false, birthday=1995-03-24}
```

* 将多条记录封装成集合 `List<Student>`，集合中每个元素是一个 JavaBean 实体类  

```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class JDBCDemo {

    //从控制台上输入的用户名和密码
    public static void main(String[] args) throws SQLException {

        // 创建一个集合
        List<Student> students = new ArrayList<>();
        Connection connection = JdbcUtils.getConnection();
        PreparedStatement ps = connection.prepareStatement("select * from student");
        // 没有参数替换
        ResultSet resultSet = ps.executeQuery();
        while(resultSet.next()) {
            // 每次循环是一个学生对象
            Student student = new Student();
            // 封装成一个学生对象
            student.setId(resultSet.getInt("id"));
            student.setName(resultSet.getString("name"));
            student.setGender(resultSet.getBoolean("gender"));
            student.setBirthday(resultSet.getDate("birthday"));
            //把数据放到集合中
            students.add(student);
        }

        // 关闭连接
        JdbcUtils.close(connection,ps,resultSet);
        // 使用数据
        for (Student stu: students) {
            System.out.println(stu);
        }
    }
}

// 输出:
// Student{id=1, name='孙悟空', gender=true, birthday=1993-03-24}
// Student{id=2, name='白骨精', gender=false, birthday=1995-03-24}
// Student{id=3, name='猪八戒', gender=true, birthday=1903-03-24}
// Student{id=4, name='嫦娥', gender=false, birthday=1993-03-11}
```

* PreparedStatement 执行 DML 操作  

```java
import java.sql.*;

public class JDBCDemo {

    public static void main(String[] args) throws SQLException {
        // insert();
        // update();
        delete();
    }

    // 插入记录
    private static void insert() throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        PreparedStatement ps = connection.prepareStatement("insert into student values(null,?,?,?)");
        ps.setString(1,"小白龙");
        ps.setBoolean(2, true);
        ps.setDate(3,java.sql.Date.valueOf("1999-11-11"));
        int row = ps.executeUpdate();
        System.out.println("插入了" + row + "条记录");
        JdbcUtils.close(connection,ps);
    }
    
    // 更新记录: 换名字和生日
    private static void update() throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        PreparedStatement ps = connection.prepareStatement("update student set name=?, birthday=? where id=?");
        ps.setString(1,"黑熊怪");
        ps.setDate(2,java.sql.Date.valueOf("1999-03-23"));
        ps.setInt(3,5);
        int row = ps.executeUpdate();
        System.out.println("更新" + row + "条记录");
        JdbcUtils.close(connection,ps);
    }

    // 删除记录: 删除第 5 条记录
    private static void delete() throws SQLException {
        Connection connection = JdbcUtils.getConnection();
        PreparedStatement ps = connection.prepareStatement("delete from student where id=?");
        ps.setInt(1,5);
        int row = ps.executeUpdate();
        System.out.println("删除了" + row + "条记录");
        JdbcUtils.close(connection,ps);
    }
}
```

## 8. JDBC 事务的处理

之前我们是使用 MySQL 的命令来操作事务。接下来我们使用 JDBC 来操作银行转账的事务

* 准备数据：

  ```mysql
  CREATE TABLE account (
      id INT PRIMARY KEY AUTO_INCREMENT,
      NAME VARCHAR(10),
      balance DOUBLE
  );
  -- 添加数据
  INSERT INTO account (NAME, balance) VALUES ('Jack', 1000), ('Rose', 1000);
  
  -- 查看表
  mysql> select * from account;
  +----+------+---------+
  | id | NAME | balance |
  +----+------+---------+
  |  1 | Jack |    1000 |
  |  2 | Rose |    1000 |
  +----+------+---------+
  2 rows in set (0.00 sec)
  ```

* API 介绍  

  | Connection 接口中与事务有关的方法        | 说明                                                         |
  | ---------------------------------------- | ------------------------------------------------------------ |
  | `void setAutoCommit(boolean autoCommit)` | 参数是 true 或 false；<br />如果设置为 false，表示关闭自动提交，相当于开启事务 |
  | `void commit()`                          | 提交事务                                                     |
  | `void rollback()`                        | 回滚事务                                                     |

* 案例：当然还是转账

  ```java
  import java.sql.*;
  
  public class JDBCDemo {
  
      // 没有异常，提交事务，出现异常回滚事务
      public static void main(String[] args) throws SQLException {
  
          // 1) 注册驱动
          Connection connection = null;
          PreparedStatement ps = null;
          try {
              // 2) 获取连接
              connection = JdbcUtils.getConnection();
              // 3) 开启事务
              connection.setAutoCommit(false);
              // 4) 获取到 PreparedStatement
              // 从 jack 扣钱
              ps = connection.prepareStatement("update account set balance = balance - ? where name=?");
              ps.setInt(1, 500);
              ps.setString(2,"Jack");
              ps.executeUpdate();
              // 出现异常
              System.out.println(100 / 0);
              // 给 rose 加钱
              ps = connection.prepareStatement("update account set balance = balance + ? where name=?");
              ps.setInt(1, 500);
              ps.setString(2,"Rose");
              ps.executeUpdate();
              // 5) 提交事务
              connection.commit();
              System.out.println("转账成功");
          } catch (Exception e) {
              e.printStackTrace();
              try {
                  // 6) 事务的回滚
                  connection.rollback();
              } catch (SQLException e1) {
                  e1.printStackTrace();
              }
              System.out.println("转账失败");
          }
          finally {
              // 7) 关闭资源
              JdbcUtils.close(connection,ps);
          }
      }
  }
  ```

