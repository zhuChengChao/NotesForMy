# 安装多个版本的MySQL

> 之前在 WinPC 机上安装了 MySQL 5.5，然后在刷题过程中发现了窗口函数，而窗口函数是 MySQL8 以后才支持的，故在本地又安装了一个 MySQL8，记录一下安装过程。

## 安装MySQL 5.5

1. 进入mysql的官网，找到相应的位置下载，我下载了 5.5.40 版本的：

   地址如下：https://downloads.mysql.com/archives/community/

   ![mysql5下载](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251728770.PNG)

2. 由于我已经安装了...我也不想再卸载再安装，后续如果重装了再来填坑吧，具体参照：

   [MySQL 5.5安装配置教程](https://blog.csdn.net/weixin_38239039/article/details/79629984)

   看了下和我之前安装的基本一致，其中下载的地址有点偏差，我在 1. 给出了更新后地址

## 安装 MySQL 8

> 首先说明此时PC的情况：
>
> 已经安装了 MySQL 5.5，在环境变量中也添加了相应的内容：`C:\Software\MySQL\bin`，因此可通过cmd直接对mysql进行操作，如下：
>
> ```bash
> C:\Users\ZhuCC>mysql -uroot -proot
> Welcome to the MySQL monitor.  Commands end with ; or \g.
> Your MySQL connection id is 1
> Server version: 5.5.40 MySQL Community Server (GPL)
> 
> Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.
> 
> Oracle is a registered trademark of Oracle Corporation and/or its
> affiliates. Other names may be trademarks of their respective
> owners.
> 
> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
> ```
>
> Over

1. 关闭 MySQL 5.5 的服务：`win + r`  ---> `services.msc` ---> `在服务窗口中找到MySQL，停止其服务`

2. 下载 MySQL 8，网址如下：`https://dev.mysql.com/downloads/mysql/`，直接可下：

   ![mysql8下载](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251728867.PNG)

   > 可以下载最新的吧，或者去`https://downloads.mysql.com/archives/community/`下载过往版本
   >
   > 我下载的是 8.0.13 不过现在已经不是最新的了

3. 将下载的`mysql-8.0.13-winx64.zip`文件，解压到你需要安装的路径即可，如

   ```bash
   C:\Software\MySQL8\mysql-8.0.13-winx64
   ```

4. 把新安装的 MySQL8 加入环境变量

   ![mysql8添加环境变量](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251728723.PNG)

5. 编辑 MySQL8 的 `my.init`文件，若不存在则直接自己创建，如下：

   > 例如，我刚刚的文件解压在C:\Software\MySQL8\mysql-8.0.13-winx64，在其内创建my.ini
   >
   > **下面的部分内容需要根据本地的配置进行相应修改**
   
   ```bash
   # 内容如下：
   [mysqld]
   # 设置3307端口，为了与旧版本的区分不冲突
   port=3307
   # 设置mysql的安装目录
   # 切记此处一定要用双斜杠\\，单斜杠我这里会出错，不过看别人的教程，有的是单斜杠。自己尝试吧
   basedir=C:\\Software\\MySQL8\\mysql-8.0.13-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=C:\\Software\\MySQL8\\date
   # 允许最大连接数
   max_connections=200
   # 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
   max_connect_errors=10
   # 服务端使用的字符集默认为UTF8
   character-set-server=utf8
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   # MySQL8.0.4之前，MySQL的密码认证插件是“mysql_native_password”，而现在使用的是“caching_sha2_password”。
   # 但因为当前有很多数据库工具和链接包都不支持“caching_sha2_password”，为了方便，暂时改回了“mysql_native_password”认证插件。
   # 默认使用“mysql_native_password”插件认证
   default_authentication_plugin=mysql_native_password
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8
   [client]
   # 设置mysql客户端连接服务端时默认使用的端口
   port=3307
   default-character-set=utf8
   ```
   
6. 初始化 MySQL，**以管理员的**身份运行cmd，如下：

   ```bash
   # 进入如下目录：
   C:\Windows\system32>cd C:\Software\MySQL8\mysql-8.0.13-winx64
   # 运行如下命令：mysqld --initialize --console
   C:\Software\MySQL8\mysql-8.0.13-winx64>mysqld --initialize --console
   2021-01-25T13:10:56.200416Z 0 [System] [MY-013169] [Server] C:\Software\MySQL8\mysql-8.0.13-winx64\bin\mysqld.exe (mysqld 8.0.13) initializing of server in progress as process 14848
   2021-01-25T13:10:56.213381Z 0 [Warning] [MY-013242] [Server] --character-set-server: 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous.
   2021-01-25T13:11:01.188180Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ikYTWgn=y7q.  # 记录此处的密码，后续登录需要使用
   2021-01-25T13:11:02.342337Z 0 [System] [MY-013170] [Server] C:\Software\MySQL8\mysql-8.0.13-winx64\bin\mysqld.exe (mysqld 8.0.13) initializing of server has completed
   ```

7. 进入 bin 目录，运行如下：mysqld --install [服务名]

   > mysql 5.5 的服务器名为 MySQL，因此此处我起名为 MySQL8

   ```bash
   C:\Software\MySQL8\mysql-8.0.13-winx64>cd bin
   C:\Software\MySQL8\mysql-8.0.13-winx64\bin>mysqld --install MySQL8
   Service successfully installed.  # 成功安装
   ```

   > 安装成功后，通过 `win + r ---> services.msc ---> 在服务中找到新添加的服务 MySQL8`
   >
   > ![mysql8服务安装成功](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251728689.PNG)

8. 继续在改终端下启动 mysql8，`net start MySQL8`，

   ```bash
   C:\Software\MySQL8\mysql-8.0.13-winx64\bin>net start MySQL8
   MySQL8 服务正在启动 .
   MySQL8 服务已经启动成功。
   ```

   > 其中 MySQL8 为上一步你起的服务名，还有如下相关命令：
   >
   > * 启动服务：`net start 服务名`
   > * 关闭服务：`net stop 服务名`
   > * 重启服务：`net restart 服务名`

9. 登录 mysql8 修改密码：继续在终端中，输入`mysql -u root -p`

   ```bash
   C:\Software\MySQL8\mysql-8.0.13-winx64\bin>mysql -u root -p
   Enter password: ************  # 这里的密码就是第6步：初始化mysql时的密码
   # 注：手打没有打对过，因此在前面复制密码，然后在这里鼠标右击即可自动复制
   # 连接成功！
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 8
   Server version: 8.0.13
   
   Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
   
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   
   mysql>
   ```

10. 修改密码：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码'`

    ```bash
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
    Query OK, 0 rows affected (0.07 sec)
    # 修改成功，MySQL8 也安装成功
    ```

11. SQLyog 连接，mysql8，如下：

    ![mysql8安装连接sqlyog](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251728894.PNG)

## 参考

https://www.cnblogs.com/zwesy/p/9276322.html



