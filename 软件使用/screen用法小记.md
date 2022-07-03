# screen用法小记

> 首先，吹爆 screen
>
> screen，**实现了不间断的会话服务**，通过SSH连接至远程服务器，当使用了screen开启的会话，不会因为你断开SSH而中断在远程服务器上运行的命令。

screen具有如下功能：

* 会话恢复：在网路中断，ssh断开连接等情况下，只要服务器还开着，就可以对会话进行恢复；
* 多窗口：每个会话都是独立运行的；
* 会话共享：多个用户可以看到相同的一个会话，即看到看到相同的输入输出

## 基本使用

### 安装

若系统中没有安装screen，通过以下命令安装：

CentOS：

```bash
yum install screen
```

ubuntu

```
apt-get install screen
```

### 创建新的窗口

```bash
screen -S 窗口名
# 窗口创建成功后会新的一个bash串口中
```

> 最好指定一个便于记忆输入的窗口名，方便后续的窗口恢复

### 查看窗口列表

```bash
$ screen -ls

# 结果
(base) zcc@e0c362370fb8:~/GA/GA_Radio/512$ screen -ls
There are screens on:
	3890.three	(04/27/20 01:42:59)	(Detached)
	3760.two	(04/27/20 01:42:12)	(Attached)
	3542.one	(04/27/20 01:40:47)	(Attached)
3 Sockets in /var/run/screen/S-zcc.
# 可以看到目前创建了3个会话：
# 分别展示为：ID.会话名
# 两个正在前台运行，一个在后台运行
```

### 退出会话

#### 临时退出

输入快捷键

1. `Ctrl + a`
2. `Ctrl + d`

可以临时退出会话，退出后状态变为Detached状态

后续可以恢复，恢复后状态变为Attached状态

#### 永久退出/关闭

当screen开启的会话中没有在运行的命令时

通过命令：`exit`

退出会话并显示`[screen is terminating]`

### 恢复会话

1. 通过`screen -ls`查看目前存在的会话；
2. 通过`screen -rd 会话名/会话ID号`恢复会话；

例如：

```bash
$ screen -ls
# There are screens on:
# 	3890.three	(04/27/20 01:42:59)	(Detached)
$ screen -rd three
# 或者
$ screen -rd 3890
```

### 会话共享

输入命令：

`screen -x 会话名/会话ID`

共享同一个会话，不同的终端内，可以看到同一个会话的实时输入输出信息

## 参考

上述只指出了基本的日常用法，其他使用方法，参考如下地址：

https://www.cnblogs.com/lpfuture/p/5786843.html