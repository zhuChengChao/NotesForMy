# Ubuntu用户管理&权限管理

> 小小记录一下 Ubuntu 中的用户管理/权限管理常用的一些命令。

本文主要分为以下几个部分：

* 用户管理
* 组管理
* 文件权限
* 给用户添加 sudo 权限
* 给用户添加 sudo 权限

> 首先先给出几个文件
>
> - `/etc/passwd`：是用于保存用户信息的文件
> - `/etc/shadow`：保存了用于的密码信息
> - `/etc/group`：保存了组信息

## 用户管理

> 提示：**创建用户/删除用户/修改其他用户密码** 的终端命令都需要通过`sudo` / 使用 `root账户` 执行

### 创建用户／设置密码／删除用户

| 命令                                           | 作用         | 说明                                                         |
| ---------------------------------------------- | ------------ | ------------------------------------------------------------ |
| useradd -m -g 组名 -u ID号 -s /bin/bash 用户名 | 添加新用户   | -m：自动建立用户家目录；<br />-g：指定用户所在的组，无则为用户名组<br />-u：指定用户ID号，ID值时尽量要大于500<br />－s：指定用户登入后所使用的shell |
| passwd 用户名                                  | 设置用户密码 | 如果是普通用户，直接用 passwd 可以修改自己的账户密码         |
| userdel -r 用户名                              | 删除用户     | -r 选项会自动删除用户家目录                                  |
| cat /etc/passwd \| grep 用户名                 | 确认用户信息 | 新建用户后，用户信息会保存在/etc/passwd文件中                |

**进一步说明**：

- 创建用户时，如果忘记添加`-m`选项指定新用户的家目录——最简单的方法就是**删除用户，重新创建**

- 创建用户时，若不指定组名则默认会创建一个和**用户名**同名的**组名**

- 用户信息保存在 `/etc/passwd` 文件中，可通过 `cat /etc/passwd`查看，里面存放的用户信息，由 6 个`:`组成的 7 个信息，分别是：

  用户名 : 密码(x，表示加密的密码) : UID(用户标识) : GID(组标识) : 用户全名或本地帐号家目录 : 登录使用的 Shell，就是登录之后，使用的终端命令，`ubuntu` 默认是 `bash`

**例如：**创建一个名为 `xianyuchao` 的账户，如下

```bash
# 1.创建账户
(base) root@ubuntu-C246-WU4:~# useradd -m -u 1314 -s /bin/bash zhangsan
# 2.查看账户
(base) root@ubuntu-C246-WU4:~# cat /etc/passwd | grep zhangsan
# 用户名:密码:UID:GID:家目录:/bin/bash
zhangsan:x:1314:1314::/home/zhangsan:/bin/bash
# 3.给账户设置密码
(base) root@ubuntu-C246-WU4:~# passwd zhangsan
输入新的 UNIX 密码： 
重新输入新的 UNIX 密码： 
passwd：已成功更新密码
# 4.切换用户
(base) root@ubuntu-C246-WU4:~# su zhangsan
zhangsan@ubuntu-C246-WU4:/root$ cd ~
zhangsan@ubuntu-C246-WU4:~$ ls
# 5.删除用户
zhangsan@ubuntu-C246-WU4:~$ exit
exit
(base) root@ubuntu-C246-WU4:~# userdel -r zhangsan
```

### 查看用户信息

| 命令        | 作用                       |
| ----------- | -------------------------- |
| id [用户名] | 查看用户UID和GID信息       |
| who         | 查看当前所有登录的用户列表 |
| whoami      | 查看当前登录用户的账户名   |

### 切换用户

| 命令        | 作用                   | 说明                                             |
| ----------- | ---------------------- | ------------------------------------------------ |
| su - 用户名 | 切换用户，并且切换目录 | `-`可以切换到用户家目录，否则保持位置不变        |
| exit        | 退出当前登录账户       |                                                  |
| sudo        | 以超级用户方式执行命令 | 在root下通过设置**visudo**设置需要使用sudo的用户 |

- `su`不接用户名，可以切换到`root`，但是不推荐使用，因为不安全
- `exit` **登陆逐级登陆，退出逐级退出**
- `sudo`，可临时赋予用户root的部分功能，具体的赋予的哪些操作需要通过root在`visudo`内进行相应的配置

### 修改用户所属组

- `usermod`可以用来设置**用户**的**主组/附加组**和**登录 Shell**，命令格式如下：
  - **主组**：通常在新建用户时指定，在`etc/passwd`的第4列**GID对应的组**
  - **附加组**：在`etc/group`中最后一列表示该组的用户列表，用于指定**用户的附加权限**

> 注：设置了用户的附加组之后，需要重新登录才能生效！

```bash
# 修改用户的主组（passwd 中的 GID）
usermod -g 组 用户名
 
# 修改用户的附加组（group文件中的最后一列）
usermod -G 组 用户名
 
# 修改用户登录Shell
usermod -s /bin/bash 用户名
```

**例：**

```bash
# 首先还是创建zhangsan用户
(base) root@ubuntu-C246-WU4:~# useradd -m -u 1314 -s /bin/bash zhangsan
# 由于 -m 因此zhangsan 账户的组为zhangsan且没有默认组、
(base) root@ubuntu-C246-WU4:~# cat /etc/passwd | grep zhangsan
zhangsan:x:1314:1314::/home/zhangsan:/bin/bash
(base) root@ubuntu-C246-WU4:~# cat /etc/group | grep zhangsan
zhangsan:x:1314:  # 此时没有附属组

# 修改zhangsan的组为 zcc 组
(base) root@ubuntu-C246-WU4:~# cat /etc/passwd | grep zcc
zcc:x:6666:6666::/home/zcc:/bin/bash  # zcc的GID为6666
(base) root@ubuntu-C246-WU4:~# usermod -g zcc zhangsan
(base) root@ubuntu-C246-WU4:~# cat /etc/passwd | grep zhangsan
zhangsan:x:1314:6666::/home/zhangsan:/bin/bash  # 修改后zhangsan的组也变成了6666

# 修改zhangsan的附加组
(base) root@ubuntu-C246-WU4:~# usermod -G zhangsan zhangsan
(base) root@ubuntu-C246-WU4:~# cat /etc/group | grep zhangsan
zhangsan:x:1314:zhangsan  # 对比上方，此时有附属组了，为zhangsan
```

## 组管理

> 提示：**创建组**/**删除组**的终端命令都需要通过`sudo`执行

| 命令                      | 作用                      |
| ------------------------- | ------------------------- |
| groupadd 组名             | 添加组                    |
| groupdel 组名             | 删除组                    |
| cat /etc/group            | 确认组信息                |
| chgrp -R 组名 文件/目录名 | 递归修改文件/目录的所属组 |

> 提示：
>
> - 组信息保存在`/etc/group`文件中
> - `/etc`目录是专门用来保存**系统配置信息**的目录

- 在实际应用中，可以预先针对**组**设置好权限，然后**将不同的用户添加到对应的组中**，从而**不用依次为每一个用户设置权限**

## 文件权限

#### 文件权限说明

`ls -l`可以查看文件夹下文件的详细信息，从左到右依次是：

```bash
d rwxr-xr-x. 21 root root 4.0k 12月 21 22：22 var
类型 权限 硬链接数 所属用于 所属用户组 大小 时间 名称
```

- 类型：`-`普通文件；`d`目录文件；`b`块特殊文件；`l`符号链接；`f`命名管道；`s`套接字文件；
- 权限：分别为**u=拥有者权限，g=组权限，o=其他用户权限**

| 权限    | 缩写 | 数字代号 |
| ------- | ---- | -------- |
| read    | r    | 4        |
| write   | w    | 2        |
| execute | x    | 1        |

> 权限限制的是非root用户，对于root用户而言没用
>
> 除了上述的读写执行权限，还有一些特殊权限：`SUID,SGID,SBIT`，不建议修改

- **硬链接数**，通俗地讲，就是有多少种方式，可以访问到当前目录/文件

#### 权限修改

| 序号 | 命令  | 作用       |
| ---- | ----- | ---------- |
| 01   | chown | 修改拥有者 |
| 02   | chgrp | 修改组     |
| 03   | chmod | 修改权限   |

- 命令格式如下：

```bash
# 修改文件|目录的拥有者
chown 用户名 文件名|目录名
# 修改文件|目录的组
chown :组名 文件名|目录名

# 递归修改文件|目录的组
chgrp -R 组名 文件名|目录名
 
# 递归修改文件权限
chmod -R 755 文件名|目录名
# 对拥有者添加执行权限
chmod u+x 文件名
# 对组删除写权限
chmod g-w 文件名
# 令其他用户的权限为可读
chmod o=r 文件名
# 对所有成员都加上读取权限
chmod a+r 文件名
```

- `chmod` 在设置权限时，可以简单地使用三个数字分别对应**u=拥有者/g=组/o=其他**用户的权限

```bash
chmod num1num2num3 文件名|目录名
```

> 当拥有者权限和组权限冲突时，已拥有者权限为准

## 给用户添加 sudo 权限

1. 用root账户打开文件 `/etc/sudoers`

2. 在如下添加你需要赋予sudo权限的用户

   ```bash
   # Allow members of group sudo to execute any command
   %sudo    ALL=(ALL:ALL) ALL
   zcc      ALL=(ALL:ALL) ALL
   zhangsan ALL=(ALL:ALL) ALL
   ```

3. `esc :wq!`保存退出

> 参考：https://www.jianshu.com/p/3d435e09712a

## 允许 root 通过 ssh 登录

1. 打开：`sudo vim /etc/ssh/sshd_config`
2. 修改如下：`PermitRootLogin yes`

> 参考：https://www.cnblogs.com/exmyth/p/10403079.html



