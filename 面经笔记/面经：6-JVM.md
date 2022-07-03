# 面经：6-JVM

> 个人整理 :muscle: 个人专用 :muscle: 暑期实习 :muscle: 秋招 :muscle: 后端开发 :muscle: 八股文 :no_mouth:

### Java 内存模型 :exclamation:

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。  

**JDK8 之前：** 

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251809314.PNG" alt="JVM角度分析线程和进程的关系" style="zoom:50%;" />

**JDK 8 时：**

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251809245.PNG" alt="JDK8的内存结构" style="zoom:50%;" />

**线程私有的部分：**

* 程序计数器 Program Counter Register
* 虚拟机栈 VM Stack
* 本地方法栈 Native Method Stack

**线程共享的部分：** 

* 堆 Heap
* 方法区 Method Area
* 直接内存(非运行时数据区的一部分)  Direct Memory

### 程序计数器 :exclamation:

Program Counter Register 程序计数器（寄存器），程序计数器是一块较小的内存空间

- **作用：是记住下一条 JVM 指令的执行地址**

  > `JVM指令 -> 解释器 -> 机器码 -> CPU执行`

- **特点**

  - **是线程私有的**：为了线程上下文切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。
  - **不会存在内存溢出**：程序计数器是**唯一一个**不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。 

### Java 虚拟机栈 

Java Virtual Machine Stacks，Java 虚拟机栈，与程序计数器一样， Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的

- 每个线程运行时所需要的内存，称为虚拟机栈；
- 每个线程由多个**栈帧（Frame）**组成，对应着每次**方法调用**时所占用的内存；
- 每个线程**只能有一个活动栈帧**，对应着当前正在执行的那个方法；
- **栈帧**：每个方法运行时需要的内存，例如方法的参数，局部变量，返回地址等都需要占用内存；

> **问题辨析**
>
> 1. **垃圾回收是否涉及栈内存？**
>
>       答：不会，栈内的栈帧内存，在方法调用结束后都会出栈；**垃圾回收只是涉及到堆内存**。
>
> 2. **栈内存分配越大越好吗？**
>
>       答：不是，栈内存划分越大会导致线程数减少，由于物理内存大小是一定的，线程会使用到栈内存，因此栈内存越大则会可运行线程数减少。
>
> 3. **方法内的局部变量是否线程安全？**
>
>    - 如果方法内局部变量没有逃离方法的作用访问，它是线程安全的
>    - 如果是局部变量引用了对象，并逃离方法的作用范围，需要考虑线程安全 
>
> 4. **那么方法/函数如何调用？**
>
>       Java 栈可用类比数据结构中栈， Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。
>
>       Java 方法有两种返回方式：
>
>       1. return 语句
>       2. 抛出异常
>
>       不管哪种返回方式都会导致栈帧被弹出。 

Java 虚拟机栈会出现两种异常：

- 栈帧过多导致栈内存溢出：**StackOverFlowError**

  若 Java 虚拟机栈的内存大小不允许动态扩展，那么**当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候**，就抛出 StackOverFlowError 异常。 

- 栈帧过大导致栈内存溢出：**OutOfMemoryError**

  若 Java 虚拟机栈的内存大小允许动态扩展，且**当线程请求栈时内存用完了，无法再动态扩展了**，此时抛出 OutOfMemoryError 异常。 

### 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 

> 在 HotSpot 虚拟机中 本地方法栈 和 Java 虚拟机栈合⼆为一

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。 

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种异常。 

### 堆 Heap

**Heap 堆**

- 通过 new 关键字，创建对象都会使用堆内存

**特点**

- 它是线程共享的，堆中对象都需要考虑线程安全的问题
- 有垃圾回收机制

堆是 Java 虚拟机所管理的内存中最大的一块， Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。 **此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作 **GC堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用**分代垃圾收集算法**，所以 Java堆 还可以细分为：

* 新生代：Eden空间、 From Survivor、 To Survivor 空间
* 老年代

进一步划分的目的是更好地回收内存，或者更快地分配内存。

![分代垃圾回收](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251809846.PNG)

**分代垃圾回收工作机制**：

- 创建的新对象首先分配在伊甸园区域，之后更多对象被创建，放入伊甸园中；
- 新生代空间不足时，触发 `Minor GC`，伊甸园和From存活的对象使用**copy复制**到to中，存活的对象年龄+1并且交换 from 和 to；
- `Minor GC` 会引发 `stop the world`，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行，这是出于在GC时，涉及到了对象地址的修改；
- 触发`Minor GC`后，伊甸园有空间了，继续将生成的对象放入，直至再次发生`Minor GC`进行相应操作；
- 当幸存区中对象的寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit）；
- 当老年代空间不足，会先尝试触发 `Minor GC`，如果之后空间仍不足，那么触发 `Full GC`，`stop the world`的时间更长；
- 若触发 `Full GC`后，老年代的空间还是不足，则会导致堆内存溢出。

### 方法区

方法区与 Java 堆一样，**是各个线程共享的内存区域**，它用于**存储已被虚拟机加载的：类信息、常量、静态变量、即时编译器编译后的代码、...**

虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个**别名叫做 Non-Heap（非堆）** ，目的应该是与 Java 堆区分开来。

> **JDK1.6 方法区采用了永久代来实现**：
>
> ![jvm内存结构jkd6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251809649.PNG)
>
> **调整永久代的大小 JVM 参数：**
>
> ```bash
> -XX:PermSize=N     # 方法区(永久代)初始大小
> -XX:MaxPermSize=N  # 方法区(永久代)最大大小,超过这个值将会抛出OutOfMemoryError异常:java.lang.OutOfMemoryError: PermGen
> ```
>
> 相对而⾔，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。
>
> **方法区和永久代的关系**：
>
> 《Java虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 **方法区和永久代的关系很像Java中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。**也就是说，永久代是HotSpot的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。 

JDK8 不采用永久代，其实现改为了**元空间**，不存在于堆内存了，即不由 JVM 对其进行管理了，元空间使用的是直接内存。 

![jvm内存结构jkd8](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810200.PNG)

下面是一些常用参数：

```bash
-XX:MetaspaceSize=N     # 设置Metaspace的初始（和最小大小）
-XX:MaxMetaspaceSize=N  # 设置Metaspace的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，**虚拟机会耗尽所有可用的系统内存**。 

**为什么要将永久代(PermGen)替换为元空间(MetaSpace)呢?** 

1. 整个永久代有一个 JVM 本身设置固定大小上限，无法进行调整；
2. 而元空间使用的是直接内存，受本机可用内存的限制
   * 你可以使用 `-XX: MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。 
   * `-XX：MetaspaceSize` 调整标志定义元空间的初始大小，如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

> 当然这只是其中一个原因，还有很多底层的原因，这里就不提了。 

### 运行时常量池

运行时常量池是方法区的一部分。 

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用） 

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。 

### class常量池 & 运行时常量池 & 字符串常量池 :exclamation:

**class 常量池**：

* **就是一张表，是静态的，存在于 `.class` 文件中**，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息；
* 每一个 Java 类被编译后，就会形成一份 class 文件，其中就包含了**常量池，即：每个class文件都有一个class常量池。**

**运行时常量池**：

* **当该类被加载**，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址；
* 即：运行时常量池就是class常量池被加载到内存之后的版本，**是方法区的一部分**

* JVM 在执行某个类的时候，必须经过**加载、连接、初始化**，而**连接又包括验证、准备、解析**三个阶段。而当类加载到内存中后，**JVM 就会将 class常量池 中的内容存放到运行时常量池中**，由此可知，**运行时常量池也是每个类都有一个**。在解析阶段，会把符号引用替换为直接引用，解析的过程会去查询**字符串常量池**，也就是我们下面要说的 StringTable，以保证运行时常量池所引用的字符串与字符串常量池中是一致的。

**字符串常量池 & StringTable**

* 在 JDK 6 及之前版本，StringTable/字符串常量池 存放在方法区

* **JDK 8 中，StringTable/字符串常量池 被放入堆中**；

  理由如下：

  - **永久代中内存回收效率很低**，只有在 Full GC 时才会触发垃圾回收；
  - 而堆中，当 Minor GC 时就会触发垃圾回收，大大减轻了常量字符串对内存的占用。

* 在 HotSpot VM 中实现字符串常量池的是一个 StringTable 类，**为 HashTable 结构，但不能扩容**，这个StringTable 只有一份实例，被所有的类共享。字符串常量由一个一个字符组成，放在了 StringTable 上。

> StringTable 性能调优：
>
> - **通过控制桶的个数 `-XX:StringTableSize=xxx`**
> - 当放入 字符串常量池 的 String 对象非常多，就会造成 Hash 冲突，导致链表过长等情况，会对性能造成一定的影响

* **代码说明1**：`String s = new String("abc");`第一次调用时，会创建两个对象，
  - 第一个对象是`"abc"`字符串存储在**字符串常量池&StringTable**中
  - 第二个对象是在 **堆内存** 中
* 代码说明2：`String s1 = new String("abc"); String s2 = "abc";`
  - s1 是**运行时期创建**，对象在堆中，同时会在字符串常量池中创建对象；
  - s2 是先在栈中创建一个对String类的对象的引用变量，然后去字符串常量池中寻找是否存在 "abc"，若存在的话，直接将 s2 指向 "abc"，如果没有的话，将 "abc" 放入字符串常量池，然后再将 s2 指向 "abc"。**“abc”存于常量池是在编译期完成的**
* 代码说明3：`String s1 = new String("abc"); String s2 = new String("abc");`
  - 总共创建了3个对象，编译期在字符串常量池中创建对象 "abc"
  - 在运行期，在堆中创建了两个 String 的对象

> 必须列出的参考：https://www.jianshu.com/p/44c9eda51aa1

### 直接内存

**Direct Memory**：

- 常见于 NIO 操作时，用于数据缓冲区
- 分配回收成本较高，但读写性能高
- 不受 JVM 内存回收管理，而是系统内存

JDK1.4 中新加入的 NIO(New/Non-Blocking I/O) 类，引入了一种基于**通道（Channel）** 与 **缓存区（Buffer）**的 I/O 方式，它可以直接使用 native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 **DirectByteBuffer 对象**作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆内存 和 系统内存 之间来回复制数据。 

> 当使用 `io()` 拷贝文件时：用户需要进行文件读写时，需要调用操作系统提供的函数
>
> ![文件读写1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810979.PNG)
>
> 使用 `directBuffer()`拷贝文件时：通过`ByteBuffer.allocateDirect`分配一块**直接内存（direct memory）**，为系统与 Java堆 共享的一块内存区域
>
> ![文件读写2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810440.PNG)

### Java 对象的创建过程 :exclamation::exclamation:

![Java对象创建过程](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810421.PNG)

1. **类加载检查**： 虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数能否在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。 

2. **分配内存**： **在类加载检查通过后，接下来虚拟机将为新生对象分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。 分配方式有 “**指针碰撞**” 和 “**空闲列表**” 两种， 选择那种分配方式由 Java 堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。 

   > 后续进一步说明
   
3. **初始化零值**：内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。 

4. **设置对象头**：初始化零值完成之后，虚拟机要对对象进行必要的设置，例如：这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。**这些信息存放在对象头中**。另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。 

5. **执行 init 方法**： 在上面工作都完成之后，**从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始**， `<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。 

> **内存分配的两种方式**：（补充内容，需要掌握） 
>
> 选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是 "标记-清除"，还是"标记-整理" （也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的
>
> 分配内存的两种方式：
>
> - **指针碰撞**
>   - 适用场合：堆内存规整（即没有内存碎片）的情况下；
>   - 原理：用过的内存全部整合到一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可；
>   - GC收集器：Serial、ParNew
> - **空闲列表**
>   - 使用场合：堆内存不规整的情况下；
>   - 原理：虚拟机会维护一个列表，该列表中会记录哪些块是可以用的，在分配的时候，找一块足够大的内存块来划分给实例对象；
>   - GC收集器：CMS
>
> **内存分配并发问题**：（补充内容，需要掌握） 
>
> 在创建对象的时候有一个很重要的问题，就是**线程安全**，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全： 
>
> - **CAS+失败重试**： CAS 是乐观锁的一种实现方式。所谓乐观锁就是：每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。 虚拟机采用 CAS 配上失败重试的方式保证更新操作的原⼦性
> - **TLAB**（Thread Local Allocation Buffer，线程本地分配缓存表）： 为每一个线程预先在 Eden 区分配一块内存， JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的CAS进行内存分配 

### 对象的访问定位方式

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。

对象的访问方式由虚拟机实现而定，目前主流的访问方式有

1. 使用**句柄**：如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池， reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息； 

   ![通过句柄访问对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810985.PNG)

2. **直接指针**：如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。 

   ![通过直接指针访问对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810805.PNG)

这两种对象访问方式各有优势：

* 使用句柄来访问的最大好处是 reference 中存储的是**稳定的句柄地址**，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。

* 使用直接指针访问方式最大的好处就是**速度快**，它节省了一次指针定位的时间开销。 

### 堆内存中对象的分配的基本策略

堆内存的基本结构：

![堆内存的基本结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810436.PNG)

上图所示的 eden区、 s0区、 s1区都属于新生代， tentired 区属于老年代。大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会 +1(Eden区→Survivor 区后对象的初始年龄变为1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。 

另外，大对象和长期存活的对象会直接进入老年代

**堆内存分配常见策略**

* 对象优先在eden区分配
* 大对象直接进入老年代
* 长期存活的对象将进入老年代

### Minor GC & Full GC :exclamation:

大多数情况下，对象在新生代中 eden 区分配。当 eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC

* **新生代GC（Minor GC）**：指发生新生代的的垃圾收集动作， Minor GC 非常频繁，回收速度一般也比较快。
* **老年代GC（Major GC/Full GC）**：指发生在老年代的GC，出现了Major GC经常会伴随至少一次的 Minor GC（并非绝对）， Major GC的速度一般会比 Minor GC 的慢10倍以上

### 判断对象是否死亡

堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象）

 **引用计数法**：

* 给对象中添加一个引用计数器，每当有一个地方引用它，计数器就 +1；当引用失效，计数器就 -1；
* 当对象没有被引用了，可以被垃圾回收
* 存在**循环引用**问题：A对象引用B，B对象引用A，而没有其他对象引用AB对象了；但是AB对象的引用计数永不为0

**可达性分析算法**

* **根对象：肯定不能被当成垃圾回收的对象**
  * Java 虚拟机中的垃圾回收器采用可达性分析来探索所有存活的对象
  * 扫描堆中的对象，看是否能够沿着 GC Root 对象为起点的引用链找到该对象，找不到，表示可以回收；
* 哪些对象可以作为 GC Root ? 
  * `System Class`：由启动类加载器加载的系统类，如Object类、HashMap类、String类...
  * `Native Stack`：Java 虚拟机在执行一些方法时，会调用一些操作系统的方法，操作方法在执行时所引用的一些 Java 对象；
  * `Thread`：活动线程中使用的一些对象；
  * `Busy Monitor`：正在加锁的对象

### 强 & 软 & 弱 & 虚引用 :exclamation::exclamation:

无论是通过引用计数法判断对象引用数量，还是通过可达性分析法判断对象的引用链是否可达，判定对象的存活都与“引用”有关。 

**强引用(StrongReference) **

> 实例化一个类：`Person p = new Person()`
>
> 在等号的左边，就是一个对象的引用，**存储在栈中**
>
> 而等号右边，就是实例化的对象，**存储在堆中**
>
> 其实这样的一个引用关系，就被称为**强引用**

- 通过 new 创建一个对象，通过赋值运算符赋值给一个变量，变量就强引用了对象
- 只有所有 GC Roots 对象都不通过『强引用』引用该对象，该对象才能被垃圾回收
- 当内存不足的时候，JVM 开始垃圾回收，对于强引用的对象，就算是出现了 OOM 也不会对该对象进行回收，打死也不回收

**软引用（SoftReference）**

- 软引用是一种相对弱化了一些的引用，需要用`java.lang.ref.SoftReference`类来实现，可以让对象豁免一些垃圾收集，对于只有软引用的对象来讲

  - **当系统内存充足时，它不会被回收**
  - **当系统内存不足时，它会被回收**

- 可以配合引用队列来释放软引用自身

  > 引用队列：软引用的对象被被回收后，**软引用自身也是一个对象**，若在创建时为软引用分配了引用队列，则当软引用引用的对象被回收时，软引用会进入引用队列
  >
  > 在引用队列中依次遍历，释放引用对象所占用内存

- 软引用可用来实现内存敏感的高速缓存

  > 场景：假如有一个应用需要读取大量的本地图片
  >
  > - 如果每次读取图片都从硬盘读取则会严重影响性能
  > - 如果一次性全部加载到内存中，又可能造成内存溢出
  >
  > 此时使用软引用可以解决这个问题
  >
  > 设计思路：使用 HashMap 来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占的空间，从而有效地避免了 OOM 的问题
  >
  > ```java
  > Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
  > ```

**弱引用（WeakReference）**

- 仅有弱引用引用该对象时，**在垃圾回收时，无论内存是否充足，都会回收弱引用对象**
- 弱引用需要用 `java.lang.ref.WeakReference` 类来实现，它比软引用生存期更短
- 当然也可以配合引用队列来释放弱引用自身

**虚引用（PhantomReference）**

* 虚引用又称为幽灵引用，需要`java.lang.ref.PhantomReference` 类来实现

- 顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期
- **如果一个对象持有虚引用，那么它就和没有任何引用一样**，在任何时候都可能被垃圾回收器回收，**虚引用不能单独使用也不能通过它访问对象，虚引用必须和引用队列ReferenceQueue联合使用**，引用对象回收时，会将虚引用入队，由 Reference Handler 线程调用虚引用相关方法释放直接内存

> **虚引用主要用来跟踪对象被垃圾回收的活动**
>
>  虚引用与软引用和弱引用的一个区别在于： 虚引用必须和引用队（ReferenceQueue）联合使用。当 GC 准备回收一个对象时，如果发现它还有虚引用，就会**在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是 否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。**程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。
>
> 特别注意，在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为软引用可以加速 JVM 对垃圾内存的回收速度，还可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生。 

### 判断一个类是无用的类

方法区主要回收的是无用的类，那么如何判断一个类是无用的类的呢？ 

类需要同时满足下面**3个条件**才能算是 “无用的类” ： 

* 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
* 加载该类的 ClassLoader（类加载器） 已经被回收。
* 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。 

虚拟机可以对满足上述3个条件的无用类进行回收，**这里说的仅仅是“可以”**，而并不是和对象一样不使用了就会必然被回收。 

### 垃圾收集算法及特点 :exclamation::exclamation::exclamation:

**标记清除 Mark Sweep **

![标记和清除](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810265.PNG)

* 分为标记和清除两个步骤
  * **标记**：标记出可以被清除的对象；
  * **清除**：把垃圾对象占用的空间清除，此处的清除为**把对象占用的内存起始结束地址进行记录**，在下次分配对象地址时可以分配到这些地址中；
* **优点：**速度快，清除时**只需要记录**垃圾对象的起始结束地址即可；
* **缺点：**容易产生内存碎片

**标记整理 Mark Compact** 

![标记整理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810038.PNG)

* 标记整理也分为两个步骤：
  * **标记**：标记出可以被清除的对象；
  * **整理**：为了避免标记清除时产生的内存碎片问题，在清理的过程中，对对象进行整理，使得对象在内存空间中更为紧凑。
* **优点**：不会产生内存碎片；
* **缺点**：在整理过程中，涉及到了对象的移动，因此速度较慢。

**复制拷贝 Copy**

![复制](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810484.PNG)

* 将内存区划分了内存大小相同的两块区域；

  * 在 FROM 区中进行一次标记处理；

  * 将 FROM 区中存活的对象复制到 TO 区域中，复制过程中完成对象的整理；

  * 在复制完成后，清空 FROM 区域，并交换 FROM 和 TO 区的位置

* **优点：**不会产生内存碎片

* **缺点：**需要使用双倍的内存空间

**分代垃圾回收**

> 见下面

### 分代垃圾回收:exclamation::exclamation:

将堆内存分为两块区域：**新生代**与**老年代**；

**分代垃圾回收工作机制**：

- 创建的新对象首先分配在新生代中的伊甸园区，之后更多对象被创建，放入伊甸园中；

- 新生代空间不足时，触发 `Minor GC`，伊甸园和From存活的对象使用**copy复制**到to中，存活的对象年龄+1并且交换from和to；

  > `Minor GC` 会引发 `stop the world`，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行，这是出于在GC时，涉及到了对象地址的修改；

- 触发`Minor GC`后，伊甸园有空间了，继续将生成的对象放入，直至再次发生`Minor GC`进行相应操作；

- 当幸存区中对象的寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit）；

- 当老年代空间不足，会先尝试触发 `Minor GC`，如果之后空间仍不足，那么触发 `Full GC`

  > `Full GC` 的 `stop the world`的时间更长

- 若触发 `Full GC`后，老年代的空间还是不足，则会导致堆内存溢出。

**这样划分的理由**：

* 有些需要长时间使用的对象放置老年代中；将那些用完后不需要再使用的对象放置于新生代中；

- 这样可**针对对象生命周期不同的特点，选择不同的垃圾回收策略**，新生代中的垃圾回收发生较为频繁，可以使用复制算法；老年代垃圾回收不太频繁，可以使用标记整理算法；在一定程度上可以提升 GC 的效率
  - **新生代使用复制算法**：因为新生代对象的**生存时间比较短**，80%的都要回收的对象，采用标记-清除算法则内存碎片化比较严重，采用复制算法可以灵活高效，且便于整理空间。
  - **老年代采用标记整理**：标记整理算法主要是为了解决标记清除算法存在内存碎片的问题，又解决了复制算法两个Survivor区的问题，因为老年代的空间比较大，不可能采用特别占用内存空间的复制算法。

![分代垃圾回收](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810364.PNG)

### 常见的垃圾回收器 :exclamation::exclamation::exclamation:

**如果说垃圾回收算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。** 

> 虽然我们对各个收集器进行比较，但并非要挑选出一个最好的收集器。因为直到现在为止还没有最好的垃圾收集器出现，更加没有万能的垃圾收集器，我们能做的就是根据具体应用场景选择适合自己的垃圾收集器。试想一下：如果有一种四海之内、任何场景下都适用的完美收集器存在，那么我们的 HotSpot 虚拟机就不会实现那么多不同的垃圾收集器了。 

**串行垃圾回收 / Serial：**

![串行](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810206.PNG)

* **工作流程**：
  * 触发 GC 时，使所有线程在一个**安全点**暂停，这是出于在 GC 时，对象的地址可能会发生修改；
  * Serial，SerialOld 都是单线程的垃圾回收器，因此当垃圾回收器线程在运行时，其他线程进入阻塞状态；
* **特点**：
  * **串行收集器是最古老，最稳定以及效率高的收集器**，但是只使用一个线程去回收使其在垃圾收集过程中可能会产生较长的停顿 ( Stop-The-World 状态)。
  * **优点**：虽然在收集垃圾过程中需要暂停所有其它的工作线程，但是它简单高效，对于限定单个 CPU 环境来说，没有线程交互的开销可以获得最高的单线程垃圾收集效率，因此 Serial 垃圾收集器依然是 Java 虚拟机运行在 Client 模式下默认的垃圾收集器
* **适用场景**：单线程、堆内存较小，适合个人电脑，即 CPU 个数少
* **配置方式**：打开**串行垃圾回收器**的 JVM 参数：`-XX:+UseSerialGC = Serial + SerialOld `
  - **Serial：用于新生代，采用复制垃圾回收算法；**
  - **SerialOld：用于老年代，采用标记整理算法；**

**并行垃圾回收器 / 吞吐量优先 / Parallel**

* **工作流程**：

  * 触发GC时，使所有线程在一个**安全点**暂停；

  ![吞吐量优先](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810395.PNG)

  * 垃圾回收器**开启多个线程**进行垃圾回收，垃圾回收线程数默认情况下与 CPU 核心个数相关
  * 使用并行的垃圾回收器，**一句话：串行垃圾收集器在新生代和老年代的并行化**
  
* **使用场景：**多线程、堆内存较大，多个CPU、让**单位时间内 STW（stop the world）的时间最短**，垃圾回收时间占比最低，这样就称**吞吐量高**

  > 可控制的吞吐量（Thoughput = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)），比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99%。高吞吐量意味着高效利用CPU时间，它多用于在后台运算而不需要太多交互的任务。

* **配置方式：**需要设置的 JVM 参数：`-XX:+UseParallelGC ~ -XX:+UseParallelOldGC `：

  - ``-XX:+UseParallelGC`：**新生代的并行垃圾回收器，使用复制算法**
  - `-XX:+UseParallelOldGC `：**老年代的并行垃圾回收器，使用标记-整理算法**

  > 在JDK1.8中默认已经开启，**开启了一个顺带也会开启另一个**

  - `-XX:+UseAdaptiveSizePolicy`：使用自适应新生代大小调整策略

  - `-XX:GCTimeRatio=ratio` ：用于调整垃圾回收时间与总时间占比

    > 公式：`1/(1+ratio)`
    >
    > ratio默认为99，即垃圾回收的时间不能超过总时间的1%

  - `-XX:MaxGCPauseMillis=ms` ：最大暂停时间，默认为 200ms

  - `-XX:ParallelGCThreads=n` ：控制垃圾回收线程数

**并发标记清除 / CMS (Concurrent Mark Sweep) / 响应时间优先**

* **CMS的流程**

  ![响应时间优先](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810187.PNG)
  * 多线程运行，老年代发生了内存不足，多个线程到达安全点停止运行；
  * **初始标记阶段（CMS initial mark）**：CMS 垃圾回收器开始工作，执行一个**初始标记**的动作(只标记根对象，很快)，**期间需要STW**；
  * **并发标记（CMS concurrent mark）和用户线程一起**：用户线程可以恢复运行，与此同时垃圾回收线程**并发标记**，进行 GC Roots 跟踪过程，找出剩余的垃圾对象；
  * **重新标记（CMS remark）**：接下来进行**重新标记，需要STW**，因为之前并发执行时，会对垃圾回收的对象造成干扰，因此要做重新标记工作；
  * **并发清除（CMS concurrent sweep）和用户线程一起**：重新标记完成后，用户线程恢复运行，垃圾回收线程做一次**并发的清理**。

* **特点**：

  * 由于**耗时最长的并发标记和并发清除**过程中，垃圾收集线程可以和用户现在一起并发工作，所以**总体上来看CMS收集器的内存回收和用户线程是一起并发地执行**。

  * **优点：并发收集低停顿**

  * **缺点：并发执行，对CPU资源压力大，采用的标记清除算法会导致大量内存碎片**

    由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说，**CMS必须在老年代堆内存用尽之前完成垃圾回收**，否则CMS回收失败时，将触发**担保机制**，串行老年代收集器将会以STW方式进行一次GC，从而造成较大的停顿时间

    **同时标记清除算法无法整理空间碎片**，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩，CMS也提供了参数 `-XX:CMSFullGCSBeForeCompaction`（默认0，即每次都进行内存整理）来指定多少次 CMS 收集之后，进行一次压缩的 Full GC
  
*  **使用场景**：多线程、堆内存较大，多个CPU、**尽可能让单次 STW 的时间最短**

*  **配置方式**：需要设置的 JVM 参数：

   - `-XX:+UseConcMarkSweepGC ~ -XX:+UseParNewGC ~ -XX:+SerialOld`

     > Conc->concurrent**并发的**，即用户线程和GC线程是并发执行的	
     >
     > MarkSweepGC一款基于**标记清除的垃圾回收器**

     - **`-XX:+UseConcMarkSweepGC`：工作于老年代的一款垃圾回收器**
     - **`-XX:+UseParNewGC`：工作于新生代的垃圾回收器**，并行收集器，使用多线程进行垃圾回收，在垃圾收集，会 Stop-the-World 暂停其他所有的工作线程直到它收集结束
     - **`-XX:+SerialOld`：并发失败**会退回到**单线程的垃圾回收器**，例如内存碎片过多导致并发失败

   - `-XX:ParallelGCThreads=n ~ -XX:ConcGCThreads=threads`

     - `-XX:ParallelGCThreads=n`：**并行**垃圾回收线程数，一般与CPU核数相同；
     - `-XX:ConcGCThreads=threads`：**并发**的垃圾回收线程数，CMS 默认启动的并发线程数是（ParallelGCThreads+3）/ 4

   - `-XX:CMSInitiatingOccupancyFraction=percent`

     - `CMSInitiatingOccupancyFraction`：执行CMS垃圾回收的内存占比；
     - 例如设置为 80%，老年代内存占用达到 80% 时，就执行一次GC；
     - 考虑到：在并发清理时会产生一些新的垃圾，称为**浮动垃圾**，而并发清理时不能清理这些浮动垃圾，这些垃圾需要在下一次垃圾清理是被清理，**因此需要预留空间对浮动垃圾进行保存**

   - `-XX:+CMSScavengeBeforeRemark`

     - 在重新标记阶段，会出现新生代的对象会引用老年代的对象的情况；
     - 而在重新标记时，会对整个堆进行扫描，而在对新生代做可达性分析时，**若新生代对象过多，会对性能造成影响**；
     - 使用上述参数**先对新生代做一个GC**，则在重新标记时扫描的对象会减少，可以提升性能。

**G1 - (Garbage One)**

* **适用场景：**
  * 同时注重吞吐量（Throughput）和低延迟（Low latency），默认的暂停目标是 200 ms
  * **超大堆内存，会将堆划分为多个大小相等的 Region**
  * 整体上是 **标记+整理** 算法，两个区域 Region 之间是 **复制** 算法 

* **配置方式**：
  * `-XX:+UseG1GC`：JDK1.8中需要手动打开，JDK1.9之后为默认
  * `-XX:G1HeapRegionSize=size`：设置 Region 的大小，只能设置为1,2,4,8,16... MB 等大小
  * `-XX:MaxGCPauseMillis=time` ：设置暂停目标

### G1 垃圾收集器 :exclamation::exclamation:

开启 G1 垃圾收集器 JVM 参数：`-XX:+UseG1GC`

> **以前收集器的特点**
>
> - 年轻代和老年代是各自独立且连续的内存块
> - 年轻代收集使用 **单eden + S0 + S1** 进行复制算法
> - 老年代收集必须扫描整个老年代区域
> - 都是以尽可能少而快速地执行 GC 为设计原则

**G1 是什么**

G1：Garbage-First 收集器，是一款面向服务端应用的垃圾收集器，应用在多处理器和大容量内存环境中，**在实现高吞吐量的同时，尽可能满足垃圾收集暂停时间的要求**。另外，它还具有以下特征：

- 像 CMS 收集器一样，能与应用程序**并发执行**

* G1 收集器设计目标是取代 CMS 收集器，它同 CMS 相比，在**以下方面表现的更出色**
  * G1 是一个**有整理内存过程**的垃圾收集器，**整体上采用标记-整理算法，局部是通过复制算法，不会产生内存碎片**
  * G1 的 Stop The World（STW）更可控，G1 **在停顿时间上添加了预测机制**，用户可以指定期望停顿时间。

- G1 把**内存划分成多个独立的子区域（Region）**，宏观上看 G1 之中不再区分年轻代和老年代，但其本身依然在小范围内要进行年轻代和老年代的区分，保留了新生代和老年代，将一部分 Region 划分为新生代，一部分 Region 划分为 老年代，**不再是物理隔离的也不需要是连续的**，**G1 只有逻辑上的分代概念，或者说每个分区都可能随 G1 的运行在不同代之间前后切换**。

**底层原理**

* Region 区域化垃圾收集器，化整为零，打破了原来新生区和老年区的壁垒，**避免了全内存扫描，只需要按照区域来进行扫描即可**。

* **核心思想是将整个堆内存区域分成大小相同的子区域（Region），在 JVM 启动时会自动设置子区域大小**

  > 在堆的使用上，**G1 并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换**。启动时可以通过参数`-XX:G1HeapRegionSize=n` 可指定分区大小（1MB~32MB，且必须是2的幂），默认将整堆划分为2048个分区。
  >
  > 大小范围在1MB~32MB，最多能设置 2048 个区域，也即能够支持的最大内存为：32MB*2048 = 64G内存

**Region 区域化垃圾收集器**

* G1 将新生代、老年代的物理空间划分取消了，同时对内存进行了区域划分

  <img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810163.png" alt="g1空间划分" style="zoom:50%;" />

* G1 算法将堆划分为若干个区域（Reign），它仍然属于分代收集器，这些 Region 的一部分包含新生代，**新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或者 Survivor 空间**

* 这些 Region 的一部分包含老年代，G1 收集器通过将对象从一个区域复制到另外一个区域完成了清理工作。这就意味着，**在正常的处理过程中，G1完成了堆的压缩（至少是部分堆的压缩），这样也就不会有 CMS 内存碎片的问题存在了**。

* 在 G1 中，还有一种特殊的区域，**叫做 Humongous（巨大的）区域**，如果一个对象占用了空间超过了分区容量50% 以上，G1收集器就认为这是一个巨型对象，这些巨型对象默认直接分配在老年代，但是如果他是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响，为了解决这个问题，G1 划分了一个 Humongous 区，它用来专门存放巨型对象。如果一个 H区 装不下一个巨型对象，那么 G1 会寻找连续的 H区 来存储，为了能找到连续的 H区，有时候不得不启动Full GC。

**回收步骤**：针对Eden区进行收集，Eden区耗尽后会被触发，主要是小区域收集 + 形成连续的内存块，避免内碎片

- Eden 区的数据移动到 Survivor 区，加入后出现 Survivor 区空间不够，Eden 区数据会晋升到Old区
- Survivor区的数据移动到新的Survivor区，部分数据晋升到Old区
- 最后Eden区收拾干净了，GC结束，用户的应用程序继续执行
- **小区域收集 + 形成连续的内存块**，最后在收集完成后，就会形成连续的内存空间，这样就解决了内存碎片的问题

**四步过程**

- **初始标记**：只标记GC Roots能直接关联到的对象
- **并发标记**：进行GC Roots Tracing（链路扫描）的过程
- **最终标记**：修正并发标记期间，因为程序运行导致标记发生变化的那一部分对象
- **筛选回收**：根据设定的停顿时间来进行价值最大化回收

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810915.png" alt="g1收集器运行示意图" style="zoom: 67%;" />

**参数配置**

* 开发人员仅仅需要申明以下参数即可
* 三步归纳：`-XX:+UseG1GC -Xmx32G -XX:MaxGCPauseMillis=100`
* `-XX:MaxGCPauseMillis=n`：最大GC停顿时间单位毫秒，这是个**软目标**，JVM尽可能停顿小于这个时间

### 垃圾收集器组合 & 选择 :exclamation:

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810505.png" alt="垃圾收集器组合关系" style="zoom:50%;" />

**Java中一共有7大垃圾收集器**

1. `-XX:UserSerialGC`：串行垃圾收集器

2. `-XX:UserParallelGC`：并行垃圾收集器

3. `-XX:UseConcMarkSweepGC`：（CMS）并发标记清除垃圾收集器

4. `-XX:UseParNewGC`：年轻代的并行垃圾回收器

   > CMS相结合

5. `-XX:UseParallelOldGC`：老年代的并行垃圾回收器

6. `-XX:UseG1GC`：G1垃圾收集器，横跨新生代与老年代

7. `-XX:UserSerialOldGC`：串行老年代垃圾收集器（已经被移除）

**垃圾收集器如何选择​**（其实在上述介绍不同的垃圾收集器是已经指出了其适用场景）

* 单CPU或者小内存，单机程序：`-XX:+UseSerialGC` 开启

* 多CPU，**需要最大的吞吐量**，如后台计算型应用
  * `-XX:+UseParallelGC`
  * `-XX:+UseParallelOldGC`

  > 上述两个参数互相激活

* 多CPU，**追求低停顿时间**，需要快速响应如互联网应用
  * `-XX:+ParNewGC`
  * `-XX:+UseConcMarkSweepGC`

**总结**

| 参数                      | 新生代垃圾收集器         | 新生代算法 | 老年代垃圾收集器                                             | 老年代算法                            |
| ------------------------- | ------------------------ | ---------- | ------------------------------------------------------------ | ------------------------------------- |
| `-XX:+UseSerialGC`        | SerialGC                 | 复制       | SerialOldGC                                                  | 标记整理                              |
| `-XX:+UseParNewGC`        | ParNew                   | 复制       | SerialOldGC                                                  | 标记整理                              |
| `-XX:+UseParallelGC`      | Parallel [Scavenge]      | 复制       | Parallel Old                                                 | 标记整理                              |
| `-XX:+UseConcMarkSweepGC` | ParNew                   | 复制       | CMS + Serial Old的收集器组合<br />Serial Old作为CMS出错的后备收集器 | CMS:标记清除<br />Serial Old:标记整理 |
| `-XX:+UseG1GC`            | G1整体上采用标记整理算法 | 局部复制   | G1整体上采用标记整理算法                                     | 局部复制                              |

### 类文件结构

根据 JVM 规范，类文件结构如下

```java
ClassFile{
    //  u4/u2 表示字节数
	u4 					magic;  		// 魔术，Class文件的标志
	u2 					minor_version;  // 版本，Class的小版本号
	u2 					major_version;  // 版本，Class的大版本号
	u2 					constant_pool_count;  				   // 常量池的数量
	cp_info 			constant_pool[constant_pool_count-1];  // 常量池信息
	u2 					access_flags;      // Class的访问修饰
	u2 					this_class;        // 当前类信息
	u2 					super_class;	   // 父类信息
	u2 					interfaces_count;  // 接口数量
	u2 					interfaces[interfaces_count];  // 接口信息，一个类可以实现多个接口
	u2 					fields_count;			// 类中的成员变量信息
	field_info  		fields[fields_count];	// 类中的成员变量信息，类中可以有多个成员变量
	u2 					methods_count;			// 类中的方法数量
	method_info 		methods[methods_count];	// 类中的方法信息，类中可以有多个方法
	u2 					attributes_count;			   // 类的附加属性信息
	attribute_info 		attributes[attributes_count];  // 类的附加属性信息
}
```

**Class文件字节码结构组织示意图**

![class文件字节码结构组织示意图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810309.PNG)

下面会按照上图结构按顺序详细介绍一下 Class 文件结构涉及到的一些组件

1. 魔数：确定这个文件是否为一个能被虚拟机接收的 Class 文件。
2. Class 文件版本：Class 文件的版本号，保证编译正常执行。
3. 常量池：常量池主要存放两大常量——**字面量**和**符号引用**。
4. 访问标志：标志用于识别一些类或者接口层次的访问信息，包括：这个 Class 是类还是接口，是否为 public 或者 abstract 类型，如果是类的话是否声明为 final 等。
5. 当前类索引 & 父类索引：**类索引用于确定这个类的全限定名**，**父类索引用于确定这个类的父类的全限定名**，由于 Java 语⾔的单继承，所以父类索引只有一个，除了 java.lang.Object 之外，所有的 java 类都有父类，因此除了 java.lang.Object 外，所有 Java 类的父类索引都不为 0。
6. 接口索引集合：接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按 implements (如果这个类本身是接口的话则是 extends ) 后的接口顺序从左到右排列在接口索引集合中。

7. 字段表集合：描述接口或类中声明的变量，字段包括类级变量以及实例变量，但不包括在方法内部声明的局部变量。
8. 方法表集合：类中的方法。
9. 属性表集合：在 Class 文件，字段表，方法表中都可以携带自己的属性表集合。 

### 类加载过程概述 :exclamation::exclamation:

类加载过程：**加载→连接→初始化**。连接过程又可分为三步：**验证→准备→解析**

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810459.PNG" alt="类加载过程" style="zoom:50%;" />

**在类加载阶段**：

1. **将类的字节码载入方法区中**，如果这个类还有父类没有加载，先加载父类（**这里就有个类加载器**）
2. 加载和连接可能是交替运行的 

**在连接阶段**：连接过程又可分为三步：验证→准备→解析

1. **验证**：验证加载的**class类是否符合 JVM 规范**，安全性检查
2. **准备**：为 static 变量 (类变量) 分配空间，设置默认值
   * static 变量在 JDK 7 之前存储于 `instanceKlass` 末尾，从 JDK 7 开始，存储于 `_java_mirror` 末尾
   * **基本static变量**：分配空间和赋值是两个步骤，**分配**空间在**准备阶段**完成，**赋值**在**初始化阶段**完成
   * **final static 基本类型/字符串常量：值在编译阶段值就确定了，赋值在准备阶段完成**
   * **final static 引用类型：赋值也会在初始化阶段完成**
3. **解析**：**将常量池中的符号引用解析为直接引用** 

**初始化阶段**：初始化即调用由（ `<cinit>()V`，虚拟机会保证这个类的『构造方法』的线程安全

* `<cinit>()V`：收集所有 static 静态代码块和静态成员赋值的代码，**合并**为一个特殊的方法 `<cinit>()V` ： 

> 对类加载阶段的进一步说明：方法区中采用 C++ 的 `instanceKlass` 描述 java 类，它的重要 field 有：
>
> * `_java_mirror` 即 java 的类镜像，例如对 String 来说，就是 String.class，作用是把 klass 暴露给 java 使用
> * `_super` 即父类
> * `_fields` 即成员变量
> * `_methods` 即方法
> * `_constants` 即常量池
> * `_class_loader` 即类加载器
> * `_vtable` 虚方法表
> * `_itable` 接口方法表
>
> ![类加载](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810667.PNG)

### 类加载器 :exclamation::exclamation:

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自 `java.lang.ClassLoader`： 

以 JDK 8 为例： 

| 名称                                      | 负责加载哪的类          | 说明                     |
| ----------------------------------------- | ----------------------- | ------------------------ |
| 启动类加载器：Bootstrap ClassLoader       | `JAVA_HOME/jre/lib`     | 无法直接访问，显示为null |
| 扩展类加载器：Extension ClassLoader       | `JAVA_HOME/jre/lib/ext` | 上级为 Bootstrap         |
| 应用程序类加载器：Application ClassLoader | `classpath`             | 上级为 Extension         |
| 自定义类加载器                            | 自定义                  | 上级为 Application       |

1. **Bootstrap ClassLoader(启动类加载器)**：最顶层的加载类，由 C++ 实现，负责加载 `%JAVA_HOME%/jre/lib` 目录下的 jar 包和类或者或被 `-Xbootclasspath` 参数指定的路径中的所有类。 
2. **ExtensionClassLoader(扩展类加载器)**：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的 jar 包。
3. **AppClassLoader(应用程序类加载器)**：面向我们用户的加载器，负责加载当前应用 classpath 下的所有jar包和类。 

### 双亲委派模型 :exclamation::exclamation::exclamation:

所谓的双亲委派，就是指**调用类加载器 loadClass 方法时查找类的规则：先由上级完成类的加载，若上级没有这个类，则由本级的类加载器来完成类的加载**

> 注意：这里的双亲，翻译为上级似乎更为合适，因为它们并没有继承关系

每一个类都有一个对应它的类加载器。系统中的 ClassLoder 在协同工作的时候会**默认使用 双亲委派模型**。来实现：**自底向上检查类是否被加载，然后自顶向下尝试加载类**，进一步：

1. 即在类加载的时候，系统会首先判断当前类是否被加载过。**已经被加载的类会直接返回，否则才会尝试加载**。 
2. 加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，**因此所有的请求最终都应该传送到顶层的启动类加载器 BootstrapClassLoader 中**。
3. 当父类加载器无法处理时，才由自己来处理。 当父类加载器为 null 时，会使用启动类加载器 BootstrapClassLoader 作为父类加载器。 

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251810520.PNG" alt="双亲委派模式" style="zoom:50%;" />

**双亲委派模型带来了什么好处呢？** 

1. 双亲委派模型保证了Java程序的稳定运行，可以**避免类的重复加载**（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类）；
2. **保证了 Java 的核心 API 不被篡改**。如果不用没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 java.lang.Object 类的话，那么程序运行的时候，系统就会出现多个不同的 Object 类。 

> 每个类加载都有一个父类加载器，我们通过下面的程序来验证。 
>
> ```java
> public class Demo {
> 
>        public static void main(String[] args) {
>         System.out.println("Demo's ClassLoader is:");
>            System.out.println("  "+Demo.class.getClassLoader());
>            System.out.println("Demo's Parent ClassLoader is:");
>            System.out.println("  "+Demo.class.getClassLoader().getParent());
>            System.out.println("Demo's GrandParent ClassLoader is:");
>            System.out.println("  "+Demo.class.getClassLoader().getParent().getParent());
>        }
>    }
> 
> // 输出：
> // Demo's ClassLoader is:
> //    sun.misc.Launcher$AppClassLoader@18b4aac2
>    //  Demo's Parent ClassLoader is:
>    //    sun.misc.Launcher$ExtClassLoader@1b6d3586
>    //  Demo's GrandParent ClassLoader is:
>    //    null  // 这个null就是Bootstrap ClassLoader
>    ```
> 
>可以体现出如下：
> 
>- AppClassLoader 的父类加载器为 ExtClassLoader；
> 
>* ExtClassLoader 的父类加载器为null；
> * null 并不代表 ExtClassLoader 没有父类加载器，而是 Bootstrap ClassLoader 。 

**双亲委派模型实现源码分析**

```java
protected Class<?> loadClass(String name, boolean resolve) 
    throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // 1. 检查该类是否已经加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {  // 没加载则去加载类
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 2. 有上级的话，委派上级 loadClass
                    c = parent.loadClass(name, false);
                } else {
                    // 3. 如果没有上级了（ExtClassLoader），则委派BootstrapClassLoader 
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
            } 
            if (c == null) {
                long t1 = System.nanoTime();
                // 4. 每一层找不到，调用findClass方法（每个类加载器自己扩展）来加载
                c = findClass(name);
                // 5. 记录耗时
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        } 
        if (resolve) {
            resolveClass(c);
        } 
        return c;
    }
}
```

例如以下代码：

```java
public class Demo {
    public static void main(String[] args) throws ClassNotFoundException {
        System.out.println(Demo.class.getClassLoader());
        // 通过loadClass加载cn.xyc.H类
        Class<?> aClass = Demo.class.getClassLoader().loadClass("cn.xyc.H");
        System.out.println(aClass.getClassLoader());
    }
}
```

1. `sun.misc.Launcher$AppClassLoader //1` 处， 开始查看已加载的类，结果没有，即为null；
2. `sun.misc.Launcher$AppClassLoader // 2 `处，委派上级`sun.misc.Launcher$ExtClassLoader.loadClass() `
3. `sun.misc.Launcher$ExtClassLoader // 1` 处，查看已加载的类，结果没有；
4. `sun.misc.Launcher$ExtClassLoader // 2 `处，委派上级，但是其上级为null，表示已经到启动类加载器了，进入`findBootstrapClassOrNull(name);  // 3`处，委托启动类加载器去查找；
5. `BootstrapClassLoader` 是在 `JAVA_HOME/jre/lib` 下找 H 这个类，显然没有；
6. `sun.misc.Launcher$ExtClassLoader // 4` 处，调用自己的 findClass 方法，是在`JAVA_HOME/jre/lib/ext` 下找 H 这个类，显然没有， 回到 `sun.misc.Launcher$AppClassLoader的 // 2` 处
7. 继续执行到 `sun.misc.Launcher$AppClassLoader // 4` 处，调用它自己的 findClass 方法，在classpath下查找，找到了 

### 打破双亲委派模式 :exclamation:

**如果我们不想用双亲委派模型怎么办？** 记录两种破坏双亲委派模式的方式：

**方式1**：为了避免双亲委派机制，我们可以自己**自定义一个类加载器，然后重写 `loadClass()` 方法即可**。 

**如何自定义类加载器?** 

除了 `BootstrapClassLoader` 其他类加载器均由 Java 实现且全部继承自 `java.lang.ClassLoader` 。如果我们要自定义自己的类加载器，很明显需要继承 ClassLoader。 

* **什么时候需要自定义类加载器**
  
  * 想加载非 classpath 随意路径中的类文件
  * 都是通过接口来使用实现，希望解耦时，常用在框架设计
* 这些类希望予以隔离，不同应用的同名类都可以加载，不冲突，常见于 tomcat 容器 
  
* **步骤**： 

  1. 继承 ClassLoader 父类

  2. 若要遵从双亲委派机制，**重写 findClass 方法**

     * 注意不是重写 loadClass 方法，**否则不会走双亲委派机制**，当然要打破双亲委派的需要重写 loadClass 方法

  3. 读取类文件的字节码 bytes

  4. 调用父类的 defineClass 方法来加载类

     > `defineClass(name, bytes, 0, bytes.length);`
     >
     > * param1：类名字
     > * param2：类文件字节码

  5. 使用者调用该类加载器的 loadClass 方法即可完成类的加载

  > 具体实现可见JVM对应的笔记内容

**方式2：SPI（Service Provider Interface，服务提供者接口）**

这里以 JDBC 加载 Driver 驱动为例进行说明：

1. 在使用 JDBC 时，通过 `Class.forName("com.mysql.jdbc.Driver")` 加载驱动，但是不写时也能正常加载驱动；

2. 查看 `DriverManager` 源码，可见其内部有一个静态代码块，内有函数 `loadInitialDrivers();` 通过其加载 Driver

3. 但是注意到，`DriverManager` 类加载器为`Bootstrap ClassLoader`，会到 `JAVA_HOME/jre/lib` 下搜索类，但该路径下必然不会存在 mysql 的 jar 包，那`DriverManager`的静态代码块中是如何加载 `com.mysql.jdbc.Driver ` 呢？

4. 查看 `loadInitialDrivers()` 方法

   ```java
   private static void loadInitialDrivers() {
       String drivers;
       try {
           drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
           	public String run() {
                   return System.getProperty("jdbc.drivers");
               }
   		});
       } catch (Exception ex) {
           drivers = null;
       }
       // 1) 使用 ServiceLoader 机制加载驱动，即SPI
       AccessControler.doPrivileged(new PrivilegedAction<Void>)(){
           public Void run() {
               ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
               Iterator<Driver> driversIterator = loadedDrivers.iterator();
               try{
                   while(driversIterator.hasNext()) {
                       driversIterator.next();
                   }
               } catch(Throwable t) {
                   // Do nothing
               } 
               return null;
           }
       });
       println("DriverManager.initialize: jdbc.drivers = " + drivers);
       
       // 2）使用 jdbc.drivers 定义的驱动名加载驱动，这也打破了双亲委派模式
       if (drivers == null || drivers.equals("")) {
           return;
       } 
       String[] driversList = drivers.split(":");
       println("number of Drivers:" + driversList.length);
       for (String aDriver : driversList) {
           try {
               println("DriverManager.Initialize: loading " + aDriver);
               // 这里的 ClassLoader.getSystemClassLoader() 就是应用程序类加载器
               Class.forName(aDriver, true, ClassLoader.getSystemClassLoader());
           } catch (Exception ex) {
               println("DriverManager.Initialize: load failed: " + ex);
           }
       }
   }
   ```

5. **SPI：通过 SPI 打破双亲委派模式**，使用 SPI 约定如下，在 jar 包的 META-INF/services 包下，以接口全限定名为文件名，文件内容是接口实现类的名称

   > 如 mysql-connector-java-xxx.jar 包，下建立目录 META-INF/services；
   >
   > 以接口全限定类名作为文件名：`java.sql.Driver`
   >
   > 文件内容就是接口的实现类：`com.mysql.jdbc.Driver`、...

6. SPI 使用：通过 `ServiceLoader.load(接口类型.class)` 方法来实现上述接口实现类

   ```java
   ServiceLoader<接口类型> allImpls = ServiceLoader.load(接口类型.class);
   Iterator<接口类型> iter = allImpls.iterator();
   while(iter.hasNext()) {
       iter.next();
   }
   ```

7. 接着看 `ServiceLoader.load` 方法

   ```java
   public static <S> ServiceLoader<S> load(Class<S> service) {
       // 获取线程上下文类加载器
       ClassLoader cl = Thread.currentThread().getContextClassLoader();
       return ServiceLoader.load(service, cl);
   }
   ```

   **线程上下文类加载器**是当前线程使用的类加载器，**默认就是应用程序类加载器**，它内部又是由 `Class.forName` 调用了线程上下文类加载器完成类加载，具体代码在 `ServiceLoader` 的内部类 `LazyIterator` 中，不再展开。

### JVM 调优 :exclamation:

> [JVM：参数调优](https://www.cnblogs.com/zhuchengchao/p/14518194.html)

**JVM 参数类型**

* 标配参数（从JDK1.0 之后都在，很稳定）

  * `java -version`
  * `java -help`
  * `java -showversion`

* X 参数（了解）

  * `-Xint`：解释执行
  * `-Xcomp`：第一次使用就编译成本地代码
  * `-Xmixed`：混合模式

* **XX参数（重点）**

  * Boolean类型

    - 公式：`-XX:+/-某个属性`

      > +表示开启 / -表示关闭

    - 如：`-XX:-PrintGCDetails`：表示关闭了GC详情输出

  * key-value类型

    - 公式：`-XX:属性key=属性value`
    - 不满意初始值，可以通过相应命令调整
    - 如：`-XX:MetaspaceSize=21807104`，调整Java元空间的值

**常用 JVM 基本配置参数**

* **堆内存**相关参数

  * **`-Xms` 初始堆内存为：物理内存的1/64** 
  * **`-Xmx` 最大堆内存为：系统物理内存的1/4**

* 打印 JVM 默认参数：`-XX:+PrintCommandLineFlags` 打印出 JVM 的默认的简单初始化参数

* 一些常用调优参数

  * `-Xms`：初始化堆内存，默认为物理内存的1/64，等价于 `-XX:initialHeapSize`
  * `-Xmx`：最大堆内存，默认为物理内存的1/4，等价于 `-XX:MaxHeapSize`
  * `-Xss`：设计单个线程栈的大小，一般默认为512K~1024K，等价于 `-XX:ThreadStackSize`
  * `-XX:MetaspaceSize`：设置元空间大小
    * 元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现，不过元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**，因此，默认情况下，元空间的大小仅受本地内存限制。
    * 但是默认的元空间大小：只有20多M `-XX:MetaspaceSize=21807104`
    * 为了防止在频繁的实例化对象的时候，让元空间出现OOM，因此可以把元空间设置的大一些：`-Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX:+PrintCommandLineFlags`

* 垃圾回收相关参数

  * `-XX:PrintGCDetails`：输出详细GC收集日志信息

  * `-Xmn`：设置年轻代大小

  * `-XX:SurvivorRatio`：调节新生代中 eden 和 S0、S1 的空间比例

    > 调节新生代中 eden 和 from、to 的空间比例，默认为 `-XX:SurvivorRatio=8`，即：`Eden:from:to = 8:1:1`，如下图：

  * `-XX:NewRatio`：配置年轻代 new 和老年代 old 在堆结构的占比，NewRadio 值就是设置老年代的占比

    > 默认：`-XX:NewRatio=2` 新生代占1，老年代2，年轻代占整个堆的1/3

  * `-XX:MaxTenuringThreshold`：设置对象进入老年代的阈值

    > 默认是15，并且在 Java8 中规定该值在 0~15之间