# Java：语言基础学习笔记-3

> 时间紧任务重，此文作为后端学习基础笔记的第三篇

## 12. Object&常用API

### 12.1 Object类

#### 12.1.1 概述

`java.lang.Object` 类是Java语言中的根类，即所有类的父类。类中描述的所有方法子类都可以使用。在对象实例化的时候，最终找的父类就是Object。

如果一个类没有特别指定父类， 那么默认则继承自Object类。例如：

```java
public class MyClass /*extends Object*/ {
    // ...
}
```

根据JDK源代码及Object类的API文档，Object类当中包含的方法有11个。今天我们主要学习其中的2个：

* `public String toString()` ：返回该对象的字符串表示。
* `public boolean equals(Object obj)` ：指示其他某个对象是否与此对象“相等”。

#### 12.1.2 toString方法

**方法摘要**

`public String toString()` ：返回该对象的字符串表示。

toString方法返回该对象的字符串表示，其实该字符串内容就是**对象的类型+@+内存地址值**。

由于toString方法返回的结果是内存地址，而在开发中，经常需要按照对象的属性得到相应的字符串表现形式，因此也需要重写它。

**覆盖重写**

如果不希望使用toString方法的默认行为，则可以对它进行覆盖重写。例如自定义的Person类：

```java
public class Person {
    private String name;
    private int age;
    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    } 
    // 省略构造器与Getter Setter
}
```

在`IntelliJ IDEA`中，可以点击 Code 菜单中的 Generate... ，也可以使用快捷键 `alt+insert` ，点击 `toString()` 选项。选择需要包含的成员变量并确定。

> 小贴士： 在我们直接使用输出语句输出对象名的时候,其实通过该对象调用了其`toString()`方法。

#### 12.1.3 equals方法

**方法摘要**

`public boolean equals(Object obj)`：指示其他某个对象是否与此对象“相等”。

调用成员方法equals并指定参数为另一个对象，则可以判断这两个对象是否是相同的。这里的“相同”有默认和自定义两种方式。

**默认地址比较**

如果没有覆盖重写equals方法，那么Object类中**默认进行 == 运算符的对象地址比较**，只要不是同一个对象，结果必然为false。

**对象内容比较**

如果希望进行对象的内容比较，即所有或指定的部分成员变量相同就判定两个对象相同，则可以覆盖重写equals方法。例如：

```java
import java.util.Objects;
public class Person {
    private String name;
    private int age;
    @Override
    public boolean equals(Object o) {
        // 如果对象地址一样，则认为相同
        if (this == o)
            return true;
        // 如果参数为空，或者类型信息不一样，则认为不同
        if (o == null || getClass() != o.getClass())
            return false;
        // 转换为当前类型
        Person person = (Person) o;
        // 要求基本类型相等，并且将引用类型交给java.util.Objects类的equals静态方法取用结果
        return age == person.age && Objects.equals(name, person.name);
    }
}
```

这段代码充分考虑了对象为空、类型一致等问题，但方法内容并不唯一。大多数IDE都可以自动生成equals方法的代码内容。在IntelliJ IDEA中，可以使用 Code 菜单中的 Generate… 选项，也可以使用快捷键 `alt+insert` ，并选择 `equals() and hashCode()` 进行自动代码生成。

#### 12.1.4 Objects类

在刚才IDEA自动重写equals代码中，使用到了 `java.util.Objects` 类，那么这个类是什么呢？

在JDK7添加了一个Objects工具类，它提供了一些方法来操作对象，它由一些静态的实用方法组成，这些方法是`null-save`（空指针安全的）或`null-tolerant`（容忍空指针的），用于计算对象的hashcode、返回对象的字符串表示形式、比较两个对象。

在比较两个对象的时候，Object的equals方法容易抛出空指针异常，而Objects类中的equals方法就优化了这个问题。

方法如下：`public static boolean equals(Object a, Object b)`：判断两个对象是否相等。

我们可以查看一下源码，学习一下：

```java
public static boolean equals(Object a, Object b) {
	return (a == b) || (a != null && a.equals(b));
}
```

### 12.2 日期时间类

#### 12.2.1 Date类

**概述**

`java.util.Date` 类表示特定的瞬间，精确到毫秒。

继续查阅Date类的描述，发现Date拥有多个构造函数，只是部分已经过时，但是其中有未过时的构造函数可以把毫秒值转成日期对象。

* `public Date()` ：分配Date对象并初始化此对象，以表示分配它的时间（精确到毫秒）。

* `public Date(long date)` ：分配Date对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即1970年1月1日00:00:00 GMT）以来的指定毫秒数。

> tips: 由于我们处于东八区，所以我们的基准时间为1970年1月1日8时0分0秒。

简单来说：

* 使用无参构造，可以自动设置当前系统时间的毫秒时刻；
* 指定long类型的构造参数，可以自定义毫秒时刻。

```java
import java.util.Date;
public class Demo01Date {
    public static void main(String[] args) {
        // 创建日期对象，把当前的时间
        System.out.println(new Date()); // Tue Jan 16 14:37:35 CST 2018
        // 创建日期对象，把当前的毫秒值转成日期对象
        System.out.println(new Date(0L)); // Thu Jan 01 08:00:00 CST 1970
    }
}
```

> tips：在使用println方法时，会自动调用Date类中的toString方法。Date类对Object类中的toString方法进行了覆盖重写，所以结果为指定格式的字符串。

**常用方法**

Date类中的多数方法已经过时，常用的方法有：

`public long getTime()` 把日期对象转换成对应的时间毫秒值。

#### 12.2.2 DateFormat类

`java.text.DateFormat`是日期/时间格式化子类的抽象类，我们通过这个类可以帮我们完成日期和文本之间的转换,也就是可以在Date对象与String对象之间进行来回转换。

* **格式化**：按照指定的格式，从Date对象转换为String对象。
* **解析**：按照指定的格式，从String对象转换为Date对象。

**构造方法**

由于DateFormat为抽象类，不能直接使用，所以需要常用的子类 `java.text.SimpleDateFormat` 。这个类需要一个模式（格式）来指定格式化或解析的标准。

构造方法为：`public SimpleDateFormat(String pattern)`：用给定的模式和默认语言环境的日期格式符号构造SimpleDateFormat。

参数pattern是一个字符串，代表日期时间的自定义格式。

**格式规则**

常用的格式规则为：

| 标识字母（区分大小写） | 含义 |
| ---------------------- | ---- |
| y                      | 年   |
| M                      | 月   |
| d                      | 日   |
| H                      | 时   |
| m                      | 分   |
| s                      | 秒   |

> 备注：更详细的格式规则，可以参考SimpleDateFormat类的API文档。

创建SimpleDateFormat对象的代码如：

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
public class Demo02SimpleDateFormat {
    public static void main(String[] args) {
        // 对应的日期格式如：2018‐01‐16 15:06:38
        DateFormat format = new SimpleDateFormat("yyyy‐MM‐dd HH:mm:ss");
    }
}
```

**常用方法**

DateFormat类的常用方法有：

`public String format(Date date)` ：将Date对象格式化为字符串。

`public Date parse(String source)` ：将字符串解析为Date对象。

**format方法**

使用format方法的代码为：

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;
/*
把Date对象转换成String
*/
public class Demo03DateFormatMethod {
    public static void main(String[] args) {
        Date date = new Date();
        // 创建日期格式化对象,在获取格式化对象时可以指定风格
        DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
        String str = df.format(date);
        System.out.println(str); // 2008年1月23日
    }
}
```

**parse方法**

使用parse方法的代码为：

```java
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
/*
把String转换成Date对象
*/
public class Demo04DateFormatMethod {
    public static void main(String[] args) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
        String str = "2018年12月11日";
        Date date = df.parse(str);
        System.out.println(date); // Tue Dec 11 00:00:00 CST 2018
    }
}
```

#### 12.2.3 练习

请使用日期时间相关的API，计算出一个人已经出生了多少天。

**思路：**

1. 获取当前时间对应的毫秒值
2. 获取自己出生日期对应的毫秒值
3. 两个时间相减（当前时间– 出生日期）

代码实现：

```java
public static void function() throws Exception {
    System.out.println("请输入出生日期 格式 YYYY‐MM‐dd");
    // 获取出生日期,键盘输入
    String birthdayString = new Scanner(System.in).next();
    // 将字符串日期,转成Date对象
    // 创建SimpleDateFormat对象,写日期模式
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy‐MM‐dd");
    // 调用方法parse,字符串转成日期对象
    Date birthdayDate = sdf.parse(birthdayString);
    // 获取今天的日期对象
    Date todayDate = new Date();
    // 将两个日期转成毫秒值,Date类的方法getTime
    long birthdaySecond = birthdayDate.getTime();
    long todaySecond = todayDate.getTime();
    long secone = todaySecond‐birthdaySecond;
    if (secone < 0){
        System.out.println("还没出生呢");
    } else {
        System.out.println(secone/1000/60/60/24);
    }
}
```

#### 12.2.4 Calendar类

日历我们都见过

`java.util.Calendar` 是日历类，在Date后出现，替换掉了许多Date的方法。该类将所有可能用到的时间信息封装为静态成员变量，方便获取。日历类就是方便获取各个时间属性的。

**获取方式**

Calendar为抽象类，由于语言敏感性，Calendar类在创建对象时并非直接创建，而是通过静态方法创建，返回子类对象，如下：

Calendar静态方法`public static Calendar getInstance()` ：使用默认时区和语言环境获得一个日历，例如：

```java
import java.util.Calendar;
public class Demo06CalendarInit {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
    }
}
```

**常用方法**

根据Calendar类的API文档，常用方法有：

* `public int get(int field)` ：返回给定日历字段的值。
* `public void set(int field, int value)` ：将给定的日历字段设置为给定值。
* `public abstract void add(int field, int amount) `：根据日历的规则，为给定的日历字段添加或减去指定的时间量。
* `public Date getTime()` ：返回一个表示此Calendar时间值（从历元到现在的毫秒偏移量）的Date对象。

Calendar类中提供很多成员常量，代表给定的日历字段：

| 字段值       | 含义                                  |
| ------------ | ------------------------------------- |
| YEAR         | 年                                    |
| MONTH        | 月（从0开始，可以+1使用）             |
| DAY_OF_MONTH | 月中的天（几号）                      |
| HOUR         | 时（12小时制）                        |
| HOUR_OF_DAY  | 时（24小时制）                        |
| MINUTE       | 分                                    |
| SECOND       | 秒                                    |
| DAY_OF_WEEK  | 周中的天（周几，周日为1，可以-1使用） |

**get/set方法**

get方法用来获取指定字段的值，set方法用来设置指定字段的值，代码使用演示：

* get 方法：

```java
import java.util.Calendar;
public class CalendarUtil {
    public static void main(String[] args) {
        // 创建Calendar对象
        Calendar cal = Calendar.getInstance();
        // 获取年
        int year = cal.get(Calendar.YEAR);
        // 获取月
        int month = cal.get(Calendar.MONTH) + 1;
        // 获取日
        int dayOfMonth = cal.get(Calendar.DAY_OF_MONTH);
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日");
    }
}
```

* set 方法：

```java
import java.util.Calendar;
public class Demo07CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        cal.set(Calendar.YEAR, 2020);
        System.out.print(year + "年" + month + "月" + dayOfMonth + "日"); // 2020年1月17日
    }
}
```

**add方法**

add方法可以对指定日历字段的值进行加减操作，如果第二个参数为正数则加上偏移量，如果为负数则减去偏移量。代码如：

```java
import java.util.Calendar;
public class Demo08CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        // 获取年
        int year = cal.get(Calendar.YEAR);
        // 获取月
        int month = cal.get(Calendar.MONTH) + 1;
        // 获取日
        int dayOfMonth = cal.get(Calendar.DAY_OF_MONTH);
        System.out.println(year + "年" + month + "月" + dayOfMonth + "日"); //2020年9月21日
        // 使用add方法
        cal.add(Calendar.DAY_OF_MONTH, 2); // 加2天
        cal.add(Calendar.YEAR, -3); // 减3年
        year = cal.get(Calendar.YEAR);
        // 获取月
        month = cal.get(Calendar.MONTH) + 1;
        // 获取日
        dayOfMonth = cal.get(Calendar.DAY_OF_MONTH);
        System.out.println(year + "年" + month + "月" + dayOfMonth + "日");//2017年9月23日
    }
}
```

**getTime方法**

Calendar中的getTime方法并不是获取毫秒时刻，而是拿到对应的Date对象。

```java
import java.util.Calendar;
import java.util.Date;
public class Demo09CalendarMethod {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        Date date = cal.getTime();
        System.out.println(date); // Tue Jan 16 16:03:09 CST 2018
    }
}
```

> 小贴士：
>
> 西方星期的开始为周日，中国为周一。
>
> 在Calendar类中，月份的表示是以0-11代表1-12月。
>
> 日期是有大小关系的，时间靠后，时间越大。

### 12.3 System类

`java.lang.System` 类中提供了大量的静态方法，可以获取与系统相关的信息或系统级操作，在System类的API文档中，常用的方法有：

* `public static long currentTimeMillis()` ：返回以毫秒为单位的当前时间。
* `public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)` ：将数组中指定的数据拷贝到另一个数组中。

#### 12.3.1 currentTimeMillis

实际上，currentTimeMillis方法就是 获取当前系统时间与1970年01月01日00:00点之间的毫秒差值

```java
import java.util.Date;
public class SystemDemo {
    public static void main(String[] args) {
        //获取当前时间毫秒值
        System.out.println(System.currentTimeMillis()); // 1516090531144
    }
}
```

**练习**

验证for循环打印数字1-9999所需要使用的时间（毫秒）

```java
public class SystemTest1 {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            System.out.println(i);
        } 
        long end = System.currentTimeMillis();
        System.out.println("共耗时毫秒：" + (end ‐ start));
    }
}
```

#### 12.3.2 arraycopy方法

`public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)` ：将数组中指定的数据拷贝到另一个数组中。

数组的拷贝动作是系统级的，性能很高。System.arraycopy方法具有5个参数，含义分别为：

| 参数序号 | 参数名称 | 参数类型 | 参数含义             |
| -------- | -------- | -------- | -------------------- |
| 1        | src      | Object   | 源数组               |
| 2        | srcPos   | int      | 源数组索引起始位置   |
| 3        | dest     | Object   | 目标数组             |
| 4        | destPos  | int      | 目标数组索引起始位置 |
| 5        | length   | int      | 复制元素个数         |

**练习**

将src数组中前3个元素，复制到dest数组的前3个位置上

* 复制元素前：src数组元素[1,2,3,4,5]，dest数组元素[6,7,8,9,10]
* 复制元素后：src数组元素[1,2,3,4,5]，dest数组元素[1,2,3,9,10]

```java
import java.util.Arrays;
public class Demo11SystemArrayCopy {
    public static void main(String[] args) {
        int[] src = new int[]{1,2,3,4,5};
        int[] dest = new int[]{6,7,8,9,10};
        System.arraycopy( src, 0, dest, 0, 3);
        /*代码运行后：两个数组中的元素发生了变化
		src数组元素[1,2,3,4,5]
		dest数组元素[1,2,3,9,10]
		*/
    }
}
```

### 12.4 StringBuilder类

#### 12.4.1 字符串拼接问题

由于String类的对象内容不可改变，所以每当进行字符串拼接时，总是会在内存中创建一个新的对象。例如：

```java
public class StringDemo {
    public static void main(String[] args) {
        String s = "Hello";
        s += "World";
        System.out.println(s);
    }
}
```

在API中对String类有这样的描述：字符串是常量，它们的值在创建后不能被更改。

根据这句话分析我们的代码，其实总共产生了三个字符串，即 "Hello" 、 "World" 和 "HelloWorld" 。引用变量s首先指向 Hello 对象，最终指向拼接出来的新字符串对象，即 HelloWord 。

![String](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509089.PNG)

由此可知，如果对字符串进行拼接操作，每次拼接，都会构建一个新的String对象，既耗时，又浪费空间。为了解决这一问题，可以使用 `java.lang.StringBuilder` 类。

#### 12.4.2 StringBuilder概述

查阅 `java.lang.StringBuilder` 的API，StringBuilder又称为可变字符序列，它是一个类似于 String 的字符串缓冲区，通过某些方法调用可以改变该序列的长度和内容。

原来StringBuilder是个字符串的缓冲区，即它是一个容器，容器中可以装很多字符串。并且能够对其中的字符串进行各种操作。

它的内部拥有一个数组用来存放字符串内容，进行字符串拼接时，直接在数组中加入新内容。StringBuilder会自动维护数组的扩容。原理如下图所示：(默认16字符空间，超过自动扩充)

![stringbuilder](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510414.PNG)

#### 12.4.3 构造方法

根据StringBuilder的API文档，常用构造方法有2个：

* `public StringBuilder()` ：构造一个空的StringBuilder容器。
* `public StringBuilder(String str)` ：构造一个StringBuilder容器，并将字符串添加进去。

```java
public class StringBuilderDemo {
    public static void main(String[] args) {
        StringBuilder sb1 = new StringBuilder();
        System.out.println(sb1); // (空白)
        // 使用带参构造
        StringBuilder sb2 = new StringBuilder("itcast");
        System.out.println(sb2); // itcast
    }
}
```

#### 12.4.4 常用方法

StringBuilder常用的方法有2个：

* `public StringBuilder append(...)` ：添加任意类型数据的字符串形式，并返回当前对象自身。
* `public String toString()` ：将当前StringBuilder对象转换为String对象。

**append方法**

append方法具有多种重载形式，可以接收任意类型的参数。任何数据作为参数都会将对应的字符串内容添加到StringBuilder中。例如：

```java
public class Demo02StringBuilder {
    public static void main(String[] args) {
        //创建对象
        StringBuilder builder = new StringBuilder();
        //public StringBuilder append(任意类型)
        StringBuilder builder2 = builder.append("hello");
        //对比一下
        System.out.println("builder:"+builder);
        System.out.println("builder2:"+builder2);
        System.out.println(builder == builder2); //true
        // 可以添加 任何类型
        builder.append("hello");
        builder.append("world");
        builder.append(true);
        builder.append(100);
        // 在我们开发中，会遇到调用一个方法后，返回一个对象的情况。然后使用返回的对象继续调用方法。
        // 这种时候，我们就可以把代码现在一起，如append方法一样，代码如下
        // 链式编程
        builder.append("hello").append("world").append(true).append(100);
        System.out.println("builder:"+builder);
    }
}
```

> 备注：StringBuilder已经覆盖重写了Object当中的toString方法。

**toString方法**

通过toString方法，StringBuilder对象将会转换为不可变的String对象。如：

```java
public class Demo16StringBuilder {
    public static void main(String[] args) {
        // 链式创建
        StringBuilder sb = new StringBuilder("Hello").append("World").append("Java");
        // 调用方法
        String str = sb.toString();
        System.out.println(str); // HelloWorldJava
    }
}
```

### 12.5 包装类

#### 12.5.1 概述

Java提供了两个类型系统，基本类型与引用类型，使用基本类型在于效率，然而很多情况，会创建对象使用，因为对象可以做更多的功能，如果想要我们的基本类型像对象一样操作，就可以使用基本类型对应的包装类，如下：

| 基本类型 | 对应的包装类（位于java.lang包中） |
| -------- | --------------------------------- |
| byte     | Byte                              |
| short    | Short                             |
| int      | Integer                           |
| long     | Long                              |
| float    | Float                             |
| double   | Double                            |
| char     | Character                         |
| boolean  | Boolean                           |

#### 12.5.2 装箱与拆箱

基本类型与对应的包装类对象之间，来回转换的过程称为”装箱“与”拆箱“：

* 装箱：从基本类型转换为对应的包装类对象。
* 拆箱：从包装类对象转换为对应的基本类型。

用Integer与 int为例：（看懂代码即可）

基本数值---->包装对象

```java
Integer i = new Integer(4);//使用构造函数函数
Integer iii = Integer.valueOf(4);//使用包装类中的valueOf方法
```

包装对象---->基本数值

```java
int num = i.intValue();
```

#### 12.5.3 自动装箱与自动拆箱

由于我们经常要做基本类型与包装类之间的转换，从Java 5（JDK 1.5）开始，基本类型与包装类的装箱、拆箱动作可以自动完成。例如：

```java
Integer i = 4;	//自动装箱。相当于Integer i = Integer.valueOf(4);
i = i + 5;		//等号右边：将i对象转成基本数值(自动拆箱) i.intValue() + 5;
				//加法运算完成后，再次装箱，把基本数值转成对象。
```

#### 12.5.3 基本类型与字符串之间的转换

**基本类型转换为String**

基本类型转换String总共有三种方式，查看课后资料可以得知，这里只讲最简单的一种方式：

```java
基本类型直接与””相连接即可；如：34+""
```

**String转换成对应的基本类型**

除了Character类之外，其他所有包装类都具有parseXxx静态方法可以将字符串参数转换为对应的基本类型：

* `public static byte parseByte(String s)` ：将字符串参数转换为对应的byte基本类型。
* `public static short parseShort(String s)` ：将字符串参数转换为对应的short基本类型。
* `public static int parseInt(String s)` ：将字符串参数转换为对应的int基本类型。
* `public static long parseLong(String s)` ：将字符串参数转换为对应的long基本类型。
* `public static float parseFloat(String s)` ：将字符串参数转换为对应的float基本类型。
* `public static double parseDouble(String s)` ：将字符串参数转换为对应的double基本类型。
* `public static boolean parseBoolean(String s)` ：将字符串参数转换为对应的boolean基本类型。

代码使用（仅以Integer类的静态方法parseXxx为例）如：

```java
public class Demo18WrapperParse {
    public static void main(String[] args) {
        int num = Integer.parseInt("100");
    }
}
```

> 注意:如果字符串参数的内容无法正确转换为对应的基本类型，则会抛出 `java.lang.NumberFormatException`异常。

## 13. Collection&泛型

### 13.1 Collection集合

#### 13.1.1 集合概述

前面我们已经学习过并使用过集合ArrayList，那么集合到底是什么呢?

**集合**：集合是java中提供的一种容器，可以用来存储多个数据。

集合和数组既然都是容器，它们有啥区别呢？

* 数组的长度是固定的。集合的长度是可变的。
* 数组中存储的是同一类型的元素，可以存储基本数据类型值。集合存储的都是对象。而且对象的类型可以不一致。在开发中一般当对象多的时候，使用集合进行存储。

#### 13.1.2 集合框架

JAVA SE 提供了满足各种需求的API，在使用这些API前，先了解其继承与接口操作架构，才能了解何时采用哪个类，以及类之间如何彼此合作，从而达到灵活应用。

集合按照其存储结构可以分为两大类，分别是单列集合 `java.util.Collection` 和双列集合 `java.util.Map` ，今天我们主要学习 Collection 集合。

**Collection**：单列集合类的根接口，用于存储一系列符合某种规则的元素，它有两个重要的子接口，分别是`java.util.List` 和 `java.util.Set` 。

* 其中， List 的特点是元素有序、元素可重复；List 接口的主要实现类有 `java.util.ArrayList`和`java.util.LinkedList` 
* Set 的特点是元素无序，而且不可重复； Set 接口的主要实现类有 `java.util.HashSet` 和 `java.util.TreeSet`。

从上面的描述可以看出JDK中提供了丰富的集合类库，为了便于初学者进行系统地学习，接下来通过一张图来描述整个集合类的继承体系。

![集合类的继承体系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509591.PNG)

其中，橙色框里填写的都是接口类型，而蓝色框里填写的都是具体的实现类。这几天将针对图中所列举的集合类进行逐一地讲解。

集合本身是一个工具，它存放在java.util包中。在 Collection 接口定义着单列集合框架中最最共性的内容。

#### 13.1.3 Collection 常用功能

Collection是所有单列集合的父接口，因此在Collection中定义了单列集合(List和Set)通用的一些方法，这些方法可用于操作所有的单列集合。方法如下：

* `public boolean add(E e)` ： 把给定的对象添加到当前集合中 。
* `public void clear()` :清空集合中所有的元素。
* `public boolean remove(E e)` : 把给定的对象在当前集合中删除。
* `public boolean contains(E e)` : 判断当前集合中是否包含给定的对象。
* `public boolean isEmpty()` : 判断当前集合是否为空。
* `public int size()` : 返回集合中元素的个数。
* `public Object[] toArray()` : 把集合中的元素，存储到数组中。

方法演示：

```java
import java.util.ArrayList;
import java.util.Collection;
public class Demo1Collection {
    public static void main(String[] args) {
        // 创建集合对象
        // 使用多态形式
        Collection<String> coll = new ArrayList<String>();
        // 使用方法
        // 添加功能 boolean add(String s)
        coll.add("小李广");
        coll.add("扫地僧");
        coll.add("石破天");
        System.out.println(coll);
        // boolean contains(E e) 判断o是否在集合中存在
        System.out.println("判断 扫地僧 是否在集合中"+coll.contains("扫地僧"));
        //boolean remove(E e) 删除在集合中的o元素
        System.out.println("删除石破天："+coll.remove("石破天"));
        System.out.println("操作之后集合中元素:"+coll);
        // size() 集合中有几个元素
        System.out.println("集合中有"+coll.size()+"个元素");
        // Object[] toArray()转换成一个Object数组
        Object[] objects = coll.toArray();
        // 遍历数组
        for (int i = 0; i < objects.length; i++) {
            System.out.println(objects[i]);
        } 
        // void clear() 清空集合
        coll.clear();
        System.out.println("集合中内容为："+coll);
        // boolean isEmpty() 判断是否为空
        System.out.println(coll.isEmpty());
    }
}
```

> tips: 有关Collection中的方法可不止上面这些，其他方法可以自行查看API学习。

### 13.2 Iterator迭代器

#### 13.2.1 Iterator接口

在程序开发中，经常需要遍历集合中的所有元素。针对这种需求，JDK专门提供了一个接口`java.util.Iterator` 。Iterator 接口也是Java集合中的一员，但它与 Collection 、 Map 接口有所不同， Collection 接口与 Map 接口主要用于存储元素，**而 Iterator 主要用于迭代访问**（即遍历）Collection 中的元素，因此 Iterator 对象也被称为迭代器。

想要遍历Collection集合，那么就要获取该集合迭代器完成迭代操作，下面介绍一下获取迭代器的方法：

* `public Iterator iterator()` : 获取集合对应的迭代器，用来遍历集合中的元素的。

下面介绍一下迭代的概念：

**迭代**：即Collection集合元素的通用获取方式。在取元素之前先要判断集合中有没有元素，如果有，就把这个元素取出来，继续在判断，如果还有就再取出出来。一直把集合中的所有元素全部取出。这种取出方式专业术语称为迭代。

Iterator接口的常用方法如下：

* `public E next()` :返回迭代的下一个元素。
* `public boolean hasNext()` :如果仍有元素可以迭代，则返回 true。

接下来我们通过案例学习如何使用Iterator迭代集合中元素：

```java
public class IteratorDemo {
    public static void main(String[] args) {
        // 使用多态方式 创建对象
        Collection<String> coll = new ArrayList<String>();
        // 添加元素到集合
        coll.add("串串星人");
        coll.add("吐槽星人");
        coll.add("汪星人");
        //遍历
        //使用迭代器 遍历 每个集合对象都有自己的迭代器
        Iterator<String> it = coll.iterator();
        // 泛型指的是 迭代出 元素的数据类型
        while(it.hasNext()){ //判断是否有迭代元素
            String s = it.next();//获取迭代出的元素
            System.out.println(s);
        }
    }
}
```

> tips：在进行集合元素取出时，如果集合中已经没有元素了，还继续使用迭代器的next方法，将会发生`java.util.NoSuchElementException`没有集合元素的错误。

#### 13.2.2 迭代器的实现原理

我们在之前案例已经完成了Iterator遍历集合的整个过程。当遍历集合时，首先通过调用t集合的 `iterator()` 方法获得迭代器对象，然后使用hashNext()方法判断集合中是否存在下一个元素，如果存在，则调用next()方法将元素取出，否则说明已到达了集合末尾，停止遍历元素。

Iterator迭代器对象在遍历集合时，内部采用指针的方式来跟踪集合中的元素，为了让初学者能更好地理解迭代器的工作原理，接下来通过一个图例来演示Iterator对象迭代元素的过程：

![Iterator迭代器](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509707.PNG)

在调用Iterator的next方法之前，迭代器的索引位于第一个元素之前，不指向任何元素，当第一次调用迭代器的next方法后，迭代器的索引会向后移动一位，指向第一个元素并将该元素返回，当再次调用next方法时，迭代器的索引会指向第二个元素并将该元素返回，依此类推，直到hasNext方法返回false，表示到达了集合的末尾，终止对元素的遍历。

#### 13.2.3 增强for

增强for循环(也称for each循环)是JDK1.5以后出来的一个高级for循环，专门用来遍历数组和集合的。它的内部原理其实是个Iterator迭代器，所以在遍历的过程中，不能对集合中的元素进行增删操作。

格式：

```java
for(元素的数据类型 变量 : Collection集合or数组){
    //写操作代码
}
```

它用于遍历Collection和数组。通常只进行遍历元素，不要在遍历的过程中对集合元素进行增删操作。

**练习1：遍历数组**

```java
public class NBForDemo1 {
    public static void main(String[] args) {
        int[] arr = {3,5,6,87};
        //使用增强for遍历数组
        for(int a : arr){//a代表数组中的每个元素
            System.out.println(a);
        }
    }
}
```

**练习2：遍历集合**

```java
public class NBFor {
    public static void main(String[] args) {
        Collection<String> coll = new ArrayList<String>();
        coll.add("小河神");
        coll.add("老河神");
        coll.add("神婆");
        //使用增强for遍历
        for(String s :coll){//接收变量s代表 代表被遍历到的集合元素
            System.out.println(s);
        }
    }
}
```

> tips: 新for循环必须有被遍历的目标。目标只能是Collection或者是数组。新式for仅仅作为遍历操作出现。

### 13.3 泛型

#### 13.3.1 泛型概述

在前面学习集合时，我们都知道集合中是可以存放任意对象的，只要把对象存储集合后，那么这时他们都会被提升成Object类型。当我们在取出每一个对象，并且进行相应的操作，这时必须采用类型转换。

大家观察下面代码：

```java
public class GenericDemo {
    public static void main(String[] args) {
        Collection coll = new ArrayList();
        coll.add("abc");
        coll.add("itcast");
        coll.add(5);//由于集合没有做任何限定，任何类型都可以给其中存放
        Iterator it = coll.iterator();
        while(it.hasNext()){
            //需要打印每个字符串的长度,就要把迭代出来的对象转成String类型
            String str = (String) it.next();
            System.out.println(str.length());
        }
    }
}
```

程序在运行时发生了问题`java.lang.ClassCastException`。 为什么会发生类型转换异常呢？ 我们来分析下：由于集合中什么类型的元素都可以存储。导致取出时强转引发运行时 `ClassCastException`。 怎么来解决这个问题呢？ 

Collection虽然可以存储各种对象，但实际上通常Collection只存储同一类型对象。例如都是存储字符串对象。因此在JDK5之后，新增了泛型(Generic)语法，让你在设计API时可以指定类或方法支持泛型，这样我们使用API的时候也变得更为简洁，并得到了编译时期的语法检查。

**泛型：可以在类或方法中预支地使用未知的类型。**

> tips:一般在创建对象时，将未知的类型确定具体的类型。当没有指定泛型时，默认类型为Object类型。

#### 13.3.2 使用泛型的好处

上一节只是讲解了泛型的引入，那么泛型带来了哪些好处呢？

* 将运行时期的ClassCastException，转移到了编译时期变成了编译失败。
* 避免了类型强转的麻烦。

通过我们如下代码体验一下：

```java
public class GenericDemo2 {
    public static void main(String[] args) {
        Collection<String> list = new ArrayList<String>();
        list.add("abc");
        list.add("itcast");
        // list.add(5);//当集合明确类型后，存放类型不一致就会编译报错
        // 集合已经明确具体存放的元素类型，那么在使用迭代器的时候，迭代器也同样会知道具体遍历元素类型
        Iterator<String> it = list.iterator();
        while(it.hasNext()){
            String str = it.next();
            //当使用Iterator<String>控制元素类型后，就不需要强转了。获取到的元素直接就是String类型
            System.out.println(str.length());
        }
    }
}
```

> tips:泛型是数据类型的一部分，我们将类名与泛型合并一起看做数据类型。

#### 13.3.3 泛型的定义与使用

我们在集合中会大量使用到泛型，这里来完整地学习泛型知识。

泛型，用来灵活地将数据类型应用到不同的类、方法、接口当中。将数据类型作为参数进行传递。

**定义和使用含有泛型的类**

定义格式：

```java
修饰符 class 类名<代表泛型的变量> { }
```

例如，API中的ArrayList集合：

```java
class ArrayList<E>{
    public boolean add(E e){ }
    public E get(int index){ }
    ....
}
```

使用泛型： 即什么时候确定泛型。

**在创建对象的时候确定泛型**

例如， `ArrayList<String> list = new ArrayList<String>();`

此时，变量E的值就是String类型,那么我们的类型就可以理解为：

```java
class ArrayList<String>{
    public boolean add(String e){ }
    public String get(int index){ }
    ...
}
```

再例如， `ArrayList<Integer> list = new ArrayList<Integer>();`

此时，变量E的值就是Integer类型,那么我们的类型就可以理解为：

```java
class ArrayList<Integer> {
    public boolean add(Integer e) { }
    public Integer get(int index) { }
    ...
}
```

举例自定义泛型类

```java
public class MyGenericClass<MVP> {
    //没有MVP类型，在这里代表 未知的一种数据类型 未来传递什么就是什么类型
    private MVP mvp;
    public void setMVP(MVP mvp) {
        this.mvp = mvp;
    } 
    public MVP getMVP() {
        return mvp;
    }
}
```

使用:

```java
public class GenericClassDemo {
    public static void main(String[] args) {
        // 创建一个泛型为String的类
        MyGenericClass<String> my = new MyGenericClass<String>();
        // 调用setMVP
        my.setMVP("大胡子登登");
        // 调用getMVP
        String mvp = my.getMVP();
        System.out.println(mvp);
        //创建一个泛型为Integer的类
        MyGenericClass<Integer> my2 = new MyGenericClass<Integer>();
        my2.setMVP(123);
        Integer mvp2 = my2.getMVP();
    }
}
```

**含有泛型的方法**

定义格式：

```java
修饰符 <代表泛型的变量> 返回值类型 方法名(参数){ }
```

例如，

```java
public class MyGenericMethod {
    public <MVP> void show(MVP mvp) {
        System.out.println(mvp.getClass());
    } 
    public <MVP> MVP show2(MVP mvp) {
        return mvp;
    }
}
```

使用格式：**调用方法时，确定泛型的类型**

```java
public class GenericMethodDemo {
    public static void main(String[] args) {
        // 创建对象
        MyGenericMethod mm = new MyGenericMethod();
        // 演示看方法提示
        mm.show("aaa");
        mm.show(123);
        mm.show(12.45);
    }
}
```

**含有泛型的接口**

定义格式：

```java
修饰符 interface 接口名<代表泛型的变量> { }
```

例如：

```java
public interface MyGenericInterface<E>{
    public abstract void add(E e);
    
    public abstract E getE();
}
```

使用格式：

1. **定义类时确定泛型的类型**

   例如：

   ```java
   public class MyImp1 implements MyGenericInterface<String> {
       @Override
       public void add(String e) {
           // 省略...
       } 
       @Override
       public String getE() {
           return null;
       }
   }
   ```

   此时，泛型E的值就是String类型。

2. **始终不确定泛型的类型，直到创建对象时，确定泛型的类型**

   例如：

   ```java
   public class MyImp2<E> implements MyGenericInterface<E> {
       @Override
       public void add(E e) {
           // 省略...
       } 
       @Override
       public E getE() {
           return null;
       }
   }
   ```

   确定泛型：

   ```java
   public class GenericInterface {
       public static void main(String[] args) {
           MyImp2<String> my = new MyImp2<String>();
           my.add("aa");
       }
   }
   ```

#### 13.3.4 泛型通配符

当使用泛型类或者接口时，传递的数据中，泛型类型不确定，可以通过通配符`<?>`表示。但是一旦使用泛型的通配符后，只能使用Object类中的共性方法，集合中元素自身方法无法使用。

**通配符基本使用**

泛型的通配符：**不知道使用什么类型来接收的时候,此时可以使用`?`,`?`表示未知通配符**。

此时只能接受数据，不能往该集合中存储数据。

举个例子大家理解使用即可：

```java
public static void main(String[] args) {
    Collection<Intger> list1 = new ArrayList<Integer>();
    getElement(list1);
    Collection<String> list2 = new ArrayList<String>();
    getElement(list2);
} 

//？代表可以接收任意类型
public static void getElement(Collection<?> coll){}
```

> tips:泛型不存在继承关系 `Collection list = new ArrayList();`这种是错误的。

**通配符高级使用：受限泛型**

之前设置泛型的时候，实际上是可以任意设置的，只要是类就可以设置。但是在JAVA的泛型中可以指定一个泛型的上限和下限。

* **泛型的上限：**
  * 格式： 类型名称 `<? extends 类 >` 对象名称
  * 意义： 只能接收该类型及其子类

* **泛型的下限：**
  * 格式： 类型名称 `<? super 类 >` 对象名称
  * 意义： 只能接收该类型及其父类型

比如：现已知Object类，String 类，Number类，Integer类，其中Number是Integer的父类

```java
public static void main(String[] args) {
    Collection<Integer> list1 = new ArrayList<Integer>();
    Collection<String> list2 = new ArrayList<String>();
    Collection<Number> list3 = new ArrayList<Number>();
    Collection<Object> list4 = new ArrayList<Object>();
    getElement(list1);
    getElement(list2);//报错
    getElement(list3);
    getElement(list4);//报错
    getElement2(list1);//报错
    getElement2(list2);//报错
    getElement2(list3);
    getElement2(list4);
}

// 泛型的上限：此时的泛型?，必须是Number类型或者Number类型的子类
public static void getElement1(Collection<? extends Number> coll){}
// 泛型的下限：此时的泛型?，必须是Number类型或者Number类型的父类
public static void getElement2(Collection<? super Number> coll){}
```

### 13.4 集合综合案例

#### 13.4.1 案例介绍

按照斗地主的规则，完成洗牌发牌的动作。 具体规则：

使用54张牌打乱顺序，三个玩家参与游戏，三人交替摸牌，每人17张牌，最后三张留作底牌。

#### 13.4.2 案例分析

* 准备牌：

  牌可以设计为一个ArrayList，每个字符串为一张牌。 每张牌由花色数字两部分组成，我们可以使用花色集合与数字集合嵌套迭代完成每张牌的组装。 牌由Collections类的shuffle方法进行随机排序。

* 发牌：

  将每个人以及底牌设计为ArrayList,将最后3张牌直接存放于底牌，剩余牌通过对3取模依次发牌。

* 看牌：

  直接打印每个集合。

#### 13.4.3 代码实现

```java
import java.util.ArrayList;
import java.util.Collections;
public class Poker {
    public static void main(String[] args) {
        /*
         * 1: 准备牌操作
         */
        //1.1 创建牌盒 将来存储牌面的
        ArrayList<String> pokerBox = new ArrayList<String>();
        //1.2 创建花色集合
        ArrayList<String> colors = new ArrayList<String>();
        //1.3 创建数字集合
        ArrayList<String> numbers = new ArrayList<String>();
        //1.4 分别给花色 以及 数字集合添加元素
        colors.add("♥");
        colors.add("♦");
        colors.add("♠");
        colors.add("♣");
        for(int i = 2;i<=10;i++){
            numbers.add(i+"");
        }
        numbers.add("J");
        numbers.add("Q");
        numbers.add("K");
        numbers.add("A");
        //1.5 创造牌 拼接牌操作
        // 拿出每一个花色 然后跟每一个数字 进行结合 存储到牌盒中
        for (String color : colors) {
            //color每一个花色
            //遍历数字集合
            for(String number : numbers){
                //结合
                String card = color+number;
                //存储到牌盒中
                pokerBox.add(card);
            }
        } 
        //1.6大王小王
		pokerBox.add("小☺");
        pokerBox.add("大☠");
        // System.out.println(pokerBox);
        // 洗牌 是不是就是将 牌盒中 牌的索引打乱
        // Collections类 工具类 都是 静态方法
        // shuffer方法
        /*
         * static void shuffle(List<?> list)
         * 使用默认随机源对指定列表进行置换。
         */
        //2:洗牌
        Collections.shuffle(pokerBox);
        //3 发牌
        //3.1 创建 三个 玩家集合 创建一个底牌集合
        ArrayList<String> player1 = new ArrayList<String>();
        ArrayList<String> player2 = new ArrayList<String>();
        ArrayList<String> player3 = new ArrayList<String>();
        ArrayList<String> dipai = new ArrayList<String>();
        //遍历 牌盒 必须知道索引
        for(int i = 0;i<pokerBox.size();i++){
            //获取 牌面
            String card = pokerBox.get(i);
            //留出三张底牌 存到 底牌集合中
            if(i>=51){//存到底牌集合中
                dipai.add(card);
            } else {
                //玩家1 %3 ==0
                if(i%3==0){
                    player1.add(card);
                }else if(i%3==1){//玩家2
                    player2.add(card);
                }else{//玩家3
                    player3.add(card);
                }
            }
        } 
        //看看
        System.out.println("令狐冲："+player1);
        System.out.println("田伯光："+player2);
        System.out.println("绿竹翁："+player3);
        System.out.println("底牌："+dipai);
    }
}
```

## 14. List&Set&数据结构&Collections

### 14.1 数据结构

#### 14.1.1 数据结构作用

当你用着java里面的容器类很爽的时候，你有没有想过，怎么ArrayList就像一个无限扩充的数组，也好像链表之类的。好用吗？好用，这就是数据结构的用处，只不过你在不知不觉中使用了。

现实世界的存储，我们使用的工具和建模。每种数据结构有自己的优点和缺点，想想如果Google的数据用的是数组的存储，我们还能方便地查询到所需要的数据吗？而算法，在这么多的数据中如何做到最快的插入，查找，删除，也是在追求更快。

我们java是面向对象的语言，就好似自动档轿车，C语言好似手动档吉普。数据结构呢？是变速箱的工作原理。你完全可以不知道变速箱怎样工作，就把自动档的车子从 A点 开到 B点，而且未必就比懂得的人慢。写程序这件事，和开车一样，经验可以起到很大作用，但如果你不知道底层是怎么工作的，就永远只能开车，既不会修车，也不能造车。当然了，数据结构内容比较多，细细的学起来也是相对费功夫的，不可能达到一蹴而就。我们将常见的数据结构：堆栈、队列、数组、链表和红黑树 这几种给大家介绍一下，作为数据结构的入门，了解一下它们的特点即可。

#### 14.1.2 常见的数据结构

数据存储的常用结构有：栈、队列、数组、链表和红黑树。我们分别来了解一下：

**栈**

栈：stack,又称堆栈，它是运算受限的线性表，其限制是仅允许在标的一端进行插入和删除操作，不允许在其他任何位置进行添加、查找、删除等操作。

简单的说：采用该结构的集合，对元素的存取有如下的特点

* 先进后出（即，存进去的元素，要在后它后面的元素依次取出后，才能取出该元素）。例如，子弹压进弹夹，先压进去的子弹在下面，后压进去的子弹在上面，当开枪时，先弹出上面的子弹，然后才能弹出下面的子弹。
* 栈的入口、出口的都是栈的顶端位置。

![栈](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509888.PNG)

这里两个名词需要注意：

* **压栈**：就是存元素。即，把元素存储到栈的顶端位置，栈中已有元素依次向栈底方向移动一个位置。
* **弹栈**：就是取元素。即，把栈的顶端位置元素取出，栈中已有元素依次向栈顶方向移动一个位置。

**队列**

队列：queue，简称队，它同堆栈一样，也是一种运算受限的线性表，其限制是仅允许在表的一端进行插入，而在表的另一端进行删除。

简单的说，采用该结构的集合，对元素的存取有如下的特点：

* 先进先出（即，存进去的元素，要在后它前面的元素依次取出后，才能取出该元素）。例如，小火车过山洞，车头先进去，车尾后进去；车头先出来，车尾后出来。
* 队列的入口、出口各占一侧。例如，下图中的左侧为入口，右侧为出口。

![队列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509319.PNG)

**数组**

数组：Array，是有序的元素序列，数组是在内存中开辟一段连续的空间，并在此空间存放元素。就像是一排出租屋，有100个房间，从001到100每个房间都有固定编号，通过编号就可以快速找到租房子的人。

简单的说,采用该结构的集合，对元素的存取有如下的特点：

* 查找元素快：通过索引，可以快速访问指定位置的元素

![数组](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509311.PNG)

* 增删元素慢

  * **指定索引位置增加元素**：需要创建一个新数组，将指定新元素存储在指定索引位置，再把原数组元素根据索引，复制到新数组对应索引的位置。如下图

    ![指定索引位置增加元素](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251509863.PNG)

  * **指定索引位置删除元素**：需要创建一个新数组，把原数组元素根据索引，复制到新数组对应索引的位
    置，原数组中指定索引位置元素不复制到新数组中。如下图

    ![指定索引位置删除元素](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510671.PNG)

**链表**

链表：linked list，由一系列结点node（链表中每一个元素称为结点）组成，结点可以在运行时动态生成。每个结点包括两个部分：一个是存储数据元素的数据域，另一个是存储下一个结点地址的指针域。我们常说的链表结构有单向链表与双向链表，那么这里给大家介绍的是**单向链表**。

![链表](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510029.PNG)

简单的说，采用该结构的集合，对元素的存取有如下的特点：

* **多个结点之间，通过地址进行连接**。例如，多个人手拉手，每个人使用自己的右手拉住下个人的左手，依次类推，这样多个人就连在一起了。

  ![多个结点之间，通过地址进行连接](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510705.PNG)

* 查找元素慢：想查找某个元素，需要通过连接的节点，依次向后查找指定元素

* 增删元素快：

  * 增加元素：只需要修改连接下个元素的地址即可。

    ![增加元素](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510191.PNG)

  * 删除元素：只需要修改连接下个元素的地址即可。

    ![删除元素](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510735.PNG)

**红黑树**

二叉树：binary tree，是每个结点不超过2的有序树（tree）。

简单的理解，就是一种类似于我们生活中树的结构，只不过每个结点上都最多只能有两个子结点。

二叉树是每个节点最多有两个子树的树结构。顶上的叫根结点，两边被称作“左子树”和“右子树”。

如图：

![二叉树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510592.PNG)

我们要说的是二叉树的一种比较有意思的叫做红黑树，红黑树本身就是一颗二叉查找树，将节点插入后，该树仍然是一颗二叉查找树。也就意味着，树的键值仍然是有序的。

**红黑树的约束:**

1. 节点可以是红色的或者黑色的
2. 根节点是黑色的
3. 叶子节点(特指空节点)是黑色的
4. 每个红色节点的子节点都是黑色的
5. 任何一个节点到其每一个叶子节点的所有路径上黑色节点数相同

**红黑树的特点:**

速度特别快,趋近平衡树,查找叶子元素最少和最多次数不多于二倍

### 14.2 List集合

我们掌握了Collection接口的使用后，再来看看Collection接口中的子类，他们都具备那些特性呢？

接下来，我们一起学习Collection中的常用几个子类（ `java.util.List` 集合、 `java.util.Set` 集合）。

#### 14.2.1 List接口介绍

`java.util.List` 接口继承自 Collection 接口，是单列集合的一个重要分支，习惯性地会将实现了 List 接口的对象称为List集合。在List集合中允许出现重复的元素，所有的元素是以一种线性方式进行存储的，在程序中可以通过索引来访问集合中的指定元素。另外，List集合还有一个特点就是元素有序，即元素的存入顺序和取出顺序一致。

看完API，我们总结一下：

List接口特点：

1. 它是一个元素存取有序的集合。例如，存元素的顺序是11、22、33。那么集合中，元素的存储就是按照11、22、33的顺序完成的）。
2. 它是一个带有索引的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。
3. 集合中可以有重复的元素，通过元素的equals方法，来比较是否为重复的元素。

> tips:我们在基础班的时候已经学习过List接口的子类java.util.ArrayList类，该类中的方法都是来自List中定义。

#### 14.2.2 List接口中常用方法

List作为Collection集合的子接口，不但继承了Collection接口中的全部方法，而且还增加了一些根据元素索引来操作集合的特有方法，如下：

* `public void add(int index, E element)` : 将指定的元素，添加到该集合中的指定位置上。
* `public E get(int index)` :返回集合中指定位置的元素。
* `public E remove(int index)` : 移除列表中指定位置的元素, 返回的是被移除的元素。
* `public E set(int index, E element)` :用指定元素替换集合中指定位置的元素,返回值的更新前的元素。

List集合特有的方法都是跟索引相关，我们在基础班都学习过，那么我们再来复习一遍吧：

```java
public class ListDemo {
    public static void main(String[] args) {
        // 创建List集合对象
        List<String> list = new ArrayList<String>();
        // 往 尾部添加 指定元素
        list.add("图图");
        list.add("小美");
        list.add("不高兴");
        System.out.println(list);
        // add(int index,String s) 往指定位置添加
        list.add(1,"没头脑");
        System.out.println(list);
        // String remove(int index) 删除指定位置元素 返回被删除元素
        // 删除索引位置为2的元素
        System.out.println("删除索引位置为2的元素");
        System.out.println(list.remove(2));
        System.out.println(list);
        // String set(int index,String s)
        // 在指定位置 进行 元素替代（改）
        // 修改指定位置元素
        list.set(0, "三毛");
        System.out.println(list);
        // String get(int index) 获取指定位置元素
        // 跟size() 方法一起用 来 遍历的
        for(int i = 0;i<list.size();i++){
            System.out.println(list.get(i));
        } 
        //还可以使用增强for
        for (String string : list) {
            System.out.println(string);
        }
    }
}
```

### 14.3 List的子类

#### 14.3.1 ArrayList集合

`java.util.ArrayList` 集合数据存储的结构是数组结构。元素增删慢，查找快，由于日常开发中使用最多的功能为查询数据、遍历数据，所以 ArrayList 是最常用的集合。

许多程序员开发时非常随意地使用ArrayList完成任何需求，并不严谨，这种用法是不提倡的。

#### 14.3.2 LinkedList集合

`java.util.LinkedList` 集合数据存储的结构是链表结构。方便元素添加、删除的集合。

LinkedList是一个双向链表，那么双向链表是什么样子的呢，我们用个图了解下：

![LinkedList](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510125.PNG)

实际开发中对一个集合元素的添加与删除经常涉及到首尾操作，而LinkedList提供了大量首尾操作的方法。这些方法我们作为了解即可：

* `public void addFirst(E e)` :将指定元素插入此列表的开头。
* `public void addLast(E e)` :将指定元素添加到此列表的结尾。
* `public E getFirst()` :返回此列表的第一个元素。
* `public E getLast()` :返回此列表的最后一个元素。
* `public E removeFirst()` :移除并返回此列表的第一个元素。
* `public E removeLast()` :移除并返回此列表的最后一个元素。
* `public E pop() `:从此列表所表示的堆栈处弹出一个元素。
* `public void push(E e)` :将元素推入此列表所表示的堆栈。
* `public boolean isEmpty()`：如果列表不包含元素，则返回true。

LinkedList是List的子类，List中的方法LinkedList都是可以使用，这里就不做详细介绍，我们只需要了解LinkedList的特有方法即可。在开发时，LinkedList集合也可以作为堆栈，队列的结构使用。

方法演示：

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> link = new LinkedList<String>();
        //添加元素
        link.addFirst("abc1");
        link.addFirst("abc2");
        link.addFirst("abc3");
        System.out.println(link);
        // 获取元素
        System.out.println(link.getFirst());
        System.out.println(link.getLast());
        // 删除元素
        System.out.println(link.removeFirst());
        System.out.println(link.removeLast());
        while (!link.isEmpty()) { //判断集合是否为空
            System.out.println(link.pop()); //弹出集合中的栈顶元素
        } 
        System.out.println(link);
    }
}
```

### 14.4 Set接口

`java.util.Set` 接口和 `java.util.List` 接口一样，同样继承自 Collection 接口，它与 Collection 接口中的方法基本一致，并没有对 Collection 接口进行功能上的扩充，只是比 Collection 接口更加严格了。与 List 接口不同的是， **Set 接口中元素无序，并且都会以某种规则保证存入的元素不出现重复。**Set 集合有多个子类，这里我们介绍其中的 `java.util.HashSet` 、 `java.util.LinkedHashSet` 这两个集合。

> tips:Set集合取出元素的方式可以采用：迭代器、增强for。

#### 14.4.1 HashSet集合介绍

`java.util.HashSet` 是 Set 接口的一个实现类，它所存储的元素是不可重复的，并且元素都是无序的(即存取顺序不一致)。 `java.util.HashSet` 底层的实现其实是一个 `java.util.HashMap` 支持，由于我们暂时还未学习，先做了解。

HashSet 是根据对象的哈希值来确定元素在集合中的存储位置，因此具有良好的存取和查找性能。保证元素唯一性的方式依赖于： hashCode 与 equals 方法。

我们先来使用一下Set集合存储，看下现象，再进行原理的讲解:

```java
public class HashSetDemo {
    public static void main(String[] args) {
        //创建 Set集合
        HashSet<String> set = new HashSet<String>();
        //添加元素
        set.add(new String("cba"));
        set.add("abc");
        set.add("bac");
        set.add("cba");
        //遍历
        for (String name : set) {
            System.out.println(name);
        }
    }
}
```

输出结果如下，说明集合中不能存储重复元素：

```
cba
abc
bac
```

> tips:根据结果我们发现字符串"cba"只存储了一个，也就是说重复的元素set集合不存储。

#### 14.4.2 HashSet存储结构

什么是哈希表呢？

在JDK1.8之前，哈希表底层采用**数组+链表**实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。

但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，哈希表存储采用**数组+链表+红黑树**实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

简单的来说，**哈希表是由数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的**，如下图所示。

![哈希表](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510311.PNG)

看到这张图就有人要问了，这个是怎么存储的呢？

为了方便大家的理解我们结合一个存储流程图来说明一下：

![HashSet存储原理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510777.PNG)

总而言之，JDK1.8引入红黑树大程度优化了HashMap的性能，那么对于我们来讲保证HashSet集合元素的唯一，其实就是根据对象的hashCode和equals方法来决定的。**如果我们往集合中存放自定义的对象，那么保证其唯一，就必须复写hashCode和equals方法建立属于当前对象的比较方式**。

#### 14.4.3 HashSet存储自定义类型元素

给HashSet中存放自定义类型元素时，需要重写对象中的hashCode和equals方法，建立自己的比较方式，才能保证HashSet集合中的对象唯一。

创建自定义Student类

```java
public class Student {
    private String name;
    private int age;
    public Student() {
    } 
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    } public String getName() {
        return name;
    } 
    public void setName(String name) {
        this.name = name;
    } 
    public int getAge() {
        return age;
    } 
    public void setAge(int age) {
        this.age = age;
    } 
    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Student student = (Student) o;
        return age == student.age &&
            Objects.equals(name, student.name);
    } 
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

```java
public class HashSetDemo2 {
    public static void main(String[] args) {
        //创建集合对象 该集合中存储 Student类型对象
        HashSet<Student> stuSet = new HashSet<Student>();
        //存储
        Student stu = new Student("于谦", 43);
        stuSet.add(stu);
        stuSet.add(new Student("郭德纲", 44));
        stuSet.add(new Student("于谦", 43));
        stuSet.add(new Student("郭麒麟", 23));
        stuSet.add(stu);
        for (Student stu2 : stuSet) {
            System.out.println(stu2);
        }
    }
} 

// 执行结果：
// Student [name=郭德纲, age=44]
// Student [name=于谦, age=43]
// Student [name=郭麒麟, age=23]
```

#### 14.4.4 LinkedHashSet

我们知道HashSet保证元素唯一，可是元素存放进去是没有顺序的，那么我们要保证有序，怎么办呢？

在HashSet下面有一个子类 `java.util.LinkedHashSet` ，它是**链表和哈希表组合**的一个数据存储结构。

演示代码如下:

```java
public class LinkedHashSetDemo {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<String>();
        set.add("bbb");
        set.add("aaa");
        set.add("abc");
        set.add("bbc");
        Iterator<String> it = set.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
    }
} 

// 结果：
// bbb
// aaa
// abc
// bbc
```

#### 14.4.5 可变参数

在JDK1.5之后，如果我们定义一个方法需要接受多个参数，并且多个参数类型一致，我们可以对其简化成如下格式：

```java
修饰符 返回值类型 方法名(参数类型... 形参名){ }
```

其实这个书写完全等价于

```java
修饰符 返回值类型 方法名(参数类型[] 形参名){ }
```

只是后面这种定义，在调用时必须传递数组，而前者可以直接传递数据即可。

JDK1.5以后。出现了简化操作。**`...` 用在参数上，称之为可变参数**。

同样是代表数组，但是在调用这个带有可变参数的方法时，不用创建数组(这就是简单之处)，直接将数组中的元素作为实际参数进行传递，其实编译成的class文件，将这些元素先封装到一个数组中，在进行传递。这些动作都在编译.class文件时，自动完成了。

代码演示：

```java
public class ChangeArgs {
    public static void main(String[] args) {
        int[] arr = { 1, 4, 62, 431, 2 };
        int sum = getSum(arr);
        System.out.println(sum);
        // 6 7 2 12 2121
        // 求 这几个元素和 6 7 2 12 2121
        int sum2 = getSum(6, 7, 2, 12, 2121);
        System.out.println(sum2);
    } 
    /*
    //完成数组 所有元素的求和 原始写法
    public static int getSum(int[] arr){
        int sum = 0;
        for(int a : arr){
            sum += a;
        } 
        return sum;
    }
    */

    //可变参数写法
    public static int getSum(int... arr) {
        int sum = 0;
        for (int a : arr) {
            sum += a;
        } 
        return sum;
    }
}
```

> tips: 上述add方法在同一个类中，只能存在一个。因为会发生调用的不确定性。
>
> 注意：如果在方法书写时，这个方法拥有多参数，参数中包含可变参数，**可变参数一定要写在参数列表的末尾位置**。

### 14.5 Collections

#### 14.5.1 常用功能

`java.utils.Collections` 是集合工具类，用来对集合进行操作。部分方法如下：

* `public static <T> boolean addAll(Collection<T> c, T... elements)` :往集合中添加一些元素。
* `public static void shuffle(List<?> list)` 打乱顺序 :打乱集合顺序。
* `public static <T> void sort(List<T> list) `:将集合中元素按照默认规则排序。
* `public static <T> void sort(List<T> list，Comparator<? super T> )` :将集合中元素按照指定规则排序。

代码演示：

```java
public class CollectionsDemo {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        //原来写法
        //list.add(12);
        //list.add(14);
        //list.add(15);
        //list.add(1000);
        //采用工具类 完成 往集合中添加元素
        Collections.addAll(list, 5, 222, 1，2);
        System.out.println(list);
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
} 
// 结果：
// [5, 222, 1, 2]
// [1, 2, 5, 222]
```

代码演示之后 ，发现我们的集合按照顺序进行了排列，可是这样的顺序是采用默认的顺序，如果想要指定顺序那该怎么办呢？

我们发现还有个方法没有讲， `public static <T> void sort(List<T> list，Comparator<? super T> )` :将集合中元素按照指定规则排序。接下来讲解一下指定规则的排列。

#### 14.5.2 Comparator比较器

我们还是先研究这个方法

`public static <T> void sort(List<T> list)` :将集合中元素按照默认规则排序。

不过这次存储的是字符串类型。

```java
public class CollectionsDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
}

// 结果：
// [aba, cba, nba, sba]
```

我们使用的是默认的规则完成字符串的排序，那么默认规则是怎么定义出来的呢？ 

说到排序了，简单的说就是两个对象之间比较大小，那么在JAVA中提供了两种比较实现的方式，一种是比较死板的采用 `java.lang.Comparable` 接口去实现，一种是灵活的当我需要做排序的时候在去选择的`java.util.Comparator` 接口完成。

那么我们采用的 `public static <T> void sort(List<T> list)` 这个方法完成的排序，**实际上要求了被排序的类型需要实现Comparable接口完成比较的功能**，在String类型上如下：

```java
public final class String 
    implements java.io.Serializable, Comparable<String>, CharSequence {
    // ...
}
```

String类实现了这个接口，并完成了比较规则的定义，但是这样就把这种规则写死了，那比如我想要字符串按照第一个字符降序排列，那么这样就要修改String的源代码，这是不可能的了。

```java
//重写排序的规则
@Override
public int compareTo(Person o) {
    //return 0;//认为元素都是相同的
    //自定义比较的规则,比较两个人的年龄(this,参数Person)
    //return this.getAge() - o.getAge();//年龄升序排序
    return o.getAge() - this.getAge();//年龄升序排序
}
```

那么这个时候我们可以使用`public static <T> void sort(List<T> list，Comparator<? super T> )` 方法灵活的完成，这个里面就涉及到了**Comparator这个接口**，位于位于java.util包下，排序是comparator能实现的功能之一，该接口代表一个比较器，比较器具有可比性！顾名思义就是做排序的，通俗地讲需要比较两个对象谁排在前谁排在后，那么比较的方法就是：`public int compare(String o1, String o2)` ：比较其两个参数的顺序。

两个对象比较的结果有三种：大于，等于，小于。

* 如果要按照升序排序，则o1 小于o2，返回（负数），相等返回0，01大于02返回（正数）; 

* 如果要按照降序排序，则o1 小于o2，返回（正数），相等返回0，01大于02返回（负数）；

> Comparator的排序规则: o1-o2:升序；o2-o1降序

操作如下:

```java
public class CollectionsDemo3 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法 按照第一个单词的降序
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.charAt(0) ‐ o1.charAt(0);
            }
        });
        System.out.println(list);
    }
}

// 结果如下：
// [sba, nba, cba, aba]
```

#### 14.5.3 Comparable vs Comparator

**Comparable**：强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的compareTo方法被称为它的自然比较方法。只能在类中实现`compareTo()`一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**Comparator**：强行对某个对象进行整体排序。可以将Comparator 传递给sort方法（如Collections.sort或Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用Comparator来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象collection提供排序。

#### 14.5.4 练习

创建一个学生类，存储到ArrayList集合中完成指定排序操作。

Student 初始类

```java
public class Student{
    private String name;
    private int age;
    public Student() {
    } 
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    } 
    public String getName() {
        return name;
    } 
    public void setName(String name) {
        this.name = name;
    } 
    public int getAge() {
        return age;
    } 
    public void setAge(int age) {
        this.age = age;
    } 
    @Override
    public String toString() {
        return "Student{" +
            "name='" + name + '\'' +
            ", age=" + age +
            '}';
    }
}
```

测试类：

```java
public class Demo {
    public static void main(String[] args) {
        // 创建四个学生对象 存储到集合中
        ArrayList<Student> list = new ArrayList<Student>();
        list.add(new Student("rose",18));
        list.add(new Student("jack",16));
        list.add(new Student("abc",16));
        list.add(new Student("ace",17));
        list.add(new Student("mark",16));
        /*
         让学生 按照年龄排序 升序
         */
        // Collections.sort(list);//要求 该list中元素类型 必须实现比较器Comparable接口
        for (Student student : list) {
            System.out.println(student);
        }
    }
}
```

发现，当我们调用`Collections.sort()`方法的时候 程序报错了。

原因：如果想要集合中的元素完成排序，那么必须要实现比较器Comparable接口。

于是我们就完成了Student类的一个实现，如下：

```java
public class Student implements Comparable<Student>{
    	....
        @Override
        public int compareTo(Student o) {
        return this.age‐o.age;//升序
    }
}
```

再次测试，代码就OK 了效果如下：

```java
Student{name='jack', age=16}
Student{name='abc', age=16}
Student{name='mark', age=16}
Student{name='ace', age=17}
Student{name='rose', age=18}
```

#### 14.5.5 扩展

如果在使用的时候，想要独立的定义规则去使用 可以采用`Collections.sort(List list,Comparetor c)`方式，自己定义规则：

```java
Collections.sort(list, new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o2.getAge()‐o1.getAge();//以学生的年龄降序
    }
});
```

效果：

```java
Student{name='rose', age=18}
Student{name='ace', age=17}
Student{name='jack', age=16}
Student{name='abc', age=16}
Student{name='mark', age=16}
```

如果想要规则更多一些，可以参考下面代码：

```java
Collections.sort(list, new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        // 年龄降序
        int result = o2.getAge()‐o1.getAge();//年龄降序
        if(result==0){//第一个规则判断完了 下一个规则 姓名的首字母 升序
            result = o1.getName().charAt(0)‐o2.getName().charAt(0);
        } 
        return result;
    }
});
```

效果如下：

```java
Student{name='rose', age=18}
Student{name='ace', age=17}
Student{name='abc', age=16}
Student{name='jack', age=16}
Student{name='mark', age=16}
```

## 15. Map

### 15.1 Map集合

#### 15.1.1 概述

现实生活中，我们常会看到这样的一种集合：IP地址与主机名，身份证号与个人，系统用户名与系统用户对象等，这种一一对应的关系，就叫做映射。Java提供了专门的集合类用来存放这种对象关系的对象，即 `java.util.Map` 接口。

我们通过查看 Map 接口描述，发现 Map 接口下的集合与 Collection 接口下的集合，它们存储数据的形式不同，如下图。

![map集合](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510208.PNG)

* Collection 中的集合，元素是孤立存在的（理解为单身），向集合中存储元素采用一个个元素的方式存储。
* Map 中的集合，元素是成对存在的(理解为夫妻)。每个元素由键与值两部分组成，通过键可以找对所对应的值。
* Collection 中的集合称为单列集合， Map 中的集合称为双列集合。
* 需要注意的是， Map 中的集合不能包含重复的键，值可以重复；每个键只能对应一个值。

#### 15.1.2 Map常用子类

通过查看Map接口描述，看到Map有多个子类，这里我们主要讲解常用的HashMap集合、LinkedHashMap集合。

* **HashMap**：存储数据采用的哈希表结构，元素的存取顺序不能保证一致。由于要保证键的唯一、不重复，需要重写键的`hashCode()`方法、`equals()`方法。
* **LinkedHashMap**：HashMap下有个子类LinkedHashMap，存储数据采用的哈希表结构+链表结构。通过链表结构可以保证元素的存取顺序一致；通过哈希表结构可以保证的键的唯一、不重复，需要重写键的`hashCode()`方法、`equals()`方法。

> tips：Map接口中的集合都有两个泛型变量，在使用时，要为两个泛型变量赋予数据类型。两个泛型变量的数据类型可以相同，也可以不同。

#### 15.1.3 Map接口中的常用方法

Map接口中定义了很多方法，常用的如下：

* `public V put(K key, V value)` : 把指定的键与指定的值添加到Map集合中。
* `public V remove(Object key)` : 把指定的键所对应的键值对元素 在Map集合中删除，返回被删除元素的值。
* `public V get(Object key)` : 根据指定的键，在Map集合中获取对应的值。
* `public Set<K> keySet()` : 获取Map集合中所有的键，存储到Set集合中。
* `public Set<Map.Entry<K,V>> entrySet()` : 获取到Map集合中所有的键值对对象的集合(Set集合)。

Map接口的方法演示 :

```java
public class MapDemo {
    public static void main(String[] args) {
        //创建 map对象
        HashMap<String, String> map = new HashMap<String, String>();
        //添加元素到集合
        map.put("黄晓明", "杨颖");
        map.put("文章", "马伊琍");
        map.put("邓超", "孙俪");
        System.out.println(map);
        //String remove(String key)
        System.out.println(map.remove("邓超"));
        System.out.println(map);
        // 想要查看 黄晓明的媳妇 是谁
        System.out.println(map.get("黄晓明"));
        System.out.println(map.get("邓超"));
    }
}
```

> tips:
>
> * 使用put方法时，若指定的键(key)在集合中没有，则没有这个键对应的值，返回null，并把指定的键值添加到集合中；
>
> * 若指定的键(key)在集合中存在，则返回值为集合中键对应的值（该值为替换前的值），并把指定键所对应的值，替换成指定的新值。

#### 15.1.4 Map集合遍历键找值

键找值方式：即通过元素中的键，获取键所对应的值

分析步骤：

1. 获取Map中所有的键，由于键是唯一的，所以返回一个Set集合存储所有的键。方法提示: `keyset()`
2. 遍历键的Set集合，得到每一个键。
3. 根据键，获取键所对应的值。方法提示: `get(K key)`

代码演示：

```java
public class MapDemo01 {
    public static void main(String[] args) {
        //创建Map集合对象
        HashMap<String, String> map = new HashMap<String,String>();
        //添加元素到集合
        map.put("胡歌", "霍建华");
        map.put("郭德纲", "于谦");
        map.put("薛之谦", "大张伟");
        //获取所有的键 获取键集
        Set<String> keys = map.keySet();
        // 遍历键集 得到 每一个键
        for (String key : keys) {
            //key 就是键
            //获取对应值
            String value = map.get(key);
            System.out.println(key+"的CP是："+value);
        }
    }
}
```

遍历图解：

![map遍历](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510360.PNG)

#### 15.1.5 Entry键值对对象

我们已经知道， Map 中存放的是两种对象，一种称为**key(键)**，一种称为**value(值)**，它们在在 Map 中是一一对应关系，这一对对象又称做 Map 中的一个 **Entry(项)** 。 Entry 将键值对的对应关系封装成了对象。即键值对对象，这样我们在遍历 Map 集合时，就可以从每一个键值对（ Entry ）对象中获取对应的键与对应的值。

既然Entry表示了一对键和值，那么也同样提供了获取对应键和对应值得方法：

* `public K getKey()` ：获取Entry对象中的键。
* `public V getValue()` ：获取Entry对象中的值。

在Map集合中也提供了获取所有Entry对象的方法：

* `public Set<Map.Entry<K,V>> entrySet()` : 获取到Map集合中所有的键值对对象的集合(Set集合)。

#### 15.1.6 Map集合遍历键值对方式

键值对方式：即通过集合中每个键值对(Entry)对象，获取键值对(Entry)对象中的键与值。

操作步骤与图解：

1. 获取Map集合中，所有的键值对(Entry)对象，以Set集合形式返回。方法提示: `entrySet()` 。
2. 遍历包含键值对(Entry)对象的Set集合，得到每一个键值对(Entry)对象。


3. 通过键值对(Entry)对象，获取Entry对象中的键与值。 方法提示: `getkey() & getValue()`

```java
public class MapDemo02 {
    public static void main(String[] args) {
        // 创建Map集合对象
        HashMap<String, String> map = new HashMap<String,String>();
        // 添加元素到集合
        map.put("胡歌", "霍建华");
        map.put("郭德纲", "于谦");
        map.put("薛之谦", "大张伟");
        // 获取 所有的 entry对象 entrySet
        Set<Entry<String,String>> entrySet = map.entrySet();
        // 遍历得到每一个entry对象
        for (Entry<String, String> entry : entrySet) {
            // 解析
            String key = entry.getKey();
            String value = entry.getValue();
            System.out.println(key+"的CP是:"+value);
        }
    }
}
```

遍历图解：

![MapEntry遍历](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510659.PNG)

> tips：Map集合不能直接使用迭代器或者foreach进行遍历。但是转成Set之后就可以使用了。

#### 15.1.7 HashMap存储自定义类型键值

练习：每位学生（姓名，年龄）都有自己的家庭住址。那么，既然有对应关系，则将学生对象和家庭住址存储到map集合中。学生作为键, 家庭住址作为值。

> 注意，学生姓名相同并且年龄相同视为同一名学生。

编写学生类：

```java
public class Student {
    private String name;
    private int age;
    public Student() {
    } 
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    } 
    public String getName() {
        return name;
    } 
    public void setName(String name) {
        this.name = name;
    } 
    public int getAge() {
        return age;
    } 
    public void setAge(int age) {
        this.age = age;
    } 
    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    } 
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

编写测试类：

```java
public class HashMapTest {
    public static void main(String[] args) {
        //1,创建Hashmap集合对象。
        Map<Student,String>map = new HashMap<Student,String>();
        //2,添加元素。
        map.put(newStudent("lisi",28), "上海");
        map.put(newStudent("wangwu",22), "北京");
        map.put(newStudent("zhaoliu",24), "成都");
        map.put(newStudent("zhouqi",25), "广州");
        map.put(newStudent("wangwu",22), "南京");
        //3,取出元素。键找值方式
        Set<Student>keySet = map.keySet();
        for(Student key: keySet){
            Stringvalue = map.get(key);
            System.out.println(key.toString()+"....."+value);
        }
    }
}
```

当给HashMap中存放自定义对象时，如果自定义对象作为key存在，这时要保证对象唯一，必须复写对象的hashCode和equals方法(如果忘记，请回顾HashSet存放自定义对象)。

如果要保证map中存放的key和取出的顺序一致，可以使用 `java.util.LinkedHashMap` 集合来存放。

#### 15.1.8 LinkedHashMap

我们知道HashMap保证成对元素唯一，并且查询速度很快，可是成对元素存放进去是没有顺序的，那么我们要保证有序，还要速度快怎么办呢？

在HashMap下面有一个子类LinkedHashMap，它是链表和哈希表组合的一个数据存储结构。

```java
public class LinkedHashMapDemo {
    public static void main(String[] args) {
        LinkedHashMap<String, String> map = new LinkedHashMap<String, String>();
        map.put("邓超", "孙俪");
        map.put("李晨", "范冰冰");
        map.put("刘德华", "朱丽倩");
        Set<Entry<String, String>> entrySet = map.entrySet();
        for (Entry<String, String> entry : entrySet) {
            System.out.println(entry.getKey() + " " + entry.getValue());
        }
    }
}

// 结果:
// 邓超 孙俪
// 李晨 范冰冰
// 刘德华 朱丽倩
```

#### 15.1.9 Map集合练习

**需求：**

计算一个字符串中每个字符出现次数。

**分析：**

1. 获取一个字符串对象
2. 创建一个Map集合，键代表字符，值代表次数。

3. 遍历字符串得到每个字符。

4. 判断Map中是否有该键。
5. 如果没有，第一次出现，存储次数为1；如果有，则说明已经出现过，获取到对应的值进行++，再次存储。
6. 打印最终结果

代码：

```java
public class MapTest {
    public static void main(String[] args) {
        //友情提示
        System.out.println("请录入一个字符串:");
        String line = new Scanner(System.in).nextLine();
        // 定义 每个字符出现次数的方法
        findChar(line);
    } 
    private static void findChar(String line) {
        //1:创建一个集合 存储 字符 以及其出现的次数
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        //2:遍历字符串
        for (int i = 0; i < line.length(); i++) {
            char c = line.charAt(i);
            //判断 该字符 是否在键集中
            if (!map.containsKey(c)) {//说明这个字符没有出现过
                //那就是第一次
                map.put(c, 1);
            } else {
                //先获取之前的次数
                Integer count = map.get(c);
                //count++;
                //再次存入 更新
                map.put(c, ++count);
            }
        } 
        System.out.println(map);
    }
}
```

### 15.2 补充知识点

#### 15.2.1 JDK9对集合添加的优化

通常，我们在代码中创建一个集合（例如，List 或 Set ），并直接用一些元素填充它。 实例化集合，几个 add方法调用，使得代码重复。

```java
public class Demo01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("abc");
        list.add("def");
        list.add("ghi");
        System.out.println(list);
    }
}
```

Java 9，添加了几种**集合工厂方法**,更方便创建少量元素的集合、map实例。

新的List、Set、Map的静态工厂方法可以更方便地创建集合的**不可变实例**。

例子：

```java
public class HelloJDK9 {
    public static void main(String[] args) {
        Set<String> str1=Set.of("a","b","c");
        //str1.add("c");这里编译的时候不会错，但是执行的时候会报错，因为是不可变的集合
        System.out.println(str1);
        
        Map<String,Integer> str2=Map.of("a",1,"b",2);
        System.out.println(str2);
        
        List<String> str3=List.of("a","b");
        System.out.println(str3);
    }
}
```

需要注意以下两点：

1. `of()`方法只是Map，List，Set这三个接口的静态方法，其父类接口和子类实现并没有这类方法，比如HashSet，ArrayList等待；
2. **返回的集合是不可变的**；

#### 15.2.2 Debug追踪

暂略...

### 15.3 模拟斗地主洗牌发牌

#### 15.3.1 案例介绍

按照斗地主的规则，完成洗牌发牌的动作。

![斗地主](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510365.PNG)

具体规则：

1. 组装54张扑克牌将
2. 54张牌顺序打乱
3. 三个玩家参与游戏，三人交替摸牌，每人17张牌，最后三张留作底牌。
4. 查看三人各自手中的牌（按照牌的大小排序）、底牌

> 规则：手中扑克牌从大到小的摆放顺序：大王,小王,2,A,K,Q,J,10,9,8,7,6,5,4,3

#### 15.3.2 案例需求分析

1. 准备牌：

   完成数字与纸牌的映射关系：

   使用双列Map(HashMap)集合，完成一个数字与字符串纸牌的对应关系(相当于一个字典)。

2. 洗牌：

   通过数字完成洗牌发牌

3. 发牌：

   将每个人以及底牌设计为ArrayList，将最后3张牌直接存放于底牌，剩余牌通过对3取模依次发牌。

   存放的过程中要求数字大小与斗地主规则的大小对应。

   将代表不同纸牌的数字分配给不同的玩家与底牌。

4. 看牌：

   通过Map集合找到对应字符展示。

   通过查询纸牌与数字的对应关系，由数字转成纸牌字符串再进行展示。

![斗地主2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251510198.PNG)

### 15.3.3 实现代码步骤

```java
public class Poker {
    public static void main(String[] args) {
        /*
         * 1组装54张扑克牌
         */
        // 1.1 创建Map集合存储
        HashMap<Integer, String> pokerMap = new HashMap<Integer, String>();
        // 1.2 创建 花色集合 与 数字集合
        ArrayList<String> colors = new ArrayList<String>();
        ArrayList<String> numbers = new ArrayList<String>();
        // 1.3 存储 花色 与数字
        Collections.addAll(colors, "♦", "♣", "♥", "♠");
        Collections.addAll(numbers, "2", "A", "K", "Q", "J", "10", "9", "8", "7", "6", "5", "4", "3");
        // 设置 存储编号变量
        int count = 1;
        pokerMap.put(count++, "大王");
        pokerMap.put(count++, "小王");
        // 1.4 创建牌 存储到map集合中
        for (String number : numbers) {
            for (String color : colors) {
                String card = color + number;
                pokerMap.put(count++, card);
            }
        } 
        /**
          * 2 将54张牌顺序打乱
          */
        // 取出编号 集合
        Set<Integer> numberSet = pokerMap.keySet();
        // 因为要将编号打乱顺序 所以 应该先进行转换到 list集合中
        ArrayList<Integer> numberList = new ArrayList<Integer>();
        numberList.addAll(numberSet);
        // 打乱顺序
        Collections.shuffle(numberList);

        // 3 完成三个玩家交替摸牌，每人17张牌，最后三张留作底牌
        // 3.1 发牌的编号
        // 创建三个玩家编号集合 和一个 底牌编号集合
        ArrayList<Integer> noP1 = new ArrayList<Integer>();
        ArrayList<Integer> noP2 = new ArrayList<Integer>();
        ArrayList<Integer> noP3 = new ArrayList<Integer>();
        ArrayList<Integer> dipaiNo = new ArrayList<Integer>();
        // 3.2发牌的编号
        for (int i = 0; i < numberList.size(); i++) {
            // 获取该编号
            Integer no = numberList.get(i);
            // 发牌
            // 留出底牌
            if (i >= 51) {
                dipaiNo.add(no);
            } else {
                if (i % 3 == 0) {
                    noP1.add(no);
                } else if (i % 3 == 1) {
                    noP2.add(no);
                } else {
                    noP3.add(no);
                }
            }
        } 
        // 4 查看三人各自手中的牌（按照牌的大小排序）、底牌
        // 4.1 对手中编号进行排序
        Collections.sort(noP1);
        Collections.sort(noP2);
        Collections.sort(noP3);
        Collections.sort(dipaiNo);
        // 4.2 进行牌面的转换
        // 创建三个玩家牌面集合 以及底牌牌面集合
        ArrayList<String> player1 = new ArrayList<String>();
        ArrayList<String> player2 = new ArrayList<String>();
        ArrayList<String> player3 = new ArrayList<String>();
        ArrayList<String> dipai = new ArrayList<String>();
        // 4.3转换
        for (Integer i : noP1) {
            // 4.4 根据编号找到 牌面 pokerMap
            String card = pokerMap.get(i);
            // 添加到对应的 牌面集合中
            player1.add(card);
        } 
        for (Integer i : noP2) {
            String card = pokerMap.get(i);
            player2.add(card);

        } 
        for (Integer i : noP3) {
            String card = pokerMap.get(i);
            player3.add(card);
        } f
            or (Integer i : dipaiNo) {
            String card = pokerMap.get(i);
            dipai.add(card);
        } 
        //4.5 查看
        System.out.println("令狐冲："+player1);
        System.out.println("石破天："+player2);
        System.out.println("鸠摩智："+player3);
        System.out.println("底牌："+dipai);
    }
}
```

