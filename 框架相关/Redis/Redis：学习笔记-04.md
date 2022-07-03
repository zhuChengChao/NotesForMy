# Redis：学习笔记-04

> 根据 bilibili 上讲解 Redis 中观看较多的视频课程而顺带做的笔记，课程地址：[Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB)，总计分为4篇，此为 4/4 篇。
>

## 10. Redis主从复制

### 10.1 概念

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称为从节点(slave/follower)；

**数据的复制是单向的，只能由主节点到从节点；**

Master以写为主，Slave 以读为主；

**默认情况下，每台Redis服务器都是主节点；**  

且一个主节点可以有多个从节点(或没有从节点)，但**一个从节点只能有一个主节点**。

**主从复制的作用主要包括：**

1. **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3. **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. **高可用（集群）基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的，原因如下：

1. 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大；
2. 从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，单台Redis最大使用内存不应该超过20G。
3. 电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是"多读少写"。

对于这种场景，我们可以使如下这种架构： 

   ![主从架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708974.png)

主从复制，读写分离；80% 的情况下都是在进行读操作；减缓服务器的压力；架构中经常使用，一主二从，只要在公司中，主从复制就是必须要使用的，因为在真实的项目中不可能单机使用Redis

> 除了上一种一对多的模型，主从节点构建的另一种模式：类似链路模式
>
> ![链式模式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708486.png)
>
> ```bash
> ####### 6379 的信息： #######
> 127.0.0.1:6379> info replication
> # Replication
> role:master  		# 主节点
> connected_slaves:1  # 有一个从节点
> slave0:ip=127.0.0.1,port=6380,state=online,offset=1186,lag=1
> master_repl_offset:1186
> repl_backlog_active:1
> repl_backlog_size:1048576
> repl_backlog_first_byte_offset:2
> repl_backlog_histlen:1185
> 
> ####### 6380 的信息： #######
> 127.0.0.1:6380> info replication
> # Replication
> role:slave                    # 作为slave
> master_host:127.0.0.1         # 依赖的主机信息
> master_port:6379
> master_link_status:up
> master_last_io_seconds_ago:4
> master_sync_in_progress:0
> slave_repl_offset:1242
> slave_priority:100
> slave_read_only:1
> connected_slaves:1			  # 绑定该从机的从机信息
> slave0:ip=127.0.0.1,port=6381,state=online,offset=211,lag=0
> master_repl_offset:211
> repl_backlog_active:1
> repl_backlog_size:1048576
> repl_backlog_first_byte_offset:2
> repl_backlog_histlen:210
> 
> ####### 6381 的信息： #######
> 127.0.0.1:6381> info replication
> # Replication
> role:slave                    # 作为slave
> master_host:127.0.0.1
> master_port:6380
> master_link_status:up
> master_last_io_seconds_ago:3
> master_sync_in_progress:0
> slave_repl_offset:253
> slave_priority:100
> slave_read_only:1
> connected_slaves:0			  # 绑定该从机的从机信息，为0，没有绑定
> master_repl_offset:0
> repl_backlog_active:0
> repl_backlog_size:1048576
> repl_backlog_first_byte_offset:0
> repl_backlog_histlen:0
> ```

### 10.2 环境配置

只配置从库，不用配置主库，主库是默认配置

```bash
127.0.0.1:6379> info replication	# 查看当前库的信息
# Replication
role:master				# 角色 master
connected_slaves:0		# 没有从机
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制3个配置文件，然后修改对应的信息

1. 端口：`port`
2. pid 名字：在windows上是不需要的
3. log文件名字：`logfile`
4. dump.rdb 名字：`dbfilename`

修改完毕之后，启动我们的3个redis服务器

```bash
redis-server.exe ./config/redis.windows-service-6379.conf  # 启动服务器1
redis-server.exe ./config/redis.windows-service-6380.conf  # 启动服务器1
redis-server.exe ./config/redis.windows-service-6381.conf  # 启动服务器1
redis-cli.exe -p 6379	# 客户端连接服务器1
redis-cli.exe -p 6380	# 客户端连接服务器2
redis-cli.exe -p 6381	# 客户端连接服务器3
```

可以通过进程信息查看。

### 10.3 主从配置

默认情况下，每台 Redis 服务器都是主节点； 我们一般情况下只用配置从机就好了。

认老大： 一主 （79）二从（80，81）  

1. 80端口从机配置：

   > 命令：
   >
   > 1. 选择主节点，作为从机：`slaveof ip port` 
   > 2. 脱离主机：`salveof no one`

    ```bash
    127.0.0.1:6380> slaveof 127.0.0.1 6379  # 找谁当自己的老大 
    OK
    127.0.0.1:6380> info replication
    # Replication
    role:slave				# 当前角色是从机
    master_host:127.0.0.1	# 可以的看到主机的信息
    master_port:6379
    master_link_status:up
    master_last_io_seconds_ago:6
    master_sync_in_progress:0
    slave_repl_offset:1
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_repl_offset:0
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
    ```

2. 81端口从机配置：

    ```bash
    127.0.0.1:6381> slaveof 127.0.0.1 6379
    OK
    127.0.0.1:6381> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6379
    master_link_status:up
    master_last_io_seconds_ago:7
    master_sync_in_progress:0
    slave_repl_offset:197
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_repl_offset:0
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
    ```

3. 查看79端口的主机信息：

    ```bash
    127.0.0.1:6379> info replication
    # Replication
    role:master
    connected_slaves:2  # 多了从机的配置
    slave0:ip=127.0.0.1,port=6380,state=online,offset=253,lag=1
    slave1:ip=127.0.0.1,port=6381,state=online,offset=253,lag=0
    master_repl_offset:267
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:2
    repl_backlog_histlen:266
    ```

**注意**：真实的从主配置应该在配置文件中配置，这样的话是永久的，上述使用的是命令，暂时的。

**操作**：

主机可以写，从机不能写只能读，主机中的所有信息和数据，都会自动被从机保存。

1. 主机写

   ```bash
   127.0.0.1:6379> set key1 value1
   OK
   ```

2. 从机只能读

   ```bash
   127.0.0.1:6380> get key1  		 # 可以读取
   "value1"
   127.0.0.1:6380> set key2 value2  # 但是不能写
   (error) READONLY You can't write against a read only slave.
   ```

3. 测试：

   1. 主机断开连接，从机依旧连接到主机的，但是没有写操作，这个时候，主机如果回来了，从机依旧可以直接获取到主机写的信息；
   2. 如果是使用命令行来配置的主从，这个只要重启了，就会变回主机；而当一旦配置从机，立马就会从主机中获取值。

### 10.4 主从复制原理

slave 启动成功连接到 master 后会发送一个 sync 同步命令， master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到 slave，并完成一次完全同步。

**全量复制**：在 slave 服务在接收到数据库文件数据后，将其存盘并加载到内存中。

**增量复制**：Master 继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行，我们的数据一定可以在从机中看到。

### 10.5 哨兵模式

> 当没有哨兵模式时：如果master宕机了，此时需要手动选择新的master
>
> 即：如果主机断开了连接，我们可以使用 `SLAVEOF no one` 让从机变成主机，其他的节点就可以手动连接到最新的这个主节点；而如果这个时候老大修复了，那就重新连接当前的主机。

而哨兵模式，则是当主机宕机时，自动选择一个从机作为主机。

主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑**哨兵模式**。Redis从2.8开始正式提供了 Sentinel（哨兵） 架构来解决这个问题。

谋朝篡位的自动版，能够后台监控主机是否故障，如果故障了根据投票数**自动将从库转换为主库**。

哨兵模式是一种特殊的模式，首先 Redis 提供了哨兵的命令，哨兵是一个**独立的进程**，作为进程，它会独立运行。**其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**  

![哨兵模式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708703.PNG)

这里的哨兵有两个作用

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到 master 宕机，会自动将 slave 切换成 master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了**多哨兵模式**。  

![多哨兵模式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708122.PNG)

**测试：**目前的配置是：一主二从

1. 配置哨兵配置文件 sentinel.conf，此处配置三个哨兵：

   ```bash
   # 哨兵1：
   bind 127.0.0.1
   port 26379
   # sentinel monitor 被监控的名称 host port 2
   sentinel monitor myredis 127.0.0.1 6379 2
   sentinel down-after-milliseconds myredis 5000
   sentinel config-epoch myredis 1
   sentinel leader-epoch myredis 1
   
   # 哨兵2：
   bind 127.0.0.1
   port 26479
   # sentinel monitor 被监控的名称 host port 2
   sentinel monitor myredis 127.0.0.1 6379 2
   sentinel down-after-milliseconds myredis 5000
   sentinel config-epoch myredis 1
   sentinel leader-epoch myredis 1
   
   # 哨兵3:
   bind 127.0.0.1
   port 26579
   # sentinel monitor 被监控的名称 host port 2
   sentinel monitor myredis 127.0.0.1 6379 2
   sentinel down-after-milliseconds myredis 5000
   sentinel config-epoch myredis 1
   sentinel leader-epoch myredis 1
   ```

   > 对上述参数说明：
   >
   > 1. `port` ：当前Sentinel服务运行的端口  
   > 2. `sentinel monitor myredis 127.0.0.1 6379 2`：Sentinel去监视一个名为myredis的主redis实例，这个主实例的IP地址为本机地址127.0.0.1，端口号为6379，而将这个主实例判断为失效至少需要2个Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行  
   > 3. `sentinel down-after-milliseconds mymaster 5000`：指定了Sentinel认为Redis实例已经失效所需的毫秒数。当实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行  
   > 4. `sentinel parallel-syncs mymaster 1`：指定了在执行故障转移时，最多可以有多少个从 Redis 实例在同步新的主实例，在从 Redis 实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长  

2. 首先启动redis服务，并配置好主从机；

3. 启动哨兵：

   ```bash
   redis-server.exe ./config/sentinel-26379.conf --sentinel
   redis-server.exe ./config/sentinel-26479.conf --sentinel
   redis-server.exe ./config/sentinel-26579.conf --sentinel
   ```
   
> linux版本中，通过 `redis-sentinel 配置文件` 进行启动。

启动成后显示：按照26379这个端口的显示内容：

```bash
   C:\Software\Redis>redis-server.exe ./config/sentinel-26379.conf --sentinel
                   _._
              _.-``__ ''-._
         _.-``    `.  `_.  ''-._           Redis 3.2.100 (00000000/0) 64 bit
     .-`` .-```.  ```\/    _.,_ ''-._
    (    '      ,       .-`  | `,    )     Running in sentinel mode
    |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
    |    `-._   `._    /     _.-'    |     PID: 22880
     `-._    `-._  `-./  _.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |           http://redis.io
     `-._    `-._`-.__.-'_.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |
     `-._    `-._`-.__.-'_.-'    _.-'
         `-._    `-.__.-'    _.-'
             `-._        _.-'
                 `-.__.-'
   
   [22880] 20 Nov 14:15:13.024 # Sentinel ID is 341a484ba91533b84bf870cdb3d8278b640cb674
   [22880] 20 Nov 14:15:13.025 # +monitor master myredis 127.0.0.1 6379 quorum 2  # +主节点监控
   [22880] 20 Nov 14:15:13.026 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379  # +从节点监控
   [22880] 20 Nov 14:15:13.030 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379  # +从节点解控
   [22880] 20 Nov 14:17:39.505 * +sentinel sentinel 40ca19bf994416a629b670570f94a3f94ea97353 127.0.0.1 26479 @ myredis 127.0.0.1 6379  # 添加了一个哨兵
   [22880] 20 Nov 14:18:06.144 * +sentinel sentinel 7988c2ac27961bfbae4aee2d121ef7fa2786856e 127.0.0.1 26579 @ myredis 127.0.0.1 6379  # 又添加了一个哨兵
```

4. 这时如果Master节点断开了，这个时候就会从从机中随机选择一个服务器 （这里面有一个投票算法）  

   ```bash
   [22880] 20 Nov 14:22:04.201 # +sdown master myredis 127.0.0.1 6379
   [22880] 20 Nov 14:22:44.311 # +new-epoch 2
   [22880] 20 Nov 14:22:44.311 # +config-update-from sentinel 7988c2ac27961bfbae4aee2d121ef7fa2786856e 127.0.0.1 26579 @ myredis 127.0.0.1 6379
   [22880] 20 Nov 14:22:44.311 # +switch-master myredis 127.0.0.1 6379 127.0.0.1 6381  # 可以看到master宕机后，选择了6381作为主节点了
   [22880] 20 Nov 14:22:44.311 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6381
   [22880] 20 Nov 14:22:44.311 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6381
   [22880] 20 Nov 14:22:44.363 # +tilt #tilt mode entered
   ```

   从哨兵日志中可以看出，当主节点宕机后，选择了6381的从机作为了主机：

   ```bash
   127.0.0.1:6381> info replication
   # Replication
   role:master
   connected_slaves:1
   slave0:ip=127.0.0.1,port=6380,state=online,offset=24494,lag=0
   master_repl_offset:24626
   repl_backlog_active:1
   repl_backlog_size:1048576
   repl_backlog_first_byte_offset:2
   repl_backlog_histlen:24625
   ```

5. 如果主机此时回来了，只能归并到新的主机下，当做从机，这就是哨兵模式的规则 

   ```bash
   C:\Software\Redis>redis-server.exe ./config/redis.windows-service-6379.conf  # 重启刚刚宕机的前master
   
   C:\Software\Redis>redis-cli.exe -p 6379
   127.0.0.1:6379> info replication  # 发现变成从机了
   # Replication
   role:slave
   master_host:127.0.0.1
   master_port:6381
   master_link_status:up
   master_last_io_seconds_ago:1
   master_sync_in_progress:0
   slave_repl_offset:55018
   slave_priority:100
   slave_read_only:1
   connected_slaves:0
   master_repl_offset:0
   repl_backlog_active:0
   repl_backlog_size:1048576
   repl_backlog_first_byte_offset:0
   repl_backlog_histlen:0
   ```

**优点：**

1. 哨兵集群，基于主从复制模式，所有的主从配置优点，它全有；
2. 主从可以切换，故障可以转移，系统的可用性就会更好；
3. 哨兵模式就是主从模式的升级，手动到自动，更加健壮。

**缺点：**

1. Redis 不好在线扩容，集群容量一旦到达上限，在线扩容就十分麻烦；
2. 实现哨兵模式的配置其实是很麻烦的，里面有很多选择！  

> 哨兵模式的全部配置：一般默认即可
>
> ```bash
> # Example sentinel.conf
> 
> # 哨兵sentinel实例运行的端口 默认26379
> port 26379
> 
> # 哨兵sentinel的工作目录
> dir /tmp
> 
> # 哨兵sentinel监控的redis主节点的 ip port
> # master-name：可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
> # quorum：配置多少个sentinel哨兵统一认为master主节点失联那么这时客观上认为主节点失联了
> # sentinel monitor <master-name> <ip> <redis-port> <quorum>
> sentinel monitor mymaster 127.0.0.1 6379 2
> 
> # 当在Redis实例中开启了requirepass foobared授权密码，这样所有连接Redis实例的客户端都要提供密码
> # 设置哨兵sentinel连接主从的密码，注意必须为主从设置一样的验证密码
> # sentinel auth-pass <master-name> <password>
> sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
> 
> # 指定多少毫秒之后主节点没有应答哨兵sentinel，此时哨兵主观上认为主节点下线，默认30秒
> # sentinel down-after-milliseconds <master-name> <milliseconds>
> sentinel down-after-milliseconds mymaster 30000
> 
> # 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步
> # 这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用
> # 可以通过将这个值设为1来保证每次只有一个slave处于不能处理命令请求的状态。
> # sentinel parallel-syncs <master-name> <numslaves>
> sentinel parallel-syncs mymaster 1
> 
> # 故障转移的超时时间 failover-timeout 可以用在以下这些方面：
> #  1. 同一个sentinel对同一个master两次failover之间的间隔时间。
> #  2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
> #  3.当想要取消一个正在进行的failover所需要的时间。
> #  4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
> # 默认三分钟
> # sentinel failover-timeout <master-name> <milliseconds>
> sentinel failover-timeout mymaster 180000
> 
> # SCRIPTS EXECUTION
> # 配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
> # 对于脚本的运行结果有以下规则：
> # 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
> # 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
> # 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
> # 一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
> # 通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
> # 通知脚本:shell编程
> # sentinel notification-script <master-name> <script-path>
> sentinel notification-script mymaster /var/redis/notify.sh
> 
> # 客户端重新配置主节点参数脚本
> # 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
> # 以下参数将会在调用脚本时传给脚本:
> # <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
> # 目前<state>总是“failover”,
> # <role>是“leader”或者“observer”中的一个。
> # 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
> # 这个脚本应该是通用的，能被多次调用，不是针对性的。
> # sentinel client-reconfig-script <master-name> <script-path>
> sentinel client-reconfig-script mymaster /var/redis/reconfig.sh 
> # 一般都是由运维来配置！
> ```

## 11. Redis 缓存穿透和雪崩

### 11.1 服务的高可用问题  

Redis 缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。

另外的一些典型问题就是，**缓存穿透、缓存雪崩和缓存击**穿。目前，业界也都有比较流行的解决方案。  

![缓存穿透](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708339.PNG)

### 11.2 缓存穿透

**概念**：

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

**解决方案**：

1. 布隆过滤器  

   布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力

   ![布隆过滤器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708766.PNG)

2. 缓存空对象

   当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；  

   ![缓存空对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708024.PNG)

   但是这种方法会存在两个问题：

   1. 如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；
   2. 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响  

### 11.3 缓存击穿

**概述**：

这里需要注意缓存穿透和缓存击穿的区别，缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，**当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库**，就像在一个屏障上凿开了一个洞。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导使数据库瞬间压力过大。  

**解决方案**：

1. 设置热点数据永不过期：

   从缓存层面来看，没有设置过期时间，所以不会出现热点 key 过期后产生的问题。

2. 加互斥锁

   分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。 

   ![缓存击穿-加互斥锁](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708346.PNG)

### 11.4 缓存雪崩  

**概念**：

缓存雪崩，是指在某一个时间段，**缓存集中过期失效**。

产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。  

![缓存雪崩](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251708708.PNG)

其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。  

**解决方案**：

1. redis高可用

   这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。

2. 限流降级

   这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

3. 数据预热

   数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。  

