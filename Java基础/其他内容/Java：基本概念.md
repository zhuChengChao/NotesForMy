# Java：基本概念

> 一些基本 Java 概念，做一个小小小小的记录

## 面向对象&面向过程

面向对象思想就是在计算机程序设计过程中，参照现实中事物，将事物的属性特征、行为特征抽象出来，描述成计算机事件的设计思想。 面向对象的语言中，包含了三大基本特征，即封装、继承和多态。 

> 个人感觉，应该是四大基本特征，应为：抽象、封装、继承、多态

**面向对象和面向过程的区别:** 

1. 编程思路不同：面向过程以实现功能的**函数开发**为主，而面向对象要首先抽象出**类、属性及其方法**，然后通过实例化类、执行方法来完成功能。 
2. 封装性：都具有封装性，但是面向过程是封装的是**功能**，而面向对象封装的是**数据和功能**。
3. 面向对象具有继承性和多态性，而面向过程没有继承性和多态性，所以面向对象优势很明显。

**面向对象的三大特性**

1. **封装：**通常认为封装是把数据和操作数据的方法封装起来，对数据的访问只能通过已定义的接口。

2. **继承：**继承是从已有类得到继承信息创建新类的过程。提供继承信息的类被称为父类（超类/基类），得到继承信息的被称为子类（派生类）。

3. **多态**：分为**编译时多态（方法重载）**和**运行时多态（方法重写）**。要实现多态需要做两件事：**一是子类继承父类并重写父类中的方法**，**二是用父类型引用子类型对象**，这样同样的引用调用同样的方法就会根据子类对象的不同而表现出不同的行为。

   > 关于重写和重载，见[Java：重载和重写]()

**关于继承的几点补充：**

1. 子类拥有父类对象所有的属性和方法（**包括私有属性和私有方法**），但是父类中的私有属性和私有方法子类是无法访问，只是拥有。因为在一个子类被创建的时候，首先会在内存中创建一个父类对象，然后在父类对象外部放上子类独有的属性，两者合起来形成一个子类的对象；
2. 子类可以拥有自己属性和方法；

3. 子类可以用自己的方式实现父类的方法。（重写）

> 参考：https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

## JDK&JRE&JVM

**JDK（Java Development Kit）：**是 Java 开发工具包，是整个 Java 的核心，包括了 Java 运行环境 JRE、Java 工具（例如：解释器-Java、编译器-Javac）和 Java 基础类库。

**JRE（ Java Runtime Environment）：**是 Java 的运行环境，包含 JVM 标准实现及 Java 核心类库。

**JVM（Java Virtual Machine）：**是 Java 虚拟机，是整个 Java 实现跨平台的最核心的部分，能够运行以 Java 语言写的软件程序。所有的 Java 程序会首先被编译为 .class 的类文件，这种类文件可以在虚拟机上执行。

**关系如下：**

**JDK = JRE + 开发工具**

**JRE = JVM + 类库**

![jvm比较](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251533356.PNG)

**总结：**

Java程序开发过程为：

1. 利用 JDK（调用 Java API）编写出 Java 源代码，存储于 .java 文件中；
2. JDK 中的编译器 Javac 将 Java 源代码编译成Java字节码，存储与 .class 文件中；
3. JRE 加载，验证，执行 Java 字节码；
4. JVM 将字节码解析为机器码并映射到 CPU 指令集或者 OS 的系统调用。

> 参考：
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA
>
> https://www.jianshu.com/p/7b99bd132470

## 字节码&JVM

**字节码：**

在 Java 中，将虚拟机可以读懂的代码，称为字节码，即 java 中的 .class 文件；

而机器可以读懂的代码为二进制命令，即**机器码**，即 0/1 组成的文件。

**Java程序运行过程如下：**

`源码(xxx.java)->javac编译器->JVM可执行的字节码(xxx.class)->JVM->机器码->程序执行`

**采用字节码的好处：**

Java 语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以 Java 程序运行时比较高效，而且，由于字节码并不专对一种特定的机器，因此，Java程序无须重新编译便可在多种不同的计算机上运行。

本质上就是采用解释性语言的好处。

> **编译型语言：**
>
> 需要通过编译器，将**源代码编译成机器码之后才能执行的语言**。一般是通过编译和链接两个步骤，编译是将我们的程序编译成机器码，链接是程序和依赖库等串联起来。
>
> 优点：编译器一般会有预编译的过程对代码进行了优化，因为编译只做了一次，运行时不会再编译，所以编译型语言效率高。
>
> 缺点：编译之后如果想要修改某一个功能，就需要整个模块重新编译。编译的时候根据对应的运行环境生成不同的机器码。不同的操作系统之间，可能会有问题。需要根据环境的不同，生成不同的可执行文件。
>
> 代表语言：C、C++、Pascal、Object-C以及最近很火的苹果新语言swift，GO
>
> **解释型语言：**
>
> 解释型语言不需要编译，相比编译型语言省了道工序，**解释型语言在运行程序的时候才逐行进行翻译**。字节码也是解释型的一部分。
>
> 优点：有良好的平台兼容性，只要安装了虚拟机，就可以。容易维护，方便快速部署，不用停机维护。
>
> 缺点：每次运行的时候都要解释一遍，性能上不如编译型语言。
>
> 代表语言：JavaScript、Python、Erlang、PHP、Perl、Ruby

**虚拟机：**

Java中引入了虚拟机（JVM）的概念，就是在机器和程序之间加入了一层抽象的虚拟机器。这台机器在各个平台中都给程序提供了接口。因此，编写 Java 程序只需要面向虚拟机编程。

> 参考：https://blog.csdn.net/a4171175/article/details/90735888

## & 和 &&

Java 中 && 和 & 都是表示与的逻辑运算符，都表示逻辑运输符 and，当两边的表达式都为 true 的时候，整个运算结果才为 true，否则为 false。

`&&`：有短路功能，当第一个表达式的值为 false 的时候，**则不再计算第二个表达式**；

```java
@Test
public void test02(){
    String str = "a";
    int i = 0;
    if(str != "a" && (i++) == 1){
        ;
    }
    System.out.println(i);  // 由于短路缘故，i=0
    if(str!="a" & (i++)==1){
        ;
    }
    System.out.println(i);  // 还是执行i++,故i=1
}
```

`&`：不管第一个表达式结果是否为 true，**第二个都会执行**。

除此之外，& 还可以用作位运算符：当 & 两边的表达式不是 Boolean 类型的时候，& 表示按位操作。

```java
// 位运算
@Test
public void test03(){
    int a = 1;  // 01
    int b = 3;  // 11
    int c = a & b;  // 01&11=01
    System.out.println(c);  // 01
}
```

## 两个二进制数的异或结果

两个二进制数异或结果是这两个二进制数差的绝对值。表达式如下：**a^b = |a-b|**。

两个二进制 a 与 b 异或，即 a 和 b 两个数按位进行运算。如果对应的位相同，则为 0（相当于对应的算术相减），如果不同即为 1（相当于对应的算术相加）。由于二进制每个位只有两种状态，要么是 0，要么是 1，**则按位异或操作可表达为按位相减取绝对值，再按位累加。**

```bash
# 例如3和5进行异或
3:0011  ^  5:0101 ---> 按位相减取绝对值，再按位累加
0-0=0
0-1=-1  ---> 取绝对值为1
1-0=1
1-1=0
---> 按位累加：0110=6
```

## Java环境变量PATH 和 CLASSPATH

**Java环境变量PATH：**

其作用是指定命令搜索路径，例如在运行`javac/java`命令时，会根据 PATH 环境变量指定的路径中查看是否能找到命令对应的程序；

在 windows 平台中，通过如下配置即可：

1. 在环境变量中添加`JAVA_HOME`，它指向jdk的安装目录，Tomcat 等软件就是通过搜索 JAVA_HOME 变量来找到并使用安装好的jdk，如下：

   ```bash
   变量名(N)：JAVA_HOME
   变量值(V)：C:\Software\Java\jdk1.8.0_241(即jdk安装路径)
   ```

2. 编辑环境变量，添加如下即可

   ```bash
   %JAVA_HOME%\bin
   ```

3. 在执行 `java/javac` 等命令时，会根据上述配置的路径找到相应的 exe 程序执行

**CLASSPATH环境变量：**

CLASSPATH环境变量的作用是指定 Java 类所在的目录，即指定类搜索路径；JVM 就是通过 CLASSPTH 来寻找类的；

而目前是不需要进行配置的。

例如：

1. 在某文件夹下创建个.java文件

   ```java
   class HelloWorld{
   	public static void main(String args[]){
   		System.out.println("hello world");
   	}
   }
   ```

2. 通过命令生成字节码文件：`javac HelloWorld.java`，生成的字节码文件在同个文件夹下；

3. 通过`java HelloWorld`运行即可

从上述可知，此时 cmd 所在的路径即为 CLASSPATH 路径，因此可以直接通过命令 `java HelloWorld` 运行 .class 字节码文件；

> 参考：
>
> https://blog.csdn.net/lizhensen/article/details/79262564
>
> https://www.jianshu.com/p/d63b099cf283

## 参数传递

**值传递 Or 引用传递？**

Java 的参数是以**值传递**的形式传入方法中，而不是引用传递；**Java 语言的参数传递只有值传递。**

当传递方法参数类型为基本数据类型（数字以及布尔值）时，一个方法是不可能修改一个基本数据类型的参数。

```java
private static void changeNum(int num){
    num = 2;
}

public static void main(String[] args) {
    int num = 1;
    System.out.println("num在调用方法前:" + num);  // num在调用方法前:1
    changeNum(num);
    System.out.println("num在调用方法后:" + num);  // num在调用方法前:1
}
```

当传递方法参数类型为引用数据类型时，一个方法将修改一个引用数据类型的参数所指向对象的值。即 Java 函数在传递引用数据类型时，也只是拷贝了引用的值罢了，**之所以能修改引用数据是因为它们同时指向了一个对象**，但这仍然是按值调用而不是引用调用。

```java
// 传入的值为指向stringBuffer对象的地址，即str与stringBuffer指向同一块区域
private static void change(StringBuffer str){
    str.append(", world!");
}

public static void main(String[] args) {
    StringBuffer stringBuffer = new StringBuffer("hello");
    System.out.println("在调用方法前:" + stringBuffer);  // hello
    change(stringBuffer);
    System.out.println("在调用方法前:" + stringBuffer);  // hello, world!
}
```

```java
// String为不可变类
private static void change(String str){
    str = str + ", world!";  // 产生了一个新的String对象，原对象并未被修改
}

public static void main(String[] args) {
    String string = "Hello";
    System.out.println("在调用方法前:" + string);  // Hello
    change(string);
    System.out.println("在调用方法前:" + string);  // Hello
}
```

> 参考：https://blog.csdn.net/qq_22408539/article/details/82945990