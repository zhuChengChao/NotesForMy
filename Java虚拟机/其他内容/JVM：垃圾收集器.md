# JVM：垃圾收集器

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 垃圾回收算法和垃圾收集器关系

> 垃圾回收算法是内存回收的方法论，垃圾收集器就是算法的落地实现

垃圾回收算法主要有以下几种

- 引用计数（几乎不用，无法解决循环引用的问题）
- 复制拷贝（用于新生代）
- 标记清除（用于老年代）
- 标记整理（用于老年代）

因为目前为止还没有完美的收集器出现，更没有万能的收集器，只是针对具体应用使用最合适的收集器，进行分代收集（哪个代用哪个收集器）

## 四种主要的垃圾收集器

- Serial：串行回收 `-XX:+UseSeriallGC`
- Parallel：并行回收 `-XX:+UseParallelGC`
- CMS：并发标记清除（Concurrent Mark Sweep），`-XX:+UseConcMarkSweepGC`
- G1：Grabage First
- ZGC：Java 11 出现

![垃圾收集器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631124.png)

### Serial

串行垃圾回收器，为单线程环境设计，只使用一个线程进行垃圾收集，会暂停所有的用户线程(STW)，只有当垃圾回收完成时，才会重新唤醒主线程继续执行。所以不适合服务器环境

![serial串行垃圾回收器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631496.png)

### Parallel

并行垃圾收集器，多个垃圾收集线程并行工作，此时用户线程也是阻塞的，适用于科学计算/大数据处理等弱交互场景，也就是说 Serial 和 Parallel 其实是类似的，不过是多了几个线程进行垃圾收集，但是主线程都会被暂停，但是相较于串行垃圾回收器，并行垃圾处理器处理时间更短

![parallel垃圾回收器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631418.png)

### CMS

并发标记清除(Concurrent Mark Sweep)，用户线程和垃圾收集线程同时执行（不一定是并行，可能是交替执行），不需要停顿用户线程，互联网公司都在使用，适用于响应时间有要求的场景。并发是可以有交互的，也就是说可以**一边进行收集，一边执行应用程序**。

![cms垃圾收集](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631113.png)

### G1

G1垃圾回收器将堆内存分割成不同区域，然后并发的进行垃圾回收

![G1垃圾收集器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631941.png)

## 垃圾收集器总结

> 注意：并行垃圾回收在单核CPU下可能会更慢

![垃圾收集器小结](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632383.png)

## 查看默认垃圾收集器

使用下面 JVM 命令，查看配置的初始参数

```bash
-XX:+PrintCommandLineFlags
```

然后运行一个程序后，能够看到它的一些初始配置信息

```bash
-XX:InitialHeapSize=267290176 -XX:MaxHeapSize=4276642816 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
```

移动到最后一句，就能看到 `-XX:+UseParallelGC` 说明使用的是并行垃圾回收

## 默认垃圾收集器有哪些

Java中一共有7大垃圾收集器

1. `UserSerialGC`：串行垃圾收集器

2. `UserParallelGC`：并行垃圾收集器

3. `UseConcMarkSweepGC`：（CMS）并发标记清除

4. `UseParNewGC`：年轻代的并行垃圾回收器

   > CMS相结合

5. `UseParallelOldGC`：老年代的并行垃圾回收器

6. `UseG1GC`：G1垃圾收集器

7. `UserSerialOldGC`：串行老年代垃圾收集器（已经被移除）

底层源码

![7种垃圾回收器代码](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632934.png)

## 各垃圾收集器的使用范围

![7种垃圾回收器图示](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632098.png)

**新生代使用的**：

- Serial Copying： UserSerialGC，串行垃圾回收器
- Parallel Scavenge：UserParallelGC，并行垃圾收集器
- ParNew：UserParNewGC，新生代并行垃圾收集器，**组合CMS使用**

**老年代使用的**：

- Serial Old / Serial MSC：UseSerialOldGC，老年代串行垃圾收集器
- Parallel Compacting / Parallel Old：UseParallelOldGC，老年代并行垃圾收集器
- CMS：UseConcMarkSwepp，并行标记清除垃圾收集器

**各区都能使用的：**

* G1：UseG1GC，G1垃圾收集器

垃圾收集器就来具体实现这些 GC 算法并实现内存回收，不同厂商，不同版本的虚拟机实现差别很大，HotSpot中包含的收集器如下图所示：

![垃圾收集器组合关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632712.png)

## 部分参数说明

- DefNew：Default New Generation
- Tenured：Old
- ParNew：Parallel New Generation
- PSYoungGen：Parallel Scavenge
- ParOldGen：Parallel Old Generation

## Java中的Server和Client模式

使用范围：一般使用 Server 模式，Client 模式基本不会使用

操作系统

- 32位的 Windows 操作系统，不论硬件如何都默认使用 Client 的 JVM 模式
- 32位的其它操作系统，2G 内存同时有 2 个 cpu 以上用 Server 模式，低于该配置还是 Client 模式
- 64位只有 Server 模式

```bash
C:\Users\ZhuCC>java -version
java version "1.8.0_241"
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)  # Server VM
```

## 新生代下的垃圾收集器

### 串行GC：Serial / Serial Copying

是一个单线程的收集器，在进行垃圾收集时候，必须暂停其他所有的工作线程直到它收集结束。

![串行垃圾收集器工作流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632499.png)

串行收集器是最古老，最稳定以及效率高的收集器，但是只使用一个线程去回收但其在垃圾收集过程中可能会产生较长的停顿 ( Stop-The-World 状态)。 虽然在收集垃圾过程中需要暂停所有其它的工作线程，但是它简单高效，对于限定单个 CPU 环境来说，没有线程交互的开销可以获得最高的单线程垃圾收集效率，因此 Serial 垃圾收集器依然是 Java 虚拟机运行在 Client 模式下默认的新生代垃圾收集器

对应 JVM 参数是：`-XX:+UseSerialGC`

开启后会使用：**Serial(年轻代用) + Serial Old(老年代用) 的收集器组合**

**即：新生代、老年代都会使用串行回收收集器，而新生代使用复制算法，老年代使用标记-整理算法**

**配置：**

```bash
-Xms1m -Xmx1m -XX:PrintGCDetails -XX:+PrintConmandLineFlags -XX:+UseParNewGC
```

**代码：**

```java
import java.util.Random;

public class Demo {

    public static void main(String[] args) {

        System.out.println("Hello GC");
        try {
            String str = "xyc";
            while (true){
                str += "_" + new Random().nextInt(123456789);
                str.intern();
            }
        }catch (Throwable e){
            e.printStackTrace();
        }
    }
}
```

**输出：**

```bash
-XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseSerialGC # 可见：使用了串行垃圾回收器
[GC (Allocation Failure) [DefNew: 507K->64K(576K), 0.0007798 secs] 507K->432K(1984K), 0.0008109 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
...
[Full GC (Allocation Failure) [Tenured: 1139K->1139K(1408K), 0.0012095 secs] 1139K->1139K(1984K), [Metaspace: 3246K->3246K(1056768K)], 0.0012271 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 576K, used 40K [0x00000000ffe00000, 0x00000000ffea0000, 0x00000000ffea0000)
  eden space 512K,   7% used [0x00000000ffe00000, 0x00000000ffe0a1e0, 0x00000000ffe80000)
  from space 64K,   0% used [0x00000000ffe90000, 0x00000000ffe90000, 0x00000000ffea0000)
  to   space 64K,   0% used [0x00000000ffe80000, 0x00000000ffe80000, 0x00000000ffe90000)
 tenured generation   total 1408K, used 1139K [0x00000000ffea0000, 0x0000000100000000, 0x0000000100000000)
   the space 1408K,  80% used [0x00000000ffea0000, 0x00000000fffbcc60, 0x00000000fffbce00, 0x0000000100000000)
 Metaspace       used 3277K, capacity 4568K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 392K, committed 512K, reserved 1048576K
```

### 并行GC：ParNew

并行收集器，使用多线程进行垃圾回收，在垃圾收集，会 Stop-the-World 暂停其他所有的工作线程直到它收集结束

![ParNew工作流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632933.png)

ParNew 收集器其实就是 Serial 收集器新生代的并行多线程版本，最常见的应用场景是配合老年代的 CMS GC 工作，其余的行为和 Serial 收集器完全一样，ParNew 垃圾收集器在垃圾收集过程中同样也要暂停所有其他的工作线程。它是很多 Java 虚拟机运行在 Server 模式下新生代的默认垃圾收集器。

常见对应 JVM 参数：`-XX:+UseParNewGC` 启动 ParNew 收集器，只影响新生代的收集，不影响老年代

**开启上述参数后，会使用：ParNew（年轻代用） + Serial Old（老年代用）的收集器组合，新生代使用复制算法，老年代采用标记-整理算法**

**配置：**

```bash
-Xms1m -Xmx1m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParNewGC
```

需要注意的是：这样配置会出现警告，即 ParNew 和 Serial Old 这样搭配，在Java8中已经不再被推荐

> ![垃圾收集器组合关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632518.png)
>
> 这里已经被画了红叉叉

**代码：**同上

**输出：**

```bash
-XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC  # 这里已经换成了UseParNewGC
[GC (Allocation Failure) [ParNew: 507K->64K(576K), 0.0003895 secs] 507K->515K(1984K), 0.0004153 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
...
[Full GC (Allocation Failure) [Tenured: 1139K->1139K(1408K), 0.0013169 secs] 1139K->1139K(1984K), [Metaspace: 3246K->3246K(1056768K)], 0.0013363 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 par new generation   total 576K, used 40K [0x00000000ffe00000, 0x00000000ffea0000, 0x00000000ffea0000)
  eden space 512K,   7% used [0x00000000ffe00000, 0x00000000ffe0a1e0, 0x00000000ffe80000)
  from space 64K,   0% used [0x00000000ffe80000, 0x00000000ffe80000, 0x00000000ffe90000)
  to   space 64K,   0% used [0x00000000ffe90000, 0x00000000ffe90000, 0x00000000ffea0000)
 tenured generation   total 1408K, used 1139K [0x00000000ffea0000, 0x0000000100000000, 0x0000000100000000)
   the space 1408K,  80% used [0x00000000ffea0000, 0x00000000fffbcc60, 0x00000000fffbce00, 0x0000000100000000)
 Metaspace       used 3277K, capacity 4568K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 392K, committed 512K, reserved 1048576K
 
Java HotSpot(TM) 64-Bit Server VM warning: Using the ParNew young collector with the Serial old collector is deprecated and will likely be removed in a future release  
# 可以看到在老年代中使用了Serial old垃圾收集器，这里产生了过去可能被移除的Warings
```

### 并行回收GC：Parallel Scavenge

因为 Serial 和 ParNew 都不推荐使用了，因此现在新生代默认使用的是 Parallel Scavenge，也就是新生代和老年代都是使用并行

![并行垃圾收集器流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632676.png)

Parallel Scavenge 收集器类似 ParNew 也是一个新生代垃圾收集器，使用复制算法，也是一个并行的多线程的垃圾收集器，俗称**吞吐量优先收集器**。**一句话：串行垃圾收集器在新生代和老年代的并行化**

**它关注的重点是：**

可控制的吞吐量（Thoughput = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)），也即比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99%。高吞吐量意味着高效利用CPU时间，它多用于在后台运算而不需要太多交互的任务。

**自适应调节策略**也是 Parallel Scavenge 收集器与 ParNew 收集器的一个重要区别。

自适应调节策略：虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间( `-XX:MaxGCPauseMills`) 或 最大的吞吐量。

常用 JVM 参数：`-XX:+UseParallelGC` 或 `-XX:+UseParallelOldGC`（可互相激活，即只要配置一个另一个也自动配置了）开启使用 Parallel Scanvenge收集器

**开启该参数后：新生代和老年代都会使用并行垃圾回收器，新生代使用复制算法，老年代使用标记-整理算法**

```bash
-Xms1m -Xmx1m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelGC
# 等价于
-Xms1m -Xmx1m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelOldGC
```

**代码：**同上

**输出**：

```bash
-XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC   # 这里已经换成了UseParallelGC
[GC (Allocation Failure) [PSYoungGen: 507K->488K(1024K)] 507K->496K(1536K), 0.0005216 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
...
[Full GC (Allocation Failure) [PSYoungGen: 346K->346K(1024K)] [ParOldGen: 420K->420K(512K)] 766K->766K(1536K), [Metaspace: 3246K->3246K(1056768K)], 0.0029494 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 1024K, used 383K [0x00000000ffe80000, 0x0000000100000000, 0x0000000100000000)
  eden space 512K, 74% used [0x00000000ffe80000,0x00000000ffedfc58,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 512K, used 420K [0x00000000ffe00000, 0x00000000ffe80000, 0x00000000ffe80000)
  object space 512K, 82% used [0x00000000ffe00000,0x00000000ffe69030,0x00000000ffe80000)
 Metaspace       used 3277K, capacity 4568K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 392K, committed 512K, reserved 1048576K
```

## 老年代下的垃圾收集器

### 串行GC ：Serial Old / MSC

Serial Old 是 Serial 垃圾收集器老年代版本，它同样是一个单线程的收集器，使用标记-整理算法，这个收集器也主要是为 Client 下 Java 虚拟机中默认的老年代垃圾收集器

在 Server 模式下，主要有两个用途

- 在 JDK1.5 之前版本中与新生代的 Parallel Scavenge 收集器搭配使用（Parallel Scavenge + Serial Old）
- 作为老年代版中使用 CMS 收集器的后备垃圾收集方案

**配置方法：**

```bash
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseSerialOldlGC
```

该垃圾收集器，目前已经不推荐使用了

### 并行GC：Parallel Old / Parallel MSC

Parallel Old 收集器是 Parallel Scavenge 的老年代版本，使用多线程的标记-整理算法，Parallel Old收集器在JDK1.6 才开始提供。

在 JDK1.6 之前，新生代使用 Parallel Scavenge 收集器只能搭配老年代的 Serial Old 收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量。

Parallel Old 正是为了在老年代同样提供吞吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，JDK1.8 后可以考虑新生代 Parallel Scavenge 和老年代 Parallel Old 垃圾收集器的搭配策略。

在 JDK1.6 以前(Parallel Scavenge + Serial Old)

在 JDK1.8 及后(Parallel Scavenge + Parallel Old)

**使用老年代并行收集器 JVM 配置**：

```bash
-Xms10m -Xmx10m -XX:PrintGCDetails -XX:+PrintConmandLineFlags -XX:+UseParallelOldlGC
```

> `-XX +UseParallelOldGC`：使用Parallel Old收集器，设置该参数后，新生代 Parallel Scavenge +老年代 Parallel Old

### CMS：并发标记清除 GC

CMS 收集器（Concurrent Mark Sweep：并发标记清除）是一种以**最短回收停顿时间为目标的收集器**

适合应用在互联网或者B/S系统的服务器上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短。

CMS 非常适合堆内存大，CPU 核数多的服务器端应用，也是 G1 出现之前大型应用的首选收集器。

![cms垃圾收集流程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632411.png)

Concurrent Mark Sweep：并发标记清除，并发收集低停顿，并发指的是与用户线程一起执行

开启该收集器的JVM参数： `-XX:+UseConcMarkSweepGC` 开启该参数后，会自动将 `-XX:+UseParNewGC`打开，开启该参数后，使用 **ParNew (年轻代用）+ CMS（老年代用） + Serial Old（老年代用）** 的收集器组合，**Serial Old 将作为 CMS 出错的后备收集器**

**JVM 参数配置**：

```bash
-Xms1m -Xmx1m -XX:+PrintGCDetails -XX:+PrintConmandLineFlags -XX:+UseConcMarkSweepGC
```

**代码：同上**

**输出：**

```bash
-XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:MaxNewSize=0 -XX:MaxTenuringThreshold=6 -XX:OldPLABSize=16 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC  # 使用了CMS+ParNewGC垃圾收集器
[GC (Allocation Failure) [ParNew (promotion failed): 1024K->1152K(1152K), 0.0014237 secs][CMS: 385K->548K(768K), 0.0017122 secs] 1024K->548K(1920K), [Metaspace: 2583K->2583K(1056768K)], 0.0031901 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 初始标记阶段
[GC (CMS Initial Mark) [1 CMS-initial-mark: 548K(768K)] 549K(1920K), 0.0000843 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]  
# 并发标记阶段
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean-start]  # 有一个预先清理
[CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 重新标记阶段
[GC (CMS Final Remark) [YG occupancy: 21 K (1152 K)][Rescan (parallel) , 0.0000664 secs][weak refs processing, 0.0000231 secs][class unloading, 0.0001237 secs][scrub symbol table, 0.0002622 secs][scrub string table, 0.0000738 secs][1 CMS-remark: 548K(768K)] 570K(1920K), 0.0005891 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 并发清除阶段
[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Hello GC
...
[GC (Allocation Failure) [ParNew (promotion failed): 772K->774K(1152K), 0.0002392 secs][CMS: 608K->606K(768K), 0.0012214 secs] 1379K->1374K(1920K), [Metaspace: 3246K->3246K(1056768K)], 0.0014832 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [CMS: 606K->606K(768K), 0.0011494 secs] 1374K->1374K(1920K), [Metaspace: 3246K->3246K(1056768K)], 0.0011668 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 par new generation   total 1152K, used 811K [0x00000000ffe00000, 0x00000000fff40000, 0x00000000fff40000)
  eden space 1024K,  79% used [0x00000000ffe00000, 0x00000000ffecac50, 0x00000000fff00000)
  from space 128K,   0% used [0x00000000fff00000, 0x00000000fff00000, 0x00000000fff20000)
  to   space 128K,   0% used [0x00000000fff20000, 0x00000000fff20000, 0x00000000fff40000)
 concurrent mark-sweep generation total 768K, used 606K [0x00000000fff40000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 3277K, capacity 4568K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 392K, committed 512K, reserved 1048576K
```

#### 四个步骤

- **初始标记（CMS initial mark）**
  - 只是标记 GC Roots 能直接关联的对象，速度很快，仍然需要暂停所有的工作线程（STW）
- **并发标记（CMS concurrent mark）和用户线程一起**
  - 进行 GC Roots 跟踪过程，和用户线程一起工作，不需要暂停工作线程。主要标记过程，标记全部对象
- **重新标记（CMS remark）**
  - 为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程，由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正
- **并发清除（CMS concurrent sweep）和用户线程一起**
  - 清除 GC Roots 不可达对象，和用户线程一起工作，不需要暂停工作线程。基于标记结果，直接清理对象

由于**耗时最长的并发标记和并发清除**过程中，垃圾收集线程可以和用户现在一起并发工作，所以**总体上来看CMS收集器的内存回收和用户线程是一起并发地执行**。

![CMS四个步骤](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632818.png)

**优点：并发收集低停顿**

**缺点：并发执行，对CPU资源压力大，采用的标记清除算法会导致大量内存碎片**

由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说，**CMS必须在老年代堆内存用尽之前完成垃圾回收**，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW方式进行一次GC，从而造成较大的停顿时间

**同时标记清除算法无法整理空间碎片**，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩，CMS也提供了参数 `-XX:CMSFullGCSBeForeCompaction`（默认0，即每次都进行内存整理）来指定多少次 CMS 收集之后，进行一次压缩的 Full GC

## 为什么新生代采用复制算法，老年代采用标整算法

### 新生代使用复制算法

因为新生代对象的**生存时间比较短**，80%的都要回收的对象，采用标记-清除算法则内存碎片化比较严重，采用复制算法可以灵活高效，且便与整理空间。

### 老年代采用标记整理

标记整理算法主要是为了解决标记清除算法存在内存碎片的问题，又解决了复制算法两个Survivor区的问题，因为老年代的空间比较大，不可能采用特别占用内存空间的复制算法。

## 垃圾收集器如何选择

### 组合的选择

单CPU或者小内存，单机程序

- `-XX:+UseSerialGC`

多CPU，需要最大的吞吐量，如后台计算型应用

- `-XX:+UseParallelGC`
- `-XX:+UseParallelOldGC`

> 上述两个参数互相激活

多CPU，追求低停顿时间，需要快速响应如互联网应用

- `-XX:+ParNewGC`
- `-XX:+UseConcMarkSweepGC`

### 总结

| 参数                      | 新生代垃圾收集器         | 新生代算法 | 老年代垃圾收集器                                             | 老年代算法                            |
| ------------------------- | ------------------------ | ---------- | ------------------------------------------------------------ | ------------------------------------- |
| `-XX:+UseSerialGC`        | SerialGC                 | 复制       | SerialOldGC                                                  | 标记整理                              |
| `-XX:+UseParNewGC`        | ParNew                   | 复制       | SerialOldGC                                                  | 标记整理                              |
| `-XX:+UseParallelGC`      | Parallel [Scavenge]      | 复制       | Parallel Old                                                 | 标记整理                              |
| `-XX:+UseConcMarkSweepGC` | ParNew                   | 复制       | CMS + Serial Old的收集器组合<br />Serial Old作为CMS出错的后备收集器 | CMS:标记清除<br />Serial Old:标记整理 |
| `-XX:+UseG1GC`            | G1整体上采用标记整理算法 | 局部复制   | G1整体上采用标记整理算法                                     | 局部复制                              |

## G1垃圾收集器

![G1垃圾收集器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632282.png)

### 开启 G1 垃圾收集器

```bash
-XX:+UseG1GC
```

### 以前收集器的特点

- 年轻代和老年代是各自独立且连续的内存块
- 年轻代收集使用 **单eden + S0 + S1** 进行复制算法
- 老年代收集必须扫描整个老年代区域
- 都是以尽可能少而快速地执行 GC 为设计原则

### G1 是什么

G1：Garbage-First 收集器，是一款面向服务端应用的垃圾收集器，应用在多处理器和大容量内存环境中，**在实现高吞吐量的同时，尽可能满足垃圾收集暂停时间的要求**。另外，它还具有以下特征：

- 像 CMS 收集器一样，能与应用程序并发执行
- 整理空闲空间更快
- 需要更多的时间来预测 GC 停顿时间
- 不希望牺牲大量的吞吐量性能
- 不需要更大的 Java Heap

G1 收集器设计目标是取代 CMS 收集器，它同 CMS 相比，在**以下方面表现的更出色**

- G1 是一个**有整理内存过程**的垃圾收集器，不会产生很多内存碎片。
- G1 的 Stop The World（STW）更可控，G1 在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

CMS 垃圾收集器虽然减少了暂停应用程序的运行时间，但是它还存在着内存碎片问题。于是，为了去除内存碎片问题，同时又保留 CMS 垃圾收集器低暂停时间的优点，JAVA7 发布了一个新的垃圾收集器：G1垃圾收集器

G1 是在2012年才在 JDK1.7 中可用，Oracle 官方计划在 JDK9 中将 G1 变成默认的垃圾收集器以替代 CMS，它是一款面向服务端应用的收集器，主要应用在多 CPU 和大内存服务器环境下，极大减少垃圾收集的停顿时间，全面提升服务器的性能，逐步替换 Java8 以前的 CMS 收集器

主要改变是：Eden，Survivor 和 Tenured 等内存区域不再是连续了，而是**变成一个个大小一样的 region**，每个region 从 1M 到 32M 不等。一个 region 有可能属于 Eden，Survivor 或者 Tenured 内存区域。

### 特点

- G1 能充分利用多 CPU，多核环境硬件优势，**尽量缩短 STW**
- G1 整体上采用标记-整理算法，局部是通过复制算法，**不会产生内存碎片**
- 宏观上看 G1 之中不再区分年轻代和老年代。把内存划分成多个独立的子区域（Region），可以近似理解为一个围棋的棋盘
- G1 收集器里面将整个内存区域都混合在一起了，但其本身依然在小范围内要进行年轻代和老年代的区分，保留了新生代和老年代，但他们不再是物理隔离的，而是通过一部分 Region 的集合，且不需要 Region 是连续的，也就是说依然会采取不同的 GC 方式来处理不同的区域
- G1 虽然也是分代收集器，但整个内存分区不存在物理上的年轻代和老年代的区别，也不需要完全独立的Survivor（to space）堆做复制准备，G1 只有逻辑上的分代概念，或者说每个分区都可能随 G1 的运行在不同代之间前后切换。

### 底层原理

Region 区域化垃圾收集器，化整为零，打破了原来新生区和老年区的壁垒，避免了全内存扫描，只需要按照区域来进行扫描即可。

区域化内存划片 Region，整体变为了一些列不连续的内存区域，避免了全内存区的 GC 操作。

**核心思想是将整个堆内存区域分成大小相同的子区域（Region），在 JVM 启动时会自动设置子区域大小**

在堆的使用上，G1 并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数`-XX:G1HeapRegionSize=n` 可指定分区大小（1MB~32MB，且必须是2的幂），默认将整堆划分为2048个分区。

大小范围在1MB~32MB，最多能设置 2048 个区域，也即能够支持的最大内存为：32MB*2048 = 64G内存

Region区域化垃圾收集器

### Region区域化垃圾收集器

G1 将新生代、老年代的物理空间划分取消了

![g1取消传统的空间划分](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632867.png)

同时对内存进行了区域划分

![g1空间划分](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632319.png)

G1 算法将堆划分为若干个区域（Reign），它仍然属于分代收集器，这些 Region 的一部分包含新生代，新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或者 Survivor 空间

这些 Region 的一部分包含老年代，G1 收集器通过将对象从一个区域复制到另外一个区域完成了清理工作。这就意味着，在正常的处理过程中，G1完成了堆的压缩（至少是部分堆的压缩），这样也就不会有 CMS 内存碎片的问题存在了。

在 G1 中，还有一种特殊的区域，叫做 Humongous（巨大的）区域，如果一个对象占用了空间超过了分区容量50% 以上，G1收集器就认为这是一个巨型对象，这些巨型对象默认直接分配在老年代，但是如果他是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响，为了解决这个问题，G1 划分了一个 Humongous 区，它用来专门存放巨型对象。如果一个 H区 装不下一个巨型对象，那么 G1 会寻找连续的 H区 来存储，为了能找到连续的 H区，有时候不得不启动Full GC。

### 回收步骤

针对Eden区进行收集，Eden区耗尽后会被触发，主要是小区域收集 + 形成连续的内存块，避免内碎片

- Eden区的数据移动到Survivor区，加入后出现Survivor区空间不够，Eden区数据会晋升到Old区
- Survivor区的数据移动到新的Survivor区，部分数据晋升到Old区
- 最后Eden区收拾干净了，GC结束，用户的应用程序继续执行

回收开始：

![eden回收1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632203.png)

回收完成后：

![eden回收2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632668.png)

小区域收集 + 形成连续的内存块，最后在收集完成后，就会形成连续的内存空间，这样就解决了内存碎片的问题

### 四步过程

- **初始标记**：只标记GC Roots能直接关联到的对象
- **并发标记**：进行GC Roots Tracing（链路扫描）的过程
- **最终标记**：修正并发标记期间，因为程序运行导致标记发生变化的那一部分对象
- **筛选回收**：根据时间来进行价值最大化回收

![g1收集器运行示意图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251632239.png)

### 参数配置

开发人员仅仅需要申明以下参数即可

三步归纳：`-XX:+UseG1GC -Xmx32G -XX:MaxGCPauseMillis=100`

`-XX:MaxGCPauseMillis=n`：最大GC停顿时间单位毫秒，这是个**软目标**，JVM尽可能停顿小于这个时间

### G1和CMS比较

- **G1 不会产生内碎片**
- **G1 是可以精准控制停顿**。该收集器是把整个堆（新生代、老年代）划分成多个固定大小的区域，每次根据允许停顿的时间去收集垃圾最多的区域。

## SpringBoot 结合 JVM GC

启动微服务时候，就可以带上 JVM 和 GC 的参数

- IDEA 开发完微服务工程
- maven 进行 clean package，然后通过 package 打成相应的 jar 包
- 要求微服务启动的时候，同时配置我们的 JVM / GC 的调优参数，这样我们就可以根据具体的业务配置我们启动的 JVM 参数

例如：

```bash
java -server -Xms1024m -Xmx1024 -XX:UseG1GC -jar xxx.jar
```
