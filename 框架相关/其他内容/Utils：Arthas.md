# Utils：Arthas

> 工欲善其事必先利其器，在工作过程中对于一些在线上不太好进行Debug的场景，这时候就可以通过 Arthas 来获取其对应方法的：输入/输出/异常/调用链路执行时间...；于是就抽空学一下 Arthas 的一些使用。
>
> > 官网文档传送门：[Arthas 用户文档](https://arthas.aliyun.com/doc/)

其实在官方文档里：[Arthas 用户文档](https://arthas.aliyun.com/doc/)，对其使用已经说的很明确了；

这里的话就想记录一下日常使用较多的一些命令之类的，方便后续个人查看；

## 准备

> 然后这里的话就先在 win 系统下测试其使用了

**下载：**打开控制台输入`curl -O https://arthas.aliyun.com/arthas-boot.jar`

```bash
C:\Software\arthas>curl -O https://arthas.aliyun.com/arthas-boot.jar  # 下载
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  138k  100  138k    0     0   138k      0  0:00:01  0:00:01 --:--:--  134k

C:\Software\arthas>java -jar arthas-boot.jar  # 运行
[INFO] arthas-boot version: 3.5.5
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 40956 org.jetbrains.jps.cmdline.Launcher
  [2]: 57500
```

**运行：**任意选择一个Java的进程即可

> 最好先下一个官方给的一个Java进程，和上面同样的方式下载，然后运行：
>
> ```bash
> C:\Software\arthas>curl -O https://arthas.aliyun.com/math-game.jar
>   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
>                                  Dload  Upload   Total   Spent    Left  Speed
> 100  4496  100  4496    0     0   4496      0  0:00:01 --:--:--  0:00:01 20529
> 
> C:\Software\arthas>java -jar math-game.jar  # 运行一个Java程序
> 13491=3*3*1499
> 169180=2*2*5*11*769
> ```

在有Java程序运行的基础上开启arthas，然后选择相应的java进程：

```bash
C:\Software\arthas>java -jar arthas-boot.jar  # 运行arthas
[INFO] arthas-boot version: 3.5.5
[INFO] Process 57500 already using port 3658
[INFO] Process 57500 already using port 8563
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 57500
  [2]: 67168 math-game.jar  # 选择这个jar进程
  [3]: 40956 org.jetbrains.jps.cmdline.Launcher

[INFO] arthas home: C:\Users\ZhuCC\.arthas\lib\3.5.5\arthas
[INFO] The target process already listen port 3658, skip attach.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'

wiki       https://arthas.aliyun.com/doc
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html
version    3.5.5
main_class
pid        57500
time       2022-02-13 21:39:08
[arthas@67168]$  # 进入arthas的命令行了
[arthas@67168]$
```

> 日常使用其实根本不用自己装，运维会去装好滴啦，分工明确！

## 常用命令

> 最好的话就是去IDEA中下载一个插件：`arthas idea`，这样的话在idea中可以直接选择需要查看的方法通过插件直接来获取到arthas的命令了。

这部分就记录一下个人在工作中使用过的一些命令。

### watch

**作用**：观察到指定函数的调用情况，能观察到的范围为：返回值、抛出异常、入参，通过编写OGNL表达式进行对应变量的查看。

> 使用的比较多的命令，可以直接用插件在方法上生成命令，如：`watch 包名.类名 方法名 '{params,returnObj,throwExp}' -v -n 5 -x 3 '1==1'`

**参数说明：**

| 参数名称          | 参数说明                                                    |
| ----------------- | ----------------------------------------------------------- |
| class-pattern     | 类名表达式匹配                                              |
| method-pattern    | 函数名表达式匹配                                            |
| express           | 观察表达式，默认值：`{params, target, returnObj}`，支持OGNL |
| condition-express | 条件表达式，支持OGNL                                        |
| [b]               | 在**函数调用之前**观察，默认关闭                            |
| [e]               | 在**函数异常之后**观察，默认关闭                            |
| [s]               | 在**函数返回之后**观察，默认关闭                            |
| [f]               | 在**函数结束之后**(正常返回和异常返回)观察，**默认打开**    |
| [E]               | 开启正则表达式匹配，默认为通配符匹配                        |
| [x:]              | 指定输出结果的属性遍历深度，默认为 1                        |

**特别说明**：

- watch 命令定义了**4个观察事件点**，即 `-b` 函数调用前，`-e` 函数异常后，`-s` 函数返回后，`-f` 函数结束后
- 4个观察事件点 `-b`、`-e`、`-s` 默认关闭，**`-f` 默认打开**，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
- 这里要注意`函数入参`和`函数出参`的区别，有可能在中间被修改导致前后不一致，除了 `-b` 事件点 `params` 代表函数入参外，其余事件都代表函数出参
- 当使用 `-b` 时，由于观察事件点是在函数调用前，此时返回值或异常均不存在
- 在watch命令的结果里，会打印出`location`信息。**`location`有三种可能值**：`AtEnter`，`AtExit`，`AtExceptionExit`。对应函数入口，函数正常return，函数抛出异常。

**使用说明：**

* **常用命令**：`watch demo.MathGame primeFactors '{params,returnObj,throwExp}' -x 2 -n 2 `

  结果：

  ![watch执行结果-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425033.PNG)

  > * 这里的`'{params,returnObj,throwExp}'`就可以按需设置了，比如我只要异常信息就设置一下`'{throwExp}'`
  > * `-n 2`：表明执行2次就退出watch；
  > * `-x 2`：输出结果的属性遍历深度，即对象里面还有对象成员，加大这个就可以输出，后续进一步说明

* **关于`-x`命令**：`watch demo.MathGame primeFactors '{target}' -x 3 -n 1 `

  > `'{target}'`表明的是当前的对象信息，因为这里函数出入输出对于`-x`体现不明朗因而用了当前对象

  ![watch执行结果-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425680.PNG)

  对比一下`-x 2`：

  ![watch执行结果-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425165.PNG)

* **函数调用前后命令**：`watch demo.MathGame primeFactors "{params,target,returnObj}" -x 2 -b -s -n 2`

  > 采用了`-s -b`，表明：第一次输出的是函数调用前的观察表达式的结果，第二次输出的是函数返回后的表达式的结果；
  >
  > 注意：这里由于是执行前和执行后，因此`-n 2`，不然只会输出执行前的结果。

  ![watch执行结果-4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425797.PNG)

* **关于条件表达式**

  * **例1参数过滤**：`watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"`
  
    > 即：只有满足条件的调用，才会有响应，上述的话是传入参数`"params[0]<0"`才输出watch结果
  
  ```bash
  [arthas@62532]$ watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"
  Press Q or Ctrl+C to abort.
  Affect(class count: 1 , method count: 1) cost in 20 ms, listenerId: 16
  method=demo.MathGame.primeFactors location=AtExceptionExit
  ts=2022-02-19 13:01:42; [cost=0.3774ms] result=@ArrayList[
      @Integer[-169794],
      @MathGame[demo.MathGame@5a10411],
  ]
  ```
  
  * **例2耗时过滤**：`watch demo.MathGame primeFactors '{params, returnObj}' '#cost>20' -x 2`
  
    > 即：当函数rt时间大于20ms才输出，这样就可以过滤掉执行时间小于200ms的调用
  
  ```bash
  [arthas@62532]$ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>1' -x 2  # 这里的话因为函数执行时间很短，因此设置了1
  Press Q or Ctrl+C to abort.
  Affect(class count: 1 , method count: 1) cost in 16 ms, listenerId: 33
  method=demo.MathGame.primeFactors location=AtExit
  ts=2022-02-19 13:30:50; [cost=1.3509ms] result=@ArrayList[
      @Object[][
          @Integer[1],
      ],
      @ArrayList[
          @Integer[106657],
      ],
  ]
  ```
  
* **`target`的说明**：表明当前对象中的属性，可以使用`target`关键字，代表当前对象

  * 查看当前对象：`watch demo.MathGame primeFactors 'target'`

    ```bash
    [arthas@62532]$ watch demo.MathGame primeFactors 'target'
    Press Q or Ctrl+C to abort.
    Affect(class count: 1 , method count: 1) cost in 18 ms, listenerId: 34
    method=demo.MathGame.primeFactors location=AtExceptionExit
    ts=2022-02-19 13:33:04; [cost=0.2689ms] result=@MathGame[
        random=@Random[java.util.Random@7a07c5b4],
        illegalArgumentCount=@Integer[4389],
    ]
    ```

  * 访问当前对象的某个属性：`watch demo.MathGame primeFactors 'target.illegalArgumentCount'`

    ```bash
    [arthas@62532]$ watch demo.MathGame primeFactors 'target.illegalArgumentCount'
    Press Q or Ctrl+C to abort.
    Affect(class count: 1 , method count: 1) cost in 19 ms, listenerId: 35
    method=demo.MathGame.primeFactors location=AtExit
    ts=2022-02-19 13:34:44; [cost=0.1083ms] result=@Integer[4435]
    method=demo.MathGame.primeFactors location=AtExit
    ```

  * 获取类中的静态字段、调用类的静态函数：`watch demo.MathGame * '{params,@demo.MathGame@random.nextInt(100)}' -n 1 -x 2`

    ```bash
    [arthas@62532]$ watch demo.MathGame * '{params,@demo.MathGame@random.nextInt(100)}' -n 1 -x 2
    Press Q or Ctrl+C to abort.
    Affect(class count: 1 , method count: 5) cost in 22 ms, listenerId: 36
    method=demo.MathGame.primeFactors location=AtExceptionExit
    ts=2022-02-19 13:40:22; [cost=0.0815ms] result=@ArrayList[
        @Object[][
            @Integer[-110932],
        ],
        @Integer[33],
    ]
    Command execution times exceed limit: 1, so command will exit. You can set it with -n option.
    ```

    > 其中Demo中random为静态成员字段：`private static Random random = new Random();`

### trace

**作用：**方法内部调用路径，并**输出方法路径上的每个节点上耗时**（很好用）

> `trace` 能方便的帮助你定位和发现因 RT 高而导致的性能问题缺陷，**但其每次只能跟踪一级方法的调用链路，**因此多级别调用的话就需要**一步步的trace** / **动态trace** / **利用正则表匹配**

**参数说明：**

| 参数名称          | 参数说明                             |
| ----------------- | ------------------------------------ |
| class-pattern     | 类名表达式匹配                       |
| method-pattern    | 方法名表达式匹配                     |
| condition-express | 条件表达式                           |
| [E]               | 开启正则表达式匹配，默认为通配符匹配 |
| [n:]              | 命令执行次数                         |
| #cost             | 方法执行耗时                         |

**使用说明：**

* **基础使用：**`trace demo.MathGame run -n 1`

  ![trace结果-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425773.PNG)

  > 上述`-n 1`和之前说明的一样，就是trace一次就退出，默认是会一直trace的，即每次调用相应方法都会trace

* **是否跳过jdk的函数**：`--skipJDKMethod <value>`，默认情况下，trace不会包含jdk里的函数调用，如果希望trace jdk里的函数，需要显式设置`--skipJDKMethod false`

  ![trace结果-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425196.PNG)

* **据调用耗时过滤：**`trace demo.MathGame run '#cost > 1' -n 1`

  ![trace结果-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425725.PNG)

  > 只会展示耗时大于1ms的调用路径，有助于在排查问题的时候，只关注异常情况；
  >
  > 这里存在一个统计不准确的问题，就是所有方法耗时加起来可能会小于该监测方法的总耗时，这个是由于 Arthas 本身的逻辑会有一定的耗时

* **动态trace**：因为trace其每次只能跟踪一级方法的调用链路，可以通过动态trace得到一定程度解决，如下：

  * 打开终端1，trace上面demo里的`run`函数，可以看到打印出 `listenerId: 1`：

    ```bash
    [arthas@70700]$ trace demo.MathGame run
    Press Q or Ctrl+C to abort.
    Affect(class count: 1 , method count: 1) cost in 81 ms, listenerId: 1
    `---ts=2022-02-19 15:50:06;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
        `---[7.0375ms] demo.MathGame:run()
            `---[0.0668ms] demo.MathGame:primeFactors() #24 [throws Exception]
    ```

  * 想要深入子函数`primeFactors`，可以打开一个新终端2，使用`telnet localhost 3658`连接上arthas，再`trace primeFactors`时，指定`listenerId`。

    > win下的telnet开启：https://jingyan.baidu.com/article/8065f87f67d4b52331249806.html；
    >
    > 然后端口3658，则是arthas的默认情况下使用的端口

    ```bash
    [arthas@70700]$ trace demo.MathGame primeFactors --listenerId 1
    Press Q or Ctrl+C to abort.                                                       Affect(class count: 1 , method count: 1) cost in 28 ms, listenerId: 1
    ```

    这时终端2打印的结果，说明已经增强了一个函数：`Affect(class count: 1 , method count: 1)`，但不再打印更多的结果。

  * 再查看终端1，可以发现trace的结果增加了一层，打印了`primeFactors`函数里的内容：

    ```
    `---ts=2022-02-19 15:51:35;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
        `---[0.6494ms] demo.MathGame:run()
            +---[0.2029ms] demo.MathGame:primeFactors() #24
            |   `---[0.1812ms] demo.MathGame:primeFactors()  # 可以看到多加了一层的trace
            `---[0.3999ms] demo.MathGame:print() #25
    
    `---ts=2022-02-19 15:51:36;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
        `---[0.8585ms] demo.MathGame:run()
            +---[0.3161ms] demo.MathGame:primeFactors() #24
            |   `---[0.2888ms] demo.MathGame:primeFactors()
            `---[0.5058ms] demo.MathGame:print() #25
    
    `---ts=2022-02-19 15:51:37;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
        `---[0.5411ms] demo.MathGame:run()
            `---[0.0941ms] demo.MathGame:primeFactors() #24 [throws Exception]
                `---[0.0787ms] demo.MathGame:primeFactors() [throws Exception]
                    `---throw:java.lang.IllegalArgumentException #46 [number is: -151442, need >= 2]
    ```

  > 通过指定`listenerId`的方式动态trace，可以不断深入。另外 `watch`/`tt`/`monitor`等命令也支持类似的功能。

* **trace多个类或者多个函数：**trace命令只会trace匹配到的函数里的子调用，**并不会向下trace多层**。因为trace是代价比较贵的，多层trace可能会导致最终要trace的类和函数非常多。但可以用**正则表匹配路径上的多个类和函数**，一定程度上达到多层trace的效果；比如：

  ```bash
  trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
  ```

* **排除掉指定的类**：使用 `--exclude-class-pattern` 参数可以排除掉指定的类，比如：

  ```bash
  trace javax.servlet.Filter * --exclude-class-pattern com.demo.TestFilter
  ```

### stack

**作用：**输出当前方法被调用的调用路径，很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。

**参数说明：**

| 参数名称          | 参数说明                             |
| ----------------- | ------------------------------------ |
| class-pattern     | 类名表达式匹配                       |
| method-pattern    | 方法名表达式匹配                     |
| condition-express | 条件表达式                           |
| [E]               | 开启正则表达式匹配，默认为通配符匹配 |
| [n:]              | 执行次数限制                         |

**使用说明：**

* 基础使用：`stack demo.MathGame primeFactors -n 1`

  > 获取方法demo.MathGame.primeFactors的调用路径

  ![stack使用-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425315.PNG)

* **据条件表达式来过滤：**`stack demo.MathGame primeFactors 'params[0]<0' -n 2`

  ```bash
  [arthas@70700]$ stack demo.MathGame primeFactors 'params[0]<0' -n 2
  Press Q or Ctrl+C to abort.
  Affect(class count: 1 , method count: 1) cost in 14 ms, listenerId: 3
  ts=2022-02-19 15:59:19;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
      @demo.MathGame.primeFactors()
          at demo.MathGame.run(MathGame.java:24)
          at demo.MathGame.main(MathGame.java:16)
  
  ts=2022-02-19 15:59:21;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@5c647e05
      @demo.MathGame.primeFactors()
          at demo.MathGame.run(MathGame.java:24)
          at demo.MathGame.main(MathGame.java:16)
  ```

* **据执行时间来过滤：**`stack demo.MathGame primeFactors '#cost>1'`

  > 大同小异

### monitor

**作用：用于监视指定类中方法的执行情况**，即：对匹配 `class-pattern`／`method-pattern`／`condition-express`的类、方法的调用进行监控。

> `monitor` 命令是一个非实时返回命令。

**参数说明：**

| 参数名称          | 参数说明                                |
| ----------------- | --------------------------------------- |
| class-pattern     | 类名表达式匹配，即类名                  |
| method-pattern    | 方法名表达式匹配，即方法名              |
| condition-express | 条件表达式                              |
| [E]               | 开启正则表达式匹配，默认为通配符匹配    |
| [c:]              | 统计周期，默认值为120秒                 |
| [b]               | 在**方法调用之前**计算condition-express |

> 用idea插件默认生成的monitor命令：`monitor demo.MathGame primeFactors -v -n 10 --cycle 10 '1==1'`，可见其也支持`[n:]`参数，**即每运行10次命令输出结果，且不断循环。**

**使用说明：**

* 监控方法`demo.MathGame primeFactors`5s后输出结果：`monitor -c 5 demo.MathGame primeFactors`

  输出内容：

  ![monitor输出结果](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425713.PNG)

* 计算条件表达式过滤统计结果(**方法执行完毕之后**)

  `monitor -c 5 demo.MathGame primeFactors "params[0] <= 2"`

  > 条件表达式：`"params[0] <= 2"`，即返回值需要满足上述条件

* 计算条件表达式过滤统计结果(**方法执行完毕之前**)

  `monitor -b -c 5 com.test.testes.MathGame primeFactors "params[0] <= 2"`

  > 条件表达式：`"params[0] <= 2"`，即传入参数需要满足`"params[0] <= 2"`

### tt（TimeTunnel）

> 在工作中没怎么用过，考虑完整性就记录一下:dog2:

**作用：**方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

> `watch` 虽然很方便和灵活，但需要提前想清楚观察表达式的拼写，这对排查问题而言要求太高，因为很多时候我们并不清楚问题出自于何方，只能靠蛛丝马迹进行猜测。
>
> 这个时候如果能记录下当时方法调用的所有入参和返回值、抛出的异常会对整个问题的思考与判断非常有帮助。

**参数说明：**

| 参数名称  | 参数说明                         |
| --------- | -------------------------------- |
| -t        | 记录某个方法在一个时间段内的调用 |
| -l        | 列出之前所有时间片段             |
| -s 表达式 | 搜索表达式，不像-l一样列出所有   |
| -i 索引号 | 查看指定索引号的详细调用信息     |
| -p        | 重新调用一次指定的方法           |

**使用说明：**

* 基本使用：`tt -t demo.MathGame primeFactors`

  > 对于一个最基本的使用来说，就是记录下当前方法的每次调用环境现场。

  ![tt使用-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425974.PNG)

  > 字段说明：
  >
  > | 表格字段  | 字段解释                                                     |
  > | --------- | ------------------------------------------------------------ |
  > | INDEX     | 时间片段记录编号，每一个编号代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作，非常重要。 |
  > | TIMESTAMP | 方法执行的本机时间，记录了这个时间片段所发生的本机时间       |
  > | COST(ms)  | 方法执行的耗时                                               |
  > | IS-RET    | 方法是否以正常返回的形式结束                                 |
  > | IS-EXP    | 方法是否以抛异常的形式结束                                   |
  > | OBJECT    | 执行对象的`hashCode()`，注意，曾经有人误认为是对象在JVM中的内存地址，但很遗憾他不是。但他能帮助你简单的标记当前执行方法的类实体 |
  > | CLASS     | 执行的类名                                                   |
  > | METHOD    | 执行的方法名                                                 |

* 条件表达式：条件表达式也是用 `OGNL` 来编写，核心的判断对象依然是 `Advice` 对象。除了 `tt` 命令之外，`watch`、`trace`、`stack` 命令也都支持条件表达式。

  - 解决方法重载

    `tt -t *Test print params.length==1`

  - 通过制定参数个数的形式解决不同的方法签名，如果参数个数一样，你还可以这样写

    `tt -t *Test print 'params[1] instanceof Integer'`

  - 解决指定参数

    `tt -t *Test print params[0].mobile=="13989838402"`

* 检索调用记录：就是根据`-l`和`-s 表达式`来进行一个检索：

  * 列出之前所有时间片段：`tt -l`；
  * 筛选出 `primeFactors` 方法的调用信息：`tt -s 'method.name=="primeFactors"'`

* 查看调用信息：通过 `-i` 参数后边跟着对应的 `INDEX` 编号查看到他的详细信息：

  ```bash
  [arthas@70700]$ tt -i 1005
   INDEX          1005
   GMT-CREATE     2022-02-19 16:24:52
   COST(ms)       0.135
   OBJECT         0x1055e4af
   CLASS          demo.MathGame
   METHOD         primeFactors
   IS-RETURN      true
   IS-EXCEPTION   false
   PARAMETERS[0]  @Integer[133275]
   RETURN-OBJ     @ArrayList[
                      @Integer[3],
                      @Integer[5],
                      @Integer[5],
                      @Integer[1777],
                  ]
  Affect(row-cnt:1) cost in 1 ms.
  ```

* 重做一次调用：`tt` 命令由于保存了当时调用的所有现场信息，所以我们可以自己主动对一个 `INDEX` 编号的时间片自主发起一次调用，从而解放你的沟通成本。此时你需要 `-p` 参数。通过 `--replay-times` 指定 调用次数，通过 `--replay-interval` 指定多次调用间隔(单位ms, 默认1000ms)

  ```bash
  [arthas@70700]$ tt -i 1005 -p --replay-times 2  # 重新调用2次1005的方法
   RE-INDEX       1005
   GMT-REPLAY     2022-02-19 16:39:25
   OBJECT         0x1055e4af
   CLASS          demo.MathGame
   METHOD         primeFactors
   PARAMETERS[0]  @Integer[133275]
   IS-RETURN      true
   IS-EXCEPTION   false
   COST(ms)       0.1307
   RETURN-OBJ     @ArrayList[
                      @Integer[3],
                      @Integer[5],
                      @Integer[5],
                      @Integer[1777],
                  ]
  Time fragment[1005] successfully replayed 1 times.
  
   RE-INDEX       1005
   GMT-REPLAY     2022-02-19 16:39:26
   OBJECT         0x1055e4af
   CLASS          demo.MathGame
   METHOD         primeFactors
   PARAMETERS[0]  @Integer[133275]
   IS-RETURN      true
   IS-EXCEPTION   false
   COST(ms)       0.11
   RETURN-OBJ     @ArrayList[
                      @Integer[3],
                      @Integer[5],
                      @Integer[5],
                      @Integer[1777],
                  ]
  Time fragment[1005] successfully replayed 2 times.
  ```

> tt这个命令真没在线上用过，好像也没使用场景，更多内容见：https://arthas.aliyun.com/doc/tt.html

### 通用操作

**耗时时间：**watch/stack/trace这个三个命令都支持`#cost`

**排除掉指定的类**：watch/trace/monitor/stack/tt 命令都支持 `--exclude-class-pattern` 参数

> 使用 `--exclude-class-pattern` 参数可以排除掉指定的类，比如：`watch javax.servlet.Filter * --exclude-class-pattern com.demo.TestFilter`

**使用-v参数打印更多信息：**watch/trace/monitor/stack/tt 命令都支持 `-v` 参数

> 当命令执行之后，没有输出结果。有两种可能：
>
> 1. 匹配到的函数没有被执行
> 2. 条件表达式结果是 false
>
> 但用户区分不出是哪种情况。使用 `-v`选项，则会打印`Condition express`的具体值和执行结果，方便确认。

### Options

> 用过的仅仅是：`json-format`

**Options全局开关：**

| 名称                   | 默认值    | 描述                                                         |
| ---------------------- | --------- | ------------------------------------------------------------ |
| unsafe                 | false     | 是否支持对系统级别的类进行增强，打开该开关可能导致把JVM搞挂，请慎重选择！ |
| dump                   | false     | 是否支持被增强了的类dump到外部文件中，如果打开开关，class文件会被dump到`/${application working dir}/arthas-class-dump/`目录下，具体位置详见控制台输出 |
| batch-re-transform     | true      | 是否支持批量对匹配到的类执行retransform操作                  |
| **json-format**        | **false** | **是否支持json化的输出**                                     |
| disable-sub-class      | false     | 是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关 |
| support-default-method | true      | 是否支持匹配到default method，默认会查找interface，匹配里面的default method。参考 [#1105](https://github.com/alibaba/arthas/issues/1105) |
| save-result            | false     | 是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到`~/logs/arthas-cache/result.log`中 |
| job-timeout            | 1d        | 异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒 |
| print-parent-fields    | true      | 是否打印在parent class里的filed                              |

**获取&修改option的值**：

* 直接输入options就可以查看所有支持的options

* options 上面的名称，如：`options json-format`

  ![options-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425466.PNG)

  > 可以看到默认是false

* **修改options的值**，这里修改`json-format`为true：`options json-format true`

  ```bash
  [arthas@70700]$ options json-format true
   NAME         BEFORE-VALUE  AFTER-VALUE
  ----------------------------------------
   json-format  false         true
  ```

  > **默认情况下`json-format`为false，如果希望`watch`/`tt`等命令结果以json格式输出，则可以设置`json-format`为true**
  >
  > * `json-format`为false：
  >
  >   ![options-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425868.PNG)
  >
  > * `json-format`为true：
  >
  >   ![options-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425439.PNG)
  >
  >   格式化一下：
  >
  >   ```json
  >   {
  >   	"@type": "java.lang.IllegalArgumentException",
  >   	"localizedMessage": "number is: -80370, need >= 2",
  >   	"message": "number is: -80370, need >= 2",
  >   	"stackTrace": [{
  >   		"className": "demo.MathGame",
  >   		"fileName": "MathGame.java",
  >   		"lineNumber": 46,
  >   		"methodName": "primeFactors",
  >   		"nativeMethod": false
  >   	}, {
  >   		"className": "demo.MathGame",
  >   		"fileName": "MathGame.java",
  >   		"lineNumber": 24,
  >   		"methodName": "run",
  >   		"nativeMethod": false
  >   	}, {
  >   		"className": "demo.MathGame",
  >   		"fileName": "MathGame.java",
  >   		"lineNumber": 16,
  >   		"methodName": "main",
  >   		"nativeMethod": false
  >   	}]
  >   }
  >   ```

## 其他命令

这部分就大致记录一下没怎么用过的一些命令，后续工作中如果有使用的就再好好学下然后放到上一趴里。

### 命令列表★

> 命令列表见：
>
> https://arthas.aliyun.com/doc/advanced-use.html
>
> https://arthas.aliyun.com/doc/commands.html

### thread*

**基础使用：**

* 通过thread命令来获取到当前运行的所有线程：

* `thread 1`会打印线程ID 1的栈，通常是main函数的线程。

  ```bash
  [arthas@67168]$ thread 1
  "main" Id=1 TIMED_WAITING
      at java.lang.Thread.sleep(Native Method)
      at java.lang.Thread.sleep(Thread.java:340)
      at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
      at demo.MathGame.main(MathGame.java:17)
  ```

**参数说明：**

| 参数名称      | 参数说明                                             | 示例                                             |
| ------------- | ---------------------------------------------------- | ------------------------------------------------ |
| id            | 线程id                                               | 查看线程ID为1的线程信息：`thread 1`              |
| [n:]          | 指定最忙的前N个线程并打印堆栈                        | 查看当前最忙的3个线程：`thread -n 3`             |
| [b]           | 找出当前阻塞其他线程的线程                           | 用于排查死锁等(存在阻塞时用)：`thread -b`        |
| [i `<value>`] | 指定cpu使用率统计的采样间隔，单位为毫秒，默认值为200 | 1000ms内最忙的3个线程：`thread -n 3 -i 1000`     |
| [--all]       | 显示所有匹配的线程                                   | `thread -all`                                    |
| [--state]     | 查看指定状态的线程                                   | 查看waitting状态的线程：`thread --state WATTING` |

> **cpu使用率是如何统计出来的？**
>
> 这里的cpu使用率与linux 命令`top -H -p <pid>` 的线程`%CPU`类似，一段采样间隔时间内，当前JVM里各个线程的增量cpu时间与采样间隔时间的比例。

> 更为详细内容：https://arthas.aliyun.com/doc/thread.html

### dashboard*

**基础使用：**输入[dashboard](https://arthas.aliyun.com/doc/dashboard.html)，按`回车/enter`，会展示当前进程的信息，按`ctrl+c`可以中断执行：

![dashboard](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261425734.PNG)

**数据说明：**

- ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应。
- NAME: 线程名
- GROUP: 线程组名
- PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
- STATE: 线程的状态
- CPU%: 线程的cpu使用率。比如采样间隔1000ms，某个线程的增量cpu时间为100ms，则cpu使用率=100/1000=10%
- DELTA_TIME: 上次采样之后线程运行增量CPU时间，数据格式为`秒`
- TIME: 线程运行总CPU时间，数据格式为`分:秒`
- INTERRUPTED: 线程当前的中断位状态
- DAEMON: 是否是daemon线程

**参数说明：**

| 参数名称 | 参数说明                                |
| -------- | --------------------------------------- |
| [i:]     | 刷新实时数据的时间间隔 (ms)，默认5000ms |
| [n:]     | 刷新实时数据的次数                      |

> 更详细的内容：https://arthas.aliyun.com/doc/dashboard.html

