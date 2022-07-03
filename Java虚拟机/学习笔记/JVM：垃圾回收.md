# JVM：垃圾回收

> 在学习了 bilibili 上 **[黑马程序员]()** 的课程 [JVM完整教程](https://www.bilibili.com/video/BV1yE411Z7AP) 后做的学习笔记，总共分为5篇笔记，本文为 2/5 篇。
>

### 内容

1. 如何判断对象可以回收
2. 垃圾回收算法
3. 分代垃圾回收
4. 垃圾回收器
5. 垃圾回收调优 

## 1. 如何判断对象可以回收 

### 1.1 引用计数法 

**描述：**

对象被引用，计数+1，当不再被引用了，计数-1

当对象没有被引用了，可以被垃圾回收

**存在问题：**

![引用计数法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622814.PNG)

循环引用：A对象引用B，B对象引用A，而没有其他对象引用AB对象了；但是AB对象的引用计数永不为0。

### 1.2 可达性分析算法 

**根对象**：肯定不能被当成垃圾回收的对象。

* Java 虚拟机中的垃圾回收器采用可达性分析来探索所有存活的对象

* 扫描堆中的对象，看是否能够沿着 GC Root 对象为起点的引用链找到该对象，找不到，表示可以回收；

* 哪些对象可以作为 GC Root ? 

**演示GC Roots**

```java
public class Demo2_2 {
    public static void main(String[] args) throws InterruptedException, IOException {
        List<Object> list1 = new ArrayList<>();
        list1.add("a");
        list1.add("b");
        System.out.println(1);
        System.in.read();

        list1 = null;
        System.out.println(2);
        System.in.read();
        System.out.println("end...");
    }
}
```

1. 其中程序，运行到第一个`System.in.read();`时，通过 jmap 抓取内存快照；

   1. 通过 jps 查看 java 进程id；

   2. 通过 jmap 抓取内存快照：`jmap -dump:format=b, live, file=1.bin 27840` 

      `-dump`：把运行时的状态抓取成一个文件；

      `format=b`：转储文件的格式，b-二进制格式;

      `live` ：只关心存活对象，且在抓取快照之前会进行一次垃圾回收;

      `file=1.bin` ：内存快照存储在对应文件

      `27840` ：对应的进程ID
      
   3. 执行结果如下：
   
      ```
      C:...>jmap -dump:format=b,live,file=1.bin 27840
      Dumping heap to C:\...\1.bin ...
      Heap dump file created
      ```
   
2. 回车，将 list1 引用置为空时，再次抓取内存快照`jmap -dump:format=b,live,file=2.bin 27840`
   
3. 通过`Eclipse Memory Analyzer`对上述生成的文件进行分析

4. 分析过程略....

5. **可以分析有哪些类作为 GC Root类**

   1. `System Class`：由启动类加载器加载的系统类，如Object类、HashMap类、String类...
   2. `Native Stack`：Java虚拟机在执行一些方法时，会调用一些操作系统的方法，操作方法在执行时所引用的一些Java对象；
   3. `Thread`：活动线程中使用的一些对象；
   4. `Busy Monitor`：正在加锁的对象。

6. 上述代码中，`new ArrayList<>();`在`list1 = null;`之后，在抓取第二次内存快照时被垃圾回收了

### 1.3 四种引用 

1. **强引用**
   * 通过 new 创建一个对象，通过赋值运算符赋值给一个变量，变量就强引用了对象
   * 只有所有 GC Roots 对象都不通过【强引用】引用该对象，该对象才能被垃圾回收

2. **软引用（SoftReference）**
   * 仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次触发垃圾回收，回收软引用对象；即一般不会轻易回收，只有内存不够时才回收
   * 可以配合引用队列来释放软引用自身

> 引用队列：软引用的对象被被回收后，**软引用自身也是一个对象**，在创建时为软引用分配了引用队列，则当软引用引用的对象被回收时，软引用会进入引用队列
>
> 在引用队列中依次遍历，释放引用占用内存

3. **弱引用（WeakReference）**

   * 仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象

   * 可以配合引用队列来释放弱引用自身

4. **虚引用（PhantomReference）**

   * 虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收
   
   * 必须配合引用队列使用，主要配合 ByteBuffer 使用，被引用对象回收时，会将虚引用入队，由 Reference Handler 线程调用虚引用相关方法释放直接内存
   
   > 在内存结构章节中提到的Cleaner对象就是一个虚引用

5. 终结器引用（FinalReference）

   * 无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队（被引用对象暂时没有被回收），再由 Finalizer 线程通过终结器引用找到被引用对象并调用它的 finalize方法，第二次 GC 时才能回收被引用对象 

#### 演示软引用

```java
/**
 * 演示软引用
 * -Xmx20m
 *  设置虚拟机内存为20Mb
 */
public class Demo2_3 {

    private static final int _4MB = 4 * 1024 * 1024;
    
    public static void main(String[] args) throws IOException {
        // 强引用
        List<byte[]> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add(new byte[_4MB]);
        }
        System.in.read();
    }
}
```

报堆内存溢出的错误：

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

将上述代码修改为：

```java
public static void main(String[] args) throws IOException {
	soft();
}

public static void soft() {
    // list --> SoftReference --> byte[]
    List<SoftReference<byte[]>> list = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB]);
        System.out.println(ref.get());
        list.add(ref);
        System.out.println(list.size());
    }
    System.out.println("循环结束：" + list.size());
    for (SoftReference<byte[]> ref : list) {
        System.out.println(ref.get());
    }
}
```

输出结果：

```java
[B@7f31245a
1
[B@6d6f6e28
2
[B@135fbaa4
3
[B@45ee12a7
4
[B@330bedb4
5
循环结束：5
null  
null
null
null           // 数组的前四个元素都变成了null
[B@330bedb4    // 只有最后一个被保存下来了
```

详细分析：添加JVM参数：`-Xmx20m -XX:+PrintGCDetails -verbose:gc`

```java
[B@7f31245a
1     // 第一次添加没问题
[B@6d6f6e28
2     // 第二次添加没问题
[B@135fbaa4
3     // 第三次添加没问题，虽然已经触发了垃圾回收，新生代的垃圾回收
[GC (Allocation Failure) [PSYoungGen: 1946K->488K(6144K)] 14234K->12982K(19968K), 0.0007510 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]   
[B@45ee12a7
4     // 第四次添加，已经触发了垃圾回收
[GC (Allocation Failure) --[PSYoungGen: 4696K->4696K(6144K)] 17191K->17215K(19968K), 0.0005606 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 4696K->4551K(6144K)] [ParOldGen: 12518K->12463K(13824K)] 17215K->17014K(19968K), [Metaspace: 3230K->3230K(1056768K)], 0.0029214 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) --[PSYoungGen: 4551K->4551K(6144K)] 17014K->17022K(19968K), 0.0006793 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 4551K->0K(6144K)] [ParOldGen: 12471K->612K(8704K)] 17022K->612K(14848K), [Metaspace: 3230K->3230K(1056768K)], 0.0038851 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@330bedb4
5    // 第五次添加，内存不足了，触发了软引用的垃圾回收，导致数组前四个内容被回收成为null
循环结束：5
null
null
null
null
[B@330bedb4
Heap
 PSYoungGen      total 6144K, used 4377K [0x00000000ff980000, 0x0000000100000000, 0x0000000100000000)
  eden space 5632K, 77% used [0x00000000ff980000,0x00000000ffdc6548,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 8704K, used 612K [0x00000000fec00000, 0x00000000ff480000, 0x00000000ff980000)
  object space 8704K, 7% used [0x00000000fec00000,0x00000000fec99278,0x00000000ff480000)
 Metaspace       used 3236K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 351K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0
```

上述代码中，软引用的对象已经为 null 了，对于这些软引用已经没必要保存在 list 对象中了，下述对软引用本身做一个清理

**配合引用队列对软引用做一个清理**

```java
/**
 * 演示软引用, 配合引用队列
 */
public class Demo2_4 {
    private static final int _4MB = 4 * 1024 * 1024;

    public static void main(String[] args) {
        List<SoftReference<byte[]>> list = new ArrayList<>();

        // 引用队列
        ReferenceQueue<byte[]> queue = new ReferenceQueue<>();

        for (int i = 0; i < 5; i++) {
            // 关联了引用队列， 当软引用所关联的byte[]被回收时，软引用自己会加入到 queue 中去
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB], queue);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());
        }

        // 从队列中获取无用的 软引用对象，并移除
        Reference<? extends byte[]> poll = queue.poll();
        while( poll != null) {
            list.remove(poll);
            poll = queue.poll();
        }
        System.out.println("===========================");
        for (SoftReference<byte[]> reference : list) {
            System.out.println(reference.get());
        }
    }
}
```

输出结果：

```java
[B@7f31245a
1
[B@6d6f6e28
2
[B@135fbaa4
3
[B@45ee12a7
4
[B@330bedb4
5
===========================
[B@330bedb4  // 可以看到list中被回收的软引用对象已经被移除了
```

#### 演示弱应用

```java
/**
 * 演示弱引用
 * -Xmx20m -XX:+PrintGCDetails -verbose:gc
 */
public class Demo2_5 {
    private static final int _4MB = 4 * 1024 * 1024;

    public static void main(String[] args) {
        //  list --> WeakReference --> byte[]
        List<WeakReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            WeakReference<byte[]> ref = new WeakReference<>(new byte[_4MB]);
            list.add(ref);
            for (WeakReference<byte[]> w : list) {
                System.out.print(w.get()+" ");
            }
            System.out.println();
        }
        System.out.println("循环结束：" + list.size());
    }
}
```

循环5次的输出结果：

```java
[B@7f31245a 
[B@7f31245a [B@6d6f6e28 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 
[GC (Allocation Failure) [PSYoungGen: 1944K->488K(6144K)] 14232K->12978K(19968K), 0.0010070 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 [B@45ee12a7  // 第四次循环触发了垃圾回收，弱引用引用的对象还没有被回收
[GC (Allocation Failure) [PSYoungGen: 4696K->504K(6144K)] 17187K->13010K(19968K), 0.0005339 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null [B@330bedb4 // 第五次循环，无法将byte[]放入list中了，触发垃圾回收，list中的第四个对象被回收
循环结束：5
```

循环6次的输出结果：

```java
[B@7f31245a 
[B@7f31245a [B@6d6f6e28 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 
[GC (Allocation Failure) [PSYoungGen: 1833K->488K(6144K)] 14121K->13010K(19968K), 0.0031088 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 [B@45ee12a7 
[GC (Allocation Failure) [PSYoungGen: 4696K->472K(6144K)] 17219K->12994K(19968K), 0.0027955 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null [B@330bedb4 
[GC (Allocation Failure) [PSYoungGen: 4680K->456K(6144K)] 17202K->12978K(19968K), 0.0046990 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null [B@2503dbd3 // 只有把第5个对象删除后，第6个对象才能放进list中
循环结束：6
```

循环10次的输出结果：

```java
[B@7f31245a 
[B@7f31245a [B@6d6f6e28 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 
[GC (Allocation Failure) [PSYoungGen: 1946K->488K(6144K)] 14234K->12978K(19968K), 0.0007414 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 [B@45ee12a7 
[GC (Allocation Failure) [PSYoungGen: 4696K->504K(6144K)] 17187K->13018K(19968K), 0.0005572 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null [B@330bedb4 
[GC (Allocation Failure) [PSYoungGen: 4712K->504K(6144K)] 17226K->13026K(19968K), 0.0004535 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null [B@2503dbd3 
[GC (Allocation Failure) [PSYoungGen: 4711K->488K(6144K)] 17233K->13026K(19968K), 0.0003885 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null [B@4b67cf4d 
[GC (Allocation Failure) [PSYoungGen: 4694K->504K(6144K)] 17233K->13042K(19968K), 0.0003820 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null null [B@7ea987ac 
[GC (Allocation Failure) [PSYoungGen: 4710K->488K(5120K)] 17249K->13026K(18944K), 0.0005374 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null null null [B@12a3a380 
[GC (Allocation Failure) [PSYoungGen: 4674K->32K(5632K)] 17212K->13042K(19456K), 0.0013349 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 32K->0K(5632K)] [ParOldGen: 13010K->630K(7680K)] 13042K->630K(13312K), [Metaspace: 3229K->3229K(1056768K)], 0.0041599 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
null null null null null null null null null [B@29453f44 
循环结束：10
```

## 2. 垃圾回收算法 

### 2.1 标记清除 

定义： Mark Sweep 

分为标记和清除两个步骤：

![标记和清除](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622064.PNG)

标记：标记出可以被清除的对象；

清除：把垃圾对象占用的空间清除，此处的清除为把对象占用的内存起始结束地址进行记录，在下次分配对象地址时可以分配到这些地址中。

**优点：**速度快，清除时只需要记录垃圾对象的起始结束地址即可；

**缺点：**容易产生内存碎片

### 2.2 标记整理 

定义：Mark Compact 

标记整理也分为两个步骤：

![标记整理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622181.PNG)

标记：标记步骤与标记清除相同；

整理：为了避免标记清除时产生的内存碎片问题，在清理的过程中，对对象进行整理，使得对象在内存空间中更为紧凑。

**优点**：不会产生内存碎片；

**缺点**：在整理过程中，涉及到了对象的移动，因此速度较慢。

### 2.3 复制拷贝

定义：Copy

![复制](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622092.PNG)

将内存区划分了内存大小相同的两块区域；

在 FROM 区中进行一次标记处理；

将 FROM 区中存活的对象复制到 TO 区域中，复制过程中完成对象的整理；

在复制完成后，清空 FROM 区域，并交换 FROM 和 TO 区的位置

**优点：**不会产生内存碎片

**缺点：**需要使用双倍的内存空间

## 3. 分代垃圾回收

- 将堆内存分为两块区域：**新生代**与**老年代**；
- 有些需要长时间使用的对象放置老年代中；将那些用完后不需要再使用的对象放置于新生代中；
- 这样可针对对象生命周期不同的特点，选择不同的垃圾回收策略，新生代中垃圾回收发生较频繁。

![分代垃圾回收](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622861.PNG)

**分代垃圾回收工作机制**：

* 创建的新对象首先分配在伊甸园区域，之后更多对象被创建，放入伊甸园中；
* 新生代空间不足时，触发 `Minor GC`，伊甸园和From存活的对象使用**copy复制**到to中，存活的对象年龄+1并且交换from和to；
* `Minor GC` 会引发 `stop the world`，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行，这是出于在GC时，涉及到了对象地址的修改；
* 触发`Minor GC`后，伊甸园有空间了，继续将生成的对象放入，直至再次发生`Minor GC`进行相应操作；
* 当幸存区中对象的寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit）；
* 当老年代空间不足，会先尝试触发 `Minor GC`，如果之后空间仍不足，那么触发 `Full GC`，`stop the world`的时间更长；
* 若触发 `Full GC`后，老年代的空间还是不足，则会导致堆内存溢出。

### 3.1 相关 VM 参数 

|                    | 参数                                                         |
| ------------------ | ------------------------------------------------------------ |
| 堆初始大小         | `-Xms`                                                       |
| 堆最大大小         | `-Xmx` 或 `-XX:MaxHeapSize=size`                             |
| 新生代大小         | `-Xmn` 或 (`-XX:NewSize=size` + `-XX:MaxNewSize=size`)       |
| 幸存区比例（动态） | `-XX:InitialSurvivorRatio=ratio` 和 `-XX:+UseAdaptiveSizePolicy` |
| 幸存区比例         | `-XX:SurvivorRatio=ratio`                                    |
| 晋升阈值           | `-XX:MaxTenuringThreshold=threshold`                         |
| 晋升详情           | `-XX:+PrintTenuringDistribution`                             |
| GC详情             | `-XX:+PrintGCDetails -verbose:gc`                            |
| FullGC前MinorGC    | `-XX:+ScavengeBeforeFullGC`                                  |

### 3.2 垃圾回收案例

```java
/**
 *  演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args){
		
    }
}
```

JVM 参数设置：

* `-Xms20M`设置初始的堆空间为20Mb
* `-Xmx20M`设置堆最大大小为20Mb
* `-Xmn10M`设置新生代大小为10Mb
* `-XX:+UseSerialGC`：使用串行垃圾回收器，JDK8下的GC不是这个，修改为这个GC后幸存区的比例才不会动态调整；
* `-XX:+PrintGCDetails -verbose:gc`：打印GC详情
* `-XX:-ScavengeBeforeFullGC`：Full GC 前 Minor GC 

**主函数中内容为空的运行结果：**

```java
// 没有发生GC，只打印了程序结束后堆的情况
Heap  // 堆内存
 // new generation 新生代的信息
 // total->总空间（由于幸存区to空间需要始终空着，不能使用，故这里没有算入to空间，因此不满10Mb）
 // used-> 已使用的空
 // [... ... ...)->内存地址
 def new generation   total 9216K, used 2158K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  // 伊甸园   空间     已经使用空间                                          
  eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee1b9a0, 0x00000000ff400000)
  // 幸存区from                                    
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  // 幸存区  to                             
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
// tenured generation 老年代的信息
// total -> 总空间    used->已经用的空间
 tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
// 元空间  --> 不属于堆空的一部分
 Metaspace       used 3231K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0
```

**主函数内加入下方代码：**

```java
ArrayList<byte[]> list = new ArrayList<>();
// 占用_7MB空间 会导致伊甸园空间不足 会触发一次Minor GC
list.add(new byte[_7MB]);
```

输出结果：

```java
// 发生了一次GC--> GC==Minor GC
// DefNew:发生在新手代  
// 1830K->650K(9216K)：回收前的内存占用->回收后的内存占用   
// (9216K):新生代总的内存占用    
// 0.0014230 secs：垃圾回收占用时间
// 1830K->650K(19456K)：整个堆回收前的内存占用->回收后的内存占用
// (19456K)：堆的总大小
// 0.0014565 secs：回收锁耗费的时间
[GC (Allocation Failure) [DefNew: 1830K->650K(9216K), 0.0014230 secs] 1830K->650K(19456K), 0.0014565 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 8065K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  // 此时伊甸园中空间使用了90%
  eden space 8192K,  90% used [0x00000000fec00000, 0x00000000ff33d8c0, 0x00000000ff400000)
  // 可以看到，此时from中有对象了
  from space 1024K,  63% used [0x00000000ff500000, 0x00000000ff5a2b80, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
 Metaspace       used 3230K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

**主函数内代码修改如下：**

```java
ArrayList<byte[]> list = new ArrayList<>();
list.add(new byte[_7MB]);
list.add(new byte[_512KB]);
```

输出结果：

```java
[GC (Allocation Failure) [DefNew: 1994K->650K(9216K), 0.0013393 secs] 1994K->650K(19456K), 0.0013693 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 8577K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  // 添加了一个_512KB的byte数组，对比上方代码伊甸园中的内存使用由90%->96%
  eden space 8192K,  96% used [0x00000000fec00000, 0x00000000ff3bd8d0, 0x00000000ff400000)
  from space 1024K,  63% used [0x00000000ff500000, 0x00000000ff5a2bb0, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
 Metaspace       used 3232K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

**主函数内代码修改如下：**

```java
ArrayList<byte[]> list = new ArrayList<>();
list.add(new byte[_7MB]);
list.add(new byte[_512KB]);
// 再放入_512KB，此时伊甸园中的内存必然不够了，再次触发GC
list.add(new byte[_512KB]);
```

输出：

```java
[GC (Allocation Failure) [DefNew: 1830K->650K(9216K), 0.0011813 secs] 1830K->650K(19456K), 0.0012089 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
// 可以看到，再加入_512KB的byte数组，触发了一次新的GC操作
[GC (Allocation Failure) [DefNew: 8494K->512K(9216K), 0.0043475 secs] 8494K->8310K(19456K), 0.0043671 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 1188K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  // 伊甸园内存占用情况
  eden space 8192K,   8% used [0x00000000fec00000, 0x00000000feca90e0, 0x00000000ff400000)
  // form区中内存占用情况
  from space 1024K,  50% used [0x00000000ff400000, 0x00000000ff480048, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 // 有对象晋升到了老年代中
 tenured generation   total 10240K, used 7798K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  76% used [0x00000000ff600000, 0x00000000ffd9db50, 0x00000000ffd9dc00, 0x0000000100000000)
 Metaspace       used 3230K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

**大对象直接晋升老年代：**

```java
ArrayList<byte[]> list = new ArrayList<>();
// _8MB对象已经超过了伊甸园的总容量--->直接将对象晋升到老年代中
list.add(new byte[_8MB]);
```

输出：

```java
Heap
 def new generation   total 9216K, used 2158K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee1b988, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 // 没有触发GC，老年代中已经有对象了，对象在新生代空间不足的情况下，直接晋升老年代
 tenured generation   total 10240K, used 8192K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  80% used [0x00000000ff600000, 0x00000000ffe00010, 0x00000000ffe00200, 0x0000000100000000)
 Metaspace       used 3231K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

**放入的对象超过堆内存容量：**

```java
ArrayList<byte[]> list = new ArrayList<>();
// 放入的对象超出堆内存容量
list.add(new byte[_8MB]);
list.add(new byte[_8MB]);
```

输出：

```java
// 堆内存溢出的自救
[GC (Allocation Failure) [DefNew: 1994K->650K(9216K), 0.0012508 secs][Tenured: 8192K->8841K(10240K), 0.0016668 secs] 10186K->8841K(19456K), [Metaspace: 3225K->3225K(1056768K)], 0.0029647 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
// 堆内存溢出的自救 Full GC
[Full GC (Allocation Failure) [Tenured: 8841K->8823K(10240K), 0.0013313 secs] 8841K->8823K(19456K), [Metaspace: 3225K->3225K(1056768K)], 0.0013494 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

// 但是自救失败...堆内存溢出啦！
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at cn.xyc.Demo2_1.main(Demo2_1.java:26)

Heap
 def new generation   total 9216K, used 246K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,   3% used [0x00000000fec00000, 0x00000000fec3d890, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 8823K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  86% used [0x00000000ff600000, 0x00000000ffe9df98, 0x00000000ffe9e000, 0x0000000100000000)
 Metaspace       used 3257K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
```

**再看一个现象：将上述代码加入线程中**

```java
new Thread(() -> {
    ArrayList<byte[]> list = new ArrayList<>();
    list.add(new byte[_8MB]);
    list.add(new byte[_8MB]);
}).start();

System.out.println("sleep....");
Thread.sleep(10000000L);
```

输出：

```java
sleep....  // 主线程开始等待，并没有因为线程Thread-0发生了堆内存溢出而结束

[GC (Allocation Failure) [DefNew: 4060K->856K(9216K), 0.0019035 secs][Tenured: 8192K->9046K(10240K), 0.0027099 secs] 12253K->9046K(19456K), [Metaspace: 4148K->4148K(1056768K)], 0.0046569 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [Tenured: 9046K->8990K(10240K), 0.0020630 secs] 9046K->8990K(19456K), [Metaspace: 4148K->4148K(1056768K)], 0.0020911 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

// Thread-0 经过 Minor GC 和 Full GC后，还是内存不足，发生堆内存溢出
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
	at cn.xyc.Demo2_1.lambda$main$0(Demo2_1.java:17)
	at cn.xyc.Demo2_1$$Lambda$1/1023892928.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
```

将`Thread.sleep(10000000L);`修改为`Thread.sleep(1000L);`，查看一下主线程的堆使用信息：

```java
Heap
 def new generation   total 9216K, used 218K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,   2% used [0x00000000fec00000, 0x00000000fec36b58, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 // 弹幕1：这里占用的空间是抛出OOM之前的内存信息
 // 弹幕2：当一个线程抛出OOM后，它所占据的内存资源会被释放
 // 弹幕3：OOM后会清空线程占用的堆内存
 tenured generation   total 10240K, used 8990K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  87% used [0x00000000ff600000, 0x00000000ffec7b48, 0x00000000ffec7c00, 0x0000000100000000)
 Metaspace       used 4170K, capacity 4676K, committed 4864K, reserved 1056768K
  class space    used 462K, capacity 496K, committed 512K, reserved 1048576K
```

## 4. 垃圾回收器

1. **串行**

   * 单线程
   * 堆内存较小，适合个人电脑，即CPU个数少

2. **吞吐量优先** 

   * 多线程
   * 堆内存较大，多个CPU
   * 让单位时间内， STW（stop the world）的时间最短，垃圾回收时间占比最低，这样就称吞吐量高 

   > 例如单位时间内，发生GC有：
   >
   > 0.2 0.2 = 0.4

3. **响应时间优先**

   * 多线程
   * 堆内存较大，多个CPU
   * 尽可能让单次 STW 的时间最短

   > 例如单位时间内，发生GC有：
   >
   > 0.1 0.1 0.1 0.1 0.1 = 0.5

### 4.1 串行 

打开**串行垃圾回收器**的 JVM 参数：`-XX:+UseSerialGC = Serial + SerialOld `

* **Serial：用于新生代，采用复制垃圾回收算法；**
* **SerialOld：用于老年代，采用标记整理算法；**

![串行](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622575.PNG)

触发GC时，使所有线程在一个**安全点**暂停，由于在GC时，对象的地址可能会发生修改；

由于此时Serial，SerialOld都是单线程的垃圾回收器，因此当垃圾回收器线程在运行时，其他线程进入阻塞状态。

### 4.2 吞吐量优先 

需要设置的 JVM 参数：

* `-XX:+UseParallelGC ~ -XX:+UseParallelOldGC `：使用**并行的垃圾回收器**

  * `UseParallelGC`：新生代的垃圾回收器
  * `UseParallelOldGC `：老年代的垃圾回收器

  > 在JDK1.8中默认已经开启，**开启了一个顺带也会开启另一个**

* `-XX:+UseAdaptiveSizePolicy`：使用自适应新生代大小调整策略

* `-XX:GCTimeRatio=ratio` ：用于调整垃圾回收时间与总时间占比

  > 公式：`1/(1+ratio)`
  >
  > ratio默认为99，即垃圾回收的时间不能超过总时间的1%

* `-XX:MaxGCPauseMillis=ms` ：最大暂停时间，默认为 200ms

* `-XX:ParallelGCThreads=n` ：控制垃圾回收线程数

![吞吐量优先](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622531.PNG)

触发GC时，使所有线程在一个**安全点**暂停；

垃圾回收器开启多个线程进行垃圾回收，垃圾回收线程数默认情况下与线程个数相关

### 4.3 响应时间优先 

需要设置的JVM参数：

* `-XX:+UseConcMarkSweepGC ~ -XX:+UseParNewGC ~ SerialOld`
  * Conc->concurrent**并发的**，即用户线程和GC线程是并发执行的	
  * MarkSweepGC一款基于**标记清除的垃圾回收器**
  * UseConcMarkSweepGC工作于老年代的一款垃圾回收器
  * UseParNewGC工作于新生代的垃圾回收器
  * 并发失败会退回到SerialOld，即单线程的垃圾回收器，例如内存碎片过多导致并发失败
* `-XX:ParallelGCThreads=n ~ -XX:ConcGCThreads=threads`
  * ParallelGCThreads：并行垃圾回收线程数，一般与CPU核数相同；
  * ConcGCThreads：并发的垃圾回收线程数，一般建议并行GC线程数的1/4
  * 即若有4个CPU，3个用于用户线程，1个用于并发GC

* `-XX:CMSInitiatingOccupancyFraction=percent`
  * CMSInitiatingOccupancyFraction：执行CMS垃圾回收的内存占比；
  * 例如设置为80%，老年代内存占用达到80%时，就执行一次GC；
  * 出于：在并发清理时会产生一些新的垃圾，称为**浮动垃圾**，而并发清理时不能清理这些浮动垃圾，这些垃圾需要在下一次垃圾清理是被清理，**因此需要预留空间对浮动垃圾进行保存**

* `-XX:+CMSScavengeBeforeRemark`
  * 在重新标记阶段，会出现新生代的对象会引用老年代的对象；
  * 而在重新标记时，会对整个堆进行扫描，在对新生代做可达性分析时，若新生代对象过多，会对性能造成影响；
  * 使用上述参数**先对新生代做一个GC**，则在重新标记时扫描的对象会减少，可以提升性能。

![响应时间优先](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622943.PNG)

**对上图进行说明：即CMS的流程**

多线程运行，老年代发生了内存不足，多个线程到达安全点停止运行；

CMS 垃圾回收器开始工作，执行一个**初始标记**的动作(只标记根对象，很快)，**期间需要STW**；

用户线程可以恢复运行，与此同时垃圾回收线程**并发标记**，找出剩余的垃圾对象；

接下来进行**重新标记，需要STW**，因为之前并发执行时，会对垃圾回收的对象造成干扰，因此要做重新标记工作；

重新标记完成后，用户线程恢复运行，垃圾回收线程做一次**并发的清理**。

### 4.4 G1 - (Garbage One)

定义：Garbage First

* 2004 论文发布
* 2009 JDK 6u14 体验
* 2012 JDK 7u4 官方支持
* 2017 JDK 9 默认

> JDK9 废除了之前的 CMS垃圾回收器

适用场景

* 同时注重吞吐量（Throughput）和低延迟（Low latency），默认的暂停目标是 200 ms
* 超大堆内存，会将堆划分为多个大小相等的 Region
* 整体上是 **标记+整理** 算法，两个区域Region之间是 **复制** 算法 

相关 JVM 参数

* `-XX:+UseG1GC`：JDK1.8中需要手动打开，JDK1.9之后为默认
* `-XX:G1HeapRegionSize=size`：设置 Region 的大小，只能设置为1,2,4,8,16...等大小
* `-XX:MaxGCPauseMillis=time` ：设置暂停目标

#### 1) G1 垃圾回收阶段

![垃圾回收阶段](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622308.PNG)

阶段1：Yong Collection 新生代垃圾收集

阶段2：Yong Collection+Concurrent Mark 新生代垃圾收集同时做一些并发标记

阶段3：Mixed Collection 混合收集，对新生代和老年代都进行一次大规模收集

#### 2) Young Collection 

下面的格子表示一个个的 Region

新生代垃圾收集时会 STW

![yongcollection1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622886.PNG)

内存紧张时，会将伊甸园e中的内容拷贝到幸存区s中

![yongcollection2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622494.PNG)

幸存区中的对象晋升到老年代o中

![yongcollection3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622930.PNG)



#### 3) Young Collection + CM 

在 Young GC 时会进行 GC Root 的初始标记 

老年代占用堆空间比例达到阈值时，进行并发标记（不会 STW），由下面的 JVM 参数决定

`-XX:InitiatingHeapOccupancyPerent=percent`（默认45%） 

![yongcollection+CM](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622549.PNG)

#### 4) Mixed Collection 

会对 E、S、O 进行全面垃圾回收

* 最终标记（Remark）会 STW
* 拷贝存活（Evacuation）会 STW

`-XX:MaxGCPauseMillis=ms`

会根据设置暂停目标时间，有选择性的回收回收价值最高的区域；

解释了Garbage First，即回**收那些垃圾最多的老年代区域**，从而达到暂定时间的目标

![MixedCollection](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622056.PNG)

#### 5) Full GC

* SerialGC
  * 新生代内存不足发生的垃圾收集 - Minor GC
  * 老年代内存不足发生的垃圾收集 - Full GC
* ParallelGC
  * 新生代内存不足发生的垃圾收集 - Minor GC
  * 老年代内存不足发生的垃圾收集 - Full GC
* CMS
  * 新生代内存不足发生的垃圾收集 - Minor GC
  * 老年代内存不足，只有当并发清理速度跟不上产生垃圾速度等情况时，才会触发Full GC
* G1
  * 新生代内存不足发生的垃圾收集 - Minor GC
  * 老年代内存不足 
    1. 老年代占用堆空间比例达到阈值时，进行并发标记
    
    2. Mixed Collection 混合收集
    
    3. 当并发垃圾回收的速度无法跟上产生垃圾的速度是，并发收集失败，退化为串行收集，即为Full GC

#### 6) Young Collection 跨代引用 

新生代回收的跨代引用（老年代引用新生代）问题 

![yongcollection跨带引用](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622952.PNG)

* 卡表（将老年代的区域再进行细分为card）与 Remembered Set（记录对于哪一个dirty card）
* 在引用变更时通过 post-write barrier + dirty card queue
* concurrent refinement threads 更新 Remembered Set 

![yongcollection跨带引用2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622586.PNG)

#### 7) Remark 

`pre-write barrier + satb_mark_queue` 

写屏障+队列

![remark](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622970.PNG)

> 黑色：已经处理完成，被引用，在结束时会存活下来的对象；
>
> 灰色：正在处理当中的，被引用了，处理完成后变黑；
>
> 白色：尚未处理的，下方的会被处理变灰再变黑；上面的被处理后还是白色，由于没有被引用被删除

**例子：**

1. 初始状态，正在处理B：

   ![remark1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622033.PNG)

2. 在处理B时，由于并发，C和B之间的连接断开：

   ![remark2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251622902.PNG)

3. 处理完成后C仍然是白色，因此会被垃圾处理掉；

4. 但是在CB处理完成后，用户又修改了C的引用地址，如下，此时C不应该再被垃圾处理掉了；

   ![remark3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623077.PNG)
   
5. 因此需要对对象的引用做进一步的检查，即remark阶段就是为了防止上述现象发生，做法如下：

   1. 当对象的引用发生改变时，JVM就会为其加入一个写屏障（弹幕：**不是线程的那个，这个应该理解为通知**）

      ![remark4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623126.PNG)

   2. 当对象的应用发生改变，写屏障的代码就会被执行

      ![remark5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623475.PNG)

   3. 会把C对象加入一个队列当中，将其变灰，表示还没有处理完

      ![remark6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623118.PNG)
   
   4. 当并发标记结束后，进入重新标记阶段，重新标记阶段会STW，此时处理队列中的对象，再做一次检查，这样C就不会被误当成对象被回收了
   
      ![remark7](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623442.PNG)

#### 8) JDK 8u20 字符串去重 

优点：节省大量内存 

缺点：略微多占用了 cpu 时间，新生代回收时间略微增加

`-XX:+UseStringDeduplication`，默认打开的

```java
String s1 = new String("hello"); // char[]{'h','e','l','l','o'}
String s2 = new String("hello"); // char[]{'h','e','l','l','o'}
```

将所有新分配的字符串放入一个队列

当新生代回收时，G1并发检查是否有字符串重复

如果它们值一样，让它们引用同一个 `char[]`

注意，与 `String.intern()` 不一样 

* `String.intern()` 关注的是字符串对象 
* 而字符串去重关注的是 `char[] `
* 在 JVM 内部，使用了不同的字符串表 

#### 9) JDK 8u40 并发标记类卸载 

所有对象都经过并发标记后，就能知道哪些类不再被使用，当一个类加载器的所有类都不再使用，则卸载它所加载的所有类 

`-XX:+ClassUnloadingWithConcurrentMark` 默认启用 

#### 10) JDK 8u60 回收巨型对象 

* 一个对象大于 region 的一半时，称之为巨型对象，如下方的粉色块

  ![巨型对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623140.PNG)

* G1 不会对巨型对象进行拷贝

* 回收时被优先考虑

* G1 会跟踪老年代所有 incoming 引用，这样老年代 incoming 引用为0 的巨型对象就可以在新生代垃圾回收时处理掉

  ![巨型对象2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623871.PNG)

  如上图的第一个粉色H，卡表中已经没有对其进行引用了，这个巨型对象可以在新生代垃圾回收时处理掉

#### 11) JDK 9 并发标记起始时间的调整 

* 并发标记必须在堆空间占满前完成，否则退化为 FullGC
* JDK 9 之前需要使用 `-XX:InitiatingHeapOccupancyPercent`
* JDK 9 可以动态调整
  * `-XX:InitiatingHeapOccupancyPercent` 用来设置初始值
  * 进行数据采样并动态调整
  * 总会添加一个安全的空档空间 

#### 12) JDK 9 更高效的回收 

* 250+增强
* 180+bug修复 

* https://docs.oracle.com/en/java/javase/12/gctuning 

## 5. 垃圾回收调优

预备知识

* 掌握 GC 相关的 JVM 参数，会基本的空间调整
* 掌握相关工具
* 明白一点：调优跟应用、环境有关，没有放之四海而皆准的法则 

```java
/*
查看虚拟机运行参数
C:\Users\ZhuCC>"C:\Software\Java\jdk1.8.0_241\bin\java" -XX:+PrintFlagsFinal -version | findstr "GC"
 */
public class Demo2_8 {
}
```

输出与垃圾回收有关的JVM参数

### 5.1 调优领域 

* 内存
* 锁竞争
* cpu 占用
* io

### 5.2 确定目标 

* 【低延迟】还是【高吞吐量】，选择合适的回收器
* 低延迟：CMS，G1，ZGC
* 高吞吐量：ParallelGC
* Zing 

### 5.3 最快的 GC 

**最快的 GC 就是不发生 GC...**

查看 FullGC 前后的内存占用，考虑下面几个问题

* 数据是不是太多？
  * `resultSet = statement.executeQuery("select * from 大表 limit n");`
* 数据表示是否太臃肿？
  * 对象图
  * 对象大小：
    * 一个Object最小也要占用16byte 
    * Integer包装类型，一个对象头占用16个字节，4个字节的值，加上对其，总共占用24个字节
    * int只需要占用4个字节
* 是否存在内存泄漏？
  * `static Map map = `，频繁向里面添加对象
  * 软/弱引用--->在内存耗尽时会被GC
  * 第三方缓存实现，Redis...

### 5.4 新生代调优

新生代的特点：

* 所有的 new 操作的内存分配非常廉价
  * TLAB thread-local allocation buffer，每个线程在伊甸园中分配的一块线程局部私有缓冲区域
  * 在分配时，首先检查TLAB中是否有可用内存，优先在此进行对象分配
  * 考虑到多线程在分配时的线程安全问题
* 死亡对象的回收代价是零
* 大部分对象用过即死
* Minor GC 的时间远远低于 Full GC

**新生代调优：**

* 新生代是不是越大越好？

  -Xmn Sets the initial and maximum size (in bytes) of the heap for the young generation (nursery). GC is performed in this region more often than in other regions. If the size for the young generation is too small, then a lot of minor garbage collections are performed. If the size is too large, then only full garbage collections are performed, which can take a long time to complete. Oracle recommends that you keep the size for the young generation greater than 25% and less than 50% of the overall heap size. 

  翻译：

  -Xmn 设置了堆中新生代的初始大小和最大大小（以字节为单位）。GC在这个区域比在其他区域更频繁地执行。如果新生代的大小太小，则会执行大量小的垃圾收集。如果太大，则会Full GC，这可能需要很长时间才能完成。Oracle建议将新生代的设置为堆总尺寸的25%~50%之间。

  > 补充：
  >
  > 总的原则还是将新生代的大小调的尽可能大；
  >
  > 考虑到新生代垃圾回收采用的复制算法，复制算法的两个阶段：1.标记；2.复制（耗费更多时间，设计到对象的移动，对象地址的更新）；
  >
  > 而新生代中绝大部分对象都是“朝生夕死”的，只有少量的对象能存活，因此复制所占用的时间也相对较短，而标记占占用时间更少；
  >
  > 因此即使新生代占用空间很大，也不会造成明显的性能下降。

* 新生代大小的理想情况：新生代能容纳所有【并发量 * (一次响应和请求过程中所产生的对象)】的数据；

* 幸存区大小设置：幸存区大到能保留【当前活跃对象+需要晋升对象】 

* 晋升阈值配置得当，让长时间存活对象尽快晋升 

  * `-XX:MaxTenuringThreshold=threshold`：调整最大晋升阈值

  * `-XX:+PrintTenuringDistribution` ：打印晋升的详细信息

    ```java
    Desired survivor size 48286924 bytes, new threshold 10 (max 10)
    //  年龄         占用空间
    - age 1: 28992024 bytes, 28992024 total
    - age 2: 1366864 bytes, 30358888 total
    - age 3: 1425912 bytes, 31784800 total
    ...
    ```

### 5.5 老年代调优 

以 CMS 为例：

* CMS 的老年代内存越大越好
  * CMS垃圾回收器是一个并发的垃圾回收器，在垃圾回收器工作同时，用户的其他线程也可以运行
  * 但是在用户的其他线程运行期间会产生新的浮动垃圾，若此时又造成了内存不足，会导致CMS并发失败
  * CMS并发失败，则退回到SerialOld串行垃圾回收器，导致STW，运行效率下降明显
  * 因此老年代内存越大越好
* 先尝试不做调优，如果没有 Full GC，那系统其实工作的还行，即使发生了Full GC 也应该先尝试调优新生代
* 观察发生 Full GC 时老年代内存占用，将老年代内存预设调大 1/4 ~ 1/3 
  * `-XX:CMSInitiatingOccupancyFraction=percent`：控制老年代内存占用达到percent比例时，启用CMS垃圾回收

### 5.6 案例 

#### 案例1

Full GC 和 Minor GC频繁

分析：GC频繁说明空间紧张

1. 若新生代空间紧张，很快被新创建的对象占满，Minor GC频繁发生，导致幸存区对象的晋升阈值降低，一些生命周期较短的对象也晋升到了老年代中；
2. 幸存区对象晋升阈值降低，会导致老年代中的空间紧张，导致Full GC频繁发生；

解决：一般从先调整新生代开始，增大新生代内存；

#### 案例2

请求高峰期发生 Full GC，单次暂停时间特别长 （CMS）

分析：

1. CMS垃圾回收分为：1.初始标记；2.并发标记；3.重新标记；4.并发清理

2. 其中重新标记占用最长时间
3. 在重新标记阶段，会扫描整个堆内存（新生代+老年代）
4. 若此时新生代对象个数较多，会使标记时间增大

解决：在重新标记之前，先对新生代中的对象做一次清理，即通过`-XX:+CMSScavengeBeforeRemark`参数，则在重新标记阶段，需要重新标记的对象大大减少，能解决单次暂停时间特别长的问题

#### 案例3 

老年代充裕情况下，发生 Full GC （CMS jdk1.7） 

分析：

1. JDK1.7 之前通过永久代作为方法区的实现；

2. 永久代中空间不足也会造成Full GC的发生；

   > JDK1.8之后修改为元空间，使用的是系统空间，其垃圾回收不由Java控制

解决：通过相应参数，增大永久代的空间大小
