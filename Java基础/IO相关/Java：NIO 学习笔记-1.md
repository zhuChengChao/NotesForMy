# Java：NIO 学习笔记-1

> 本笔记是学习了bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 发布的课程 [NIO视频](https://www.bilibili.com/video/BV14W411u7ro)，从而做的学习笔记。

### 主要内容

* Java NIO 简介
* Java NIO 与 IO 的主要区别
* 缓冲区（Buffer）和通道（Channel）
* 文件通道（FileChannel）
* NIO 的非阻塞式网络通信
  * 选择器（Selector）
  * SocketChannel、ServerSocketChannel、DatagramChannel
* 管道（Pipe）
* Java NIO2（Path、Paths与Files）

## Java NIO简介

Java NIO（New IO 或 Non Blocking IO）是从 Java 1.4 版本开始引入的一个新的 IO API，可以替代标准的 Java IO API。**NIO 支持面向缓冲区的、基于通道的 IO 操作**。NIO 将以更加高效的方式进行文件的读写操作。

## Java NIO 与 IO 的主要区别

| IO                        | NIO                           |
| ------------------------- | ----------------------------- |
| 面向流（Stream Oriented） | 面向缓冲区（Buffer Oriented） |
| 阻塞IO（Blocking IO）     | 非阻塞IO（Non Blocking IO）   |
| /                         | 选择器（Selectors）           |

## 通道和缓冲区

**Java NIO系统的核心在于：通道（Channel）和缓冲区（Buffer）**。通道表示打开到 IO 设备（例如：文件、套接字）的连接。若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

**简而言之，Channel 负责传输， Buffer 负责存储。**

### 缓冲区（Buffer）

**缓冲区（Buffer） ：一个用于特定基本数据类型的容器**。由 `java.nio` 包定义的，所有缓冲区都是 Buffer 抽象类的子类。

Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的。

Buffer 就像一个数组，可以保存多个相同类型的数据。根据数据类型不同（boolean除外），有以下Buffer常用子类：

* ByteBuffer
 * CharBuffer
 * ShortBuffer
 * IntBuffer
 * LongBuffer
 * FloatBuffer
 * DoubleBuffer

上述Buffer类他们都采用相似的方式进行管理数据，只是各自管理的数据类型不同而已。都是通过如下方法获取一个Buffer对象：

`static XxxBuffer allocate(int capacity)`：创建一个容量为 capacity 的 XxxBuffer 对象

```java
public static void test1(){
    /*
     * 一、缓冲区（Buffer）:在 java NIO 中负者数据的存储。缓冲区就是数组。用于存储不同类型的数据。
     * 根据数据类型的不同(boolean 除外)，有以下 Buffer 常用子类：
     * ByteBuffer
     * CharBuffer
     * ShortBuffer
     * IntBuffer
     * LongBuffer
     * FloatBuffer
     * DoubleBuffer
     *  上述缓冲区的管理方式几乎一致，通过allocate()获取缓冲区
     */
    // 1.分配一个指定大小的缓冲区
    ByteBuffer buf = ByteBuffer.allocate(1024);
}
```

#### 缓冲区的基本属性

**容量（capacity）**：表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。

**限制（limit）**：第一个不应该读取或写入的数据的索引，**即位于limit后的数据不可读写**。缓冲区的limit不能为负，并且不能大于其容量。

**位置（position）**：下一个要读取或写入的数据的索引。缓冲区的 position 不能为负，并且不能大于其限制（limit）。

**标记（mark）与重置（reset）**：标记是一个索引，通过Buffer中的`mark()`方法指定Buffer中一个特定的position，之后可以通过调用`reset()`方法恢复到这个 position。

**标记、位置、限制、容量遵守以下不变式：**`0<=mark<=position<=limit<=capacity`

![缓冲区的基本属性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251526827.PNG)

#### Buffer的常用方法

| 方法                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `Buffer clear()`         | 清空缓冲区并返回对缓冲区的引用，但是缓冲区中的内容依然存在，只是“被遗忘”了 |
| `Buffer flip()`          | 将缓冲区的限制limit设置为当前位置position，并将当前位置position重置为0 |
| `int capacity()`         | 返回Buffer的capacity大小                                     |
| `int limit()`            | 返回Buffer的界限limit的位置                                  |
| `Buffer limit(int n)`    | 将设置缓冲区界限为 n, 并返回一个具有新 limit 的缓冲区对象    |
| `int position()`         | 返回缓冲区的当前位置 position                                |
| `Buffer position(int n)` | 将设置缓冲区的当前位置为 n , 并返回修改后的 Buffer 对象      |
| `boolean hasRemaining()` | 判断缓冲区中是否还有元素                                     |
| `int remaining()`        | 返回 position 和 limit 之间的元素个数                        |
| `Buffer mark()`          | 对缓冲区设置标记                                             |
| `Buffer reset()`         | 将位置 position 转到以前设置的 mark 所在的位置               |
| `Buffer rewind()`        | 将位置 position 设置为 0， 取消设置的 mark，即可重新读数据   |

#### 缓冲区的数据操作

Buffer 所有子类提供了两个用于数据操作的方法：`get()` 与 `put()` 方法

**获取 Buffer 中的数据：**

* `get()` ：读取单个字节
* `get(byte[] dst)`：批量读取多个字节到 dst 中
* `get(int index)`：读取指定索引位置的字节(不会移动 position)

**放入数据到 Buffer 中：**

* `put(byte b)`：将给定单个字节写入缓冲区的当前位置
* `put(byte[] src)`：将 src 中的字节写入缓冲区的当前位置
* `put(int index, byte b)`：将指定字节写入缓冲区的索引位置(不会移动 position) 

```java
 /**
  * 二、缓冲区存取数据的两个核心方法：
  * 	put():存入数据到缓冲区中
  * 	get():获取缓存区中的数据
  * 三、缓冲区中的四个核心属性：
  * 	capacity：容量，表示缓冲区中最大存储数据的容量。一旦声明不能改变。
  * 	limit：界限，表示缓冲区中可以操作数据的大小。(limit后数据不能进行读写)
  * 	position：位置，表示缓冲区中正在操作数据的位置。
  * 	mark:标记，表示记录当前position位置。可以通过reset()恢复到mark的位置。
  */
public static void test2(){
    String str="abcde";

    // 1.分配一个指定大小的缓冲区
    ByteBuffer buf=ByteBuffer.allocate(1024);
    System.out.println("--------------allocate()----------------");
    System.out.println(buf.position());     // 0
    System.out.println(buf.limit());        // 1024
    System.out.println(buf.capacity());     // 1024

    // 2.利用put()存放数据到缓冲区中
    buf.put(str.getBytes());
    System.out.println("-------------put()-------------");
    System.out.println(buf.position());     // 5
    System.out.println(buf.limit());        // 1024
    System.out.println(buf.capacity());     // 1024

    // 3.切换读取数据模式
    buf.flip();
    System.out.println("--------------flip()------------");
    System.out.println(buf.position());     // 0
    System.out.println(buf.limit());        // 5
    System.out.println(buf.capacity());     // 1024

    // 4.利用get()读取缓冲区中的数据
    byte[] dst=new byte[buf.limit()];
    buf.get(dst);
    System.out.println(new String(dst, 0, dst.length));   // abcde

    System.out.println("--------------get()------------");
    System.out.println(buf.position());     // 5
    System.out.println(buf.limit());        // 5
    System.out.println(buf.capacity());     // 1024

    // 5.rewind():可重复读
    buf.rewind();
    System.out.println("--------------rewind()------------");
    System.out.println(buf.position());     //0
    System.out.println(buf.limit());        //5
    System.out.println(buf.capacity());     //1024

    // 6.clear():清空缓冲区。但是缓冲区中的数据依然存在，但是处在“被遗忘”状态
    buf.clear();
    System.out.println("--------------clear()------------");
    System.out.println(buf.position());     //0
    System.out.println(buf.limit());        //1024
    System.out.println(buf.capacity());     //1024

    System.out.println((char)buf.get());    // 还是能读取单个字节：a
}

public static void test3(){
    String str="abcde";
    ByteBuffer buf=ByteBuffer.allocate(1024);
    buf.put(str.getBytes());
    buf.flip();

    byte[] dst=new byte[buf.limit()];
    buf.get(dst,0,2);
    System.out.println(new String(dst,0,2));  //ab
    System.out.println(buf.position());  // 2

    // mark():标记
    buf.mark();
    buf.get(dst,2,2);//再读两个位置
    System.out.println(new String(dst, 2, 2)); // cd
    System.out.println(buf.position());  // 4

    // reset():恢复到mark的位置
    buf.reset();
    System.out.println(buf.position());  // 2

    // 判断缓冲区中是否还有剩余数据
    if(buf.hasRemaining()){
        // 获取缓冲区中可以操作的数量
        System.out.println(buf.remaining());  // 3
    }
}
```

### 直接与非直接缓冲区

* 字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）。

* 直接字节缓冲区可以通过调用此类的 `allocateDirect()` 工厂方法来创建。此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。

* 直接字节缓冲区还可以通过 `FileChannel` 的 `map()` 方法将文件区域直接映射到内存中来创建。该方法返回`MappedByteBuffer` 。Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。
  
* 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 `isDirect()` 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

#### 非直接缓冲区

![非直接缓冲区](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251526158.PNG)

#### 直接缓冲区

![直接缓冲区](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251526582.PNG)

```java
/* 
 * 四、直接缓冲区与非直接缓冲区：
 * 非直接缓冲区：
 *		通过allocate()方法分配缓冲区，将缓冲区建立在JVM的内存中。  
 * 直接缓冲区：
 * 		通过allocateDirect()方法分配直接缓冲区，将缓冲区建立在物理内存中。可以提高效率
 *      此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区。
 *      直接缓冲区的内容可以驻留在常规的垃圾回收堆之外。
 *      将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。
 *      最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。
 *      直接字节缓冲区还可以过 通过FileChannel 的 map() 方法 将文件区域直接映射到内存中来创建。该方法返回MappedByteBuffer
 */
public static void test4(){
    // 分配非直接缓冲区
    ByteBuffer buffer1 = ByteBuffer.allocate(1024);
    System.out.println(buffer1.isDirect());  // false

    // 分配直接缓冲区
    ByteBuffer buffer2 = ByteBuffer.allocateDirect(1024);
    System.out.println(buffer2.isDirect());  // true
}
```

### 通道（Channel）

通道（Channel）：由 `java.nio.channels` 包定义的。Channel 表示 IO 源与目标打开的连接。

Channel 类似于传统的“流”。只不过 **Channel本身不能直接访问数据，Channel 只能与Buffer 进行交互**。

#### 发展过程

1. 最早期的操作系统：所有的 IO 接口都由 CPU 独立负责，对导致 CPU 占用率高

   ![通道1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251526641.PNG)

2. 传统数据传输方式：在内存和 IO 之间通过 DMA 进行数据传输，但当产生大量的数据传输请求时，会出现总线冲突问题，影响性能

   ![通道2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527944.PNG)

   

3. 使用通道方式：通道成为完全独立的处理器，专门用于 IO 操作，拥有一套自身的命令与传输方式；而 CPU 无关，

   ![通道3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527272.PNG)

   


#### 通道实现类

**Java 为 Channel 接口提供的最主要实现类如下**

1. FileChannel：用于读取、写入、映射和操作文件的通道。
2. DatagramChannel：通过 UDP 读写网络中数据的通道。
3. SocketChannel：通过 TCP 读写网络中的数据。
4. ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个SocketChannel。 

#### 获取通道

**方式一：**获取通道的一种方式是对支持通道的对象调用 `getChannel()` 方法。支持通道的类如下：

本地IO：

* FileInputStream
* FileOutputStream
* RandomAccessFile

网络IO：

* DatagramSocket
* Socket
* ServerSocket

**方式二：**在 JDK1.7 中的 NIO.2 中

获取通道的其他方式是使用 `Files` 类的静态方法 `newByteChannel()` 获取字节通道。

或者通过通道的静态方法`open()`打开并返回指定通道。

#### 通道的数据传输

将 Buffer 中数据写入 Channel，例如：

```java
// 将Buffer中的数据写入Channel对于的文件中
int bytesWritten = inChannel.write(buf);
```

从 Channel 读取数据到 Buffer，例如：

```java
// 将channel对应的文件中的读取数据到buffer中
int bytesRead = inChannel.read(buf);
```

### 分散(Scatter)和聚集(Gather)

**分散读取（Scattering Reads）**是指从 Channel 中读取的数据“分散”到多个 Buffer 中。

![分散读取](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527893.PNG)

> 注意：按照缓冲区的顺序，从 Channel 中读取的数据依次将 Buffer 填满。

**聚集写入（Gathering Writes）**是指将多个 Buffer 中的数据“聚集”到 Channel。

![聚集写入](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527307.PNG)

> 注意：按照缓冲区的顺序，写入 position 和 limit 之间的数据到 Channel 。

#### `transferFrom()`&`transferTo()`

都是采用直接缓冲区的方式

`transferTo()`：将数据从源通道传输到其他 Channel 中：

```java
RandomAccessFile fromFile = new RandomAccessFile("./linux.words", "rw");
// 获取FileChannel
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("./linux.words.bak", "rw");
// 获取FileChannel
FileChannel toChannel = toFile.getChannel();

// 定义传输位置
long position = 0l;
// 最多传输的字节数
long count = fromChannel.size();

// 将数据从源通道传输到另一个通道
toChannel.transferFrom(fromChannel, position, count);
```

`transferFrom()`：将数据从其他Channel传输到本通道中

```java
// 其他代码同上
fromChannel.transferTo(position, count, toChannel);
```

### 代码演示

```java
/*
 * 一、通道(Channel):用于源节点与目标节点的连接。在java NIO中负责缓冲区中数据的传输。Channel本身不存储数据，需要配合缓冲区进行传输。
 *
 * 二、通道的主要实现类
 *    java.nio.channels.Channel 接口：
 *        |--FileChannel：用于读取、写入、映射和操作文件的通道。
 *        |--SocketChannel：通过 TCP 读写网络中的数据。
 *        |--ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。
 *        |--DatagramChannel：通过 UDP 读写网络中的数据通道。
 *
 * 三、获取通道
 * 1.java针对支持通道的类提供了getChannel()方法
 *      本地IO：
 *      	FileInputStream/FileOutputStream
 *      	RandomAccessFile
 *
 *      网络IO：
 *      	Socket
 *      	ServerSocket
 *      	DatagramSocket
 *
 * 2.在JDK 1.7 中的NIO.2 针对各个通道提供了静态方法 open()
 * 3.在JDK 1.7 中的NIO.2 的Files工具类的 newByteChannel()
 *
 * 四、通道之间的数据传输
 * 		transferFrom()
 * 		transferTo()
 *
 * 五、分散(Scatter)与聚集(Gather)
 * 		分散读取（Scattering Reads）：将通道中的数据分散到多个缓冲区中
 * 		聚集写入（Gathering Writes）：将多个缓冲区中的数据聚集到通道中
 *
 * 六、字符集：Charset
 * 		编码：字符串-》字符数组
 * 		解码：字符数组-》字符串
 */
public static void test1() throws IOException{
    // 利用通道完成文件的复制(非直接缓冲区)
    FileInputStream fis = new FileInputStream("./1.jpg");
    FileOutputStream fos = new FileOutputStream("./2.jpg");

    // 1.获取通道
    FileChannel inChannel = fis.getChannel();
    FileChannel outChannel = fos.getChannel();

    // 2.分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 3.将通道中的数据存入缓冲区中
    while (inChannel.read(buffer) != -1){
        buffer.flip();  // 切换读取数据的模式
        // 4.将缓冲区中的数据写入通道中
        outChannel.write(buffer);
        buffer.clear();
    }

    outChannel.close();
    inChannel.close();
    fos.close();
    fis.close();
}

// 使用直接缓冲区完成文件的复制(内存映射文件)
public static void test2() throws IOException{
    // 获取通道
    // 在JDK 1.7 中的 NIO.2 针对各个通道提供了静态方法 open()
    FileChannel inChannel = FileChannel.open(Paths.get("./1.jpg"),
                                             StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("./3.jpg"),
                                              StandardOpenOption.WRITE, 
                                              StandardOpenOption.READ,
                                              StandardOpenOption.CREATE);

    // 内存映射文件:通过FileChannel的map()方法 将文件区域直接映射到内存中来创建
    MappedByteBuffer inMappedBuf = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
    MappedByteBuffer outMappedBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());
    // 直接对缓冲区进行数据的读写操作
    byte[] dst = new byte[inMappedBuf.limit()];
    inMappedBuf.get(dst);
    outMappedBuf.put(dst);
    // 关闭
    outChannel.close();
    inChannel.close();
}

// 通道之间的数据传输(直接缓冲区)
public static void test3() throws IOException{
    FileChannel inChannel = FileChannel.open(Paths.get("./1.jpg"),
                                             StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("./4.jpg"),
                                              StandardOpenOption.WRITE, 
                                              StandardOpenOption.READ, 
                                              StandardOpenOption.CREATE);


    inChannel.transferTo(0, inChannel.size(), outChannel);
    outChannel.transferFrom(inChannel, 0, inChannel.size());

    outChannel.close();
    inChannel.close();
}

// 分散和聚集
public static void test4() throws IOException{
    RandomAccessFile raf1 = new RandomAccessFile("linux.words", "rw");
    // 1.获取通道
    FileChannel channel1 = raf1.getChannel();

    // 2.分配指定大小的缓冲区
    ByteBuffer buffer1 = ByteBuffer.allocate(100);
    ByteBuffer buffer2 = ByteBuffer.allocate(1024);

    // 3.分散读取
    ByteBuffer[] buffers = {buffer1, buffer2};
    channel1.read(buffers);

    for(ByteBuffer buffer: buffers){
        buffer.flip();
    }
    // 输出数据
    System.out.println(new String(buffers[0].array(), 0, buffers[0].limit()));
    System.out.println("----------");
    System.out.println(new String(buffers[1].array(), 0, buffers[1].limit()));

    // 4.聚集写入
    RandomAccessFile raf2 = new RandomAccessFile("linux.words.bak", "rw");
    FileChannel channel2 = raf2.getChannel();
    channel2.write(buffers);
}

// 输出支持的字符集
public static void test5(){
    SortedMap<String, Charset> stringCharsetSortedMap = Charset.availableCharsets();
    Set<Map.Entry<String, Charset>> entries = stringCharsetSortedMap.entrySet();
    for (Map.Entry<String, Charset> entry: entries){
        System.out.println(entry.getKey()+"="+entry.getValue());
    }
}

// 字符集
public static void test6() throws IOException{
    Charset charset = Charset.forName("GBK");
    // 获取编码器
    CharsetEncoder encoder = charset.newEncoder();
    // 获取解码器
    CharsetDecoder decoder = charset.newDecoder();

    CharBuffer charBuffer = CharBuffer.allocate(1024);
    charBuffer.put("你好");
    charBuffer.flip();

    // 进行编码
    ByteBuffer encodeResult = encoder.encode(charBuffer);
    for(int i=0;i<encodeResult.limit();i++){
        System.out.print(encodeResult.get()+" ");  //-60 -29 -70 -61
    }

    // 解码
    encodeResult.flip();
    CharBuffer decodeResult = decoder.decode(encodeResult);
    System.out.println(decodeResult.toString());  // 你好
}
```


## 文件通道（FileChannel）

### FileChannel 的常用方法

| 方法                            | 描述                                                |
| ------------------------------- | --------------------------------------------------- |
| `int read(ByteBuffer dst)`      | 从 Channel 对应的文件中读取数据到 ByteBuffer        |
| `long read(ByteBuffer[] dsts)`  | 将 Channel 对应的文件中的数据“分散”到ByteBuffer[]   |
| `int write(ByteBuffer src)`     | 将 ByteBuffer 中的数据写入到 Channel 通向的文件     |
| `long write(ByteBuffer[] srcs)` | 将 ByteBuffer[] 中的数据“聚集”到 Channel 通向的文件 |
| `long position()`               | 返回此通道的文件位置                                |
| `FileChannel position(long p)`  | 设置此通道的文件位置                                |
| `long size()`                   | 返回此通道的文件的当前大小                          |
| `FileChannel truncate(long s)`  | 将此通道的文件截取为给定大小                        |
| `void force(boolean metaData)`  | 强制将所有对此通道的文件更新写入到存储设备中        |

## NIO的非阻塞式网络通信

### 阻塞与非阻塞

**传统的 IO 流都是阻塞式的**。也就是说，当一个线程调用 `read()` 或 `write()` 时，该线程被阻塞，直到有一些数据被读取或写入，**该线程在此期间不能执行其他任务**。因此，在完成网络通信进行 IO 操作时，由于线程会阻塞，所以服务器端必须为每个客户端都提供一个独立的线程进行处理，当服务器端需要处理大量客户端时，性能急剧下降。

**Java NIO 是非阻塞模式的**。当线程从某通道进行读写数据时，若没有数据可用时，**该线程可以进行其他任务**。线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。因此，NIO 可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端。

### 选择器（Selector）

选择器（Selector）是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，**利用 Selector 可使一个单独的线程管理多个Channel。Selector 是非阻塞 IO 的核心**。

SelectableChannle 的结构如下图：

![SelectableChannle的结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527648.PNG)

### 选择器（Selector）的应用


创建 Selector ：通过调用 `Selector.open()` 方法创建一个 Selector。

```java
// 创建选择器
Selector selector = Selector.open();
```

向选择器注册通道：`SelectableChannel.register(Selector sel, int ops)`

```java
// 创建一个Socket套接字
Socket socket = new Socket(InetAddress.getByName("127.0.0.1"), 9898);
// 获取SocketChannel
SocketChannel channel = socket.getChannel();
// 创建选择器
Selector selector = Selector.open();
// 将SocketChannel切换到非阻塞模式
channel.configureBlocking(false);
// 向Selector注册Channel
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

当调用 `register(Selector sel, int ops)` 将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数 ops 指定。

### SelectionKey

可以监听的事件类型（可使用 SelectionKey 的四个常量表示）：

* 读 : `SelectionKey.OP_READ`    （1）

* 写 : `SelectionKey.OP_WRITE`  （4）

* 连接 : `SelectionKey.OP_CONNECT`  （8）

* 接收 : `SelectionKey.OP_ACCEPT`    （16）

若注册时不止监听一个事件，则可以使用“位或”操作符连接。

例： 

```java
// 注册"监听事件"
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

SelectionKey：表示 SelectableChannel 和 Selector 之间的注册关系。每次向选择器注册通道时就会选择一个事件(选择键)。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作。

| 方法                          | 描述                             |
| ----------------------------- | -------------------------------- |
| `int interestOps()`           | 获取感兴趣事件集合               |
| `int readyOps()`              | 获取通道已经准备就绪的操作的集合 |
| `SelectableChannel channel()` | 获取注册通道                     |
| `Selector selector()`         | 返回选择器                       |
| `boolean isReadable()`        | 检测 Channel 中读事件是否就绪    |
| `boolean isWritable()`        | 检测 Channel 中写事件是否就绪    |
| `boolean isConnectable()`     | 检测 Channel 中连接是否就绪      |
| `boolean isAcceptable()`      | 检测 Channel 中接受是否就绪      |

### Selector 的常用方法

| 方法                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `Set<SelectionKey> keys()`         | 所有的 SelectionKey 集合。代表注册在该Selector上的SelectionKey |
| `Set<SelectionKey> selectedKeys()` | 被选择的 SelectionKey 集合。返回此Selector的已选择键集       |
| `int select()`                     | 监控所有注册的Channel，当它们中间有需要处理的 IO 操作时，该方法返回，<br />并将对应得的 SelectionKey 加入被选择的SelectionKey 集合中，该方法返回这些 Channel 的数量 |
| `int select(long timeout)`         | 可以设置超时时长的 select() 操作                             |
| `int selectNow()`                  | 执行一个立即返回的 select() 操作，该方法不会阻塞线程         |
| `Selector wakeup()`                | 使一个还未返回的 select() 方法立即返回                       |
| `void close()`                     | 关闭该选择器                                                 |

### SocketChannel

Java NIO 中的 SocketChannel 是一个连接到TCP网络套接字的通道。

操作步骤：

1. 打开 SocketChannel
2. 读写数据
3. 关闭 SocketChannel 

Java NIO 中的 ServerSocketChannel 是一个可以监听新进来的 TCP 连接的通道，就像标准 IO 中的 ServerSocket一样。

### DatagramChannel

Java NIO 中的 DatagramChannel 是一个能收发 UDP 包的通道。

操作步骤：

1. 打开 DatagramChannel
2. 接收/发送数据 

### 测试代码

#### 说明

```java
/*
 * 一、使用NIO 完成网络通信的三个核心：
 * 
 * 1、通道(Channel):负责连接
 *      java.nio.channels.Channel 接口：
 *           |--SelectableChannel
 *               |--SocketChannel
 *               |--ServerSocketChannel
 *               |--DatagramChannel
 *               
 *               |--Pipe.SinkChannel
 *               |--Pipe.SourceChannel
 *               
 * 2.缓冲区(Buffer):负责数据的存取
 * 
 * 3.选择器(Selector):是 SelectableChannel 的多路复用器。用于监控SelectableChannel的IO状况
 */
```

#### 阻塞式的Socket通信-1

**服务器接受成功后不回应**

```java
public static void main(String[] args) throws IOException {
    //server();
    client();
}

public static void client() throws IOException{
    // 1. 获取通道
    SocketChannel socketChannel = SocketChannel.open(
        new InetSocketAddress("127.0.0.1",9898)
    );
    
    FileChannel inChannel = FileChannel.open(
        Paths.get("1.jpg"), StandardOpenOption.READ
    );

    // 2. 分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 3. 读取本地文件，并发送到服务器
    while (inChannel.read(buffer) != -1){
        buffer.flip();
        socketChannel.write(buffer);
        buffer.clear();
    }

    // 4. 关闭通道
    inChannel.close();
    socketChannel.close();
}

public static void server() throws IOException{
    // 1. 获取通道
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    FileChannel outChannel = FileChannel.open(
        Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE
    );

    // 2. 绑定连接
    serverSocketChannel.bind(new InetSocketAddress(9898));

    // 3. 获取客户端连接的通道
    SocketChannel socketChannel = serverSocketChannel.accept();

    // 4.分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 5. 接受客户端的数据，并保存到本地
    while (socketChannel.read(buffer) != -1){
        buffer.flip();
        outChannel.write(buffer);
        buffer.clear();
    }

    //6. 关闭通道
    socketChannel.close();
    outChannel.close();
    serverSocketChannel.close();
}
```

#### 阻塞式的Socket通信-2

**服务器接受成功后发送回应消息**

```java
public static void main(String[] args) throws IOException {
    //server();
    client();
}

public static void client() throws IOException{
    // 1. 获取通道
    SocketChannel socketChannel = SocketChannel.open(
        new InetSocketAddress("127.0.0.1",9898)
    );
    FileChannel inChannel = FileChannel.open(
        Paths.get("1.jpg"), StandardOpenOption.READ
    );

    // 2. 分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 3. 读取本地文件，并发送到服务器
    while (inChannel.read(buffer) != -1){
        buffer.flip();
        socketChannel.write(buffer);
        buffer.clear();
    }

    // 不加导致一直阻塞！！！
    socketChannel.shutdownOutput();

    // 接受服务器的反馈
    int len = 0;
    while ((len = socketChannel.read(buffer)) != -1){
        buffer.flip();
        System.out.println(new String(buffer.array(), 0, len));
        buffer.clear();

    }

    // 4. 关闭通道
    inChannel.close();
    socketChannel.close();
}

public static void server() throws IOException{
    // 1. 获取通道
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    FileChannel outChannel = FileChannel.open(
        Paths.get("3.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE
    );

    // 2. 绑定连接
    serverSocketChannel.bind(new InetSocketAddress(9898));

    // 3. 获取客户端连接的通道
    SocketChannel socketChannel = serverSocketChannel.accept();

    // 4.分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 5. 接受客户端的数据，并保存到本地
    while (socketChannel.read(buffer) != -1){
        buffer.flip();
        outChannel.write(buffer);
        buffer.clear();
    }

    // 发送反馈给客户端
    buffer.put("服务器端成功接受到数据".getBytes());
    buffer.flip();
    socketChannel.write(buffer);

    //6. 关闭通道
    socketChannel.close();
    outChannel.close();
    serverSocketChannel.close();
}
```

#### 非阻塞式Socket通信-TCP

```java
public static void main(String[] args)throws IOException {
    client();
    //server();
}

// 客户端
public static void client() throws IOException{
    System.out.println("client");
    // 1. 获取通道
    SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
    // 2. 切换非阻塞模式
    socketChannel.configureBlocking(false);
    // 3. 分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    // 4. 发送数据给服务器
    Scanner scanner = new Scanner(System.in);
    while (scanner.hasNext()){
        String str = scanner.next();
        buffer.put((new Date().toString() + "\n" + str).getBytes());
        buffer.flip();
        socketChannel.write(buffer);
        buffer.clear();
    }

    //5. 关闭通道
    socketChannel.close();
}

// 服务器
public static void server() throws IOException{
    System.out.println("server");
    // 1. 获取通道
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    // 2. 切换非阻塞模式
    serverSocketChannel.configureBlocking(false);
    // 3. 绑定连接
    serverSocketChannel.bind(new InetSocketAddress(9898));
    // 4. 获取选择器
    Selector selector = Selector.open();
    // 5. 将通道注册到选择器上
    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

    // 6.轮寻式的获取选择器上已经“准备就绪”的事件
    while (selector.select() > 0){
        // 7. 获取当前选择器中所有注册的“选择键（已就绪的监听事件）”
        Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();

        while (iterator.hasNext()){
            // 8. 获取准备“就绪”的事件
            SelectionKey key = iterator.next();

            // 9. 判断是具体是什么事件准备就绪
            if(key.isAcceptable()){
                // 10. 若“接受就绪”，获取客户端连接
                SocketChannel socketChannel = serverSocketChannel.accept();
                // 11.切换非阻塞模式
                socketChannel.configureBlocking(false);
                // 12.将该通道注册到选择器上
                socketChannel.register(selector, SelectionKey.OP_READ);
            }else if(key.isReadable()){
                // 13. 获取当前选择器上“读就绪”状态的通道
                SocketChannel socketChannel = (SocketChannel) key.channel();
                // 14. 读取数据
                ByteBuffer buf = ByteBuffer.allocate(1024);
                int len = 0;
                while ((len=socketChannel.read(buf)) > 0){
                    buf.flip();
                    System.out.println(new String(buf.array(), 0, len));
                    buf.clear();
                }
            }
        }
        // 15. 取消选择键SelectionKey
        iterator.remove();
    }
}
```

#### 非阻塞Socket通信-UDP

```java
public static void main(String[] args)throws IOException {
    clinet();
    //server();
}

public static void clinet() throws IOException{
    // 1. 获取通道
    DatagramChannel datagramChannel = DatagramChannel.open();
    // 2. 切换非阻塞模式
    datagramChannel.configureBlocking(false);
    // 3. 分配指定大小的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    // 4. 发送数据给服务器
    Scanner scanner = new Scanner(System.in);
    while (scanner.hasNext()){
        String str = scanner.next();
        buffer.put((new Date().toString() + "\n" + str).getBytes());
        buffer.flip();
        datagramChannel.send(buffer, new InetSocketAddress("127.0.0.1", 9898));
        buffer.clear();
    }
    //5. 关闭通道
    datagramChannel.close();
}

public static void server() throws IOException{
    // 1. 获取通道
    DatagramChannel datagramChannel = DatagramChannel.open();
    // 2. 切换非阻塞模式
    datagramChannel.configureBlocking(false);
    // 3. 绑定连接
    datagramChannel.bind(new InetSocketAddress(9898));
    // 4. 获取选择器
    Selector selector = Selector.open();
    // 5. 将通道注册到选择器上
    datagramChannel.register(selector, SelectionKey.OP_READ);

    // 6.轮寻式的获取选择器上已经“准备就绪”的事件
    while (selector.select() > 0){
        // 7. 获取当前选择器中所有注册的“选择键（已就绪的监听事件）”
        Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();

        while (iterator.hasNext()){
            // 8. 获取准备“就绪”的事件
            SelectionKey selectionKey = iterator.next();

            if(selectionKey.isReadable()){
                // 9. 读取数据
                ByteBuffer buf = ByteBuffer.allocate(1024);

                datagramChannel.receive(buf);
                buf.flip();
                System.out.println(new String(buf.array(), 0, buf.limit()));
                buf.clear();
            }
        }

        // 7.取消选择键SelectionKey
        iterator.remove();
    }
}
```

## **管道** (Pipe)

**Java NIO 管道是2个线程之间的单向数据连接**。

Pipe 有一个 source 通道和一个 sink 通道。数据会被写到 sink 通道，从 source 通道读取。

![管道](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251527157.png)

### 向管道写数据

```java
// 1.获取管道
Pipe pipe = Pipe.open();
// 2.将缓冲区中的数据写入管道
ByteBuffer buffer = ByteBuffer.allocate(1024);
buffer.put("通过单向管道发送数据".getBytes());
buffer.flip();
Pipe.SinkChannel sinkChannel = pipe.sink();
sinkChannel.write(buffer);
```

### 从管道读数据

从读取管道的数据，需要访问source通道

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```

调用source通道的`read()`方法来读取数据

```java
Pipe.SourceChannel sourceChannel = pipe.source();
ByteBuffer buffer = ByteBuffer.allocate(1024);
int len = sourceChannel.read(buffer);
System.out.println(new String(buffer.array(), 0, len));
```

**测试：**

```java
//1.获取管道
Pipe pipe = Pipe.open();
//2.将缓冲区中的数据写入管道
ByteBuffer buffer = ByteBuffer.allocate(1024);
buffer.put("通过单向管道发送数据".getBytes());
buffer.flip();
Pipe.SinkChannel sinkChannel = pipe.sink();
sinkChannel.write(buffer);

//3.读取缓冲区中的数据
Pipe.SourceChannel sourceChannel = pipe.source();
buffer.flip();
int len = sourceChannel.read(buffer);
System.out.println(new String(buffer.array(), 0, len));
```

## Java NIO2

### NIO.2

随着 JDK 7 的发布，Java 对 NIO 进行了极大的扩展，增强了对文件处理和文件系统特性的支持，以至于我们称他们为 NIO.2。因为 NIO 提供的一些功能，NIO 已经成为文件处理中越来越重要的部分。

### Path 与 Paths

`java.nio.file.Path` 接口代表一个平台无关的平台路径，描述了目录结构中文件的位置。

Paths 提供的 `get()` 方法用来获取 Path 对象：

* `Path get(String first, String … more)` : 用于将多个字符串串连成路径。

Path 常用方法：

* `boolean endsWith(String path)` : 判断是否以 path 路径结束

* `boolean startsWith(String path)` : 判断是否以 path 路径开始

* `boolean isAbsolute()` : 判断是否是绝对路径

* `Path getFileName()` : 返回与调用 Path 对象关联的文件名

* `Path getName(int idx)` : 返回的指定索引位置 idx 的路径名称

* `int getNameCount()` : 返回Path 根目录后面元素的数量

* `Path getParent()` ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径

* `Path getRoot()` ：返回调用 Path 对象的根路径

* `Path resolve(Path p)` :将相对路径解析为绝对路径

* `Path toAbsolutePath()` : 作为绝对路径返回调用 Path 对象

* `String toString()` ：返回调用 Path 对象的字符串表示形式 

### Files 类

`java.nio.file.Files` 用于操作文件或目录的工具类。

Files 常用方法：

* `Path copy(Path src, Path dest, CopyOption … how)` : 文件的复制
* `Path createDirectory(Path path, FileAttribute<?> … attr)` : 创建一个目录
* `Path createFile(Path path, FileAttribute<?> … arr)` : 创建一个文件
* `void delete(Path path)` : 删除一个文件
* `Path move(Path src, Path dest, CopyOption…how)` : 将 src 移动到 dest 位置
* `long size(Path path)` : 返回 path 指定文件的大小

Files 常用方法：用于判断

* `boolean exists(Path path, LinkOption … opts)` : 判断文件是否存在
* `boolean isDirectory(Path path, LinkOption … opts)` : 判断是否是目录
* `boolean isExecutable(Path path)` : 判断是否是可执行文件
* `boolean isHidden(Path path)` : 判断是否是隐藏文件
* `boolean isReadable(Path path)` : 判断文件是否可读
* `boolean isWritable(Path path)` : 判断文件是否可写
* `boolean notExists(Path path, LinkOption … opts)` : 判断文件是否不存在
* `public static <A extends BasicFileAttributes> A readAttributes(Path path,Class<A> type,LinkOption... options)` : 获取与 path 指定的文件相关联的属性。

Files 常用方法：用于操作内容

* `SeekableByteChannel newByteChannel(Path path, OpenOption…how)` : 获取与指定文件的连接，how 指定打开方式。
* `DirectoryStream newDirectoryStream(Path path)` : 打开 path 指定的目录
* `InputStream newInputStream(Path path, OpenOption…how)`:获取 InputStream 对象
* `OutputStream newOutputStream(Path path, OpenOption…how)` : 获取 OutputStream对象

### 自动资源管理

Java 7 增加了一个新特性，该特性提供了另外一种管理资源的方式，这种方式能自动关闭文件。这个特性有时被称为**自动资源管理 (Automatic Resource Management, ARM)**，该特性以 try 语句的扩展版为基础。自动资源管理主要用于，当不再需要文件（或其他资源）时，可以防止无意中忘记释放它们。

自动资源管理基于 try 语句的扩展形式：

```java
try(需要关闭的资源声明){
	//可能发生异常的语句
}catch(异常类型 变量名){
	//异常的处理语句
}
...
finally{
	//一定执行的语句
}
```

当 try 代码块结束时，自动释放资源。因此**不需要显示的调用 close() 方法**。该形式也称为“带资源的 try 语句”。

**注意：**

1. try 语句中声明的资源被隐式声明为 final ，资源的作用局限于带资源的 try 语句
2. 可以在一条 try 语句中管理多个资源，每个资源以`;` 隔开即可。
3. 需要关闭的资源，必须实现了 AutoCloseable 接口或其子接口 Closeable 

