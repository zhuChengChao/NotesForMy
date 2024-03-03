# Java：语言基础学习笔记-5

> 时间紧任务重，此文作为后端学习基础笔记的第五篇

## 20. 字节流&字符流

### 20.1 IO概述

#### 20.1.1 什么是IO

生活中，你肯定经历过这样的场景。当你编辑一个文本文件，忘记了 ctrl+s ，可能文件就白白编辑了。当你电脑上插入一个U盘，可以把一个视频，拷贝到你的电脑硬盘里。那么数据都是在哪些设备上的呢？键盘、内存、硬盘、外接设备等等。

我们把这种数据的传输，可以看做是一种数据的流动，按照流动的方向，**以内存为基准**，分为输入input和输出output ，**即流向内存是输入流，流出内存的输出流**。

Java中I/O操作主要是指使用 java.io 包下的内容，进行输入、输出操作。输入也叫做读取数据，输出也叫做作写出数据。

#### 20.1.2 IO的分类

根据数据的流向分为：**输入流和输出流**。

* **输入流** ：把数据从其他设备上读取到内存中的流。
* **输出流** ：把数据从内存中写出到其他设备上的流。

格局数据的类型分为：**字节流和字符流**。

* **字节流** ：以字节为单位，读写数据的流。
* **字符流** ：以字符为单位，读写数据的流。

#### 20.1.3 IO的流向说明图解

![IO的流向说明图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515485.PNG)

#### 20.1.4 顶级父类们

|        | 输入流                       | 输出流                        |
| ------ | ---------------------------- | ----------------------------- |
| 字节流 | 字节输入流 <br />InputStream | 字节输出流 <br />OutputStream |
| 字符流 | 字符输入流 <br />Reader      | 字符输出流 <br />Writer       |

### 20.2 字节流

#### 20.2.1 一切皆为字节

一切文件数据(文本、图片、视频等)在存储时，都是以二进制数字的形式保存，都一个一个的字节，那么传输时一样如此。所以，字节流可以传输任意文件数据。在操作流的时候，我们要时刻明确，无论使用什么样的流对象，底层传输的始终为二进制数据。

#### 20.2.2 字节输出流：OutputStream

`java.io.OutputStream` 抽象类是表示字节输出流的所有类的超类，将指定的字节信息写出到目的地。它定义了字节输出流的基本共性功能方法。

* `public void close()` ：关闭此输出流并释放与此流相关联的任何系统资源。
* `public void flush()` ：刷新此输出流并强制任何缓冲的输出字节被写出。
* `public void write(byte[] b)` ：将 b.length字节从指定的字节数组写入此输出流。
* `public void write(byte[] b, int off, int len)` ：从指定的字节数组写入len字节，从偏移量off开始输出到此输出流。
* `public abstract void write(int b)` ：将指定的字节输出流。

> tips：close方法，当完成流的操作时，必须调用此方法，释放系统资源。
>

#### 20.2.3 FileOutputStream类

`OutputStream` 有很多子类，我们从最简单的一个子类开始。

`java.io.FileOutputStream` 类是文件输出流，用于将数据写出到文件。

**构造方法**

* `public FileOutputStream(File file)` ：创建文件输出流以写入由指定的File对象表示的文件。
* `public FileOutputStream(String name)` ： 创建文件输出流以指定的名称写入文件。

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有这个文件，会创建该文件。如果有这个文件，会清空这个文件的数据。

构造举例，代码如下：

```java
public class FileOutputStreamConstructor throws IOException {
    public static void main(String[] args) {
        // 使用File对象创建流对象
        File file = new File("a.txt");
        FileOutputStream fos = new FileOutputStream(file);
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("b.txt");
    }
}
```

**写出字节数据**

1. **写出字节**： write(int b) 方法，每次可以写出一个字节数据，代码使用演示

   ```java
   public class FOSWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileOutputStream fos = new FileOutputStream("fos.txt");
           // 写出数据
           fos.write(97); // 写出第1个字节
           fos.write(98); // 写出第2个字节
           fos.write(99); // 写出第3个字节
           // 关闭资源
           fos.close();
       }
   }
   
   // 输出结果：abc
   ```

   > tips：
   >
   > 1. 虽然参数为int类型四个字节，但是只会保留一个字节的信息写出。
   > 2. 流操作完毕后，必须释放系统资源，调用close方法，千万记得。

2. **写出字节数组**： write(byte[] b) ，每次可以写出数组中的数据，代码使用演示：

   ```java
   public class FOSWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileOutputStream fos = new FileOutputStream("fos.txt");
           // 字符串转换为字节数组
           byte[] b = "黑马程序员".getBytes();
           // 写出字节数组数据
           fos.write(b);
           // 关闭资源
           fos.close();
       }
   }
   
   // 输出结果：黑马程序员
   ```

3. **写出指定长度字节数组**： `write(byte[] b, int off, int len)` ,每次写出从off索引开始，len个字节，代码使用演示：

   ```java
   public class FOSWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileOutputStream fos = new FileOutputStream("fos.txt");
           // 字符串转换为字节数组
           byte[] b = "abcde".getBytes();
           // 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
           fos.write(b,2,2);
           // 关闭资源
           fos.close();
       }
   } 
   //输出结果：cd
   ```

**数据追加续写**

经过以上的演示，每次程序运行，创建输出流对象，都会清空目标文件中的数据。如何保留目标文件中数据，还能继续添加新数据呢？

* `public FileOutputStream(File file, boolean append)` ： 创建文件输出流以写入由指定的 File对象表示的文件。
* `public FileOutputStream(String name, boolean append)` ： 创建文件输出流以指定的名称写入文件。

这两个构造方法，参数中都需要传入一个boolean类型的值， true 表示追加数据， false 表示清空原有数据。

这样创建的输出流对象，就可以指定是否追加续写了，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt"，true);
        // 字符串转换为字节数组
        byte[] b = "abcde".getBytes();
        // 追加的方式写入文件
        fos.write(b);
        // 关闭资源
        fos.close();
    }
} 
// 文件操作前：cd
// 文件操作后：cdabcde
```

**写出换行**

Windows系统里，换行符号是 `\r\n` 

代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");
        // 定义字节数组
        byte[] words = {97,98,99,100,101};
        // 遍历数组
        for (int i = 0; i < words.length; i++) {
            // 写出一个字节
            fos.write(words[i]);
            // 写出一个换行, 换行符号转成数组写出
            fos.write("\r\n".getBytes());
        } 
        // 关闭资源
        fos.close();
    }
}

// 输出结果：
// a 
// b 
// c 
// d 
// e
```

> 回车符 `\r` 和换行符 `\n` ：
>
> * 回车符：回到一行的开头（return）。
> * 换行符：下一行（newline）。
>
> 系统中的换行：
>
> * Windows系统里，每行结尾是 `回车+换行` ，即 `\r\n` ；
> * Unix系统里，每行结尾只有`换行` ，即  `\n` ；
> * Mac系统里，每行结尾是 `回车` ，即 `\r` 。从 Mac OS X开始与Linux统一。

#### 20.2.4 字节输入流：InputStream

`java.io.InputStream` 抽象类是表示字节输入流的所有类的超类，可以读取字节信息到内存中。它定义了字节输入流的基本共性功能方法。

* `public void close()` ：关闭此输入流并释放与此流相关联的任何系统资源。
* `public abstract int read()` ： 从输入流读取数据的下一个字节。
* `public int read(byte[] b)` ： 从输入流中读取一些字节数，并将它们存储到字节数组 b中 。

> tips：close方法，当完成流的操作时，必须调用此方法，释放系统资源。
>

#### 20.2.5 FileInputStream类

`java.io.FileInputStream` 类是文件输入流，从文件中读取字节。

**构造方法**

* `FileInputStream(File file)` ： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的File对象file命名。
* `FileInputStream(String name)` ： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名name命名。

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有该文件,会抛出 `FileNotFoundException`。

构造举例，代码如下：

```java
public class FileInputStreamConstructor throws IOException{
    public static void main(String[] args) {
        // 使用File对象创建流对象
        File file = new File("a.txt");
        FileInputStream fos = new FileInputStream(file);
        // 使用文件名称创建流对象
        FileInputStream fos = new FileInputStream("b.txt");
    }
}
```

**读取字节数据**

1. **读取字节**： **read 方法，每次可以读取一个字节的数据，提升为int类型**，读取到文件末尾，返回 -1 ，代码使用演示：

   ```java
   public class FISRead {
       public static void main(String[] args) throws IOException{
           // 使用文件名称创建流对象
           FileInputStream fis = new FileInputStream("read.txt");
           // 读取数据，返回一个字节
           int read = fis.read();
           System.out.println((char) read);
           read = fis.read();
           System.out.println((char) read);
           read = fis.read();
           System.out.println((char) read);
           read = fis.read();
           System.out.println((char) read);
           read = fis.read();
           System.out.println((char) read);
           // 读取到末尾,返回‐1
           read = fis.read();
           System.out.println(read);
           // 关闭资源
           fis.close();
       }
   } 
   // 输出结果：
   // a 
   // b 
   // c 
   // d 
   // e 
   // ‐1
   ```

   循环改进读取方式，代码使用演示：

   ```java
   public class FISRead {
       public static void main(String[] args) throws IOException{
           // 使用文件名称创建流对象
           FileInputStream fis = new FileInputStream("read.txt");
           // 定义变量，保存数据
           int b;
           // 循环读取
           while ((b = fis.read())!=‐1) {
               System.out.println((char)b);
           } 
           // 关闭资源
           fis.close();
       }
   } 
   //输出结果：
   // a 
   // b 
   // c 
   // d 
   // e
   ```

   > tips：
   >
   > 1. 虽然读取了一个字节，但是会自动提升为int类型。
   > 2. 流操作完毕后，必须释放系统资源，调用close方法，千万记得。

2. **使用字节数组读取**：`read(byte[] b)` ，每次读取b的长度个字节到数组中，返回读取到的有效字节个数，读取到末尾时，返回 -1 ，代码使用演示：

   ```java
   public class FISRead {
       public static void main(String[] args) throws IOException{
           // 使用文件名称创建流对象.
           FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
           // 定义变量，作为有效个数
           int len;
           // 定义字节数组，作为装字节数据的容器
           byte[] b = new byte[2];
           // 循环读取
           while (( len= fis.read(b))!=‐1) {
               // 每次读取后,把数组变成字符串打印
               System.out.println(new String(b));
           } 
           // 关闭资源
           fis.close();
       }
   } 
   // 输出结果：
   // ab
   // cd
   // ed
   ```

   错误数据 d ，是由于最后一次读取时，只读取一个字节 e ，数组中，上次读取的数据没有被完全替换，所以要通过 len ，获取有效的字节，代码使用演示：

   ```java
   public class FISRead {
       public static void main(String[] args) throws IOException{
           // 使用文件名称创建流对象.
           FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
           // 定义变量，作为有效个数
           int len;
           // 定义字节数组，作为装字节数据的容器
           byte[] b = new byte[2];
           // 循环读取
           while (( len= fis.read(b))!=‐1) {
               // 每次读取后,把数组的有效字节部分，变成字符串打印
               System.out.println(new String(b，0，len));// len 每次读取的有效字节个数
           } 
           // 关闭资源
           fis.close();
       }
   } 
   // 输出结果：
   // ab
   // cd
   // e
   ```

   > tips：使用数组读取，每次读取多个字节，减少了系统间的IO操作次数，从而提高了读写的效率，建议开发中使用。

#### 20.2.6 字节流练习：图片复制

**复制原理图解**

![复制原理图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515958.PNG)

**案例实现**

复制图片文件，代码使用演示：

```java
public class Copy {
    public static void main(String[] args) throws IOException {
        // 1.创建流对象
        // 1.1 指定数据源
        FileInputStream fis = new FileInputStream("D:\\test.jpg");
        // 1.2 指定目的地
        FileOutputStream fos = new FileOutputStream("test_copy.jpg");
        // 2.读写数据
        // 2.1 定义数组
        byte[] b = new byte[1024];
        // 2.2 定义长度
        int len;
        // 2.3 循环读取
        while ((len = fis.read(b))!=‐1) {
            // 2.4 写出数据
            fos.write(b, 0 , len);
        } 
        // 3.关闭资源
        fos.close();
        fis.close();
    }
}
```

> tips：流的关闭原则：先开后关，后开先关。
>

### 20.3 字符流

当使用字节流读取文本文件时，可能会有一个小问题。就是遇到中文字符时，可能不会显示完整的字符，那是因为一个中文字符可能占用多个字节存储。所以Java提供一些字符流类，以字符为单位读写数据，专门用于处理文本文件。

#### 20.3.1 字符输入流Reader

`java.io.Reader` 抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。

* `public void close()` ：关闭此流并释放与此流相关联的任何系统资源。
* `public int read()` ： 从输入流读取一个字符。
* `public int read(char[] cbuf)` ： 从输入流中读取一些字符，并将它们存储到字符数组cbuf中 。

#### 20.3.2 FileReader类

`java.io.FileReader` 类是读取字符文件的便利类。构造时使用**系统默认的字符编码**和**默认字节缓冲区**。

> tips：
>
> 1. 字符编码：字节与字符的对应规则。Windows系统的中文编码默认是GBK编码表。
> 2. idea中为UTF-8。
> 3. 字节缓冲区：一个字节数组，用来临时存储字节数据。

**构造方法**

* `FileReader(File file)` ： 创建一个新的 FileReader ，给定要读取的File对象。
* `FileReader(String fileName)` ： 创建一个新的 FileReader ，给定要读取的文件的名称。

当你创建一个流对象时，必须传入一个文件路径。类似于FileInputStream 。

构造举例，代码如下：

```java
public class FileReaderConstructor throws IOException{
    public static void main(String[] args) {
        // 使用File对象创建流对象
        File file = new File("a.txt");
        FileReader fr = new FileReader(file);
        // 使用文件名称创建流对象
        FileReader fr = new FileReader("b.txt");
    }
}
```

**读取字符数据**


1. **读取字符**： read 方法，每次可以读取**一个字符**的数据，**提升为int类型**，读取到文件末尾，返回 -1 ，循环读取，代码使用演示：
   
   ```java
   public class FRRead {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileReader fr = new FileReader("read.txt");
           // 定义变量，保存数据
           int b;
           while ((b = fr.read())!=‐1) {
               System.out.println((char)b);
           } 
           // 关闭资源
           fr.close();
       }
   } 
   // 输出结果：
   // 黑 
   // 马 
   // 程 
   // 序 
   // 员
   ```
   
   > tips：虽然读取了一个字符，但是会自动提升为int类型。


2. **使用字符数组读取**： `read(char[] cbuf)` ，每次读取b的长度个字符到数组中，返回读取到的有效字符个数，读取到末尾时，返回 -1 ，代码使用演示：

   ```java
   public class FRRead {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileReader fr = new FileReader("read.txt");
           // 定义变量，保存有效字符个数
           int len;
           // 定义字符数组，作为装字符数据的容器
           char[] cbuf = new char[2];
           // 循环读取
           while ((len = fr.read(cbuf))!=‐1) {
               System.out.println(new String(cbuf));
           } 
           // 关闭资源
           fr.close();
       }
   } 
   // 输出结果：
   // 黑马
   // 程序
   // 员序
   ```

   获取有效的字符改进，代码使用演示：

   ```java
   public class FISRead {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileReader fr = new FileReader("read.txt");
           // 定义变量，保存有效字符个数
           int len;
           // 定义字符数组，作为装字符数据的容器
           char[] cbuf = new char[2];
           // 循环读取
           while ((len = fr.read(cbuf))!=‐1) {
               System.out.println(new String(cbuf,0,len));
           } 
           // 关闭资源
           fr.close();
       }
   } 
   // 输出结果：
   // 黑马
   // 程序
   // 员
   ```

#### 20.3.3 字符输出流：Writer

`java.io.Writer` 抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。

* `void write(int c)` 写入单个字符。
* `void write(char[] cbuf)` 写入字符数组。
* `abstract void write(char[] cbuf, int off, int len)` 写入字符数组的某一部分，off数组的开始索引，len写的字符个数。
* `void write(String str)` 写入字符串。
* `void write(String str, int off, int len)` 写入字符串的某一部分，off字符串的开始索引，len写的字符个数。
* `void flush()` 刷新该流的缓冲。
* `void close()` 关闭此流，但要先刷新它。

#### 20.3.4 FileWriter类

`java.io.FileWriter` 类是写出字符到文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

**构造方法**

* `FileWriter(File file)` ： 创建一个新的 FileWriter，给定要读取的File对象。
* `FileWriter(String fileName)` ： 创建一个新的 FileWriter，给定要读取的文件的名称。

当你创建一个流对象时，必须传入一个文件路径，类似于FileOutputStream。

构造举例，代码如下：

```java
public class FileWriterConstructor {
    public static void main(String[] args) throws IOException {
        // 使用File对象创建流对象
        File file = new File("a.txt");
        FileWriter fw = new FileWriter(file);
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("b.txt");
    }
}
```

**基本写出数据**

写出字符：`write(int b)` 方法，每次可以写出一个字符数据，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");
        // 写出数据
        fw.write(97); // 写出第1个字符
        fw.write('b'); // 写出第2个字符
        fw.write('C'); // 写出第3个字符
        fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字
        /*
         【注意】关闭资源时,与FileOutputStream不同。
         如果不关闭,数据只是保存到缓冲区，并未保存到文件。
         */
        // fw.close();
    }
} 
// 输出结果：
// abC田
```

> tips：
>
> 1. 虽然参数为int类型四个字节，但是只会保留一个字符的信息写出。
> 2. 未调用close方法，数据只是保存到了缓冲区，并未写出到文件中。

**关闭和刷新**

因为内置缓冲区的原因，如果不关闭输出流，无法写出字符到文件中。但是关闭的流对象，是无法继续写出数据的。如果我们既想写出数据，又想继续使用流，就需要 flush 方法了。

* flush ：刷新缓冲区，流对象可以继续使用。
* close ：先刷新缓冲区，然后通知系统释放资源。流对象不可以再被使用了。

代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");
        // 写出数据，通过flush
        fw.write('刷'); // 写出第1个字符
        fw.flush();
        fw.write('新'); // 继续写出第2个字符，写出成功
        fw.flush();
        // 写出数据，通过close
        fw.write('关'); // 写出第1个字符
        fw.close();
        fw.write('闭'); // 继续写出第2个字符,【报错】java.io.IOException: Stream closed
        fw.close();
    }
}
```

> tips：即便是flush方法写出了数据，操作的最后还是要调用close方法，释放系统资源。

**写出其他数据**

1. **写出字符数组** ： `write(char[] cbuf)` 和 `write(char[] cbuf, int off, int len)` ，每次可以写出字符数组中的数据，用法类似FileOutputStream，代码使用演示：

   ```java
   public class FWWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileWriter fw = new FileWriter("fw.txt");
           // 字符串转换为字节数组
           char[] chars = "黑马程序员".toCharArray();
           // 写出字符数组
           fw.write(chars); // 黑马程序员
           // 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
           fw.write(b,2,2); // 程序
           // 关闭资源
           fos.close();
       }
   }
   ```

2. **写出字符串**： `write(String str)` 和 `write(String str, int off, int len)` ，每次可以写出字符串中的数据，更为方便，代码使用演示：

   ```java
   public class FWWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象
           FileWriter fw = new FileWriter("fw.txt");
           // 字符串
           String msg = "黑马程序员";
           // 写出字符数组
           fw.write(msg); //黑马程序员
           // 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
           fw.write(msg,2,2); // 程序
           // 关闭资源
           fos.close();
       }
   }
   ```

3. 续写和换行：操作类似于FileOutputStream。

   ```java
   public class FWWrite {
       public static void main(String[] args) throws IOException {
           // 使用文件名称创建流对象，可以续写数据
           FileWriter fw = new FileWriter("fw.txt"，true);
           // 写出字符串
           fw.write("黑马");
           // 写出换行
           fw.write("\r\n");
           // 写出字符串
           fw.write("程序员");
           // 关闭资源
           fw.close();
       }
   }
   
   // 输出结果:
   // 黑马
   // 程序员
   ```

> tips：
>
> 1. 字符流，只能操作文本文件，不能操作图片，视频等非文本文件。
> 2. 当我们单纯读或者写文本文件时使用字符流其他情况使用字节流。

### 20.4 IO异常的处理

#### 20.4.1 JDK7前处理

之前的入门练习，我们一直把异常抛出，而实际开发中并不能这样处理，建议使用 try...catch...finally 代码块，处理异常部分，代码使用演示：

```java
public class HandleException1 {
    public static void main(String[] args) {
        // 声明变量
        FileWriter fw = null;
        try {
            //创建流对象
            fw = new FileWriter("fw.txt");
            // 写出数据
            fw.write("黑马程序员"); //黑马程序员
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 20.4.2 JDK7的处理

还可以使用JDK7优化后的 **try-with-resource** 语句，该语句确保了每个资源在语句结束时关闭。所谓的资源（resource）是指在程序完成后，必须关闭的对象。

格式：

```java
try (创建流对象语句，如果多个,使用';'隔开) {
    // 读写数据
} catch (IOException e) {
    e.printStackTrace();
}
```

代码使用演示：

```java
public class HandleException2 {
    public static void main(String[] args) {
        // 创建流对象
        try ( FileWriter fw = new FileWriter("fw.txt"); ) {
            // 写出数据
            fw.write("黑马程序员"); //黑马程序员
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 20.4.3 JDK9的改进

JDK9中 **try-with-resource** 的改进，对于引入对象的方式，支持的更加简洁。被引入的对象，同样可以自动关闭，无需手动close，我们来了解一下格式。

改进前格式：

```java
// 被final修饰的对象
final Resource resource1 = new Resource("resource1");
// 普通对象
Resource resource2 = new Resource("resource2");
// 引入方式：创建新的变量保存
try (Resource r1 = resource1;
     Resource r2 = resource2) {
// 使用对象
}
```

改进后格式：

```java
// 被final修饰的对象
final Resource resource1 = new Resource("resource1");
// 普通对象
Resource resource2 = new Resource("resource2");
// 引入方式：直接引入
try (resource1; resource2) {
    // 使用对象
}
```

改进后，代码使用演示：

```java
public class TryDemo {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        final FileReader fr = new FileReader("in.txt");
        FileWriter fw = new FileWriter("out.txt");
        // 引入到try中
        try (fr; fw) {
            // 定义变量
            int b;
            // 读取数据
            while ((b = fr.read())!=‐1) {
                // 写出数据
                fw.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 20.5 属性集

#### 20.5.1 概述

`java.util.Properties` 继承于 Hashtable ，来表示一个持久的属性集。它使用键值结构存储数据，每个键及其对应值都是一个字符串。该类也被许多Java类使用，比如获取系统属性时，`System.getProperties` 方法就是返回一个 Properties 对象。

#### 20.5.2 Properties类

**构造方法**

`public Properties()` :创建一个空的属性列表。

**基本的存储方法**

* `public Object setProperty(String key, String value)` ： 保存一对属性。
* `public String getProperty(String key)` ：使用此属性列表中指定的键搜索属性值。
* `public Set<String> stringPropertyNames()` ：所有键的名称的集合。

```java
public class ProDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties properties = new Properties();
        // 添加键值对元素
        properties.setProperty("filename", "a.txt");
        properties.setProperty("length", "209385038");
        properties.setProperty("location", "D:\\a.txt");
        // 打印属性集对象
        System.out.println(properties);
        // 通过键,获取属性值
        System.out.println(properties.getProperty("filename"));
        System.out.println(properties.getProperty("length"));
        System.out.println(properties.getProperty("location"));
        // 遍历属性集,获取所有键的集合
        Set<String> strings = properties.stringPropertyNames();
        // 打印键值对
        for (String key : strings ) {
            System.out.println(key+" ‐‐ "+properties.getProperty(key));
        }
    }
} 
// 输出结果：
// {filename=a.txt, length=209385038, location=D:\a.txt}
// a.txt
// 209385038
// D:\a.txt
// filename ‐‐ a.txt
// length ‐‐ 209385038
// location ‐‐ D:\a.txt
```

**与流相关的方法**

`public void load(InputStream inStream)` ： 从字节输入流中读取键值对。

参数中使用了字节输入流，通过流对象，可以关联到某文件上，这样就能够加载文本中的数据了。文本数据格式:

```java
filename=a.txt
length=209385038
location=D:\a.txt
```

加载代码演示：

```java
public class ProDemo2 {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties pro = new Properties();
        // 加载文本中信息到属性集
        pro.load(new FileInputStream("read.txt"));
        // 遍历集合并打印
        Set<String> strings = pro.stringPropertyNames();
        for (String key : strings ) {
            System.out.println(key+" ‐‐ "+pro.getProperty(key));
        }
    }
} 
// 输出结果：
// filename ‐‐ a.txt
// length ‐‐ 209385038
// location ‐‐ D:\a.txt
```

> tips：文本中的数据，必须是键值对形式，可以使用空格、等号、冒号等符号分隔。 

## 21. 缓冲流&转换流&序列化流

### 21.1 缓冲流

昨天学习了基本的一些流，作为IO流的入门，今天我们要见识一些更强大的流。比如能够高效读写的缓冲流，能够转换编码的转换流，能够持久化存储对象的序列化流等等。这些功能更为强大的流，都是在基本的流对象基础之上创建而来的，就像穿上铠甲的武士一样，相当于是对基本流对象的一种增强。

#### 21.1.1 概述

缓冲流，也叫高效流，是对4个基本的 FileXxx 流的增强，所以也是4个流，按照数据类型分类：

* **字节缓冲流**： BufferedInputStream ， BufferedOutputStream
* **字符缓冲流**： BufferedReader ， BufferedWriter

缓冲流的基本原理，是在创建流对象时，会**创建一个内置的默认大小的缓冲区数组**，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

#### 21.1.2 字节缓冲流

**构造方法**

* `public BufferedInputStream(InputStream in)` ：创建一个新的缓冲输入流。
* `public BufferedOutputStream(OutputStream out)` ： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```

**效率测试**

查询API，缓冲流读写方法与基本的流是一致的，我们通过复制大文件（375MB），测试它的效率。

1. 基本流，代码如下：

   ```java
   public class BufferedDemo {
       public static void main(String[] args) throws FileNotFoundException {
           // 记录开始时间
           long start = System.currentTimeMillis();
           // 创建流对象
           try (
               FileInputStream fis = new FileInputStream("jdk9.exe");
               FileOutputStream fos = new FileOutputStream("copy.exe")
           ){
               // 读写数据
               int b;
               while ((b = fis.read()) != ‐1) {
                   fos.write(b);
               }
           } catch (IOException e) {
               e.printStackTrace();
           } 
           // 记录结束时间
           long end = System.currentTimeMillis();
           System.out.println("普通流复制时间:"+(end ‐ start)+" 毫秒");
       }
   } 
   
   // 十几分钟过去了...
   ```

2. 缓冲流，代码如下：

   ```java
   public class BufferedDemo {
       public static void main(String[] args) throws FileNotFoundException {
           // 记录开始时间
           long start = System.currentTimeMillis();
           // 创建流对象
           try (
               BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
               BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
           ){
               // 读写数据
               int b;
               while ((b = bis.read()) != ‐1) {
                   bos.write(b);
               }
           } catch (IOException e) {
               e.printStackTrace();
           } // 记录结束时间
           long end = System.currentTimeMillis();
           System.out.println("缓冲流复制时间:"+(end ‐ start)+" 毫秒");
       }
   } 
   
   // 缓冲流复制时间:8016 毫秒
   ```

如何更快呢？

使用数组的方式，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
        long start = System.currentTimeMillis();
        // 创建流对象
        try (
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
            // 读写数据
            int len;
            byte[] bytes = new byte[8*1024];
            while ((len = bis.read(bytes)) != ‐1) {
                bos.write(bytes, 0 , len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } 
        // 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流使用数组复制时间:"+(end ‐ start)+" 毫秒");
    }
} 

// 缓冲流使用数组复制时间:666 毫秒
```

#### 21.1.3 字符缓冲流

**构造方法**

* `public BufferedReader(Reader in)`：创建一个新的缓冲输入流。
* `public BufferedWriter(Writer out)` ： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```

**特有方法**

字符缓冲流的基本方法与普通字符流调用方式一致，不再阐述，我们来看它们具备的特有方法。

* **BufferedReader**： `public String readLine()` : 读一行文字。
* **BufferedWriter**： `public void newLine()` : 写一行行分隔符，由系统属性定义符号。

readLine 方法演示，代码如下：

```java
public class BufferedReaderDemo {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("in.txt"));
        // 定义字符串,保存读取的一行文字
        String line = null;
        // 循环读取,读取到最后返回null
        while ((line = br.readLine())!=null) {
            System.out.print(line);
            System.out.println("‐‐‐‐‐‐");
        } 
        // 释放资源
        br.close();
    }
}
```

newLine 方法演示，代码如下：

```java
public class BufferedWriterDemo throws IOException {
    public static void main(String[] args) throws IOException {
        // 创建流对象
        BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
        // 写出数据
        bw.write("黑马");
        // 写出换行
        bw.newLine();
        bw.write("程序");
        bw.newLine();
        bw.write("员");
        bw.newLine();
        // 释放资源
        bw.close();
    }
}

// 输出效果:
// 黑马
// 程序
// 员
```

#### 21.1.4 练习:文本排序

请将文本信息恢复顺序。

```
3.侍中、侍...
8.愿陛下...
4.将军向宠...
2.宫中府中...
1.先帝创业未半而中道崩殂...
9.今当远离...
6.臣本布衣...
7.先帝知臣谨慎...
5.亲贤臣...
```

**案例分析**

1. 逐行读取文本信息。
2. 解析文本信息到集合中。
3. 遍历集合，按顺序，写出文本信息。

**案例实现**

```java
public class BufferedTest {
    public static void main(String[] args) throws IOException {
        // 创建map集合,保存文本数据,键为序号,值为文字
        HashMap<String, String> lineMap = new HashMap<>();
        // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("in.txt"));
        BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
        // 读取数据
        String line = null;
        while ((line = br.readLine())!=null) {
            // 解析文本
            String[] split = line.split("\\.");
            // 保存到集合
            lineMap.put(split[0],split[1])
        } // 释放资源
        br.close();
        // 遍历map集合
        for (int i = 1; i <= lineMap.size(); i++) {
            String key = String.valueOf(i);
            // 获取map中文本
            String value = lineMap.get(key);
            // 写出拼接文本
            bw.write(key+"."+value);
            // 写出换行
            bw.newLine();
        } 
        // 释放资源
        bw.close();
    }
}
```

### 21.2 转换流

#### 21.2.1 字符编码和字符集

**字符编码**

计算机中储存的信息都是用二进制数表示的，而我们在屏幕上看到的数字、英文、标点符号、汉字等字符是二进制数转换之后的结果。按照某种规则，将字符存储到计算机中，称为**编码** 。反之，将存储在计算机中的二进制数按照某种规则解析显示出来，称为**解码** 。

比如说，按照A规则存储，同样按照A规则解析，那么就能显示正确的文本f符号。反之，按照A规则存储，再按照B规则解析，就会导致乱码现象。

**字符编码 Character Encoding** : 就是一套自然语言的字符与二进制数之间的对应规则。

**字符集**

**字符集Charset** ：也叫**编码表**。是一个系统支持的所有字符的集合，包括各国家文字、标点符号、图形符号、数字等。

计算机要准确的存储和识别各种字符集符号，需要进行字符编码，一套字符集必然至少有一套字符编码。常见字符集有ASCII字符集、GBK字符集、Unicode字符集等。

![字符集](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515528.PNG)

可见，当指定了**编码**，它所对应的**字符集**自然就指定了，所以编码才是我们最终要关心的。

* **ASCII字符集** ：

  * ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统，用于显示现代英语，主要包括控制字符（回车键、退格、换行键等）和可显示字符（英文大小写字符、阿拉伯数字和西文符号）。
  * 基本的ASCII字符集，使用7位（bits）表示一个字符，共128字符。
  * ASCII的扩展字符集使用8位（bits）表示一个字符，共256字符，方便支持欧洲常用字符。

* ISO-8859-1字符集：

  * 拉丁码表，别名Latin-1，用于显示欧洲使用的语言，包括荷兰、丹麦、德语、意大利语、西班牙语等。

  * ISO-5559-1使用单字节编码，兼容ASCII编码。

* GBxxx字符集：

  * GB就是国标的意思，是为了显示中文而设计的一套字符集。
  * **GB2312：简体中文码表**。一个小于127的字符的意义与原来相同。但两个大于127的字符连在一起时，就表示一个汉字，这样大约可以组合了包含7000多个简体汉字，此外数学符号、罗马希腊的字母、日文的假名们都编进去了，连在ASCII里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的"全角"字符，而原来在127号以下的那些就叫"半角"字符了。
  * **GBK：最常用的中文码表**。是在GB2312标准基础上的扩展规范，使用了双字节编码方案，共收录了21003个汉字，完全兼容GB2312标准，同时支持繁体汉字以及日韩汉字等。
  * **GB18030：最新的中文码表**。收录汉字70244个，采用多字节编码，每个字可以由1个、2个或4个字节组成。支持中国国内少数民族的文字，同时支持繁体汉字以及日韩汉字等。

* Unicode字符集 ：

  * Unicode编码系统为表达任意语言的任意字符而设计，是业界的一种标准，也称为统一码、标准万国码。
  * 它最多使用4个字节的数字来表达每个字母、符号，或者文字。
  * 有三种编码方案，UTF-8、UTF-16和UTF-32。最为常用的UTF-8编码。
  * **UTF-8编码**，可以用来表示Unicode标准中任何字符，它是电子邮件、网页及其他存储或传送文字的应用中，优先采用的编码。互联网工程工作小组（IETF）要求所有互联网协议都必须支持UTF-8编码。所以，我们开发Web应用，也要使用UTF-8编码。它使用一至四个字节为每个字符编码，编码规则：
    1. 128个US-ASCII字符，只需一个字节编码。
    2. 拉丁文等字符，需要二个字节编码。
    3. 大部分常用字（含中文），使用三个字节编码。
    4. 其他极少使用的Unicode辅助字符，使用四字节编码。

#### 21.2.2 编码引出的问题

在IDEA中，使用 FileReader 读取项目中的文本文件。由于IDEA的设置，都是默认的 UTF-8 编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码。那么如何读取GBK编码的文件呢？

```java
public class ReaderDemo {
    public static void main(String[] args) throws IOException {
        FileReader fileReader = new FileReader("E:\\File_GBK.txt");
        int read;
        while ((read = fileReader.read()) != ‐1) {
            System.out.print((char)read);
        } fileReader.close();
    }
} 
// 输出结果：
// ���
```

#### 21.2.3 InputStreamReader类

转换流 `java.io.InputStreamReader` ，是Reader的子类，**是从字节流到字符流的桥梁**。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以接受平台的默认字符集。

**构造方法**

* `InputStreamReader(InputStream in)` : 创建一个使用默认字符集的字符流。
* `InputStreamReader(InputStream in, String charsetName)` : 创建一个指定字符集的字符流。

构造举例，代码如下：

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

**指定编码读取**

```java
public class ReaderDemo2 {
    public static void main(String[] args) throws IOException {
        // 定义文件路径,文件为gbk编码
        String FileName = "E:\\file_gbk.txt";
        // 创建流对象,默认UTF8编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream(FileName));
        // 创建流对象,指定GBK编码
        InputStreamReader isr2 = new InputStreamReader(new FileInputStream(FileName), "GBK");
        // 定义变量,保存字符
        int read;
        // 使用默认编码字符流读取,乱码
        while ((read = isr.read()) != ‐1) {
            System.out.print((char)read); // ��Һ�
        } 
        isr.close();
        // 使用指定编码字符流读取,正常解析
        while ((read = isr2.read()) != ‐1) {
            System.out.print((char)read);// 大家好
        } 
        isr2.close();
    }
}
```

#### 21.2.4 OutputStreamWriter类

转换流 `java.io.OutputStreamWriter` ，是Writer的子类，是**从字符流到字节流的桥梁**。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。

**构造方法**

* `OutputStreamWriter(OutputStream in)` : 创建一个使用默认字符集的字符流。
* `OutputStreamWriter(OutputStream in, String charsetName)` : 创建一个指定字符集的字符流。

构造举例，代码如下：

```java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

**指定编码写出**

```java
public class OutputDemo {
    public static void main(String[] args) throws IOException {
        // 定义文件路径
        String FileName = "E:\\out.txt";
        // 创建流对象,默认UTF8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FileName));
        // 写出数据
        osw.write("你好"); // 保存为6个字节
        osw.close();
        // 定义文件路径
        String FileName2 = "E:\\out2.txt";
        // 创建流对象,指定GBK编码
        OutputStreamWriter osw2 = new OutputStreamWriter(new FileOutputStream(FileName2),"GBK");
        // 写出数据
        osw2.write("你好");// 保存为4个字节
        osw2.close();
    }
}
```

**转换流理解图解**

![转换流理解图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515297.PNG)

**转换流是字节与字符间的桥梁！**

#### 21.2.5 练习: 转换文件编码

将GBK编码的文本文件，转换为UTF-8编码的文本文件。

**案例分析**

1. 指定GBK编码的转换流，读取文本文件。
2. 使用UTF-8编码的转换流，写出文本文件。

案例实现

```java
public class TransDemo {
    public static void main(String[] args) {
        // 1.定义文件路径
        String srcFile = "file_gbk.txt";
        String destFile = "file_utf8.txt";
        // 2.创建流对象
        // 2.1 转换输入流,指定GBK编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream(srcFile) , "GBK");
        // 2.2 转换输出流,默认utf8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(destFile));
        // 3.读写数据
        // 3.1 定义数组
        char[] cbuf = new char[1024];
        // 3.2 定义长度
        int len;
        // 3.3 循环读取
        while ((len = isr.read(cbuf))!=‐1) {
            // 循环写出
            osw.write(cbuf,0,len);
        } 
        // 4.释放资源
        osw.close();
        isr.close();
    }
}
```

### 21.3 序列化

#### 21.3.1 概述

Java 提供了一种**对象序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该**对象的数据 、对象的类型和对象中存储的属性** 等信息。字节序列写出到文件之后，相当于文件中持久保存了一个对象的信息。

反之，该字节序列还可以从文件中读取回来，**重构对象，对它进行反序列化**。 对象的数据 、对象的类型和对象中存储的数据信息，都可以用来在内存中创建对象。看图理解序列化：

![对象序列化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515955.PNG)

#### 21.3.2 ObjectOutputStream类

`java.io.ObjectOutputStream` 类，将Java对象的原始数据类型写出到文件，实现对象的持久存储。

**构造方法**

`public ObjectOutputStream(OutputStream out)` ： 创建一个指定OutputStream的ObjectOutputStream。

构造举例，代码如下：

```java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```

**序列化操作**

一个对象要想序列化，必须满足两个条件:

  * 该类必须实现 `java.io.Serializable` 接口， `Serializable` 是一个**标记接口**，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出 `NotSerializableException` 。
  * 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用 `transient` 关键字修饰。

  ```java
public class Employee implements java.io.Serializable {
    public String name;
    public String address;
    public transient int age; // transient瞬态修饰成员,不会被序列化
    public void addressCheck() {
        System.out.println("Address check : " + name + " ‐‐ " + address);
    }
}
  ```

写出对象方法：`public final void writeObject (Object obj)` : 将指定的对象写出。

```java
public class SerializeDemo{
    public static void main(String [] args) {
        Employee e = new Employee();
        e.name = "zhangsan";
        e.address = "beiqinglu";
        e.age = 20;
        try {
            // 创建序列化流对象
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
            // 写出对象
            out.writeObject(e);
            // 释放资源
            out.close();
            fileOut.close();
            System.out.println("Serialized data is saved"); // 姓名，地址被序列化，年龄没有被序列化。
        } catch(IOException i) {
            i.printStackTrace();
        }
    }
} 

// 输出结果：
// Serialized data is saved
```

#### 21.3.3 ObjectInputStream类

`ObjectInputStream`反序列化流，将之前使用`ObjectOutputStream`序列化的原始数据恢复为对象。

**构造方法**

`public ObjectInputStream(InputStream in)` ： 创建一个指定InputStream的ObjectInputStream。

**反序列化操作1**：如果能找到一个对象的class文件，我们可以进行反序列化操作，调用 ObjectInputStream 读取对象的方法：

* `public final Object readObject()` : 读取一个对象。

  ```java
  public class DeserializeDemo {
      public static void main(String [] args) {
          Employee e = null;
          try {
              // 创建反序列化流
              FileInputStream fileIn = new FileInputStream("employee.txt");
              ObjectInputStream in = new ObjectInputStream(fileIn);
              // 读取一个对象
              e = (Employee) in.readObject();
              // 释放资源
              in.close();
              fileIn.close();
          }catch(IOException i) {
              // 捕获其他异常
              i.printStackTrace();
              return;
          }catch(ClassNotFoundException c) {
              // 捕获类找不到异常
              System.out.println("Employee class not found");
              c.printStackTrace();
              return;
          } 
          // 无异常,直接打印输出
          System.out.println("Name: " + e.name); // zhangsan
          System.out.println("Address: " + e.address); // beiqinglu
          System.out.println("age: " + e.age); // 0
      }
  }
  ```

* 对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个`ClassNotFoundException` 异常。 

**反序列化操作2**：另外，当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个 `InvalidClassException` 异常。发生这个异常的原因如下：

* 该类的序列版本号与从流中读取的类描述符的版本号不匹配
* 该类包含未知数据类型
* 该类没有可访问的无参数构造方法

`Serializable` 接口给需要序列化的类，提供了一个序列版本号。 `serialVersionUID` 该版本号的目的在于验证序列化的对象和对应类是否版本匹配。

```java
public class Employee implements java.io.Serializable {
    // 加入序列版本号
    private static final long serialVersionUID = 1L;
    public String name;
    public String address;
    // 添加新的属性 ,重新编译, 可以反序列化,该属性赋为默认值.
    public int eid;
    public void addressCheck() {
        System.out.println("Address check : " + name + " ‐‐ " + address);
    }
}
```

#### 21.3.4 练习：序列化集合

1. 将存有多个自定义对象的集合序列化操作，保存到 list.txt 文件中。
2. 反序列化 list.txt ，并遍历集合，打印对象信息。

**案例分析**

1. 把若干学生对象 ，保存到集合中。
2. 把集合序列化。
3. 反序列化读取时，只需要读取一次，转换为集合类型。
4. 遍历集合，可以打印所有的学生信息

**案例实现**

```java
public class SerTest {
    public static void main(String[] args) throws Exception {
        // 创建 学生对象
        Student student = new Student("老王", "laow");
        Student student2 = new Student("老张", "laoz");
        Student student3 = new Student("老李", "laol");
        ArrayList<Student> arrayList = new ArrayList<>();
        arrayList.add(student);
        arrayList.add(student2);
        arrayList.add(student3);
        // 序列化操作
        // serializ(arrayList);
        // 反序列化
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("list.txt"));
        // 读取对象,强转为ArrayList类型
        ArrayList<Student> list = (ArrayList<Student>)ois.readObject();
        for (int i = 0; i < list.size(); i++ ){
            Student s = list.get(i);
            System.out.println(s.getName()+"‐‐"+ s.getPwd());
        }
    } 
    private static void serializ(ArrayList<Student> arrayList) throws Exception {
        // 创建 序列化流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("list.txt"));
        // 写出对象
        oos.writeObject(arrayList);
        // 释放资源
        oos.close();
    }
}
```

### 21.4 打印流

#### 21.4.1 概述

平时我们在控制台打印输出，是调用 print 方法和 println 方法完成的，这两个方法都来自于`java.io.PrintStream` 类，该类能够方便地打印各种数据类型的值，是一种便捷的输出方式。

#### 21.4.2 PrintStream类

**构造方法**

`public PrintStream(String fileName) `： 使用指定的文件名创建一个新的打印流。

构造举例，代码如下：

```java
PrintStream ps = new PrintStream("ps.txt")；
```

**改变打印流向**

`System.out` 就是 `PrintStream` 类型的，只不过它的流向是系统规定的，打印在控制台上。不过，既然是流对象，我们就可以玩一个"小把戏"，改变它的流向。

```java
public class PrintDemo {
    public static void main(String[] args) throws IOException {
        // 调用系统的打印流,控制台直接输出97
        System.out.println(97);
        // 创建打印流,指定文件的名称
        PrintStream ps = new PrintStream("ps.txt");
        // 设置系统的打印流流向,输出到ps.txt
        System.setOut(ps);
        // 调用系统的打印流,ps.txt中输出97
        System.out.println(97);
    }
}
```

## 22. 网络编程

### 22.1 网络编程入门

#### 22.1.1 软件结构

**C/S结构** ：全称为Client/Server结构，是指客户端和服务器结构。常见程序有qq、迅雷等软件。

**B/S结构** ：全称为Browser/Server结构，是指浏览器和服务器结构。常见浏览器有谷歌、火狐等。

两种架构各有优势，但是无论哪种架构，都离不开网络的支持。网络编程，就是在一定的协议下，实现两台计算机的通信的程序。

#### 22.1.2 网络通信协议

**网络通信协议**：通信协议是对计算机必须遵守的规则，只有遵守这些规则，计算机之间才能进行通信。这就好比在道路中行驶的汽车一定要遵守交通规则一样，协议中对数据的传输格式、传输速率、传输步骤等做了统一规定，通信双方必须同时遵守，最终完成数据交换。

**TCP/IP协议**： 传输控制协议/因特网互联协议( Transmission Control Protocol/Internet Protocol)，是Internet最基本、最广泛的协议。它定义了计算机如何连入因特网，以及数据如何在它们之间传输的标准。它的内部包含一系列的用于处理数据通信的协议，并采用了4层的分层模型，每一层都呼叫它的下一层所提供的协议来完成自己的需求。

![四层模型](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515331.PNG)

#### 22.1.3 协议分类

通信的协议还是比较复杂的，`java.net` 包中包含的类和接口，它们提供低层次的通信细节。我们可以直接使用这些类和接口，来专注于网络程序开发，而不用考虑通信的细节。

`java.net` 包中提供了两种常见的网络协议的支持：

**TCP**：传输控制协议 (Transmission Control Protocol)。TCP协议是面向连接的通信协议，即传输数据之前，在发送端和接收端建立逻辑连接，然后再传输数据，它提供了两台计算机之间可靠无差错的数据传输。

三次握手：TCP协议中，在发送数据的准备阶段，客户端与服务器之间的三次交互，以保证连接的可靠。

* 第一次握手，客户端向服务器端发出连接请求，等待服务器确认。
* 第二次握手，服务器端向客户端回送一个响应，通知客户端收到了连接请求。
* 第三次握手，客户端再次向服务器端发送确认信息，确认连接。整个交互过程如下图所示。

![三次握手](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515055.PNG)

完成三次握手，连接建立后，客户端和服务器就可以开始进行数据传输了。由于这种面向连接的特性，TCP协议可以保证传输数据的安全，所以应用十分广泛，例如下载文件、浏览网页等。

**UDP**：用户数据报协议(User Datagram Protocol)。UDP协议是一个面向无连接的协议。传输数据时，不需要建立连接，不管对方端服务是否启动，直接将数据、数据源和目的地都封装在数据包中，直接发送。每个数据包的大小限制在64k以内。它是不可靠协议，因为无连接，所以传输速度快，但是容易丢失数据。日常应用中,例如视频会议、QQ聊天等。

#### 22.1.4 网络编程三要素

**要素1：协议**：计算机网络通信必须遵守的规则，已经介绍过了，不再赘述。

**要素2：IP地址：指互联网协议地址（Internet Protocol Address）**，俗称IP。IP地址用来给一个网络中的计算机设备做唯一的编号。假如我们把“个人电脑”比作“一台电话”的话，那么“IP地址”就相当于“电话号码”。

* **IP地址分类**：

  * IPv4：是一个32位的二进制数，通常被分为4个字节，表示成 a.b.c.d 的形式，例如 192.168.65.100 。其中a、b、c、d都是0~255之间的十进制整数，那么最多可以表示42亿个。

  * IPv6：由于互联网的蓬勃发展，IP地址的需求量愈来愈大，但是网络地址资源有限，使得IP的分配越发紧张。有资料显示，全球IPv4地址在2011年2月分配完毕。

    为了扩大地址空间，拟通过IPv6重新定义地址空间，采用128位地址长度，每16个字节一组，分成8组十六进制数，表示成 ABCD:EF01:2345:6789:ABCD:EF01:2345:6789 ，号称可以为全世界的每一粒沙子编上一个网址，这样就解决了网络地址资源数量不够的问题。

* **常用命令**：

  * 查看本机IP地址，在控制台输入：`ipconfig`

  * 检查网络是否连通，在控制台输入：

  ```java
  ping 空格 IP地址
  ping 220.181.57.216
  ```

* **特殊的IP地址**

  本机IP地址： `127.0.0.1` 、 `localhost` 。

**要素3：端口号**

* 网络的通信，本质上是两个进程（应用程序）的通信。每台计算机都有很多的进程，那么在网络通信时，如何区分这些进程呢？ 

* 如果说IP地址可以唯一标识网络中的设备，那么**端口号就可以唯一标识设备中的进程（应用程序）了**。 

* **端口号：用两个字节表示的整数，它的取值范围是0~65535**。其中，0~1023之间的端口号用于一些知名的网络服务和应用，普通的应用程序需要使用1024以上的端口号。如果端口号被另外一个服务或应用所占用，会导致当前程序启动失败。

利用 **协议 + IP地址 + 端口号** 三元组合，就可以标识网络中的进程了，那么进程间的通信就可以利用这个标识与其它进程进行交互。

### 22.1 TCP通信程序

#### 22.2.1 概述

TCP通信能实现两台计算机之间的数据交互，通信的两端，要严格区分为客户端（Client）与服务端（Server）。

**两端通信时步骤：**

1. 服务端程序，需要事先启动，等待客户端的连接。
2. 客户端主动连接服务器端，连接成功才能通信。服务端不可以主动连接客户端。

**在Java中，提供了两个类用于实现TCP通信程序：**

1. 客户端： `java.net.Socket` 类表示。创建 Socket 对象，向服务端发出连接请求，服务端响应请求，两者建立连接开始通信。
2. 服务端： `java.net.ServerSocket` 类表示。创建 ServerSocket 对象，相当于开启一个服务，并等待客户端的连接。

#### 22.2.2 Socket类

Socket 类：该类实现客户端套接字，套接字指的是两台设备之间通讯的端点。

**构造方法**

`public Socket(String host, int port)` :创建套接字对象并将其连接到指定主机上的指定端口号。如果指定的host是null ，则相当于指定地址为回送地址。

> tips：回送地址(127.x.x.x) 是本机回送地址（Loopback Address），主要用于网络软件测试以及本地机进程间通信，无论什么程序，一旦使用回送地址发送数据，立即返回，不进行任何网络传输。

构造举例，代码如下：

```java
Socket client = new Socket("127.0.0.1", 6666);
```

**成员方法**

* `public InputStream getInputStream()` ： 返回此套接字的输入流。
* 如果此Scoket具有相关联的通道，则生成的InputStream的所有操作也关联该通道。
  
* 关闭生成的InputStream也将关闭相关的Socket。
  
* `public OutputStream getOutputStream()` ： 返回此套接字的输出流。
* 如果此Scoket具有相关联的通道，则生成的OutputStream的所有操作也关联该通道。
  * 关闭生成的OutputStream也将关闭相关的Socket。
  
* `public void close()` ：关闭此套接字。

  * 一旦一个socket被关闭，它不可再使用。
  * 关闭此socket也将关闭相关的InputStream和OutputStream 。

* `public void shutdownOutput()` ： 禁用此套接字的输出流。
* 任何先前写出的数据将被发送，随后终止输出流。

#### 22.2.3 ServerSocket类

`ServerSocket` 类：这个类实现了服务器套接字，该对象等待通过网络的请求。

**构造方法**

`public ServerSocket(int port)` ：使用该构造方法在创建ServerSocket对象时，就可以将其绑定到一个指定的端口号上，参数port就是端口号。

构造举例，代码如下：

```java
ServerSocket server = new ServerSocket(6666);
```

**成员方法**

`public Socket accept()` ：侦听并接受连接，返回一个新的Socket对象，用于和客户端实现通信。该方法会一直阻塞直到建立连接。

#### 22.2.4 简单的TCP网络程序

TCP通信分析图解

1. 【服务端】启动，创建ServerSocket对象，等待连接。


2. 【客户端】启动，创建Socket对象，请求连接。
3. 【服务端】接收连接，调用accept方法，并返回一个Socket对象。
4. 【客户端】Socket对象，获取OutputStream，向服务端写出数据。
5. 【服务端】Scoket对象，获取InputStream，读取客户端发送的数据。

到此，客户端向服务端发送数据成功。

![TCP通信分析图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515306.PNG)

自此，服务端向客户端回写数据。

6. 【服务端】Socket对象，获取OutputStream，向客户端回写数据。

7. 【客户端】Scoket对象，获取InputStream，解析回写数据。

8. 【客户端】释放资源，断开连接。

**客户端向服务器发送数据**

* **服务端实现：**

```java
public class ServerTCP {
    public static void main(String[] args) throws IOException {
        System.out.println("服务端启动 , 等待连接 .... ");
        // 1.创建 ServerSocket对象，绑定端口，开始等待连接
        ServerSocket ss = new ServerSocket(6666);
        // 2.接收连接 accept 方法, 返回 socket 对象.
        Socket server = ss.accept();
        // 3.通过socket 获取输入流
        InputStream is = server.getInputStream();
        // 4.一次性读取数据
        // 4.1 创建字节数组
        byte[] b = new byte[1024];
        // 4.2 据读取到字节数组中.
        int len = is.read(b);
        // 4.3 解析数组,打印字符串信息
        String msg = new String(b, 0, len);
        System.out.println(msg);
        //5.关闭资源.
        is.close();
        server.close();
    }
}
```

* **客户端实现：**

```java
public class ClientTCP {
    public static void main(String[] args) throws Exception {
        System.out.println("客户端 发送数据");
        // 1.创建 Socket ( ip , port ) , 确定连接到哪里.
        Socket client = new Socket("localhost", 6666);
        // 2.获取流对象 . 输出流
        OutputStream os = client.getOutputStream();
        // 3.写出数据.
        os.write("你好么? tcp ,我来了".getBytes());
        // 4. 关闭资源 .
        os.close();
        client.close();
    }
}
```

**服务器向客户端回写数据**

* **服务端实现：**

```java
public class ServerTCP {
    public static void main(String[] args) throws IOException {
        System.out.println("服务端启动 , 等待连接 .... ");
        // 1.创建 ServerSocket对象，绑定端口，开始等待连接
        ServerSocket ss = new ServerSocket(6666);
        // 2.接收连接 accept 方法, 返回 socket 对象.
        Socket server = ss.accept();
        // 3.通过socket 获取输入流
        InputStream is = server.getInputStream();
        // 4.一次性读取数据
        // 4.1 创建字节数组
        byte[] b = new byte[1024];
        // 4.2 据读取到字节数组中.
        int len = is.read(b);
        // 4.3 解析数组,打印字符串信息
        String msg = new String(b, 0, len);
        System.out.println(msg);
        // =================回写数据=======================
        // 5. 通过 socket 获取输出流
        OutputStream out = server.getOutputStream();
        // 6. 回写数据
        out.write("我很好,谢谢你".getBytes());
        // 7.关闭资源.
        out.close();
        is.close();
        server.close();
    }
}
```

* **客户端实现：**

```java
public class ClientTCP {
    public static void main(String[] args) throws Exception {
        System.out.println("客户端 发送数据");
        // 1.创建 Socket ( ip , port ) , 确定连接到哪里.
        Socket client = new Socket("localhost", 6666);
        // 2.通过Scoket,获取输出流对象
        OutputStream os = client.getOutputStream();
        // 3.写出数据.
        os.write("你好么? tcp ,我来了".getBytes());
        // ==============解析回写=========================
        // 4. 通过Scoket,获取 输入流对象
        InputStream in = client.getInputStream();
        // 5. 读取数据数据
        byte[] b = new byte[100];
        int len = in.read(b);
        System.out.println(new String(b, 0, len));
        // 6. 关闭资源 .
        in.close();
        os.close();
        client.close();
    }
}
```

### 22.3 综合案例

#### 22.3.1 文件上传案例

文件上传分析图解

1. 【客户端】输入流，从硬盘读取文件数据到程序中。
2. 【客户端】输出流，写出文件数据到服务端。
3. 【服务端】输入流，读取文件数据到服务端程序。
4. 【服务端】输出流，写出文件数据到服务器硬盘中。

![文件上传案例](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515078.PNG)

**基本实现**

* **服务端实现：**

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 建立连接
        Socket accept = serverSocket.accept();
        // 3. 创建流对象
        // 3.1 获取输入流,读取文件数据
        BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
        // 3.2 创建输出流,保存到本地 .
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.jpg"));
        // 4. 读写数据
        byte[] b = new byte[1024 * 8];
        int len;
        while ((len = bis.read(b)) != ‐1) {
            bos.write(b, 0, len);
        } //5. 关闭 资源
        bos.close();
        bis.close();
        accept.close();
        System.out.println("文件上传已保存");
    }
}
```

* **客户端实现：**

```java
public class FileUPload_Client {
    public static void main(String[] args) throws IOException {
        // 1.创建流对象
        // 1.1 创建输入流,读取本地文件
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("test.jpg"));
        // 1.2 创建输出流,写到服务端
        Socket socket = new Socket("localhost", 6666);
        BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
        //2.写出数据.
        byte[] b = new byte[1024 * 8 ];
        int len ;
        while (( len = bis.read(b))!=‐1) {
            bos.write(b, 0, len);
            bos.flush();
        } 
        System.out.println("文件发送完毕");
        // 3.释放资源
        bos.close();
        socket.close();
        bis.close();
        System.out.println("文件上传完毕 ");
    }
}
```

**文件上传优化分析**

1. 文件名称写死的问题

   服务端，保存文件的名称如果写死，那么最终导致服务器硬盘，只会保留一个文件，建议使用系统时间优化，保证文件名称唯一，代码如下：

   ```java
   FileOutputStream fis = new FileOutputStream(System.currentTimeMillis()+".jpg"); // 文件名称
   BufferedOutputStream bos = new BufferedOutputStream(fis);
   ```

2. 循环接收的问题

   服务端，指保存一个文件就关闭了，之后的用户无法再上传，这是不符合实际的，使用循环改进，可以不断的接收不同用户的文件，代码如下：

   ```java
   // 每次接收新的连接,创建一个Socket
   while(true){
       Socket accept = serverSocket.accept();
       //......
   }
   ```

3. 效率问题

   服务端，在接收大文件时，可能耗费几秒钟的时间，此时不能接收其他用户上传，所以，使用多线程技术优化，代码如下：

   ```java
   while(true){
       Socket accept = serverSocket.accept();
       // accept 交给子线程处理.
       new Thread(() ‐> {
           // ......
           InputStream bis = accept.getInputStream();
           // ......
       }).start();
   }
   ```

**优化实现**

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 循环接收,建立连接
        while (true) {
            Socket accept = serverSocket.accept();
            /*
             3. socket对象交给子线程处理,进行读写操作
             Runnable接口中,只有一个run方法,使用lambda表达式简化格式
             */
            new Thread(() ‐> {
                try (
                    //3.1 获取输入流对象
                    BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
                    //3.2 创建输出流对象, 保存到本地 .
                    FileOutputStream fis = new FileOutputStream(System.currentTimeMillis() + ".jpg");
                    BufferedOutputStream bos = new BufferedOutputStream(fis);) {
                    // 3.3 读写数据
                    byte[] b = new byte[1024 * 8];
                    int len;
                    while ((len = bis.read(b)) != ‐1) {
                        bos.write(b, 0, len);
                    } 
                    //4. 关闭 资源
                    bos.close();
                    bis.close();
                    accept.close();
                    System.out.println("文件上传已保存");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

**信息回写分析图解**

前四步与基本文件上传一致.

5. 【服务端】获取输出流，回写数据。
6. 【客户端】获取输入流，解析回写数据。

![信息回写](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251515688.png)

**回写实现**

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 循环接收,建立连接
        while (true) {
            Socket accept = serverSocket.accept();
            /*
             3. socket对象交给子线程处理,进行读写操作
             Runnable接口中,只有一个run方法,使用lambda表达式简化格式
             */
            new Thread(() ‐> {
                try (
                    //3.1 获取输入流对象
                    BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
                    //3.2 创建输出流对象, 保存到本地 .
                    FileOutputStream fis = new FileOutputStream(System.currentTimeMillis() + ".jpg");
                    BufferedOutputStream bos = new BufferedOutputStream(fis);
                ) {
                    // 3.3 读写数据
                    byte[] b = new byte[1024 * 8];
                    int len;
                    while ((len = bis.read(b)) != ‐1) {
                        bos.write(b, 0, len);
                    } 
                    // 4.=======信息回写===========================
                    System.out.println("back ........");
                    OutputStream out = accept.getOutputStream();
                    out.write("上传成功".getBytes());
                    out.close();
                    //================================
                    //5. 关闭 资源
                    bos.close();
                    bis.close();
                    accept.close();
                    System.out.println("文件上传已保存");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

客户端实现：

```java
public class FileUpload_Client {
    public static void main(String[] args) throws IOException {
        // 1.创建流对象
        // 1.1 创建输入流,读取本地文件
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("test.jpg"));
        // 1.2 创建输出流,写到服务端
        Socket socket = new Socket("localhost", 6666);
        BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
        //2.写出数据.
        byte[] b = new byte[1024 * 8 ];
        int len ;
        while (( len = bis.read(b))!=‐1) {
            bos.write(b, 0, len);
        } 
        // 关闭输出流,通知服务端,写出数据完毕
        socket.shutdownOutput();
        System.out.println("文件发送完毕");
        // 3. =====解析回写============
        InputStream in = socket.getInputStream();
        byte[] back = new byte[20];
        in.read(back);
        System.out.println(new String(back));
        in.close();
        // ============================
        // 4.释放资源
        socket.close();
        bis.close();
    }
}
```

#### 22.3.2 模拟B\S服务器

略了...

## 23. 函数式接口

### 22.1 函数式接口

#### 22.1.1 概念

函数式接口在Java中是指：**有且仅有一个抽象方法的接口**。

函数式接口，即适用于函数式编程场景的接口。而Java中的函数式编程体现就是Lambda，所以函数式接口就是可以适用于Lambda使用的接口。只有确保接口中有且仅有一个抽象方法，Java中的Lambda才能顺利地进行推导。

> 备注：“语法糖”是指使用更加方便，但是原理不变的代码语法。例如在遍历集合时使用的for-each语法，其实底层的实现原理仍然是迭代器，这便是“语法糖”。从应用层面来讲，Java中的Lambda可以被当做是匿名内部类的“语法糖”，但是二者在原理上是不同的。

#### 22.1.2 格式

只要确保接口中有且仅有一个抽象方法即可

```java
修饰符 interface 接口名称 {
    public abstract 返回值类型 方法名称(可选参数信息);
    // 其他非抽象方法内容
}
```

由于接口当中抽象方法的 public abstract 是可以省略的，所以定义一个函数式接口很简单：

```java
public interface MyFunctionalInterface {
    void myMethod();
}
```

#### 22.1.3 @FunctionalInterface注解

与 @Override 注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解： @FunctionalInterface 。该注解可用于一个接口的定义上：

```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void myMethod();
}
```

一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。需要注意的是，即使不使用该注解，只要满足函数式接口的定义，这仍然是一个函数式接口，使用起来都一样。

#### 22.1.4 自定义函数式接口

对于刚刚定义好的 MyFunctionalInterface 函数式接口，典型使用场景就是作为方法的参数：

```java
public class Demo09FunctionalInterface {
    // 使用自定义的函数式接口作为方法参数
    private static void doSomething(MyFunctionalInterface inter) {
        inter.myMethod(); // 调用自定义的函数式接口方法
    } 
    public static void main(String[] args) {
        // 调用使用函数式接口的方法
        doSomething(() ‐> System.out.println("Lambda执行啦！"));
    }
}
```

### 22.2 函数式编程

在兼顾面向对象特性的基础上，Java语言通过Lambda表达式与方法引用等，为开发者打开了函数式编程的大门。

下面我们做一个初探。

#### 22.2.1 Lambda的延迟执行

有些场景的代码执行后，结果不一定会被使用，从而造成性能浪费。而Lambda表达式是延迟执行的，这正好可以作为解决方案，提升性能。

**性能浪费的日志案例**

> 注：日志可以帮助我们快速的定位问题，记录程序运行过程中的情况，以便项目的监控和优化。

一种典型的场景就是对参数进行有条件使用，例如对日志消息进行拼接后，在满足条件的情况下进行打印输出：

```java
public class Demo01Logger {
    private static void log(int level, String msg) {
        if (level == 1) {
            System.out.println(msg);
        }
    } 
    
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
        log(1, msgA + msgB + msgC);
    }
}
```

这段代码存在问题：无论级别是否满足要求，作为 log 方法的第二个参数，三个字符串一定会首先被拼接并传入方法内，然后才会进行级别判断。如果级别不符合要求，那么字符串的拼接操作就白做了，存在性能浪费。

> 备注：SLF4J 是应用非常广泛的日志框架，它在记录日志时为了解决这种性能浪费的问题，并不推荐首先进行字符串的拼接，而是将字符串的若干部分作为可变参数传入方法中，仅在日志级别满足要求的情况下才会进行字符串拼接。
>
> 例如： LOGGER.debug("变量{}的取值为{}。", "os", "macOS") ，其中的大括号 {} 为占位符。如果满足日志级别要求，则会将“os”和“macOS”两个字符串依次拼接到大括号的位置；否则不会进行字符串拼接。这也是一种可行解决方案，但Lambda可以做到更好。

**体验Lambda的更优写法**

使用Lambda必然需要一个函数式接口：

```java
@FunctionalInterface
public interface MessageBuilder {
    String buildMessage();
}
```

然后对 log 方法进行改造：

```java
public class Demo02LoggerLambda {
    private static void log(int level, MessageBuilder builder) {
        if (level == 1) {
            System.out.println(builder.buildMessage());
        }
    }
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
        log(1, () ‐> msgA + msgB + msgC );
    }
}
```

这样一来，只有当级别满足要求的时候，才会进行三个字符串的拼接；否则三个字符串将不会进行拼接。

**证明Lambda的延迟**

下面的代码可以通过结果进行验证：

```java
public class Demo03LoggerDelay {
    private static void log(int level, MessageBuilder builder) {
        if (level == 1) {
            System.out.println(builder.buildMessage());
        }
    } 
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
        log(2, () ‐> {
            System.out.println("Lambda执行！");
            return msgA + msgB + msgC;
        });
    }
}
```

从结果中可以看出，在不符合级别要求的情况下，Lambda将不会执行。从而达到节省性能的效果。

> 扩展：实际上使用内部类也可以达到同样的效果，只是将代码操作延迟到了另外一个对象当中通过调用方法来完成。而是否调用其所在方法是在条件判断之后才执行的。

#### 22.2.2 Lambda作为参数和返回值

如果抛开实现原理不说，Java中的Lambda表达式可以被当作是匿名内部类的替代品。如果方法的参数是一个函数式接口类型，那么就可以使用Lambda表达式进行替代。使用Lambda表达式作为方法参数，其实就是使用函数式接口作为方法参数。

例如 `java.lang.Runnable` 接口就是一个函数式接口，假设有一个 startThread 方法使用该接口作为参数，那么就可以使用Lambda进行传参。这种情况其实和 Thread 类的构造方法参数为 Runnable 没有本质区别。

```java
public class Demo04Runnable {
    private static void startThread(Runnable task) {
        new Thread(task).start();
    } 
    public static void main(String[] args) {
        startThread(() ‐> System.out.println("线程任务执行！"));
    }
}
```

类似地，如果一个方法的返回值类型是一个函数式接口，那么就可以直接返回一个Lambda表达式。当需要通过一个方法来获取一个 java.util.Comparator 接口类型的对象作为排序器时，就可以调该方法获取。

```java
import java.util.Arrays;
import java.util.Comparator;
public class Demo06Comparator {
    private static Comparator<String> newComparator() {
        return (a, b) ‐> b.length() ‐ a.length();
    } 
    public static void main(String[] args) {
        String[] array = { "abc", "ab", "abcd" };
        System.out.println(Arrays.toString(array));
        Arrays.sort(array, newComparator());
        System.out.println(Arrays.toString(array));
    }
}
```

其中直接return一个Lambda表达式即可。

### 22.3 常用函数式接口

JDK提供了大量常用的函数式接口以丰富Lambda的典型使用场景，它们主要在`java.util.function`包中被提供。

下面是最简单的几个接口及使用示例。

#### 22.3.1 Supplier接口

`java.util.function.Supplier<T`> 接口仅包含一个无参的方法： `T get()` 。用来获取一个泛型参数指定类型的对象数据。由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“**对外提供**”一个符合泛型类型的对象数据。

```java
import java.util.function.Supplier;
public class Demo08Supplier {
    private static String getString(Supplier<String> function) {
        return function.get();
    } 
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        System.out.println(getString(() ‐> msgA + msgB));
    }
}
```

#### 22.3.2 练习：求数组元素最大值

**题目**

使用 Supplier 接口作为方法参数类型，通过Lambda表达式求出int数组中的最大值。提示：接口的泛型请使用`java.lang.Integer` 类。

解答

```java
public class Demo02Test {
    //定一个方法,方法的参数传递Supplier,泛型使用Integer
    public static int getMax(Supplier<Integer> sup){
        return sup.get();
    } 
    public static void main(String[] args) {
        int arr[] = {2,3,4,52,333,23};
        //调用getMax方法,参数传递Lambda
        int maxNum = getMax(()‐>{
            //计算数组的最大值
            int max = arr[0];
            for(int i : arr){
                if(i>max){
                    max = i;
                }
            } 
            return max;
        });
        System.out.println(maxNum);
    }
}
```

#### 22.3.3 Consumer接口

`java.util.function.Consumer<T>` 接口则正好与Supplier接口相反，它不是生产一个数据，而是消费一个数据，其数据类型由泛型决定。

**抽象方法：accept**

Consumer 接口中包含抽象方法 `void accept(T t)` ，意为消费一个指定泛型的数据。基本使用如：

```java
import java.util.function.Consumer;
public class Demo09Consumer {
    private static void consumeString(Consumer<String> function) {
        function.accept("Hello");
    } 
    public static void main(String[] args) {
        consumeString(s ‐> System.out.println(s));
    }
}
```

当然，更好的写法是使用方法引用。

**默认方法：andThen**

如果一个方法的参数和返回值全都是 Consumer 类型，那么就可以实现效果：消费数据的时候，首先做一个操作，然后再做一个操作，实现组合。而这个方法就是 Consumer 接口中的default方法 andThen 。下面是JDK的源代码：

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) ‐> { accept(t); after.accept(t); };
}
```

> 备注： `java.util.Objects` 的 `requireNonNull` 静态方法将会在参数为null时主动抛出`NullPointerException` 异常。这省去了重复编写if语句和抛出空指针异常的麻烦。
>

要想实现组合，需要两个或多个Lambda表达式即可，而 andThen 的语义正是“一步接一步”操作。例如两个步骤组合的情况：

```java
import java.util.function.Consumer;
public class Demo10ConsumerAndThen {
    private static void consumeString(Consumer<String> one, Consumer<String> two) {
        one.andThen(two).accept("Hello");
    } 
    public static void main(String[] args) {
        consumeString(
            s ‐> System.out.println(s.toUpperCase()),
            s ‐> System.out.println(s.toLowerCase()));
    }
}
```

运行结果将会首先打印完全大写的HELLO，然后打印完全小写的hello。当然，通过链式写法可以实现更多步骤的组合。

#### 22.3.4 练习：格式化打印信息

**题目**

下面的字符串数组当中存有多条信息，请按照格式“ 姓名：XX。性别：XX。 ”的格式将信息打印出来。要求将打印姓名的动作作为第一个 Consumer 接口的Lambda实例，将打印性别的动作作为第二个 Consumer 接口Lambda实例，将两个 Consumer 接口按照顺序“拼接”到一起。

**解答**

```java
import java.util.function.Consumer;
public class DemoConsumer {
    public static void main(String[] args) {
        String[] array = { "迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男" };
        printInfo(s ‐> System.out.print("姓名：" + s.split(",")[0]),
                  s ‐> System.out.println("。性别：" + s.split(",")[1] + "。"),
                  array);
    } 
    private static void printInfo(Consumer<String> one, Consumer<String> two, String[] array) {
        for (String info : array) {
            one.andThen(two).accept(info); // 姓名：迪丽热巴。性别：女。
        }
    }
}
```

#### 22.3.5 Predicate接口

有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用`java.util.function.Predicate<T>` 接口。

**抽象方法：test**

Predicate 接口中包含一个抽象方法： `boolean test(T t)` 。用于条件判断的场景：

```java
import java.util.function.Predicate;
public class Demo15PredicateTest {
    private static void method(Predicate<String> predicate) {
        boolean veryLong = predicate.test("HelloWorld");
        System.out.println("字符串很长吗：" + veryLong);
    } 
    public static void main(String[] args) {
        method(s ‐> s.length() > 5);
    }
}
```

条件判断的标准是传入的Lambda表达式逻辑，只要字符串长度大于5则认为很长。

**默认方法：and**

既然是条件判断，就会存在与、或、非三种常见的逻辑关系。其中将两个 Predicate 条件使用“与”逻辑连接起来实现“并且”的效果时，可以使用default方法 and 。其JDK源码为：

```java
default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) ‐> test(t) && other.test(t);
}
```

如果要判断一个字符串既要包含大写“H”，又要包含大写“W”，那么：

```java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
    private static void method(Predicate<String> one, Predicate<String> two) {
        boolean isValid = one.and(two).test("Helloworld");
        System.out.println("字符串符合要求吗：" + isValid);
    } 
    public static void main(String[] args) {
        method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```

**默认方法：or**

与 and 的“与”类似，默认方法 or 实现逻辑关系中的“或”。JDK源码为：

```java
default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) ‐> test(t) || other.test(t);
}
```

如果希望实现逻辑“字符串包含大写H或者包含大写W”，那么代码只需要将“and”修改为“or”名称即可，其他都不变：

```java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
    private static void method(Predicate<String> one, Predicate<String> two) {
        boolean isValid = one.or(two).test("Helloworld");
        System.out.println("字符串符合要求吗：" + isValid);
    } 
    public static void main(String[] args) {
        method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```

**默认方法：negate**

“与”、“或”已经了解了，剩下的“非”（取反）也会简单。默认方法 negate 的JDK源代码为：

```java
default Predicate<T> negate() {
    return (t) ‐> !test(t);
}
```

从实现中很容易看出，它是执行了test方法之后，对结果boolean值进行“!”取反而已。一定要在 test 方法调用之前调用 negate 方法，正如 and 和 or 方法一样：

```java
import java.util.function.Predicate;
public class Demo17PredicateNegate {
    private static void method(Predicate<String> predicate) {
        boolean veryLong = predicate.negate().test("HelloWorld");
        System.out.println("字符串很长吗：" + veryLong);
    } 
    public static void main(String[] args) {
        method(s ‐> s.length() < 5);
    }
}
```

#### 22.3.6 练习：集合信息筛选

**题目**

数组当中有多条“姓名+性别”的信息如下，请通过 Predicate 接口的拼装将符合要求的字符串筛选到集合ArrayList 中，需要同时满足两个条件：

1. 必须为女生；
2. 姓名为4个字。

```java
public class DemoPredicate {
    public static void main(String[] args) {
        String[] array = { "迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男", "赵丽颖,女" };
    }
}
```

解答

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;
public class DemoPredicate {
    public static void main(String[] args) {
        String[] array = { "迪丽热巴,女", "古力娜扎,女", "马尔扎哈,男", "赵丽颖,女" };
        List<String> list = filter(array,
                                   s ‐> "女".equals(s.split(",")[1]),
                                   s ‐> s.split(",")[0].length() == 4);
        System.out.println(list);
    } 
    private static List<String> filter(String[] array, Predicate<String> one,
                                       Predicate<String> two) {
        List<String> list = new ArrayList<>();
        for (String info : array) {
            if (one.and(two).test(info)) {
                list.add(info);
            }
        } 
        return list;
    }
}
```

#### 22.3.7 Function接口

`java.util.function.Function<T,R>` 接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。

**抽象方法：apply**

Function 接口中最主要的抽象方法为： `R apply(T t)` ，根据类型T的参数获取类型R的结果。

使用的场景例如：将 String 类型转换为 Integer 类型。

```java
import java.util.function.Function;
public class Demo11FunctionApply {
    private static void method(Function<String, Integer> function) {
        int num = function.apply("10");
        System.out.println(num + 20);
    } 
    public static void main(String[] args) {
        method(s ‐> Integer.parseInt(s));
    }
}
```

当然，最好是通过方法引用的写法。

**默认方法：andThen**

Function 接口中有一个默认的 `andThen` 方法，用来进行组合操作。JDK源代码如：

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) ‐> after.apply(apply(t));
}
```

该方法同样用于“先做什么，再做什么”的场景，和 Consumer 中的 andThen 差不多：

```java
import java.util.function.Function;
public class Demo12FunctionAndThen {
    private static void method(Function<String, Integer> one, Function<Integer, Integer> two) {
        int num = one.andThen(two).apply("10");
        System.out.println(num + 20);  // 220
    } 
    public static void main(String[] args) {
        method(str‐>Integer.parseInt(str)+10, i ‐> i *= 10);
    }
}
```

第一个操作是将字符串解析成为int数字，第二个操作是乘以10。两个操作通过 andThen 按照前后顺序组合到了一起。

请注意，Function的前置条件泛型和后置条件泛型可以相同。

#### 22.3.8 练习：自定义函数模型拼接

**题目**

请使用 Function 进行函数模型的拼接，按照顺序需要执行的多个函数操作为：`String str = "赵丽颖,20"`;


1. 将字符串截取数字年龄部分，得到字符串；
2. 将上一步的字符串转换成为int类型的数字；
3. 将上一步的int数字累加100，得到结果int数字。

**解答**

```java
import java.util.function.Function;
public class DemoFunction {
    public static void main(String[] args) {
        String str = "赵丽颖,20";
        int age = getAgeNum(str, s ‐> s.split(",")[1],
                            s ‐>Integer.parseInt(s),
                            n ‐> n += 100);
        System.out.println(age);
    } 
    private static int getAgeNum(String str, Function<String, String> one,
                                 Function<String, Integer> two,
                                 Function<Integer, Integer> three) {
        return one.andThen(two).andThen(three).apply(str);
    }
}
```
