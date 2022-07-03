# Java：并发笔记-02

> 说明：观看了 bilibili 上 **[黑马程序员](https://space.bilibili.com/37974444/)** 的课程 [java并发编程](https://www.bilibili.com/video/BV16J411h7Rd) 后所作的学习笔记，总计拆分为9篇笔记，此为2/9。

## 3. 共享模型之管程-1

### 本章内容-1

- 共享问题
- synchronized
- 线程安全分析

### 3.1 共享带来的问题

#### 小故事

- 老王（操作系统）有一个功能强大的算盘（CPU），现在想把它租出去，赚一点外快

  ![内存共享-小故事1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545614.PNG)

- 小南、小女（线程）来使用这个算盘来进行一些计算，并按照时间给老王支付费用

- 但小南不能一天24小时使用算盘，他经常要小憩一会（sleep），又或是去吃饭上厕所（阻塞IO操作），有时还需要一根烟，没烟时思路全无（wait）这些情况统称为（阻塞）

  ![内存共享-小故事2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545032.PNG)

- 在这些时候，算盘没利用起来（不能收钱了），老王觉得有点不划算

- 另外，小女也想用用算盘，如果总是小南占着算盘，让小女觉得不公平

- 于是，老王灵机一动，想了个办法 [ 让他们每人用一会，轮流使用算盘 ]

- 这样，当小南阻塞的时候，算盘可以分给小女使用，不会浪费，反之亦然

- 最近执行的计算比较复杂，需要存储一些中间结果，而学生们的脑容量（工作内存）不够，所以老王申请了一个笔记本（主存），把一些中间结果先记在本上

- 计算流程是这样的

  ![内存共享-小故事3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545988.PNG)

- 但是由于分时系统，有一天还是发生了事故

- 小南刚读取了初始值 0 做了个 +1 运算，还没来得及写回结果

- 老王说 [ 小南，你的时间到了，该别人了，记住结果走吧 ]，于是小南念叨着 [ 结果是1，结果是1...] 不甘心地到一边待着去了（上下文切换）

- 老王说 [ 小女，该你了 ]，小女看到了笔记本上还写着 0 做了一个 -1 运算，将结果 -1 写入笔记本

- 这时小女的时间也用完了，老王又叫醒了小南：[小南，把你上次的题目算完吧]，小南将他脑海中的结果 1 写入了笔记本

  ![内存共享-小故事4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545301.PNG)

- 小南和小女都觉得自己没做错，但笔记本里的结果是 1 而不是 0

#### Java 的体现

两个线程对初始值为 0 的静态变量一个做自增，一个做自减，各做 5000 次，结果是 0 吗？

```java
static int counter = 0;

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            counter++;
        }
    }, "t1");
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            counter--;
        }
    }, "t2");
    t1.start();
    t2.start();
    t1.join();
    t2.join();
    LoggerUtils.LOGGER.debug("{}",counter);
}
```

#### **问题分析**

以上的结果可能是**正数、负数、零**。为什么呢？因为 Java 中对静态变量的自增，自减并不是原子操作，要彻底理解，必须从字节码来进行分析

例如对于 i++ 而言（i 为静态变量），实际会产生如下的 JVM 字节码指令：

```java
getstatic i 	// 获取静态变量i的值
iconst_1 		// 准备常量1
iadd 			// 自增
putstatic i 	// 将修改后的值存入静态变量i
```

而对应 i-- 也是类似：

```java
getstatic i 	// 获取静态变量i的值
iconst_1 		// 准备常量1
isub 			// 自减
putstatic i 	// 将修改后的值存入静态变量i
```

而 Java 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换：

![内存交换](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545721.PNG)

如果是单线程以上 8 行代码是顺序执行（不会交错）没有问题：

![单线程执行](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545321.PNG)

但多线程下这 8 行代码可能交错运行：

1. 出现负数的情况：

![多线程出现负数](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545729.PNG)

2. 出现正数的情况：

![多线程出现正数数](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545333.PNG)

#### 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的
- 问题出在多个线程访问**共享资源**
  - 多个线程读共享资源其实也没有问题
  - 在多个线程对共享资源读写操作时发生**指令交错**，就会出现问题
- 一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为**临界区**

例如，下面代码中的临界区

```java
static int counter = 0;

static void increment(){
    counter ++;
}
static void decrement(){
    counter --;
}
```

#### **竞态条件 Race Condition**

多个线程在临界区内执行，由于代码的**执行序列不同**而导致结果无法预测，称之为发生了**竞态条件**

### 3.2 synchronized 解决方案

#### 互斥

为了避免临界区的竞态条件发生，有多种手段可以达到目的。

- 阻塞式的解决方案：synchronized，Lock
- 非阻塞式的解决方案：原子变量

本次课使用阻塞式的解决方案：synchronized，来解决上述问题，即俗称的【对象锁】，它采用互斥的方式让同一时刻至多只有一个线程能持有【对象锁】，其它线程再想获取这个【对象锁】时就会阻塞住。这样就能保证拥有锁的线程可以安全的执行临界区内的代码，不用担心线程上下文切换.

> 注意：
>
> 虽然 Java 中互斥和同步都可以采用 synchronized 关键字来完成，但它们还是有区别的：
>
> - **互斥**是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区代码
> - **同步**是由于线程执行的先后、顺序不同、需要一个线程等待其它线程运行到某个点

#### synchronized

语法：

```java
synchronized(对象){  // 线程1，线程2（blocked）
    // 临界区
}
```

解决：

```java
static int counter = 0;
static final Object room = new Object();

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (room) {
                counter++;
            }
        }
    }, "t1");

    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (room) {
                counter--;
            }
        }
    }, "t2");

    t1.start();
    t2.start();
    t1.join();
    t2.join();
    LoggerUtils.LOGGER.debug("{}",counter);
}
```

你可以做这样的类比：

![synchronize类比](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545461.PNG)

- synchronized(对象) 中的对象，可以想象为一个房间（room），有唯一入口（门）房间只能一次进入一人进行计算，线程 t1，t2 想象成两个人
- 当线程 t1 执行到 synchronized(room) 时就好比 t1 进入了这个房间，并锁住了门拿走了钥匙，在门内执行count++ 代码
- 这时候如果 t2 也运行到了 synchronized(room) 时，它发现门被锁住了，只能在门外等待，发生了上下文切换，阻塞住了
- 这中间即使 t1 的 cpu 时间片不幸用完，被踢出了门外（不要错误理解为锁住了对象就能一直执行下去哦），这时门还是锁住的，t1 仍拿着钥匙，t2 线程还在阻塞状态进不来，只有下次轮到 t1 自己再次获得时间片时才能开门进入
- 当 t1 执行完 synchronized{} 块内的代码，这时候才会从 obj 房间出来并解开门上的锁，唤醒 t2 线程把钥匙给他。t2 线程这时才可以进入 obj 房间，锁住了门拿上钥匙，执行它的 count-- 代码

用图来表示：

![synchronize执行图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545767.PNG)

#### 思考：

synchronized 实际是用**对象锁**保证了**临界区内代码的原子性**，临界区内的代码对外是不可分割的，不会被线程切换所打断。

为了加深理解，请思考下面的问题

- 如果把 synchronized(obj) 放在 for 循环的外面，如何理解？答：锁住的代码更多了，体现了原子性
- 如果 t1 synchronized(obj1) 而 t2 synchronized(obj2) 会怎样运作？答：锁对象不同，并不能达到互斥效果，体现了锁对象
- 如果 t1 synchronized(obj) 而 t2 没有加会怎么样？如何理解？答：并不能达到互斥效果，t2 不需要遵守锁的规则，体现了锁对象锁对象

#### 面向对象改进

把需要保护的共享变量放入一个类

```java
public class Test {

    public static void main(String[] args) throws InterruptedException {
        Room room = new Room();

        Thread t1 = new Thread(()->{
            for (int i = 0; i < 5000; i++) {
                room.increment();
            }
        }, "t1");

        Thread t2 = new Thread(()->{
            for (int j = 0; j < 5000; j++) {
                room.decrement();
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        LoggerUtils.LOGGER.debug("count:{}", room.get());
    }

}

class Room{
    int value = 0;

    public void increment(){
        synchronized (this){
            value++;
        }
    }

    public void decrement(){
        synchronized (this){
            value--;
        }
    }

    public int get(){
        synchronized (this){
            return value;
        }
    }
}

```

### 3.3 方法上的 synchronized

锁住对象——**对象锁**

```java
class Test{
	public synchronized void test(){
        
    }
}

// 等价于
class Test{
    public void test(){
        synchronized(this){
            
        }
    }
}

```

锁住类对象——**类锁**

```java
class Test{
    public synchronized static void test(){
        
    }
}
// 等价于
class Test{
    public static void test(){
        synchronized(Test.class){
            
        }
    }
}

```

#### 不加 synchronized 的方法

不加 synchronzied 的方法就好比不遵守规则的人，不去老实排队（好比翻窗户进去的）

#### 所谓的“线程八锁”

其实就是考察 synchronized 锁住的是哪个对象

**情况1**：先输出1再输出2  /  先输出2再输出1

```java
public class Test {

    public static void main(String[] args) {
        Number n1 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n1.b()).start();
    }

}

class Number{
    public synchronized void a(){
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}

```

**情况2**：先过1s然后输出1再输出2 / 先输出2经过1s后再输出1

```java
public class Test {

    public static void main(String[] args) {
        Number n1 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n1.b()).start();
    }
}

class Number{
    public synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}

```

**情况3**：一切皆有可能，就是在输出1之前都需要等待1s

```java
public class Test {

    public static void main(String[] args) {
        Number n1 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n1.b()).start();
        new Thread(()-> n1.c()).start();
    }
}

class Number{
    public synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }

    public void c(){
        LoggerUtils.LOGGER.debug("3");
    }
}
```

**情况4**：锁不同，先输出2，经过1s后输出1

```java
public class Test {

    public static void main(String[] args) {
        Number n1 = new Number();
        Number n2 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n2.b()).start();
    }
}

class Number{
    public synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}
```

**情况5**：锁不同（一个类锁一个对象锁），先输出2，经过1s后输出1

```java
public class Test {

    public static void main(String[] args) {
        Number n1 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n1.b()).start();
    }
}

class Number{
    public static synchronized void a(){  // 类锁
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){  // 对象锁
        LoggerUtils.LOGGER.debug("2");
    }
}
```

**情况6**：先过1s输出1，然后再输出2 / 先输出2，过1s后再输出1

```java
public class Test04 {

    public static void main(String[] args) {
        Number n1 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n1.b()).start();
    }
}

class Number{
    public static synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public static synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}
```

**情况7**：先输出2，经过1s后输出1

```java
public class Test04 {

    public static void main(String[] args) {
        Number n1 = new Number();
        Number n2 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n2.b()).start();
    }
}

class Number{
    public static synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}

```

**情况8**：先过1s输出1，然后再输出2 / 先输出2，过1s后再输出1

```java
public class Test04 {

    public static void main(String[] args) {
        Number n1 = new Number();
        Number n2 = new Number();
        LoggerUtils.LOGGER.debug("main start");
        new Thread(()-> n1.a()).start();
        new Thread(()-> n2.b()).start();
    }
}

class Number{
    public static synchronized void a(){
        Sleeper.sleep(1);
        LoggerUtils.LOGGER.debug("1");
    }

    public static synchronized void b(){
        LoggerUtils.LOGGER.debug("2");
    }
}

```

### 3.4 变量的线程安全分析

#### 成员变量和静态变量是否线程安全？

- 如果它们没有共享，则线程安全
- 如果它们被共享了，根据它们的状态是否能够改变，又分两种情况
  - 如果只有读操作，则线程安全
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全

#### 局部变量是否线程安全？

- 局部变量是线程安全的
- 但局部变量引用的对象则未必
  - 如果该对象没有逃离方法的作用范围，它是线程安全的
  - 如果该对象逃离方法的作用范围，需要考虑线程安全

#### 局部变量线程安全分析

```java
public static void test1(){
    int i = 10;
    i++;
}
```

每个线程调用 `test1()` 方法时局部变量 i，会在每个线程的栈帧内存中被创建多份，因此不存在共享

```java
public static void test1();
	descriptor: ()V
	flags: ACC_PUBLIC, ACC_STATIC
    Code:
    	stack=1, locals=1, args_size=0
            0: bipush 10
            2: istore_0
            3: iinc 0, 1
            6: return
        LineNumberTable:
            line 10: 0
            line 11: 3
            line 12: 6
    	LocalVariableTable:
    	Start Length Slot Name Signature
    		3      4    0    i         I
```

如图：

![局部变量线程安全](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545618.PNG)

局部变量的引用稍有不同

先看一个成员变量的例子

```java
public class Test {

    static final int THREAD_NUMBER=2;
    static final int LOOP_NUMBER=200;

    public static void main(String[] args) {
        ThreadUnsafe threadUnsafe = new ThreadUnsafe();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(()->{
                threadUnsafe.method1(LOOP_NUMBER);
            }, "Thread"+i).start();
        }
    }
}

class ThreadUnsafe{
    ArrayList<String> list = new ArrayList<>();

    public void method1(int loopNumber){
        for (int i = 0; i < loopNumber; i++) {
            method2(list);
            method3(list);
        }
    }

    private void method2(ArrayList<String> list){
        list.add("1");
    }

    private void method3(ArrayList<String> list){
        list.remove(0);
    }
}

```

其中一种情况是，如果线程2 还未 add，线程1 remove 就会报错：

```
Exception in thread "Thread1" Exception in thread "Thread0" java.lang.ArrayIndexOutOfBoundsException: -1
	at java.util.ArrayList.add(ArrayList.java:463)
	at cn.xyc.n4.ThreadUnsafe.method2(Test05.java:31)
	at cn.xyc.n4.ThreadUnsafe.method1(Test05.java:25)
	at cn.xyc.n4.Test05.lambda$main$0(Test05.java:14)
	at java.lang.Thread.run(Thread.java:748)
```

分析：

- 无论哪个线程中的 method2 引用的都是同一个对象中的 list 成员变量
- method3 与 method2 分析相同
- ![线程引用同一个成员变量](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545724.PNG)

将 list 修改为局部变量

```java
class Threadsafe{

    public void method1(int loopNumber){
        // 将list修改为局部变量
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < loopNumber; i++) {
            method2(list);
            method3(list);
        }
    }

    private void method2(ArrayList<String> list){
        list.add("1");
    }

    private void method3(ArrayList<String> list){
        list.remove(0);
    }
}
```

那么就不会有上述问题了

分析：

- list 是局部变量，每个线程调用时会创建其不同实例，没有共享
- 而 method2 的参数是从 method1 中传递过来的，与 method1 中引用同一个对象
- method3 的参数分析与 method2 相同
- ![线程引用局部成员变量](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545234.PNG)

方法访问修饰符带来的思考，如果把 method2 和 method3 的方法修改为 public 会不会代理线程安全问题？

- 情况1：有其它线程调用 method2 和 method3——无问题；

- 情况2：在 情况1 的基础上，为 ThreadSafe 类添加子类，子类覆盖 method2 或 method3 方法，即

  ```java
  public class Test05 {
  
      static final int THREAD_NUMBER=2;
      static final int LOOP_NUMBER=200;
  
      public static void main(String[] args) {
          ThreadSafe threadUnsafe = new ThreadSafeSubClass();
          for (int i = 0; i < THREAD_NUMBER; i++) {
              new Thread(()->{
                  threadUnsafe.method1(LOOP_NUMBER);
              }, "Thread"+i).start();
          }
      }
  }
  
  class ThreadSafe{
  
      public void method1(int loopNumber){
          ArrayList<String> list = new ArrayList<>();
          for (int i = 0; i < loopNumber; i++) {
              method2(list);
              method3(list);
          }
      }
  
      public void method2(ArrayList<String> list){
          list.add("1");
      }
  
      public void method3(ArrayList<String> list){
          list.remove(0);
      }
  }
  
  class ThreadSafeSubClass extends ThreadUnsafe{
  
      @Override
      public void method3(ArrayList<String> list) {
          // 子类和父类共享资源了
          // 局部变量的引用被暴露了
          new Thread(()->{
              list.remove(0);
          }).start();
      }
  }
  ```

  可能输出：

  ```
  Exception in thread "Thread-172" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
  	at java.util.ArrayList.rangeCheck(ArrayList.java:657)
  	at java.util.ArrayList.remove(ArrayList.java:496)
  	at cn.xyc.n4.ThreadUnsafeSubClass.lambda$method3$0(Test05.java:45)
  	at java.lang.Thread.run(Thread.java:748)
  ```

解决：

- 将方法的访问修饰符修改为private，限制子类无法覆盖；
- 加final修饰符，阻止子类重新该方法

从这个例子可以看出 private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】

#### 常见线程安全类

- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable
- java.util.concurrent 包下的类

这里说它们是线程安全的是指，多个线程调用它们同一个实例的某个方法时，是线程安全的。也可以理解为：

```java
Hashtable table = new Hashtable();

new Thread(()->{
	table.put("key", "value1");
}).start();

new Thread(()->{
	table.put("key", "value2");
}).start();

// hashtable源码：
public synchronized V put(K key, V value){
    ...
}
```

它们的每个方法是原子的

但注意它们多个方法的组合不是原子的，见后面分析

#### 线程安全类方法的组合

分析下面代码是否线程安全？

```java
Hashtable table = new Hashtable();
// 线程1，线程2
if(table.get("key") == null){
    table.put("key", value);
}
```

![线程安全类组合不安全](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251545299.PNG)

#### 不可变类线程安全性

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的；

有同学或许有疑问，String 有 replace，substring 等方法【可以】改变值啊，那么这些方法又是如何保证线程安全的呢？

```java
public class Immutable{
    private int value = 0;
    
    public Immutable(int value){
        this.value = value;
    }
    
    public int getValue(){
        return this.value;
    }
}
```

如果想增加一个增加的方法呢？ 

```java
public class Immutable{
    private int value = 0;
    
    public Immutable(int value){
        this.value = value;
    }
    
    public int getValue(){
        return this.value;
    }
    
    public Immutable add(int v){
        // 新创建一个对象给你
        return new Immutable(this.value + v);
    }
}
```

#### 实例分析

* 例1：

```java
// Sevlet运行在tomcat环境下，只有一个实例，会被多个线程共享使用
public class MyServlet extends HttpServlet {
    // 是否安全？ ——> 不安全，HashMap不是线程安全类
	Map<String,Object> map = new HashMap<>();
    // 是否安全？ ——> 安全，不可变类
	String S1 = "...";
    // 是否安全？ ——> 安全
    final String S2 = "...";
    // 是否安全？ ——> 不安全，不是线程安全类
    Date D1 = new Date();
    // 是否安全？ ——> 不安全，final引用值固定不可变，但是对象中的属性还是可以被修改
    final Date D2 = new Date();
    
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        // 使用上述变量
    }
}
```

* 例2：

```java
public class MyServlet extends HttpServlet {
    // 是否安全？ ——> 不安全，HttpServlet只有一份，因此userService也会被多个线程共享
    private UserService userService = new UserServiceImpl();
    
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
    	userService.update(...);
    }
}

public class UserServiceImpl implements UserService {
    // 记录调用次数
    private int count = 0;
    public void update() {
        // ... 属于临界区，count就成了共享资源
        count++;
    }
}
```

* 例3： 

```java
// Spring AOP 中对象没有特殊说明都是单例的，多个线程使用，成员变量被共享
@Aspect
@Component
public class MyAspect {
    // 是否安全？ ——> 不安全，成员变量被共享了
    private long start = 0L;
    
    @Before("execution(* *(..))")  // 前置通知
    public void before() {
        start = System.nanoTime();
    }
    
    @After("execution(* *(..))")  // 后置通知
    public void after() {
        long end = System.nanoTime();
        System.out.println("cost time:" + (end-start));
    }
}
```

* 例4： 

```java
public class MyServlet extends HttpServlet {
    // 是否安全？ ——> 安全，UserServiceImpl为线程安全
    private UserService userService = new UserServiceImpl();
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        userService.update(...);
    }
}

public class UserServiceImpl implements UserService {
    // 是否安全 ——> 安全，Dao中无可更改的成员变量，Dao为线程安全
    private UserDao userDao = new UserDaoImpl();
    public void update() {
        userDao.update();
    }
}

public class UserDaoImpl implements UserDao {
    // 是否安全？ ——> 安全，没有成员变量
    public void update() {
        String sql = "update user set password = ? where username = ?";
        // 是否安全 ——> 安全，局部变量
        try (Connection conn = DriverManager.getConnection("","","")){
            // ...
        } catch (Exception e) {
            // ...
        }
    }
}
```

* 例5：

```java
public class MyServlet extends HttpServlet {
    // 是否安全？ --> 不安全，UserServiceImpl不安全
    private UserService userService = new UserServiceImpl();
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        userService.update(...);
    }
}

public class UserServiceImpl implements UserService {
    // 是否安全？ --> 不安全，UserDaoImpl类不安全
    private UserDao userDao = new UserDaoImpl();
    public void update() {
        userDao.update();
    }
}

public class UserDaoImpl implements UserDao {
    // 是否安全？  ——> 不安全，将conn作为类中的成员变量，
    // 由于HttpServlet/UserServiceImpl/UserDaoImpl都是单例
    // 因此，conn会被多个线程共享
    private Connection conn = null;
    public void update() throws SQLException {
        String sql = "update user set password = ? where username = ?";
        conn = DriverManager.getConnection("","","");
        // ...
        conn.close();
    }
}
```

* 例6：

```java
public class MyServlet extends HttpServlet {
    // 是否安全？ --> 安全，UserServiceImpl是安全的
    private UserService userService = new UserServiceImpl();
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        userService.update(...);
    }
}
public class UserServiceImpl implements UserService {
    public void update() {
        // 每次创建新的UserDaoImpl，因此线程安全，但是不推荐
        UserDao userDao = new UserDaoImpl();
        userDao.update();
    }
}
public class UserDaoImpl implements UserDao {
    // 是否安全？ --> 不安全
    private Connection = null;
    public void update() throws SQLException {
        String sql = "update user set password = ? where username = ?";
        conn = DriverManager.getConnection("","","");
        // ...
        conn.close();
    }
}
```

* 例7：

```java
public abstract class Test {
    
    public void bar() {
        // 是否安全 --> 不安全
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        foo(sdf);  // 暴露给其他对象了
    }
    
    public abstract foo(SimpleDateFormat sdf);  // 抽象方法
    
    public static void main(String[] args) {
        new Test().bar();
    }
}
```

其中 foo 的行为是不确定的，可能导致不安全的发生，被称之为**外星方法** 

```java
public void foo(SimpleDateFormat sdf) {
    String dateStr = "1999-10-11 00:00:00";
    for (int i = 0; i < 20; i++) {
        new Thread(() -> {
            try {
                sdf.parse(dateStr);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

请比较 JDK 中 String 类的实现 

* 例8： 线程不安全，**个人理解如下**：Integer在后续修改值后的其表示的对象变了，那么加锁加在了不同的对象上，导致线程不安全

```java
private static Integer i = 0;
public static void main(String[] args) throws InterruptedException {
    List<Thread> list = new ArrayList<>();
    for (int j = 0; j < 2; j++) {
        Thread thread = new Thread(() -> {
            for (int k = 0; k < 5000; k++) {
                synchronized (i) {
                    i++;
                }
            }
        }, "" + j);
        list.add(thread);
    }
    list.stream().forEach(t -> t.start());
    list.stream().forEach(t -> {
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    log.debug("{}", i);
}
```

### 3.5 习题

#### 卖票练习

测试下面代码是否存在线程安全问题，并尝试改正

```java
public class ExerciseSell {

    public static void main(String[] args) {

        TicketWindow ticketWindow = new TicketWindow(1000);
        // 所有线程的集合
        List<Thread> list = new ArrayList<>();
        // 用来存储卖出去多少张票
        List<Integer> sellCount = new Vector<>();

        // 模拟1000人去抢票
        for (int i = 0; i < 1000; i++) {
            Thread t = new Thread(()->{
                // 分析这里的竞态条件
                Sleeper.sleep(0.01);
                int count = ticketWindow.sell(randomAmount());
                sellCount.add(count);
            });
            list.add(t);
            t.start();
        }

        // 等待线程结束
        list.forEach((t)->{
            try {
                t.join();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        });

        // 买出去的票求和
        LoggerUtils.LOGGER.debug("selled count:{}", sellCount.stream().mapToInt(c -> c).sum());
        // 剩余票数
        LoggerUtils.LOGGER.debug("remainder count:{}", ticketWindow.getCount());
    }

    // Random 为线程安全
    static Random random = new Random();
    // 随机1~5
    public static int randomAmount(){
        return random.nextInt(5) + 1;
    }
}

class TicketWindow{
    private int count;

    public TicketWindow(int count) {
        this.count = count;
    }

    public int getCount() {
        return count;
    }

    public int sell(int amount) {
        if (this.count >= amount) {
            this.count -= amount;
            return amount;
        } else {
            return 0;
        }
    }
}
```

上述代码会出现线程安全问题：

```java
// 出现票多买的情况
15:07:12.211 cn.util.LoggerUtils [main] - selled count:1006
15:07:12.214 cn.util.LoggerUtils [main] - remainder count:0
```

另外，用下面的代码行不行，为什么？ 

```java
List<Integer> sellCount = new ArrayList<>();
// 会有概率出错：
// Exception in thread "main" java.lang.NullPointerException
// 但是为什么呢？？？
```

修改如下：

```java
public synchronized int sell(int amount) {
    if (this.count >= amount) {
        this.count -= amount;
        return amount;
    } else {
        return 0;
    }
}
```

#### 转账练习

测试下面代码是否存在线程安全问题，并尝试改正

```java
public class ExerciseTransfer {

    public static void main(String[] args) throws InterruptedException {
        Account a = new Account(1000);
        Account b = new Account(1000);

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                a.transfer(b, randomAmount());
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                b.transfer(a, randomAmount());
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        // 查看转账2000次后的总金额
        LoggerUtils.LOGGER.debug("total:{}",(a.getMoney() + b.getMoney()));
    }

    // Random 为线程安全
    static Random random = new Random();
    // 随机 1~100
    public static int randomAmount() {
        return random.nextInt(100) +1;
    }
}

class Account{

    private int money;

    public Account(int money){
        this.money = money;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public void transfer(Account target, int account){
        if(this.money >= account){
            this.setMoney(this.getMoney() - account);
            target.setMoney(target.getMoney() + account);
        }
    }
}
```

这样改正行不行，为什么？ 

```java
public synchronized void transfer(Account target, int account){
    if(this.money >= account){
        this.setMoney(this.getMoney() - account);
        target.setMoney(target.getMoney() + account);
    }
}
```

上述修改不行，因为转账涉及到了两个对象，在方法上加 synchronized 只是对于当前调用该方法的对象加了锁，应做如下修改：

```java
public void transfer(Account target, int account){
    synchronized (Account.class){
        if(this.money >= account){
            this.setMoney(this.getMoney() - account);
            target.setMoney(target.getMoney() + account);
        }
    }
}
```

