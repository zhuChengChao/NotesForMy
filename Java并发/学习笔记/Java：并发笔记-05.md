# Java：并发笔记-05

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为5/9。

## 4. 共享模型之内存 

### 本章内容

上一章讲解的 Monitor 主要关注的是访问共享变量时，保证临界区代码的原子性 

这一章我们进一步深入学习共享变量在多线程间的【可见性】问题与多条指令执行时的【有序性】问题

### 4.1 Java 内存模型 

JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等。

JMM 体现在以下几个方面

- **原子性** - 保证指令不会受到线程上下文切换的影响
- **可见性** - 保证指令不会受 cpu 缓存的影响
- **有序性** - 保证指令不会受 cpu 指令并行优化的影响 

### 4.2 可见性

#### 退不出的循环

先来看一个现象，main 线程对 run 变量的修改对于 t 线程不可见，导致了 t 线程无法停止：

```java
static boolean run = true;

public static void main(String[] args) {
    Thread t = new Thread(()->{
        while (run){ ; }
    }, "t1");
    t.start();

    Sleeper.sleep(1);
    run = false;   // 线程t不会如预想的停下来
    LoggerUtils.LOGGER.debug("change run...");
}
```

为什么呢？分析一下： 

1. 初始状态， t 线程刚开始从主内存读取了 run 的值到工作内存 

   ![可见性1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547384.PNG)

2. 因为 t 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中，减少对主存中 run 的访问，提高效率

   ![可见性2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547730.PNG)

3. 1 秒之后，main 线程修改了 run 的值，并同步至主存，而 t 是从自己工作内存中的高速缓存中读取这个变量的值，结果永远是旧值

   ![可见性3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547143.PNG)

#### 解决方法：volatile

volatile（易变关键字） 

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存 

#### 可见性 vs 原子性

前面例子体现的实际就是可见性，它保证的是在多个线程之间，**一个线程对 volatile 变量的修改对另一个线程可见， 但是不能保证原子性**，仅用在一个写线程，多个读线程的情况： 上例从字节码理解是这样的： 

```java
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
putstatic run // 线程 main 修改 run 为 false， 仅此一次
getstatic run // 线程 t 获取 run false
```

比较一下之前我们将线程安全时举的例子：两个线程一个 i++ 一个 i-- ，只能保证看到最新值，不能解决指令交错 

```java
// 假设i的初始值为0
getstatic i // 线程2-获取静态变量i的值 线程内i=0
getstatic i // 线程1-获取静态变量i的值 线程内i=0

iconst_1 	// 线程1-准备常量1
iadd     	// 线程1-自增 线程内i=1
putstatic i // 线程1-将修改后的值存入静态变量i 静态变量i=1
iconst_1 	// 线程2-准备常量1
isub 		// 线程2-自减 线程内i=-1
putstatic i // 线程2-将修改后的值存入静态变量i 静态变量i=-1
```

> 注意 synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。
>
> 但缺点是synchronized 是属于重量级操作，性能相对更低
>
> 如果在前面示例的死循环中加入 System.out.println() 会发现即使不加 volatile 修饰符，线程 t 也能正确看到对 run 变量的修改了，想一想为什么
>
> 原因如下：
>
> ```java
> public void println(boolean x) {
>    	synchronized (this) {
>        	print(x);
>        	newLine();
>    	}
> }
> ```

### 原理： CPU缓存结构原理

> #### CPU 缓存结构
>
> ![CPU内存缓存图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547410.PNG)
>
> 速度比较
>
> | 从 cpu 到 ... | 大约需要的时钟周期 |
> | ------------- | ------------------ |
> | 寄存器        | 1 cycle            |
> | L1            | 3~4 cycle          |
> | L2            | 10~20 cycle        |
> | L3            | 40~45 cycle        |
> | 内存          | 120~240 cycle      |
>
> #### CPU 缓存读
>
> 读取数据流程如下 
>
> - 根据低位，计算在缓存中的索引
> - 判断是否有效
>   - 0 去内存读取新数据更新缓存行
>   - 1 再对比高位组标记是否一致 
>     - 一致，根据偏移量返回缓存数据
>     - 不一致，去内存读取新数据更新缓存行 
>
> #### 内存屏障
>
> Memory Barrier（Memory Fence） 
>
> - 可见性 
>   - 写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中 
>   - 而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据 
> - 有序性 
>   - 写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后 
>   - 读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前
>
> ![读写屏障](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547745.PNG)

### 模式：两阶段终止

见：[Java：并发笔记-01-模式：终止模式之两阶段终止](https://www.cnblogs.com/zhuchengchao/p/14490196.html)部分

### 模式：Balking

#### 定义 

Balking （犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回

#### 实现 

例如：

```java
public class MonitorService {

    // 用来表示是否已经有线程已经在执行启动了
    private volatile boolean starting;

    public void start(){
        LoggerUtils.LOGGER.debug("尝试启动监控线程...");
        synchronized (this){
            if(starting){
                return;
            }
            starting = true;
        }

        // 真正启动监控线程...
    }
}
```

当前端页面多次点击按钮调用 start 时 

输出：

```java
[http-nio-8080-exec-1] cn.itcast.monitor.service.MonitorService - 该监控线程已启动?(false)
[http-nio-8080-exec-1] cn.itcast.monitor.service.MonitorService - 监控线程已启动...
[http-nio-8080-exec-2] cn.itcast.monitor.service.MonitorService - 该监控线程已启动?(true)
[http-nio-8080-exec-3] cn.itcast.monitor.service.MonitorService - 该监控线程已启动?(true)
[http-nio-8080-exec-4] cn.itcast.monitor.service.MonitorService - 该监控线程已启动?(true)
```

它还经常用来实现线程安全的单例：

```java
public final class Singleton{
    // 构造函数私有化
    private Singleton() {
    }
    
    private static Singleton INSTANCE = null;
    
    // 静态方法上加synchronized，类锁
    public static synchronized Singleton getInstance(){
        if(INSTANCE != null){
            return INSTANCE;
        }
        // 这里才创建对象
        INSTANCE = new Singleton();
        return INSTANCE;
    }
}
```

对比一下**保护性暂停模式**：保护性暂停模式用在一个线程等待另一个线程的执行结果，当条件不满足时线程等待。 

### 4.3 有序性

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序，思考下面一段代码

```java
static int i;
static int j;
// 在某个线程内执行如下赋值操作
i = ...;
j = ...;
```

可以看到，至于是先执行 i 还是 先执行 j ，对最终的结果不会产生影响。所以，上面代码真正执行时，既可以是：

```java
i = ...;
j = ...;
```

也可以是：

```
j = ...;
i = ...;
```

这种特性称之为『指令重排』，多线程下『指令重排』会影响正确性。为什么要有重排指令这项优化呢？从 CPU执行指令的原理来理解一下吧 

#### 原理：指令级并行

> #### 名词
>
> * **Clock Cycle Time**
>
> 主频的概念大家接触的比较多，而 CPU 的 Clock Cycle Time（时钟周期时间），等于主频的倒数，意思是 CPU 能够识别的最小时间单位，比如说 4G 主频的 CPU 的 Clock Cycle Time 就是 0.25 ns，作为对比，我们墙上挂钟的 Cycle Time 是 1s 
>
> 例如，运行一条加法指令一般需要一个时钟周期时间 
>
> * **CPI** 
>
> 有的指令需要更多的时钟周期时间，所以引出了 CPI （Cycles Per Instruction）指令平均时钟周期数
>
> * **IPC** 
>
> IPC（Instruction Per Clock Cycle） 即 CPI 的倒数，表示每个时钟周期能够运行的指令数 
>
> #### CPU 执行时间
>
> 程序的 CPU 执行时间，即我们前面提到的 user + system 时间，可以用下面的公式来表示 
>
> ```
> 程序 CPU 执行时间 = 指令数 * CPI * Clock Cycle Time
> ```
>
> #### 鱼罐头的故事 
>
> 加工一条鱼需要 50 分钟，只能一条鱼、一条鱼顺序加工... 
>
> ![鱼罐头故事1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547913.PNG)
>
> 可以将每个鱼罐头的加工流程细分为 5 个步骤： 
>
> - 去鳞清洗 10分钟
> - 蒸煮沥水 10分钟
> - 加注汤料 10分钟
> - 杀菌出锅 10分钟
> - 真空封罐 10分钟 
>
> ![鱼罐头故事2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547521.PNG)
>
> 即使只有一个工人，最理想的情况是：他能够在 10 分钟内同时做好这 5 件事，因为对第一条鱼的真空装罐，不会影响对第二条鱼的杀菌出锅... 
>
> #### 指令重排序优化 
>
> 事实上，现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。为什么这么做呢？可以想到指令还可以再划分成一个个更小的阶段，例如，每条指令都可以分为： **取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回** 这 5 个阶段
>
> ![指令重排序优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547323.PNG)
>
> > 术语参考：
> >
> > - instruction fetch (IF)：取指令
> > - instruction decode (ID)：指令译码
> > - execute (EX)：执行指令
> > - memory access (MEM)：内存访问
> > - register write back (WB) ：数据写回
>
> 在不改变程序结果的前提下，这些指令的各个阶段可以通过**重排序**和**组合**来实现**指令级并行**，这一技术在 80's 中叶到 90's 中叶占据了计算架构的重要地位。 
>
> > 提示：分阶段，分工是提升效率的关键！ 
>
> 指令重排的前提是，重排指令不能影响结果，例如：
>
> ```java
> // 可以重排的例子
> int a = 10; // 指令1
> int b = 20; // 指令2
> System.out.println( a + b );
> 
> // 不能重排的例子
> int a = 10;     // 指令1
> int b = a - 5;  // 指令2
> ```
>
> > 参考： Scoreboarding and the Tomasulo algorithm (which is similar to scoreboarding but makes use of register renaming) are two of the most common techniques for implementing out-of-order execution and instruction-level parallelism. 
>
> #### 支持流水线的处理器 
>
> 现代 CPU 支持**多级指令流水线**，例如支持同时执行 `取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回` 的处理器，就可以称之为**五级指令流水线**。这时 CPU 可以在一个时钟周期内，同时运行五条指令的不同阶段（相当于一条执行时间最长的复杂指令），IPC = 1，本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了指令地吞吐率。 
>
> ![指令重排序优化2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547134.PNG)
>
> #### SuperScalar 处理器 
>
> 大多数处理器包含多个执行单元，并不是所有计算功能都集中在一起，可以再细分为整数运算单元、浮点数运算单元等，这样可以把多条指令也可以做到并行获取、译码等，CPU 可以在一个时钟周期内，执行多于一条指令，IPC>1 
>
> ![指令重排序优化3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251547612.png)
>
> 

#### 诡异的结果

```java
int num = 0;
boolean ready = false;

// 线程1 执行此方法
public void actor1(I_Result r){
    if(ready){
        r.r1 = num + num;
    }else{
        r.r1 = 1;
    }
}

// 线程2 执行此方法
public void actor2(I_Result r){
	num = 2;
    ready = true;
}
```

I_Result 是一个对象，有一个属性 r1 用来保存结果，问，可能的结果有几种？

有同学这么分析

情况1：线程1 先执行，这时 ready = false，所以进入 else 分支结果为 1

情况2：线程2 先执行 num = 2，但没来得及执行 ready = true，线程1 执行，还是进入 else 分支，结果为1

情况3：线程2 执行到 ready = true，线程1 执行，这回进入 if 分支，结果为 4（因为 num 已经执行过了） 

但我告诉你，结果还有可能是 0 😁😁😁，信不信吧！ 

这种情况下是：线程2 执行 ready = true，切换到线程1，进入 if 分支，相加为 0，再切回线程2 执行 num = 2 相信很多人已经晕了 😵😵😵 

这种现象叫做**指令重排**，是 JIT 编译器在运行时的一些优化，这个现象需要通过大量测试才能复现： 

借助 java 并发压测工具 jcstress https://wiki.openjdk.java.net/display/CodeTools/jcstress 

```bash
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.openjdk.jcstress -DarchetypeArtifactId=jcstress-java-test-archetype -DarchetypeVersion=0.5 -DgroupId=cn.itcast -DartifactId=ordering -Dversion=1.0
```

创建 maven 项目，提供如下测试类：

```java
@JCStressTest
// Outline the outcomes here. The default outcome is provided, you need to remove it:
@Outcome(id = "0, 0", expect = Expect.ACCEPTABLE, desc = "Default outcome.")
@State
public class ConcurrencyTest {

    int num = 0;
    boolean ready = false;

    @Actor
    public void actor1(II_Result r) {
        // Put the code for first thread here
        if(ready){
            r.r1 = num + num;
        }else{
            r.r1 = 1;
        }
    }

    @Actor
    public void actor2(II_Result r) {
        // Put the code for second thread here
        num = 2;
        ready = true;
    }
}
```

**执行：**

1. 通过 `maven->LifeCycle->package` 对该 module 进行打包；
2. 终端命令进入target 目录，执行如下命令：`java -jar jcstress.jar `

会输出我们感兴趣的结果，摘录其中一次结果： 

```
  Observed state   Occurrences   Expectation  Interpretation

            0, 0           274    ACCEPTABLE  Default outcome.

            1, 0     8,935,163     FORBIDDEN  No default case provided, assume FORBIDDEN

            4, 0    15,094,833     FORBIDDEN  No default case provided, assume FORBIDDEN
```

可以看到，出现结果为 0 的情况有 274 次，虽然次数相对很少，但毕竟是出现了。 

#### 解决方法 

volatile 修饰的变量，可以禁用指令重排

```java
volatile boolean ready = false;
```

结果为： 

```java
*** INTERESTING tests
  Some interesting behaviors observed. This is for the plain curiosity.

  0 matching test results.
```

#### happens-before

happens-before 规定了对共享变量的写操作对其它线程的读操作可见，它是**可见性与有序性的一套规则总结**，抛开以下 happens-before 规则，JMM 并不能保证一个线程对共享变量的写，对于其它线程对该共享变量的读可见 

- 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见 

  ```java
  static int x;
  static Object m = new Object();
  
  new Thread(()->{
      synchronized(m) {
          x = 10;
      }
  },"t1").start();
  
  new Thread(()->{
      synchronized(m) {
          System.out.println(x);
      }
  },"t2").start();
  ```

- 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见 

  ```java
  volatile static int x;
  new Thread(()->{
      x = 10;
  },"t1").start();
  
  new Thread(()->{
      System.out.println(x);
  },"t2").start();
  ```

- 线程 start 前对变量的写，对该线程开始后对该变量的读可见 

  ```java
  static int x;
  x = 10;
  
  new Thread(()->{
      System.out.println(x);
  },"t2").start();
  ```

- 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束） 

  ```java
  static int x;
  Thread t1 = new Thread(()->{
      x = 10;
  },"t1");
  
  t1.start();
  t1.join();
  System.out.println(x);
  ```

- 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过t2.interrupted 或 t2.isInterrupted） 

  ```java
  static int x;
  
  public static void main(String[] args) {
      Thread t2 = new Thread(()->{
          while(true) {
              if(Thread.currentThread().isInterrupted()) {  // 其他线程得知 t2 被打断后对变量的读可
                  System.out.println(x);  // 10
                  break;
              }
          }
      },"t2");
      t2.start();
      
      new Thread(()->{
          sleep(1);
          x = 10;  // 线程 t1 打断 t2（interrupt）前对变量的写
          t2.interrupt();
      },"t1").start();
      
      while(!t2.isInterrupted()) {
          Thread.yield();
      }
      // 其他线程得知 t2 被打断后对变量的读可
      System.out.println(x);  // 10
  }
  ```

- 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见 

- 具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z ，配合 volatile 的防指令重排，有下面的例子

  ```java
  volatile static int x;
  static int y;
  
  new Thread(()->{
      y = 10;
      x = 20;
  },"t1").start();
  
  new Thread(()->{
      // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
      System.out.println(x);
  },"t2").start();
  ```

### 4.4 习题

#### balking 模式习题 

希望 `doInit()` 方法仅被调用一次，下面的实现是否有问题，为什么？ 

```java
public class TestVolatile {
    volatile boolean initialized = false;
    void init() {
        if (initialized) {
            return;
        }
        doInit();
        initialized = true;
    }
    private void doInit() {
    }
}
```

有问题，volatile 只涉及可见性，而不能保证原子性，故可能多次调用doInit() 方法。

#### 线程安全单例习题

单例模式有很多实现方法，饿汉、懒汉、静态内部类、枚举类，试分析每种实现下获取单例对象（即调用 getInstance）时的线程安全，并思考注释中的问题

饿汉式：类加载就会导致该单实例对象被创建

懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会创建

* **实现1： 饿汉单例** 

```java
// 问题1：为什么加 final-->防止被子类覆盖而对其单例的破坏
// 问题2：如果实现了序列化接口, 还要做什么来防止反序列化破坏单例-->加一个方法readResolve
public final class Singleton implements Serializable {
    // 问题3：为什么设置为私有? 是否能防止反射创建新的实例?-->防止通过该方法创建对象；不能防止通过反射创建新的实例，反射可以获取所有的构造函数，并破坏其私有性，即暴力反射
    private Singleton() {}
    // 问题4：这样初始化是否能保证单例对象创建时的线程安全?-->能，静态变量在类加载时期被初始化，JVM保证其线程安全性
    private static final Singleton INSTANCE = new Singleton();
    // 问题5：为什么提供静态方法而不是直接将 INSTANCE 设置为 public, 说出你知道的理由-->用方法可以有更好的封装性；用方法还可支持泛型
    public static Singleton getInstance() {
        return INSTANCE;
    }
    public Object readResolve() {
        return INSTANCE;
    }
}
```

* **实现2：枚举单例** 

```java
// 问题1：枚举单例是如何限制实例个数的-->枚举在定义时有实例，就只能有几个实例，相当于枚举类内部的静态成员变量
// 问题2：枚举单例在创建时是否有并发问题-->没有，实例为静态成员变量，在类加载时期完成，JVM保证其线程安全
// 问题3：枚举单例能否被反射破坏单例-->不能
// 问题4：枚举单例能否被反序列化破坏单例-->不能
// 问题5：枚举单例属于懒汉式还是饿汉式-->饿汉式，在类加载时候就被创建了
// 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做-->给枚举类加个构造方法
enum Singleton {
    INSTANCE;
}
```

* **实现3： 懒汉式单例**

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    // 分析这里的线程安全, 并说明有什么缺点
    // 安全是安全的，但是导致锁颗粒度大，每次获取都要加锁
    public static synchronized Singleton getInstance() {
        if( INSTANCE != null ){
            return INSTANCE;
        }
        INSTANCE = new Singleton();
        return INSTANCE;
    }
}
```

* **实现4：DCL (double-checked locking ) 懒汉单例** 

```java
public final class Singleton {
    private Singleton() { }
    // 问题1：解释为什么要加 volatile ?  --> 防止指令重排序，即发生先赋值再构造的情况出现
    private static volatile Singleton INSTANCE = null;
    // 问题2：对比实现3, 说出这样做的意义 --> 不需要每次都加锁，性能优越性
    public static Singleton getInstance() {
        if (INSTANCE != null) {
            return INSTANCE;
        }
        synchronized (Singleton.class) {
            // 问题3：为什么还要在这里加为空判断, 之前不是判断过了吗-->防止首次创建时多线程并发问题
            if (INSTANCE != null) {  // t2
                return INSTANCE;
            }
            INSTANCE = new Singleton();
            return INSTANCE;
        }
    }
}
```

* **实现5：静态内部类懒汉单例**

```java
public final class Singleton {
    private Singleton() { }
    // 问题1：属于懒汉式还是饿汉式-->静态内部类：懒汉式，类加载即为懒惰的，类在第一次用到时才会触发类加载操作，只有调用了getInstance时，才会加载LazyHolder类
    private static class LazyHolder {
        static final Singleton INSTANCE = new Singleton();
    }
    // 问题2：在创建时是否有并发问题-->没有并发问题，只有类加载的时候才会被触发
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

### 原理：volatile

volatile 的底层实现原理是内存屏障，Memory Barrier（Memory Fence） 

- 对 volatile 变量的写指令后会加入写屏障
- 对 volatile 变量的读指令前会加入读屏障 

#### 如何保证可见性 

写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中 

```java
public void actor2(I_Result r) {
    num = 2;
    ready = true; // ready 是 volatile 赋值带写屏障
    // 写屏障
}
```

而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据 

```java
public void actor1(I_Result r) {
    // 读屏障
    // ready 是 volatile 读取值带读屏障
    if(ready) {
        r.r1 = num + num;
    } else {
        r.r1 = 1;
    }
}
```

![读写屏障2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548276.PNG)

#### 如何保证有序性 

屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后 

```java
public void actor2(I_Result r) {
    num = 2;
    ready = true; // ready 是 volatile 赋值带写屏障
    // 写屏障
}
```

读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前 

```java
public void actor1(I_Result r) {
    // 读屏障
    // ready 是 volatile 读取值带读屏障
    if(ready) {
        r.r1 = num + num;
    } else {
        r.r1 = 1;
    }
}
```

![读写屏障3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548634.PNG)

还是那句话，**不能解决指令交错**：

- 写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去
- 而有序性的保证也只是保证了本线程内相关代码不被重排序 

![读写屏障4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548631.PNG)

#### double-checked locking 问题 

以著名的 double-checked locking 单例模式为例 

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    public static Singleton getInstance() {
        if(INSTANCE == null) { 
            // 首次访问会同步，而之后的使用没有 synchronized
            synchronized(Singleton.class) {  // t2-->到这里阻塞
                if (INSTANCE == null) {      // t1-->运行到这里
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

以上的实现特点是： 

- 懒惰实例化
- 首次使用 getInstance() 才使用 synchronized 加锁，后续使用时无需加锁
- 有隐含的，但很关键的一点：第一个 if 使用了 INSTANCE 变量，**是在同步块之外** 

但在多线程环境下，**上面的代码是有问题的**，getInstance 方法对应的字节码为： 

```java
0: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
3: ifnonnull 37			
6: ldc #3 				// class cn/itcast/n5/Singleton
8: dup
9: astore_0
10: monitorenter
11: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
14: ifnonnull 27
17: new #3 				// class cn/itcast/n5/Singleton， 
20: dup					// 复制一份对象的引用，即操作栈中有两个对象的引用
21: invokespecial #4 	// Method "<init>":()V 消耗一个引用，调用构造
24: putstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton; 第二个引用给静态变量INSTANCE赋值
27: aload_0
28: monitorexit
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
```

其中

- 17 表示创建对象，将对象引用入栈    // new Singleton
- 20 表示复制一份对象引用  // 引用地址
- 21 表示利用一个对象引用，调用构造方法
- 24 表示利用一个对象引用，赋值给 static INSTANCE

也许 jvm 会优化为：先执行 24，再执行 21，即先赋值再调用构造。如果两个线程 t1，t2 按如下时间序列执行： 

![double-check-lock](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548946.PNG)

关键在于 0: getstatic 这行代码在 monitor 控制之外，它就像之前举例中不守规则的人，可以越过 monitor 读取 INSTANCE 变量的值 

这时 t1 还未完全将构造方法执行完毕，如果在构造方法中要执行很多初始化操作，那么 t2 拿到的是将是一个**未初始化完毕的单例** 

对 INSTANCE 使用 volatile 修饰即可，可以禁用指令重排，但要注意在 JDK 5 以上的版本的 volatile 才会真正有效 

> synchronized 不能阻止重排序；若共享变量完全被 synchronized 保护，则不存在有序、原子、可见问题

#### double-checked locking 解决 

```java
public final class Singleton {
    private Singleton() { }
    // 加上volatile，阻止指令重排序
    private static volatile Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized代码块
        if (INSTANCE == null) {
            synchronized (Singleton.class) { // t2
                // 也许有其它线程已经创建实例，所以再判断一次
                if (INSTANCE == null) { // t1
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

字节码上看不出来 volatile 指令的效果

```java
// ---------------------> 加入对 INSTANCE 变量的读屏障
0: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
3: ifnonnull 37
6: ldc #3 				// class cn/itcast/n5/Singleton
8: dup
9: astore_0
10: monitorenter // -----------------------> 保证原子性、可见性
11: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
14: ifnonnull 27
17: new #3 				// class cn/itcast/n5/Singleton
20: dup
21: invokespecial #4 	// Method "<init>":()V
24: putstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
// ---------------------> 加入对 INSTANCE 变量的写屏障
27: aload_0
28: monitorexit // ------------------------> 保证原子性、可见性
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 		// Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
```

如上面的注释内容所示，读写 volatile 变量时会加入内存屏障（Memory Barrier（Memory Fence）），保证下面两点： 

- 可见性
  - 写屏障（sfence）保证在该屏障之前的 t1 对共享变量的改动，都同步到主存当中
  - 而读屏障（lfence）保证在该屏障之后 t2 对共享变量的读取，加载的是主存中最新数据
- 有序性
  - 写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后
  - 读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前
- 更底层是读写变量时使用 lock 指令来多核 CPU 之间的可见性与有序性 

![double-check-lock2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251548181.PNG)

### 本章小结

本章重点讲解了 JMM 中的

- 可见性 - 由 JVM 缓存优化引起
- 有序性 - 由 JVM 指令重排序优化引起
- happens-before 规则
- 原理方面
  - CPU 指令并行
  - volatile
- 模式方面
  - 两阶段终止模式的 volatile 改进
  - 同步模式之 balking



