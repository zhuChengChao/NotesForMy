

# Java：NIO 学习笔记-3

> 对比学习！之前看了下 [尚硅谷](https://space.bilibili.com/302417610) 中关于 IO 内容的讲解，此处又看了下 [黑马程序员](https://space.bilibili.com/37974444/) 中的类似课程：[JAVA通信架构I/O模式](https://www.bilibili.com/video/BV1kT4y1M7vt)，同样也根据课程内容做了相应的学习笔记，分为两篇，此为 2/2 篇。

## 3. JAVA NIO 深入剖析

在讲解利用 NIO 实现通信架构之前，我们需要先来了解一下 NIO 的基本特点和使用。

### 3.1 Java NIO 基本介绍

* Java NIO（New IO）也有人称之为 Java non-blocking IO 是从 Java 1.4 版本开始引入的一个新的 IO API，可以替代标准的 Java IO API。NIO 与原来的 IO 有同样的作用和目的，但是使用的方式完全不同，NIO 支持面**向缓冲区**的、基于**通道**的 IO 操作。NIO 将以更加高效的方式进行文件的读写操作。**NIO 可以理解为非阻塞 IO**，传统的 IO 的 read 和 write 只能阻塞执行，线程在读写 IO 期间不能干其他事情，比如调用`socket.read()`时，如果服务器一直没有数据传输过来，线程就一直阻塞，而 NIO 中可以配置socket为非阻塞模式。
*  NIO 相关类都被放在 `java.nio` 包及子包下，并且对原 `java.io` 包中的很多类进行改写。
* NIO 有三大核心部分：**Channel( 通道) ，Buffer( 缓冲区)， Selector( 选择器)**
* Java NIO 的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情；非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
* 通俗理解：NIO 是可以做到用一个线程来处理多个操作的。假设有 1000 个请求过来，根据实际情况，可以分配 20 或者 80 个线程来处理。不像之前的阻塞 IO 那样，非得分配 1000 个。

### 3.2 NIO 和 BIO 的比较

* BIO 以**流的方式**处理数据，而 NIO 以**块的方式**处理数据块， I/O 的效率比流 I/O 高很多
* BIO 是**阻塞**的，NIO 则是**非阻塞**的
* BIO 基于**字节流和字符流进行操作**，而 NIO 基于 **Channel（通道）和 Buffer（缓冲区）进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道

| NIO                       | BIO                   |
| ------------------------- | --------------------- |
| 面向缓冲区（Buffer）      | 面向流（Stream）      |
| 非阻塞（Non Blocking IO） | 阻塞IO（Blocking IO） |
| 选择器（Selectors）       |                       |

### 3.3 NIO 三大核心原理示意图

NIO 有三大核心部分：**Channel( 通道) ，Buffer( 缓冲区), Selector( 选择器)**

#### Buffer缓冲区

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成 NIO Buffer 对象，并提供了一组方法，用来方便的访问该块内存。相比较直接对数组的操作，Buffer API 更加容易操作和管理。

#### Channel（通道）

Java NIO 的通道类似流，但又有些不同：既可以从通道中读取数据，又可以写数据到通道。但流的（input或output）读写通常是单向的。 通道可以非阻塞读取和写入通道，通道可以支持读取或写入缓冲区，也支持异步地读写。

#### Selector选择器

Selector 是一个 Java NIO 组件，可以能够检查一个或多个 NIO 通道，并确定哪些通道已经准备好进行读取或写入。这样，一个单独的线程可以管理多个 channel，从而管理多个网络连接，提高效率

![Selector架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527536.png)

* 每个 channel 都会对应一个 Buffer
* 一个线程对应 Selector ， 一个 Selector 对应多个 channel
* 程序切换到哪个 channel 是**由事件决定**的
*  Selector 会根据不同的事件，在各个通道上切换
* Buffer 就是一个内存块，底层是一个数组
* 数据的读取写入是通过 Buffer 完成的，BIO 中要么是输入流，或者是输出流，不能双向，但是 NIO 的 Buffer是可以读也可以写。

Java NIO 系统的核心在于：通道(Channel)和缓冲区(Buffer)。通道表示打开到 IO 设备(例如：文件、 套接字)的连接。若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

**简而言之，Channel 负责传输， Buffer 负责存取数据**

### 3.4 NIO核心一：缓冲区(Buffer)

#### 缓冲区（Buffer）

一个用于特定基本数据类型的容器。由 `java.nio` 包定义的，所有缓冲区都是 Buffer 抽象类的子类。Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的

![缓冲区buffer](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527811.png)

#### Buffer 类及其子类

**Buffer** 就像一个数组，可以保存多个相同类型的数据。根据数据类型不同 ，有以下 Buffer 常用子类： 

* ByteBuffer 
* CharBuffer 
* ShortBuffer 
* IntBuffer 
*  LongBuffer 
* FloatBuffer 
* DoubleBuffer 

上述 Buffer 类他们都采用相似的方法进行管理数据，只是各自管理的数据类型不同而已。都是通过如下方法获取一个 Buffer 对象：

`static XxxBuffer allocate(int capacity) : 创建一个容量为capacity 的 XxxBuffer 对象`

#### 缓冲区的基本属性

Buffer 中的重要概念： 

* **容量 (capacity)** ：作为一个内存块，Buffer具有一定的固定大小，也称为"容量"，缓冲区容量不能为负，并且创建后不能更改。 
*  **限制 (limit)**：表示缓冲区中可以操作数据的大小（limit 后数据不能进行读写）。缓冲区的限制不能为负，并且不能大于其容量。 **写入模式，限制等于buffer的容量；读取模式下，limit等于写入的数据量**。
* **位置 (position)**：下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限制 
* **标记 (mark)与重置 (reset)**：标记是一个索引，通过 Buffer 中的 `mark()` 方法 指定 Buffer 中一个特定的 position，之后可以通过调用 `reset()` 方法恢复到这个 position.

**标记、位置、限制、容量遵守以下不变式： 0 <= mark <= position <= limit <= capacity**

**图示:**

![缓冲区的基本属性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527281.png)

#### Buffer常见方法

```java
Buffer clear() 			// 清空缓冲区并返回对缓冲区的引用
Buffer flip() 			// 为将缓冲区的界限设置为当前位置，并将当前位置设置为0
int capacity() 			// 返回Buffer的capacity大小
boolean hasRemaining() 	// 判断缓冲区中是否还有元素
int limit() 			// 返回Buffer的界限(limit)的位置
Buffer limit(int n) 	// 将设置缓冲区界限为n, 并返回一个具有新limit的缓冲区对象
Buffer mark() 			// 对缓冲区设置标记
int position() 			// 返回缓冲区的当前位置position
Buffer position(int n) 	// 将设置缓冲区的当前位置为n , 并返回修改后的Buffer对象
int remaining() 		// 返回position和limit之间的元素个数
Buffer reset() 			// 将位置position转到以前设置的mark所在的位置
Buffer rewind() 		// 将位置设为为0，取消设置的mark
```

#### 缓冲区的数据操作

```java
// Buffer所有子类提供了两个用于数据操作的方法：get()/put()

// get获取 Buffer中的数据：
get()				// 读取单个字节
get(byte[] dst)		// 批量读取多个字节到 dst 中
get(int index)		// 读取指定索引位置的字节(不会移动 position)
    
// put放到入数据到Buffer中：
put(byte b)				// 将给定单个字节写入缓冲区的当前位置
put(byte[] src)			// 将src中的字节写入缓冲区的当前位置
put(int index, byte b)	// 将指定字节写入缓冲区的索引位置(不会移动 position)
```

**使用Buffer读写数据一般遵循以下四个步骤：**

1. 写入数据到Buffer
2. 调用`flip()`方法，转换为读取模式

3. 从Buffer中读取数据

4. 调用`buffer.clear()`方法或者`buffer.compact()`方法清除缓冲区

#### 案例演示

```java
public class TestBuffer {
   
    @Test
    public void test1(){
        String str = "zhuchengchao";
        //1. 分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        System.out.println("-----------------allocate()----------------");
        System.out.println(buf.position());     // 0
        System.out.println(buf.limit());        // 1024
        System.out.println(buf.capacity());     // 1024

        //2. 利用 put() 存入数据到缓冲区中
        buf.put(str.getBytes());
        System.out.println("-----------------put()----------------");
        System.out.println(buf.position());     // 12
        System.out.println(buf.limit());        // 1024
        System.out.println(buf.capacity());     // 1024

        //3. 切换读取数据模式
        buf.flip();
        System.out.println("-----------------flip()----------------");
        System.out.println(buf.position());     // 0
        System.out.println(buf.limit());        // 12
        System.out.println(buf.capacity());     // 1024

        //4. 利用 get() 读取缓冲区中的数据
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst, 0, dst.length));  // zhuchengchao
        System.out.println("-----------------get()----------------");
        System.out.println(buf.position());     // 12
        System.out.println(buf.limit());        // 12
        System.out.println(buf.capacity());     // 1024

        //5. rewind() : 可重复读
        buf.rewind();
        System.out.println("-----------------rewind()----------------");
        System.out.println(buf.position());     // 0
        System.out.println(buf.limit());        // 12
        System.out.println(buf.capacity());     // 1024

        //6. clear() : 清空缓冲区. 但是缓冲区中的数据依然存在，但是处于“被遗忘”状态
        buf.clear();
        System.out.println("-----------------clear()----------------");
        System.out.println(buf.position());     // 0
        System.out.println(buf.limit());        // 1024
        System.out.println(buf.capacity());     // 1024
        System.out.println((char)buf.get());    // z
        System.out.println(buf.position());     // 1
    }
   
   
    @Test
    public void test2(){
        String str = "zhuchengchao";

        ByteBuffer buf = ByteBuffer.allocate(1024);

        buf.put(str.getBytes());
        buf.flip();
        byte[] dst = new byte[buf.limit()];
        buf.get(dst, 0, 2);
        System.out.println(new String(dst, 0, 2));  // zh
        System.out.println(buf.position());  // 2

        //mark() : 标记
        buf.mark();
        buf.get(dst, 2, 2);
        System.out.println(new String(dst, 2, 2));  // uc
        System.out.println(buf.position());  // 4

        //reset() : 恢复到 mark 的位置
        buf.reset();
        System.out.println(buf.position()); // 2

        //判断缓冲区中是否还有剩余数据
        if(buf.hasRemaining()){
            //获取缓冲区中可以操作的数量
            System.out.println(buf.remaining());  // 10
        }
    }
    
    @Test
    public void test3(){
        //分配直接缓冲区
        ByteBuffer buf = ByteBuffer.allocateDirect(1024);
        System.out.println(buf.isDirect());  // true
    }
}
```

#### 直接与非直接缓冲区

什么是直接内存与非直接内存？

根据官方文档的描述：

`byte byffer`可以是两种类型，一种是基于直接内存（也就是非堆内存）；另一种是非直接内存（也就是堆内存）。

对于直接内存来说，JVM 将会在 IO 操作上具有更高的性能，因为它直接作用于本地系统的 IO 操作。

而非直接内存，也就是堆内存中的数据，如果要作 IO 操作，会先从本进程内存复制到直接内存，再利用本地 IO 处理。

从数据流的角度，非直接内存是下面这样的作用链：

```
本地IO-->直接内存-->非直接内存-->直接内存-->本地IO
```

而直接内存是：

```
本地IO-->直接内存-->本地IO
```

很明显，在做 IO 处理时，比如网络发送大量数据时，**直接内存会具有更高的效率**。

直接内存使用 `allocateDirect`创建，但是它比申请普通的堆内存需要耗费更高的性能。不过，这部分的数据是在 JVM 之外的，因此它不会占用应用的内存。所以呢，当你有很大的数据要缓存，并且它的生命周期又很长，那么就比较适合使用直接内存。**只是一般来说，如果不是能带来很明显的性能提升，还是推荐直接使用堆内存**。

字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 `isDirect()` 方法来确定。

**使用场景**

- 有很大的数据需要存储，它的生命周期又很长
- 适合频繁的 IO 操作，比如网络并发场景

### 3.5 NIO 核心二：通道(Channel)

#### 通道 Channel 概述

通道（Channel）：由 `java.nio.channels` 包定义的。Channel 表示 IO 源与目标打开的连接。 Channel 类似于传统的“流”。**只不过 Channel 本身不能直接访问数据，Channel 只能与 Buffer 进行交互**。

NIO 的通道类似于流，但有些区别如下：

* 通道可以**同时进行读写**，而流只能读或者只能写

*  通道可以实现**异步读写数据**

*  通道可以从缓冲读数据，也可以写数据到缓冲:

BIO 中的 stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道(Channel)是双向的，可以读操作，也可以写操作。

Channel 在 NIO 中是一个接口

```java
public interface Channel extends Closeable{}
```

#### 常用的 Channel 实现类

* FileChannel：用于读取、写入、映射和操作文件的通道。

* DatagramChannel：通过 UDP 读写网络中的数据通道。

* SocketChannel：通过 TCP 读写网络中的数据。

* ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

  > ServerSocketChannel 类似 ServerSocket , SocketChannel 类似 Socket

#### FileChannel 类

获取通道的一种方式是对支持通道的对象调用 `getChannel()`方法。支持通道的类如下：

* FileInputStream
* FileOutputStream
* RandomAccessFile
* DatagramSocket
* Socket
* ServerSocket

* 获取通道的其他方式是使用 Files 类的静态方法 `newByteChannel()` 获取字节通道；或者通过通道的静态方法 `open()` 打开并返回指定通道

#### FileChannel 的常用方法

```java
int read(ByteBuffer dst) 		// 从Channel读取数据到ByteBuffer中
long read(ByteBuffer[] dsts)	// 将Channel中的数据“分散”到ByteBuffer[]中
int write(ByteBuffer src)		// 将ByteBuffer中的数据写入到Channel中
long write(ByteBuffer[] srcs) 	// 将ByteBuffer[]到中的数据“聚集”到Channel中
long position() 				// 返回此通道的文件位置
FileChannel position(long p) 	// 设置此通道的文件位置
long size() 					// 返回此通道的文件的当前大小
FileChannel truncate(long s) 	// 将此通道的文件截取为给定大小
void force(boolean metaData) 	// 强制将所有对此通道的文件更新写入到存储设备中
```

#### 案例1-本地文件写数据

需求：使用前面学习后的 ByteBuffer(缓冲) 和 FileChannel(通道)， 将 "hello,黑马Java程序员！" 写入到 data.txt 中

```java
package cn.xyc.nio;

import org.junit.Test;

import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ChannelTest {

    @Test
    public void write(){
        FileOutputStream fos = null;
        try {
            // 1、字节输出流通向目标文件
            fos = new FileOutputStream("data01.txt");
            // 2、得到字节输出流对应的通道Channel
            FileChannel channel = fos.getChannel();
            // 3、分配缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("hello,黑马Java程序员！".getBytes());
            // 4、把缓冲区切换成写出模式
            buffer.flip();
            channel.write(buffer);
            channel.close();
            System.out.println("写数据到文件中！");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 案例2-本地文件读数据

需求：使用前面学习后的 ByteBuffer(缓冲) 和 FileChannel(通道)， 将 data01.txt 中的数据读入到程序，并显示在控制台屏幕

```java
@Test
public void read() throws Exception{
    // 1、定义一个文件字节输入流与源文件接通
    FileInputStream fis = new FileInputStream("data01.txt");
    // 2、需要得到文件字节输入流的文件通道
    FileChannel channel = fis.getChannel();
    // 3、定义一个缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    // 4、读取数据到缓冲区
    channel.read(buffer);
    // 5、读取出缓冲区中的数据并输出即可
    buffer.flip();
    System.out.println(new String(buffer.array(), 0, buffer.remaining()));
}
```

#### 案例3-使用Buffer完成文件复制

使用 FileChannel(通道) ，完成文件的拷贝。

```java
@Test
public void copy() throws Exception {
    // 源文件
    File srcFile = new File("C:\\Users\\dlei\\Desktop\\BIO,NIO,AIO\\文件\\壁纸.jpg");
    File destFile = new File("C:\\Users\\dlei\\Desktop\\BIO,NIO,AIO\\文件\\壁纸new.jpg");
    // 得到一个字节字节输入流
    FileInputStream fis = new FileInputStream(srcFile);
    // 得到一个字节输出流
    FileOutputStream fos = new FileOutputStream(destFile);
    // 得到的是文件通道
    FileChannel isChannel = fis.getChannel();
    FileChannel osChannel = fos.getChannel();
    // 分配缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    while(true){
        // 必须先清空缓冲然后再写入数据到缓冲区
        buffer.clear();
        // 开始读取一次数据
        int flag = isChannel.read(buffer);
        if(flag == -1){
            break;
        }
        // 已经读取了数据 ，把缓冲区的模式切换成可读模式
        buffer.flip();
        // 把数据写出到
        osChannel.write(buffer);
    }
    isChannel.close();
    osChannel.close();
    System.out.println("复制完成！");
}
```

#### 案例4-分散 (Scatter) 和聚集 (Gather)

分散读取（Scatter）：是指把Channel通道的数据读入到多个缓冲区中去

聚集写入（Gathering）：是指将多个 Buffer 中的数据“聚集”到 Channel。

```java
// 分散和聚集
@Test
public void test() throws IOException{
	RandomAccessFile raf1 = new RandomAccessFile("1.txt", "rw");
	// 1. 获取通道
	FileChannel channel1 = raf1.getChannel();
	
	// 2. 分配指定大小的缓冲区
	ByteBuffer buf1 = ByteBuffer.allocate(100);
	ByteBuffer buf2 = ByteBuffer.allocate(1024);
	
	// 3. 分散读取
	ByteBuffer[] bufs = {buf1, buf2};
	channel1.read(bufs);
	
	for (ByteBuffer byteBuffer : bufs) {
		byteBuffer.flip();
	}
	
	System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));
	System.out.println("-----------------");
	System.out.println(new String(bufs[1].array(), 0, bufs[1].limit()));
	
	// 4. 聚集写入
	RandomAccessFile raf2 = new RandomAccessFile("2.txt", "rw");
	FileChannel channel2 = raf2.getChannel();
	
	channel2.write(bufs);
}
```
#### 案例5-transferFrom()

从目标通道中去复制原通道数据

```java
@Test
public void testTransferFrom() throws Exception {
    // 1、字节输入管道
    FileInputStream is = new FileInputStream("data01.txt");
    FileChannel isChannel = is.getChannel();
    // 2、字节输出流管道
    FileOutputStream fos = new FileOutputStream("data03.txt");
    FileChannel osChannel = fos.getChannel();
    // 3、复制
    osChannel.transferFrom(isChannel,isChannel.position(),isChannel.size());
    isChannel.close();
    osChannel.close();
}
```

#### 案例6-transferTo()

把原通道数据复制到目标通道

```java
@Test
public void test02() throws Exception {
    // 1、字节输入管道
    FileInputStream is = new FileInputStream("data01.txt");
    FileChannel isChannel = is.getChannel();
    // 2、字节输出流管道
    FileOutputStream fos = new FileOutputStream("data04.txt");
    FileChannel osChannel = fos.getChannel();
    // 3、复制
    isChannel.transferTo(isChannel.position() , isChannel.size() , osChannel);
    isChannel.close();
    osChannel.close();
}
```

### 3.6 NIO核心三：选择器(Selector)

#### 选择器(Selector)概述

选择器（Selector） 是 SelectableChannel 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector 可使一个单独的线程管理多个 Channel。

**Selector 是非阻塞 IO 的核心**

![Selector继承关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251528311.png)

* Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连接，就会使用到 Selector(选择器)
* Selector 能够检测多个注册的通道上是否有事件发生(注意:多个 Channel 以事件的方式可以注册到同一个Selector)，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。
* 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程
* 避免了多线程之间的上下文切换导致的开销

#### 选择器（Selector）的应用

创建 Selector ：通过调用 `Selector.open()` 方法创建一个 Selector。

```java
Selector selector = Selector.open();
```

向选择器注册通道：`SelectableChannel.register(Selector sel, int ops)`

```java
// 1. 获取通道
ServerSocketChannel ssChannel = ServerSocketChannel.open();
// 2. 切换非阻塞模式
ssChannel.configureBlocking(false);
// 3. 绑定连接
ssChannel.bind(new InetSocketAddress(9898));
// 4. 获取选择器
Selector selector = Selector.open();
// 5. 将通道注册到选择器上, 并且指定“监听接收事件”
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

当调用 `register(Selector sel, int ops)` 将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数 ops 指定。可以监听的事件类型（可使用 SelectionKey  的四个常量表示）：

* 读（1）: `SelectionKey.OP_READ` 
* 写（4） : `SelectionKey.OP_WRITE`
* 连接（8）: `SelectionKey.OP_CONNECT` 
*  接收（16）: `SelectionKey.OP_ACCEPT`
* 若注册时不止监听一个事件，则可以使用“位或”操作符连接。

```java
int interestSet = SelectionKey.OP_READ|SelectionKey.OP_WRITE 
```

### 3.7 NIO非阻塞式网络通信原理分析

#### Selector 示意图和特点说明

Selector 可以实现： 一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

![Selector架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527226.png)

#### 服务端流程

* 获取通道，当客户端连接服务端时，服务端会通过 ServerSocketChannel 得到 SocketChannel：`ServerSocketChannel ssChannel = ServerSocketChannel.open();`

* 切换非阻塞模式：`ssChannel.configureBlocking(false);`

* 绑定连接：`ssChannel.bind(new InetSocketAddress(9999));`

* 获取选择器：`Selector selector = Selector.open();`

* 将通道注册到选择器上, 并且指定“监听接收事件”：`ssChannel.register(selector, SelectionKey.OP_ACCEPT);`

* **轮询式的获取选择器上已经“准备就绪”的事件**

  ```java
  // 轮询式的获取选择器上已经“准备就绪”的事件
   while (selector.select() > 0) {
          System.out.println("轮一轮");
          // 7. 获取当前选择器中所有注册的“选择键(已就绪的监听事件)”
          Iterator<SelectionKey> it = selector.selectedKeys().iterator();
          while (it.hasNext()) {
              // 8. 获取准备“就绪”的是事件
              SelectionKey sk = it.next();
              // 9. 判断具体是什么事件准备就绪
              if (sk.isAcceptable()) {
                  // 10. 若“接收就绪”，获取客户端连接
                  SocketChannel sChannel = ssChannel.accept();
                  // 11. 切换非阻塞模式
                  sChannel.configureBlocking(false);
                  // 12. 将该通道注册到选择器上
                  sChannel.register(selector, SelectionKey.OP_READ);
              } else if (sk.isReadable()) {
                  // 13. 获取当前选择器上“读就绪”状态的通道
                  SocketChannel sChannel = (SocketChannel) sk.channel();
                  // 14. 读取数据
                  ByteBuffer buf = ByteBuffer.allocate(1024);
                  int len = 0;
                  while ((len = sChannel.read(buf)) > 0) {
                      buf.flip();
                      System.out.println(new String(buf.array(), 0, len));
                      buf.clear();
                  }
              }
              // 15. 取消选择键 SelectionKey
              it.remove();
          }
      }
  }
  ```

#### 客户端流程

* 获取通道：`SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9999));`

* 切换非阻塞模式：`sChannel.configureBlocking(false);`

* 分配指定大小的缓冲区：`ByteBuffer buf = ByteBuffer.allocate(1024);`

* 发送数据给服务端：

	
	```java
	Scanner scan = new Scanner(System.in);
	while(scan.hasNext()){
		String str = scan.nextLine();
		buf.put((new SimpleDateFormat("yyyy/MM/dd HH:mm:ss").format(System.currentTimeMillis())
				+ "\n" + str).getBytes());
		buf.flip();
		sChannel.write(buf);
		buf.clear();
	}
	//关闭通道
	sChannel.close();
	```
### 3.8 NIO 非阻塞式网络通信入门案例

**需求**：服务端接收客户端的连接请求，并接收多个客户端发送过来的事件。

#### 代码案例

**客户端：**

```java
package cn.xyc.select;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.text.SimpleDateFormat;
import java.util.Scanner;

public class Client {

    public static void main(String[] args) throws Exception {
        //1. 获取通道
        SocketChannel channel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8888));
        //2. 切换非阻塞模式
        channel.configureBlocking(false);
        // 3. 分配指定大小的缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 4. 发送数据给服务端
        Scanner sc = new Scanner(System.in);
        while (sc.hasNext()){
            String str = sc.nextLine();
            String time = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss").format(System.currentTimeMillis());
            buffer.put((time + '\n' + str).getBytes());
            buffer.flip();
            channel.write(buffer);
            buffer.clear();
        }
        channel.close();
    }
}
```

**服务端：**

```java
package cn.xyc.select;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class Server {
    public static void main(String[] args) throws Exception{
        // 1. 获取通道
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        // 2. 切换非阻塞模式
        ssChannel.configureBlocking(false);
        // 3. 绑定连接
        ssChannel.bind(new InetSocketAddress(8888));
        // 4. 获取选择器
        Selector selector = Selector.open();
        // 5. 将通道注册到选择器上, 并且指定“监听接收事件”
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
        // 6. 轮询式的获取选择器上已经“准备就绪”的事件
        while (selector.select() > 0){
            System.out.println("轮一轮");
            // 7. 获取当前选择器中所有注册的“选择键(已就绪的监听事件)”
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()){
                // 8. 获取准备“就绪”的是事件
                SelectionKey selectionKey = iterator.next();
                // 9. 判断具体是什么事件准备就绪
                if(selectionKey.isAcceptable()){
                    // 10. 若“接收就绪”，获取客户端连接
                    SocketChannel sChannel = ssChannel.accept();
                    // 11. 切换非阻塞模式
                    sChannel.configureBlocking(false);
                    // 12. 将该通道注册到选择器上
                    sChannel.register(selector, SelectionKey.OP_READ);
                }else if(selectionKey.isReadable()){
                    // 13. 获取当前选择器上“读就绪”状态的通道
                    SocketChannel sChannel = (SocketChannel) selectionKey.channel();
                    // 14. 读取数据
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = sChannel.read(buffer)) > 0){
                        buffer.flip();
                        System.out.println(new String(buffer.array(), 0, len));
                        buffer.clear();
                    }
                }
                // 15. 取消选择键 SelectionKey
                iterator.remove();
            }
        }
    }
}
```

### 3.9  NIO 网络编程应用实例-群聊系统

#### 目标

**需求:进一步理解 NIO 非阻塞网络编程机制，实现多人群聊**

* 编写一个 NIO 群聊系统，实现客户端与客户端的通信需求（非阻塞）
* 服务器端：可以监测用户上线，离线，并实现消息转发功能
* 客户端：通过 Channel 可以无阻塞发送消息给其它所有客户端用户，同时可以接受其它客户端用户通过服务端转发来的消息

#### 服务端代码实现

```java
package cn.xyc.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

public class Server {

    // 定义属性
    // 选择器
    private Selector selector;
    // 服务器的通道
    private ServerSocketChannel ssChannel;
    // 端口号
    private static final int PORT=8888;

    // 构造器初始化工作
    public Server() {
        try {
            // 1、获取通道
            ssChannel = ServerSocketChannel.open();
            // 2、切换为非阻塞模式
            ssChannel.configureBlocking(false);
            // 3、绑定连接的端口
            ssChannel.bind(new InetSocketAddress(PORT));
            // 4、获取选择器Selector
            selector = Selector.open();
            // 5、将通道都注册到选择器上去，并且开始指定监听接收事件
            ssChannel.register(selector, SelectionKey.OP_ACCEPT);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    // 监听
    public void listen(){
        System.out.println("监听线程: " + Thread.currentThread().getName());
        try {
            // 当选择器中有注册的通道，会阻塞等待监听事件
            while (selector.select() > 0){
                System.out.println("开始一轮事件处理~~~");
                // 7、获取选择器中的所有注册的通道中已经就绪好的事件
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                // 8、开始遍历这些准备好的事件
                while (iterator.hasNext()){
                    // 提取当前这个事件
                    SelectionKey selectionKey = iterator.next();
                    // 9、判断这个事件具体是什么
                    if(selectionKey.isAcceptable()){
                        // 10、是接收事件：直接获取当前接入的客户端通道
                        SocketChannel sChannel = ssChannel.accept();
                        // 11 、切换成非阻塞模式
                        sChannel.configureBlocking(false);
                        // 12、将本客户端通道注册到选择器
                        System.out.println(sChannel.getRemoteAddress() + " 上线 ");
                        sChannel.register(selector , SelectionKey.OP_READ);
                    }else if(selectionKey.isReadable()){
                        // 13、是客户端传输消息事件：则读取客户端消息
                        readData(selectionKey);
                    }

                    // 14、处理完毕之后需要移除当前事件
                    iterator.remove();
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            // 发生异常处理....
        }
    }

    // 读取客户端消息
    private void readData(SelectionKey selectionKey) {
        // 取到关联的channel
        SocketChannel channel = null;
        try {
            // 得到channel
            channel = (SocketChannel) selectionKey.channel();
            // 创建buffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int count = channel.read(buffer);
            // 根据count的值做处理
            if(count > 0) {
                // 把缓存区的数据转成字符串
                String msg = new String(buffer.array(), 0, count);
                // 输出该消息
                System.out.println("form 客户端: " + msg);
                // 向其它的客户端转发消息(去掉自己), 专门写一个方法来处理
                sendInfoToOtherClients(msg, channel);
            }
        }catch (Exception e){
            try {
                // e.printStackTrace();
                System.out.println(channel.getRemoteAddress() + " 离线了..");
                // 取消注册
                selectionKey.cancel();
                // 关闭通道
                channel.close();
            } catch (IOException e1) {
                e1.printStackTrace();
            }

        }
    }

    private void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {

        System.out.println("服务器转发消息中...");
        System.out.println("服务器转发数据给客户端的线程: " + Thread.currentThread().getName());

        // 遍历所有注册到selector上的SocketChannel,并排除self与服务器的channel
        for (SelectionKey key: selector.keys()){
            // 通过 key 取出对应的 SocketChannel
            Channel targetChannel = key.channel();
            // 排除自己 + 服务器的channel
            if(targetChannel != self && targetChannel instanceof SocketChannel){
                // 转型
                SocketChannel dest = (SocketChannel)targetChannel;
                // 将msg 存储到buffer
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                // 将buffer 的数据写入 通道
                dest.write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        //创建服务器对象
        Server groupChatServer = new Server();
        groupChatServer.listen();
    }
}
```

#### 客户端代码实现

```java
package cn.xyc.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Scanner;

public class Client {

    // 定义相关的属性
    private final String HOST = "127.0.0.1";
    private final int PORT = 8888;
    private SocketChannel socketChannel;
    private Selector selector;
    private String username;

    // 构造器, 完成初始化工作
    public Client() throws IOException {
        // 连接服务器
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
        // 获取选择器Selector
        selector = Selector.open();
        // 设置非阻塞
        socketChannel.configureBlocking(false);
        // 将channel 注册到selector
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 得到username
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + " is ok...");
    }

    // 向服务器发送消息
    public void sendInfo(String info) {

        info = username + " 说：" + info;
        try {
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 读取从服务器端回复的消息
    public void readInfo() {
        try {
            // 选择非阻塞式的selectNow
            int readChannels = selector.selectNow();
            if (readChannels > 0){
                // 有可以用的通道
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();
                    socketChannel = (SocketChannel) key.channel();
                    // 得到一个Buffer
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    // 读取
                    socketChannel.read(buffer);
                    // 把读到的缓冲区的数据转成字符串
                    String msg = new String(buffer.array());
                    System.out.println(msg);
                    // 删除当前的selectionKey, 防止重复操作
                    iterator.remove();
                }
            }else {
                // System.out.println("没有可以用的通道...");
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws IOException {
        // 启动我们客户端
        Client chatClient = new Client();
        // 启动一个线程, 每隔3秒，读取从服务器发送数据
        new Thread(() -> {
            while (true){
                chatClient.readInfo();
                try {
                    Thread.currentThread().sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        // 发送数据给服务器端
        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNextLine()) {
            String s = scanner.nextLine();
            chatClient.sendInfo(s);
        }
    }
}
```

## 4. JAVA AIO深入剖析

### AIO 编程

Java AIO(NIO.2) ： **异步非阻塞**，服务器实现模式为**一个有效请求一个线程**，客户端的 I/O 请求都是由 **OS 先完成了再通知服务器应用去启动线程进行处理**。

AIO：异步非阻塞，基于 NIO 的，可以称之为 NIO2.0

| BIO          | NIO                 | AIO                             |
| ------------ | ------------------- | ------------------------------- |
| Socket       | SocketChannel       | AsynchronousSocketChannel       |
| ServerSocket | ServerSocketChannel | AsynchronousServerSocketChannel |

与 NIO 不同，当进行读写操作时，只须直接调用 API 的 read 或 write 方法即可, **这两种方法均为异步的**

* 对于读操作而言，当有流可读取时，操作系统会将可读的流传入 read 方法的缓冲区
* 对于写操作而言，当操作系统将 write 方法传递的流写入完毕时，操作系统主动通知应用程序

即可以理解为，read/write 方法都是异步的，完成后会主动调用回调函数。

在 JDK1.7 中，这部分内容被称作NIO.2，主要在 `Java.nio.channels` 包下增加了下面四个异步通道：

```java
AsynchronousSocketChannel
AsynchronousServerSocketChannel
AsynchronousFileChannel
AsynchronousDatagramChannel
```

## 5. BIO / NIO / AIO 课程总结

**BIO、NIO、AIO：**

- Java BIO ： 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
- Java NIO ： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器（Selector）上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
- Java AIO(NIO.2) ： 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的 I/O 请求都是由 OS 先完成了再通知服务器应用去启动线程进行处理。

**BIO、NIO、AIO适用场景分析:**

- BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。
- NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。
- AIO 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，**充分调用OS参与并发操作**，编程比较复杂，JDK7 开始支持。



