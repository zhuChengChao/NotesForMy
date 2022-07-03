# Java：并发笔记-01

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为1/9。

## 1. 进程与线程

### 本章内容

- 进程和线程的概念
- 并行和并发的概念
- 线程基本应用

### 1.1 进程与线程

**进程**

- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 IO 的；
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程；
- 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器等），也有的程序只能启动一个实例进程（例如网易云音乐、360 安全卫士等）

**线程**

- 一个进程之内可以分为一到多个线程。
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行
- 在 Java 中，**线程作为最小调度单位，进程作为资源分配的最小单位**。 
- 在 windows 中进程是不活动的，只是作为线程的容器

**二者对比**

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集
- 进程拥有共享的资源，如内存空间等，供其内部的线程共享
- 进程间通信较为复杂
  - 同一台计算机的进程通信称为 IPC（Inter-process communication）
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP
- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低

### 1.2 并行与并发

单核 cpu 下，线程实际还是 **串行执行** 的。操作系统中有一个组件叫做**任务调度器**，将 cpu 的时间片（windows下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于 cpu 在线程间（时间片很短）的切换非常快，人类感觉是同时运行的。总结为一句话就是： **微观串行，宏观并行** 

一般会将这种 线程轮流使用 CPU 的做法称为**并发——concurrent**

| CPU  | 时间片 1 | 时间片 2 | 时间片 3 | 时间片 4 |
| ---- | -------- | -------- | -------- | -------- |
| core | 线程 1   | 线程 2   | 线程 3   | 线程 4   |

![并发](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251544962.PNG)

多核CPU下，每个核（core）都可以调度运行线程，这时候线程可以是并行的。

| CPU    | 时间片 1 | 时间片 2 | 时间片 3 | 时间片 4 |
| ------ | -------- | -------- | -------- | -------- |
| core 1 | 线程 1   | 线程 1   | 线程 3   | 线程 3   |
| core 2 | 线程 2   | 线程 4   | 线程 2   | 线程 4   |

引用 Rob Pike 的一段描述：

- **并发（concurrent）是同一时间应对（dealing with）多件事情的能力**
- **并行（parallel）是同一时间动手做（doing）多件事情的能力**

> 例子
>
> - 家庭主妇做饭、打扫卫生、给孩子喂奶，她一个人轮流交替做这多件事，这时就是并发
> - 家庭主妇雇了个保姆，她们一起这些事，这时既有并发，也有并行（这时会产生竞争，例如锅只有一口，一个人用锅时，另一个人就得等待）
> - 雇了3个保姆，一个专做饭、一个专打扫卫生、一个专喂奶，互不干扰，这时是并行

### 应用

#### 应用：同步与异步

> 同步和异步，不需要等待结果，通过普通线程实现

没有用线程时，方法的调用是**同步**的： 

```java
// 同步调用
public class Sync {
    public static void main(String[] args) {
        FileReader.read(Constans.MP3_FULL_PATH);
        LoggerUtils.LOGGER.debug("do other things ...");
    }
}
// 输出："do other things ..."在最后输出
```

使用了线程后，方法的调用时**异步**的： 

```java
// 异步调用
public class Async {

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                FileReader.read(Constans.MP3_FULL_PATH);
            }
        }).start();
        LoggerUtils.LOGGER.debug("do other things ...");
    }
}
// 输出："do other things ..."不会在最后输出
```

> 上述程序中，采用了某些包，这里列出一下
>
> ```java
> package cn.util;
> 
> import java.io.File;
> import java.io.FileInputStream;
> import java.io.IOException;
> 
> public class FileReader {
> 
>     public static void read(String filename) {
>         int idx = filename.lastIndexOf(File.separator);
>         String shortName = filename.substring(idx + 1);
>         try (FileInputStream in = new FileInputStream(filename)) {
>             long start = System.currentTimeMillis();
>             LoggerUtils.LOGGER.debug("read [{}] start ...", shortName);
>             byte[] buf = new byte[1024];
>             int n = -1;
>             do {
>                 n = in.read(buf);
>             } while (n != -1);
>             long end = System.currentTimeMillis();
>             LoggerUtils.LOGGER.debug("read [{}] end ... cost: {} ms", shortName, end - start);
>         } catch (IOException e) {
>             e.printStackTrace();
>         }
>     }
> }
> 
> package cn.util;
> public class LoggerUtils {
>        public static final Logger LOGGER = LoggerFactory.getLogger(LoggerUtils.class);
> }
> 
> package cn.xyc;
> public class Constans {
>        public static final String MP3_FULL_PATH = "C:\\Users\\ZhuCC\\Music\\一路向北 - 周杰伦.mp3";
> }
> ```

以调用方角度来讲，如果

- 需要等待结果返回，才能继续运行就是同步
- 不需要等待结果返回，就能继续运行就是异步

**1）设计**

多线程可以让方法执行变为异步的（即不要巴巴干等着）比如说读取磁盘文件时，假设读取操作花费了 5 秒钟，如果没有线程调度机制，这 5 秒 cpu 什么都做不了，其它代码都得暂停...

**2）结论**

- 比如在项目中，视频文件需要转换格式等操作比较费时，这时开一个新线程处理视频转换，避免阻塞主线程
- tomcat 的异步 servlet 也是类似的目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作线程
- ui 程序中，开线程进行其他操作，避免阻塞 ui 线程

#### 应用：提高效率

充分利用多核 cpu 的优势，提高运行效率。想象下面的场景，执行 3 个计算，最后将计算结果汇总。

```
计算1 花费10ms
计算2 花费11ms
计算3 花费9ms
汇总需要 1 ms
```

- 如果是串行执行，那么总共花费的时间是 10 + 11 + 9 + 1 = 31ms
- 但如果是四核 cpu，各个核心分别使用线程 1 执行计算 1，线程 2 执行计算 2，线程 3 执行计算 3，那么 3 个线程是并行的，花费时间只取决于最长的那个线程运行的时间，即 11ms 最后加上汇总时间只会花费 12ms

> 注意：需要在多核 cpu 才能提高效率，单核仍然时是轮流执行

**总结**

1. 单核 cpu 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，不至于一个线程总占用 cpu，别的线程没法干活
2. 多核 cpu 可以并行跑多个线程，但能否提高程序运行效率还是要分情况的
   - 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分
   - 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
3. IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO - BIO】，这时相当于线程虽然不用 cpu，但需要一直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO - NIO】和【异步 IO - AIO】优化。

## 2. Java 线程

### 本章内容

- 创建和运行线程
- 查看线程
- 线程 API
- 线程状态

### 2.1 创建和运行线程

**方法一：直接使用 Thread**

```java
// 创建线程对象
Thread thread = new Thread(){
    @Override
    public void run(){
        // 要执行的任务
    }
};
// 启动线程
thread.start();
```

例如：

```java
// 构造方法的参数是给线程指定名字，推荐
Thread thread = new Thread("t1"){
    @Override
    public void run(){
        //  要执行的任务
        System.out.println("threading...");
    }
};
thread.start();

// 输出：
11:10:06.750 cn.util.LoggerUtils [t1] - hello thread
```

**方法二：使用 Runnable 配合 Thread**

把【线程】和【任务】（要执行的代码）分开

- Thread 代表线程
- Runnable 可运行的任务（线程要执行的代码）

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // 要执行的任务
    }
};
// 创建线程对象
Thread thread = new Thread(runnable);
// 启动线程
thread.start();
```

例如：

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // 要执行的任务
        LoggerUtils.LOGGER.debug("runnable+thread");
    }
};
// 创建线程对象:参数1 是任务对象; 参数2 是线程名字，推荐
Thread thread = new Thread(runnable, "t2");
// 启动线程
thread.start();

// 输出：
// 11:13:12.729 cn.util.LoggerUtils [t2] - runnable+thread
```

Java 8 以后可以使用 lambda 精简代码 

```java
Runnable runnable = () -> {
    // 要执行的任务
    LoggerUtils.LOGGER.debug("runnable+thread");
};
```

**Thread 与 Runnable 的关系：**

分析 Thread 的源码，理清它与 Runnable 的关系...

```java
// 1 构造函数
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
// 2 init方法中继续传递Runnable
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize) {
    init(g, target, name, stackSize, null, true);
}
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    // ...
    this.target = target;
    // ...
}

// 最终的run方法，即调用了run方法
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

小结：

- 方法1 是把线程和任务合并在了一起，方法2 是把线程和任务分开了
- 用 Runnable 更容易与线程池等高级 API 配合
- 用 Runnable 让任务类脱离了 Thread 继承体系，更灵活

**方法三：FutureTask 配合 Thread**

FutureTask 能够接收 Callable 类型的参数，**用来处理有返回结果的情况**

```java
// 创建任务对象
FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        LoggerUtils.LOGGER.debug("FutureTask+Callable");
        Thread.sleep(2000);
        return 100;
    }
});

// 参数1 是任务对象; 参数2 是线程名字，推荐
new Thread(task, "t3").start();

// 主线程阻塞，同步等待 task 执行完毕的结果
Integer result = task.get();
LoggerUtils.LOGGER.debug("result:{}", result);

// 输出结果：
// 11:24:55.097 cn.util.LoggerUtils [t3] - FutureTask+Callable
// 11:24:57.102 cn.util.LoggerUtils [main] - result:100
```

### 2.2 观察多个线程同时运行

主要是理解

- 交替执行
- **谁先谁后，不由我们控制**

### 2.3 查看进程线程的方法

**windows**

- 任务管理器可以查看进程和线程数，也可以用来杀死进程
- `tasklist` 查看进程
- `taskkill` 杀死进程

**linux**

- `ps -ef` 查看所有进程
- `ps -fT -p <PID>` 查看某个进程（PID）的所有线程
- `kill` 杀死进程
- `top` 按大写 H 切换是否显示线程
- `top -H -p <PID>` 查看某个进程（PID）的所有线程

**Java**

- `jps` 命令查看所有 Java 进程

- `jstack <PID>` 查看某个 Java 进程（PID）的所有线程状态

- `jconsole` 来查看某个 Java 进程中线程的运行情况（图形界面）

  > `jconsole` 远程监控配置
  >
  > - 需要以如下方式运行你的 java 类
  >
  >   ```bash
  >   java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 -Dcom.sun.management.jmxremote.authenticate=是否认证 java类
  >   ```
  >
  > - 修改 `/etc/hosts` 文件将 127.0.0.1 映射至主机名
  >
  > - 如果要认证访问，还需要做如下步骤
  >
  >   - 复制 jmxremote.password 文件
  >   - 修改 jmxremote.password 和 jmxremote.access 文件的权限为 600 即文件所有者可读写
  >   - 连接时填入 controlRole（用户名），R&D（密码）

### 2.4 原理之线程运行

**栈与栈帧**

Java Virtual Machine Stacks （Java 虚拟机栈）

我们都知道 JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟机就会为其分配一块栈内存。

- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

> 1. **只有主线程的情况下的Debug：**
>
> ```java
> public class TestFrames {
> 
>        public static void main(String[] args) {
>            method1(10);
>        }
> 
>        private static void method1(int x){
>            int y = x + 1;
>            Object m = method2();
>            System.out.println(m);
>        }
> 
>        private static Object method2(){
>            Object n = new Object();
>            return n;
>        }
> }
> ```
>
> 通过Debug进行调试：
>
> ![栈帧](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251544706.PNG)
>
> 更详细的画图说明：
>
> ![栈帧2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251544300.PNG)
>
> 2. **多线程情况下的Debug**
>
> ```java
> public class TestFrames {
> 
>        public static void main(String[] args) {
> 
>            Thread thread = new Thread(){
>                @Override
>                public void run() {
>                    method1(20);
>                }
>            };
>            thread.setName("t1");
>            thread.start();
> 
>            method1(10);
>        }
> 
>        private static void method1(int x){
>            int y = x + 1;
>            Object m = method2();
>            System.out.println(m);
>        }
> 
>        private static Object method2(){
>            Object n = new Object();
>            return n;
>        }
> }
> ```
>
> 通过Debug进行调试：
>
> > 注：在Debug调试的断点模式需要选择为Thread，默认为ALL（在断点上右击选择）
>
> ![多线程Debug调试](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545129.PNG)
>
> 更详细的画图说明：
>
> ![栈帧3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545535.PNG)

**线程上下文切换（Thread Context Switch）**

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

- 线程的 cpu 时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行
- 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，程序计数器是线程私有的

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
- Context Switch 频繁发生会影响性能

### 2.5 常见方法

| 方法名           | 功能说明                                                     | 注意                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| start()          | 启动一个新线程，在新的线程运行 run 方法中的代码              | start 方法只是让线程进入就绪，里面代码不一定立刻运行（CPU的时间片还没分给它）。<br />每个线程对象的 start 方法只能调用一次，如果调用了多次会出现 `IllegalThreadStateException` |
| run()            | 新线程启动后会调用的方法                                     | 如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会调用 Runnable 中的 run 方法，否则默认不执行任何操作。<br />但可以创建 Thread 的子类对象， 来覆盖默认行为。 |
| join()           | 等待线程运行结束                                             | 在调用的线程中，等待调用join的线程结束后，继续执行后面的语句；<br />即A线程中调用B.join()，则A线程阻塞到B线程执行完毕时继续执行。 |
| join(long n)     | 等待线程运行结束，最多等待 n 毫秒                            |                                                              |
| getId()          | 获取线程长整型的 id                                          | id 唯一                                                      |
| getName()        | 获取线程名                                                   |                                                              |
| setName(String)  | 修改线程名                                                   |                                                              |
| getPriority()    | 获取线程优先级                                               |                                                              |
| setPriority(int) | 修改线程优先级                                               | java中规定线程优先级是1~10 的整数，较大的优先级能提高该线程被 CPU 调度的机率。 |
| getState()       | 获取线程状态                                                 | Java 中线程状态是用 6 个 enum 表示，分别为: <br />NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED |
| isInterrupted()  | 判断是否被打断                                               | 不会清除打断标记                                             |
| isAlive()        | 线程是否存活（还没有运行完毕）                               |                                                              |
| interrupt()      | 打断线程                                                     | 如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，**并清除打断标记**；<br />如果打断的正在运行的线程，则会**设置打断标记**；<br />park 的线程被打断，也**会设置打断标记** |
| interrupted()    | 判断当前线程是否被打断                                       | 调用后会清除打断标记<br />static方法                         |
| currentThread()  | 获取当前正在执行的线程                                       | static方法                                                   |
| sleep(long n)    | 让当前执行的线程休眠n毫秒，休眠时让出 cpu 的时间片给其它线程 | static方法                                                   |
| yield()          | 提示线程调度器让出当前线程对CPU的使用                        | 主要是为了测试和调试<br />static方法                         |

### 2.6 start 与 run

**调用 run**

```java
public static void main(String[] args) {
    Thread t1 = new Thread("t1") {
        @Override
        public void run() {
            LoggerUtils.LOGGER.debug(Thread.currentThread().getName());
            FileReader.read(Constans.MP3_FULL_PATH);
        }
    };
    t1.run();
    LoggerUtils.LOGGER.debug("do other things ...");
}

// 输出：
12:44:09.646 cn.util.LoggerUtils [main] - main
12:44:09.650 cn.util.LoggerUtils [main] - read [一路向北 - 周杰伦.mp3] start ...
12:44:09.689 cn.util.LoggerUtils [main] - read [一路向北 - 周杰伦.mp3] end ... cost: 38 ms
12:44:09.689 cn.util.LoggerUtils [main] - do other things ...
```

**程序仍在 main 线程运行**， `FileReader.read()` 方法调用还是同步的。

**调用 start** 

将上述代码的 `t1.run()` 改为 `t1.start()`;

输出：

```
12:45:32.460 cn.util.LoggerUtils [main] - do other things ...
12:45:32.460 cn.util.LoggerUtils [t1] - t1
12:45:32.464 cn.util.LoggerUtils [t1] - read [一路向北 - 周杰伦.mp3] start ...
12:45:32.503 cn.util.LoggerUtils [t1] - read [一路向北 - 周杰伦.mp3] end ... cost: 39 ms
```

程序在 t1 线程运行，`FileReader.read()` 方法调用是异步的 

**小结** 

- 直接调用 run 是在主线程中执行了 run，没有启动新的线程
- 使用 start 是启动新的线程，通过新的线程间接执行 run 中的代码 

### 2.7 sleep 与 yield 

**sleep**

1. 调用 sleep 会让当前线程从 *Running* 进入 *Timed Waiting* 状态（阻塞）

   ```java
   public static void main(String[] args) {
       Thread t1 = new Thread("t1") {
           @Override
           public void run() {
               try {
                   Thread.sleep(2000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       };
   
       LoggerUtils.LOGGER.debug("t1 state: {}", t1.getState());
       t1.start();
       LoggerUtils.LOGGER.debug("t1 state: {}", t1.getState());
   
       // 主线程休眠
       try {
           Thread.sleep(500);
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
       LoggerUtils.LOGGER.debug("t1 state: {}", t1.getState());
   }
   
   // 输出结果：
   // 13:30:10.290 cn.util.LoggerUtils [main] - t1 state: NEW
   // 13:30:10.294 cn.util.LoggerUtils [main] - t1 state: RUNNABLE
   // 13:30:10.795 cn.util.LoggerUtils [main] - t1 state: TIMED_WAITING
   ```

2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 `InterruptedException`

   ```java
   public static void main(String[] args) throws InterruptedException {
       Thread t1 = new Thread("t1") {
           @Override
           public void run() {
               LoggerUtils.LOGGER.debug("enter sleep...");
               try {
                   Thread.sleep(2000);
               } catch (InterruptedException e) {
                   LoggerUtils.LOGGER.debug("wake up...");
                   e.printStackTrace();
               }
           }
       };
       t1.start();
   
       Thread.sleep(1000);  // 让主线程休眠
       LoggerUtils.LOGGER.debug("interrupt...");
       t1.interrupt();  // 打断线程
   }
   
   // 输出结果：
   // 13:34:57.140 cn.util.LoggerUtils [t1] - enter sleep...
   // 13:34:57.911 cn.util.LoggerUtils [main] - interrupt...
   // 13:34:57.911 cn.util.LoggerUtils [t1] - wake up...
   // java.lang.InterruptedException: sleep interrupted
   ```

3. 睡眠结束后的线程未必会立刻得到执行

4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性

   ```java
   TimeUnit.SECONDS.sleep(1);
   TimeUnit.MILLISECONDS.sleep(1000);
   ```

**yield**

1. 调用 yield 会让当前线程从 *Running* 进入 *Runnable* 就绪状态，然后调度执行其它线程
2. 具体的实现依赖于操作系统的任务调度器

**线程优先级** 

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用 

```java
public static void main(String[] args) {
    Runnable task1 = () -> {
        int count = 0;
        for (;;) {
            System.out.println("---->1 " + count++);
        }
    };
    Runnable task2 = () -> {
        int count = 0;
        for (;;) {
            // 交出对CPU的控制
            // Thread.yield();
            System.out.println("              ---->2 " + count++);
        }
    };
    Thread t1 = new Thread(task1, "t1");
    Thread t2 = new Thread(task2, "t2");
    // 设定优先级
    t1.setPriority(Thread.MIN_PRIORITY);
    t2.setPriority(Thread.MAX_PRIORITY);
    t1.start();
    t2.start();
}
```

### 应用：限制对 CPU 的使用

**sleep 实现**

* 在没有利用 cpu 来计算时，不要让 while(true) 空转浪费 cpu，这时可以使用 yield 或 sleep 来让出 cpu 的使用权给其他程序

  ```java
  while(true) {
      try {
      	Thread.sleep(50);
      } catch (InterruptedException e) {
      	e.printStackTrace();
      }
  }
  ```

* 可以用 **wait** 或 **条件变量** 达到类似的效果

* 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行同步的场景

* **sleep 适用于无需锁同步的场景**

**wait实现**

```java
synchronized(锁对象) {
    while(条件不满足) {
        try {
            锁对象.wait();
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
    // do sth...
}
```

**条件变量实现**

```java
lock.lock();  // 上锁
try {
    while(条件不满足) {
        try {
            条件变量.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    // do sth...
} finally {
    lock.unlock();  // 释放锁
}
```

### 2.8 join 方法详解

**为什么需要 join**

下面的代码执行，打印 r 是什么？

```java
static int r = 0;

public static void main(String[] args) {
    test1();
}

private static void test1(){
    LoggerUtils.LOGGER.debug("开始");

    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            LoggerUtils.LOGGER.debug("开始");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LoggerUtils.LOGGER.debug("结束");
            r = 10;
        }
    });
    t1.start();
    LoggerUtils.LOGGER.debug("结果为:{}", r);
    LoggerUtils.LOGGER.debug("结束");
}
```

分析

- 因为主线程和线程 t1 是并行执行的，t1 线程需要 1 秒之后才能算出 r=10
- 而主线程一开始就要打印 r 的结果，所以只能打印出 r=0

解决方法：

- 用 sleep 行不行？为什么？可以是可以，但是 t1 线程运行时间存在不确定性；

- 用 join，加在 t1.start() 之后即可

  ```java
  t1.start();
  t1.join();
  ```

```java
static int r = 0;
LoggerUtils.LOGGER.debug("开始");

Thread t1 = new Thread(new Runnable() {
    @Override
    public void run() {
        LoggerUtils.LOGGER.debug("开始");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LoggerUtils.LOGGER.debug("结束");
        r = 10;
    }
});
t1.start();
t1.join();
LoggerUtils.LOGGER.debug("结果为:{}", r);
LoggerUtils.LOGGER.debug("结束");

// 输出：
// 21:56:48.998 cn.util.LoggerUtils [main] - 开始
// 21:56:49.018 cn.util.LoggerUtils [Thread-0] - 开始
// 21:56:50.062 cn.util.LoggerUtils [Thread-0] - 结束
// 21:56:50.063 cn.util.LoggerUtils [main] - 结果为:10
// 21:56:50.064 cn.util.LoggerUtils [main] - 结束
```

> 评价：
>
> - 需要外部共享变量 r，不符合面向对象封装的思想
> - 必须等待线程结束，不能配合线程池使用 

以调用方角度来讲，如果

- 需要等待结果返回，才能继续运行就是同步
- 不需要等待结果返回，就能继续运行就是异步

![join1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251550279.PNG)

**等待多个结果**

问：下面代码 cost 大约多少秒？

```java
public class Test2 {
    static int r1 = 0;
    static int r2 = 0;

    public static void main(String[] args) throws InterruptedException {
        test2();
    }

    private static void test2() throws InterruptedException {

        Thread t1 = new Thread(() -> {
            Sleeper.sleep(1);
            r1 = 10;
        });
        Thread t2 = new Thread(() -> {
            Sleeper.sleep(2);
            r2 = 20;
        });

        long start = System.currentTimeMillis();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        long end = System.currentTimeMillis();
        LoggerUtils.LOGGER.debug("r1:{} r2:{} cost:{}", r1, r2, end-start);
    }
}

```

分析如下

- 第一个 join：等待 t1 时, t2 并没有停止, 而在运行；
- 第二个 join：1s 后, 执行到此, t2 也运行了 1s, 因此也只需再等待 1s

如果颠倒两个 join 呢？答：最终都是输出

```
15:19:54.777 cn.util.LoggerUtils [main] - r1:10 r2:20 cost:2005
```

![join2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251550482.PNG)

**有时效的 join——join(long n)，最多等待n毫秒**

1. 等够时间

```java
static int r1 = 0;
static int r2 = 0;

public static void main(String[] args) throws InterruptedException {
    test3();
}

private static void test3() throws InterruptedException {

    Thread t1 = new Thread(() -> {
        Sleeper.sleep(1);
        r1 = 10;
    });

    long start = System.currentTimeMillis();
    t1.start();
    // 线程执行结束会导致 join 结束
    t1.join(1500);
    long end = System.currentTimeMillis();
    LoggerUtils.LOGGER.debug("r1:{} r2:{} cost:{}", r1, r2, end-start);
}

//结果：15:25:22.137 cn.util.LoggerUtils [main] - r1:10 r2:0 cost:1002
```

2. 没等够时间 

```java
static int r1 = 0;
static int r2 = 0;

public static void main(String[] args) throws InterruptedException {
    test3();
}

private static void test3() throws InterruptedException {

    Thread t1 = new Thread(() -> {
        Sleeper.sleep(2);
        r1 = 10;
    });

    long start = System.currentTimeMillis();
    t1.start();
    // 线程执行结束会导致 join 结束
    t1.join(1500);
    long end = System.currentTimeMillis();
    LoggerUtils.LOGGER.debug("r1:{} r2:{} cost:{}", r1, r2, end-start);
}

// 结果：15:26:04.903 cn.util.LoggerUtils [main] - r1:0 r2:0 cost:1501
```

### 2.9 interrupt 方法详解

* interrupt 可以打断 sleep，wait，join 的线程

* 上述的这几个方法都会让线程进入**阻塞状态**

* 打断 sleep 的线程, 会清空打断状态，以 sleep 为例：

```java
public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        LoggerUtils.LOGGER.debug("sleep...");
        Sleeper.sleep(5);
    }, "t1");

    t1.start();
    Sleeper.sleep(1);
    LoggerUtils.LOGGER.debug("interrupt");
    t1.interrupt();
    LoggerUtils.LOGGER.debug("打断标记:{}", t1.isInterrupted());
}

// 输出结果：
// 15:50:38.890 cn.util.LoggerUtils [t1] - sleep...
// 15:50:39.660 cn.util.LoggerUtils [main] - interrupt
// java.lang.InterruptedException: sleep interrupted
// 15:50:39.660 cn.util.LoggerUtils [main] - 打断标记:false
```

**打断正常运行的线程**

* 打断正常运行的线程, 不会清空打断状态，即打断状态为 true；

* 而打断sleep，wait，join这种阻塞线程，会通过异常提示，而打断标记被清除，即为false

```java
public static void main(String[] args) {
    Thread t2 = new Thread(()->{
        while (true){
            // 获取当前线程
            Thread current = Thread.currentThread();
            boolean interrupted = current.isInterrupted();
            if(interrupted){
                LoggerUtils.LOGGER.debug("打断状态:{}", interrupted);
                break;
            }
        }
    }, "t2");

    t2.start();
    Sleeper.sleep(1);
    LoggerUtils.LOGGER.debug("interrupt");
    t2.interrupt();
    LoggerUtils.LOGGER.debug("打断标记:{}", t2.isInterrupted());
}

// 输出：
// 15:55:33.884 cn.util.LoggerUtils [main] - interrupt
// 15:55:33.886 cn.util.LoggerUtils [t2] - 打断状态:true
// 15:55:33.886 cn.util.LoggerUtils [main] - 打断标记:true
```

**打断 park 线程**

* 打断 park 线程, 不会清空打断状态

```java
public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        LoggerUtils.LOGGER.debug("park...");
        LockSupport.park();  // 不打断则会一直停留在此
        LoggerUtils.LOGGER.debug("uppark...");
        LoggerUtils.LOGGER.debug("打断状态：{}", Thread.currentThread().isInterrupted());
    }, "t1");
    t1.start();

    Sleeper.sleep(4);
    t1.interrupt();
}

// 输出
// 18:35:26.987 cn.util.LoggerUtils [t1] - park...
// 18:35:30.730 cn.util.LoggerUtils [t1] - uppark...
// 18:35:30.730 cn.util.LoggerUtils [t1] - 打断状态：true
```

如果打断标记已经是 true, 则 park 会失效

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5; i++) {
            LoggerUtils.LOGGER.debug("park...");
            LockSupport.park();
            LoggerUtils.LOGGER.debug("打断状态：{}", Thread.currentThread().isInterrupted());
        }
    });
    t1.start();
    Sleeper.sleep(4);
    t1.interrupt();
}

// 输出：
// 18:37:02.132 cn.util.LoggerUtils [Thread-0] - park...
// 18:37:05.913 cn.util.LoggerUtils [Thread-0] - 打断状态：true
// 18:37:05.919 cn.util.LoggerUtils [Thread-0] - park...
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - 打断状态：true
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - park...
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - 打断状态：true
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - park...
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - 打断状态：true
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - park...
// 18:37:05.920 cn.util.LoggerUtils [Thread-0] - 打断状态：true
```

> 提示：
>
> 可以使用 `Thread.interrupted()` 清除打断状态，清楚打断标记后，park又能生效了

### 模式：终止模式之两阶段终止

两阶段终止：Two Phase Termination

在一个线程 T1 中如何“优雅”终止线程 T2？这里的【优雅】指的是给 T2 一个料理后事的机会。

#### 错误思路

- 使用线程对象的 `stop()` 方法停止线程
  - stop 方法会真正杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其它线程将永远无法获取锁
- 使用 `System.exit(int)` 方法停止线程
  - 目的仅是停止一个线程，但这种做法会让整个程序都停止

#### 两阶段终止模式

![终止模式](Java：并发笔记-01\终止模式.PNG)

**利用 isInterrupted**

interrupt 可以打断正在执行的线程，无论这个线程是在 sleep，wait，join，还是正常运行

```java
public class TwoPhaseTermination {

    private Thread thread;

    public void start(){
        thread = new Thread(()->{
            while (true){
                Thread current = Thread.currentThread();
                if(current.isInterrupted()){
                    LoggerUtils.LOGGER.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);  // 情况1
                    LoggerUtils.LOGGER.debug("执行监控");  // 情况2
                } catch (InterruptedException e) {
                    // 情况1的时候，再把打断标志置位
                    current.interrupt();
                }
            }
        }, "监控线程");
        thread.start();
    }

    public void stop(){
        thread.interrupt();
    }
}

// 调用：
TwoPhaseTermination tpt = new TwoPhaseTermination();
tpt.start();
Thread.sleep(3500);
LoggerUtils.LOGGER.debug("stop");
tpt.stop();

// 结果：
// 16:31:28.031 cn.util.LoggerUtils [监控线程] - 执行监控
// 16:31:29.033 cn.util.LoggerUtils [监控线程] - 执行监控
// 16:31:30.034 cn.util.LoggerUtils [监控线程] - 执行监控
// 16:31:30.818 cn.util.LoggerUtils [main] - stop
// 16:31:30.818 cn.util.LoggerUtils [监控线程] - 料理后事
```

**利用停止标记**

```java
// 停止标记用 volatile 是为了保证该变量在多个线程之间的可见性
// 我们的例子中，即主线程把它修改为 true 对 t1 线程可见
public class TwoPhaseTermination {

    private Thread thread;
    private volatile boolean stop = false;

    public void start(){
        thread = new Thread(()->{
            while (true){
                Thread current = Thread.currentThread();
                if(stop){
                    LoggerUtils.LOGGER.debug("料理后事");
                    break;
                }
                Sleeper.sleep(1);
                LoggerUtils.LOGGER.debug("将结果保存");
            }
        }, "监控线程");
        thread.start();
    }

    public void stop(){
        stop = true;
        thread.interrupt();
    }
}

// 调用：
public static void main(String[] args) {
    TwoPhaseTermination t = new TwoPhaseTermination();
    t.start();
    Sleeper.sleep(3.5);
    LoggerUtils.LOGGER.debug("stop");
    t.stop();
}

// 输出：
// 20:19:09.271 cn.util.LoggerUtils [监控线程] - 将结果保存
// 20:19:10.273 cn.util.LoggerUtils [监控线程] - 将结果保存
// 20:19:11.274 cn.util.LoggerUtils [监控线程] - 将结果保存
// 20:19:11.588 cn.util.LoggerUtils [main] - stop
// 20:19:11.592 cn.util.LoggerUtils [监控线程] - 将结果保存
// 20:19:11.592 cn.util.LoggerUtils [监控线程] - 料理后事
```

### 2.10 不推荐的方法

还有一些不推荐使用的方法，这些方法已过时，容易破坏同步代码块，造成线程死锁

| 方法名    | 功能说明             |
| --------- | -------------------- |
| stop()    | 停止线程运行         |
| suspend() | 挂起（暂停）线程运行 |
| resume()  | 恢复线程运行         |

### 2.11 主线程与守护线程

默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。

```java
LoggerUtils.LOGGER.debug("开始运行...");
Thread t1 = new Thread(()->{
    LoggerUtils.LOGGER.debug("开始运行");
    Sleeper.sleep(2);
    LoggerUtils.LOGGER.debug("运行结束...");
}, "deamon");
// 设置该线程为守护进程
t1.setDaemon(true);
t1.start();

Sleeper.sleep(1);
LoggerUtils.LOGGER.debug("主线程结束...");

// 结果：
// 19:49:15.083 cn.util.LoggerUtils [main] - 开始运行...
// 19:49:15.123 cn.util.LoggerUtils [deamon] - 开始运行
// 19:49:16.125 cn.util.LoggerUtils [main] - 主线程结束...
```

> 注意
>
> - 垃圾回收器线程就是一种守护线程
> - Tomcat 中的 Acceptor 和 Poller 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等待它们处理完当前请求

### 2.12 五种状态

这是从 **操作系统** 层面来描述的

![线程的5种状态](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545597.PNG)

- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由 CPU 调度执行
- 【运行状态】指获取了 CPU 时间片，正在运行中的状态
  - 而当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
  - 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入【阻塞状态】
  - 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑调度它们
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态

### 2.13 六种状态

**这是从 Java API 层面来描述的**

根据 Thread.State 枚举，分为六种状态

![线程的6种状态](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545639.PNG)

- NEW 线程刚被创建，但是还没有调用 start() 方法
- RUNNABLE 当调用了 start() 方法之后，注意，Java API 层面的 RUNNABLE 状态涵盖了【操作系统】层面的【可运行状态】、【运行状态】和【阻塞状态】（**由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为是可运行**）
- BLOCKED ， WAITING ， TIMED_WAITING 都是 Java API 层面对【阻塞状态】的细分，后面会在状态转换一节详述
- TERMINATED 当线程代码运行结束

**六种状态演示代码：**

```java
public class TestState {

    public static void main(String[] args) {

        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                // 不start该线程，new状态
                LoggerUtils.LOGGER.debug("running...");
            }
        };

        Thread t2 = new Thread("t2") {
            @Override
            public void run() {
                while(true) {
                    // 线程一直运行，为runnable状态
                }
            }
        };
        t2.start();

        Thread t3 = new Thread("t3") {
            @Override
            public void run() {
                // 运行完结束，为TERMINATED 状态
                LoggerUtils.LOGGER.debug("running...");
            }
        };
        t3.start();

        Thread t4 = new Thread("t4") {
            @Override
            public void run() {
                synchronized (TestState.class) {
                    try {
                        Thread.sleep(1000000); // timed_waiting
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t4.start();

        Thread t5 = new Thread("t5") {
            @Override
            public void run() {
                try {
                    t2.join(); // waiting  无限制等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        t5.start();

        Thread t6 = new Thread("t6") {
            @Override
            public void run() {
                synchronized (TestState.class) {
                    // blocked  因为对象锁被t4给拿着，t6拿不到锁
                    try {
                        Thread.sleep(1000000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t6.start();

        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        LoggerUtils.LOGGER.debug("t1 state {}", t1.getState());
        LoggerUtils.LOGGER.debug("t2 state {}", t2.getState());
        LoggerUtils.LOGGER.debug("t3 state {}", t3.getState());
        LoggerUtils.LOGGER.debug("t4 state {}", t4.getState());
        LoggerUtils.LOGGER.debug("t5 state {}", t5.getState());
        LoggerUtils.LOGGER.debug("t6 state {}", t6.getState());
    }
}

// 结果：
// 20:10:54.823 cn.util.LoggerUtils [t3] - running...
// 20:10:55.088 cn.util.LoggerUtils [main] - t1 state NEW
// 20:10:55.090 cn.util.LoggerUtils [main] - t2 state RUNNABLE
// 20:10:55.090 cn.util.LoggerUtils [main] - t3 state TERMINATED
// 20:10:55.090 cn.util.LoggerUtils [main] - t4 state TIMED_WAITING
// 20:10:55.090 cn.util.LoggerUtils [main] - t5 state WAITING
// 20:10:55.090 cn.util.LoggerUtils [main] - t6 state BLOCKED
```

### 应用：统筹（烧水泡茶）

阅读华罗庚《统筹方法》，给出烧水泡茶的多线程解决方案，提示

- 参考图二，用两个线程（两个人协作）模拟烧水泡茶过程
  - 文中办法乙、丙都相当于任务串行
  - 而图一相当于启动了 4 个线程，有点浪费
- 用 sleep(n) 模拟洗茶壶、洗水壶等耗费的时间

> 附：华罗庚《统筹方法》
>
> 统筹方法，是一种安排工作进程的数学方法。它的实用范围极广泛，在企业管理和基本建设中，以及关系复杂的科研项目的组织与管理中，都可以应用。
>
> 怎样应用呢？主要是把工序安排好。
>
> 比如，想泡壶茶喝。当时的情况是：开水没有；水壶要洗，茶壶、茶杯要洗；火已生了，茶叶也有了。怎么办？
>
> - 办法甲：洗好水壶，灌上凉水，放在火上；在等待水开的时间里，洗茶壶、洗茶杯、拿茶叶；等水开
>   了，泡茶喝。
> - 办法乙：先做好一些准备工作，洗水壶，洗茶壶茶杯，拿茶叶；一切就绪，灌水烧水；坐待水开了，泡茶喝。
> - 办法丙：洗净水壶，灌上凉水，放在火上，坐待水开；水开了之后，急急忙忙找茶叶，洗茶壶茶杯，泡茶喝。
>
> 哪一种办法省时间？我们能一眼看出，第一种办法好，后两种办法都窝了工。
>
> 这是小事，但这是引子，可以引出生产管理等方面有用的方法来。
>
> 水壶不洗，不能烧开水，因而洗水壶是烧开水的前提。没开水、没茶叶、不洗茶壶茶杯，就不能泡茶，因而这些又是泡茶的前提。它们的相互关系，可以用下边的箭头图来表示：
>
> ![烧茶1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545132.PNG)
>
> 从这个图上可以一眼看出，办法甲总共要16分钟（而办法乙、丙需要20分钟）。如果要缩短工时、提高工作效率，应当主要抓烧开水这个环节，而不是抓拿茶叶等环节。同时，洗茶壶茶杯、拿茶叶总共不过4分钟，大可利用“等水开”的时间来做。
>
> 是的，这好像是废话，卑之无甚高论。有如走路要用两条腿走，吃饭要一口一口吃，这些道理谁都懂得。但稍有变化，临事而迷的情况，常常是存在的。在近代工业的错综复杂的工艺过程中，往往就不是像泡茶喝这么简单了。任务多了，几百几千，甚至有好几万个任务。关系多了，错综复杂，千头万绪，往往出现“万事俱备，只欠东风”的情况。由于一两个零件没完成，耽误了一台复杂机器的出厂时间。或往往因为抓的不是关键，连夜三班，急急忙忙，完成这一环节之后，还得等待旁的环节才能装配。
>
> 洗茶壶，洗茶杯，拿茶叶，或先或后，关系不大，而且同是一个人的活儿，因而可以合并成为：
>
> ![烧茶2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545608.PNG)
>
> 看来这是“小题大做”，但在工作环节太多的时候，这样做就非常必要了。
>
> 这里讲的主要是时间方面的事，但在具体生产实践中，还有其他方面的许多事。这种方法虽然不一定能直接解决所有问题，但是，我们利用这种方法来考虑问题，也是不无裨益的。

#### 解法1：join

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        LoggerUtils.LOGGER.debug("洗水壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("烧开水");
        Sleeper.sleep(5);
    },"老王");

    Thread t2 = new Thread(() -> {
        LoggerUtils.LOGGER.debug("洗茶壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("洗茶杯");
        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("拿茶叶");
        Sleeper.sleep(1);
        try {
            t1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LoggerUtils.LOGGER.debug("泡茶");
    },"小王");

    t1.start();
    t2.start();
}

// 20:19:15.605 cn.util.LoggerUtils [老王] - 洗水壶
// 20:19:15.605 cn.util.LoggerUtils [小王] - 洗茶壶
// 20:19:16.608 cn.util.LoggerUtils [小王] - 洗茶杯
// 20:19:16.608 cn.util.LoggerUtils [老王] - 烧开水
// 20:19:18.609 cn.util.LoggerUtils [小王] - 拿茶叶
// 20:19:21.609 cn.util.LoggerUtils [小王] - 泡茶 
```

解法1 的缺陷： 

- 上面模拟的是小王等老王的水烧开了，小王泡茶，如果反过来要实现老王等小王的茶叶拿来了，老王泡茶呢？代码最好能适应两种情况
- 上面的两个线程其实是各执行各的，如果要模拟老王把水壶交给小王泡茶，或模拟小王把茶叶交给老王泡茶呢？

#### 解法2：wait/notify 

```java
static String kettle = "冷水";
static String tea = null;
static final Object lock = new Object();
static boolean maked = false;

public static void makeTea(){
    new Thread(()->{
        LoggerUtils.LOGGER.debug("洗水壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("烧开水");
        Sleeper.sleep(5);
        synchronized (lock){
            kettle = "开水";
            lock.notifyAll();
            while (tea == null){
                try {
                    lock.wait();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }

            if(!maked){
                LoggerUtils.LOGGER.debug("拿({})泡({})", kettle, tea);
                maked = true;
            }
        }
    }, "老王").start();

    new Thread(()->{
        LoggerUtils.LOGGER.debug("洗茶壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("洗茶杯");
        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("拿茶叶");
        Sleeper.sleep(1);
        synchronized (lock){
            tea = "花茶";
            lock.notifyAll();
            while (kettle.equals("冷水")){
                try {
                    lock.wait();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }

            if(!maked){
                LoggerUtils.LOGGER.debug("拿({})泡({})", kettle, tea);
                maked = true;
            }
        }
    }, "小王").start();
}

// 输出：
21:55:35.705 cn.util.LoggerUtils [小王] - 洗茶壶
21:55:35.706 cn.util.LoggerUtils [老王] - 洗水壶
21:55:36.710 cn.util.LoggerUtils [小王] - 洗茶杯
21:55:36.710 cn.util.LoggerUtils [老王] - 烧开水
21:55:38.711 cn.util.LoggerUtils [小王] - 拿茶叶
21:55:41.711 cn.util.LoggerUtils [老王] - 拿(开水)泡(花茶)
```

解法2 解决了解法1 的问题，不过老王和小王需要相互等待，不如他们只负责各自的任务，泡茶交给第三人来做 

#### 解法3：第三者协调 

```java
public static void makeTea(){
    
    static final Object lock = new Object();

    new Thread(()->{
        LoggerUtils.LOGGER.debug("洗水壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("烧开水");
        Sleeper.sleep(5);
        synchronized (lock){
            kettle = "开水";
            lock.notifyAll();
        }
    }, "老王").start();

    new Thread(()->{
        LoggerUtils.LOGGER.debug("洗茶壶");
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("洗茶杯");
        Sleeper.sleep(2);
        LoggerUtils.LOGGER.debug("拿茶叶");
        Sleeper.sleep(1);
        synchronized (lock){
            tea = "花茶";
            lock.notifyAll();
        }
    }, "小王").start();

    new Thread(() -> {
        synchronized (lock) {
            while (kettle.equals("冷水") || tea == null) {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            LoggerUtils.LOGGER.debug("拿({})泡({})", kettle, tea);
        }
    }, "王夫人").start();
}

// 输出：
20:13:18.202 c.S3 [小王] - 洗茶壶
20:13:18.202 c.S3 [老王] - 洗水壶
20:13:19.206 c.S3 [小王] - 洗茶杯
20:13:19.206 c.S3 [老王] - 烧开水
20:13:21.206 c.S3 [小王] - 拿茶叶
20:13:24.207 c.S3 [王夫人] - 拿(开水)泡(花茶)
```

### 本章小结

本章的重点在于掌握

- 线程创建
- 线程重要 api，如 start，run，sleep，join，interrupt 等
- 线程状态
- 应用方面
- 异步调用：主线程执行期间，其它线程异步执行耗时操作
  - 提高效率：并行计算，缩短运算时间
  - 同步等待：join
  - 统筹规划：合理使用线程，得到最优效果
- 原理方面
  - 线程运行流程：栈、栈帧、上下文切换、程序计数器
  - Thread 两种创建方式的源码
- 模式方面
  - 终止模式之两阶段终止