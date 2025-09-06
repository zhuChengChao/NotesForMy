# 面经：3-Java基础

> 个人整理 :muscle: 个人专用 :muscle: 暑期实习 :muscle: 秋招 :muscle: 后端开发 :muscle: 八股文 :no_mouth:

### JDK & JRE & JVM

**JDK（Java Development Kit）：**是 Java 开发工具包，是整个 Java 的核心，包括了 Java 运行环境 JRE、Java 工具（例如：解释器-Java、编译器-Javac）和 Java 基础类库。

**JRE（ Java Runtime Environment）：**是 Java 的运行环境，包含 JVM 标准实现及 Java 核心类库。

**JVM（Java Virtual Machine）：**是 Java 虚拟机，是整个 Java 实现跨平台的最核心的部分，能够运行以 Java 语言写的软件程序。所有的 Java 程序会首先被编译为 .class 的类文件，这种类文件可以在虚拟机上执行。

> JVM 可以理解的代码就叫做 **字节码** （即扩展名为 .class 的文件），它不面向任何特定的处理器，只面向虚拟机。 Java 语⾔通过字节码的方式，在⼀定程度上解决了传统解释型语⾔执行效率低的问题，同时又保留了解释型语⾔可移植的特点。 

**关系如下：**

**JDK = JRE + 开发工具**

**JRE = JVM + 类库**

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251807017.PNG" alt="jvm比较" style="zoom:67%;" />

**总结：**

Java程序开发过程为：

1. 利用 JDK（调用 Java API）编写出 Java 源代码，存储于 .java 文件中；

2. JDK 中的编译器 Javac 将 Java 源代码编译成 Java 字节码，存储与 .class 文件中；

3. JRE 加载，验证，执行 Java 字节码；

4. JVM 将字节码解析为机器码并映射到 CPU 指令集或者 OS 的系统调用。

   > 需要格外注意的是 `.class -> 机器码` 这⼀步。 在这⼀步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的（也就是所谓的热点代码），所以后面引进了 JIT（Just In Time） 编译器，而 JIT 属于运行时编译。当 JIT 编译器完成第⼀次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们为什么经常会说 **Java 是编译与解释共存的语言**。 

### 字节码 和 JVM :exclamation:

**字节码：**

* 在 Java 中，将 JVM 可以读懂的代码，称为字节码，即 Java 中的 .class 文件；

* 机器码：机器可以读懂的代码为二进制命令，即 0/1 组成的文件。

**Java程序运行过程如下：**

`源码(xxx.java)->javac编译器->JVM可执行的字节码(xxx.class)->JVM->机器码->程序执行`

**采用字节码的好处：**

Java 语言通过字节码的方式，**在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。**所以 Java 程序运行时比较高效，而且，由于字节码并不专对一种特定的机器，因此，Java程序无须重新编译便可在多种不同的计算机上运行(一次编译到处运行)。

本质上就是采用解释性语言的好处。

> **编译型语言：**
>
> - 需要通过编译器，将**源代码编译成机器码之后才能执行的语言**。
> - 一般是通过编译和链接两个步骤：
>   - 编译是将我们的程序编译成机器码；
>   - 链接是程序和依赖库等串联起来。
> - **优点**：编译器一般会有**预编译**的过程对代码进行了优化，因为编译只做了一次，运行时不会再编译，所以**编译型语言效率高**。 
> - **缺点**：编译之后如果想要修改某一个功能，就需要整个模块重新编译。编译的时候根据对应的运行环境生成不同的机器码。不同的操作系统之间，可能会有问题。需要根据环境的不同，生成不同的可执行文件。
> - 代表语言：C、C++、Pascal、Object-C以及最近很火的苹果新语言swift，GO
>
> **解释型语言：**
>
> - 解释型语言不需要编译，相比编译型语言省了道工序，**解释型语言在运行程序的时候才逐行进行翻译**。字节码也是解释型的一部分。
> - **优点**：有良好的平台兼容性，只要安装了虚拟机，就可以。容易维护，方便快速部署，不用停机维护。
> - **缺点**：每次运行的时候都要解释一遍，性能上不如编译型语言。 
> - 代表语言：JavaScript、Python、Erlang、PHP、Perl、Ruby 

**虚拟机：**

Java 中引入了虚拟机（JVM）的概念，就是在机器和程序之间加入了一层抽象的虚拟机器。这台机器在各个平台中都给程序提供了接口。因此，**编写 Java 程序只需要面向虚拟机编程**。

> 参考：https://blog.csdn.net/a4171175/article/details/90735888

### Java 和 C++的区别

* 都是面向对象的语⾔，都支持封装、继承和多态 
* Java **不提供指针**来直接访问内存，程序内存更加安全 
* Java 的类是**单继承**的， C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承
* Java 有**自动内存管理机制**，不需要程序员⼿动释放无用内存

**从而引出的 Java 缺点**：

* 使用大量的内存，依靠虚拟机运行，运行速度相对较慢；
* 不能和底层打交道，不支持底层操作；
* 启动时间慢；
* 因为 Java 中删除了指针，所以不如 C/C++ 灵活

### 面向对象与面向过程

面向对象思想就是在计算机程序设计过程中，参照现实中事物，将事物的属性特征、行为特征抽象出来，描述成计算机事件的设计思想。 面向对象的语言中，包含了三大基本特征，即封装、继承和多态。 

> 个人感觉，应该是四大基本特征，应为：抽象、封装、继承、多态

**面向对象和面向过程的区别:** 

1. 编程思路不同：面向过程以实现功能的**函数开发**为主，而面向对象要首先抽象出**类、属性及其方法**，然后通过实例化类、执行方法来完成功能。 
2. 封装性：都具有封装性，但是面向过程是封装的是**功能**，而面向对象封装的是**数据和功能**。
3. 面向对象具有**继承性和多态性**，而面向过程没有继承性和多态性，所以面向对象优势很明显。

### 面向对象的三大特性 :exclamation::exclamation:

1. **封装：**通常认为封装是把数据和操作数据的方法封装起来，对数据的访问只能通过已定义的接口。
2. **继承：**继承是从**已有类得到继承信息创建新类的过程**。提供继承信息的类被称为父类（超类/基类），得到继承信息的被称为子类（派生类）。
3. **多态**：分为**编译时多态（方法重载）**和**运行时多态（方法重写）**。要实现运行时多态需要做两件事：**一是子类继承父类并重写父类中的方法**，**二是用父类型引用子类型对象**，这样同样的引用调用同样的方法就会根据子类对象的不同而表现出不同的行为。

> **关于继承的几点补充：**
>
> 1. 子类拥有父类对象所有的属性和方法（**包括私有属性和私有方法**），但是父类中的私有属性和方法子类是**无法访问，只是拥有**。因为在一个子类被创建的时候，首先会在内存中创建一个父类对象，然后在父类对象外部放上子类独有的属性，两者合起来形成一个子类的对象；
> 2. 子类可以拥有自己属性和方法；
> 3. 子类可以用自己的方式实现父类的方法，即重写。

> 参考：https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

### == 和 equals 的区别 :exclamation::exclamation:

**==**

对于基本类型和引用类型 == 的效果是不同的，如下：

- 基本类型：比较的**值是否相同**；
- 引用类型：比较的是是否引用同一对象，**即引用对象的地址值是否相等**

> ```java
>int int_a = 10;
> int int_b = 10;
> System.out.println(int_a==int_b);  // true，基本类型的比较
> 
> String stra = "abc";
> String strb = "abc";
> String strc = new String("abc");  // 通过new String()方法重新开辟了内存空间
> String strd = new String("abc");
> System.out.println(stra==strb);  // true
> System.out.println(stra==strc);  // false
> System.out.println(strc==strd);  // false
> ```

**equals 方法**

用来比较两个对象的内容是否相等。如果没有对 equals 方法进行重写，则比较的是引用类型的变量所指向的对象的地址（很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等）。

> ```java
>// Person没有对equals方法进行重新，而String对equals方法进行了重写
> private class Person{
>     public String name;
> 
>        public Person(String name) {
>         this.name = name;
>        }
>    }
>    
> @Test
> public void test02(){
>     String strc = new String("abc");
>     String strd = new String("abc");
>        System.out.println(strc.equals(strd));  // true
>    
>        Person p1 = new Person("张三");
>     Person p2 = new Person("张三");
>        System.out.println(p1.equals(p2));  // false
>    }
>    ```
> 
> 查看 Person 类中的 equals 方法：本质上 equals 就是 ==，即比较两个对象地址
>
> ```java
>public boolean equals(Object obj) {
>     return (this == obj);
> }
>    ```
> 
> 查看 String 类中的 equals 方法：
>
> ```java
>public boolean equals(Object anObject) {
>     if (this == anObject) {
>         return true;  // 地址相同直接返回true
>        }
>        if (anObject instanceof String) {  // 判断是否为String对象
>            String anotherString = (String)anObject;
>            int n = value.length;
>            if (n == anotherString.value.length) {  // 长度相同才进行比较
>                char v1[] = value;
>                char v2[] = anotherString.value;
>                int i = 0;
>                while (n-- != 0) {			// 按字符比较
>                    if (v1[i] != v2[i])
>                        return false;
>                    i++;
>                }
>                return true;
>            }
>        }
>        return false;
>    }
>    ```
> 
> 注意：equals不能用于基础类型的比较，因为equals是一个方法，而基础类似并不是一个对象，没有对应的方法

> 参考：
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA
>
> https://blog.csdn.net/qq_41701956/article/details/103223461

### `hashCode()` & `equals()` 关系 :exclamation::exclamation::exclamation:

**`equals()` 方法：**

首先，原始的 equals 方法，只是用于比较两个对象的内存地址是否为同一个，如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

而重写 `equals()` 方法，是**为了实现当两个对象指向的内存地址相同或者两个对象各自字段值相同，那么标识其为同一个对象**。

**`hashCode()` 方法：**

提到 hashCode，自然联想到哈希表，通过 hashCode 方法，将 key 映射到哈希表中的一个位置，从而达到最好情况下以 O(1) 的时间复杂度来查询。

```java
// Object类的源码中，hashcode()是一个native方法，哈希值的计算利用的是内存的地址
public native int hashCode();
```

而：**重写 `equals()` 一定要重写 `hashCode()` ​​**

**为什么要重写？即重写 equals 方法必须要重写 hashcode方法**

> 这个问题应该是有个前提，就是你需要用到 HashMap、HashSet 等 Java 集合，用不到哈希表的话，其实仅仅重写 `equals()` 方法也可以。而工作中的场景是常常用到 Java 集合，所以 Java 官方建议重写 `equals()` 就一定要重写 `hashCode()` 方法。 

1. **提高效率**：对于对象集合的判重，如果一个集合含有 10000 个对象实例，仅仅使用 `equals()` 方法的话，那么对于一个对象判重就需要比较 10000 次，随着集合规模的增大，时间开销是很大的。但是同时使用哈希表的话，就能**快速定位到对象的大概存储位置**，并且在定位到大概存储位置后，后续比较过程中，**如果两个对象的 hashCode 不相同，也不再需要调用 equals() 方法，从而大大减少了 equals() 比较次数**。 
2. 在实际应用过程中，如果仅仅重写了 `equals()`，而没有重写 `hashCode()` 方法，会出现什么情况？**字段属性值完全相同的两个对象因为 hashCode 不同，所以在 HashMap 中的 table 数组的下标不同，从而这两个对象就会同时存在于集合中，所以重写 `equals()` 就一定要重写 `hashCode()` 方法。**

**`hashCode()` 与 `equals()` 的相关规定：**

1. 如果两个对象相等，则 hashCode 一定也是相同的；

2. 两个对象相等，对两个对象分别调用 equals 方法都返回 true；

3. 两个对象有相同的 hashCode 值，但它们也不一定是相等的；

   > 例如：`System.out.println("通话".hashCode() == "重地".hashCode());`，结果为true
   >
   > 在散列表中，`hashCode()`相等，只是说明两个键值对的哈希值相等，但是哈希值相等，并不一定可以得出键值对相等。

4. equals 方法被覆盖过，则 hashCode 方法也必须被覆盖，上面说了原因了；

5. `hashCode()` 的**默认行为是对堆上的对象产生独特值**。如果没有重写 `hashCode()`，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

> 参考：https://blog.csdn.net/sixingmiyi39473/article/details/78306296

### Hash 冲突解决方法 :exclamation:

key 值不同的元素可能会映射到哈希表的同一地址上就会发生哈希冲突。解决办法：

**开放地址法**

* **线性探测法**：ThreadLocalMap 使用

  * 线性再散列法是形式最简单的处理冲突的方法。**插入元素时，如果发生冲突，算法会简单的从该槽位置向后循环遍历hash表，直到找到表中的下一个空槽，并将该元素放入该槽中**（会导致相同hash值的元素挨在一起和其他hash值对应的槽被占用）
  * 查找元素时，首先散列值所指向的槽，如果没有找到匹配，则继续从该槽遍历hash表，直到：
    1. 找到相应的元素；
    2. 找到一个空槽，指示查找的元素不存在；
    3. 整个hash表遍历完毕
  * **优点**：思路清晰，算法简单
  * **缺点**：
    * 删除工作非常困难，只能标上已被删除的标记，否则，将会影响以后的查找
    * 容易产生**堆聚现象**，即存入哈希表的记录在表中连成一片，一旦出现堆聚，就将引起进一步的堆聚

* **线性补偿探测法**：将线性探测的步长从 1 改为 Q ，即将上述算法中的 `hash ＝ (hash ＋ 1) % m` 改为 `hash ＝ (hash ＋ Q) % m`

  > 而且要求 Q 与 m 是互质的

* **伪随机探测法**

  * 将线性探测的步长从常数改为随机数，即：`hash ＝ (hash ＋ RN) % m`，其中 RN 为一个随机数
  * 这样就能使不同的关键字具有不同的探测次序， 从而**可以避免或减少堆聚**。

**拉链法**： hashmap 使用

* 对于相同的哈希值，使用链表进行连接
* **优点**
  * 拉链法处理冲突简单，且**无堆积现象**
  * 拉链法中各链表上的结点空间是**动态申请**的，故它更适合于造表前无法确定表长的情况
  * 在用拉链法构造的散列表中，删除结点的操作易于实现。只要简单地删去链表上相应的结点即可
* **缺点**：指针需要额外的空间

**再哈希法**：

* 同时构造多个不同的哈希函数，当发生冲突时，使用第二个、第三个、哈希函数计算地址，直到无冲突。
* 缺点：计算时间增加

**建立一个公共溢出区**：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表

### 重载和重写 :exclamation::exclamation::exclamation:

|            | 重载（Overloading） | 重写（Overriding）                   |
| ---------- | ------------------- | ------------------------------------ |
| 发生地点   | 发生本类中          | 父子类、接口与实现类                 |
| 方法名称   | 一致                | 一致                                 |
| 参数列表   | 必须修改            | 不能修改                             |
| 返回值类型 | 可以修改            | 可以修改，但必须是父类返回值的子类   |
| 异常       | 可以修改            | 可以减少或删除，但不能扩展           |
| 访问修饰符 | 可以修改            | 可以修改，但不能比父类的访问权限更低 |
| 构造函数   | 可以重载            | 不能重写                             |

1. **重载：编译时多态**，同一个类中的同名的方法，**参数列表不同，与返回值无关**。

   1. 方法名必须相同；
   2. 方法的参数列表一定不一样；
   3. 访问修饰符和返回值类型可以相同也可以不同；

2. **重写（又名覆盖）：运行时多态**，发生在子类与父类之间、子类重写父类的方法具有**相同的返回类型、更好的访问权限**，简而言之：**就是具体的实现类对于父类的该方法实现不满意，需要自己再写一个满足于自己要求的方法。**

   1. **方法名必须相同，参数列表必须相同，返回值类型可以不同**（见下方）

      > 自JDK1.5之后的版本，**返回值也可以不同，但是必须是父类返回值的子类**，其中涉及到了 Java 中的"语法糖"，即在将 *.java 编译为 *.class过程中，进行了一些自动转换，本质上生成了一种桥接方法`synthetic bridge`，在此不进行具体展开，后续关于JVM的笔记中会补充说明

   2. **访问权限不能比父类中被重写的方法的访问权限更低**。例如：如果父类的一个方法被声明为public，那么在子类中重写该方法就不能声明为 protected。

   3. 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为private、final、static的方法。

      > static：**重写是基于运行时的多态**，而static方法是编译时静态绑定的，static 方法跟类的任何实例都不相关，所以概念上不适用。 
      >
      > 静态方法：静态的方法可以被继承，但是不能重写。如果父类和子类中存在同样名称和参数的静态方法，那么该子类的方法会把原来继承过来的父类的方法隐藏，而不是重写。通俗的讲就是父类的方法和子类的方法是两个没有关系的方法，具体调用哪一个方法是看是哪个对象的引用；这种父子类方法也不再存在多态的性质。

   4. 构造方法不能被重写。


> 参考：
>
> https://blog.csdn.net/qq_42014192/article/details/89707483

### 构造方法

**构造函数特性：**

1. 名字与类名相同；
2. 没有返回值，但不能用 void 声明构造函数；
3. 生成类的对象时自动执行，无需调用；
4. Constructor 不能被 override（重写） ,但是可以 overload（重载）；

**默认构造方法：同在 Java 中定义⼀个不做事且没有参数的构造方法的作用**

Java 程序在执行子类的构造方法之前，如果没有显示用 `super()` 来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法/无参构造”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 `super()` 来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。

解决办法是：在父类里加上一个不做事且没有参数的构造方法，即**默认构造方法**。

> 参考：https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

### Java 中创建对象的方式

1. 使用 new 关键字：`Person person = new Person();`

2. 使用 Class 类的 newInstance 方法，该方法调用 **无参的构造器** 创建对象（反射）

   ```java
   // 使用 Class 类的 newInstance 方法
   String str2 = (String) Class.forName("java.lang.String").newInstance();
   String str3 = String.class.newInstance();
   
   // 使用 Constructor 类的 newInstance 方法
   Constructor<String> constructor = String.class.getConstructor();
   String str4 = constructor.newInstance();
   ```

3. 使用 clone() 方法：

   ```java
   // 要使用 clone 方法，我们需要先实现 Cloneable 接口并实现其定义的clone方法。
   Person person1 = new Person("刘桂香");
   Person person2 = (Person) person1.clone();
   ```

4. 反序列化，比如调用 ObjectInputStream 类的 `readObject()` 方法。

   ```java
   // 将对象写入文件——序列化
   ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("./data.obj"));
   out.writeObject(person2);
   out.close();
   
   // 将对象读取出来——反序列化
   ObjectInputStream in = new ObjectInputStream(new FileInputStream("./data.obj"));
   Person person3 = (Person) in.readObject();
   in.close();
   System.out.println(person3);
   ```

> 参考：https://www.cnblogs.com/wxd0108/p/5685817.html

### 静态变量和实例变量的区别

静态变量属于类的级别，而实例变量属于对象的级别。

**静态变量：**是被 static 修饰的变量，也称为类变量，它属于类，因此不管创建多少个对象，静态变量在内存中有且仅有一个拷贝；静态变量可以实现让多个对象共享内存。

> **静态的注意事项**：
>
> 1. 静态方法只能访问静态成员（包括静态成员变量和静态成员方法），不能访问非静态成员或方法；非静态方法可以访问静态也可以访问非静态方法或成员。
> 2. 静态方法中不能出现 this，super 关键字。因为静态是优先于对象存在的，所以不能出现 this，super 关键字。
> 3. 主函数是静态的。

**实例变量：**属于某一实例，需要先创建对象，然后通过对象才能访问到它。

> 参考：https://www.cnblogs.com/jasonboren/p/11052500.html

### 抽象类和接口的区别​ :exclamation::exclamation:

**抽象类**：使用 abstract 修饰，子类用 extends 继承；

**接口**：使用 interface 修饰，采用 implements 实现；

**总结**：抽象类和接口的异同

|                  | 抽象类                             | 接口                                                         |
| ---------------- | ---------------------------------- | ------------------------------------------------------------ |
| **类修饰符**     | public/不加                        | public/不加                                                  |
| **构造方法**     | 可以有                             | 不能有                                                       |
| **普通成员变量** | 可以有<br />与类的普通成员变量相同 | 不能有，即使定义了也是被默认修饰为<br /> public static final |
| **普通成员方法** | 可以有，和类的普通成员方法相同     | JKD8 中可以有用default修饰的有方法体的方法<br />JDK9 之后可以有 |
| **抽象方法**     | 可以用 public/protected/不加       | 只能是 public 的，且默认是 public abstract                   |
| **静态方法**     | 可以有                             | JDK8 中可以有单用static修饰的有方法体的方法<br />JDK9 之后可以有 |
| **静态成员**     | 可以有                             | 有，且定义的变量会被默认加上 public static final             |
| **继承/实现**    | 只能继承一个抽象类                 | 可以实现多个接口<br />接口之间还可以多继承                   |

其中表中的**不加**，即表示默认的访问修饰符，只在本包中可见，在外包中不可见；如果外包中继承/实现就会无法找到相应的类/接口/抽象方法；而在本包中使用无恙。

> JDK 1.8 前，抽象类的方法默认访问权限为 protected
>
> JDK 1.8 时，抽象类的方法默认访问权限变为 default

**构造函数：**

1. 抽象类中可以定义构造函数（但是抽象类不能被实例化）；
2. 接口不能定义构造函数；

> ```java
>// 抽象类
> public abstract class AbstractTest {
>     // 正常，抽象类可以定义构造函数
>     public AbstractTest() {
>            System.out.println("abstract class");
>        }
>    }
>    
> // 接口
> public interface InterfaceTest {
>     // error：Interface abstract method cannot have body
>     public InterfaceTest(){}
> }
> ```

**成员变量：**

1. 抽象类中的成员权限可以是 public、默认、protected、private（但抽象方法是为了重写的，因此不能被 private 修饰）；
2. 而接口中的成员**只可以是 public**（方法默认：public abstract 、成员变量默认：public static final）；

> ```java
>// 抽象类：
> public abstract class AbstractTest {
>     public AbstractTest() {
>         System.out.println("abstract class");
>        }
>    	
>        // 抽象类的成员变量
>     public int a = 0;    // public修饰
>        int b = 0;           // 默认
>        protected int c = 0; // protected
>        private int d = 0;   // private,成员变量是可以用private修饰的
>        private String e = "hello";
>    
>        public void out(){
>         System.out.println(d);
>            System.out.println(e);
>        }
>    }
>    
> /******************分割线*********************/
> 
> // 接口：
> public interface InterfaceTest {
>    	
>        // 定义的变量默认就是 public static final
>        int a = 10;
>        // error:Modifier 'protected' not allowed here
>     protected int b = 10;
>     // error:Modifier 'default' not allowed here
>     default int c = 20;
>     // info:Modifier 'public' is redundant for interface field
>     public int d = 30;
>     // public static final可以不加，是默认的
>     public static final int e = 40;
> }
> ```

**成员方法：**

1. 抽象类中可以有抽象方法和具体方法。
2. JDK8 之前接口中只能有抽象方法（public abstract），**但在 JDK1.8中，允许在接口中包含带有具体实现的方法，使用 default 修饰，这类方法就是默认方法。**
3. 抽象类中可以包含静态方法。
4. JDK8 之前接口中不可以包含静态方法，**同样在JDK1.8 以后可以包含**，之前不能包含是因为，接口不可以实现方法，只可以定义方法，所以不能使用静态方法（因为静态方法必须实现）。现在可以包含了，**只能直接用接口调用静态方法**。
5. JDK1.8中，接口仍然不可以包含静态代码块。

> 演示一下：
>
> ```java
> // 抽象类
> public abstract class AbstractTest {
>        // ...和前面一页
>    
>        // 具体方法，在继承AbstractTest的类中可以直接调用
>     public void method1(){
>            System.out.println(d);
>            System.out.println(e);
>        }
>    
>        // 静态方法，可以通过 AbstractTest.method2()进行调用
>        public static String f = "world";
>     public static void method2(){
>            System.out.println(f);
>        }
>    
>        // 抽象方法，后续抽象方法必须是继承类中实现，否则继承类也是个抽象类
>        public abstract void method3();
>     abstract void method4();
>        protected abstract void method5();
>        // error: illegal combination of midifiers: 'abstract' and 'private'
>        // private abstract void method6();
>    }
>    
> /******************分割线*********************/
>    
>    // 接口
>    public interface InterfaceTest {
>    
>        int a = 10;
>        // public static final可以不加，可以不加
>     public static final int b = 20;
> 
>     // 抽象方法：public abstract可以不加，是默认，且只能用这个修饰
>     public abstract void method1();
> 
>     // 默认方法：必须只能用default修饰
>     default void method2(){
>            System.out.println("默认方法");
>        }
>    
>     // 静态方法: 只能通过InterfaceTest.method3()调用，不能通过实现的类去调用
>        // 只能用public去修饰，可加可不加，其他的 protected/private/default都是不允许的
>        public static void method3(){
>         System.out.println("静态方法");
>        }
>    }
>    ```

### short 自动类型转换

**问：**

* `short s1 = 1; s1 = s1 + 1;` 有什么错？
* 那么 `short s1 = 1; s1 += 1;`呢？有没有错误？

**答：**

对于 `short s1 = 1; s1 = s1 + 1;` 来说，在 `s1 + 1` 运算时会自动提升表达式的类型为 int ，那么将 int 型值赋值给 short 型变量，s1 会出现类型转换错误。

对于 `short s1 = 1; s1 += 1;` 来说，**`+=` 是 Java 语言规定的运算符**，Java 编译器会对它进行特殊处理，因此可以正确编译。

### 装箱和拆箱：Integer & int

**装箱拆箱**：

1. 自动装箱是 Java 编译器在基本数据类型和对应的包装类之间做的一个转化。比如：把 int 转化成 Integer，double 转化成 Double 等等。反之就是自动拆箱。

2. 原始类型：boolean、char、byte、short、int、long、float、double

3. 封装类型：Boolean、Character、Byte、Short、Integer、Long、Float、Double

**Integer 和 int 的区别**：

1. int 是 Java 的八种基本数据类型之一，而 Integer 是 Java 为 int 类型提供的封装类；
2. int 型变量的默认值是 0，Integer 变量的默认值是 null，这一点说明 Integer 可以区分出未赋值和值为 0 的区分；
3. Integer 变量必须实例化后才可以使用，而 int 不需要。

**关于 Integer 和 int 的比较的延伸：**

1. 由于 Integer 变量实际上是对一个 Integer 对象的引用，所以两个通过 new 生成的 Integer 变量永远是不相等的，因为其内存地址是不同的；

   ```java
   Integer i = new Integer(100);
   Integer j = new Integer(100);
   System.out.println(i==j);  // false，==比较的是内存地址，必然为false
   System.out.println(i.equals(j));  // 这个就是true了
   ```

2. Integer 变量和 int 变量比较时，只要两个变量的值是相等的，则结果为 true。因为包装类 Integer 和基本数据类型 int 类型进行比较时，**Java 会自动拆包装类为 int**，然后进行比较，实际上就是两个 int 型变量在进行比较；

   ```java
   Integer i = new Integer(100);
   int j = 100;
   System.out.println(i==j);  // true，i会自动拆包
   
   Integer k = new Integer(100);
   Integer z = new Integer(200);
   // k+i进行了算数运算，触发自动拆箱，因此比较的是数值是否相等
   System.out.println(z==(k+i));  // true
   
   // 注：和其他类型比较时的情况：
   Long x = 200L;
   Long y = 0L;
   // k+i进行了算数运算，触发自动拆箱，因此比较的是数值是否相等
   System.out.println(x==(k+i));  // true
   // k+i进行了算数运算，触发自动拆箱，使用equals时自动装箱为Integer类型的数据，x为Long类型数据，类型不一致，结果为false
   System.out.println(x.equals(k+i));  // false
   // 由于此时y为Long类型数据，因此装箱为Long类型数据，比较结果为true
   System.out.println(x.equals(k+i+y)); // true
   ```

3. 非 new 生成的 Integer 变量和 `new Integer()` 生成的变量进行比较时，结果为 false。因为**非 new 生成的 Integer 变量指向的是 Java 常量池中的对象**（有范围的，具体见下方），而 `new Integer()` 生成的变量指向堆中新建的对象，两者在内存中的地址不同；

   ```java
   Integer i = new Integer(100);
   Integer j = 100;  // 实际调用的是Integer.valueOf(int i)函数
   System.out.println(i==j);  // false
   ```

4. 对于两个非 new 生成的 Integer 对象进行比较时，如果两个变量的值在区间 [-128, 127] 之间，则比较结果为 true，否则为 false。Java 在编译 Integer i = 100 时，会编译成 `Integer i = Integer.valueOf(100)`，而 Integer 类型的 valueOf 的源码如下所示：

   ```java
   Integer i = 127;
   Integer j = 127;
   System.out.println(i==j);  // true，有缓存存在
   
   Integer i = 128;
   Integer j = 128;
   System.out.println(i==j);  // false，超出缓存了
   
   // 源码
   public static Integer valueOf(int i) {
       if (i >= IntegerCache.low && i <= IntegerCache.high)
           return IntegerCache.cache[i + (-IntegerCache.low)];
       return new Integer(i);
   }
   ```

   从上面的代码中可以看出：Java 对于 [-128, 127] 之间的数会进行**缓存**，比如：`Integer i = 127`，会将 127 进行缓存，下次再写 `Integer j = 127` 的时候，就会直接从缓存中取出，而对于这个区间之外的数就需要 new 了。

**包装类的缓存**：针对上述第4点

- Boolean：全部缓存
- Byte：全部缓存
- Character：<= 127 缓存
- Short：-128 ~ 127 缓存
- Long：-128 ~ 127 缓存
- Integer：-128 ~ 127 缓存
- Float：没有缓存
- Doulbe：没有缓存

> 参考：
>
> https://blog.csdn.net/aoxiangzhe/article/details/81297157
>
> https://www.cnblogs.com/javatech/p/3650460.html

### switch 表达式

在 `switch(expr)` 中，expr 只能是一个**整数表达式** / **枚举常量** / **String类型**

整数表达式：可以是 int 基本数据类型或者 Integer 包装类型。由于byte、short、char 都可以**隐式转换**为 int，所以，这些类型以及这些类型的包装类型也都是可以的。

**JDK1.7 版本之后 String类 和 枚举类 就可以作用在 switch 上了。**这个功能其实是语法糖。

而 long 类型不符合 switch 的语法规定，并且不能被隐式的转换为 int 类型，所以，不能作用于 switch 语句中。

**switch支持String类型：**

```java
public class Candy6_1 {
    public static void choose(String str) {
        switch (str) {
            case "hello": {
                System.out.println("hello");
                break;
            }
            case "world": {
                System.out.println("world");
                break;
            }
            case "BM":{
                System.out.println("BM");
                break;
            }
            case "C.":{
                System.out.println("C.");
                break;
            }
        }
    }
}
```

> **注意** switch 配合 String 和枚举使用时，变量不能为null，原因分析完语法糖转换后的代码应当自然清楚

上述代码会被编译器转换为：

```java
public class Candy6_1 {
    public Candy6_1() {} 
    public static void choose(String str) {
        byte x = -1;
        switch(str.hashCode()) {
            case 2123: // hashCode值可能相同，需要进一步用 equals 比较
                if (str.equals("C.")) {
                    x = 0;
                } else if (str.equals("BM")) {
                    x = 1;
                }
            case 99162322: // hello 的 hashCode
                if (str.equals("hello")) {
                    x = 2;
                } 
                break;
            case 113318802: // world 的 hashCode
                if (str.equals("world")) {
                    x = 3;
                }
        } 
        switch(x) {
            case 0:
                System.out.println("C.");
                break;
            case 1:
                System.out.println("BM");
                break;
            case 2:
                System.out.println("hello");
                break;
            case 3:
                System.out.println("world");
        }
    }
}
```

可以看到，**执行了两遍 switch**，第一遍是根据字符串的 hashCode 和 equals 将字符串的转换为相应 byte 类型，第二遍才是利用 byte 执行进行比较。

为什么第一遍时必须既比较 hashCode，又利用 equals 比较呢？hashCode 是为了提高效率，减少可能的比较；而 equals 是为了防止 hashCode 冲突，例如上方的 `BM` 和 `C.` 这两个字符串的 hashCode 值都是2123。

**switch 支持 enum 类型：**

switch 枚举的例子，原始代码： 

```java
enum Sex {
    MALE, FEMALE;
}
public class Candy7 {
    public static void foo(Sex sex) {
        switch (sex) {
            case MALE:
                System.out.println("男"); break;
            case FEMALE:
                System.out.println("女"); break;
        }
    }
}
```

转换后代码： 

```java
public class Candy7{
    /**
      * 定义一个合成类（仅jvm使用，对我们不可见）
      * 用来映射枚举的 ordinal 与数组元素的关系
      * 枚举的 ordinal 表示枚举对象的序号，从 0 开始
      * 即 MALE 的 ordinal()=0，FEMALE 的 ordinal()=1
      */
    static class $MAP {
        // 数组大小即为枚举元素个数，里面存储case用来对比的数字
        static int[] map = new int[2];
        static {
            map[Sex.MALE.ordinal()] = 1;
            map[Sex.FEMALE.ordinal()] = 2;
        }
    } 
    public static void foo(Sex sex) {
        int x = $MAP.map[sex.ordinal()];
        switch (x) {
            case 1:
                System.out.println("男");
                break;
            case 2:
                System.out.println("女");
                break;
        }
    }
}
```

> 关于枚举类的一点点说明：
>
> ```java
> enum Sex {  // 枚举类其实也是一个class
>        // 实例对象，与普通类的区别为：
>        //    普通类对象无穷多个；枚举类中实例对象是有限的，此处只有两个实例对象
>     MALE, FEMALE  
> }
> ```
>
> 转换后代码：
>
> ```java
> public final class Sex extends Enum<Sex> {
>        public static final Sex MALE;
>        public static final Sex FEMALE;
>        private static final Sex[] $VALUES;
> 
>        static {
>            MALE = new Sex("MALE", 0);
>            FEMALE = new Sex("FEMALE", 1);
>            $VALUES = new Sex[]{MALE, FEMALE};
>        }
>     
>        // 唯一构造函数，程序员不能调用此构造函数
>     private Sex(String name, int ordinal) {
>         super(name, ordinal);
>     } 
>     public static Sex[] values() {
>         return $VALUES.clone();
>     } 
>     public static Sex valueOf(String name) {
>         return Enum.valueOf(Sex.class, name);
>        }
>    }
>    ```

> 参考：
>
> https://blog.csdn.net/u012110719/article/details/46316659
>
> https://www.bilibili.com/video/BV1yE411Z7AP?p=136 到 https://www.bilibili.com/video/BV1yE411Z7AP?p=138

### 字节和字符的区别

**字节 byte：**

1. 一个字节包含8个位(bit)，因此byte的取值范围为-128~127；
2. 字节是存储容量的基本单位；

**字符 char：**

1. 字符是数字、字母、汉字以及其他语言的各种符号；
2. Java 采用 unicode 来表示字符，**Java 中的一个 char 是2个字节**，一个中文或英文字符的 unicode 编码都占2个字节；
3. 在 GB2312 编码或 GBK 编码中，一个英文字母字符存储需要1个字节，一个汉字字符存储需要2个字节；
4. 在 UTF-8 编码中，一个英文字母字符存储需要1个字节，一个汉字字符储存需要3到4个字节；
5. 在 UTF-16 编码中，一个英文字母字符存储需要2个字节，一个汉字字符储存需要3到4个字节（Unicode扩展区的一些汉字存储需要4个字节）；

> 参考：https://www.cnblogs.com/wugongzi/p/11334052.html

### String 设计为不可变类 :exclamation:

**不可变类：**类在被实例化之后，不可被重新赋值，Java 提供的八个包装类和String类都是不可变类。

通过 String 的源码可以看到

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];  // 用private final修饰成员变量
	/** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

可以看到 **String 的本质是一个char数组**，是对字符串数组的封装，并且是被 final 修饰的，创建后不可改变。

在 Java 中将 String 设计成不可变的是综合考虑到各种因素的结果。主要的原因主要有以下三点：

1. **便于实现字符常量串池（StringTable）**：字符串常量池是 Java 堆内存中一个特殊的存储区域, 当创建一个 String 对象时，假如此字符串值已经存在于常量池中，则不会创建一个新的对象，而是引用已经存在的对象；

   > 即在堆中有一个 StringTable，用于存放字符串常量，具体内容在JVM笔记中说明

2. **加快字符串处理速度：**由于 String 是不可变的，**保证了 hashcode 的唯一性，于是在创建String对象时其hashcode就可以放心的缓存了，不需要重新计算。**这也就是 Map 喜欢将 String 作为 Key 的原因，处理速度要快过其它的键对象。所以 HashMap 中的键往往都使用 String。

3. **避免安全问题：**String 被许多的 Java 类(库)用来当做参数，例如：网络连接地址 URL、文件路径 path、还有反射机制所需要的 String 参数等， **假如 String 不是固定不变的，将会引起各种安全隐患**。

4. **使多线程安全：**

   ```java
   // 下面的例子中可以看到：可变的StringBuilder参数就被改变了，而String传入的参数却未被修改
   public class test {
       // 不可变的String
       public static String appendStr(String s) {
           s += "bbb"; // 创建一个新的String对象
           return s;
       }
   
       // 可变的StringBuilder
       public static StringBuilder appendSb(StringBuilder sb) {
           return sb.append("bbb");
       }
   
       public static void main(String[] args) {
           String s = new String("aaa");
           String ns = test.appendStr(s);
           System.out.println("String aaa>>>" + s.toString());  // String aaa>>>aaa
           // StringBuilder做参数
           StringBuilder sb = new StringBuilder("aaa");
           StringBuilder nsb = test.appendSb(sb);
           System.out.println("StringBuilder aaa >>>" + sb.toString());  // StringBuilder aaa >>>aaabbb
       }
   }
   ```

   所以 String 不可变的安全性就体现在这里。在并发场景下，多个线程同时读一个资源，是安全的，不会引发竞争，但对资源进行写操作时是不安全的，**但是不可变对象不能被写，所以保证了多线程的安全**。

> 额外说明：创建自定义不可变类需要遵守的规则：
>
> 1. 使用 private 和 final 修饰成员变量。
>
> 2. 提供带参构造方法，用于初始化成员变量。
>
> 3. 不要为成员变量提供 setter 方法。
>
> 4. 如果成员变量中有可变类时需要重写 Object 中的 hashCode 方法和 equals 方法。
>
> > 参考：https://www.php.cn/java/base/437469.html

> 参考：https://www.cnblogs.com/wkfvawl/p/11693260.html

### String & StringBuilder & StringBuffer :exclamation::exclamation::exclamation:

1. **执行效率：**StringBuilder > StringBuffer > String

   **String最慢的原因：**String 为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的。

2. **线程安全：**在线程安全上，StringBuilder 是线程不安全的，而 StringBuffer 是线程安全的

   如果一个 StringBuffer 对象在字符串缓冲区被多个线程使用时，StringBuffer 中很多方法可以带有**synchronized关键字**，所以可以保证线程是安全的，但 StringBuilder 的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的 StringBuilder。

   > **StringBuffer 的补充：**
   >
   > 说明：StringBuffer 中并不是所有方法都使用了 Synchronized 修饰来实现同步：
   >
   > ```java
   > // 带关键字synchronized
   > @Override
   > public synchronized StringBuffer insert(int offset, Object obj) {
   >        toStringCache = null;
   >        super.insert(offset, String.valueOf(obj));
   >        return this;
   > }
   > 
   > // 不带关键字synchronized
   > @Override
   > public StringBuffer insert(int dstOffset, CharSequence s) {
   >        // Note, synchronization achieved via invocations of other StringBuffer methods，通过调用其他的StringBuffer来实现同步
   >        // after narrowing of s to specific type
   >        // Ditto for toStringCache clearing
   >        super.insert(dstOffset, s);
   >        return this;
   > }
   > ```

3. 总结一下：

   **String：适用于少量的字符串操作的情况**

   **StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况**

   **StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况**

> 参考：https://www.cnblogs.com/su-feng/p/6659064.html

### String 对象创建的区别

**创建方式1：**`String str = "i"`

**创建方式2：**`String str = new String("i")`

不一样，因为内存的分配方式不一样。`String str = "i"` 的方式，Java 虚拟机会将其分配到常量池中；而 `String str = new String("i")` 则会被分到堆内存中。

```java
public class StringTest {
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = "abc";
        String str3 = new String("abc");
        String str4 = new String("abc");
        // 在编译阶段会被优化，会进行拼接后在常量池中进行查找，若存在直接指向
        String str5="ab"+"c"; 

        System.out.println(str1 == str2);      // true
        System.out.println(str1 == str3);      // false
        System.out.println(str3 == str4);      // false
        System.out.println(str3.equals(str4)); // true
        System.out.println(str1 == str5);      // true
    }
}
```

**进一步说明：**

在执行 `String str1 = "abc"` 的时候，JVM 会首先检查字符串常量池中是否已经存在该字符串对象，如果已经存在，那么就不会再创建了，直接返回该字符串在字符串常量池中的内存地址；如果该字符串还不存在字符串常量池中，那么就会在字符串常量池中创建该字符串对象，然后再返回。所以在执行 `String  str2 = "abc"` 的时候，因为字符串常量池中已经存在 “abc” 字符串对象了，就不会在字符串常量池中再次创建了，所以栈内存中 str1 和 str2 的内存地址都是指向 "abc" 在字符串常量池中的位置，所以 `str1 == str2` 的运行结果为 true。

而在执行 `String  str3 = new  String("abc")` 的时候，JVM 会首先检查字符串常量池中是否已经存在 “abc” 字符串，如果已经存在，则不会在字符串常量池中再创建了；如果不存在，则就会在字符串常量池中创建 "abc" 字符串对象，**然后再到堆内存中再创建一份字符串对象**，把字符串常量池中的 "abc" 字符串内容拷贝到内存中的字符串对象中，然后返回堆内存中该字符串的内存地址，即栈内存中存储的地址是堆内存中对象的内存地址。`String  str4 = new String("abc")` 是在堆内存中又创建了一个对象，所以 `str3 == str4` 运行的结果是 false。

![stringtest](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808713.PNG)

> **上图是有点问题的**：
>
> * 在 JDK6 用永久代实现的方法区中的确是包含了字符串常量区
>
> * 而在 JDK8 中字符串常量区已经从方法区（不在堆内存）中移到了堆空间中，即上图的字符串常量区应该在堆内存中
> * 更为具体的关于方法区的内容见：[6-JVM：方法区]()

> 参考：
>
> https://www.cnblogs.com/nov5026/p/7248509.html
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

### String::intern() 方法 :exclamation:

> 关于 **字符串常量池** 的概念解释：[6-JVM：class常量池 & 运行时常量池 & 字符串常量池]()

使用 String 类中的 `intern()` 方法，**主动将字符串常量池中还没有的字符串对象放入字符串常量池**

* JDK8 将这个字符串对象尝试放入字符串常量池（已经移到了堆中）
  * 如果有则并不会放入，直接返回相应的字符串对象；
  * 如果没有则放入字符串常量池，**其实就是记录了一下首次出现的实例引用**
* JDK1.6 将这个字符串对象尝试放入字符串常量池 （还在方法区中）
  * 如果有则并不会放入；
  * 如果没有会把此对象复制一份（即**调用intern方法的对象和放入串池的对象是两个对象**），然后把复制的对象放入字符串常量池（在永久代中），再把串池中的对象返回

**对于intern说明1：JDK1.8环境下**

```java
// 将“ab” 放入串池，此时StringTable为["ab"]
String x = "ab";
// "a", "b" 是常量被放入串池当中, 此时StringTable为["ab", "a", "b"]
// 而new出来的对象在堆中new String("a")与new String("b")
// 而s仅为堆中的一个对象 String("ab")，并不在串池中
String s = new String("a") + new String("b");

// 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池
String s2 = s.intern();  // 会返回串池中的对象

System.out.println(s2 == x);  // true, 都是指向了串池中的对象
System.out.println(s == x);   // flase，s为堆中的对象
```

**对于intern说明2.1：JDK1.6环境下**

```java
String s = new String("a") + new String("b");
// 上述执行后：StringTable为["a", "b"]
// s为堆中的一个对象 String("ab")
String s2 = s.intern();  // 将s拷贝一份放入串池，返回结果为串池中的对象
String x = "ab";  // x也指向串池中的对象
System.out.println(s2 == x);  // true, 都是指向了串池中的对象
System.out.println(s == x);   // flase，s为堆中的对象，x在永久代中
```

**对于intern说明2.2：JDK1.8环境下**

```java
// 在上述说明2.1的前提下，在JDK8下执行下述判断
System.out.println(s2 == x);  // true, 都是指向了串池中的对象
System.out.println(s == x);   // true
// JDK8：s.intern()把这个s对象放入串池，而JDK6则是拷贝s一份放入，因此上述为false，这里为true
// 更进一步：JDK8的字符串常量池是在堆中的，s.intern()当s没有在常量池中时，常量池只是记录了一下s出现的引用罢了
```

**对于intern说明3：JDK1.8环境下**

```java
String str1 = new StringBuilder("a").append("b").toString();
System.out.println(str1);  // ab
System.out.println(str1.intern());  // ab
System.out.println(str1 == str1.intern());  // true

String str2 = new StringBuilder("ja").append("va").toString();
System.out.println(str2);  // java
System.out.println(str2.intern());  // java
System.out.println(str2 == str2.intern());  // false !!!
```

* 对于 `java.intern()` 后，对比 `str2 == str2.intern()` 结果为 false 的进一步说明
* 这是由于 JVM 在加载 `sun.misc.Version` 这个类时，已经将字符串 `java` 放入了字符串常量池中，因此再次通过 `intern` 放入 `java` 时，会返回字符串常量池中已经存在的那个 `java` 对象

### Object & String & Math 的常用方法

**Object 常用方法：**

- `clone()` 方法：用于创建并返回当前对象的一份拷贝；
- `getClass()` 方法：用于返回当前运行时对象的 Class；
- `toString()` 方法：返回对象的字符串表示形式；
- `finalize()` 方法：实例被垃圾回收器回收时触发的方法；
- `equals()` 方法：用于比较两个对象的内存地址是否相等，一般需要重写；
- `hashCode()` 方法：用于返回对象的哈希值，重写了equals方法时也需要对其进行重写；
- `notify()` 方法：唤醒一个在此对象监视器上等待的线程。如果有多个线程在等待只会唤醒一个；
- `notifyAll()` 方法：作用跟 notify() 一样，只不过会唤醒在此对象监视器上等待的所有线程，而不是一个线程；
- `wait()` 方法：让当前对象等待；
- .......

**String常用的方法**：

* `indexOf()`：返回指定字符的索引
* `charAt()`：返回指定索引处的字符
* `replace()`：字符串替换
* `trim()`：去除字符串两端空白
* `split()`：分割字符串，返回一个分割后的字符串数组
* `getBytes()`：返回字符串的 byte 类型数组
* `length()`：返回字符串长度
* `toLowerCase()`：将字符串转成小写字母
* `toUpperCase()`：将字符串转成大写字符
* `substring()`：截取字符串
* `equals()`：字符串比较
* `toCharArray()`：返回字符串对于的char数组

**Math的常用方法**

* 特殊说明：`Math.round(-1.5)`：四舍五入函数，等于 -1，因为在数轴上取值时，中间值（0.5）**向右取整**，所以正 0.5 是往上取整，结果为1，负 0.5 是直接舍弃，结果为0。

* 算数计算：
  * `Math.sqrt()` : 计算平方根
  * `Math.cbrt()` : 计算立方根
  * `Math.pow(a, b)` : 计算a的b次方
  * `Math.max( , )` : 计算最大值
  * `Math.min( , )` : 计算最小值
  * `Math.abs()` : 取绝对值

* 进位：
  * `Math.ceil()`: 天花板的意思，就是逢余进一
  * `Math.floor()` : 地板的意思，就是逢余舍一
  * `Math.rint()`: 四舍五入，返回double值。注意.5的时候会取偶数
  * `Math.round()`: 四舍五入，float时返回int值，double时返回long值

* 随机数：
  * `Math.random()`: 取得一个[0, 1)范围内的随机数

> 参考：https://blog.csdn.net/xuexiangjys/article/details/79849888

### final & finally & finalize :exclamation::exclamation:

**final：**final 可以用来修饰类，方法和变量（成员变量或局部变量）

1. **修饰类**：当用 final 修饰类的时，表明该类不能被其他类所继承。（final 类中所有的成员方法都会隐式的定义为 final 方法）
2. **修饰方法**：把方法锁定，以防止继承类对其进行更改。
3. **修饰变量**：final 成员变量表示常量，只能被赋值一次，赋值后其值不再改变。类似于 C++ 中的 const。
   
   1. 当 final 修饰一个**基本数据类型**时，表示该基本数据类型的值一旦在初始化后便不能发生变化；
   
   2. 如果  final 修饰一个**引用类型**时，则在对其初始化之后便不能再让其指向其他对象了，但该引用所指向的对象的内容是可以发生变化的，这本质上是一回事，因为引用的值是一个地址，final 要求值，即地址的值不发生变化。
   
      > 进一步举例：**final 修饰的 StringBuffer 还可以进行 append**
      >
      > final 修饰的是一个引用变量，**那么这个引用始终只能指向这个对象，但是这个对象内部的属性是可以变化的**。
      >
      > **官方文档解释：**
      >
      > once a final variable has been assigned, it always contains the same value. If a final variable holds a reference to an object, then the state of the object may be changed by operations on the object, but the variable will always refer to the same object.
      >
      > 一旦一个final修饰的变量被赋值了，它总是包含相同的值。如果一个final变量持有对一个对象的引用，那么该对象的状态可以通过对该对象的操作而改变，但该变量将始终引用同一个对象。
   
   注意：**final修饰一个成员变量（属性），必须要显示初始化。**这里有两种初始化方式，一种是在变量声明的时候初始化；第二种方法是在声明变量的时候不赋初值，但是要在这个变量所在的类的所有的构造函数中对这个变量赋初值。

**finally**：finally作为异常处理的一部分，它只能用在 try/catch 语句中，并且附带一个语句块，表示这段语句最终一定会被执行（不管有没有抛出异常），经常被用在需要释放资源的情况下。

> 其实不然，存在finally语句不被执行的情况，见后续说明

**finalize：**Object 类的一个方法，在垃圾回收时会调用被回收对象的 finalize。

> 参考：https://www.cnblogs.com/ktao/p/8586966.html

### finally 关键字 :exclamation:

**finally 概述**

在 Java 语言的异常处理中，finally 块的作用就是为了保证无论出现什么情况，**finally 块里的代码一定会被执行**。

由于程序执行 return 就意味着结束对当前函数的调用并跳出这个函数体，因此任何语句要执行都只能在 return 前执行（除非碰到 exit 函数），**因此 finally 块里的代码也是在 return 之前执行的**。

此外，如果 try-finally 或者 try-catch-finally 中都有 return，**那么 finally 块中的 return 将会覆盖别处的 return 语句**，最终返回到调用者那里的是 finally 中 return 的值。即：程序在执行到 return 时会首先将返回值存储在一个指定的位置，然后再去执行 finally 块，最后再返回。

此外，当 finally 中存在 return，**则会吞掉try-catch中的异常信息**，见下面的例子。

**从字节码的角度进行解释 finally**

- **说明1：finally一定会被执行**

  ```java
  // Java代码
  public class Demo3_11_4 {
      public static void main(String[] args) {
          int i = 0;
          try {
              i = 10;
          } catch (Exception e) {
              i = 20;
          } finally {
              i = 30;
          }
      }
  }
  
  // 字节码：
  Code:
    stack=1, locals=4, args_size=1
      0: iconst_0
      1: istore_1           // 0 -> i
      2: bipush        10   // try --------------------------------------
      4: istore_1           // 10 -> i                                  |
      5: bipush        30   // finally                                  |
      7: istore_1           // 30 -> i                                  |
      8: goto          27   // return -----------------------------------
      11: astore_2          // catch Exceptin -> e ----------------------
      12: bipush        20  //                                          |
      14: istore_1          // 20 -> i                                  |
      15: bipush        30  // finally                                  |
      17: istore_1		  // 30 -> i                                  |
      18: goto          27  // return -----------------------------------
      21: astore_3          // catch any -> slot 3 ----------------------
      22: bipush        30  // finally                                  |
      24: istore_1          // 30 -> i                                  |
      25: aload_3           // <- slot 3  有这个slot3                    |
      26: athrow            // throw 抛出---------------------------------
      27: return
    Exception table:
      from    to  target type
      2     5    11   Class java/lang/Exception
      2     5    21   any   // 剩余的异常类型，比如 Error，监测[2, 5)中的其他异常/error
      11    15    21   any  // 剩余的异常类型，比如 Error，监测[11, 15)->catch块中的其他异常/error
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
      12       3     2     e   Ljava/lang/Exception;
      0      28     0  args   [Ljava/lang/String;
      2      26     1     i   I
  ```

  可以看到 finally 中的代码被复制了 3 份，分别放入 try 流程，catch 流程以及 catch 剩余的异常类型流程

- **说明2：而当finally代码块中出现了return**

  ```java
  // Java代码
  public class Demo3_12_1 {
      public static void main(String[] args) {
          int result = test();
          System.out.println(result);  // 20
      }
  
      public static int test() {
          try {
              return 10;
          } finally {
              return 20;
          }
      }
  }
  
  // 字节码
  Code:
    stack=1, locals=2, args_size=0
      0: bipush        10  // <- 10 放入栈顶
      2: istore_0          // 10 -> slot 0 (把10放入slot0中，并从栈顶移除)
      3: bipush        20  // < - 20 放入栈顶
      5: ireturn           // 返回栈顶 int(20)
      6: astore_1          // catch any -> slot 1
      7: bipush        20  // <- 20 放入栈顶
      9: ireturn           // 返回栈顶 int(20)
    Exception table:
      from    to  target type
      0     3     6   any
  ```

  由于 finally 中的 ireturn 被插入了所有可能的流程，因此返回结果肯定以 finally 的为准

  至于字节码中第 2 行，似乎没啥用，且留个伏笔，看下个例子：`Demo3_12_2`

  跟上例中的 finally 相比，**发现没有 athrow 了**，这告诉我们：**如果在 finally 中出现了 return，会吞掉异常！**可以试一下下面的代码

  ```java
  public class Demo3_12_1 {
      public static void main(String[] args) {
          int result = test();
          System.out.println(result);  // 20
      }
  
      public static int test() {
          try {
              int i = 1/0;  // 加不加结果都一样
              return 10;
          } finally {
              return 20;
          }
      }
  }
  ```

  很危险的操作，异常被吞掉了，即使发生了异常也不知道了

- **说明三：finally 对返回值影响**

  ```java
  // Java代码
  public class Demo3_12_2 {
      public static void main(String[] args) {
          int result = test();
          System.out.println(result);  // 10
      }
  
      public static int test() {
          int i = 10;
          try {
              return i;
          } finally {
              // 对于基本类型，在finally块中改变 return 的值没有任何影响
              i = 20;
          }
      }
  }
  
  // 字节码：
  Code:
    stack=1, locals=3, args_size=0
      0: bipush        10  // <- 10 放入栈顶
      2: istore_0		     // 10 -> i
      3: iload_0			 // <- i(10)
      4: istore_1		     // 10 -> slot1，暂存至slot1，目的是为了固定返回值 ☆
      5: bipush        20  // <- 20 放入栈顶
      7: istore_0          // 20 -> i
      8: iload_1		     // <- slot 1(10) 载入 slot 1 暂存的值 ☆
      9: ireturn           // 返回栈顶的 int(10)
      10: astore_2         // 期间出现异常，存储异常到slot2中
      11: bipush        20 // <- 20 放入栈顶
      13: istore_0         // 20 -> i
      14: aload_2          // 异常加载进栈
      15: athrow           // athrow出异常
    Exception table:
      from    to  target type
      3     5    10   any
  ```

  **程序在执行到 return 时会首先将返回值存储在一个指定的位置**（体现在字节码中的4），然后去执行 finally 块，最后再返回。

  因此，对基本数据类型，在 finally 块中改变 return 的值没有任何影响，直接覆盖掉（即上述的例子）；

  但是，对引用类型是有影响的，返回的是在 finally 对前面 return 语句返回对象的修改值

  ```java
  public static void main(String[] args) {
      System.out.println(test().toString());  // abcdef,finally中对引用类型的修改是用影响的
  }
  
  public static StringBuilder test() {
      StringBuilder str = new StringBuilder("abc");
      try {
          System.out.println("try");
          return str;
      }finally {
          System.out.println("finally");
          str.append("def");
      }
  }
  ```

> 参考：
>
> https://www.bilibili.com/video/BV1yE411Z7AP?p=126 ~ https://www.bilibili.com/video/BV1yE411Z7AP?p=128

**finally 是不是一定会被执行到？**不一定。下面列举执行不到的情况：

- 当程序进入 try 块之前就出现异常时，会直接结束，不会执行 finally 块中的代码；

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

- 当程序在 try 块中强制退出时也不会去执行 finally 块中的代码，比如在 try 块中执行 **exit 方法**

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

- 程序所在的线程死亡

- 关闭 CPU

> 参考：https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

**try-catch-finally 中那个部分可以省略？**

1. catch 可以省略。**try 只适合处理运行时异常，try+catch 适合处理运行时异常+编译期异常**。也就是说，如果你只用 try 去处理编译期异常却不加以 catch 处理，编译是通不过的，因为编译器硬性规定，**编译期异常如果选择捕获，则必须用 catch 显示声明以便进一步处理**。而运行时异常在编译时没有如此规定，所以 catch 可以省略，你加上catch 编译器也觉得无可厚非。

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

### static 关键字 :exclamation:

1. **静态变量**：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过 `类名.静态变量名` 来访问它。静态变量在内存中只存在一份；

2. **静态方法**：静态方法在类加载的时候就存在的，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。其只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字；

   > 针对上述两点：
   >
   > 1. 被 static 修饰的静态变量与静态方法为静态资源，**静态资源是类加载时被加载的，而非静态资源是类new的时候加载的**；
   > 2. 静态方法不能引用非静态资源：非静态资源为new的时候才会产生的东西，对于初始化后就存在的静态资源来说，根本不认识它。
   > 3. 静态方法里面可以引用静态资源；
   > 4. 非静态方法里面可以引用静态资源：非静态方法就是实例方法，那是new之后才产生的，那么属于类的内容它都认识。

3. **静态内部类**：非静态内部类依赖于外部类的实例，**而静态内部类不需要**。当然静态内部类不能访问外部类的非静态的变量和方法；

   > 通过静态内部类还可以实现安全的单例模式

4. **静态代码块**：静态代码块在**类初始化时**运行一次；

   - **初始化顺序**：静态变量和静态代码块优先于实例变量和普通语句块，静态变量和静态代码块的初始化顺序取决于它们在代码中的顺序。

     ```java
     public class InitialOrderTest {
     
         public static String staticField = "静态变量";  // ——1
         public static String staticMethod = staticMethod();  // ——2
     
         {
             System.out.println("普通语句块");  // ——4:只有new的时候才会输出
         }
     
         static {
             System.out.println(staticField);
             System.out.println("静态语句块");  // ——3
         }
     
         public String field = "实例变量";
     
         public static String staticMethod(){
             System.out.println("静态变量调用静态方法");
             return "staticMethod";
         }
     
         // 最后才是构造函数的初始化
         public InitialOrderTest() {
             System.out.println("构造函数");    // ——5:只有new的时候才会输出
         }
     
         public static void main(String[] args) {
             new InitialOrderTest();
         }
     }
     // 结果：
     // 静态变量调用静态方法
     // 静态变量
     // 静态语句块
     // 普通语句块
     // 构造函数
     ```

     > 从字节码的角度对上述调用顺序进行解释：
     >
     > 1. `<cinit>()V`
     >
     >    编译器会按从上至下的顺序，收集所有 static 静态代码块和静态成员赋值的代码，**合并**为一个特殊的方法 `<cinit>()V` ： 
     >
     >    ```java
     >    // java源码：
     >    public class Demo3_8_1 {
     >        static int i = 10;
     >        static {
     >            i = 20;
     >        } 
     >        static {
     >            i = 30;
     >        }
     >    }
     >    
     >    // 对应的字节码：
     >    0: bipush 10    	// 把10放入栈中   
     >    2: putstatic #2 	// Field i:I，常量池中找到i变量
     >    5: bipush 20		// 把20放入栈中
     >    7: putstatic #2 	// Field i:I
     >    10: bipush 30		// 把20放入栈中
     >    12: putstatic #2 	// Field i:I
     >    15: return
     >    ```
     >
     >    `<cinit>()V` 方法会在类加载的初始化阶段被调用 
     >
     > 2. `<init>()V`
     >
     >    编译器会按从上至下的顺序，收集所有 `{} 代码块`和 `成员变量赋值` 的代码，形成新的构造方法，但原始构造方法内的代码总是在最后
     >
     >    ```java
     >    // java源码：
     >    public class Demo3_8_2 {
     >        private String a = "s1";
     >                 
     >        {
     >            b = 20;
     >        } 
     >                 
     >        private int b = 10;
     >                 
     >        {
     >            a = "s2";
     >        } 
     >                 
     >        public Demo3_8_2(String a, int b) {
     >            this.a = a;
     >            this.b = b;
     >        }
     >                 
     >        public static void main(String[] args){
     >            Demo3_8_2 d = new Demo3_8_2("s3", 30);
     >            System.out.println(d.a);  // "s3"
     >            System.out.println(d.b);  // 30
     >        }
     >    }
     >             
     >    // 对应的字节码：
     >    Code:
     >       stack=2, locals=3, args_size=3
     >           0: aload_0			 // this
     >           1: invokespecial #1   // super.<init>()V
     >           4: aload_0
     >           5: ldc #2 			 // <- "s1"
     >           7: putfield #3 		 // -> this.a
     >           10: aload_0
     >           11: bipush 20 		 // <- 20
     >           13: putfield #4 		 // -> this.b
     >           16: aload_0
     >           17: bipush 10		 // <- 10
     >           19: putfield #4		 // -> this.b
     >           22: aload_0
     >           23: ldc #5			 // <- "s2"
     >           25: putfield #3 		 // -> this.a
     >           28: aload_0 			 // ------------------------------
     >           29: aload_1 			 // <- slot_1(a) "s3" 			 |
     >           30: putfield #3		 // -> this.a 				     |
     >           33: aload_0			 //								 |
     >           34: iload_2			 // <- slot 2(b) 30 			 |
     >           35: putfield #4		 // -> this.b --------------------
     >           38: return
     >    ```
     >
     > > 参考：https://www.bilibili.com/video/BV1yE411Z7AP?p=116 ~ https://www.bilibili.com/video/BV1yE411Z7AP?p=117

   - 静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问

     ```java
     public class InitialOrderTest {
     
         static {
             // 不能访问，报错↓↓↓
             // System.out.println(staticField);
             // 可以赋值
             staticField = "?????";
             System.out.println("静态语句块");
         }
     
         public static String staticField;
     
         static {
             System.out.println(staticField);  // 结果：?????
         }
         
         public static void main(String[] args) {
             new InitialOrderTest();
         }
     }
     ```

     > 对上述代码查看静态代码块的字节码如下：
     >
     > ```java
     > {
     >  	// 类加载的链接中的准备阶段，为static变量分配空间，即staticField已经分配好了空间
     >  	public static java.lang.String staticField;  
     >    	descriptor: Ljava/lang/String;
     >    	flags: ACC_PUBLIC, ACC_STATIC
     > 
     >      public cn.xyc.other.InitialOrderTest();
     >      // ... 略
     >      public static void main(java.lang.String[]);
     >      // ... 略
     > 
     >  	// 在类的初始化阶段调用<cinit>()V完成对静态变量的赋值
     >      static {};
     >        descriptor: ()V
     >        flags: ACC_STATIC
     >        Code:
     >          stack=2, locals=0, args_size=0
     >             0: ldc           #4  // String ?????
     >             2: putstatic     #5  // Field staticField:Ljava/lang/String;
     >             5: getstatic     #6  // Field java/lang/System.out:Ljava/io/PrintStream;
     >             8: ldc           #7  // String 静态语句块
     >            10: invokevirtual #8  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
     >            13: getstatic     #6  // Field java/lang/System.out:Ljava/io/PrintStream;
     >            16: getstatic     #5  // Field staticField:Ljava/lang/String;
     >            19: invokevirtual #8  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
     >            22: return
     >          LineNumberTable:
     >            line 9: 0
     >            line 10: 5
     >            line 16: 13
     >            line 17: 22
     > }
     > SourceFile: "InitialOrderTest.java"
     > ```

   - 静态代码块是严格按照：父类静态代码块->子类静态代码块的顺序加载的，且只加载一次。

   > **对初始化顺序的补充：**存在继承的情况下，初始化顺序为：
   >
   > 1. 父类（静态变量、静态代码块）
   > 2. 子类（静态变量、静态代码块）
   > 3. 父类（实例变量、普通代码块）
   > 4. 父类（构造函数）
   > 5. 子类（实例变量、普通代码块）
   > 6. 子类（构造函数）

> 参考：
>
> https://www.cnblogs.com/xrq730/p/4820992.html
>
> https://www.cnblogs.com/GrimMjx/p/10105626.html

### super & transient 关键字

**super关键字**

1. 访问父类的构造函数：可以使用 `super()` 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。

2. 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

3. `this()` 和 `super()` 不能同时出现在一个构造函数里面，**因为 this(...) 必然会调用其它的构造函数，其它的构造函数必然也会有 super(...) 语句的存在**，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。

   > 简而言之：`super()` 和 `this()` 同时出现的话，会出现初始化父类两次的不安全操作

**transient 关键字**

* 对于**不想进行序列化的变量**，使用 transient 关键字修饰。

* transient 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化。当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。**transient 只能修饰变量**，不能修饰类和方法。

> 参考：https://www.cnblogs.com/wshuage/archive/2018/11/13/9955130.html

### Java 中的值传递 :exclamation::exclamation:

Java 的参数是以**值传递**的形式传入方法中，而不是引用传递；**Java 语言的参数传递只有值传递。**

当传递方法参数类型为**基本数据类型（数字以及布尔值）/ String类型**时，一个方法是不可能修改一个基本数据类型的参数。

```java
private static void changeNum(int num){
    num = 2;
}

public static void main(String[] args) {
    int num = 1;
    System.out.println("num在调用方法前:" + num);  // num在调用方法前:1
    changeNum(num);
    System.out.println("num在调用方法后:" + num);  // num在调用方法后:1
}
```

当传递方法参数类型为**引用数据类型**时，一个方法将修改一个引用数据类型的参数所指向对象的值。即 Java 函数在传递引用数据类型时，也只是拷贝了引用的值罢了，**之所以能修改引用数据是因为它们同时指向了一个对象**，但这仍然是按值调用而不是引用调用。

```java
// 传入的值为指向stringBuffer对象的地址，即str与stringBuffer指向同一块区域
private static void change(StringBuffer str){
    str.append(", world!");
}

public static void main(String[] args) {
    StringBuffer stringBuffer = new StringBuffer("hello");
    System.out.println("在调用方法前:" + stringBuffer);  // hello
    change(stringBuffer);
    System.out.println("在调用方法后:" + stringBuffer);  // hello, world!
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
    System.out.println("在调用方法后:" + string);  // Hello
}
```

> 参考：https://blog.csdn.net/qq_22408539/article/details/82945990

### & 和 && 的区别 :exclamation::exclamation:

Java 中 && 和 & 都是表示与的逻辑运算符，都表示逻辑运输符 and，当两边的表达式都为 true 的时候，整个运算结果才为 true，否则为 false。

`&&`：有短路功能，当第一个表达式的值为 false 的时候，**则不再计算第二个表达式**；

`&`：不管第一个表达式结果是否为 true，**第二个都会执行**。

除此之外，& 还可以用作位**运算符**：当 & 两边的表达式不是 Boolean 类型的时候，& 表示按位操作。

```java
int a = 1;  // 01
int b = 3;  // 11
int c = a & b;  // 01&11=01
System.out.println(c);  // 01
```

### 深克隆和浅克隆 :exclamation::exclamation::exclamation:

 **如何实现对象的克隆？**

1. 将A对象的值分别通过set方法加入B对象中；

   ```java
   Student stu1 = new Student();
   stu1.setName("张三");
   Student stu2 = new Student();
   stu2.setName(stu1.getName());
   ```

2. 实现 Cloneable 接口并重写 Object 类中的 `clone()` 方法；

   - **浅克隆**：

     - 需要复制的类实现 Cloneable 接口（该接口为标记接口，接口中无任何方法）
     - 覆盖 `clone()` 方法，方法中调用 `super.clone()` 方法得到需要的复制对象。

     ```java
     import java.util.Date;
     
     public class Student implements Cloneable {
         private String name;
         private Date birthday;
     
         // get/set...
     
         @Override
         protected Object clone() throws CloneNotSupportedException {
             return super.clone();
         }
     
         public static void main(String[] args) throws CloneNotSupportedException {
             // 测试
             Student stu1 = new Student();
             stu1.setName("张三");
             stu1.setBirthday(new Date());
             Student stu2 = (Student) stu1.clone();
             System.out.println(stu2.getName());
             System.out.println(stu2.getBirthday());
     
             // 被拷贝对象中引用对象
             System.out.println(stu1.getBirthday() == stu2.getBirthday());  // true
         }
     }
     ```
     
   - **深克隆**
   
     - 通过覆盖 Object 类的 `clone()` 方法可以实现深克隆，将 `clone()` 方法的修饰符修改为 public，总而言之：深克隆把要复制的对象所引用的对象都复制了一遍。
   
     ```java
     @Override
     public Object clone() throws CloneNotSupportedException {
         Student student = (Student) super.clone();
         student.setBirthday(new Date());  // 拷贝对象和原始对象的引用类型引用不同对象
         return student;
     }
     
     // 测试
     // 被拷贝对象中引用对象 
      System.out.println(stu1.getBirthday() == stu2.getBirthday());  // false
     ```

3. 实现 Serializable 接口，通过**对象的序列化和反序列化实现克隆**，可以实现真正的深克隆;

   - 序列化就是将对象写到流的过程，写到流中的对象是原有对象的一个拷贝，而原对象仍然存在于内存中。
   - 通过序列化实现的拷贝**不仅可以复制对象本身，而且可以复制其引用的成员对象**，因此通过序列化将对象写到一个流中，再从流里将其读出来，**可以实现深克隆**。

4. 工具类 BeanUtils 和 PropertyUtils 进行对象复制

   > 有待验证

**深克隆和浅克隆的区别**

1. 浅克隆：拷贝对象和原始对象的引用类型引用同一个对象。**浅克隆只是复制了对象的引用地址，两个对象指向同一个内存地址**，所以修改其中任意的值，另一个值都会随之变化，这就是浅克隆。

   ```java
   @Override  
   public Object clone() {  
       Student stu = null;  
       try{  
           stu = (Student)super.clone();   //浅复制  
       }catch(CloneNotSupportedException e) {  
           e.printStackTrace();  
       }   
       return stu;  
   }  
   ```

2. 深克隆：**拷贝对象和原始对象的引用类型引用不同对象**。深拷贝是将对象及值复制过来，两个对象修改其中任意的值另一个值不会改变，这就是深拷贝。

   ```java
   // 简单来说，在深克隆中，除了对象本身被复制外，对象所包含的所有成员变量也将复制。
   @Override  
   public Object clone() {  
       Student stu = null;  
       try{  
           stu = (Student)super.clone();   // 浅克隆
       }catch(CloneNotSupportedException e) {  
           e.printStackTrace();  
       }  
       stu.addr = (Address)addr.clone();   // 深克隆
       return stu;  
   }  
   ```

> **克隆的补充：**
>
> 深克隆的实现就是在引用类型所在的类实现 Cloneable 接口，**并使用 public 访问修饰符重写 clone 方法**。
>
> Java 中定义的 clone 没有深浅之分，都是统一的调用 Object 的 clone 方法。为什么会有深克隆的概念？是由于我们在实现的过程中刻意的嵌套了 clone 方法的调用。也就是说深克隆就是在需要克隆的对象类型的类中重新实现克隆方法 clone()。

> 参考：https://blog.csdn.net/ztchun/article/details/79110096

### 什么是 Java 的序列化 :exclamation::exclamation:

**概念：**

序列化：把Java对象转换为字节序列的过程。

反序列化：把字节序列恢复为Java对象的过程。

**解释：**

对象序列化是一个用于**将对象状态转换为字节流的过程**，可以将其保存到磁盘文件中或通过网络发送到任何其他程序。**从字节流创建对象**的相反的过程称为反序列化。而创建的字节流是与平台无关的，在一个平台上序列化的对象可以在不同的平台上反序列化。序列化是为了解决在对象流进行读写操作时所引发的问题。

序列化的实现：将需要被序列化的类实现 Serializable 接口，该接口没有需要实现的方法，只是用于标注该对象是可被序列化的，然后使用一个输出流（如：FileOutputStream）来构造一个 ObjectOutputStream 对象，接着使用 ObjectOutputStream 对象的 `writeObject(Object obj)` 方法可以将参数为 obj 的对象写出，要恢复的话则使用输入流。

**序列化/反序列化操作**

```java
public class Person implements Serializable{
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
            "name='" + name + '\'' +
            ", age=" + age +
            '}';
    }
}

// 序列化/反序列化对象
public class SerializableTest {

    public static void main(String[] args) throws Exception {

        Person person = new Person("王麻子", 40);

        // 序列化对象
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("./data.obj"));
        out.writeObject("你好!");      //写入字面值常量
        out.writeObject(new Date());   //写入匿名Date对象
        out.writeObject(person);       //写入person对象
        out.close();
        // 反序列化对象
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("./data.obj"));
        System.out.println("obj1:" + (String)in.readObject());  // 读取字面值常量
        System.out.println("obj2:" + (Date)in.readObject());  // 读取匿名Date对象
        System.out.println("obj3:" + (Person)in.readObject());  // 读取person对象
        in.close();
    }
}

// 结果：
// obj1:你好!
// obj2:Thu Aug 20 22:02:51 CST 2020
// obj3:Person{name='王麻子', age=40}
```

**什么情况下需要序列化？**

1. 当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；

2. 当你想用套接字在网络上传送对象的时候；

   - 当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以**二进制序列**的形式在网络上传送。发送方需要把这个 Java 对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为 Java 对象。
   - 只能将支持 `java.io.Serializable` 接口的对象写入流中。每个 serializable 对象的类都被编码，编码内容包括类名和类签名、对象的字段值和数组值，以及从初始对象中引用的其他所有**对象的闭包**。

3. 当你想通过 RMI 传输对象的时候。

   > Remote Method Invocation，远程方法调用
   >
   > https://www.cnblogs.com/grefr/p/5046313.html

> 参考：https://www.cnblogs.com/shoshana-kong/p/10538661.html

### wait/notify 方法在 Object 类而不是 Thread 类中

**一个很明显的原因是 Java 提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。**如果线程需要等待某些锁，那么调用对象中的 `wait()` 方法就有意义了。**如果 `wait()` 方法定义在 Thread 类中，线程正在等待的是哪个锁就不明显了**。简单的说，由于 wait，notify 和 notifyAll 都是锁级别的操作，所以把他们定义在 Object 类中因为锁属于对象。

> 参考：
>
> https://blog.csdn.net/pulusite/article/details/82287462
>
> https://blog.csdn.net/zl1zl2zl3/article/details/106695410

### Error 和 Exception/异常 相关内容:exclamation::exclamation:

首先，Java 提供了三种可抛出异常(Throwable Exception)：

- Exception
  - **受检查异常(Checked Exception)**
  - **运行时异常(Runtime Exception)**
- **错误(Error)**

**相同点：**

Exception 和 Error 都是继承了 **Throwable 类**，在 Java 中只有 Throwable 类型的实例才可以被抛出或者捕获，它是异常处理机制的基本类型。

**不同点：**

1. Exception：是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。
2. Exception 又分为 **受检查(checked)异常** 和 **不可检查(unchecked)/运行时** 异常：
   1. **可检查异常(Checked Exception)**，编译期异常，编译的时候已经发现了有可能的错误情况，**必须显式的进行捕获处理**，这是**编译期检查的一部分**，要么用 try … catch… 捕获，要么用 throws 声明抛出，交给父类处理。
   2. **不可检查时异常是指运行时异常(Runtime Exception)**，像 NullPointerException、ArrayIndexOutOfBoundsException 之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，**不会在编译期强制要求处理**，即：**可以编译通过，但是一运行就停止了，程序不会自己处理；**
3. Error：一般是指与虚拟机相关的问题，如：系统崩溃、虚拟机错误、内存空间不足、方法调用栈溢出等。这类错误将会导致应用程序中断，仅靠程序本身无法恢复和预防；

**类图/常见的异常类：**

Throwable：

- **Error**：IOError、LinkageError、ReflectionError、ThreadDeath、VirtualMachineError、OutOfMemoryError、StackOverflowError
- **Exception**
  - CloneNotSupportedException：在没有实现`Cloneable` 接口的实例上调用 Object 的 clone 方法
  - **DataFormatException**：格式转换的异常
  - **InterruptedException**：当阻塞方法收到中断请求的时候就会抛出InterruptedException异常
  - **IOException**：当发生某种 I/O 异常时，抛出此异常。此类是失败或中断的 I/O 操作生成的异常的通用类
  - ReflectiveOperationException：与反射有关的异常
  - **RuntimeException：运行时异常**
    - ArithmeticException：当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例；
    - ClassCastException：当试图将对象强制转换为不是实例的子类时，抛出该异常；
    - **ConcurrentModificationException**；并发修改异常，一般在多线程中，对集合类进行并发的修改时会抛出该异常；
    - **IllegalArgumentException**：抛出的异常表明向方法传递了一个不合法或不正确的参数；
    - **IndexOutOfBoundsException**：指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出；
    - **NoSuchElementException**：出现这个异常的原因之一是因为线程访问越界，访问到了不存在的元素
    - **NullPointerException**：当应用程序试图访问空对象时，则抛出该异常；
    - SecurityException：由安全管理器抛出的异常，指示存在安全侵犯；
  - SQLException：提供关于数据库访问错误或其他错误信息的异常

**throw 和 throws 的区别**

* **throw：**
  1. **作用在方法内**，表示抛出具体异常，由方法体内的语句处理。
  2. **具体向外抛出的动作**，所以它抛出的是一个异常实体类。若执行了throw一定是抛出了某种异常。

* **throws：**

  1. **用在方法的声明上**，表示如果抛出异常，则由该方法的调用者来进行异常处理。

  2. 主要的声明这个方法会抛出某种类型的异常，让它的使用者知道捕获异常的类型。

  3. **出现异常是一种可能性**，但不一定会发生异常。

* **实例：**

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

> 参考：
>
> https://blog.csdn.net/meism5/article/details/90414205
>
> https://www.cnblogs.com/hongwei2085/p/8858540.html
>
> https://www.cnblogs.com/baxianhua/p/9162103.html
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA
>
> https://blog.csdn.net/qq_38080370/article/details/87880511

### 主线程捕获子线程的异常 :exclamation:

**线程设计的理念：“线程的问题应该线程自己本身来解决，而不要委托到外部”**

**正常情况下，如果不做特殊的处理，在主线程中是不能够捕获到子线程中的异常的**，如下：

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
// 输出： 并未输出"==========="，因此未捕获到子线程的异常
// Exception in thread "Thread-0" java.lang.RuntimeException: RuntimeException!!!
// 	at cn.xyc.ThreadingTest.ThreadExceptionRunner.run(ThreadExceptionRunner.java:8)
// 	at java.lang.Thread.run(Thread.java:748)
// over
```

**想要在主线程中捕获子线程的异常：(没有深刻体会...)**

**方式一：**使用ExecutorService

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

> 参考：https://blog.csdn.net/wild46cat/article/details/80808555

### Java 的泛型与类型擦除 :exclamation::exclamation:

**开篇：**

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();		
System.out.println(l1.getClass() == l2.getClass());  // 存在类型擦除：true
```

泛型是通过类型擦除来实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息。

例如：`List<String>` 在运行时仅用一个 List 来表示。这样做的目的，是确保能和 Java 5 之前的版本开发二进制类库进行兼容。

**类型擦除：**

泛型信息只存在于代码编译阶段，**在进入 JVM 之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除**。

在泛型类被类型擦除的时候，之前泛型类中的类型参数部分如果没有指定上限，如 `<T>` 则会被转译成普通的 Object 类型，如果指定了上限如 `<T extends String>` 则类型参数就被替换成类型上限。

**泛型中值得注意的地方：**

1. 泛型类或者泛型方法中，不接受 8 种基本数据类型

   ```java
   // 无法通过编译
   List<int> li1 = new ArrayList<>();
   List<boolean> li2 = new ArrayList<>();
   // 需要转换成其包装类
   List<Integer> li1 = new ArrayList<>();
   List<Boolean> li2 = new ArrayList<>();
   ```

2. Java 不能创建具体类型的泛型数组

   ```java
   // 无法通过编译
   List<Integer>[] li3 = new ArrayList<Integer>[];
   List<Boolean>[] li4 = new ArrayList<Boolean>[];
   ```

   原因还是类型擦除带来的影响，`List<Integer>`和 `List<Boolean>`在 JVM 中等同于`List<Object>`, 所有的类型信息都被擦除, 程序也无法分辨一个数组中的元素类型具体是 `List<Integer>`类型还是 `List<Boolean>`类型。

> **补充：**
>
> ```java
> List<String> list = new ArrayList<String>();
> ```
>
> 1. 两个 String 其实只有第一个起作用，后面一个没什么卵用，只不过 JDK7 才开始支持 `List<String>list = new ArrayList<>` 这种写法。
>
> 2. 第一个 String 就是告诉编译器，List 中存储的是 String 对象，也就是**起类型检查的作用**，之后编译器会擦除泛型占位符，以保证兼容以前的代码。

> 参考：https://blog.csdn.net/briblue/article/details/76736356

### 泛型中的限定通配符和非限定通配符 `?`

**通配符 `？`**：除了用 `<T>`表示泛型外，还有 `<?>`这种形式。**`?` 被称为通配符**。

> 为何需要引入通配符 `?`的概念：
>
> ```java
> class Base{}
> class Sub extends Base{}
> 
> // 以下代码无问题：
> Sub sub = new Sub();
> Base base = sub;  // 父类的引用指向子类对象
> List<Sub> lsub = new ArrayList<>();
> // error! 无法编译通过
> List<Base> lbase = lsub;
> ```
>
> 编译器不会让它通过的。Sub 是 Base 的子类，不代表 `List<Sub>`和 `List<Base>`有继承关系。
>
> 但是，在现实编码中，确实有这样的需求，**希望泛型能够处理某一范围内的数据类型**，比如某个类和它的子类，对此 Java 引入了通配符这个概念。
>
> 因此，**通配符的出现是为了指定泛型中的类型范围**。

通配符的三种形式：

1. `<?>`被称作无限定的通配符。
2. `<? extends T>`被称作有上限的通配符。
3. `<? super T>`被称作有下限的通配符。

**无限定的通配符`？`**：它其中的 `?` 其实代表的是未知类型，所以涉及到 `?` 时的操作，一定与具体类型无关；

**有上限的通配符`<? extends T>`**：确保类型必须是 T 的子类来设定类型的上界，例如 `List<? extends Number>` 可以接受 `List<Integer>` 或 `List<Float>`。

**有下限的通配符`<? super T>`**：它通过确保类型必须是 T 的父类来设定类型的下界

```java
// 定义类
class Person{
    private String name;
    // 构造 / toString
}

class Student extends Person{
	// 构造
}

class Animal{
    private String name;
    // 构造 / toString
}

@Test
public void test06(){
    ArrayList<Person> persons = new ArrayList<>();
    persons.add(new Person("张三"));
    persons.add(new Person("李四"));
    persons.add(new Person("王五"));
    printOnlyPerson(persons);
    printPerson(persons);
    printWho(persons);
    
    ArrayList<Student> students = new ArrayList<Student>();
    students.add(new Student("小红"));
    students.add(new Student("小明"));
    students.add(new Student("小丁"));
    // printOnlyPerson(students);  // error
    printPerson(persons);
    printWho(persons);
    
    ArrayList<Animal> animal = new ArrayList<Animal>();
    animal.add(new Animal("小猫"));
    animal.add(new Animal("小狗"));
    animal.add(new Animal("小鸟"));
    // printOnlyPerson(animal);  // error
    // printPerson(animal);      // error
    printWho(animal);
}

// 只能打印限定Person类别
public void printOnlyPerson(ArrayList<Person> persons){
    Iterator<Person> iterator = persons.iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next().toString());
    }
}

// 可以打印Person类及其子类
public void printPerson(ArrayList<? extends Person> persons){
    Iterator<? extends Person> iterator = persons.iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next().toString());
    }
}

// 可以打印任何东西
public void printWho(ArrayList<?> who){
    Iterator<?> iterator = who.iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next().toString());
    }
}
```

> **补充：**Array 不支持泛型，要用 List 代替 Array，因为 List 可以提供编译器的类型安全保证，而 Array 却不能。

> 参考：https://blog.csdn.net/briblue/article/details/76736356

### Java 中的反射 :exclamation::exclamation:

**Java中反射的意思：**

Java 反射指的是在 Java 程序运行状态中，对于任何一个类，都可以获得这个类的所有属性和方法；对于给定的一个对象，都能够调用它的任意一个属性和方法。这种**动态获取类的内容以及动态调用对象的方法称为反射机制**。

**详解：**

首先，**每个类都有一个 Class 对象**，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。类加载相当于 Class 对象的加载，**类在第一次使用时才动态加载到 JVM 中**。也可以使用 `Class.forName("全限定类名")` 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，**并且这个类可以在运行时才加载进来**，甚至在编译时期该类的 .class 不存在也可以加载进来。

**`Class类` 和 `java.lang.reflect` 一起对反射提供了支持**，java.lang.reflect 类库主要包含了以下三个类：

1. **Field** ：可以使用 `get()` 和 `set()` 方法读取和修改 Field 对象关联的字段；

   ```java
   // 定义类
   class Person{
       private String name;
   
       // 构造/toString() ...
   }
   
   @Test
   public void test09() throws Exception{
       Person person = new Person("李二甩");
       System.out.println(person);
       Class<? extends Person> c = person.getClass();  // 获得Person类对象
   
       // 定义要修改的属性
       Field name = c.getDeclaredField("name");
       // 暴力反射
       name.setAccessible(true);
       // 修改属性，传入要设置的对象和值
       name.set(person, "李三甩");
       System.out.println(person);  // Person{name='李三甩'}
   }
   ```
   
2. **Method** ：可以使用 `invoke()` 方法调用与 Method 对象关联的方法；

   ```java
   class Cat{
       public void speak(String str){
           System.out.println(str);
       }
   
       public void run(int i){
           System.out.println(i + "s");
       }
   }
   
   @Test
   public void test10() throws Exception{
       Cat cat  = new Cat();
       Class<? extends Cat> c = cat.getClass();
   
       // getMethod()方法需要传入方法名，和参数类型
       Method speak = c.getMethod("speak", String.class);
       // invoke()表示调用的意思，需要传入对象和参数
       speak.invoke(cat, "喵喵");
       
   	// getMethod()方法需要传入方法名，和参数类型
       Method run = c.getMethod("run", int.class);
       run.invoke(cat,3600);
   }
   ```

3. **Constructor** ：可以用 Constructor 创建新的对象。

   ```java
   @Test
   public void test11() throws Exception{
       Class<?> c = Class.forName("java.lang.Boolean");
       // 获取构造函数数组
       // Constructor<?>[] constructors = c.getConstructors();
       // 获取构造函数
       Constructor<?> constructor = c.getConstructor(String.class);
       Boolean bool = (Boolean) constructor.newInstance("true");
       System.out.println(bool);
   }
   ```

**应用举例**：工厂模式，使用反射机制，根据全限定类名获得某个类的 Class 实例。

**反射的优缺点？**

* **优点：**运行期类型的判断，`class.forName()` 动态加载类，提高代码的灵活度；

* **缺点：**尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。
1. **性能开销 ：**反射涉及了动态类型的解析，**所以 JVM 无法对这些代码进行优化**。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
  
2. 安全限制 ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
  
3. **内部暴露：**由于反射允许代码执行一些在正常情况下不被允许的操作（比如：访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

> 参考：
>
> https://www.cnblogs.com/nerxious/archive/2012/12/24/2829446.html
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

### Java 动态代理概述 :exclamation::exclamation::exclamation:

**动态代理：**当想要给实现了某个接口的类中的方法，加一些额外的处理。比如说加日志，加事务等。可以给这个类创建一个代理，故名思议就是创建一个新的类，这个类不仅包含原来类方法的功能，而且还在原来的基础上添加了额外处理的新功能。**这个代理类并不是定义好的，是动态生成的**。具有解耦意义，灵活，扩展性强。

**Java领域中，常用的动态代理实现方式有两种**

* JDK 原生动态代理：利用**JDK反射机制**生成代理;

  > 动态代理类和被代理类必须继承同一个接口：之所以实现相同的接口，目的是为了**尽可能保证代理对象内部结构和目标对象一致**，这样我们对代理对象的操作最终都可以转移到目标对象身上，代理对象只需专注增强代码的编写。

* 另外一种是使用**CGLIB（Code Generation Library）代理**

  > 由于 JDK 动态代理存在限制：只能为接口创建代理实例，而**对于没有实现接口的类，就可以通过 CGLIB 来实现动态代理实例**
  >
  > **首先需要引入CGLIB相关Jar包**：其实针对类实现代理，**主要对指定的类生成一个子类，覆盖其中的方法**，添加额外功能，因为是继承，所以该类方法不能用 final 来声明

**动态代理的应用：**Spring 的 AOP 、加事务、加权限、加日志。

> 参考：
>
> https://blog.csdn.net/woshichuanqihan/article/details/52205137
>
> bravo1988的回答：https://www.zhihu.com/question/20794107

### 实现动态代理 :exclamation::exclamation:

**JDK 原生动态代理：动态代理类和被代理类必须继承同一个接口。动态代理只能对接口中声明的方法进行代理。**

> 之所以实现相同的接口，目的是为了尽可能保证代理对象内部结构和目标对象一致，这样我们对代理对象的操作最终都可以转移到目标对象身上，**代理对象只需专注增强代码的编写**。

1. **Proxy**

   `java.lang.reflect.Proxy` 是所有动态代理的父类。它通过静态方法 `newProxyInstance()` 来创建动态代理的 class 对象和实例；

   ```java
   public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler handler)  throws IllegalArgumentException
   
   // loader：一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载；
   
   // interfaces：一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了 
   
   // handler：一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上。
   
   // 通过Proxy.newProxyInstance创建的代理对象是在Jvm运行时动态生成的一个对象，它并不是我们的InvocationHandler类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象。
   ```
   
2. **InvocationHandler**

   每一个动态代理实例都有一个关联的InvocationHandler。通过代理实例调用方法，方法调用请求会被转发给InvocationHandler的invoke方法，而在调用invoke 方法的前后去做一些额外的事情，从而实现动态代理。

   ```java
   public interface InvocationHandler { 
   	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable; 
   }
   // proxy: 指代我们所代理的那个真实对象
   // method: 指代的是我们所要调用真实对象的某个方法的 Method 对象
   // args: 指代的是调用真实对象某个方法时接受的参数
   ```

3. 举例

   > ```java
   > /* 动态代理实现方式小案例：*/
   > 
   > // 定义一个卖电脑的接口：
   > package cn.itcast.proxy;
   > public interface SaleComputer {
   >        public String sale(double money);
   >        public void show();
   > }
   > 
   > // 实现该接口，定义一个真实类
   > package cn.itcast.proxy;
   > 
   > public class Lenovo implements SaleComputer {
   >        @Override
   >        public String sale(double money) {
   >            System.out.println("花了" + money + "元买了一台联想电脑...");
   >            return "联想电脑";
   >        }
   >        @Override
   >        public void show() {
   >            System.out.println("展示电脑....");
   >        }
   > }
   > 
   > package cn.itcast.proxy;
   > import java.lang.reflect.InvocationHandler;
   > import java.lang.reflect.Method;
   > import java.lang.reflect.Proxy;
   > 
   > public class ProxyTest {
   > 
   >        public static void main(String[] args) {
   >            // 1.创建真实对象
   >            Lenovo lenovo = new Lenovo();
   > 
   >            // 2.动态代理增强lenovo对象
   >            /*
   > 			三个参数：
   > 				1. 类加载器：真实对象.getClass().getClassLoader()
   > 				2. 接口数组：真实对象.getClass().getInterfaces()
   > 				3. 处理器：new InvocationHandler()
   >          */
   >         SaleComputer proxy_lenovo = (SaleComputer) Proxy.newProxyInstance(lenovo.getClass().getClassLoader(), lenovo.getClass().getInterfaces(), new InvocationHandler() {
   >             /*
   >          	代理逻辑编写的方法：代理对象调用的所有方法都会触发该方法执行
   >          		参数：
   >                 	1. proxy:代理对象
   >                 	2. method：代理对象调用的方法，被封装为的对象
   >                 	3. args:代理对象调用的方法时，传递的实际参数
   >          */
   >             @Override
   >             public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   >                 /*
   >              	System.out.println("该方法执行了....");
   >                  System.out.println(method.getName());
   >                  System.out.println(args[0]);
   >                  */
   >                 //判断是否是sale方法
   >                 if(method.getName().equals("sale")){
   >                     //1.增强参数
   >                     double money = (double) args[0];
   >                     money = money * 0.85;
   >                     System.out.println("专车接你....");
   >                     //使用真实对象调用该方法
   >                     String obj = (String) method.invoke(lenovo, money);
   >                     System.out.println("免费送货...");
   >                     //2.增强返回值
   >                     return obj+"_鼠标垫";
   >                 }else{
   >                     Object obj = method.invoke(lenovo, args);
   >                     return obj;
   >                 }
   >             }
   >         });
   > 
   >         // 3.调用方法
   >         String computer = proxy_lenovo.sale(8000);
   >         System.out.println(computer);
   >         proxy_lenovo.show();
   >     }
   > }
   > ```

**CGLIB（Code Generation Library）代理**

1. 导入 jar 包：`asm-5.2.jar`，`cglib-3.2.5.jar`

   > ```xml
   > <dependencies>
   >     <!-- https://mvnrepository.com/artifact/org.ow2.asm/asm -->
   >     <dependency>
   >         <groupId>org.ow2.asm</groupId>
   >         <artifactId>asm</artifactId>
   >         <version>5.2</version>
   >     </dependency>
   > 
   >     <!-- https://mvnrepository.com/artifact/cglib/cglib -->
   >     <dependency>
   >         <groupId>cglib</groupId>
   >         <artifactId>cglib</artifactId>
   >         <version>3.2.5</version>
   >     </dependency>
   > 
   > </dependencies>
   > ```

2. 举例

   > ```java
   > import net.sf.cglib.proxy.Enhancer;
   > import net.sf.cglib.proxy.MethodInterceptor;
   > import net.sf.cglib.proxy.MethodProxy;
   > 
   > import java.lang.reflect.Method;
   > 
   > public class CglibProxy implements MethodInterceptor {
   > 
   >     // 需要代理的目标对象
   >     private Object target;
   > 
   >     // 重写拦截方法
   >     @Override
   >     public Object intercept(Object obj, Method method, Object[] arr, MethodProxy proxy) throws Throwable {
   >         System.out.println("Cglib动态代理，监听开始！");
   >         Object invoke = method.invoke(target, arr);//方法执行，参数：target 目标对象 arr参数数组
   >         System.out.println("Cglib动态代理，监听结束！");
   >         return invoke;
   >     }
   > 
   >     // 定义获取代理对象方法
   >     public Object getCglibProxy(Object objectTarget){
   >         // 为目标对象target赋值
   >         this.target = objectTarget;
   >         Enhancer enhancer = new Enhancer();
   >         // 设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
   >         enhancer.setSuperclass(objectTarget.getClass());
   >         // 设置回调
   >         enhancer.setCallback(this);
   >         // 创建并返回代理对象
   >         Object result = enhancer.create();
   >         return result;
   >     }
   > 
   >     public static void main(String[] args) {
   >         // 实例化CglibProxy对象
   >         CglibProxy cglib = new CglibProxy();
   >         Lenovo lenovo =  (Lenovo) cglib.getCglibProxy(new Lenovo());//获取代理对象
   >         lenovo.sale(10000);
   >     }
   > }
   > 
   > // 输出：
   > // Cglib动态代理，监听开始！
   > // 花了10000.0元买了一台联想电脑...
   > // Cglib动态代理，监听结束！
   > ```

> 参考：
>
> https://www.cnblogs.com/ynzj123/p/12921288.html
>
> https://blog.csdn.net/qq_42807370/article/details/87354522

### Java 中的 IO 流 :exclamation:

**按功能来分**：输入流（input）、输出流（output）。

**按类型来分**：字节流 和 字符流。

* 字节流按 8 位传输，以字节为单位输入输出数据；
* 字符流按 16 位传输，以字符为单位输入输出数据；
* 但是不管文件读写还是网络发送接收，信息的最小存储单元都是字节。

**按流的角色来份**：节点流 和 处理流。

* 节点流：可以从或向一个特定的地方（节点）读写数据。如 FileReader；
* 处理流：是**对一个已存在的流的链接和封装**，通过所封装的流的功能调用实现数据读写。如 BufferedReader处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

> 参考：https://www.cnblogs.com/wxgblogs/p/5647415.html
>

**字节流**：InputStream/OutputStream 是字节流的抽象类，这两个抽象类又派生了若干子类，不同的子类分别处理不同的操作类型。具体子类如下所示：

* InputStream
  * FileInputStream
  * FilterInputStream
    * BufferedInputStream
    * DataInputStream
    * PushBackInputStream
  * PipeInputStream
  * SequenceInputStream
  * ByteArrayInputStream
  * ObjectInputStream
* OutputStream
  * FileOutputStream
  * FilterOutputStream
    - BufferedOutputStream
    - DataOutputStream
    - PrintStream
  * PipeOutputStream
  * ByteArrayOutputStream
  * ObjectInputStream

**字符流**：Reader/Writer 是字符的抽象类，这两个抽象类也派生了若干子类，不同的子类分别处理不同的操作类型。

* Reader
  * FileReader
  * StringReader
  * PipedReader
  * CharArrayReader
  * InputStreamReader
* Writer
  * FileWriter
  * StringWirter
  * PipedWriter
  * CharArrayWriter
  * OutputStreamWirter

> 参考：
>
> https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA

### BIO & NIO & AIO :exclamation::exclamation:

**BIO、NIO、AIO：**

- **Java BIO (Blocking I/O)** ： 同步阻塞IO，当线程调用 `read()` 或 `write()` 时，该线程被阻塞，直到有一些数据被读取或写入，**该线程在此期间不能执行其他任务**。由于其存在阻塞，因此服务器实现模式为**一个连接一个线程**，即客户端有连接请求时服务器端就需要启动一个线程进行处理。

- **Java NIO (Non-blocking/New I/O)**： 同步非阻塞IO，服务器实现模式为**一个请求一个线程**，即客户端发送的连接请求都会注册到**多路复用器（Selector）上**，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。因此，NIO 可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端。

- **Java AIO(NIO.2 / Asynchronous I/O)** ： 异步非阻塞IO，服务器实现模式为**一个有效请求一个线程**，客户端的 I/O 请求都是由 OS 先完成了再通知服务器应用去启动线程进行处理。

  > 与 NIO 不同，当进行读写操作时，只须直接调用 API 的 read 或 write 方法即可，**这两种方法均为异步的**，对于读操作而言，当有流可读取时，操作系统会将可读的流传入 read 方法的缓冲区；对于写操作而言，当操作系统将 write 方法传递的流写入完毕时，操作系统主动通知应用程序；即可以理解为，read/write 方法都是异步的，完成后会主动调用回调函数。在 JDK1.7 中，这部分内容被称作NIO.2，主要在 `Java.nio.channels` 包下增加了下面四个异步通道：
  >
  > - AsynchronousSocketChannel/AsynchronousServerSocketChannel
  > - AsynchronousFileChannel
  > - AsynchronousDatagramChannel

- | BIO          | NIO                 | AIO                             |
  | ------------ | ------------------- | ------------------------------- |
  | Socket       | SocketChannel       | AsynchronousSocketChannel       |
  | ServerSocket | ServerSocketChannel | AsynchronousServerSocketChannel |

**BIO、NIO、AIO适用场景分析:**

- BIO 方式适用于**连接数目比较小且固定的架构**，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。
- NIO 方式适用于**连接数目多且连接比较短（轻操作）的架构**，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。
- AIO 方式使用于**连接数目多且连接比较长（重操作）的架构**，比如相册服务器，**充分调用OS参与并发操作**，编程比较复杂，JDK7 开始支持。

**BIO、NIO比较：**

- BIO 以**流的方式**处理数据(面向字节流)，而 NIO 以**块的方式**处理数据块(面向缓冲区)， I/O 的效率比流 I/O 高很多
- BIO 是**阻塞**的，NIO 则是**非阻塞**的。
- BIO 基于**字节流和字符流进行操作**，而 NIO 基于 **Channel（通道）和 Buffer（缓冲区）进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道
- BIO图示：![JavaBIO](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808335.png)
- NIO图示：![JavaNIO](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808160.png)

### 谈谈对 NIO 的理解 :exclamation::exclamation:

概述：

- NIO 支持面**向缓冲区**的、基于**通道**的 IO 操作，其以更加高效的方式进行文件的读写操作，即非阻塞 IO；
- **Java NIO系统的核心在于：通道（Channel）+ 缓冲区（Buffer）+ 选择器（Selector），Channel 负责传输， Buffer 负责存储，而 Selector 可使一个单独的线程管理多个Channel。Selector 是非阻塞 IO 的核心。**
- ![Selector架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808662.png)
  - 每个 Channel 都会对应一个 Buffer
  - 一个线程对应 Selector ， 一个 Selector 对应多个 Channel 
  - 程序切换到哪个 Channel 是**由事件决定**的
  - Selector 会根据不同的事件，在各个通道上切换
  - Buffer 就是一个内存块，底层是一个数组
  - 数据的读取写入是通过 Buffer 完成的，且 NIO 的 Buffer是可以读也可以写是双向的。

**NIO核心之一：通道（Channel）**

- 通道表示打开到 IO 设备的连接，类似于传统的“流”。只不过 **Channel本身不能直接访问数据，Channel 只能与Buffer 进行交互**；
- 数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中，这是双向的（同时进行读写）；
- 通道一般会注册到选择器上，当然需要将通道配置为非阻塞的模式。

**NIO核心之一：缓冲区（Buffer）**

- 本质上是一块可以写入数据，然后可以从中读取数据的内存，主要用于与通道进行交互，**数据是从通道读入缓冲区，从缓冲区写入通道中的**。

- Buffer就像一个数组，可以保存多个相同类型的数据：ByteBuffer/CharBuffer/IntBuffer...

- 获取Buffer对象：`static XxxBuffer allocate(int capacity)`，创建一个容量为 capacity 的 XxxBuffer 对象

- **缓冲区的基本属性**

  - **容量（capacity）**：表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。
  - **限制（limit）**：第一个不应该读取或写入的数据的索引，**即位于limit后的数据不可读写**。缓冲区的limit不能为负，并且不能大于其容量。
  - **位置（position）**：下一个要读取或写入的数据的索引。缓冲区的 position 不能为负，并且不能大于其限制（limit）。
  - **标记（mark）与重置（reset）**：标记是一个索引，通过Buffer中的`mark()`方法指定Buffer中一个特定的position，之后可以通过调用`reset()`方法恢复到这个 position。
  - **标记、位置、限制、容量遵守以下不变式：**`0<=mark<=position<=limit<=capacity`

  图示：![缓冲区的基本属性](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808416.PNG)

- **通道+缓冲区的读取数据示例：**

  ```java
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
      // 先清空缓冲然后再写入数据到缓冲区
      buffer.clear();
  }
  
  // 4.关闭相应资源
  outChannel.close();
  inChannel.close();
  fos.close();
  fis.close();
  ```

  > 更多示例见：[Java：NIO 学习笔记-1：通道和缓冲区-代码演示](https://www.cnblogs.com/zhuchengchao/p/14523180.html)

**NIO核心之一：选择器**

- **Selector 可使一个单独的线程管理多个Channel，并确定哪些Channel已经准备好进行读取或写入。Selector 是非阻塞 IO 的核心**

- 即：**Selector 用于监听多个通道的事件**（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道，只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程

  > 可以监听的事件类型，用SelectionKey表示：读-`SelectionKey.OP_READ`；写-`SelectionKey.OP_WRITE` ；连接-`SelectionKey.OP_CONNECT`；接收-`SelectionKey.OP_ACCEPT`  

- 使用：非阻塞式Socket通信-TCP

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
  
      // 5. 关闭通道
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
      // 4. 获取选择器，打开选择器
      Selector selector = Selector.open();
      // 5. 将通道注册到选择器上，并且指定“监听接收事件”
      serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
  
      // 6.轮寻式的获取选择器上已经“准备就绪”的事件，该方法会一直阻塞直到有至少一个事件到达
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

### JDK 8 新特性 :exclamation:

- **Lambda 表达式**

- **方法引用**

- **函数式接口**

- **接口的默认方法和静态方法**

- **Stream API**

- **HashMap / ConcurrentHashMap 变化**

  > 具体在集合笔记中查看即可：
  >
  > 1. HashMap：JDK7 中 采用头插法，JDK 8 中采用尾插法，头插容易导致HashMap链表死循环
  > 2. HashMap：JDK8 中将 链表（Entry数组 + 链表） 方式修改成 链表+红黑树（Node数组 + 链表/红黑树） 的形式
  > 3. ConcurrentHashMap：
  >    - JDK 7：中 ConcurrentHashMap 采用了 **数组 + Segment + 分段锁** 的方式实现。
  >    - JDK 8：中 ConcurrentHashMap 参考了 JDK8 HashMap 的实现，采用了 **数组 + 链表/红黑树** 的实现方式来设计，**并发控制使用 synchronized 和 CAS 来操作，并取消了分段锁 Segment**

- **JVM 新特性**：永久代(PermGen)替换为元空间(MetaSpace)，且元空间不存在于 JVM 中，不再由 JVM 对其进行管理，元空间使用的是直接内存

**Lambda表达式**

- Lambda表达式**允许把函数作为一个方法的参数**

  > 事实上，把Lambda表达式可以看做是**匿名内部类的一种简写方式**。
  >
  > 当然，前提是这个匿名内部类对应的必须是接口，而且**接口中必须只有一个函数**
  >
  > Lambda 表达式就是直接编写函数的：参数列表、代码体、返回值等信息，**用函数来代替完整的匿名内部类**

- 基本语法：

  ```java
  (参数列表) -> {代码块}
  ```

  - 参数类型可省略，编译器可以自己推断
  - 如果只有一个参数，圆括号可以省略
  - 代码块如果只是一行代码，大括号也可以省略
  - 如果代码块是一行，且是有结果的表达式，`return`可以省略

- **用法示例**

  ```java
  // 1.不需要参数,返回值为 5  
  ()->5
  
  // 2.接收一个参数(数字类型),返回其2倍的值  
  x -> 2 * x
  
  // 3. 接受2个参数(数字),并返回他们的差值  
  (x, y) -> x – y  
  
  // 4. 接收2个int型整数,返回他们的和  
  (int x, int y) -> x + y  
  
  // 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
  (String s) -> System.out.print(s)
  ```

**方法引用**

- 方法引用使得开发者可以**将已经存在的方法作为变量来传递使用**。方法引用可以和Lambda表达式配合使用。

- 总共有四类方法引用

  | 语法                   | 描述                             |
  | ---------------------- | -------------------------------- |
  | 类名::静态方法名       | 类的静态方法的引用               |
  | 类名::非静态方法名     | 类的非静态方法的引用             |
  | 实例对象::非静态方法名 | 类的指定实例对象的非静态方法引用 |
  | 类名::new              | 类的构造方法引用                 |

**函数式接口**

- 函数式接口必须满足：**内部只有一个函数**

- JDK8 提供了一个特殊的注解`@FunctionalInterface`来显示地表明函数接口

  > 这是考虑到 函数接口的脆弱性，只要有人在接口里添加多一个方法，那么这个接口就不是函数接口了，就会导致编译失败。

- Function 类型接口：表示**有参数，有返回值的函数**。

  ```java
  @FunctionalInterface
  public interface Function<T, R> {
  	// 接收一个参数T，返回一个结果R
      R apply(T t);
  }
  ```

- Consumer系列接口：**消费者接口，不返回任何结果**。

  ```java
  @FunctionalInterface
  public interface Consumer<T> {
  	// 接收T类型参数，不返回结果
      void accept(T t);
  }
  ```

- Predicate系列接口：**系列参数不固定，但是返回的一定是boolean类型**

  ```java
  @FunctionalInterface
  public interface Predicate<T> {
  	// 接收T类型参数，返回boolean类型结果
      boolean test(T t);
  }
  ```

- Supplier系列：供应者，顾名思义，**只产出，不收取。所以不接受任何参数，返回T类型结果**。

  ```java
  @FunctionalInterface
  public interface Supplier<T> {
  	// 无需参数，返回一个T类型结果
      T get();
  }
  ```

**接口的默认方法和静态方法**

- JDK8 支持在接口中定义**默认方法**和**静态方法**
- 默认方法，使用default进行修饰
  - 默认方法使得开发者可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。
  - 默认方法和抽象方法之间的区别在于抽象方法需要实现，而默认方法不需要。接口提供的默认方法会被接口的实现类继承或者重写
- 静态方法：可以直接用接口调用这些静态方法
- 进一步：**默认方法允许在不打破现有继承体系的基础上改进接口**。该特性在官方库中的应用是：给`java.util.Collection`接口添加新方法，如`stream()`、`parallelStream()`、`forEach()`和`removeIf()`等等

**Streams**

- 新增的Stream API（`java.util.stream`）将真正的函数式编程风格引入了 Java 库中。这是目前为止最大的一次对 Java 库的完善，以便开发者能够写出更加有效、更加简洁和紧凑的代码。

- Steam API 极大得简化了集合操作，当然不只是集合

- 获取流的方式：

  - 所有的 Collection 集合都可以通过 stream 默认方法获取流；
  - Stream 接口的静态方法 of 可以获取数组对应的流。

- Stream 操作有两个基础的特征

  - **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同**流式风格（fluent style）**。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路(shortcircuiting)。
  - **内部迭代**： 以前对集合遍历都是通过Iterator或者增强for的方式，显式的在集合外部进行迭代，这叫做外部迭代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

- 流模型的操作很丰富，这里介绍一些常用的API。这些方法可以被分成两种：

  - ![常用方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251808979.PNG)

  - **延迟方法**：返回值类型仍然是 Stream 接口自身类型的方法，因此支持链式调用。（除了终结方法外，其余方法均为延迟方法。）

    > `Stream<T> filter(Predicate<? super T> predicate);`
    >
    > `<R> Stream<R> map(Function<? super T, ? extends R> mapper)`

  - **终结方法**：返回值类型不再是 Stream 接口自身类型的方法，因此不再支持类似 StringBuilder 那样的链式调用，常见的终结方法有：count、forEach、sum

    > `void forEach(Consumer<? super T> action);`

  > 流操作的一些代码：
  >
  > ```java
  > List<String> one = new ArrayList<>();  // 添加元素略
  > List<String> two = new ArrayList<>();  // 添加元素略
  > Stream<String> streamOne = one.stream()
  >     .filter(s -> s.length() == 3)  // 第一个队伍只要名字为3个字的成员姓名；
  >     .limit(3);                     // 第一个队伍筛选之后只要前3个人；
  > 
  > Stream<String> streamTwo = two.stream()
  >     .filter(s -> s.startsWith("张"))  // 第二个队伍只要姓张的成员姓名；
  >     .skip(2);                         // 第二个队伍筛选之后不要前2个人；
  > 
  > Stream.concat(streamOne,streamTwo)    // 将两个队伍合并为一个队伍；
  >     .map(Person::new)                 // 根据姓名创建Person对象；
  >     .forEach(System.out::println);    // 打印整个队伍的Person对象信息。
  > ```
  >
  > 关于流的更多操作见 [01语言基础笔记]()

### Java内部类：成员/局部/静态/匿名

内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。

**成员内部类**

1. **成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括 private 成员和 static 成员）**

2. 如果要访问外部类同名的成员，需要以下面的形式进行访问：
   - 外部类.this.成员变量
   - 外部类.this.成员方法
   
3. 在外部类中如果要访问内部类的成员，**必须先创建一个成员内部类对象**，再通过指向这个对象的引用来访问

4. 要创建成员内部类的对象，前提是必须存在一个外部类的对象：`Outer outer = new Outer(); Outer.Inner inner= outer.new Inner();`

5. 内部类可以拥有 private、protected、public 访问权限及包访问权限；

6. 成员内部类中不能存在任何static的变量和方法，可以定义常量。

   ```java
   public class Outer {
   
       private static int num1 = 10;
       private int num2 = 20;
       private String name = "Java";
   
       // 外部类普通方法
       public void outer_func1(){
           System.out.println("外部类Outer的普通方法:outer_func1");
       }
   
       // 外部类静态方法
       public static void outer_func2(){
           System.out.println("外部类Outer的普通方法:outer_func2");
       }
   
       // 内部类
       class Inner{
           private String name = "Gava";
           // 6. 成员内部类中不能存在任何static的变量和方法，可以定义常量。
           // static int number = 1;   // 编译错误
           static final int number = 20;  // 可以
   
           public void inner_func(){
               System.out.println("内部类Inner方法inner_func");
               // 1.成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和static成员）
               System.out.println(num1);
               System.out.println(num2);
               outer_func1();
               outer_func2();
   
               // 2. 如果要访问外部类同名的成员，需要以下面的形式进行访问：
               System.out.println(name);             // 访问内部类的name
               System.out.println(Outer.this.name);  // 访问外部类的成员
           }
       }
   
       public static void main(String[] args) {
           // 4.创建成员内部类的对象
           Outer outer = new Outer();
           Outer.Inner inner = outer.new Inner();
           // 3.在外部类中如果要访问内部类的成员
           inner.inner_func();
       }
   }
   ```

**局部内部类**

1. **在方法中定义的内部类称为局部内部类**；

2. **局部内部类就像是方法里面的一个局部变量一样**，是不能有 public、protected、private 以及 static 修饰的；

   > 可以用 final 修饰；
   >
   > 可以用 abstract 修饰，amazing

3. 可以访问当前代码块内的常量，和此外围类所有的成员。

4. 局部内部类只能在定义该内部类的方法内实例化，不可以在此方法外对其实例化。

5. 局部内部类对象不能使用该内部类所在方法的非 final 局部变量

   > 在 JDK1.8 中新增了 Effectively final 功能，即：
   >
   > 局部内部类和匿名内部类访问的局部变量必须由final修饰，java8 开始，可以不加 final 修饰符，**由系统默认添加**。java将这个功能称为：Effectively final 功能。
   >
   > > 参考：https://blog.csdn.net/sinat_26342009/article/details/45077723
   
   ```java
   // 局部内部类
   public class Outer{
   
       private static int num1 = 10;
       private int num2 = 20;
       private String name = "Java";
   
       // 定义外部类方法
       public void outer_func(int k){
           // 当前代码块内的常量
           final String name = "Gava";
           // 当前代码块内的变量
           // 由于Effectively final 下面代码等同于 final int num = 10;
           int num = 10;
           // num = 20;  // 编译不通过：由于在内部类中，调用了System.out.println(num);
   
           // 1.在方法中定义的内部类称为局部内部类；
           // 2.不能有public、protected、private以及static修饰的
           // public/protected/private/static class Inner{ // error
           // 可以用 final 和 abstract 修饰: final class Inner/abstract class Inner
           class Inner{
               // 不可以定义静态变量
               // static int i = 100;
               public void inner_func(){
                   // 3.可以访问当前代码块内的常量，和此外围类所有的成员。
                   // 访问外部类的变量，如果没有与内部类同名的变量，则可直接用变量名
                   System.out.println(num1);
                   System.out.println(num2);
                   System.out.println(name);
                   // 访问外部类与内部类同名的变量
                   System.out.println(Outer.this.name);
                   // 局部内部类和匿名内部类访问的局部变量必须由final修饰
                   System.out.println(num);
                   System.out.println(k);
               }
           }
   
           // 4.局部内部类只能在定义该内部类的方法内实例化
           Inner inner = new Inner();
           inner.inner_func();
       }
   
       public static void main(String[] args) {
           Outer outer = new Outer();
           outer.outer_func(10);
       }
   }
   ```

**静态内部类**

静态内部类是**不需要依赖于外部类的**，通常称为嵌套类（nested class）；

对于普通内部类对象，其隐含的保存了一个指向创建它的外围类对象的引用；然而当内部类用static修饰后变为静态内部类，就没有这个引用了，这就意味着：

1. 要创建嵌套类的对象，并不需要其外围类的对象；

2. 不能从嵌套类的对象中访问非静态的外围类对象。

   ```java
   // 静态内部类
   public class Outer{
       private static int num1 = 10;
       private int num2 = 20;
       private String name = "Java";
   
       // 外部类普通方法
       public void outer_func1(){
           System.out.println("外部类Outer的普通方法:outer_func1");
       }
   
       // 外部类静态方法
       public static void outer_func2(){
           System.out.println("外部类Outer的普通方法:outer_func2");
       }
   
       // 静态内部类可以用public、protected、private修饰
       // 静态内部类可以定义静态类或非静态内部类
       public static class Inner{
           static int num3 = 30;
           String name = "Gava";
   
           // 静态内部类里的静态方法
           static void inner_func1(){
               System.out.println("静态内部类的静态方法");
               // 静态内部类只能访问外部类的静态成员（静态变量、静态方法）
               System.out.println(num1);
               // System.out.println(num2);  //error
               // outer_func1();             //error
               outer_func2();
           }
   
           // 静态内部类非静态方法
           void inner_func2(){
               System.out.println("静态内部类的非静态方法");
               System.out.println(num1);
               System.out.println(num3);
               System.out.println(name); // 可以访问静态内部类中的非静态成员
           }
       }
   
       public static void main(String[] args) {
           // 静态内部类的静态方法
           Outer.Inner.inner_func1();
   
           // 静态内部类的非静态方法
           Inner inner = new Inner();
           inner.inner_func2();
       }
   }
   ```

**匿名内部类**

简单地说：**匿名内部类就是没有名字的内部类**。

什么情况下需要使用匿名内部类？如果满足下面的一些条件，使用匿名内部类是比较合适的：

1. 只用到类的一个实例。
2. 类在定义后马上用到。
3. 类非常小（sun推荐是在4行代码以下）
4. 给类命名并不会导致你的代码更容易被理解。

在使用匿名内部类时，要记住以下几个原则：

1. 匿名内部类不能有构造方法（**匿名内部类是唯一一种没有构造器的类**）
2. 匿名内部类不能定义任何静态成员、方法和类。（即不能用static进行修饰）
3. 匿名内部类不能是 public,protected,private,static。
4. 只能创建匿名内部类的一个实例。
5. 一个匿名内部类一定是在 new 的后面，用其隐含实现一个接口或实现一个类。
6. **因匿名内部类为局部内部类**，所以局部内部类的所有限制都对其生效。

> 参考：https://www.cnblogs.com/ldl326308/p/9477566.html

### Java 中的修饰符大全解

Java 语言提供了很多修饰符，大概分为两类： 

1. 访问权限修饰符 
2. 非访问权限修饰符

**访问权限修饰符**

| 修饰符    | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| public    | 共有访问。对所有的类都可见                                   |
| protected | 保护型访问。对同一个包可见，对不同的包的子类可见             |
| default   | 默认访问权限。只对同一个包可见，**注意对不同的包的子类不可见** |
| private   | 私有访问。只对同一个类可见，其余都不见                       |

**非访问权限修饰符**

| 修饰符       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| static       | 用来创建类方法和类变量                                       |
| final        | 用来修饰类、方法和变量，<br />final 修饰的类不能够被继承，<br />final 修饰的方法不能被继承类重写，<br />final 修饰的变量为常量，是不可修改的 |
| abstract     | 用来创建抽象类和抽象方法                                     |
| synchronized | 用于多线程的同步                                             |
| volatile     | 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。<br />而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。<br />这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值 |
| transient    | 序列化的对象包含被 transient 修饰的实例变量时，java 虚拟机(JVM)跳过该特定的变量 |

**类的修饰符**

* **外部类修饰符**

  | 外部类修饰符             | 说明                                                         |
  | ------------------------ | ------------------------------------------------------------ |
  | public（访问控制符）     | 将一个类声明为公共类，它可以被任何对象访问，一个程序的主类必须是公共类 |
  | default（访问控制符）    | 类只对包内可见，包外不可见                                   |
  | abstract（非访问控制符） | 将一个类声明为抽象类，抽象类不能用来实例化对象， 声明抽象类的唯一目的是为了将来对该类进行扩充， 抽象类可以包含抽象方法和非抽象方法 |
  | final（非访问控制符）    | 将一个类生命为最终（即非继承类），表示它不能被其他类继承     |

  > 注意： 
  >
  > 1. protected 和 private 不能修饰外部类，是因为外部类放在包中，只有两种可能，包可见和包不可见。
  > 2. final 和 abstract 不能同时修饰外部类，因为该类要么能被继承要么不能被继承，二者只能选其一。 
  > 3. 不能用 static 修饰类，因为类加载后才会加载静态成员变量。所以不能用 static 修饰类和接口，因为类还没加载，无法使用 static 关键字。

* **内部类修饰符**

  内部类与成员变量地位一致，所以可以 public、protected、default 和 private，同时还可以用 static 修饰，表示嵌套静态内部类，不用实例化外部类，即可调用。

* **方法修饰符**

  | **方法修饰符**              | 说明                                                         |
  | --------------------------- | ------------------------------------------------------------ |
  | public（公共控制符）        | 包外包内都可以调用该方法                                     |
  | protected（保护访问控制符） | 指定该方法可以被它的类和子类进行访问 [具体细节可参考](http://blog.csdn.net/dawn_after_dark/article/details/74453915) |
  | default(默认权限）          | 指定该方法只对同包可见，对不同包（含不同包的子类）不可见     |
  | private（私有控制符）       | 指定此方法只能有自己类等方法访问，其他的类不能访问（包括子类），非常严格的控制 |
  | final                       | 指定方法已完备，不能再进行继承扩充                           |
  | static                      | 指定不需要实例化就可以激活的一个方法，即在内存中只有一份，通过类名即可调用 |
  | synchronize                 | 同步修饰符，在多个线程中，该修饰符用于在运行前，对它所属的方法加锁，以防止其他线程的访问，运行结束后解锁 |
  | native                      | 本地修饰符。指定此方法的方法体是用其他语言在程序外部编写的   |
  | abstract                    | 抽象方法是一种没有任何实现的方法，该方法的的具体实现由子类提供。 抽象方法不能被声明成 final 和 static。  任何继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类。  如果一个类包含若干个抽象方法，那么该类必须声明为抽象类。 抽象类可以不包含抽象方法。  抽象方法的声明以分号结尾，例如：`public abstract sample();` |

* **成员变量修饰符**

  | **成员变量修饰符**          | 说明                                                         |
  | --------------------------- | ------------------------------------------------------------ |
  | public（公共访问控制符）    | 指定该变量为公共的，它可以被任何对象的方法访问               |
  | protected（保护访问控制符） | 指定该变量可以别被自己的类和子类访问。在子类中可以覆盖此变量 |
  | default(默认权限）          | 指定该变量只对同包可见，对不同包（含不同包的子类）不可见     |
  | private（私有访问控制符）   | 指定该变量只允许自己的类的方法访问，其他任何类（包括子类）中的方法均不能访问 |
  | final                       | 最终修饰符，指定此变量的值不能变                             |
  | static（静态修饰符）        | 指定变量被所有对象共享，即所有实例都可以使用该变量。变量属于这个类 |
  | transient（过度修饰符）     | 指定该变量是系统保留，暂无特别作用的临时性变量。不持久化     |
  | volatile（易失修饰符）      | 指定该变量可以同时被几个线程控制和修改，保证两个不同的线程总是看到某个成员变量的同一个值 |
  | final+static                | 一起使用来创建常量                                           |

* **局部变量修饰符**

  only final is permitted：只有final是被允许的。

  1. 为什么不能赋予权限修饰符？ 

     因为局部变量的生命周期为一个方法的调用期间，所以没必要为其设置权限访问字段，既然你都能访问到这个方法，所以就没必要再为其方法内变量赋予访问权限，因为该变量在方法调用期间已经被加载进了虚拟机栈，换句话说就是肯定能被当前线程访问到，所以设置没意义。 

  2. 为什么不能用 static 修饰 

     我们都知道静态变量在方法之前先加载的，所以如果在方法内设置静态变量，可想而知，方法都没加载，你能加载成功方法内的静态变量？

**接口的修饰符**

* **接口修饰符**
  1. 接口修饰符只能用 public、default 和 abstract。 
  2. 不能用 final、static 修饰。
  3. 接口默认修饰为 abstract。
* **接口中方法修饰符**
  1. only public & abstract are permitted：意思只能用 public abstract 修饰，当然如果你什么都不写，默认就是public abstract。 
  2. 注意：在Java1.8之后，接口允许定义static 静态方法了！所以也可以用static来修饰！

> 参考：https://www.nowcoder.com/test/question/done?tid=38386652&qid=56132#summary
