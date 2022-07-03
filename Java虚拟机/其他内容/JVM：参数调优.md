#  JVM：参数调优

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

### 前言

查看 JVM 系统默认值：使用 jps 和 jinfo 进行查看

```bash
-Xms：初始堆空间
-Xmx：最大堆空间
-Xss：栈空间
```

-Xms 和 -Xmx 最好调整一致，防止 JVM 频繁进行收集和回收

## JVM参数类型

标配参数（从JDK1.0 - Java12都在，很稳定）

- `java -version`
- `java -help`
- `java -showversion`

X 参数（了解）

- `-Xint`：解释执行
- `-Xcomp`：第一次使用就编译成本地代码
- `-Xmixed`：混合模式

**XX参数（重点）**

- Boolean类型

  - 公式：`-XX:+/-某个属性`

    > +表示开启 / -表示关闭

  - 如：`-XX:-PrintGCDetails`：表示关闭了GC详情输出

- key-value类型

  - 公式：`-XX:属性key=属性value`
  - 不满意初始值，可以通过相应命令调整
  - 如：`-XX:MetaspaceSize=21807104`，调整Java元空间的值

### 使用

**查看运行的 Java 程序，JVM 参数是否开启，具体值为多少？**

首先我们运行一个HelloGC的java程序

```java
public class HelloGC {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("hello GC");
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

然后使用下列命令查看它的默认参数

```bash
jps：查看java的后台进程
jinfo：查看正在运行的java程序
```

具体使用：

```bash
C:\Users\ZhuCC\Desktop\Java\Code\DemoCode>jps -l
34452 org.jetbrains.jps.cmdline.Launcher
37396 sun.tools.jps.Jps
37624 HelloGC
23756
```

查看到 HelloGC 的进程号为：37624

我们使用 jinfo -flag 然后查看是否开启 PrintGCDetails 这个参数

```bash
jinfo -flag PrintGCDetails 37624
```

得到的内容为

```bash
-XX:-PrintGCDetails
```

上面提到了，`-`号表示关闭，即没有开启 PrintGCDetails 这个参数

下面我们需要在启动 HelloGC 的时候，增加 PrintGCDetails 这个参数，需要在运行程序的时候配置 JVM 参数

在 IDEA 中可以通过 `Run->Edit Configurations->VM options` 中进行添加相应的 JVM 参数配置：

然后在 VM Options 中加入下面的代码，现在 + 号表示开启

```bash
-XX:+PrintGCDetails
```

然后在使用 jinfo 查看我们的配置

```bash
jinfo -flag PrintGCDetails 33788
```

得到的结果为

```bash
-XX:+PrintGCDetails
```

我们看到原来的`-`号变成了`+`号，说明我们通过 VM Options 配置的 JVM 参数已经生效了

使用下列命令，会把 JVM 的全部默认参数输出

```bash
jinfo -flags PID
```

如，对上述 HelloGC 程序使用该命令，输出如下：

```bash
C:\Users\ZhuCC\Desktop\Java\Code\DemoCode>jinfo -flags 33788
Attaching to process ID 33788, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.241-b07
# 不是默认的 JVM 配置，会根据使用环境的新能而确定相应值
Non-default VM flags: -XX:CICompilerCount=3 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=427819008
0 -XX:MaxNewSize=1426063360 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496
 -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTi
meStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
# 通过Command line自行配置的JVM参数
Command line:  -XX:+PrintGCDetails -javaagent:C:\Software\JetBrains\IntelliJ IDEA 2017.3.7\lib\idea
_rt.jar=7172:C:\Software\JetBrains\IntelliJ IDEA 2017.3.7\bin -Dfile.encoding=UTF-8
```

### 题外话

两个经典参数：`-Xms` 和 `-Xmx`，这两个参数如何解释

这两个参数，还是属于 XX 参数，因为取了别名

- `-Xms` 等价于 `-XX:InitialHeapSize` ：初始化堆内存（默认只会用最大物理内存的1/64）
- `-Xmx` 等价于 `-XX:MaxHeapSize` ：最大堆内存（默认只会用最大物理内存的1/4）

## JVM 默认参数

`java -XX:+PrintFlagsInitial`：主要是查看初始默认值

- `java -XX:+PrintFlagsInitial -version`
- `java -XX:+PrintFlagsInitial`（重要参数）

```bash
C:\Users\ZhuCC>java -XX:+PrintFlagsInitial
[Global flags]
     intx ActiveProcessorCount                    = -1          {product}
    uintx AdaptiveSizeDecrementScaleFactor        = 4           {product}
    uintx AdaptiveSizeMajorGCDecayTimeScale       = 10          {product}
    uintx AdaptiveSizePausePolicy                 = 0           {product}
     ...
    intx CICompilerCount                           = 2          {product}
     ...
```

`java -XX:+PrintFlagsFinal`：表示修改以后，最终的值，即经过调整后的JVM参数值

```bash
C:\Users\ZhuCC>java -XX:+PrintFlagsFinal
[Global flags]
     intx ActiveProcessorCount                      = -1            {product}
    uintx AdaptiveSizeDecrementScaleFactor          = 4             {product}
    uintx AdaptiveSizeMajorGCDecayTimeScale         = 10            {product}
    uintx AdaptiveSizePausePolicy                   = 0             {product}
	 ...
     intx CICompilerCount                          := 3             {product}
     ...
```

**注意：**如果有 `:=` 表示修改过的， `=` 表示没有修改过的

## 常用 JVM 基本配置参数

![永久代替换为元空间](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631565.png)

> 划下重点：
>
> * **JDK8 使用了元空间取代了 JDK6 中的永久代**
> * **元空间用本地物理内存，不再由JVM管理**
> * **字符串池和类的静态变量还是在Java堆中**

### 查看堆内存

查看 JVM 的初始化堆内存 `-Xms` 和最大堆内存 `-Xmx`

```java
public class HeapSize {
    public static void main(String[] args) {
        // 返回Java虚拟机中内存的总量
        long totalMemory = Runtime.getRuntime().totalMemory();

        // 返回Java虚拟机中试图使用的最大内存量
        long maxMemory = Runtime.getRuntime().maxMemory();

        System.out.println("TOTAL_MEMORY(-Xms) = " + totalMemory + "(Byte) -- " + (totalMemory / (double)1024 / 1024) + "MB");
        System.out.println("MAX_MEMORY(-Xmx) = " + maxMemory + "(Byte) -- " + (maxMemory / (double)1024 / 1024) + "MB");
    }
}
```

运行结果为：

```
TOTAL_MEMORY(-Xms) = 257425408(Byte) -- 245.5MB
MAX_MEMORY(-Xmx) = 3803185152(Byte) -- 3627.0MB
```

**`-Xms` 初始堆内存为：物理内存的1/64** 

**`-Xmx` 最大堆内存为：系统物理内存的1/4**

### 打印 JVM 默认参数

使用 JVM 参数 `-XX:+PrintCommandLineFlags` 打印出 JVM 的默认的简单初始化参数

比如我的机器输出为：

```bash
-XX:InitialHeapSize=267290176 -XX:MaxHeapSize=4276642816 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
```

### 一些常用调优参数

- `-Xms`：初始化堆内存，默认为物理内存的1/64，等价于 `-XX:initialHeapSize`

- `-Xmx`：最大堆内存，默认为物理内存的1/4，等价于 `-XX:MaxHeapSize`

- `-Xss`：设计单个线程栈的大小，一般默认为512K~1024K，等价于 `-XX:ThreadStackSize`
  
  > 进一步说明：
  >
  > - 使用 `jinfo -flag ThreadStackSize` 会发现 `-XX:ThreadStackSize=0`
  > - 这个值的大小是取决于平台的
  >   - Linux/x64：1024KB
  >   - OS X：1024KB
  >   - Oracle Solaris：1024KB
  >   - Windows：取决于虚拟内存的大小
  
- `-Xmn`：设置年轻代大小

- `-XX:MetaspaceSize`：设置元空间大小
  
  - 元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现，不过元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**，因此，默认情况下，元空间的大小仅受本地内存限制。
  - 但是默认的元空间大小：只有20多M `-XX:MetaspaceSize=21807104`
  - 为了防止在频繁的实例化对象的时候，让元空间出现OOM，因此可以把元空间设置的大一些：`-Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX:+PrintCommandLineFlags`
  
- `-XX:PrintGCDetails`：输出详细GC收集日志信息

- `-XX:SurvivorRatio`：调节新生代中 eden 和 S0、S1的空间比例

- `-XX:NewRatio`：配置年轻代 new 和老年代 old 在堆结构的占比

- `-XX:MaxTenuringThreshold`：设置对象进入老年代的阈值

### 垃圾回收

JVM 命令 `-XX:PrintGCDetails`：输出详细GC收集日志信息

- **GC**
- **Full GC**

我们使用一段代码，制造出垃圾回收的过程

首先我们设置一下程序的启动配置: 设置初始堆内存为10M，最大堆内存为10M

```bash
-Xms10m -Xmx10m -XX:+PrintGCDetails
```

然后用下列代码，创建一个大于堆空间大小的byte类型数组

```java
byte[] byteArray = new byte[50 * 1024 * 1024];
```

运行后，发现会出现下列错误，这就是OOM：java内存溢出，也就是堆空间不足

```bash
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at HeapSize.main(HeapSize.java:5)
```

同时还打印出了GC垃圾回收时候的详情

```bash
[GC (Allocation Failure) [PSYoungGen: 1658K->488K(2560K)] 1658K->632K(9728K), 0.0015767 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 488K->488K(2560K)] 632K->648K(9728K), 0.0004406 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 160K->598K(7168K)] 648K->598K(9728K), [Metaspace: 3212K->3212K(1056768K)], 0.0033779 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 598K->598K(9728K), 0.0001481 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 598K->580K(7168K)] 598K->580K(9728K), [Metaspace: 3212K->3212K(1056768K)], 0.0031963 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 2560K, used 138K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 6% used [0x00000000ffd00000,0x00000000ffd228c8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 7168K, used 580K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 8% used [0x00000000ff600000,0x00000000ff691358,0x00000000ffd00000)
 Metaspace       used 3259K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
```

问题发生的原因：

因为们通过 `-Xms10m` 和 `-Xmx10m` 只给Java堆栈设置了10M的空间，但是创建了50M的对象，因此就会出现空间不足，而导致出错

同时在垃圾收集的时候，我们看到有两个过程：GC 和 Full GC

**GC过程：**

```bash
[GC (Allocation Failure) [PSYoungGen: 1972K->504K(2560K)] 1972K->740K(9728K), 0.0156109 secs] [Times: user=0.00 sys=0.00, real=0.03 secs]
# 其中：GC (Allocation Failure)：表示分配失败，那么就需要触发年轻代空间中的内容被回收
# [PSYoungGen: 1972K->504K(2560K)] 1972K->740K(9728K)
```

![GC说明](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631996.png)

**Full GC：**

> Full GC 大部分发生在老年代

```bash
[Full GC (Allocation Failure) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 160K->598K(7168K)] 648K->598K(9728K), [Metaspace: 3212K->3212K(1056768K)], 0.0033779 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
```

![FullGC说明](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631575.png)

**规律：**

```
[名称： GC前内存占用 -> GC后内存占用 (该区内存总大小)]
```

当我们出现了老年代垃圾回收都没有用的时候，就会出现OOM异常

### 调整Heap中各区策略

1. `-XX:SurvivorRatio`

   调节新生代中 eden 和 from、to 的空间比例，默认为 `-XX:SurvivorRatio=8`，即：`Eden:from:to = 8:1:1`，如下图：

   ![SurvivorRatio为8](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631447.PNG)

   设置为：`-XX:SurvivorRatio=4`，则为 `Eden:from:to = 4:1:1`，如下图：

   ![SurvivorRatio为4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631780.PNG)

   `SurvivorRatio` 值就是设置 eden 区的比例占多少，from 和 to 相同

   > Java堆从GC的角度还可以细分为：新生代（Eden区，From Survivor区合To Survivor区）和老年代
   >
   > ![堆空间](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631300.png)
   >
   > 1. 对象从eden与SurvivorFrom复制到SurvivorTo中，年龄 + 1
   >
   >       进一步：首先，当Eden区满的时候会触发第一次GC，把还活着的对象拷贝到SurvivorFrom去，当Eden区再次触发GC的时候会扫描Eden区合From区域，对这两个区域进行垃圾回收，经过这次回收后还存活的对象，则直接复制到To区域（如果对象的年龄已经到达老年的标准，则赋值到老年代区），通知把这些对象的年龄 + 1；
   >
   > 2. 清空eden与SurvivorFrom
   >
   >       然后，清空eden，SurvivorFrom中的对象，也即复制之后有交换，谁空谁是to；
   >
   > 3. SurvivorTo和SurvivorFrom互换
   >
   >       最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区，部分对象会在From和To区域中复制来复制去，如此交换15次（由JVM参数`MaxTenuringThreshold`决定，这个参数默认为15），最终如果还是存活，就存入老年代

2. **-XX:NewRatio**

   配置年轻代 new 和老年代 old 在堆结构的占比，NewRadio 值就是设置老年代的占比

   默认：`-XX:NewRatio=2` 新生代占1，老年代2，年轻代占整个堆的1/3，如下图：

   ![NewRatio为2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631899.PNG)

   `-XX:NewRatio=4`：新生代占1，老年代占4，年轻代占整个堆的1/5，这样会导致新生代新生代特别小，造成频繁的进行GC收集

   ![NewRatio为4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251631481.PNG)

3. **-XX:MaxTenuringThreshold**

   设置对象进入老年代的阈值，SurvivorTo 和 SurvivorFrom 互换，原 SurvivorTo 成为下一次 GC 时的SurvivorFrom 区，部分对象会在 From 和 To 区域中复制来复制去，如此交换 n 次（n 就是通过该 JVM 参数设置），最终如果还是存活，就存入老年代

   这里就是调整这个次数的，默认是15，并且在 Java8 中规定该值在 0~15之间

   查看默认进入老年代年龄：`jinfo -flag MaxTenuringThreshold PID`

   `-XX:MaxTenuringThreshold=0`：设置垃圾最大年龄。如果设置为0的话，则年轻对象不经过 Survivor 区，直接进入老年代。对于年老代比较多的应用，可以提高效率。

   而如果将此值设置为一个较大的值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即被回收，减少 Full GC 发生的次数。