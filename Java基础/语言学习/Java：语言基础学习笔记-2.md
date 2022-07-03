# Java：语言基础学习笔记-2

> 时间紧任务重，此文作为后端学习基础笔记的第二篇（总共六篇）

## 07. Scanner&Random&ArrayList

### 7.1 API

#### 7.1.1 概述

API(Application Programming Interface)，应用程序编程接口。Java API是一本程序员的字典 ，是JDK中提供给我们使用的类的说明文档。这些类将底层的代码实现封装了起来，我们不需要关心这些类是如何实现的，只需要学习这些类如何使用即可。所以我们可以通过查询API的方式，来学习Java提供的类，并得知如何使用它们。

#### 7.1.2 API使用步骤

1. 打开帮助文档。
2. 点击显示，找到索引，看到输入框。
3. 你要找谁？在输入框里输入，然后回车。
4. 看包。java.lang下的类不需要导包，其他需要。
5. 看类的解释和说明。
6. 学习构造方法。


7. 使用成员方法。

### 7.2 Scanner类

了解了API的使用方式，我们通过Scanner类，熟悉一下查询API，并使用类的步骤。

#### 7.2.1 什么是Scanner类

一个可以解析基本类型和字符串的简单文本扫描器。 例如，以下代码使用户能够从 System.in 中读取一个数：

```java
Scanner sc = new Scanner(System.in);
int i = sc.nextInt();
```

> 备注：System.in 系统输入指的是通过键盘录入数据。

#### 7.2.2 引用类型使用步骤

**导包**

使用import关键字导包，在类的所有代码之前导包，引入要使用的类型，java.lang包下的所有类无需导入。 格式：`import 包名.类名;`

举例：`java.util.Scanner;`

**创建对象**

使用该类的构造方法，创建一个该类的对象。 格式：`数据类型 变量名 = new 数据类型(参数列表);`

举例：`Scanner sc = new Scanner(System.in);`

**调用方法**

调用该类的成员方法，完成指定功能。 格式：`变量名.方法名();`

举例：`int i = sc.nextInt(); // 接收一个键盘录入的整数`

#### 7.2.3 Scanner使用步骤

**查看类**

`java.util.Scanner` ：该类需要import导入后使用。

**查看构造方法**

`public Scanner(InputStream source)` : 构造一个新的 Scanner ，它生成的值是从指定的输入流扫描的。

**查看成员方法**

`public int nextInt() `：将输入信息的下一个标记扫描为一个 int 值。

使用Scanner类，完成接收键盘录入数据的操作，代码如下：

```java
//1. 导包
import java.util.Scanner;

public class Demo01_Scanner {
    public static void main(String[] args) {
        //2. 创建键盘录入数据的对象
        Scanner sc = new Scanner(System.in);
        //3. 接收数据
        System.out.println("请录入一个整数：");
        int i = sc.nextInt();
        //4. 输出数据
        System.out.println("i:"+i);
    }
}
```

#### 7.2.4 练习

**求和**：键盘录入两个数据并求和，代码如下：

```java
import java.util.Scanner;
public class Test01Scanner {
    public static void main(String[] args) {
        // 创建对象
        Scanner sc = new Scanner(System.in);
        // 接收数据
        System.out.println("请输入第一个数据：");
        int a = sc.nextInt();
        System.out.println("请输入第二个数据：");
        int b = sc.nextInt();
        // 对数据进行求和
        int sum = a + b;
        System.out.println("sum:" + sum);
    }
}
```

**取最值**：键盘录入三个数据并获取最大值，代码如下：

```java
import java.util.Scanner;
public class Test02Scanner {
    public static void main(String[] args) {
        // 创建对象
        Scanner sc = new Scanner(System.in);
        // 接收数据
        System.out.println("请输入第一个数据：");
        int a = sc.nextInt();
        System.out.println("请输入第二个数据：");
        int b = sc.nextInt();
        System.out.println("请输入第三个数据：");
        int c = sc.nextInt();
        // 如何获取三个数据的最大值
        int temp = (a > b ? a : b);
        int max = (temp > c ? temp : c);
        System.out.println("max:" + max);
    }
}
```

#### 7.2.5 匿名对象

**概念**：创建对象时，只有创建对象的语句，却没有把对象地址值赋值给某个变量。虽然是创建对象的简化写法，但是应用场景非常有限。

**匿名对象** ：没有变量名的对象。

格式：`new 类名(参数列表);`

举例：`new Scanner(System.in);`

**应用场景**

1. 创建匿名对象直接调用方法，没有变量名。

   ```java
   new Scanner(System.in).nextInt();
   ```

2. 一旦调用两次方法，就是创建了两个对象，造成浪费，请看如下代码。

   ```java
     new Scanner(System.in).nextInt();
     new Scanner(System.in).nextInt();
   ```

   > 小贴士：一个匿名对象，只能使用一次。

3. 匿名对象可以作为方法的参数和返回值

  * 作为参数：

  ```java
  class Test {
      public static void main(String[] args) {
          // 普通方式
          Scanner sc = new Scanner(System.in);
          input(sc);
          //匿名对象作为方法接收的参数
          input(new Scanner(System.in));
      } 
      public static void input(Scanner sc){
          System.out.println(sc);
      }
  }
  ```

  * 作为返回值

  ```java
  class Test2 {
      public static void main(String[] args) {
          // 普通方式
          Scanner sc = getScanner();
      } 
      public static Scanner getScanner(){
          //普通方式
          //Scanner sc = new Scanner(System.in);
          //return sc;
          //匿名对象作为方法返回值
          return new Scanner(System.in);
      }
  }
  ```


### 7.3 Random类

#### 7.3.1 什么是Random类

此类的实例用于生成**伪随机数**。

例如，以下代码使用户能够得到一个随机数：

```java
Random r = new Random();
int i = r.nextInt();
```

#### 7.3.2 Random使用步骤

**查看类**

`java.util.Random` ：该类需要 import导入使后使用。

**查看构造方法**

`public Random()` ：创建一个新的随机数生成器。

**查看成员方法**

`public int nextInt(int n)` ：返回一个伪随机数，范围在 0 （包括）和 指定值 n （不包括）之间的 int 值。

使用Random类，完成生成3个10以内的随机整数的操作，代码如下：

```java
//1. 导包
import java.util.Random;
public class Demo01_Random {
    public static void main(String[] args) {
        //2. 创建键盘录入数据的对象
        Random r = new Random();
        for(int i = 0; i < 3; i++){
            //3. 随机生成一个数据
            int number = r.nextInt(10);
            //4. 输出数据
            System.out.println("number:"+ number);
        }
    }
}
```

> 备注：创建一个 Random 对象，每次调用 nextInt() 方法，都会生成一个随机数。

#### 7.3.3 练习

**获取随机数**

获取1-n之间的随机数，包含n，代码如下：

```java
// 导包
import java.util.Random;
public class Test01Random {
    public static void main(String[] args) {
        int n = 50;
        // 创建对象
        Random r = new Random();
        // 获取随机数
        int number = r.nextInt(n) + 1;
        // 输出随机数
        System.out.println("number:" + number);
    }
}
```

**猜数字小游戏**

游戏开始时，会随机生成一个1-100之间的整数 number 。玩家猜测一个数字 guessNumber ，会与 number 作比较，系统提示大了或者小了，直到玩家猜中，游戏结束。

> 小贴士：先运行程序代码，理解此题需求，经过分析后，再编写代码

```java
// 导包
import java.util.Random;
public class Test02Random {
    public static void main(String[] args) {
        // 系统产生一个随机数1‐100之间的。
        Random r = new Random();
        int number = r.nextInt(100) + 1;
        while(true){
            // 键盘录入我们要猜的数据
            Scanner sc = new Scanner(System.in);
            System.out.println("请输入你要猜的数字(1‐100)：");
            int guessNumber = sc.nextInt();
            // 比较这两个数据(用if语句)
            if (guessNumber > number) {
                System.out.println("你猜的数据" + guessNumber + "大了");
            } else if (guessNumber < number) {
                System.out.println("你猜的数据" + guessNumber + "小了");
            } else {
                System.out.println("恭喜你,猜中了");
                break;
            }
        }
    }
}
```

### 7.4 ArrayList类

#### 7.4.1 引入：对象数组

使用学生数组，存储三个学生对象，代码如下： 

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
} 
public class Test01StudentArray {
    public static void main(String[] args) {
        //创建学生数组
        Student[] students = new Student[3];
        //创建学生对象
        Student s1 = new Student("曹操",40);
        Student s2 = new Student("刘备",35);
        Student s3 = new Student("孙权",30);
        //把学生对象作为元素赋值给学生数组
        students[0] = s1;
        students[1] = s2;
        students[2] = s3;
        //遍历学生数组
        for(int x=0; x<students.length; x++) {
            Student s = students[x];
            System.out.println(s.getName()+"‐‐‐"+s.getAge());
        }
    }
}
```

到目前为止，我们想存储对象数据，选择的容器，只有对象数组。而数组的长度是固定的，无法适应数据变化的需求。为了解决这个问题，Java提供了另一个容器 `java.util.ArrayList` 集合类,让我们可以更便捷的存储和操作对象数据。

#### 7.4.2 什么是ArrayList类 

`java.util.ArrayList` 是大小可变的数组的实现，存储在内的数据称为元素。此类提供一些方法来操作内部存储的元素。 ArrayList 中可不断添加元素，其大小也自动增长。

#### 7.4.3 ArrayList使用步骤

**查看类**

`java.util.ArrayList <E>` ：该类需要 import导入使后使用。

`<E>` ，表示一种指定的数据类型，叫做泛型。 E ，取自Element（元素）的首字母。在出现 E 的地方，我们使用一种引用数据类型将其替换即可，表示我们将存储哪种引用类型的元素。代码如下：

```java
ArrayList<String>，ArrayList<Student>
```

**查看构造方法**

`public ArrayList()` ：构造一个内容为空的集合。

**基本格式:**

```java
ArrayList<String> list = new ArrayList<String>();
```

在JDK 7后,右侧泛型的尖括号之内可以留空，但是<>仍然要写。简化格式：

```java
ArrayList<String> list = new ArrayList<>();
```

**查看成员方法**

`public boolean add(E e)` ： 将指定的元素添加到此集合的尾部。

参数 `E e` ，在构造ArrayList对象时， `<E>` 指定了什么数据类型，那么 `add(E e)` 方法中，只能添加什么数据类型的对象。

使用ArrayList类，存储三个字符串元素，代码如下：

```java
public class Test02StudentArrayList {
    public static void main(String[] args) {
        //创建学生数组
        ArrayList<String> list = new ArrayList<>();
        //创建学生对象
        String s1 = "曹操";
        String s2 = "刘备";
        String s3 = "孙权";
        //打印学生ArrayList集合
        System.out.println(list);
        //把学生对象作为元素添加到集合
        list.add(s1);
        list.add(s2);
        list.add(s3);
        //打印学生ArrayList集合
        System.out.println(list);
    }
}
```

#### 7.4.4 常用方法和遍历

对于元素的操作,基本体现在——增、删、查。常用的方法有：

* `public boolean add(E e) `：将指定的元素添加到此集合的尾部。
* `public E remove(int index)` ：移除此集合中指定位置上的元素。返回被删除的元素。
* `public E get(int index) `：返回此集合中指定位置上的元素。返回获取的元素。
* `public int size()` ：返回此集合中的元素数。遍历集合时，可以控制索引范围，防止越界。

这些都是最基本的方法，操作非常简单，代码如下:

```java
public class Demo01ArrayListMethod {
    public static void main(String[] args) {
        //创建集合对象
        ArrayList<String> list = new ArrayList<String>();
        //添加元素
        list.add("hello");
        list.add("world");
        list.add("java");
        //public E get(int index):返回指定索引处的元素
        System.out.println("get:"+list.get(0));
        System.out.println("get:"+list.get(1));
        System.out.println("get:"+list.get(2));
        //public int size():返回集合中的元素的个数
        System.out.println("size:"+list.size());
        //public E remove(int index):删除指定索引处的元素，返回被删除的元素
        System.out.println("remove:"+list.remove(0));
        //遍历输出
        for(int i = 0; i < list.size(); i++){
            System.out.println(list.get(i));
        }
    }
}
```

#### 7.4.5 如何存储基本数据类型

ArrayList对象不能存储基本类型，只能存储引用类型的数据。类似 `<int>` 不能写，但是存储基本数据类型对应的包装类型是可以的。所以，想要存储基本类型数据，`<>` 中的数据类型，必须转换后才能编写，转换写法如下：

| 基本类型 | 基本类型包装类 |
| -------- | -------------- |
| byte     | Byte           |
| short    | Short          |
| int      | Integer        |
| long     | Long           |
| float    | Float          |
| double   | Double         |
| char     | Character      |
| boolean  | Boolean        |

我们发现，只有 Integer 和 Character 需要特殊记忆，其他基本类型只是首字母大写即可。那么存储基本类型数据，代码如下：

```java
public class Demo02ArrayListMethod {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        System.out.println(list);
    }
}
```

#### 7.4.6 ArrayList练习

**数值添加到集合**：生成6个1~33之间的随机整数,添加到集合,并遍历

```java
public class Test01ArrayList {
    public static void main(String[] args) {
        // 创建Random 对象
        Random random = new Random();
        // 创建ArrayList 对象
        ArrayList<Integer> list = new ArrayList<>();
        // 添加随机数到集合
        for (int i = 0; i < 6; i++) {
            int r = random.nextInt(33) + 1;
            list.add(r);
        } 
        // 遍历集合输出
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}
```

**对象添加到集合**：自定义4个学生对象,添加到集合,并遍历

```java
public class Test02ArrayList {
    public static void main(String[] args) {
        //创建集合对象
        ArrayList<Student> list = new ArrayList<Student>();
        //创建学生对象
        Student s1 = new Student("赵丽颖",18);
        Student s2 = new Student("唐嫣",20);
        Student s3 = new Student("景甜",25);
        Student s4 = new Student("柳岩",19);
        //把学生对象作为元素添加到集合中
        list.add(s1);
        list.add(s2);
        list.add(s3);
        list.add(s4);
        //遍历集合
        for(int x = 0; x < list.size(); x++) {
            Student s = list.get(x);
            System.out.println(s.getName()+"‐‐‐"+s.getAge());
        }
    }
}
```

**打印集合方法**：定义以指定格式打印集合的方法(ArrayList类型作为参数)，使用{}扩起集合，使用@分隔每个元素。格式参照 {元素@元素@元素}。

```java
public class Test03ArrayList {
    public static void main(String[] args) {
        // 创建集合对象
        ArrayList<String> list = new ArrayList<String>();
        // 添加字符串到集合中
        list.add("张三丰");
        list.add("宋远桥");
        list.add("张无忌");
        list.add("殷梨亭");
        // 调用方法
        printArrayList(list);
    } 
    public static void printArrayList(ArrayList<String> list) {
        // 拼接左括号
        System.out.print("{");
        // 遍历集合
        for (int i = 0; i < list.size(); i++) {
            // 获取元素
            String s = list.get(i);
            // 拼接@符号
            if (i != list.size() ‐ 1) {
                System.out.print(s + "@");
            } else {
                // 拼接右括号
                System.out.print(s + "}");
            }
        }
    }
}
```

**获取集合方法**：定义获取所有偶数元素集合的方法(ArrayList类型作为返回值)

```java
public class Test04ArrayList {
    public static void main(String[] args) {
        // 创建Random 对象
        Random random = new Random();
        // 创建ArrayList 对象
        ArrayList<Integer> list = new ArrayList<>();
        // 添加随机数到集合
        for (int i = 0; i < 20; i++) {
            int r = random.nextInt(1000) + 1;
            list.add(r);
        } 
        // 调用偶数集合的方法
        ArrayList<Integer> arrayList = getArrayList(list);
        System.out.println(arrayList);
    } 
    public static ArrayList<Integer> getArrayList(ArrayList<Integer> list) {
        // 创建小集合,来保存偶数
        ArrayList<Integer> smallList = new ArrayList<>();
        // 遍历list
        for (int i = 0; i < list.size(); i++) {
            // 获取元素
            Integer num = list.get(i);
            // 判断为偶数,添加到小集合中
            if (num % 2 == 0){
                smallList.add(num);
            }
        } 
        // 返回小集合
        return smallList;
    }
} 
```

## 08. String&static&Arrays&Math

### 8.1 String类

#### 8.1.1 String类概述

**概述**：

`java.lang.String` 类代表字符串。Java程序中所有的字符串文字（例如 "abc" ）都可以被看作是实现此类的实例。

类 String 中包括用于检查各个字符串的方法，比如用于比较字符串，搜索字符串，提取子字符串以及创建具有翻译为大写或小写的所有字符的字符串的副本。

**特点**

1. 字符串不变：字符串的值在创建后不能被更改。

   ```java
   String s1 = "abc";
   s1 += "d";
   System.out.println(s1); // "abcd"
   // 内存中有"abc"，"abcd"两个对象，s1从指向"abc"，改变指向，指向了"abcd"。
   ```

2. 因为String对象是不可变的，所以它们可以被共享。

   ```java
   String s1 = "abc";
   String s2 = "abc";
   // 内存中只有一个"abc"对象被创建，同时被s1和s2共享。
   ```

3. "abc" 等效于 char[] data={ 'a' , 'b' , 'c' } 。

   ```java
   //例如：
   String str = "abc";
   //相当于：
   char data[] = {'a', 'b', 'c'};
   String str = new String(data);
   // String底层是靠字符数组实现的
   ```

#### 8.1.2 使用步骤

**查看类**

`java.lang.String` ：此类不需要导入。

**查看构造方法**

* `public String()` ：初始化新创建的 String对象，以使其表示空字符序列。
* `public String(char[] value)` ：通过当前参数中的字符数组来构造新的String。
* `public String(byte[] bytes)` ：通过使用平台的默认字符集解码当前参数中的字节数组来构造新的String。

构造举例，代码如下：

```java
// 无参构造
String str = new String();
// 通过字符数组构造
char chars[] = {'a', 'b', 'c'};
String str2 = new String(chars);
// 通过字节数组构造
byte bytes[] = { 97, 98, 99 };
String str3 = new String(bytes);
```

#### 8.1.3 常用方法

**判断功能的方法**

* `public boolean equals(Object anObject)` ：将此字符串与指定对象进行比较。
* `public boolean equalsIgnoreCase(String anotherString)` ：将此字符串与指定对象进行比较，忽略大小写。

方法演示，代码如下：

```java
public class String_Demo01 {
    public static void main(String[] args) {
        // 创建字符串对象
        String s1 = "hello";
        String s2 = "hello";
        String s3 = "HELLO";
        // boolean equals(Object obj):比较字符串的内容是否相同
        System.out.println(s1.equals(s2)); // true
        System.out.println(s1.equals(s3)); // false
        System.out.println("‐‐‐‐‐‐‐‐‐‐‐");
        //boolean equalsIgnoreCase(String str):比较字符串的内容是否相同,忽略大小写
        System.out.println(s1.equalsIgnoreCase(s2)); // true
        System.out.println(s1.equalsIgnoreCase(s3)); // true
        System.out.println("‐‐‐‐‐‐‐‐‐‐‐");
    }
}
```

> Object 是” 对象”的意思，也是一种引用类型。作为参数类型，表示任意对象都可以传递到方法中。

**获取功能的方法**

* `public int length()` ：返回此字符串的长度。
* `public String concat(String str)` ：将指定的字符串连接到该字符串的末尾。
* `public char charAt(int index)` ：返回指定索引处的 char值。
* `public int indexOf(String str) `：返回指定子字符串第一次出现在该字符串内的索引。
* `public String substring(int beginIndex)` ：返回一个子字符串，从beginIndex开始截取字符串到字符串结尾。
* `public String substring(int beginIndex, int endIndex)` ：返回一个子字符串，从beginIndex到endIndex截取字符串。含beginIndex，不含endIndex。

方法演示，代码如下：

```java
public class String_Demo02 {
    public static void main(String[] args) {
        //创建字符串对象
        String s = "helloworld";
        // int length():获取字符串的长度，其实也就是字符个数
        System.out.println(s.length());
        System.out.println("‐‐‐‐‐‐‐‐");
        // String concat (String str):将将指定的字符串连接到该字符串的末尾.
        String s = "helloworld";
        String s2 = s.concat("**hello itheima");
        System.out.println(s2);// helloworld**hello itheima
        // char charAt(int index):获取指定索引处的字符
        System.out.println(s.charAt(0));
        System.out.println(s.charAt(1));
        System.out.println("‐‐‐‐‐‐‐‐");
        // int indexOf(String str):获取str在字符串对象中第一次出现的索引,没有返回‐1
        System.out.println(s.indexOf("l"));
        System.out.println(s.indexOf("owo"));
        System.out.println(s.indexOf("ak"));
        System.out.println("‐‐‐‐‐‐‐‐");
        // String substring(int start):从start开始截取字符串到字符串结尾
        System.out.println(s.substring(0));
        System.out.println(s.substring(5));
        System.out.println("‐‐‐‐‐‐‐‐");
        // String substring(int start,int end):从start到end截取字符串。含start，不含end。
        System.out.println(s.substring(0, s.length()));
        System.out.println(s.substring(3,8));
    }
}
```

**转换功能的方法**

* `public char[] toCharArray()` ：将此字符串转换为新的字符数组。
* `public byte[] getBytes()` ：使用平台的默认字符集将该 String编码转换为新的字节数组。
* `public String replace(CharSequence target, CharSequence replacement)` ：将与target匹配的字符串使用replacement字符串替换。

方法演示，代码如下：

```java
public class String_Demo03 {
    public static void main(String[] args) {
        //创建字符串对象
        String s = "abcde";
        // char[] toCharArray():把字符串转换为字符数组
        char[] chs = s.toCharArray();
        for(int x = 0; x < chs.length; x++) {
            System.out.println(chs[x]);
        } 
        System.out.println("‐‐‐‐‐‐‐‐‐‐‐");
        // byte[] getBytes ():把字符串转换为字节数组
        byte[] bytes = s.getBytes();
        for(int x = 0; x < bytes.length; x++) {
            System.out.println(bytes[x]);
        } 
        System.out.println("‐‐‐‐‐‐‐‐‐‐‐");
        // 替换字母it为大写IT
        String str = "itcast itheima";
        String replace = str.replace("it", "IT");
        System.out.println(replace); // ITcast ITheima
        System.out.println("‐‐‐‐‐‐‐‐‐‐‐");
    }
}
```

> CharSequence 是一个接口，也是一种引用类型。作为参数类型，可以把String对象传递到方法中。

**分割功能的方法**

* `public String[] split(String regex)` ：将此字符串按照给定的regex（规则）拆分为字符串数组。

方法演示，代码如下：

```java
public class String_Demo03 {
    public static void main(String[] args) {
        //创建字符串对象
        String s = "aa|bb|cc";
        String[] strArray = s.split("|"); // ["aa","bb","cc"]
        for(int x = 0; x < strArray.length; x++) {
            System.out.println(strArray[x]); // aa bb cc
        }
    }
}
```

#### 8.1.4 String类的练习

**拼接字符串**：定义一个方法，把数组{1,2,3}按照指定个格式拼接成一个字符串。格式参照如下：`[word1#word2#word3]`。

```java
public class StringTest1 {
    public static void main(String[] args) {
        //定义一个int类型的数组
        int[] arr = {1, 2, 3};
        //调用方法
        String s = arrayToString(arr);
        //输出结果
        System.out.println("s:" + s);
    } 
    /*
	 * 写方法实现把数组中的元素按照指定的格式拼接成一个字符串
	 * 两个明确：
	 * 返回值类型：String
	 * 参数列表：int[] arr
	 */
    public static String arrayToString(int[] arr) {
        // 创建字符串s
        String s = new String("[");
        // 遍历数组，并拼接字符串
        for (int x = 0; x < arr.length; x++) {
            if (x == arr.length ‐ 1) {
                s = s.concat(arr[x] + "]");
            } else {
                s = s.concat(arr[x] + "#");
            }
        } 
        return s;
    }
}
```

**统计字符个数：**键盘录入一个字符，统计字符串中大小写字母及数字字符个数

```java
public class StringTest2 {
    public static void main(String[] args) {
        //键盘录入一个字符串数据
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入一个字符串数据：");
        String s = sc.nextLine();
        //定义三个统计变量，初始化值都是0
        int bigCount = 0;
        int smallCount = 0;
        int numberCount = 0;
        //遍历字符串，得到每一个字符
        for(int x=0; x<s.length(); x++) {
            char ch = s.charAt(x);
            //拿字符进行判断
            if(ch>='A'&&ch<='Z') {
                bigCount++;
            }else if(ch>='a'&&ch<='z') {
                smallCount++;
            }else if(ch>='0'&&ch<='9') {
                numberCount++;
            }else {
                System.out.println("该字符"+ch+"非法");
            }
        } 
        //输出结果
        System.out.println("大写字符："+bigCount+"个");
        System.out.println("小写字符："+smallCount+"个");
        System.out.println("数字字符："+numberCount+"个");
    }
}
```

### 8.2 static关键字

#### 8.2.1 概述

关于 static 关键字的使用，它可以用来修饰的成员变量和成员方法，被修饰的成员是属于类的，而不是单单是属于某个对象的。也就是说，既然属于类，就可以不靠创建对象来调用了。

#### 8.2.2 定义和使用格式

**类变量**

当 static 修饰成员变量时，该变量称为类变量。该类的每个对象都共享同一个类变量的值。任何对象都可以更改该类变量的值，但也可以在不创建该类的对象的情况下对类变量进行操作。

**类变量**：使用 static关键字修饰的成员变量。

定义格式：`static 数据类型 变量名;`

举例：`static int numberID;`

比如说，基础班新班开班，学员报到。现在想为每一位新来报到的同学编学号（sid），从第一名同学开始，sid为1，以此类推。学号必须是唯一的，连续的，并且与班级的人数相符，这样以便知道，要分配给下一名新同学的学号是多少。这样我们就需要一个变量，与单独的每一个学生对象无关，而是与整个班级同学数量有关。

所以，我们可以这样定义一个静态变量numberOfStudent，代码如下：

```java
public class Student {
    private String name;
    private int age;
    // 学生的id
    private int sid;
    // 类变量，记录学生数量，分配学号
    public static int numberOfStudent = 0;
    public Student(String name, int age){
        this.name = name;
        this.age = age;
        // 通过 numberOfStudent 给学生分配学号
        this.sid = ++numberOfStudent;
    } 
    // 打印属性值
    public void show() {
        System.out.println("Student : name=" + name + ", age=" + age + ", sid=" + sid );
    }
}
public class StuDemo {
    public static void main(String[] args) {
        Student s1 = new Student("张三", 23);
        Student s2 = new Student("李四", 24);
        Student s3 = new Student("王五", 25);
        Student s4 = new Student("赵六", 26);
        s1.show(); // Student : name=张三, age=23, sid=1
        s2.show(); // Student : name=李四, age=24, sid=2
        s3.show(); // Student : name=王五, age=25, sid=3
        s4.show(); // Student : name=赵六, age=26, sid=4
    }
}
```

**静态方法：**当 static 修饰成员方法时，该方法称为类方法 。静态方法在声明中有 static ，建议使用类名来调用，而不需要创建类的对象。调用方式非常简单。

**类方法**：使用 static关键字修饰的成员方法，习惯称为静态方法。

定义格式：

```java
修饰符 static 返回值类型 方法名 (参数列表){
	// 执行语句
}
```

举例：在Student类中定义静态方法

```java
public static void showNum() {
	System.out.println("num:" + numberOfStudent);
}
```

**静态方法调用的注意事项：**

* 静态方法可以直接访问类变量和静态方法。
* 静态方法不能直接访问普通成员变量或成员方法。反之，成员方法可以直接访问类变量或静态方法。
* 静态方法中，不能使用this关键字。

> 小贴士：静态方法只能访问静态成员。

**调用格式**：被static修饰的成员可以并且建议通过**类名直接访问**。虽然也可以通过对象名访问静态成员，原因即多个对象均属于一个类，共享使用同一个静态成员，但是不建议，会出现警告信息。

格式：

```java
// 访问类变量
类名.类变量名；
// 调用静态方法
类名.静态方法名(参数)；
```

调用演示，代码如下：

```java
public class StuDemo2 {
    public static void main(String[] args) {
        // 访问类变量
        System.out.println(Student.numberOfStudent);
        // 调用静态方法
        Student.showNum();
    }
}
```

#### 8.2.3 静态原理图解

static 修饰的内容：

* 是随着类的加载而加载的，且只加载一次。
* 存储于一块固定的内存区域（静态区），所以，可以直接被类名调用。
* 它优先于对象存在，所以，可以被所有对象共享。

![静态原理图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251502111.PNG)

#### 8.2.4 静态代码块

**静态代码块**：定义在成员位置，使用static修饰的代码块 { }。

* 位置：类中方法外。
* 执行：随着类的加载而执行且执行一次，优先于main方法和构造方法的执行。

格式：

```java
public class ClassName{
    static {
        // 执行语句
    }
}
```

作用：给类变量进行初始化赋值。用法演示，代码如下：

```java
public class Game {
    public static int number;
    public static ArrayList<String> list;
    static {
        // 给类变量赋值
        number = 2;
        list = new ArrayList<String>();
        // 添加元素到集合中
        list.add("张三");
        list.add("李四");
    }
}
```

> 小贴士：
>
> static 关键字，可以修饰变量、方法和代码块。在使用的过程中，其主要目的还是想在不创建对象的情况下，去调用方法。下面将介绍两个工具类，来体现static 方法的便利。

### 8.3 Arrays类

#### 8.3.1 概述

`java.util.Arrays` 此类包含用来操作数组的各种方法，比如排序和搜索等。其所有方法均为静态方法，调用起来非常简单。

#### 8.3.2 操作数组的方法

`public static String toString(int[] a) `：返回指定数组内容的字符串表示形式。

```java
public static void main(String[] args) {
    // 定义int 数组
    int[] arr = {2,34,35,4,657,8,69,9};
    // 打印数组,输出地址值
    System.out.println(arr); // [I@2ac1fdc4
    // 数组内容转为字符串
    String s = Arrays.toString(arr);
    // 打印字符串,输出内容
    System.out.println(s); // [2, 34, 35, 4, 657, 8, 69, 9]
}
```

`public static void sort(int[] a) `：对指定的 int 型数组按数字升序进行排序。

```java
public static void main(String[] args) {
    // 定义int 数组
    int[] arr = {24, 7, 5, 48, 4, 46, 35, 11, 6, 2};
    System.out.println("排序前:"+ Arrays.toString(arr)); // 排序前:[24, 7, 5, 48, 4, 46, 35, 11, 6, 2]
    // 升序排序
    Arrays.sort(arr);
    System.out.println("排序后:"+ Arrays.toString(arr));// 排序后:[2, 4, 5, 6, 7, 11, 24, 35, 46, 48]
}
```

#### 8.3.3 练习

请使用 Arrays 相关的API，将一个随机字符串中的所有字符升序排列，并倒序打印。

```java
public class ArraysTest {
    public static void main(String[] args) {
        // 定义随机的字符串
        String line = "ysKUreaytWTRHsgFdSAoidq";
        // 转换为字符数组
        char[] chars = line.toCharArray();
        // 升序排序
        Arrays.sort(chars);
        // 反向遍历打印
        for (int i = chars.length‐1; i >= 0 ; i‐‐) {
            System.out.print(chars[i]+" "); // y y t s s r q o i g e d d a W U T S R K H F A
        }
    }
}
```

### 8.4 Math类

#### 8.4.1 概述

`java.lang.Math` 类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。类似这样的工具类，其所有方法均为静态方法，并且不会创建对象，调用起来非常简单。

#### 8.4.2 基本运算的方法

`public static double abs(double a)` ：返回 double 值的绝对值。

```java
double d1 = Math.abs(‐5); //d1的值为5
double d2 = Math.abs(5); //d2的值为5
```

`public static double ceil(double a)`：返回大于等于参数的最小的整数。

```java
double d1 = Math.ceil(3.3); //d1的值为 4.0
double d2 = Math.ceil(‐3.3); //d2的值为 ‐3.0
double d3 = Math.ceil(5.1); //d3的值为 6.0
```

`public static double floor(double a)` ：返回小于等于参数最大的整数。

```java
double d1 = Math.floor(3.3); //d1的值为3.0
double d2 = Math.floor(‐3.3); //d2的值为‐4.0
double d3 = Math.floor(5.1); //d3的值为 5.0
```

`public static long round(double a)` ：返回最接近参数的 long。(相当于四舍五入方法)

```java
long d1 = Math.round(5.5); //d1的值为6.0
long d2 = Math.round(5.4); //d2的值为5.0
```

#### 8.4.3 练习

请使用 Math 相关的API，计算在 -10.8 到 5.9 之间，绝对值大于 6 或者小于 2.1 的整数有多少个？

```java
public class MathTest {
    public static void main(String[] args) {
        // 定义最小值
        double min = ‐10.8;
        // 定义最大值
        double max = 5.9;
        // 定义变量计数
        int count = 0;
        // 范围内循环
        for (double i = Math.ceil(min); i <= max; i++) {
            // 获取绝对值并判断
            if (Math.abs(i) > 6 || Math.abs(i) < 2.1) {
                // 计数
                count++;
            }
        } 
        System.out.println("个数为: " + count + " 个");
    }
}
```

## 09. 继承&super&this&抽象类

### 9.1 继承

#### 9.1.1 概述

**由来**：多个类中存在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要继承那一个类即可。如图所示：

![生活中的继承](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251502033.PNG)

其中，多个类可以称为子类，单独那一个类称为父类、超类（superclass）或者基类。

继承描述的是事物之间的所属关系，这种关系是： **is-a** 的关系。例如，图中兔子属于食草动物，食草动物属于动物。可见，父类更通用，子类更具体。我们通过继承，可以使多种事物之间形成一种关系体系。

**继承定义**：就是**子类继承父类的属性和行为**，使得子类对象具有与父类相同的属性、相同的行为。子类可以直接访问父类中的**非私有**的属性和行为。

**好处**

1. 提高**代码的复用性**。
2. 类与类之间产生了关系，是**多态的前提**。

#### 9.1.2 继承的格式

通过 extends 关键字，可以声明一个子类继承另外一个父类，定义格式如下：

```java
class 父类 {
...
} 
class 子类 extends 父类 {
...
}
```

继承演示，代码如下：

```java
/**
  * 定义员工类Employee，做为父类
  */
class Employee {
    String name; // 定义name属性
    // 定义员工的工作方法
    public void work() {
        System.out.println("尽心尽力地工作");
    }
} 
/**
  * 定义讲师类Teacher 继承 员工类Employee
  */
class Teacher extends Employee {
    // 定义一个打印name的方法
    public void printName() {
        System.out.println("name=" + name);
    }
} 
/**
  * 定义测试类
  */
public class ExtendDemo01 {
    public static void main(String[] args) {
        // 创建一个讲师类对象
        Teacher t = new Teacher();
        // 为该员工类的name属性进行赋值
        t.name = "小明";
        // 调用该员工的printName()方法
        t.printName(); // name = 小明
        // 调用Teacher类继承来的work()方法
        t.work(); // 尽心尽力地工作
    }
}
```

#### 9.1.3 继承后的特点：成员变量

当类之间产生了关系后，其中各类中的成员变量，又产生了哪些影响呢？

**成员变量不重名：**如果子类父类中出现**不重名**的成员变量，这时的访问是**没有影响的**。代码如下：

```java
class Fu {
    // Fu中的成员变量。
    int num = 5;
} 
class Zi extends Fu {
    // Zi中的成员变量
    int num2 = 6;
    // Zi中的成员方法
    public void show() {
        // 访问父类中的num，
        System.out.println("Fu num="+num); // 继承而来，所以直接访问。
        // 访问子类中的num2
        System.out.println("Zi num2="+num2);
    }
} 
class ExtendDemo02 {
    public static void main(String[] args) {
        // 创建子类对象
        Zi z = new Zi();
        // 调用子类中的show方法
        z.show();
    }
} 
//演示结果：
Fu num = 5
Zi num2 = 6
```

**成员变量重名：**如果子类父类中出现**重名**的成员变量，这时的访问是**有影响的**。代码如下：

```java
class Fu {
    // Fu中的成员变量。
    int num = 5;
} 
class Zi extends Fu {
    // Zi中的成员变量
    int num = 6;
    public void show() {
        // 访问父类中的num
        System.out.println("Fu num=" + num);
        // 访问子类中的num
        System.out.println("Zi num=" + num);
    }
} 
class ExtendsDemo03 {
    public static void main(String[] args) {
        // 创建子类对象
        Zi z = new Zi();
        // 调用子类中的show方法
        z.show();
    }
} 
演示结果：
Fu num = 6
Zi num = 6
```

子父类中出现了同名的成员变量时，在子类中需要访问父类中非私有成员变量时，需要使用 super 关键字，修饰父类成员变量，类似于之前学过的 this 。

**使用格式：**

```
super.父类成员变量名
```

子类方法需要修改，代码如下：

```java
class Zi extends Fu {
    // Zi中的成员变量
    int num = 6;
    public void show() {
        //访问父类中的num
        System.out.println("Fu num=" + super.num);
        //访问子类中的num
        System.out.println("Zi num=" + this.num);
    }
} 
// 演示结果：
Fu num = 5
Zi num = 6
```

> 小贴士：Fu 类中的成员变量是非私有的，子类中可以直接访问。若Fu 类中的成员变量私有了，子类是不能直接访问的。通常编码时，我们遵循封装的原则，使用private修饰成员变量，那么如何访问父类的私有成员变量呢？对！可以在父类中提供公共的getXxx方法和setXxx方法。

#### 9.1.4 继承后的特点：成员方法

当类之间产生了关系，其中各类中的成员方法，又产生了哪些影响呢？

**成员方法不重名：**如果子类父类中出现**不重名**的成员方法，这时的调用是**没有影响的**。对象调用方法时，会先在子类中查找有没有对应的方法，若子类中存在就会执行子类中的方法，若子类中不存在就会执行父类中相应的方法。代码如下：

```java
class Fu{
    public void show(){
        System.out.println("Fu类中的show方法执行");
    }
} 
class Zi extends Fu{
    public void show2(){
        System.out.println("Zi类中的show2方法执行");
    }
} 
public class ExtendsDemo04{
    public static void main(String[] args) {
        Zi z = new Zi();
        //子类中没有show方法，但是可以找到父类方法去执行
        z.show();
        z.show2();
    }
}
```

**成员方法重名：重写(Override)：**如果子类父类中出现重名的成员方法，这时的访问是一种特殊情况，叫做**方法重写 (Override)**。

**方法重写** ：子类中出现与父类一模一样的方法时（返回值类型，方法名和参数列表都相同），会出现覆盖效果，也称为重写或者复写。声明不变，重新实现。

代码如下：

```java
class Fu {
    public void show() {
        System.out.println("Fu show");
    }
} 
class Zi extends Fu {
    //子类重写了父类的show方法
    public void show() {
        System.out.println("Zi show");
    }
} 
public class ExtendsDemo05{
    public static void main(String[] args) {
        Zi z = new Zi();
        // 子类中有show方法，只执行重写后的show方法
        z.show(); // Zi show
    }
}
```

**重写的应用**

子类可以根据需要，定义特定于自己的行为。既沿袭了父类的功能名称，又根据子类的需要重新实现父类方法，从而进行扩展增强。比如新的手机增加来电显示头像的功能，代码如下：

```java
class Phone {
    public void sendMessage(){
        System.out.println("发短信");
    } 
    public void call(){
        System.out.println("打电话");
    } 
    public void showNum(){
        System.out.println("来电显示号码");
    }
}
//智能手机类
class NewPhone extends Phone {
    //重写父类的来电显示号码功能，并增加自己的显示姓名和图片功能
    public void showNum(){
        //调用父类已经存在的功能使用super
        super.showNum();
        //增加自己特有显示姓名和图片功能
        System.out.println("显示来电姓名");
        System.out.println("显示头像");
    }
} 
public class ExtendsDemo06 {
    public static void main(String[] args) {
        // 创建子类对象
        NewPhone np = new NewPhone()；
            // 调用父类继承而来的方法
            np.call();
        // 调用子类重写的方法
        np.showNum();
    }
}
```

> 小贴士：这里重写时，用到super.父类成员方法，表示调用父类的成员方法。

**注意事项**

1. 子类方法覆盖父类方法，必须要保证权限大于等于父类权限。
2. 子类方法覆盖父类方法，返回值类型、函数名和参数列表都要一模一样。

#### 9.1.5 继承后的特点：构造方法

当类之间产生了关系，其中各类中的构造方法，又产生了哪些影响呢？

首先我们要回忆两个事情，构造方法的定义格式和作用。

1. 构造方法的名字是与类名一致的。所以子类是无法继承父类构造方法的。
2. 构造方法的作用是初始化成员变量的。所以子类的初始化过程中，必须先执行父类的初始化动作。子类的构造方法中默认有一个 `super()` ，表示调用父类的构造方法，父类成员变量初始化后，才可以给子类使用。

代码如下：

```java
class Fu {
    private int n;
    Fu(){
        System.out.println("Fu()");
    }
}
class Zi extends Fu {
    Zi(){
        // super（），调用父类构造方法
        super();
        System.out.println("Zi()");
    }
} public class ExtendsDemo07{
    public static void main (String args[]){
        Zi zi = new Zi();
    }
} 
// 输出结果：
Fu()
Zi()
```

#### 9.1.6 super和this

**父类空间优先于子类对象产生**

在每次创建子类对象时，先初始化父类空间，再创建其子类对象本身。目的在于子类对象中包含了其对应的父类空间，便可以包含其父类的成员，如果父类成员非private修饰，则子类可以随意使用父类成员。代码体现在子类的构造方法调用时，一定先调用父类的构造方法。理解图解如下：

![父类空间优先于子类对象产生](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251502242.PNG)

**super和this的含义**

* **super** ：代表父类的存储空间标识(可以理解为父亲的引用)。

* **this** ：代表当前对象的引用(谁调用就代表谁)。

**super和this的用法**

1. 访问成员

   ```java
   this.成员变量 ‐‐ 本类的
   super.成员变量 ‐‐ 父类的
   
   this.成员方法名() ‐‐ 本类的
   super.成员方法名() ‐‐ 父类的
   ```

   用法演示，代码如下：

   ```java
   class Animal {
       public void eat() {
           System.out.println("animal : eat");
       }
   } 
   class Cat extends Animal {
       public void eat() {
           System.out.println("cat : eat");
       } 
       public void eatTest() {
           this.eat(); // this 调用本类的方法
           super.eat(); // super 调用父类的方法
       }
   } 
   public class ExtendsDemo08 {
       public static void main(String[] args) {
           Animal a = new Animal();
           a.eat();
           Cat c = new Cat();
           c.eatTest();
       }
   } 
   
   // 输出结果为：
   // animal : eat
   // cat : eat
   // animal : eat
   ```

2. 访问构造方法

   ```java
   this(...) ‐‐ 本类的构造方法
   super(...) ‐‐ 父类的构造方法
   ```

  > 子类的每个构造方法中均有默认的`super()`，调用父类的空参构造。手动调用父类构造会覆盖默认的`super()`。
  >
  > `super()` 和 `this()` 都必须是在构造方法的第一行，所以不能同时出现。

#### 9.1.7 继承的特点 

1. Java支持多层继承(继承体系)。

   ```java
   //一个类只能有一个父类，不可以有多个父类。
   class C extends A{} //ok
   class C extends A，B... //error
   ```

2. Java支持多层继承(继承体系)。 

   ```java
   class A{}
   class B extends A{}
   class C extends B{}
   ```

   > 顶层父类是Object类。所有的类默认继承Object，作为父类。 

3. 子类和父类是一种相对的概念。 

### 9.2 抽象类

#### 9.2.1 概述

**由来**

父类中的方法，被它的子类们重写，子类各自的实现都不尽相同。那么父类的方法声明和方法主体，只有声明还有意义，而方法主体则没有存在的意义了。我们把没有方法主体的方法称为**抽象方法**。

Java语法规定，**包含抽象方法的类就是抽象类**。

**定义**

* **抽象方法** ： 没有方法体的方法。
* **抽象类**：包含抽象方法的类。

#### 9.2.2 abstract使用格式

**抽象方法：**使用 abstract 关键字修饰方法，该方法就成了抽象方法，抽象方法只包含一个方法名，而没有方法体。

定义格式：`修饰符 abstract 返回值类型 方法名 (参数列表);`

代码举例：`public abstract void run()；`

**抽象类：**如果一个类包含抽象方法，那么该类必须是抽象类。

定义格式：

```java
abstract class 类名字 {
    // 包含有抽象方法
}
```

代码举例：

```java
public abstract class Animal {
	public abstract void run()；
}
```

**抽象的使用**

继承抽象类的子类必须**重写父类所有的抽象方法**。否则，该子类也必须声明为抽象类。最终，必须有子类实现该父类的抽象方法，否则，从最初的父类到最终的子类都不能创建对象，失去意义。

代码举例：

```java
public class Cat extends Animal {
    public void run (){
        System.out.println("小猫在墙头走~~~")；
    }
} 
public class CatTest {
    public static void main(String[] args) {
        // 创建子类对象
        Cat c = new Cat();
        // 调用run方法
        c.run();
    }
} 
// 输出结果：
// 小猫在墙头走~~~
```

此时的方法重写，是子类对父类抽象方法的完成实现，我们将这种方法重写的操作，也叫做**实现方法**。

#### 9.2.3 注意事项

关于抽象类的使用，以下为语法上要注意的细节，虽然条目较多，但若理解了抽象的本质，无需死记硬背。

1. **抽象类不能创建对象**，如果创建，编译无法通过而报错。只能创建其非抽象子类的对象。

   > 理解：假设创建了抽象类的对象，调用抽象的方法，而抽象方法没有具体的方法体，没有意义。

2. **抽象类中，可以有构造方法**，是供子类创建对象时，初始化父类成员使用的。

   > 理解：子类的构造方法中，有默认的 `super()`，需要访问父类构造方法。

3. 抽象类中，不一定包含抽象方法，但是有抽象方法的类必定是抽象类。

   > 理解：未包含抽象方法的抽象类，目的就是不想让调用者创建该类对象，通常用于某些特殊的类结构设计。

4. 抽象类的子类，必须重写抽象父类中所有的抽象方法，否则，编译无法通过而报错。除非该子类也是抽象类。

   > 理解：假设不重写所有抽象方法，则类中可能包含抽象方法。那么创建对象后，调用抽象的方法，没有意义。

### 9.3 继承的综合案例

#### 9.3.1 综合案例：群主发普通红包

群主发普通红包。某群有多名成员，群主给成员发普通红包。普通红包的规则：

1. 群主的一笔金额，从群主余额中扣除，平均分成n等份，让成员领取。
2. 成员领取红包后，保存到成员余额中。

请根据描述，完成案例中所有类的定义以及指定类之间的继承关系，并完成发红包的操作。

#### 9.3.2 案例分析

根据描述分析，得出如下继承体系：

![群主发普通红包](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251502529.PNG)

#### 9.3.3 案例实现

定义用户类： 

```java
public class User {
    // 成员变量
    private String username; // 用户名
    private double leftMoney; // 余额
    // 构造方法
    public User() { }
    public User(String username, double leftMoney) {
        this.username = username;
        this.leftMoney = leftMoney;
    } 
    // get/set方法
    public String getUsername() {
        return username;
    } 
    public void setUsername(String username) {
        this.username = username;
    } 
    public double getLeftMoney() {
        return leftMoney;
    } 
    public void setLeftMoney(double leftMoney) {
        this.leftMoney = leftMoney;
    } 
    // 展示信息的方法
    public void show() {
        System.out.println("用户名:"+ username +" , 余额为:" + leftMoney + "元");
    }
}
```

定义群主类： 

```java
public class QunZhu extends User {
    // 添加构造方法
    public QunZhu() {
    } 
    public QunZhu(String username, double leftMoney) {
        // 通过super 调用父类构造方法
        super(username, leftMoney);
    } 
    /**
      * 群主发红包，就是把一个整数的金额，分层若干等份。
      * 1.获取群主余额,是否够发红包.
      * 	不能则返回null,并提示.
      * 	能则继续.
      * 2.修改群主余额.
      * 3.拆分红包.
      * 	3.1.如果能整除，那么就平均分。
      * 	3.2.如果不能整除，那么就把余数分给最后一份。
      */
    public ArrayList<Double> send(int money, int count) {
        // 获取群主余额
        double leftMoney = getLeftMoney();
        if(money > leftMoney) {
            return null;
        } 
        // 修改群主余额的
        setLeftMoney(leftMoney ‐ money);
        // 创建一个集合,保存等份金额
        ArrayList<Double> list = new ArrayList<>();
        // 扩大100倍,相当于折算成'分'为单位,避免小数运算损失精度的问题
        money = money * 100;
        // 每份的金额
        int m = money / count;
        // 不能整除的余数
        int l = money % count;
        // 无论是否整除,n‐1份,都是每份的等额金额
        for (int i = 0; i < count ‐ 1; i++) {
            // 缩小100倍,折算成 '元'
            list.add(m / 100.0);
        } 
        // 判断是否整除
        if (l == 0) {
            // 能整除, 最后一份金额,与之前每份金额一致
            list.add(m / 100.0);
        } else {
            // 不能整除, 最后一份的金额,是之前每份金额+余数金额
            list.add((m + l) / 100.00);
        } 
        // 返回集合
        return list;
    }
}
```

定义成员类： 

```java
public class Member extends User {
    public Member() {
    } 
    public Member(String username, double leftMoney) {
        super(username, leftMoney);
    } 
    // 打开红包,就是从集合中,随机取出一份,保存到自己的余额中
    public void openHongbao(ArrayList<Double> list) {
        // 创建Random对象
        Random r = new Random();
        // 随机生成一个角标
        int index = r.nextInt(list.size());
        // 移除一个金额
        Double money = list.remove(index);
        // 直接调用父类方法,设置到余额
        setLeftMoney( money );
    }
}
```

定义测试类： 

```java
public class Test {
    public static void main(String[] args) {
        // 创建一个群主对象
        QunZhu qz = new QunZhu("群主" , 200);
        // 创建一个键盘录入
        Scanner sc = new Scanner();
        System.out.println("请输入金额:");
        int money = sc.nextInt();
        System.out.println("请输入个数:");
        int count = sc.nextInt();
        // 发送红包
        ArrayList<Double> sendList = s.send(money,count);
        // 判断,如果余额不足
        if(sendList == null){
            System.out.println(" 余额不足...");
            return;
        } 
        // 创建三个成员
        Member m = new Member();
        Member m2 = new Member();
        Member m3 = new Member();
        // 打开红包
        m.openHongbao(sendList);
        m2.openHongbao(sendList);
        m3.openHongbao(sendList);
        // 展示信息
        qz.show();
        m.show();
        m2.show();
        m3.show();
    }
}
```

## 10. 接口&多态

### 10.1 接口

#### 10.1.1 概述

接口，是Java语言中一种引用类型，是方法的集合，如果说类的内部封装了成员变量、构造方法和成员方法，那么**接口的内部主要就是封装了方法**，包含抽象方法（JDK7及以前），默认方法和静态方法（JDK 8），私有方法（JDK9）。

接口的定义，它与定义类方式相似，但是使用 interface 关键字。它**也会被编译成.class文件**，但一定要明确它并不是类，而是另外一种引用数据类型。

> 引用数据类型：数组，类，接口。

接口的使用，它不能创建对象，但是可以被实现（implements，类似于被继承）。一个实现接口的类（可以看做是接口的子类），需要实现接口中所有的抽象方法，创建该类对象，就可以调用方法了，否则它必须是一个抽象类。

#### 10.1.2 定义格式

```java
public interface 接口名称 {
    // 抽象方法  JDK7及以前
    // 默认方法  JDK 8开始支持
    // 静态方法  JDK 8开始支持
    // 私有方法  JDK 9开始支持
}
```

* 接口中含有抽象方法

  **抽象方法**：使用 abstract 关键字修饰，可以省略，没有方法体。该方法供子类实现使用。

  代码如下：

  ```java
  public interface InterFaceName {
  	public abstract void method();
  }
  ```

* 含有默认方法和静态方法

  **默认方法**：使用 default 修饰，不可省略，供子类调用或者子类重写。

  **静态方法**：使用 static 修饰，供接口直接调用。

  代码如下：

  ```java
  public interface InterFaceName {
      public default void method() {
          // 执行语句
      } 
      public static void method2() {
          // 执行语句
      }
  }
  ```

* 含有私有方法和私有静态方法

  **私有方法**：使用 private 修饰，供接口中的默认方法或者静态方法调用。

  代码如下：

  ```java
  public interface InterFaceName {
      private void method() {
          // 执行语句
      }
  }
  ```

  > 私有方法供接口中的默认方法使用；
  >
  > 私有静态方法供接口中的静态方法使用，默认方法也可以调用

#### 10.1.3 基本实现

**实现的概述**

类与接口的关系为实现关系，即**类实现接口**，该类可以称为接口的实现类，也可以称为接口的子类。实现的动作类似继承，格式相仿，只是关键字不同，实现使用 implements 关键字。

非抽象子类实现接口：

1. 必须重写接口中所有抽象方法。
2. 继承了接口的默认方法，即可以直接调用，也可以重写。

实现格式：

```java
class 类名 implements 接口名 {
    // 重写接口中抽象方法【必须】
    // 重写接口中默认方法【可选】
}
```

**抽象方法的使用**

必须全部实现，代码如下：

定义接口：

```java
public interface LiveAble {
    // 定义抽象方法
    public abstract void eat();
    public abstract void sleep();
}
```

定义实现类：

```java
public class Animal implements LiveAble {
    @Override
    public void eat() {
        System.out.println("吃东西");
    } 
    
    @Override
    public void sleep() {
        System.out.println("晚上睡");
    }
}
```

定义测试类：

```java
public class InterfaceDemo {
    public static void main(String[] args) {
        // 创建子类对象
        Animal a = new Animal();
        // 调用实现后的方法
        a.eat();
        a.sleep();
    }
} 

// 输出结果：
// 吃东西
// 晚上睡
```

**默认方法的使用**

可以继承，可以重写，二选一，**但是只能通过实现类的对象来调用**。

1. 继承默认方法，代码如下：

   定义接口：

   ```java
   public interface LiveAble {
       public default void fly(){
           System.out.println("天上飞");
       }
   }
   ```

   定义实现类：

   ```java
   public class Animal implements LiveAble {
       // 继承，什么都不用写，直接调用
   }
   ```

   定义测试类：

   ```java
   public class InterfaceDemo {
       public static void main(String[] args) {
           // 创建子类对象
           Animal a = new Animal();
           // 调用默认方法
           a.fly();
       }
   } 
   
   // 输出结果：
   // 天上飞
   ```

2. 重写默认方法，代码如下：

   定义接口：

   ```java
   public interface LiveAble {
       public default void fly(){
           System.out.println("天上飞");
       }
   }
   ```

   定义实现类：

   ```java
   public class Animal implements LiveAble {
       @Override
       public void fly() {
           System.out.println("自由自在的飞");
       }
   }
   ```

   定义测试类：

   ```java
   public class InterfaceDemo {
       public static void main(String[] args) {
           // 创建子类对象
           Animal a = new Animal();
           // 调用重写方法
           a.fly();
       }
   } 
   
   // 输出结果：
   // 自由自在的飞
   ```

**静态方法的使用**

静态与.class 文件相关，**只能使用接口名调用**，不可以通过实现类的类名或者实现类的对象调用，代码如下：

定义接口：

```java
public interface LiveAble {
    public static void run(){
        System.out.println("跑起来~~~");
    }
}
```

定义实现类：

```java
public class Animal implements LiveAble {
    // 无法重写静态方法
}
```

定义测试类：

```java
public class InterfaceDemo {
    public static void main(String[] args) {
        // Animal.run(); // 【错误】无法继承方法,也无法调用
        LiveAble.run(); //
    }
} 

// 输出结果：
// 跑起来~~~
```

**私有方法的使用**

**私有方法**：只有默认方法可以调用。

**私有静态方法**：默认方法和静态方法可以调用。

如果一个接口中有多个默认方法，并且方法中有重复的内容，那么可以抽取出来，封装到私有方法中，供默认方法去调用。从设计的角度讲，私有的方法是对默认方法和静态方法的辅助。

定义接口：

```java
public interface LiveAble {
    default void func(){
        func1();
        func2();
    } 
    private void func1(){
        System.out.println("跑起来~~~");
    } 
    private void func2(){
        System.out.println("跑起来~~~");
    }
}
```

> JKD8是不行的，JDK9是可以的

#### 10.1.4 接口的多实现

之前学过，在继承体系中，一个类只能继承一个父类。而对于接口而言，一个类是可以实现多个接口的，这叫做**接口的多实现**。并且，一个类能继承一个父类，同时实现多个接口。

实现格式：

```java
class 类名 [extends 父类名] implements 接口名1,接口名2,接口名3... {
    // 重写接口中抽象方法【必须】
    // 重写接口中默认方法【不重名时可选】
}
```

> [ ]： 表示可选操作。

**抽象方法**

接口中，有多个抽象方法时，实现类必须重写所有抽象方法。如果抽象方法有重名的，只需要重写一次.代码如下：

定义多个接口：

```java
interface A {
    public abstract void showA();
    public abstract void show();
} 
interface B {
    public abstract void showB();
    public abstract void show();
}
```

定义实现类：

```java
public class C implements A,B{
    @Override
    public void showA() {
        System.out.println("showA");
    } 
    @Override
    public void showB() {
        System.out.println("showB");
    } 
    @Override
    public void show() {
        System.out.println("show");
    }
}
```

**默认方法**

接口中，有多个默认方法时，实现类都可继承使用。**如果默认方法有重名的，必须重写一次**。代码如下：

定义多个接口：

```java
interface A {
    public default void methodA(){}
    public default void method(){}
} 
interface B {
    public default void methodB(){}
    public default void method(){}
}
```

定义实现类：

```java
public class C implements A,B{
    @Override
    public void method() {
        System.out.println("method");
    }
}
```

**静态方法**

接口中，存在同名的静态方法并不会冲突，原因是**只能通过各自接口名访问静态方法**。

**优先级的问题**

当一个类，既继承一个父类，又实现若干个接口时，父类中的成员方法与接口中的默认方法重名，**子类就近选择执行父类的成员方法**。代码如下：

定义接口：

```java
interface A {
    public default void methodA(){
        System.out.println("AAAAAAAAAAAA");
    }
}
```

定义父类：

```java
class D {
    public void methodA(){
        System.out.println("DDDDDDDDDDDD");
    }
}
```

定义子类：

```java
class C extends D implements A {
	// 未重写methodA方法
}
```

定义测试类：

```java
public class Test {
    public static void main(String[] args) {
        C c = new C();
        c.methodA();  // 就近调用父类的方法
    }
} 
// 输出结果:
// DDDDDDDDDDDD
```

#### 10.1.5 接口的多继承

一个接口能继承另一个或者多个接口，这和类之间的继承比较相似。接口的继承使用 extends 关键字，子接口继承父接口的方法。**如果父接口中的默认方法有重名的，那么子接口需要重写一次**。代码如下：

定义父接口：

```java
interface A {
    public default void method(){
        System.out.println("AAAAAAAAAAAAAAAAAAA");
    }
} 
interface B {
    public default void method(){
        System.out.println("BBBBBBBBBBBBBBBBBBB");
    }
}
```

定义子接口：

```java
interface D extends A,B{
    @Override
    public default void method() {
        System.out.println("DDDDDDDDDDDDDD");
    }
}
```

> 小贴士：
>
> 子接口重写默认方法时，default关键字可以保留。
>
> 子类重写默认方法时，default关键字不可以保留。

#### 10.1.6 其他成员特点

接口中，无法定义成员变量，但是可以定义常量，其值不可以改变，**默认使用public static final修饰**。

接口中，没有构造方法，不能创建对象。

接口中，没有静态代码块。

### 10.2 多态

#### 10.2.1 概述

**引入**

多态是继封装、继承之后，面向对象的第三大特性。

生活中，比如跑的动作，小猫、小狗和大象，跑起来是不一样的。再比如飞的动作，昆虫、鸟类和飞机，飞起来也是不一样的。可见，**同一行为，通过不同的事物，可以体现出来的不同的形态**。多态，描述的就是这样的状态。

**多态定义： 是指同一行为，具有多个不同表现形式。**

**前提**


1. 继承或者实现【二选一】
2. 方法的重写【意义体现：不重写，无意义】
3. 父类引用指向子类对象【格式体现】

#### 10.2.2 多态的体现

多态体现的格式：

```java
父类类型 变量名 = new 子类对象；
变量名.方法名();
```

> 父类类型：指子类对象继承的父类类型，或者实现的父接口类型。

代码如下：

```java
Fu f = new Zi();
f.method();
```

**当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，执行的是子类重写后方法。**

代码如下：

定义父类：

```java
public abstract class Animal {
    public abstract void eat();
}
```

定义子类：

```java
class Cat extends Animal {
    public void eat() {
        System.out.println("吃鱼");
    }
} 
class Dog extends Animal {
    public void eat() {
        System.out.println("吃骨头");
    }
}
```

定义测试类：

```java
public class Test {
    public static void main(String[] args) {
        // 多态形式，创建对象
        Animal a1 = new Cat();
        // 调用的是 Cat 的 eat
        a1.eat();
        // 多态形式，创建对象
        Animal a2 = new Dog();
        // 调用的是 Dog 的 eat
        a2.eat();
    }
}
```

#### 10.2.3 多态的好处

实际开发的过程中，父类类型作为方法形式参数，传递子类对象给方法，进行方法的调用，更能体现出多态的扩展性与便利。代码如下：

定义父类：

```java
public abstract class Animal {
	public abstract void eat();
}
```

定义子类：

```java
class Cat extends Animal {
    public void eat() {
        System.out.println("吃鱼");
    }
} 
class Dog extends Animal {
    public void eat() {
        System.out.println("吃骨头");
    }
}
```

定义测试类：

```java
public class Test {
    public static void main(String[] args) {
        // 多态形式，创建对象
        Cat c = new Cat();
        Dog d = new Dog();
        // 调用showCatEat
        showCatEat(c);
        // 调用showDogEat
        showDogEat(d);
        /** 
		  *以上两个方法, 均可以被showAnimalEat(Animal a)方法所替代
		  *而执行效果一致
		  */
        showAnimalEat(c);
        showAnimalEat(d);
    } 
    public static void showCatEat (Cat c){
        c.eat();
    } 
    public static void showDogEat (Dog d){
        d.eat();
    } 
    public static void showAnimalEat (Animal a){
        a.eat();
    }
}
```

由于多态特性的支持，showAnimalEat方法的Animal类型，是Cat和Dog的父类类型，父类类型接收子类对象，当然可以把Cat对象和Dog对象，传递给方法。

当eat方法执行时，多态规定，执行的是子类重写的方法，那么效果自然与showCatEat、showDogEat方法一致，所以showAnimalEat完全可以替代以上两方法。

不仅仅是替代，在扩展性方面，无论之后再多的子类出现，我们都不需要编写showXxxEat方法了，直接使用showAnimalEat都可以完成。

所以，多态的好处，体现在，可以使程序编写的更简单，并有良好的扩展。

#### 10.2.4 引用类型转换

多态的转型分为向上转型与向下转型两种：

**向上转型**

向上转型：**多态本身是子类类型向父类类型向上转换的过程，这个过程是默认的**。

当父类引用指向一个子类对象时，便是向上转型。

使用格式：

```java
父类类型 变量名 = new 子类类型();
如：Animal a = new Cat();
```

**向下转型**

向下转型：**父类类型向子类类型向下转换的过程，这个过程是强制的**。

一个已经向上转型的子类对象，将父类引用转为子类引用，可以使用强制类型转换的格式，便是向下转型。

使用格式：

```java
子类类型 变量名 = (子类类型) 父类变量名;
如:Cat c =(Cat) a;
```

**为什么要转型**

当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误。也就是说，**不能调用子类拥有，而父类没有的方法**。编译都错误，更别说运行了。这也是多态给我们带来的一点"小麻烦"。所以，想要调用子类特有的方法，必须做向下转型。

转型演示，代码如下：

定义类：

```java
abstract class Animal {
    abstract void eat();
} 
class Cat extends Animal {
    public void eat() {
        System.out.println("吃鱼");
    } 
    public void catchMouse() {
        System.out.println("抓老鼠");
    }
} 
class Dog extends Animal {
    public void eat() {
        System.out.println("吃骨头");
    } 
    public void watchHouse() {
        System.out.println("看家");
    }
}
```

定义测试类：

```java
public class Test {
    public static void main(String[] args) {
        // 向上转型
        Animal a = new Cat();
        a.eat(); // 调用的是 Cat 的 eat
        // 向下转型
        Cat c = (Cat)a;
        c.catchMouse(); // 调用的是 Cat 的 catchMouse
    }
}
```

**转型的异常**

转型的过程中，一不小心就会遇到这样的问题，请看如下代码：

```java
public class Test {
    public static void main(String[] args) {
        // 向上转型
        Animal a = new Cat();
        a.eat(); // 调用的是 Cat 的 eat
        // 向下转型
        Dog d = (Dog)a;
        d.watchHouse(); // 调用的是 Dog 的 watchHouse 【运行报错】
    }
}
```

这段代码可以通过编译，但是运行时，却报出了 ClassCastException ，类型转换异常！这是因为，明明创建了Cat类型对象，运行时，当然不能转换成Dog对象的。这两个类型并没有任何继承关系，不符合类型转换的定义。

为了避免ClassCastException的发生，Java提供了 instanceof 关键字，给引用变量做类型的校验，格式如下：

```java
变量名 instanceof 数据类型
如果变量属于该数据类型，返回true。
如果变量不属于该数据类型，返回false。
```

所以，转换前，我们最好先做一个判断，代码如下：

```java
public class Test {
    public static void main(String[] args) {
        // 向上转型
        Animal a = new Cat();
        a.eat(); // 调用的是 Cat 的 eat
        // 向下转型
        if (a instanceof Cat){
            Cat c = (Cat)a;
            c.catchMouse(); // 调用的是 Cat 的 catchMouse
        } else if (a instanceof Dog){
            Dog d = (Dog)a;
            d.watchHouse(); // 调用的是 Dog 的 watchHouse
        }
    }
}
```

### 10.3 接口多态的综合案例

#### 10.3.1 笔记本电脑

笔记本电脑（laptop）通常具备使用USB设备的功能。在生产时，笔记本都预留了可以插入USB设备的USB接口，但具体是什么USB设备，笔记本厂商并不关心，只要符合USB规格的设备都可以。

定义USB接口，具备最基本的开启功能和关闭功能。鼠标和键盘要想能在电脑上使用，那么鼠标和键盘也必须遵守USB规范，实现USB接口，否则鼠标和键盘的生产出来也无法使用。

#### 10.3.2 案例分析

进行描述笔记本类，实现笔记本使用USB鼠标、USB键盘

* USB接口，包含开启功能、关闭功能
* 笔记本类，包含运行功能、关机功能、使用USB设备功能
* 鼠标类，要实现USB接口，并具备点击的方法
* 键盘类，要实现USB接口，具备敲击的方法

#### 10.3.3 案例实现

定义USB接口：

```java
interface USB {
    void open();// 开启功能
    void close();// 关闭功能
}
```

定义鼠标类：

```java
class Mouse implements USB {
    public void open() {
        System.out.println("鼠标开启，红灯闪一闪");
    } 
    public void close() {
        System.out.println("鼠标关闭，红灯熄灭");
    } 
    public void click(){
        System.out.println("鼠标单击");
    }
}
```

定义键盘类:

```java
class KeyBoard implements USB {
    public void open() {
        System.out.println("键盘开启，绿灯闪一闪");
    } 
    public void close() {
        System.out.println("键盘关闭，绿灯熄灭");
    } 
    public void type(){
        System.out.println("键盘打字");
    }
}
```

定义笔记本类：

```java
class Laptop {
    // 笔记本开启运行功能
    public void run() {
        System.out.println("笔记本运行");
    } 
    // 笔记本使用usb设备，这时当笔记本对象调用这个功能时，必须给其传递一个符合USB规则的USB设备
    public void useUSB(USB usb) {
        // 判断是否有USB设备
        if (usb != null) {
            usb.open();
            // 类型转换,调用特有方法
            if(usb instanceof Mouse){
                Mouse m = （Mouse）usb；
                    m.click();
            }else if (usb instanceof KeyBoard){
                KeyBoard kb = (KeyBoard)usb;
                kb.type();
            } 
            usb.close();
        }
    } 
    public void shutDown() {
        System.out.println("笔记本关闭");
    }
}
```


测试类，代码如下：

```java
public class Test {
    public static void main(String[] args) {
        // 创建笔记本实体对象
        Laptop lt = new Laptop();
        // 笔记本开启
        lt.run();
        // 创建鼠标实体对象
        Usb u = new Mouse();
        // 笔记本使用鼠标
        lt.useUSB(u);
        // 创建键盘实体对象
        KeyBoard kb = new KeyBoard();
        // 笔记本使用键盘
        lt.useUSB(kb);
        // 笔记本关闭
        lt.shutDown();
    }
}
```

## 11. final&权限&内部类&引用类型

### 11.1 final关键字

#### 11.1.1 概述

学习了继承后，我们知道，子类可以在父类的基础上改写父类内容，比如，方法重写。那么我们能不能随意的继承API中提供的类，改写其内容呢？显然这是不合适的。为了避免这种随意改写的情况，Java提供了 final 关键字，用于修饰**不可改变**内容。

**final**： 不可改变。可以用于修饰类、方法和变量。

* 类：被修饰的类，不能被继承。
* 方法：被修饰的方法，不能被重写。
* 变量：被修饰的变量，不能被重新赋值。

#### 11.1.2 使用方式

**修饰类**

格式如下：

```java
final class 类名 {
}
```

查询API发现像 `public final class String` 、 `public final class Math` 、 `public final class Scanner`等，很多我们学习过的类，都是被final修饰的，目的就是供我们使用，而不让我们所以改变其内容。

**修饰方法**

格式如下：

```java
修饰符 final 返回值类型 方法名(参数列表){
	//方法体
}
```

重写被 final 修饰的方法，编译时就会报错。

**修饰变量**

1. **局部变量：基本类型**

  基本类型的局部变量，被final修饰后，只能赋值一次，不能再更改。代码如下：

  ```java
public class FinalDemo1 {
    public static void main(String[] args) {
        // 声明变量，使用final修饰
        final int a;
        // 第一次赋值
        a = 10;
        // 第二次赋值
        a = 20; // 报错,不可重新赋值
        // 声明变量，直接赋值，使用final修饰
        final int b = 10;
        // 第二次赋值
        b = 20; // 报错,不可重新赋值
    }
}
  ```

思考，如下两种写法，哪种可以通过编译？

写法1：

  ```java
final int c = 0;
for (int i = 0; i < 10; i++) {
    c = i;
    System.out.println(c);
}
  ```

写法2：

  ```java
for (int i = 0; i < 10; i++) {
    final int c = i;
    System.out.println(c);
}
  ```

根据 final 的定义，写法1报错！写法2，为什么通过编译呢？因为每次循环，都是一次新的变量c。这也是大家需要注意的地方。

2. **局部变量：引用类型**

引用类型的局部变量，被final修饰后，只能指向一个对象，地址不能再更改。但是不影响对象内部的成员变量值的修改，代码如下：

  ```java
public class FinalDemo2 {
    public static void main(String[] args) {
        // 创建 User 对象
        final User u = new User();
        // 创建 另一个 User对象
        u = new User(); // 报错，指向了新的对象，地址值改变。
        // 调用setName方法
        u.setName("张三"); // 可以修改
    }
}
  ```

3. **成员变量**

成员变量涉及到初始化的问题，初始化方式有两种，只能二选一：

  * 显示初始化；

  ```java
public class User {
    final String USERNAME = "张三";
    private int age;
}
  ```

  * 构造方法初始化。

  ```java
public class User {
    final String USERNAME ;
    private int age;
    public User(String username, int age) {
        this.USERNAME = username;
        this.age = age;
    }
}
  ```

  > 被final修饰的常量名称，一般都有书写规范，所有字母都大写。

### 11.2 权限修饰符

#### 11.2.1 概述

在Java中提供了四种访问权限，使用不同的访问权限修饰符修饰时，被修饰的内容会有不同的访问权限，

* public：公共的
* protected：受保护的
* default：默认的
* private：私有的

#### 11.2.2 不同权限的访问能力

|                        | public | protected | default（空的） | private |
| ---------------------- | ------ | --------- | --------------- | ------- |
| 同一类中               | √      | √         | √               | √       |
| 同一包中(子类与无关类) | √      | √         | √               |         |
| 不同包的子类           | √      | √         |                 |         |
| 不同包中的无关类       | √      |           |                 |         |

可见，public具有最大权限。private则是最小权限。

编写代码时，如果没有特殊的考虑，建议这样使用权限：

* 成员变量使用 private ，隐藏细节。
* 构造方法使用 public ，方便创建对象。
* 成员方法使用 public ，方便调用方法。

> 小贴士：不加权限修饰符，其访问能力与default修饰符相同

### 11.3 内部类

#### 11.3.1 概述

**内部类：**将一个类A定义在另一个类B里面，里面的那个类A就称为**内部类**，B则称为**外部类**。

**成员内部类：**定义在类中，方法外的类。

定义格式：

```java
class 外部类 {
    class 内部类{
    }
}
```

在描述事物时，若一个事物内部还包含其他事物，就可以使用内部类这种结构。比如，汽车类 Car 中包含发动机类 Engine ，这时， Engine 就可以使用内部类来描述，定义在成员位置。

代码举例：

```java
class Car { //外部类
    class Engine { //内部类
    }
}
```

**访问特点**

* 内部类可以直接访问外部类的成员，包括私有成员。
* 外部类要访问内部类的成员，必须要建立内部类的对象。

创建内部类对象格式：`外部类名.内部类名 对象名 = new 外部类型().new 内部类型();`

访问演示，代码如下：

定义类：

```java
public class Person {
    private boolean live = true;
    class Heart {
        public void jump() {
            // 直接访问外部类成员
            if (live) {
                System.out.println("心脏在跳动");
            } else {
                System.out.println("心脏不跳了");
            }
        }
    } 
    public boolean isLive() {
        return live;
    } 
    public void setLive(boolean live) {
        this.live = live;
    }
}
```

定义测试类：

```java
public class InnerDemo {
    public static void main(String[] args) {
        // 创建外部类对象
        Person p = new Person();
        // 创建内部类对象
        Heart heart = p.new Heart();
        // 调用内部类方法
        heart.jump();
        // 调用外部类方法
        p.setLive(false);
        // 调用内部类方法
        heart.jump();
    }
} 
// 输出结果:
// 心脏在跳动
// 心脏不跳了
```

内部类仍然是一个独立的类，在编译之后会内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号，比如，`Person$Heart.class`

#### 11.3.2 匿名内部类

匿名内部类 ：是内部类的简化写法。它的本质是一个**带具体实现的父类或者父接口的匿名的子类对象**。

开发中，最常用到的内部类就是匿名内部类了。以接口举例，当你使用一个接口时，似乎得做如下几步操作：

1. 定义子类
2. 重写接口中的方法
3. 创建子类对象
4. 调用重写后的方法

我们的目的，最终只是为了调用方法，那么能不能简化一下，把以上四步合成一步呢？匿名内部类就是做这样的快捷方式。

**前提：**匿名内部类必须继承一个父类或者实现一个父接口。

**格式**

```java
new 父类名或者接口名(){
    // 方法重写
    @Override
    public void method() {
        // 执行语句
    }
};
```

**使用方式**

以接口为例，匿名内部类的使用，代码如下：

定义接口：

```java
public abstract class FlyAble{
    public abstract void fly();
}
```

创建匿名内部类，并调用：

```java
public class InnerDemo {
    public static void main(String[] args) {
        /*
		 1.等号右边:是匿名内部类，定义并创建该接口的子类对象
		 2.等号左边:是多态赋值,接口类型引用指向子类对象
		 */
        FlyAble f = new FlyAble(){
            public void fly() {
                System.out.println("我飞了~~~");
            }
        };
        //调用 fly方法,执行重写后的方法
        f.fly();
    }
}
```

通常在方法的形式参数是接口或者抽象类时，也可以将匿名内部类作为参数传递。代码如下：

```java
public class InnerDemo2 {
    public static void main(String[] args) {
        /*
		 1.等号右边:定义并创建该接口的子类对象
		 2.等号左边:是多态,接口类型引用指向子类对象
		 */
        FlyAble f = new FlyAble(){
            public void fly() {
                System.out.println("我飞了~~~");
            }
        };
        // 将f传递给showFly方法中
        showFly(f);
    } 
    public static void showFly(FlyAble f) {
        f.fly();
    }
}
```

以上两步，也可以简化为一步，代码如下：

```java
public class InnerDemo3 {
    public static void main(String[] args) {
        /*
		 创建匿名内部类,直接传递给showFly(FlyAble f)
		 */
        showFly( new FlyAble(){
            public void fly() {
                System.out.println("我飞了~~~");
            }
        });
    } 
    public static void showFly(FlyAble f) {
        f.fly();
    }
}
```

### 11.4 引用类型用法总结

实际的开发中，引用类型的使用非常重要，也是非常普遍的。我们可以在理解基本类型的使用方式基础上，进一步去掌握引用类型的使用方式。基本类型可以作为成员变量、作为方法的参数、作为方法的返回值，那么当然引用类型也是可以的。

#### 11.4.1 class作为成员变量

在定义一个类Role（游戏角色）时，代码如下：

```java
class Role {
    int id; // 角色id
    int blood; // 生命值
    String name; // 角色名称
}
```

使用 int 类型表示 角色id和生命值，使用 String 类型表示姓名。此时， String 本身就是引用类型，由于使用的方式类似常量，所以往往忽略了它是引用类型的存在。如果我们继续丰富这个类的定义，给 Role 增加武器，穿戴装备等属性，我们将如何编写呢？

定义武器类，将增加攻击能力：

```java
class Weapon {
    String name; // 武器名称
    int hurt;    // 伤害值
}
```

定义穿戴盔甲类，将增加防御能力，也就是提升生命值：

```java
class Armour {
    String name;// 装备名称
    int protect;// 防御值
}
```

定义角色类：

```java
class Role {
    int id;
    int blood;
    String name;
    // 添加武器属性
    Weapon wp;
    // 添加盔甲属性
    Armour ar;
    // 提供get/set方法
    public Weapon getWp() {
        return wp;
    } 
    public void setWeapon(Weapon wp) {
        this.wp = wp;
    } 
    public Armour getArmour() {
        return ar;
    } 
    public void setArmour(Armour ar) {
        this.ar = ar;
    } 
    // 攻击方法
    public void attack(){
        System.out.println("使用"+ wp.getName() +", 造成"+wp.getHurt()+"点伤害");
    } 
    // 穿戴盔甲
    public void wear(){
        // 增加防御,就是增加blood值
        this.blood += ar.getProtect();
        System.out.println("穿上"+ar.getName()+", 生命值增加"+ar.getProtect());
    }
}
```

测试类：

```java
public class Test {
    public static void main(String[] args) {
        // 创建Weapon 对象
        Weapon wp = new Weapon("屠龙刀" , 999999);
        // 创建Armour 对象
        Armour ar = new Armour("麒麟甲",10000);
        // 创建Role 对象
        Role r = new Role();
        // 设置武器属性
        r.setWeapon(wp);
        // 设置盔甲属性
        r.setArmour(ar);
        // 攻击
        r.attack();
        // 穿戴盔甲
        r.wear();
    }
} 

// 输出结果:
// 使用屠龙刀,造成999999点伤害
// 穿上麒麟甲 ,生命值增加10000
```

> 类作为成员变量时，对它进行赋值的操作，实际上，是赋给它该类的一个对象。

#### 11.4.2 interface作为成员变量

接口是对方法的封装，对应游戏当中，可以看作是扩展游戏角色的技能。所以，如果想扩展更强大技能，我们在Role中，可以增加接口作为成员变量，来设置不同的技能。

定义接口：

```java
// 法术攻击
public interface FaShuSkill {
	public abstract void faShuAttack();
}
```

定义角色类：

```java
public class Role {
    FaShuSkill fs;
    public void setFaShuSkill(FaShuSkill fs) {
        this.fs = fs;
    } 
    // 法术攻击
    public void faShuSkillAttack(){
        System.out.print("发动法术攻击:");
        fs.faShuAttack();
        System.out.println("攻击完毕");
    }
}
```

定义测试类：

```java
public class Test {
    public static void main(String[] args) {
        // 创建游戏角色
        Role role = new Role();
        // 设置角色法术技能
        role.setFaShuSkill(new FaShuSkill() {
            @Override
            public void faShuAttack() {
                System.out.println("纵横天下");
            }
        });
        // 发动法术攻击
        role.faShuSkillAttack();
        // 更换技能
        role.setFaShuSkill(new FaShuSkill() {
            @Override
            public void faShuAttack() {
                System.out.println("逆转乾坤");
            }
        });
        // 发动法术攻击
        role.faShuSkillAttack();
    }
} 

// 输出结果:
// 发动法术攻击:纵横天下
// 攻击完毕
// 发动法术攻击:逆转乾坤
// 攻击完毕
```

> 我们使用一个接口，作为成员变量，以便随时更换技能，这样的设计更为灵活，增强了程序的扩展性。
>
> 接口作为成员变量时，对它进行赋值的操作，实际上，是赋给它该接口的一个子类对象。

#### 11.4.3 interface作为方法参数和返回值类型

当接口作为方法的参数时，需要传递什么呢？当接口作为方法的返回值类型时，需要返回什么呢？

对，其实都是它的子类对象。 ArrayList 类我们并不陌生，查看API我们发现，实际上，它是 java.util.List 接口的实现类。所以，当我们看见 List 接口作为参数或者返回值类型时，当然可以将 ArrayList 的对象进行传递或返回。

请观察如下方法：获取某集合中所有的偶数。

定义方法：

```java
public static List<Integer> getEvenNum(List<Integer> list) {
    // 创建保存偶数的集合
    ArrayList<Integer> evenList = new ArrayList<>();
    // 遍历集合list,判断元素为偶数,就添加到evenList中
    for (int i = 0; i < list.size(); i++) {
        Integer integer = list.get(i);
        if (integer % 2 == 0) {
            evenList.add(integer);
        }
    } 
    /* 返回偶数集合
     因为getEvenNum方法的返回值类型是List,而ArrayList是List的子类,
     所以evenList可以返回
     */
    return evenList;
}
```

调用方法：

```java
public class Test {
    public static void main(String[] args) {
        // 创建ArrayList集合,并添加数字
        ArrayList<Integer> srcList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            srcList.add(i);
        } 
        /* 
          获取偶数集合
		  因为getEvenNum方法的参数是List,而ArrayList是List的子类,
		  所以srcList可以传递
		 */
        List list = getEvenNum(srcList);
        System.out.println(list);
    }
}
```

> 接口作为参数时，传递它的子类对象。
>
> 接口作为返回值类型时，返回它的子类对象。
