# JVM：内存溢出OOM

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 经典错误

JVM 中常见的两个 OOM 错误

**StackoverflowError**：栈溢出

**OutofMemoryError**：java heap space：堆溢出

除此之外，还有以下的错误

- java.lang.StackOverflowError
- java.lang.OutOfMemoryError：java heap space
- java.lang.OutOfMemoryError：GC overhead limit exceeeded
- java.lang.OutOfMemoryError：Direct buffer memory
- java.lang.OutOfMemoryError：unable to create new native thread
- java.lang.OutOfMemoryError：Metaspace

## 架构

OOM 都属于 Error，而不是 Exception

![throwable架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633355.png)

## StackOverflowError

堆栈溢出，我们用最简单的一个递归调用，就会造成堆栈溢出，也就是深度的方法调用

栈一般是512K~1024K，不断的深度调用，直到栈被撑破

> `-Xss`：设计单个线程栈的大小，一般默认为512K~1024K，等价于 `-XX:ThreadStackSize`
>
> > 进一步说明：
> >
> > - 使用 `jinfo -flag ThreadStackSize` 会发现 `-XX:ThreadStackSize=0`
> > - 这个值的大小是取决于平台的
> >   - Linux/x64：1024KB
> >   - OS X：1024KB
> >   - Oracle Solaris：1024KB
> >   - Windows：取决于虚拟内存的大小

```java
public class StackOverflowErrorDemo {

    private static Integer i=0;

    public static void main(String[] args) {
        stackOverflowError();
    }
    /**
     * 栈一般是512K，不断的深度调用，直到栈被撑破
     * Exception in thread "main" java.lang.StackOverflowError
     */
    private static void stackOverflowError() {
        System.out.println(i++);
        stackOverflowError();
    }
}
```

运行结果

```bash
10461
Exception in thread "main" java.lang.StackOverflowError
```

## OutOfMemoryError

### java heap space

堆内存溢出：创建了很多对象，导致堆空间不够存储

```java
public class JavaHeapSpaceDemo {
    public static void main(String[] args) {
        // JVM参数调整堆空间的大小 -Xms10m -Xmx10m
        // 创建一个 80M的字节数组
        byte [] bytes = new byte[80 * 1024 * 1024];
    }
}
```

我们创建一个80M的数组，会直接出现 Java heap space

```bash
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at JavaHeapSpaceDemo.main(JavaHeapSpaceDemo.java:5)
```

### GC overhead limit exceeded

GC回收时间过长时会抛出 OutOfMemoryError，过长的定义是：**超过了98%的时间用来做 GC，并且回收了不到2% 的堆内存。**

连续多次GC都只回收了不到2%的极端情况下，才会抛出该异常。假设不抛出 GC overhead limit 错误会造成什么情况呢？那就是GC清理的这点内存很快会再次被填满，迫使GC再次执行，这样就形成了恶性循环，CPU的使用率一直都是100%，而GC却没有任何成果。

![GCOverheadLimit](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633006.png)

**代码演示：**

为了更快的达到效果，我们首先需要设置JVM启动参数

```bash
# 堆内存大小      打印出GC回收情况	    最大直接内存大小
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
```

这个异常出现的步骤就是，我们不断的像list中插入String对象，直到启动GC回收

```java
/**
 * GC 回收超时
 * JVM参数配置: -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
 */
public class GCOverheadLimitDemo {
    public static void main(String[] args) {
        int i = 0;
        List<String> list = new ArrayList<>();
        try {
            while(true) {
                list.add(String.valueOf(++i).intern());
            }
        } catch (Exception e) {
            System.out.println("***************i:" + i);
            e.printStackTrace();
            throw e;
        } finally {

        }
    }
}
```

运行结果：

```bash
[GC (Allocation Failure) [PSYoungGen: 2048K->500K(2560K)] 2048K->972K(9728K), 0.0034656 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2548K->505K(2560K)] 3020K->2737K(9728K), 0.0028468 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2553K->485K(2560K)] 4785K->4613K(9728K), 0.0026048 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2533K->496K(2560K)] 6661K->6368K(9728K), 0.0035775 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 496K->0K(2560K)] [ParOldGen: 5872K->6302K(7168K)] 6368K->6302K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0562960 secs] [Times: user=0.11 sys=0.02, real=0.06 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2048K->898K(2560K)] [ParOldGen: 6302K->7030K(7168K)] 8350K->7928K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0414180 secs] [Times: user=0.08 sys=0.00, real=0.04 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2048K->2006K(2560K)] [ParOldGen: 7030K->7030K(7168K)] 9078K->9036K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0385825 secs] [Times: user=0.08 sys=0.00, real=0.04 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2048K->2047K(2560K)] [ParOldGen: 7030K->7030K(7168K)] 9078K->9078K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0277052 secs] [Times: user=0.11 sys=0.00, real=0.03 secs] 
...很多个Full GC...
[Full GC (Ergonomics) [PSYoungGen: 2047K->2047K(2560K)] [ParOldGen: 7073K->7073K(7168K)] 9121K->9121K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0355338 secs] [Times: user=0.06 sys=0.00, real=0.04 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2047K->2047K(2560K)] [ParOldGen: 7075K->7075K(7168K)] 9123K->9123K(9728K), [Metaspace: 3234K->3234K(1056768K)], 0.0323376 secs] [Times: user=0.05 sys=0.00, real=0.03 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2047K->0K(2560K)] [ParOldGen: 7093K->603K(7168K)] 9141K->603K(9728K), [Metaspace: 3259K->3259K(1056768K)], 0.0064373 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 2560K, used 76K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 3% used [0x00000000ffd00000,0x00000000ffd13250,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 603K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 8% used [0x00000000ff600000,0x00000000ff696f20,0x00000000ffd00000)
 Metaspace       used 3267K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K

Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
```

我们能够看到多次Full GC，并没有清理出空间，在多次执行GC操作后，就抛出异常 `java.lang.OutOfMemoryError: GC overhead limit exceeded`

### Direct buffer memory

NIO：主要是由于NIO引起的

写 NIO 程序的时候经常会使用 ByteBuffer 来读取或写入数据，这是一种**基于通道 (Channel) 与缓冲区 (Buffer) 的 I/O 方式**，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆里面的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据。

`ByteBuffer.allocate(capability)`：第一种方式是**分配 JVM 堆内存**，属于 GC 管辖范围，由于需要拷贝所以速度相对较慢；

`ByteBuffer.allocteDirect(capability)`：第二种方式是**分配 OS 本地内存**，不属于 GC 管辖范围，由于不需要内存的拷贝，所以速度相对较快

但如果不断分配本地内存，堆内存很少使用，那么 JVM 就不需要执行 GC，DirectByteBuffer 对象就不会被回收，这时候会出现堆内存充足，但本地内存可能已经使用光了的情况，当再次尝试分配本地内存就会出现`OutOfMemoryError:Direct buffer memory`，那么程序就奔溃了。

**一句话概括：本地内存不足，但是堆内存充足的时候，就会出现这个问题。**

我们使用 `-XX:MaxDirectMemorySize=5m` 配置能使用的堆外物理内存为5M

```bash
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
```

然后我们申请一个6M的空间

```java
// 只设置了5M的物理内存使用，但是却分配6M的空间
ByteBuffer bb = ByteBuffer.allocateDirect(6 * 1024 * 1024);
```

这个时候，运行就会出现问题了，如下：

```bash
[GC (System.gc()) [PSYoungGen: 1657K->504K(2560K)] 1657K->672K(9728K), 0.0008168 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 504K->0K(2560K)] [ParOldGen: 168K->623K(7168K)] 672K->623K(9728K), [Metaspace: 3227K->3227K(1056768K)], 0.0033185 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 2560K, used 98K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 4% used [0x00000000ffd00000,0x00000000ffd18aa0,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 7168K, used 623K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 8% used [0x00000000ff600000,0x00000000ff69bed8,0x00000000ffd00000)
 Metaspace       used 3260K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
```

### unable to create new native thread

不能够创建更多的新的线程了，也就是说创建线程的上限达到了

在高并发场景的时候，会应用到；当高并发请求服务器时，经常会出现如下异常`java.lang.OutOfMemoryError:unable to create new native thread`，准确说该 native thread 异常与对应的平台有关

**导致原因：**

- 应用创建了太多线程，一个应用进程创建的线程数目，超过系统承载极限
- 服务器并不允许你的应用程序创建这么多线程，linux系统默认：运行单个进程可以创建的线程为1024个，如果应用创建超过这个数量，就会报 `java.lang.OutOfMemoryError:unable to create new native thread`

**解决方法：**

1. 想办法降低你应用程序创建线程的数量，分析应用是否真的需要创建这么多线程，如果不是，改代码将线程数降到最低
2. 对于有的应用，确实需要创建很多线程，远超过linux系统默认1024个线程限制，可以通过修改linux服务器配置，扩大linux默认限制

```java
/**
 * 无法创建更多的线程
 */
public class UnableCreateNewThreadDemo {
    public static void main(String[] args) {
        for (int i = 0; ; i++) {  // 死循环
            System.out.println("************** i = " + i);
            new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

这个时候，就会出现下列的错误，线程数大概在 900 多个

> 在windows下TM的创建了100060才报错！

```bash
Exception in thread "main" java.lang.OutOfMemoryError: unable to cerate new native thread
```

> * 如何查看线程数：`ulimit -u`
>
> * 同一个线程不能进行两次start，会出现：`Exception in thread "main" java.lang.IllegalThreadStateException`的异常，可进入源码进行分析一波

### Metaspace

元空间内存不足，Matespace 元空间应用的是本地内存

`-XX:MetaspaceSize` 的默认的大小约为20M

**元空间是什么**：

元空间(实现)就是我们的方法区(概念)，存放的是类模板，类信息，常量池等

Metaspace 是方法区 HotSpot 中的实现，它与持久代(JDK7)最大的区别在于：**Metaspace 并不在虚拟内存中，而是使用本地内存**，也即在 java8 中，class metadata（the virtual machines internal presentation of Java class），被存储在叫做 Matespace 的 native memory

永久代（java8 后被元空间 Metaspace 取代了）存放了以下信息：

- 虚拟机加载的类信息
- 常量池（*不是在堆中吗？*）
- 静态变量（*类的静态变量还是在堆中的吧*）
- 及时编译后的代码

模拟 Metaspace 空间溢出，我们不断生成类往元空间里灌输，类占据的空间总会超过Metaspace指定的空间大小

**代码**：

在模拟异常生成时候，因为初始化的元空间为20M，因此我们使用 JVM 参数调整元空间的大小，为了更好的效果

```bash
-XX:MetaspaceSize=8m -XX:MaxMetaspaceSize=8m
```

**代码如下**：

```java
/**
 * 元空间溢出
 */
public class MetaspaceOutOfMemoryDemo {

    // 静态类
    static class OOMTest {

    }
    public static void main(final String[] args) {
        // 模拟计数多少次以后发生异常
        int i =0;
        try {
            while (true) {
                i++;
                // 使用Spring的动态字节码技术
                Enhancer enhancer = new Enhancer();
                enhancer.setSuperclass(OOMTest.class);
                enhancer.setUseCache(false);
                enhancer.setCallback(new MethodInterceptor() {
                    @Override
                    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                        return methodProxy.invokeSuper(o, args);
                    }
                });
            }
        } catch (Exception e) {
            System.out.println("发生异常的次数:" + i);
            e.printStackTrace();
        } finally {

        }
    }
}
```

会出现以下错误：

```bash
发生异常的次数: 201
java.lang.OutOfMemoryError:Metaspace
```