# Java：异常

> 对 Java 中的 **异常** ，做一个微不足道的小小小小记

## Error 和 Exception 

**相同点：**

Exception 和Error 都是继承了 **Throwable 类**，在 Java 中只有 Throwable 类型的实例才可以被抛出或者捕获，它是异常处理机制的基本类型。

**不同点：**

1. Exception：是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。
2. Exception 又分为 **可检查(checked)异常** 和 **不可检查(unchecked)异常**：
   1. **可检查异常**在源代码里必须显式的进行捕获处理，这是**编译期检查的一部分**。
   2. **不可检查时异常是指运行时异常**，像 NullPointerException、ArrayIndexOutOfBoundsException 之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并**不会在编译期强制要求**。
3. 一般是指与虚拟机相关的问题，如：系统崩溃、虚拟机错误、内存空间不足、方法调用栈溢出等。这类错误将会导致应用程序中断，仅靠程序本身无法恢复和预防；

**类图：**

![异常](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251533653.PNG)

## 运行时异常与受检异常

首先，Java 提供了三种可抛出异常(Throwable Exception)：

* 受检查异常(Checked Exception)
* 运行时异常(Runtime Exception)
* 错误(Error)

受检查异常(Checked Exception) / 编译期异常 / 非运行时异常：编译时异常在你编译的时候已经发现了有可能的错误情况**会强制你添加异常处理**，要么用 try … catch… 捕获，要么用 throws 声明抛出，交给父类处理。

运行时异常(Runtime Exception)：如：空指针异常、指定的类找不到、数组越界、方法传递参数错误、数据类型转换错误。**可以编译通过，但是一运行就停止了，程序不会自己处理；**

## 常见的异常类有哪些

Throwable：

* **Error**
  * IOError；
  * LinkageError；
  * ReflectionError；
  * ThreadDeath；
  * VirtualMachineError；
* **Exception**
  * CloneNotSupportedException：在没有实现`Cloneable` 接口的实例上调用 Object 的 clone 方法
  * DataFormatException：格式转换的异常
  * InterruptedException：当阻塞方法收到中断请求的时候就会抛出InterruptedException异常
  * IOException：当发生某种 I/O 异常时，抛出此异常。此类是失败或中断的 I/O 操作生成的异常的通用类。
  * ReflectiveOperationException：与反射有关的异常
  * **RuntimeException：运行时异常**
    * ArithmeticException：当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例；
    * ClassCastException：当试图将对象强制转换为不是实例的子类时，抛出该异常；
    * ConcurrentModificationException；并发修改异常，一般在多线程中，对集合类进行并发的修改时会抛出该异常；
    * IllegalArgumentException：抛出的异常表明向方法传递了一个不合法或不正确的参数；
    * IndexOutOfBoundsException：指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出；
    * NoSuchElementException：出现这个异常的原因之一是因为线程访问越界，访问到了不存在的元素
    * NullPointerException：当应用程序试图访问空对象时，则抛出该异常；
    * SecurityException：由安全管理器抛出的异常，指示存在安全侵犯；
  * SQLException：提供关于数据库访问错误或其他错误信息的异常

## throw 和 throws 

**throw：**

1. **作用在方法内**，表示抛出具体异常，由方法体内的语句处理。
2. **具体向外抛出的动作**，所以它抛出的是一个异常实体类。若执行了throw一定是抛出了某种异常。

**throws：**

1. **用在方法的声明上**，表示如果抛出异常，则由该方法的调用者来进行异常处理。
2. 主要的声明这个方法会抛出会抛出某种类型的异常，让它的使用者知道捕获异常的类型。
3. 出现异常是一种可能性，但不一定会发生异常。

**实例：**

```java
void testException(int a) throws IOException, ...{  // 声明可能抛出的异常
    try{
        ......
    }catch(Exception1 e){
        throw e;  // 抛出异常
    }catch(Exception2 e){
        System.out.println("出错了！");
    }
    if(a!=b)
        throw new Exception3("自定义异常");
}
```

## 主线程捕获到子线程

由一个问题：**主线程可以捕获到子线程的异常吗？**而来

**线程设计的理念：“线程的问题应该线程自己本身来解决，而不要委托到外部”**

正常情况下，如果不做特殊的处理，在主线程中是不能够捕获到子线程中的异常的，如下：

```java
// 定义异常类：
public class ThreadExceptionRunner implements Runnable{
    @Override
    public void run() {
        throw new RuntimeException("RuntimeException!!!");
    }
}

// 测试：
public class ThreadExceptionDemo {
    public static void main(String[] args) {
        // 主线程里去捕捉异常
        try {
            Thread thread = new Thread(new ThreadExceptionRunner());
            thread.start();  // 使用线程执行任务
        }catch (Exception e){
            System.out.println("===========");
            e.printStackTrace();
        }
        System.out.println("over");
    }
}
// 输出： **并未输出"==========="，因此未捕获到子线程的异常**
// Exception in thread "Thread-0" java.lang.RuntimeException: RuntimeException!!!
// 	at cn.xyc.ThreadingTest.ThreadExceptionRunner.run(ThreadExceptionRunner.java:8)
// 	at java.lang.Thread.run(Thread.java:748)
// over
```

**想要在主线程中捕获子线程的异常：(没有深刻体会...)**

**方式一：**使用 ExecutorService

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadExceptionDemo {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool(new HandleThreadFactory());
        exec.execute(new ThreadExceptionRunner());
        exec.shutdown();
    }
}

import java.util.concurrent.ThreadFactory;
public class HandleThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        System.out.println("create thread t");
        Thread t = new Thread(r);
        System.out.println("set uncaughtException for t");
        t.setUncaughtExceptionHandler(new MyUncaughtExceptionHandle());
        return t;
    }
}

public class MyUncaughtExceptionHandle implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("catch:" + e);
    }
}

public class ThreadExceptionRunner implements Runnable{
    @Override
    public void run() {
        throw new RuntimeException("RuntimeException!!!");
    }
}
```

**方式二：**使用 Thread 的静态方法。

`Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandle());`

```java
public class ThreadExceptionDemo {
    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandle());
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new ThreadExceptionRunner());
        exec.shutdown();
    }
}
```

## finally 相关

### finally 执行时刻

**finally 块中的代码什么时候被执行？**

在 Java 语言的异常处理中，finally 块的作用就是为了保证无论出现什么情况，finally 块里的代码一定会被执行。

由于程序执行 return 就意味着结束对当前函数的调用并跳出这个函数体，因此任何语句要执行都只能在 return 前执行（除非碰到 exit 函数），**因此 finally 块里的代码也是在 return 之前执行的**。

此外，如果 try-finally 或者 catch-finally 中都有 return，**那么 finally 块中的 return 将会覆盖别处的 return 语句**，最终返回到调用者那里的是 finally 中 return 的值。

> **从字节码角度对其进行说明：**
>
> 先看下finally在字节码中的体现：
>
> * java代码：
>
> ```java
> public class Demo3_11_4 {
>   public static void main(String[] args) {
>         int i = 0;
>         try {
>             i = 10;
>         } catch (Exception e) {
>             i = 20;
>         } finally {
>             i = 30;
>         }
>   }
> }
> ```
>
> * 字节码：
>
> ```java
> Code:
> stack=1, locals=4, args_size=1
> 0: iconst_0
> 1: istore_1          // 0 -> i
> 2: bipush        10  // try --------------------------------------
> 4: istore_1          // 10 -> i                                  |
> 5: bipush        30  // finally                                  |
> 7: istore_1          // 30 -> i                                  |
> 8: goto          27  // return -----------------------------------
> 11: astore_2          // catch Exceptin -> e ----------------------
> 12: bipush        20  //                                          |
> 14: istore_1          // 20 -> i                                  |
> 15: bipush        30  // finally                                  |
> 17: istore_1		  // 30 -> i                                  |
> 18: goto          27  // return -----------------------------------
> 21: astore_3          // catch any -> slot 3 ----------------------
> 22: bipush        30  // finally                                  |
> 24: istore_1          // 30 -> i                                  |
> 25: aload_3           // <- slot 3  有这个slot3                    |
> 26: athrow            // throw 抛出---------------------------------
> 27: return
> Exception table:
> from    to  target type
>    2     5    11   Class java/lang/Exception
>    2     5    21   any  // 剩余的异常类型，比如 Error，监测[2, 5)中的其他异常/error
>   11    15    21   any  // 剩余的异常类型，比如 Error，监测[11, 15)->catch块中的其他异常/error
> LocalVariableTable:
> Start  Length  Slot  Name   Signature
>  12       3     2     e   Ljava/lang/Exception;
>   0      28     0  args   [Ljava/lang/String;
>   2      26     1     i   I
> ```
>
> 可以看到 finally 中的代码被复制了 3 份，分别放入 try 流程，catch 流程以及 catch 剩余的异常类型流程 
>
> **而当finally代码块中出现了return**
>
> * java代码
>
> ```java
> public class Demo3_12_1 {
>   public static void main(String[] args) {
>         int result = test();
>         System.out.println(result);  // 20
>   }
> 
>   public static int test() {
>         try {
>             return 10;
>         } finally {
>             return 20;
>         }
>   }
> }
> ```
>
> * 字节码：
>
> ```java
> public static int test();
> descriptor: ()I
> flags: ACC_PUBLIC, ACC_STATIC
> Code:
> stack=1, locals=2, args_size=0
>  0: bipush        10  // <- 10 放入栈顶
>  2: istore_0          // 10 -> slot 0 (把10放入slot0中，并从从栈顶移除)
>  3: bipush        20  // < - 20 放入栈顶
>  5: ireturn           // 返回栈顶 int(20)
>  6: astore_1          // catch any -> slot 1
>  7: bipush        20  // <- 20 放入栈顶
>  9: ireturn           // 返回栈顶 int(20)
> Exception table:
>  from    to  target type
>      0     3     6   any
> ```
>
> - 由于 finally 中的 ireturn 被插入了所有可能的流程，因此返回结果肯定以 finally 的为准
>
> - 至于字节码中第 2 行，似乎没啥用，且留个伏笔，看下个例子：`Demo3_12_2`
>
> - 跟上例中的 finally 相比，**发现没有 athrow 了**，这告诉我们：如果在 finally 中出现了 return，会吞掉异常！可以试一下下面的代码
>
>   ```java
>   public class Demo3_12_1 {
>      public static void main(String[] args) {
>          int result = test();
>          System.out.println(result);  // 20
>      }
>     
>      public static int test() {
>          try {
>              int i = 1/0;  // 加不加结果都一样
>              return 10;
>          } finally {
>              return 20;
>          }
>      }
>   }
>   ```
>
>   很危险的操作，异常被吞掉了，即使发生了异常也不知道了
>
> **finally 对返回值影响** 
>
> * java代码
>
> ```java
> public class Demo3_12_2 {
>      public static void main(String[] args) {
>            int result = test();
>            System.out.println(result);  // 10
>      }
> 
>      public static int test() {
>            int i = 10;
>            try {
>                return i;
>            } finally {
>                i = 20;
>            }
>      }
> }
> ```
>
> * 字节码
>
> ```java
> public static int test();
> descriptor: ()I
> flags: ACC_PUBLIC, ACC_STATIC
> Code:
>  stack=1, locals=3, args_size=0
>     0: bipush        10  // <- 10 放入栈顶
>     2: istore_0		  // 10 -> i
>     3: iload_0			  // <- i(10)
>     4: istore_1		  // 10 -> slot1，暂存至slot1，目的是为了固定返回值 ☆
>     5: bipush        20  // <- 20 放入栈顶
>     7: istore_0          // 20 -> i
>     8: iload_1		      // <- slot 1(10) 载入 slot 1 暂存的值 ☆
>     9: ireturn           // 返回栈顶的 int(10)
>    10: astore_2          // 期间出现异常，存储异常到slot2中
>    11: bipush        20  // <- 20 放入栈顶
>    13: istore_0          // 20 -> i
>    14: aload_2           // 异常加载进栈
>    15: athrow            // athrow出异常
>  Exception table:
>     from    to  target type
>         3     5    10   any
> ```
>
> > 参考：
> >
> > https://www.bilibili.com/video/BV1yE411Z7AP?p=126 ~ https://www.bilibili.com/video/BV1yE411Z7AP?p=128

### finally 是否必执行

**finally 是不是一定会被执行到？**

不一定。下面列举执行不到的情况：

* 当程序进入 try 块之前就出现异常时，会直接结束，不会执行 finally 块中的代码；

  ```java
  @Test
  public void test08(){
      int i = 1;
      i = i/0;
      try {
          System.out.println("try");
      }finally {
          System.out.println("finally");
      }
  }
  ```

* 当程序在 try 块中强制退出时也不会去执行 finally 块中的代码，比如在 try 块中执行 exit 方法

  ```java
  public void test08(){
      int i = 1;
      try {
          System.out.println("try");
          System.exit(0);
      }finally {
          System.out.println("finally");
      }
  }
  ```

### try-catch-finally

问：**try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？**

会。程序在执行到 return 时会首先将返回值存储在一个指定的位置（上面对finally的字节码中可以体现），其次去执行 finally 块，最后再返回。

因此，对基本数据类型，在 finally 块中改变 return 的值没有任何影响，直接覆盖掉；

```java
// 下述代码返回的结果为1
public static int test(){
    String str = "1.1";
    int i = 0;
    try {
        System.out.println("try");
        int j = 1/0;
    }catch (Exception e){
        e.printStackTrace();
        System.out.println("catch");
        return ++i;
    }finally {
        System.out.println("finally");
        return i;
    }
}
```

而对引用类型是有影响的，返回的是在 finally 对 前面 return 语句返回对象的修改值

```java
// 下述代码返回的结果为def
public static String test02(){
    String str = "abc";
    try {
        System.out.println("try");
        int j = 1/0;
    }catch (Exception e){
        e.printStackTrace();
        System.out.println("catch");
        return str = "def";
    }finally {
        System.out.println("finally");
        return str;
    }
}
```

问：**try-catch-finally 中那个部分可以省略？**

1. catch 可以省略。**try 只适合处理运行时异常，try+catch 适合处理运行时异常+普通异常**。也就是说，如果你只用 try 去处理普通异常却不加以 catch 处理，编译是通不过的，因为编译器硬性规定，**普通异常如果选择捕获，则必须用 catch 显示声明以便进一步处理**。而运行时异常在编译时没有如此规定，所以 catch 可以省略，你加上catch 编译器也觉得无可厚非。

   ```java
   @Test
   public void test11(){
       try {
           int i = 1/0;
       }finally {
           System.out.println("finally");
           return;
       }
   }
   ```

2. finally也可以省略

   ```java
   @Test
   public void test11(){
       try {
           int i = 1/0;
       }catch (ArithmeticException e){
           e.printStackTrace();
       }
   }
   ```

## 参考

https://www.cnblogs.com/baxianhua/p/9162103.html

https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

https://blog.csdn.net/qq_38080370/article/details/87880511

https://blog.csdn.net/wild46cat/article/details/80808555

https://www.cnblogs.com/ktao/p/8586966.html