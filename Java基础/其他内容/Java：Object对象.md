# Java：Object对象

> 对 Java 中的 **Object** 对象，做一个微不足道的小小小小记

## Object 的常用方法有哪些

- `clone()` 方法：用于创建并返回当前对象的一份拷贝；

  > 在[Java：创建对象小记](https://www.cnblogs.com/zhuchengchao/p/14292290.html)这里做了个小结

- `getClass()` 方法：用于返回当前运行时对象的 Class；

- `toString()` 方法：返回对象的字符串表示形式；

- `finalize()` 方法：实例被垃圾回收器回收时触发的方法；

- `equals()` 方法：用于比较两个对象的内存地址是否相等，一般需要重写；

- `hashCode()` 方法：用于返回对象的哈希值；

- `notify()` 方法：唤醒一个在此对象监视器上等待的线程。如果有多个线程在等待只会唤醒一个。

- `notifyAll()` 方法：作用跟 notify() 一样，只不过会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

- `wait()` 方法：让当前对象等待；

- .......

## finalize & final & finally

**finalize：**Object 类的一个方法，在垃圾回收时会调用被回收对象的 finalize。

**final：**final可以用来修饰类，方法和变量（成员变量或局部变量）

1. 修饰类：当用 final 修饰类的时，表明该类不能被其他类所继承。（final 类中所有的成员方法都会隐式的定义为 final 方法）

2. 修饰方法：把方法锁定，以防止继承类对其进行更改。

3. 修饰变量：final 成员变量表示常量，只能被赋值一次，赋值后其值不再改变。类似于 C++ 中的 const。

   1. 当 final 修饰一个基本数据类型时，表示该基本数据类型的值一旦在初始化后便不能发生变化；

   2. 如果  final 修饰一个引用类型时，则在对其初始化之后便不能再让其指向其他对象了，但该引用所指向的对象的内容是可以发生变化的，这本质上是一回事，因为引用的值是一个地址，final 要求值，即地址的值不发生变化。

      > 进一步说明：
      >
      > 当final 修饰的是一个引用变量，**那么这个引用始终只能指向这个对象，但是这个对象内部的属性是可以变化的**。
      >
      > **官方文档解释：**
      >
      > once a final variable has been assigned, it always contains the same value. If a final variable holds a reference to an object, then the state of the object may be changed by operations on the object, but the variable will always refer to the same object.
      >
      > 一旦一个final修饰的变量被赋值了，它总是包含相同的值。如果一个final变量持有对一个对象的引用，那么该对象的状态可以通过对该对象的操作而改变，但该变量将始终引用同一个对象。

   3. **final修饰一个成员变量（属性），必须要显示初始化。**这里有两种初始化方式，一种是在变量声明的时候初始化；第二种方法是在声明变量的时候不赋初值，但是要在这个变量所在的类的所有的构造函数中对这个变量赋初值。

**finally**：finally作为异常处理的一部分，它只能用在 try/catch 语句中，并且附带一个语句块，表示这段语句最终一定会被执行（不管有没有抛出异常），经常被用在需要释放资源的情况下。

> 其实不然，存在finally语句不被执行的情况，见[Java：异常小记](https://www.cnblogs.com/zhuchengchao/p/14293391.html)

## wait/notify

wait/notify 用于线程的等待与唤醒，那**为什么 wait/notify 方法放在 Object 类中而不是 Thread 类中？**

**一个很明显的原因是 Java 提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。**如果线程需要等待某些锁，那么调用对象中的 `wait()` 方法就有意义了。**如果 `wait()` 方法定义在 Thread 类中，线程正在等待的是哪个锁就不明显了**。简单的说，由于 wait，notify 和 notifyAll 都是锁级别的操作，所以把他们定义在 Object 类中因为锁属于对象。

## 参考

https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

https://www.cnblogs.com/ktao/p/8586966.html

https://blog.csdn.net/pulusite/article/details/82287462

https://blog.csdn.net/zl1zl2zl3/article/details/106695410