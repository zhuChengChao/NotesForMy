# JVM：内存结构

> 在学习了 bilibili 上 **[黑马程序员]()** 的课程 [JVM完整教程](https://www.bilibili.com/video/BV1yE411Z7AP) 后做的学习笔记，总共分为5篇笔记，本文为 1/5 篇。

### 内容

1. 程序计数器
2. 虚拟机栈
3. 本地方法栈
4. 堆
5. 方法区
6. 直接内存

## 1. 程序计数器

![程序计数器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624869.PNG)

### 1.1 定义 

Program Counter Register 程序计数器（寄存器）

* 作用：是记住下一条 JVM 指令的执行地址
* 特点
  * 是线程私有的
  * 不会存在内存溢出 

### 1.2 作用 

`JVM指令 -> 解释器 -> 机器码 -> CPU执行`

```java
// 二进制字节码 JVM指令        // java源代码
0: getstatic #20 			// PrintStream out = System.out;
3: astore_1 				// --
4: aload_1 					// out.println(1);
5: iconst_1					// --
6: invokevirtual #26		// --
9: aload_1 					// out.println(2);
10: iconst_2 				// --
11: invokevirtual #26 		// --
14: aload_1 				// out.println(3);
15: iconst_3 				// --
16: invokevirtual #26 		// --
19: aload_1 				// out.println(4);
20: iconst_4 				// --
21: invokevirtual #26 		// --
24: aload_1 				// out.println(5);
25: iconst_5 				// --
26: invokevirtual #26 		// --
29: return
```

## 2. 虚拟机栈 

![虚拟机栈](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624499.PNG)

### 2.1 定义 

Java Virtual Machine Stacks，Java 虚拟机栈 

* 每个线程运行时所需要的内存，称为虚拟机栈；

* 每个线程由多个**栈帧（Frame）**组成，对应着每次方法调用时所占用的内存；
* 每个线程**只能有一个活动栈帧**，对应着当前正在执行的那个方法；
* 栈帧：每个方法运行时需要的内存，例如方法的参数，局部变量，返回地址等都需要占用内存；

**问题辨析**

1. 垃圾回收是否涉及栈内存？

   答：不会，栈内的栈帧内存，在方法调用结束后都会出栈；**垃圾回收只是涉及到堆内存**。

2. 栈内存分配越大越好吗？

   答：不是，栈内存划分越大会导致线程数减少，由于物理内存大小是一定的，线程会使用到栈内存，因此栈内存越大则会可运行线程数减少。

3. 方法内的局部变量是否线程安全？

   * 如果方法内局部变量没有逃离方法的作用访问，它是线程安全的
   * 如果是局部变量引用了对象，并逃离方法的作用范围，需要考虑线程安全 

### 2.2 栈内存溢出 

* 栈帧过多导致栈内存溢出
* 栈帧过大导致栈内存溢出 

#### 案例：递归爆栈

```java
private static int count;

public static void main(String[] args) {
    try {
        method1();
    }catch (Throwable e){
        e.printStackTrace();
        System.out.println(count);
    }
}

// 一直入递归函数，没有出递归函数的条件
private static void method1(){
    count++;
    method1();
}
```

输出结果：

```java
java.lang.StackOverflowError
	at cn.xyc.Test01.method1(Test01.java:19)
	at cn.xyc.Test01.method1(Test01.java:19)
	at cn.xyc.Test01.method1(Test01.java:19)
	...
23564
```

**修改栈内存大小：**

> 在 IDEA 中打开程序的运行设置 `Edit Configurations...`
>
> 在`VM options`加上修改栈内存大小的参数`-Xss256k`，

重新运行代码，输出如下：

```java
java.lang.StackOverflowError
	at cn.xyc.Test01.method1(Test01.java:19)
	at cn.xyc.Test01.method1(Test01.java:19)
	at cn.xyc.Test01.method1(Test01.java:19)
	...
4053  // 栈变小了
```

#### 案例：json 数据转换

```java
public class Test02 {

    public static void main(String[] args) throws JsonProcessingException {
        Dept d = new Dept();
        d.setName("Market");

        Emp e1 = new Emp();
        e1.setName("zhang");
        e1.setDept(d);

        Emp e2 = new Emp();
        e2.setName("li");
        e2.setDept(d);

        d.setEmps(Arrays.asList(e1, e2));

        // { name: 'Market', emps: [{ name:'zhang', dept:{ name:'', emps: [ {}]} },] }
        ObjectMapper mapper = new ObjectMapper();
        System.out.println(mapper.writeValueAsString(d));
    }
}

@Data
class Emp {
    private String name;
    private Dept dept;
}

@Data
class Dept {
    private String name;
    private List<Emp> emps;
}
```

输出报错：

```java
Infinite recursion (StackOverflowError)
```

修改：

```java
// 在emp类中的成员属性Dept上加上
@JsonIgnore
private Dept dept;
```

修改完成后输出：

```java
{"name":"Market","emps":[{"name":"zhang"},{"name":"li"}]}
```

### 2.3 线程运行诊断 

#### 案例1：CPU占用过多

```java
/**
 * 演示 cpu 占用过高
 */
public class Demo1_16 {

    public static void main(String[] args) {
        new Thread(null, () -> {
            System.out.println("1...");
            while(true) { }  // 死循环，导致cpu一直空转
        }, "thread1").start();


        new Thread(null, () -> {
            System.out.println("2...");
            try {
                Thread.sleep(1000000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread2").start();

        new Thread(null, () -> {
            System.out.println("3...");
            try {
                Thread.sleep(1000000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread3").start();
    }
}
```

**定位：**

* 用 top 定位哪个进程对 cpu 的占用过高；

* `ps H -eo pid, tid, %cpu | grep 进程id` （用 ps 命令进一步定位是那个线程引起的cpu占用过高） 

  > pid：进程 ID
  >
  > tid：线程 ID
  >
  > %cpu：cpu占用情况

* jstack 进程 id

  * jps 查看所有 java 进程
  * `jstack <PID>` 查看某个 Java 进程（PID）的所有线程状态
  * 然后可以根据 **线程id-tid** 找到有问题的线程，进一步定位到问题代码的源码行号

#### 案例2：程序运行很长时间没有结果 

```java
/**
 * 演示线程死锁
 */
class A{};
class B{};
public class Demo1_3 {
    static A a = new A();
    static B b = new B();
    
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            synchronized (a) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (b) {
                    System.out.println("我获得了 a 和 b");
                }
            }
        }).start();
        Thread.sleep(1000);
        new Thread(()->{
            synchronized (b) {
                synchronized (a) {
                    System.out.println("我获得了 a 和 b");
                }
            }
        }).start();
    }
}
```

* 通过 jps 查看当前的 java 进程

* `jstack <PID>` 查看某个 Java 进程（PID）的所有线程状态

  ```java
  Found one Java-level deadlock:
  =============================
  "Thread-1":
    waiting to lock monitor 0x000000001c3edbd8 (object 0x000000076b2aa738, a cn.xyc.A),
    which is held by "Thread-0"
  "Thread-0":
    waiting to lock monitor 0x000000001c3f0728 (object 0x000000076b2ac158, a cn.xyc.B),
    which is held by "Thread-1"
  
  Java stack information for the threads listed above:
  ===================================================
  "Thread-1":
          at cn.xyc.Test4.lambda$main$1(Test4.java:24)
          - waiting to lock <0x000000076b2aa738> (a cn.xyc.A)
          - locked <0x000000076b2ac158> (a cn.xyc.B)
          at cn.xyc.Test4$$Lambda$2/2093631819.run(Unknown Source)
          at java.lang.Thread.run(Thread.java:748)
  "Thread-0":
          at cn.xyc.Test4.lambda$main$0(Test4.java:16)
          - waiting to lock <0x000000076b2ac158> (a cn.xyc.B)
          - locked <0x000000076b2aa738> (a cn.xyc.A)
          at cn.xyc.Test4$$Lambda$1/1023892928.run(Unknown Source)
          at java.lang.Thread.run(Thread.java:748)
  
  Found 1 deadlock.
  ```

  可以看到出现了死锁，Thread-1 需要的锁被 Thread-0 持有， Thread-0 需要的锁被 Thread-1 持有，出现死锁的情况。

## 3. 本地方法栈 

![本地方法栈](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625046.PNG)

**本地方法**：不是由 Java 代码编写的方法，而由 C/C++ 编写的，与底层操作系统进行交互的方法。

例如：

```java
// Object类中，这些带native的方法
protected native Object clone() throws CloneNotSupportedException;
public native int hashCode();
public final native void notify();
// ...
```

## 4. 堆 

![堆](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625394.PNG)

### 4.1 定义 

Heap 堆

* 通过 new 关键字，创建对象都会使用堆内存

特点

* 它是线程共享的，堆中对象都需要考虑线程安全的问题
* 有垃圾回收机制 

### 4.2 堆内存溢出 

**演示：**

```java
/**
 * 演示堆内存溢出 java.lang.OutOfMemoryError: Java heap space
 * -Xmx8m  控制堆空间的最大空间值
 */
public class Demo1_5 {

    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "hello";
            while (true) {
                list.add(a); // hello, hellohello, hellohellohellohello ...
                a = a + a;  // hellohellohellohello
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println(i);
        }
    }
}
```

输出：

```java
java.lang.OutOfMemoryError: Java heap space
26
```

### 4.3 堆内存诊断 

1. `jps` 工具：查看当前系统中有哪些 java 进程
2. `jmap` 工具：查看堆内存占用情况 `jmap -heap 进程id`，只能查看一个时刻
3. `jconsole` 工具：图形界面的，多功能的监测工具，可以连续监测 

#### 案例1：工具使用演示

```java
/**
 * 演示堆内存
 */
public class Demo1_4 {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("1...");
        Thread.sleep(30000);
        byte[] array = new byte[1024 * 1024 * 10]; // 10 Mb
        System.out.println("2...");
        Thread.sleep(20000);
        array = null;
        System.gc();  // 调用gc方法进行垃圾回收
        System.out.println("3...");
        Thread.sleep(1000000L);
    }
}
```

##### jmap 工具使用

1）运行上述程序

2）输入 jps 查看当前 java 进程

```j
7760
13380  Jps
10776  RemoteMavenServer
6556   Test6
13864  Launcher
```

3）不同时刻使用 jmap 命令

3.1）输出1时的时间点快照：

```java
C:\Users\ZhuCC>jmap -heap 6556
Attaching to process ID 6556, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.241-b07

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:   // 堆配置信息
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4278190080 (4080.0MB)  // 最大堆内存
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 1426063360 (1360.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:  		 // 堆内存占用
PS Young Generation  
Eden Space:
   capacity = 67108864 (64.0MB)
   used     = 5370928 (5.1221160888671875MB)  // 对比下面
   free     = 61737936 (58.87788391113281MB)
   8.00330638885498% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 179306496 (171.0MB)
   used     = 0 (0.0MB)
   free     = 179306496 (171.0MB)
   0.0% used

1759 interned Strings occupying 157688 bytes.
```

3.2）输出2时的时间点快照：这时`byte[] array`已经被创建了

```java
C:\Users\ZhuCC>jmap -heap 6556
Attaching to process ID 6556, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.241-b07

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4278190080 (4080.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 1426063360 (1360.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 67108864 (64.0MB)
   // 对比上面，这里的used已经增加了10Mb空间了，这就是新创建的byte数组所占用的空间
   used     = 15856704 (15.12213134765625MB)  
   free     = 51252160 (48.87786865234375MB)
   23.62833023071289% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 179306496 (171.0MB)
   used     = 0 (0.0MB)
   free     = 179306496 (171.0MB)
   0.0% used

1760 interned Strings occupying 157736 bytes.
```

3.3）输出3时的时间点快照：这时`byte[] array`已经被垃圾回收了

```java
C:\Users\ZhuCC>jmap -heap 6556
Attaching to process ID 6556, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.241-b07

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4278190080 (4080.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 1426063360 (1360.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 67108864 (64.0MB)
   // byte数组已经被释放了，进行了垃圾回收，使用量又变小了
   used     = 1342200 (1.2800216674804688MB)  
   free     = 65766664 (62.71997833251953MB)
   2.0000338554382324% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 179306496 (171.0MB)
   used     = 665560 (0.6347274780273438MB)
   free     = 178640936 (170.36527252197266MB)
   0.3711856596651133% used

1746 interned Strings occupying 156744 bytes.
```

##### jconsole 工具使用

1. 演示程序运行；

2. 控制台输入 jconsole ，选择相应 java 进程，选择不安全连接

3. 监视情况如下：

   ![jconsole](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625112.PNG)

#### 案例2：

垃圾回收后，内存占用仍然很高

```java
/**
 * 演示查看对象个数 堆转储 dump
 */
public class Demo1_13 {
    public static void main(String[] args) throws InterruptedException {
        List<Student> students = new ArrayList<>();
        for (int i = 0; i < 200; i++) {
            students.add(new Student());
        }
        Thread.sleep(1000000000L);
    }
}
class Student {
    private byte[] big = new byte[1024*1024];  // 1Mb
}
```

通过 `jmap -heap <PID>` 查看

```java
C:\Users\ZhuCC>jmap -heap 15116
Attaching to process ID 15116, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.241-b07

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4278190080 (4080.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 1426063360 (1360.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 241172480 (230.0MB)
   used     = 28806776 (27.47228240966797MB)
   free     = 212365704 (202.52771759033203MB)
   11.944470612899117% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 10616992 (10.125152587890625MB)
   free     = 393056 (0.374847412109375MB)
   96.43002464657738% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 280494080 (267.5MB)
   // 老年代中占用了很高的内存使用量
   used     = 186248584 (177.62049102783203MB)  
   free     = 94245496 (89.87950897216797MB)
   66.40018356180637% used

1743 interned Strings occupying 156600 bytes.
```

##### jvisualvm 工具

> j + visual + vm

控制台输入命令：`jvisualvm`

选择对于的 java 进程

`监视栏`中的 `堆Dump`可以抓取堆的快照
1. 其中有功能可以查找 `堆中最大的对象`
2. 可以看到`ArrayList`对象占用的内存最大，点击进入查看详细信息
3. 又可以看到`ArrayList`中的`elementData`存储了 `Student`对象，对象内的 byte[] 属性大约占用了1Mb的空间
4. 而`ArrayList`中有200个`Student` 对象，即占用了200Mb的空间大小

## 5. 方法区

![方法区](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625674.PNG)

### 5.1 定义

[JVM规范-方法区定义](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html)

The Java Virtual Machine has a *method area* that is shared among all Java Virtual Machine threads. （Java虚拟机有一个在所有Java虚拟机线程之间共享的方法区域。）The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods ([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9)) used in class and instance initialization and interface initialization.（它存储每个类的结构，如运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括类和实例初始化以及接口初始化中使用的特殊方法。）

The method area is created on virtual machine start-up.（方法区域是在虚拟机启动时创建的。） Although the method area is logically part of the heap, simple implementations may choose not to either garbage collect or compact it. This specification does not mandate the location of the method area or the policies used to manage compiled code. The method area may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger method area becomes unnecessary. The memory for the method area does not need to be contiguous.

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the method area, as well as, in the case of a varying-size method area, control over the maximum and minimum method area size.

The following exceptional condition is associated with the method area:

- If memory in the method area cannot be made available to satisfy an allocation request, the Java Virtual Machine throws an `OutOfMemoryError`.

### 5.2 组成

JDK1.6 方法区采用了永久代来实现

![jvm内存结构jkd6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625839.PNG)

JDK8 不采用永久代，其实现改为了元空间，不存在于堆内存了，即不由 JVM 对其进行管理了

![jvm内存结构jkd8](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625898.PNG)

### 5.3 方法区内存溢出 

1.8 以前会导致永久代内存溢出 

```java
/**
 * 演示永久代内存溢出 java.lang.OutOfMemoryError: PermGen space
 * -XX:MaxPermSize=8m  永久代内存大小
 */
public class Demo1_8 extends ClassLoader { // 可以用来加载类的二进制字节码
    public static void main(String[] args) {
        int j = 0;
        try {
            Demo1_8 test = new Demo1_8();
            for (int i = 0; i < 10000; i++, j++) {
                // ClassWriter 作用是生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                // 版本号， public， 类名, 包名, 父类， 接口
                cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                // 返回 byte[]
                byte[] code = cw.toByteArray();
                // 执行了类的加载
                test.defineClass("Class" + i, code, 0, code.length); // Class 对象
            }
        } finally {
            System.out.println(j);
        }
    }
}
```

1.8 之后会导致元空间内存溢出

```java
/**
  * 演示元空间内存溢出 java.lang.OutOfMemoryError: Metaspace
  * -XX:MaxMetaspaceSize=8m
  */
// 代码相同 略
```

动态加载/生成类的场景：

* spring 中存在
* mybatis 中存在
* ...

### 5.4 运行时常量池

> **反编译代码：**
>
> ```java
> // 二进制字节码（类基本信息，常量池，类方法定义，包含了虚拟机指令）
> public class HelloWorld {
>     public static void main(String[] args) {
>         System.out.println("hello world");
>     }
> }
> ```
>
> 编译完成后，进入 target 目录找到相应的 .class 文件，采用`javap -v HelloWorld.class`对其进行反编译
>
> ```java
> C:/.../>javap -v HelloWorld.class
> // 类的基本信息如下：
> Classfile /C:/.../HelloWorld.class
>   Last modified 2020-9-9; size 547 bytes
>   MD5 checksum 72b642df992eec6e9c33e4a8b09d4022
>   Compiled from "HelloWorld.java"
> public class cn.xyc.HelloWorld
>   minor version: 0
>   major version: 52
>   flags: ACC_PUBLIC, ACC_SUPER
> // 常量池如下：地址+符号
> Constant pool: 
>    #1 = Methodref          #6.#20   // java/lang/Object."<init>":()V
>    #2 = Fieldref           #21.#22  // java/lang/System.out:Ljava/io/PrintStream;
>    #3 = String             #23      // hello world
>    #4 = Methodref          #24.#25  // java/io/PrintStream.println:(Ljava/lang/String;)V
>    #5 = Class              #26      // cn/xyc/HelloWorld
>    #6 = Class              #27      // java/lang/Object
>    #7 = Utf8               <init>
>    #8 = Utf8               ()V
>    #9 = Utf8               Code
>   #10 = Utf8               LineNumberTable
>   #11 = Utf8               LocalVariableTable
>   #12 = Utf8               this
>   #13 = Utf8               Lcn/xyc/HelloWorld;
>   #14 = Utf8               main
>   #15 = Utf8               ([Ljava/lang/String;)V
>   #16 = Utf8               args
>   #17 = Utf8               [Ljava/lang/String;
>   #18 = Utf8               SourceFile
>   #19 = Utf8               HelloWorld.java
>   #20 = NameAndType        #7:#8          // "<init>":()V
>   #21 = Class              #28            // java/lang/System
>   #22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
>   #23 = Utf8               hello world
>   #24 = Class              #31            // java/io/PrintStream
>   #25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
>   #26 = Utf8               cn/xyc/HelloWorld
>   #27 = Utf8               java/lang/Object
>   #28 = Utf8               java/lang/System
>   #29 = Utf8               out
>   #30 = Utf8               Ljava/io/PrintStream;
>   #31 = Utf8               java/io/PrintStream
>   #32 = Utf8               println
>   #33 = Utf8               (Ljava/lang/String;)V
> // 类中的方法定义如下：
> {
>   public cn.xyc.HelloWorld();  // 默认构造方法
>     descriptor: ()V
>     flags: ACC_PUBLIC
>     Code:
>       stack=1, locals=1, args_size=1
>          0: aload_0
>          1: invokespecial #1  // Method java/lang/Object."<init>":()V
>          4: return
>       LineNumberTable:
>         line 3: 0
>       LocalVariableTable:
>         Start  Length  Slot  Name   Signature
>             0       5     0  this   Lcn/xyc/HelloWorld;
> 
>   public static void main(java.lang.String[]);
>     descriptor: ([Ljava/lang/String;)V
>     flags: ACC_PUBLIC, ACC_STATIC
>     Code:
>       stack=2, locals=1, args_size=1
>          // 以下为虚拟机指令：0 3 5 8 -> 4条   #2 表示进行查表翻译，查的即为常量池表
>          0: getstatic     #2   // Field java/lang/System.out:Ljava/io/PrintStream;
>          3: ldc           #3   // String hello world
>          5: invokevirtual #4   // Method java/io/PrintStream.println(Ljava/lang/String;)V
>          8: return
>       LineNumberTable:
>         line 5: 0
>         line 6: 8
>       LocalVariableTable:
>         Start  Length  Slot  Name   Signature
>             0       9     0  args   [Ljava/lang/String;
> }
> SourceFile: "HelloWorld.java"
> ```

* **常量池，就是一张表**，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息
* 运行时常量池，常量池是 `*.class` 文件中的，当该类被加载，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址

### 5.5 StringTable 

先看几道面试题： 

```java
String s1 = "a";
String s2 = "b";
String s3 = "a" + "b";
String s4 = s1 + s2;
String s5 = "ab";
String s6 = s4.intern();
// 问：下面的结果
System.out.println(s3 == s4);  // flase, s3在串池中，而s4在堆中
System.out.println(s3 == s5);  // true，都指向了常量池
System.out.println(s3 == s6);  // true，intern()把对象放入串池，并返回对象

String x2 = new String("c") + new String("d");
String x1 = "cd";
x2.intern();
System.out.println(x1 == x2);  // false, x1引用了串池中对象，x2为堆中对象
// 问：如果调换了【最后两行代码】的位置呢，如果是jdk1.6呢
// 如果调换x1和x2的位置，即
// x2.intern();
// String x1 = "cd";
// 此时在jdk1.8中为true、在jdk1.6中为false，详见后续5.6中对于intern的说明
// 即：JDK1.8 调用intern()若串池中没有这个字符串则放入
```

**demo1:**

```java
// 源码
public class Test8 {
    public static void main(String[] args) {
        String s1 = "a";  // 懒惰创建
        String s2 = "b";  // 懒惰创建
        String s3 = "ab";  // 懒惰创建
    }
}

// 反编译结果：
// 常量池：
Constant pool:
  #2 = String             #25            // a
  #3 = String             #26            // b
  #4 = String             #27            // ab
  #25 = Utf8               a
  #26 = Utf8               b
  #27 = Utf8               ab

// 代码：
Code:
  stack=1, locals=4, args_size=1
     0: ldc           #2                  // String a
     2: astore_1
     3: ldc           #3                  // String b
     5: astore_2
     6: ldc           #4                  // String ab
     8: astore_3
     9: return

// 说明：
//    - 常量池中的信息，都会被加载到运行时常量池中，这时 a b ab 都是常量池中的符号，还没有变为java字符串对象
//    - ldc #2    会把 a 符号变为 “a” 字符串对象，若 StringTable 中不存在，则放入，变为：StringTable[ "a" ]
//    - ldc #3    同上流程，变为：StringTable[ "a", "b" ]
//    - ldc #4    同上流程，变为：StringTable[ "a", "b", "ab" ]
// 注：StringTable 为 hashtable 结构，不能扩容
```

**demo2：接上**

```java
// 源码
String s4 = s1 + s2;

// 反编译结果：
// 常量池：
Constant pool:
   #2 = String             #30  // a
   #3 = String             #31  // b
   #4 = String             #32  // ab
   #5 = Class              #33  // java/lang/StringBuilder
   #6 = Methodref          #5.#29  // java/lang/StringBuilder."<init>":()V
   #7 = Methodref          #5.#34  // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #5.#35  // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #29 = NameAndType        #11:#12  // "<init>":()V
  #30 = Utf8               a
  #31 = Utf8               b
  #32 = Utf8               ab
  #33 = Utf8               java/lang/StringBuilder
  #34 = NameAndType        #38:#39  // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #35 = NameAndType        #40:#41  // toString:()Ljava/lang/String;
	
// 代码：
Code:
  stack=2, locals=5, args_size=1
     0: ldc           #2  // String a
     2: astore_1
     3: ldc           #3  // String b
     5: astore_2
     6: ldc           #4  // String ab
     8: astore_3
     9: new           #5  // class java/lang/StringBuilder
    12: dup
    13: invokespecial #6  // Method java/lang/StringBuilder."<init>V
    16: aload_1
    17: invokevirtual #7  // Method java/lang/StringBuilder.append:va/lang/String;)Ljava/lang/StringBuilder;
    20: aload_2
    21: invokevirtual #7  // Method java/lang/StringBuilder.append:va/lang/String;)Ljava/lang/StringBuilder;
    24: invokevirtual #8  // Method java/lang/StringBuilder.toStrinLjava/lang/String;
    27: astore        4
    29: return  
        
// 总结，上述代码做了如下 String s4 = s1 + s2;
//   new StringBuilder(s1, s2).toString() -> new String("ab")
```

**demo3：再接上**

```java
// 源码
String s5 = "a" + "b";

// 反编译结果：
// 常量池：
Constant pool:
   #4 = String             #33  // ab
   #33 = Utf8               ab

// 代码：
Code:
      stack=2, locals=6, args_size=1
		// ...源码对应的指令如下：
        29: ldc           #4  // String ab
        31: astore        5
        33: return

// 分析，对比String s3 = "ab";
// 其源码为：6: ldc  #4  // String ab
// 同样是寻找常量池中 #4 的位置，因此：
// s3 == s5 的结果为true
// 出于 javac 在编译器的优化，在编译期间，s5的结果已经确定为ab，不会再被修改了
```

### 5.6 StringTable 特性 

* 常量池中的字符串仅是符号，**第一次用到时才变为对象**

* 利用串池的机制，来避免重复创建字符串对象

  > ```java
  > /**
  >  * 演示字符串字面量也是【延迟】成为对象的
  >  */
  > public class TestString {
  >        public static void main(String[] args) {
  >            int x = args.length;
  >            System.out.println(); // 字符串个数 2275
  > 
  >            System.out.print("1");
  >            System.out.print("2");
  >            System.out.print("3");
  >            System.out.print("4");
  >            System.out.print("5");
  >            System.out.print("6");
  >            System.out.print("7");
  >            System.out.print("8");
  >            System.out.print("9");
  >            System.out.print("0");
  >            System.out.print("1"); // 字符串个数 2285
  >            System.out.print("2");
  >            System.out.print("3");
  >            System.out.print("4");
  >            System.out.print("5");
  >            System.out.print("6");
  >            System.out.print("7");
  >            System.out.print("8");
  >            System.out.print("9");
  >            System.out.print("0");
  >            System.out.print(x); // 字符串个数
  >        }
  > }
  > ```

* 字符串变量拼接的原理是 StringBuilder （1.8）

* **字符串常量拼接的原理是编译期优化**

* 可以使用 `intern()` 方法，**主动将串池中还没有的字符串对象放入串池**

  * 1.8 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则**放入串池， 会把串池中的对象返回**
  * 1.6 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有会把此对象复制一份（即**调用intern方法的对象和放入串池的对象是两个对象**），把复制的对象放入串池， 再把串池中的对象返回 

**对于intern说明：JDK1.8**

```java
public static void main(String[] args) {
    String x = "ab";
    String s = new String("a") + new String("b");
    
    String s2 = s.intern();  // 会返回串池中的对象
    
    System.out.println( s2 == x);  // true, 都是指向了串池中的对象
	System.out.println( s == x );  // flase，s为堆中的对象
}
```

说明：

```java
// 将“ab” 放入串池，此时StringTable为["ab"]
String x = "ab";
// "a", "b" 是常量被放入串池当中, 此时StringTable为["ab", "a", "b"]
// 而new出来的对象在堆中new String("a")与new String("b")
// 而s仅为堆中的一个对象 String("ab")，并不在串池中
String s = new String("a") + new String("b");
// 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池,会把串池中的对象返回
String s2 = s.intern();
```

**对于intern说明：JDK1.6**

```java
String s = new String("a") + new String("b");
// 上述执行后：StringTable为["a", "b"]
// s为堆中的一个对象 String("ab")
String s2 = s.intern();  // 将s拷贝一份放入串池，返回结果为串池中的对象
String x = "ab";  // x也指向串池中的对象
System.out.println( s2 == x);  // true, 都是指向了串池中的对象
System.out.println( s == x );  // flase，s为堆中的对象
```

而在JDK1.8的环境下：

```java
System.out.println( s2 == x );  // true, 都是指向了串池中的对象
System.out.println( s == x );   // true，JDK8：s.intern()把这个s对象放入串池，而JDK6则是拷贝s一份放入，因此上述为false，这里为true
```

### 5.7 StringTable 位置

JDK1.6 中，StringTable 在永久代中；

**JDK1.8 中，StringTable 被放入堆中**；

理由如下：

* 永久代中内存回收效率很低，只有在 Full GC 时才会触发垃圾回收；
* 而堆中，当 Minor GC 时就会触发垃圾回收，大大减轻了常量字符串对内存的占用。

**证明如下：**

```java
/**
 * 演示 StringTable 位置
 * 在jdk8下设置 -Xmx10m -XX:-UseGCOverheadLimit
 * 在jdk6下设置 -XX:MaxPermSize=10m  // 设置永久代内存空间
 */
public class Demo1_6 {

    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<String>();
        int i = 0;
        try {
            for (int j = 0; j < 260000; j++) {
                // 把整数转换为String对象后放入StringTable中
                list.add(String.valueOf(j).intern());
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
```

JDK1.6结果：

```java
java.lang.OutOfMemoryError: PermGen space
```

JDK1.8结果：

```java
// -Xmx10m  只设置堆最大内存，报错如下：
// 下述错误是由于花了超过98%的时间来进行GC，而回收的堆空间只是2%
java.lang.OutOfMemoryError: GC overhead limit exceeded
// -Xmx10m -XX:-UseGCOverheadLimit  // 关闭这个开关
// 则错误如下：
java.lang.OutOfMemoryError: Java heap space
```

### 5.8 StringTable 垃圾回收

**演示代码：**

```java
/**
 * 演示 StringTable 垃圾回收
 * -Xmx10m -XX:+PrintStringTableStatistics -XX:+PrintGCDetails -verbose:gc
 * JVM参数解释如下：
 *    -Xmx10m：设置堆内存空间最大为10Mb
 *    -XX:+PrintStringTableStatistics： 打印StringTable的统计信息
 *    -XX:+PrintGCDetails -verbose:gc：打印垃圾回收的相应信息
 */
public class Demo1_7 {
    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        try {
            
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
```

输出结果：

```java
0
Heap
 PSYoungGen      total 2560K, used 1810K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 88% used [0x00000000ffd00000,0x00000000ffec4a50,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 0K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 0% used [0x00000000ff600000,0x00000000ff600000,0x00000000ffd00000)
 Metaspace       used 3232K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
// 符号表：
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13305 =    319320 bytes, avg  24.000
Number of literals      :     13305 =    568576 bytes, avg  42.734
Total footprint         :           =   1047984 bytes
Average bucket size     :     0.665
Variance of bucket size :     0.665
Std. dev. of bucket size:     0.816
Maximum bucket size     :         6
// StringTable信息， hashtable的结构
StringTable statistics:
//                           桶的个数      占用字节量
Number of buckets       :     60013 =    480104 bytes, avg   8.000
// 	                         键值个数      占用字节量     
Number of entries       :      1758 =     42192 bytes, avg  24.000
//                      字符串常量个数      占用字节量  
Number of literals      :      1758 =    157640 bytes, avg  89.670
//                                        总的占用空间   
Total footprint         :           =    679936 bytes
Average bucket size     :     0.029
Variance of bucket size :     0.029
Std. dev. of bucket size:     0.172
Maximum bucket size     :         3
```

在try块中加入如下：100个常量

```java
try {
    for (int j = 0; j < 100; j++) { // j=100, j=10000
        String.valueOf(j).intern();
        i++;
    }
}
```

StringTable 信息如下：

```java
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
// 	                         键值个数增加了100个
Number of entries       :      1858 =     44592 bytes, avg  24.000
Number of literals      :      1858 =    162440 bytes, avg  87.427
Total footprint         :           =    687136 bytes
Average bucket size     :     0.031
Variance of bucket size :     0.031
Std. dev. of bucket size:     0.176
Maximum bucket size     :         3
```

在try块中加入如下：10000个常量，由于虚拟机对堆空间大小设置为10Mb，因此会出现内存不够的情况，发生垃圾回收

```java
try {
    for (int j = 0; j < 10000; j++) { // j=100, j=10000
        String.valueOf(j).intern();
        i++;
    }
}
```

StringTable 信息如下：

```java
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
// 	                          键值个数并没有增加相应个数，即增加10000个，因为发生了垃圾回收
Number of entries       :      5275 =    126600 bytes, avg  24.000
Number of literals      :      5275 =    326536 bytes, avg  61.903
Total footprint         :           =    933240 bytes
```

输出结果中可以看到GC信息如下：

```java
// GC 由于内存空间分配失败，而触发垃圾空间回收
[GC (Allocation Failure) [PSYoungGen: 2048K->504K(2560K)] 2048K->682K(9728K), 0.0015966 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

### 5.9 StringTable 性能调优

**通过控制桶的个数-XX:StringTableSize=xxx**

```java
/**
 * 演示串池大小对性能的影响
 * -Xms500m -Xmx500m -XX:+PrintStringTableStatistics -XX:StringTableSize=1009
 * -XX:StringTableSize=1009  设置桶个数大小
 */
public class Demo1_24 {
    public static void main(String[] args) throws IOException {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
            String line = null;
            long start = System.nanoTime();
            while (true) {
                line = reader.readLine();
                if (line == null) {
                    break;
                }
                line.intern();  // 将单词放入串池
            }
            // 查看入池花费时间
            System.out.println("cost:" + (System.nanoTime() - start) / 1000000);
        }
    }
}
```

> linux.words文件中大致存了48w个单词

设置桶个数大小为200000w输出如下

```java
// 添加虚拟机参数：-Xms500m -Xmx500m -XX:+PrintStringTableStatistics -XX:StringTableSize=200000
cost:213  // 花费时间ms
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13309 =    319416 bytes, avg  24.000
Number of literals      :     13309 =    568664 bytes, avg  42.728
Total footprint         :           =   1048168 bytes
Average bucket size     :     0.665
Variance of bucket size :     0.666
Std. dev. of bucket size:     0.816
Maximum bucket size     :         6
StringTable statistics:
//                           桶个数为200000
Number of buckets       :    200000 =   1600000 bytes, avg   8.000
Number of entries       :    481510 =  11556240 bytes, avg  24.000
Number of literals      :    481510 =  29731304 bytes, avg  61.746
Total footprint         :           =  42887544 bytes
Average bucket size     :     2.408
Variance of bucket size :     2.420
Std. dev. of bucket size:     1.556
Maximum bucket size     :        12
```

默认桶个数（即去掉`-XX:StringTableSize=200000`），输出如下：

```java
cost:295  // 花费时间ms
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13309 =    319416 bytes, avg  24.000
Number of literals      :     13309 =    568664 bytes, avg  42.728
Total footprint         :           =   1048168 bytes
Average bucket size     :     0.665
Variance of bucket size :     0.666
Std. dev. of bucket size:     0.816
Maximum bucket size     :         6
StringTable statistics:
//                           桶个数为60013
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :    481510 =  11556240 bytes, avg  24.000
Number of literals      :    481510 =  29731304 bytes, avg  61.746
Total footprint         :           =  41767648 bytes
Average bucket size     :     8.023
Variance of bucket size :     8.085
Std. dev. of bucket size:     2.843
Maximum bucket size     :        23
```

设置桶个数为1009，结果如下：

> 该值范围：`StringTable size of 100 is invalid; must be between 1009 and 2305843009213693951`

```java
cost:7521  // 时间明显变慢，即多次发生了哈希碰撞
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13309 =    319416 bytes, avg  24.000
Number of literals      :     13309 =    568664 bytes, avg  42.728
Total footprint         :           =   1048168 bytes
Average bucket size     :     0.665
Variance of bucket size :     0.666
Std. dev. of bucket size:     0.816
Maximum bucket size     :         6
StringTable statistics:        // 桶的个数为1009
Number of buckets       :      1009 =      8072 bytes, avg   8.000
Number of entries       :    481510 =  11556240 bytes, avg  24.000
Number of literals      :    481510 =  29731304 bytes, avg  61.746
Total footprint         :           =  41295616 bytes
Average bucket size     :   477.215
Variance of bucket size :   433.737
Std. dev. of bucket size:    20.826
Maximum bucket size     :       544
```

**考虑将字符串对象是否入栈**

```java
/**
 * 演示 intern 减少内存占用
 * -XX:StringTableSize=200000 -XX:+PrintStringTableStatistics
 * -Xsx500m -Xmx500m -XX:+PrintStringTableStatistics -XX:StringTableSize=200000
 */
public class Demo1_25 {
    public static void main(String[] args) throws IOException {
        List<String> address = new ArrayList<>();
        System.in.read();  // 等待用户输入
        for (int i = 0; i < 10; i++) {  // 重复10次 将单词读入内存
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
                String line = null;
                long start = System.nanoTime();
                while (true) {
                    line = reader.readLine();
                    if(line == null) {
                        break;
                    }
                    address.add(line); // 防止被垃圾回收
                    // address.add(line.intern());  // 做一个入池操作
                }
                System.out.println("cost:" +(System.nanoTime()-start)/1000000);
            }
        }
        System.in.read();
    }
}
```

1. 运行程序；

2. 使用 `jvisualvm`，选择启动的进程，选择 **抽样器**，选择 **内存**，可以查看内存使用情况，如下：

   ![内存抽样器1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625020.PNG)

   可以看到此时，String对象占用内存为2.2%

3. 程序回车，使得单词`address.add(line);`，此时 `String对象+char[]` 占用内存可达80%多，且堆中总共使用了约 300Mb 的字节数

   ![内存抽样器2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625538.PNG)

4. 将 String 做一个入池操作，如下：`address.add(line.intern()); `再次运行程序，此时堆占用内存约180Mb，且 `String对象+char[]` 占用内存可达70%多，相对而言占用堆内存大幅减少

   > ps：视频颜色中chra[]数组只占了20%，与String对象相加只占用了40%不到的样子，且用的内存也较本地实验少，原因未知...

   ![内存抽样器3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625583.PNG)

**结论：若应用中有大量的字符串，且字符串可能存在重复问题，可以使得字符串入池来减少堆内存的使用。**

## 6. 直接内存 

### 6.1 定义

**Direct Memory** 

* 常见于 NIO 操作时，用于数据缓冲区
* 分配回收成本较高，但读写性能高（ByteBuffer 的演示实验）
* 不受 JVM 内存回收管理，而是系统内存

**演示 ByteBuffer 作用：-> 读写性能高**

```java
/**
 * 演示 ByteBuffer 作用
 */
public class Demo1_9 {
    static final String FROM = "D:\\movie\\damingwangchao001.mkv";  // 618 MB
    static final String TO = "D:\\movie\\damingwangchao001-fuben.mkv";
    static final int _1Mb = 1024 * 1024;

    public static void main(String[] args) {
        io(); // io 用时：8718.4229  1223.9355  1175.8901
        directBuffer(); // directBuffer 用时：474.8089  786.7621  436.4568
    }

    private static void io() {
        long start = System.nanoTime();
        try (FileInputStream from = new FileInputStream(FROM);
             FileOutputStream to = new FileOutputStream(TO);
        ) {
            byte[] buf = new byte[_1Mb];
            while (true) {
                int len = from.read(buf);
                if (len == -1) {
                    break;
                }
                to.write(buf, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("io 用时：" + (end - start) / 1000_000.0);
    }

    private static void directBuffer() {
        long start = System.nanoTime();
        try (FileChannel from = new FileInputStream(FROM).getChannel();
             FileChannel to = new FileOutputStream(TO).getChannel();
        ) {
            ByteBuffer bb = ByteBuffer.allocateDirect(_1Mb);
            while (true) {
                int len = from.read(bb);
                if (len == -1) {
                    break;
                }
                bb.flip();
                to.write(bb);
                bb.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("directBuffer 用时：" + (end - start) / 1000_000.0);
    }
}
```

当使用 `io()` 拷贝文件时：用户需要进行文件读写时，需要调用操作系统提供的函数

![文件读写1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625642.PNG)

使用 `directBuffer()`拷贝文件时：通过`ByteBuffer.allocateDirect`分配一块**直接内存（direct memory）**，为系统与 java堆共享的一块内存区域

![文件读写2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625197.PNG)

### 6.2 分配和回收原理

**直接内存不受 JVM 内存回收管理**

**演示：演示直接内存溢出**

```java
/**
 * 演示直接内存溢出
 */
public class Demo1_10 {
    static int _100Mb = 1024 * 1024 * 100;

    public static void main(String[] args) {
        List<ByteBuffer> list = new ArrayList<>();
        int i = 0;
        try {
            while (true) {
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_100Mb);
                list.add(byteBuffer);
                i++;
            }
        } finally {
            System.out.println(i);
        }
        // 方法区是jvm规范，jdk6 中对方法区的实现称为永久代
        //               jdk8 对方法区的实现称为元空间
    }
}
```

输出：错误原因是直接缓冲区内存溢出

```java
36
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
```

**演示：禁用显式回收对直接内存的影响**

```java
/**
 * 禁用显式回收对直接内存的影响
 */
public class Demo1_26 {
    static int _1Gb = 1024 * 1024 * 1024;

    /*
     * -XX:+DisableExplicitGC 显式的
     */
    public static void main(String[] args) throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1Gb);
        System.out.println("分配完毕...");
        System.in.read();
        System.out.println("开始释放...");
        byteBuffer = null;
        System.gc(); // 显式的垃圾回收，Full GC
        System.in.read();
    }
}
```

> 由于分配的内存为直接内存，故原先的关于java的分析工具已经无法观察到现象了，通过windows中的任务管理器内存进行查看

1. 启动程序前内存使用情况：

   ![直接内存分配1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625094.PNG)

2. 启动后，在成功分配内存后，内存的使用情况：

   ![直接内存分配2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625212.PNG)

   可见，内存多出了1GB的占用；

3. 将内存释放后的情况：

   ![直接内存分配3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251625617.PNG)

   可见，占用的1GB的直接内存被回收了

问：直接内存，不受 JVM 内存回收管理，而上述垃圾回收导致了内存被释放，原理如何？

> **直接内存分配与回收的底层原理：Unsafe**
>
> ```java
> /**
>  * 直接内存分配与回收的底层原理：Unsafe
>  */
> public class Demo1_27 {
>     static int _1Gb = 1024 * 1024 * 1024;
> 
>     public static void main(String[] args) throws IOException {
>         Unsafe unsafe = getUnsafe();
>         // 分配内存，返回分配的直接内存的地址
>         long base = unsafe.allocateMemory(_1Gb);
>         unsafe.setMemory(base, _1Gb, (byte) 0);
>         System.in.read();
> 
>         // 释放直接内存
>         unsafe.freeMemory(base);
>         System.in.read();
>     }
> 
>     public static Unsafe getUnsafe() {
>         try {
>             // 通过反射创建Unsafe对象
>             Field f = Unsafe.class.getDeclaredField("theUnsafe");
>             // 暴力反射
>             f.setAccessible(true);
>             Unsafe unsafe = (Unsafe) f.get(null);
>             return unsafe;
>         } catch (NoSuchFieldException | IllegalAccessException e) {
>             throw new RuntimeException(e);
>         }
>     }
> }
> ```
>
> > Unsafe类，可以做一些分配直接内存，释放直接内存的操作；
> >
> > 不能直接创建Unsafe对象，只能通过反射创建Unsafe对象
>
> 程序运行时内存的分配与回收效果如上一个程序；
>
> 通过`unsafe.setMemory`分配内存，通过`unsafe.freeMemory`回收内存，与 JVM 的 GC 回收无用的对象不同，直接内存必须主动进行回收

对 `ByteBuffer` 进行源码分析如下：

```java
// 1.ByteBuffer.allocateDirect(_1Gb);
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}

// 2. DirectByteBuffer
DirectByteBuffer(int cap) {                   // package-private
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        base = unsafe.allocateMemory(size);  // 这里用了unsafe对象，对直接内存进行分配
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    // 解释见3.与4.
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}

// 3. cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
// 3.1 对于回调任务对象new Deallocator(base, size, cap)，如下
private static class Deallocator implements Runnable
{

     private static Unsafe unsafe = Unsafe.getUnsafe();

     private long address;
     private long size;
     private int capacity;

     private Deallocator(long address, long size, int capacity) {
         assert (address != 0);
         this.address = address;
         this.size = size;
         this.capacity = capacity;
     }

     public void run() {
         if (address == 0) {
             // Paranoia
             return;
         }
         unsafe.freeMemory(address);  // 在这里释放了分配的地址
         address = 0;
         Bits.unreserveMemory(size, capacity);
     }
}

// 3.2 Cleaner对象，在java类库中为一种特殊的类型，称为虚引用类型
// 其特点为：当其关联的对象被回收时，Cleaner会触发clean方法
// 而上述cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
// cleaner 关联的 this 对象为 ByteBuffer，被垃圾回收掉时，触发了Cleaner中的clean方法
public void clean() {
    if (remove(this)) {
        try {
            this.thunk.run();  // 这里的run，即上述3.1中的run方法，释放分配空间
        } catch (final Throwable var2) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    if (System.err != null) {
                        (new Error("Cleaner terminated abnormally", var2)).printStackTrace();
                    }
                    System.exit(1);
                    return null;
                }
            });
        }
    }
}
```

**总结：**

* 使用了 Unsafe 对象完成直接内存的分配回收，并且回收需要主动调用 freeMemory 方法
* ByteBuffer 的实现类内部，**使用了 Cleaner （虚引用）来监测 ByteBuffer 对象**，一旦 ByteBuffer 对象被垃圾回收，那么就会由 ReferenceHandler 线程通过 Cleaner 的 clean 方法调用 freeMemory 来释放直接内存

> 关于JVM参数：`-XX:+DisableExplicitGC`     
>
> 一般在做 JVM 调优时，会加入上述参数，参数的作为为：禁用显示的垃圾回收，即让代码中的` System.gc(); `无效；
>
> ` System.gc(); `相当于通过代码进行一次显示的垃圾回收，而由于该垃圾回收触发的是一次 Full GC，比较影响性能；
>
> 但是加上该参数后，会影响到直接内存回收机制，此时 ByteBuffer 不再被显示的回收了，因此 ByteBuffer 只能等到真正的垃圾回收时才会被清理，其对应的直接内存才会被释放掉。
>
> 解决方式：直接通过 Unsafe 对象调用 freeMemory 进行内存释放，其手动管理直接内存。

