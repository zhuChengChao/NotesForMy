# JVM：类加载与字节码技术-2

> 在学习了 bilibili 上 **[黑马程序员]()** 的课程 [JVM完整教程](https://www.bilibili.com/video/BV1yE411Z7AP) 后做的学习笔记，总共分为5篇笔记，本文为 4/5 篇。

### 内容

> 这部分内容在上一篇笔记中：
>
> 1. 类文件结构
> 2. 字节码指令

3. 编译期处理
4. 类加载阶段
5. 类加载器
6. 运行期优化 

![类加载与字节码技术](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624806.PNG)

## 3. 编译期处理 

所谓的 **语法糖** ，其实就是指 java 编译器把 `*.java` 源码编译为 `*.class` 字节码的过程中，自动生成和转换的一些代码，主要是为了减轻程序员的负担，算是 java 编译器给我们的一个额外福利（给糖吃嘛）

> 注意，以下代码的分析，借助了 javap 工具，idea 的反编译功能，idea 插件 jclasslib 等工具。另外，编译器转换的结果直接就是 class 字节码，只是为了便于阅读，给出了几乎等价的 java 源码方式，并不是编译器还会转换出中间的 java 源码，切记。 

### 3.1 默认构造器 

```java
public class Candy1{
    
}
```

编译成class后的代码： 

```java
public class Candy1 {
	// 这个无参构造是编译器帮助我们加上的
    public Candy1() {
        // 即调用父类 Object 的无参构造方法，即调用 java/lang/Object." <init>":()V
        super(); 
    }
}
```

### 3.2 自动拆装箱 

这个特性是 JDK 5 开始加入的，代码片段1： 

```java
public class Candy2{
    public static void main(String[] args){
        Integer x = 1;
        int y = x;
    }
}
```

这段代码在 JDK 5 之前是无法编译通过的，必须改写为代码片段2 : 

```java
public class Candy2{
    public static void main(String[] args){
        Integer x = Integer.valueOf(1);
        int y = x.intValue();
    }
}
```

显然之前版本的代码太麻烦了，需要在基本类型和包装类型之间来回切换（尤其是集合类中操作的都是包装类型），因此这些转换的事情在JDK5以后都在编译阶段完成。即代码片段1都会在编译阶段被转换为代码片段2。

### 3.3 泛型集合取值 

泛型也是在 JDK 5 开始加入的特性，但 java 在编译泛型代码后会执行 **泛型擦除** 的动作，即泛型信息在编译为字节码之后就丢失了，实际的类型都当做了 Object 类型来处理： 

```java
public class Candy3 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(10); // 实际调用的是 List.add(Object e)
        Integer x = list.get(0); // 实际调用的是 Object obj = List.get(int index);
    }
}
```

所以在取值时，编译器真正生成的字节码中，还要额外做一个类型转换的操作：

```java
// 需要将 Object 转为 Integer
Integer x = (Integer)list.get(0);
```

如果前面的 x 变量类型修改为 int 基本类型那么最终生成的字节码是： 

```java
// 需要将 Object 转为 Integer, 并执行拆箱操作
int x = ((Integer)list.get(0)).intValue();
```

还好这些麻烦事都不用自己做。 

擦除的是字节码上的泛型信息，可以看到 LocalVariableTypeTable 仍然保留了方法参数泛型的信息 

```java
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #2  // class java/util/ArrayList
         3: dup
         4: invokespecial #3  // Method java/util/ArrayList."<init>":()V
         7: astore_1
         8: aload_1
         9: bipush        10
        11: invokestatic  #4  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        14: invokeinterface   // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z--->接口方法调用，可以看到传入参数类型为Object类型
        19: pop
        20: aload_1
        21: iconst_0          // 设置get需要的下标
        22: invokeinterface   // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;--->接口方法调用，返回的类型为Object类型
        27: checkcast     #7  // class java/lang/Integer--->进行类型转换
        30: astore_2
        31: return
      LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 20
        line 11: 31
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      32     0  args   [Ljava/lang/String;
            8      24     1  list   Ljava/util/List;
           31       1     2     x   Ljava/lang/Integer;
      LocalVariableTypeTable:  // 这里保留了方法参数泛型的信息
        Start  Length  Slot  Name   Signature // 如下方slot1中保存了Integer泛型信息
            8      24     1  list   Ljava/util/List<Ljava/lang/Integer;>;
```

使用反射，仍然能够获得这些信息： 

```java
public Set<Integer> test(List<String> list, Map<Integer, Object> map){}
```

```java
Method test = Candy3.class.getMethod("test", List.class, Map.class);  // 不需要带泛型
Type[] types = test.getGenericParameterTypes();
for (Type type : types) {
    if (type instanceof ParameterizedType) {  // 判断type是不是泛型类型
        ParameterizedType parameterizedType = (ParameterizedType) type;
        System.out.println("原始类型 - " + parameterizedType.getRawType());
        Type[] arguments = parameterizedType.getActualTypeArguments();
        for (int i = 0; i < arguments.length; i++) {
            System.out.printf("泛型参数[%d] - %s\n", i, arguments[i]);
        }
    }
}
```

输出：

```java
原始类型 - interface java.util.List
泛型参数[0] - class java.lang.String
原始类型 - interface java.util.Map
泛型参数[0] - class java.lang.Integer
泛型参数[1] - class java.lang.Object
```

### 3.4 可变参数 

可变参数也是 JDK 5 开始加入的新特性： 例如： 

```java
public class Candy4{
    public static void foo(String... args){
        String[] array = args;  // 直接赋值
        System.out.println(array);
    }
    public static void main(String[] args){
        foo("hello", "world");
    }
}
```

可变参数 `String... args` 其实是一个 `String[] args` ，从代码中的赋值语句中就可以看出来。 同样 java 编译器会在编译期间将上述代码变换为： 

```java
public class Candy4 {
    public static void foo(String[] args) {
        String[] array = args; // 直接赋值
        System.out.println(array);
    } 
    public static void main(String[] args) {
    	foo(new String[]{"hello", "world"});
    }
}
```

> **注意**：如果调用了 `foo()` 则等价代码为 `foo(new String[]{})` ，创建了一个空的数组，而不会
> 传递 null 进去 

### 3.5 foreach 循环 

仍是 JDK 5 开始引入的语法糖，数组的循环： 

```java
public class Candy5_1{
    public static void main(String[] args){
        int[] array = {1, 2, 3, 4, 5}; // 数组赋初值的简化写法也是语法糖哦
        for(int e: array){
            System.out.println(e);
        }
    }
}
```

会被编译器转换为：

```java
public class Candy5_1 {
    public Candy5_1() {
    } 
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 3, 4, 5};
        for(int i = 0; i < array.length; ++i) {
            int e = array[i];
            System.out.println(e);
        }
    }
}
```

而集合的循环： 

```java
public class Candy5_2 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5);
        for (Integer i : list) {
            System.out.println(i);
        }
    }
}
```

实际被编译器转换为对迭代器的调用： 

```java
public class Candy5_2 {
    public Candy5_2() {
    } 
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        Iterator iter = list.iterator();
        while(iter.hasNext()) {
            Integer e = (Integer)iter.next();  // 底层都是Object类型
            System.out.println(e);
        }
    }
}
```

> **注意**：foreach 循环写法，能够配合数组，以及所有实现了 Iterable 接口的集合类一起使用，其中 Iterable 用来获取集合的迭代器（ Iterator ） 

### 3.6 switch 字符串 

从 JDK 7 开始，switch 可以作用于**字符串和枚举类**，这个功能其实也是语法糖，例如： 

```java
public class Candy6_1 {
    public static void choose(String str) {
        switch (str) {
            case "hello": {
                System.out.println("h");
                break;
            }
            case "world": {
                System.out.println("w");
                break;
            }
        }
    }
}
```

> **注意** switch 配合 String 和枚举使用时，变量不能为null，原因分析完语法糖转换后的代码应当自然清楚 

会被编译器转换为：

```java
public class Candy6_1 {
    public Candy6_1() {
    } 
    public static void choose(String str) {
        byte x = -1;
        switch(str.hashCode()) {
            case 99162322: // hello 的 hashCode
                if (str.equals("hello")) {
                    x = 0;
                } 
                break;
            case 113318802: // world 的 hashCode
                if (str.equals("world")) {
                    x = 1;
                }
        } 
        switch(x) {
            case 0:
                System.out.println("h");
                break;
            case 1:
                System.out.println("w");
        }
    }
}
```

可以看到，执行了两遍 switch，第一遍是根据字符串的 hashCode 和 equals 将字符串的转换为相应 byte 类型，第二遍才是利用 byte 执行进行比较。

为什么第一遍时必须既比较 hashCode，又利用 equals 比较呢？hashCode 是为了提高效率，减少可能的比较；而 equals 是为了防止 hashCode 冲突，例如 BM 和 C. 这两个字符串的hashCode值都是2123 ，如果有如下代码： 

```java
public class Candy6_2 {
    public static void choose(String str) {
        switch (str) {
            case "BM": {
                System.out.println("h");
                break;
            }
            case "C.": {
                System.out.println("w");
                break;
            }
        }
    }
}
```

会被编译器转换为： 

```java
public class Candy6_2 {
    public Candy6_2() {
    } 
    public static void choose(String str) {
        byte x = -1;
        switch(str.hashCode()) {
            case 2123: // hashCode 值可能相同，需要进一步用 equals 比较
                if (str.equals("C.")) {
                    x = 1;
                } else if (str.equals("BM")) {
                    x = 0;
                }
            default:
                switch(x) {
                    case 0:
                        System.out.println("h");
                        break;
                    case 1:
                        System.out.println("w");
                }
        }
    }
}
```

### 3.7 switch 枚举 

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
      * 定义一个合成类（仅 jvm 使用，对我们不可见）
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

### 3.8 枚举类 

JDK 7 新增了枚举类，以前面的性别枚举为例： 

```java
enum Sex {  // 也是一个class
    // 实例对象，与普通类的区别为：普通类对象无穷多个；枚举类中实例对象是有限的，此处只有两个实例对象
    MALE, FEMALE  
}
```

转换后代码： 

```java
public final class Sex extends Enum<Sex> {
    public static final Sex MALE;
    public static final Sex FEMALE;
    private static final Sex[] $VALUES;
    
    static {
        MALE = new Sex("MALE", 0);
        FEMALE = new Sex("FEMALE", 1);
        $VALUES = new Sex[]{MALE, FEMALE};
    }
    
    /**
	  * Sole constructor. Programmers cannot invoke this constructor.
	  * 唯一构造函数，程序员不能调用此构造函数。
	  * It is for use by code emitted by the compiler in response to
	  * enum type declarations.
	  * 
	  * @param name - The name of this enum constant, which is the identifier used to declare it.
	  * @param ordinal - The ordinal of this enumeration constant (its position in the enum declaration, where the initial constant is assigned
	  */
    private Sex(String name, int ordinal) {
        super(name, ordinal);
    } 
    public static Sex[] values() {
        return $VALUES.clone();
    } 
    public static Sex valueOf(String name) {
        return Enum.valueOf(Sex.class, name);
    }
}
```

### 3.9 try-with-resources 

JDK 7 开始新增了对需要关闭的资源处理的特殊语法 `try-with-resources`： 

```java
try(资源变量 = 创建资源对象){
    
} catch() {
    
}
```

其中资源对象需要实现 `AutoCloseable` 接口，例如 `InputStream` 、 `OutputStream` 、`Connection` 、 `Statement` 、 `ResultSet` 等接口都实现了 `AutoCloseable` ，使用 `try-with-resources` 可以不用写 `finally` 语句块，编译器会帮助生成关闭资源代码，例如： 

```java
public class Candy9 {
    public static void main(String[] args) {
        try(InputStream is = new FileInputStream("d:\\1.txt")) {
            System.out.println(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

会被转换为： 

```java
public class Candy9 {
    public Candy9() {
    }

    public static void main(String[] args) {
        try {
            InputStream is = new FileInputStream("d:\\1.txt");
            Throwable t = null;
            try{
                System.out.println(is);
            }catch(Throwable e1){
                // t 是我们代码出现的异常
                t = e1;
                throw e1;
            }finally {
                // 判断了资源不为空
                if (is != null) {
                    // 如果我们代码有异常
                    if (t != null) {
                        try {
                            is.close();
                        } catch (Throwable e2) {
                            // 如果 close 出现异常，作为被压制异常添加
                            t.addSuppressed(e2);
                        }
                    }else {
                        // 如果我们代码没有异常
                        // close 出现的异常就是最后 catch 块中的 e
                        is.close();
                    }
                }
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

为什么要设计一个 `addSuppressed(Throwable e)` （添加被压制异常）的方法呢？是为了防止异常信息的丢失（想想 `try-with-resources` 生成的 `fianlly` 中如果抛出了异常）： 

```java
public class Test6 {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            int i = 1/0;
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
} 
class MyResource implements AutoCloseable {  // 实现了AutoCloseable
    public void close() throws Exception {
        throw new Exception("close 异常");  // 内层，被压制的异常
    }
}
```

输出： 

```java
java.lang.ArithmeticException: / by zero
	at cn.xyc.Test6.main(Test6.java:22)
	Suppressed: java.lang.Exception: close 异常  // 内层，被压制的异常
		at cn.xyc.MyResource.close(Test6.java:30)
		at cn.xyc.Test6.main(Test6.java:23)
```

如以上代码所示，两个异常信息都不会丢失。

### 3.10 方法重写时的桥接方法 

我们都知道，方法重写时对返回值分两种情况：

* 父子类的返回值完全一致
* 子类返回值可以是父类返回值的子类（比较绕口，见下面的例子） 

```java
class A {
    public Number m() {
        return 1;
    }
}

class B extends A {
    @Override
    // 子类 m 方法的返回值是 Integer 是父类 m 方法返回值 Number 的子类
    public Integer m() {
        return 2;
    }
}
```

对于子类，java 编译器会做如下处理： 

```java
class B extends A {
    public Integer m() {
        return 2;
    } 
    // 此方法才是真正重写了父类 public Number m() 方法
    public synthetic bridge Number m() {
        // 调用 public Integer m()
        return m();
    }
}
```

其中桥接方法比较特殊，仅对 java 虚拟机可见，并且与原来的 public Integer m() 没有命名冲突，可以用下面反射代码来验证： 

```java
for (Method m : B.class.getDeclaredMethods()) {
	System.out.println(m);
}
```

会输出： 

```java
public java.lang.Integer test.candy.B.m()
public java.lang.Number test.candy.B.m()
```

### 3.11 匿名内部类 

源代码：

```java
public class Candy11{
    public static void main(String[] args){
        Runnable runnable = new Runnable(){
            @Override
            public void run(){
                System.out.println("ok");
            }
        }
    }
}
```

转换后代码： 

```java
// 额外生成的类--->拆分了一个新的类
final class Candy11$1 implements Runnable {
    Candy11$1() {
    } 
    public void run() {
        System.out.println("ok");
    }
}
```

```java
public class Candy11 {
    public static void main(String[] args) {
        Runnable runnable = new Candy11$1();
    }
}
```

引用局部变量的匿名内部类，源代码： 

```java
public class Candy11 {
    public static void test(final int x) {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("ok:" + x);
            }
        };
    }
}
```

转换后代码： 

```java
// 额外生成的类
final class Candy11$1 implements Runnable {
    int val$x;  // 与上述不同，这里多了一个属性
    Candy11$1(int x) {
        this.val$x = x;
    } 
    public void run() {
        System.out.println("ok:" + this.val$x);
    }
}
```

```java
public class Candy11 {
    public static void test(final int x) {
        Runnable runnable = new Candy11$1(x);
    }
}
```

> **注意**：这同时解释了为什么匿名内部类引用局部变量时，局部变量必须是 final 的：因为在创建`Candy11$1` 对象时，将 x 的值赋值给了 `Candy11$1` 对象的 `val$x`，所以 x 不应该再发生变化了，如果变化，那么 `val$x` 属性没有机会再跟着一起变化。

## 4. 类加载阶段 

### 4.1 加载 

* 将类的字节码载入方法区中，内部采用 C++ 的 `instanceKlass` 描述 java 类，它的重要 field 有：
  * `_java_mirror` 即 java 的类镜像，例如对 String 来说，就是 String.class，作用是把 klass 暴露给 java 使用
  * `_super` 即父类
  * `_fields` 即成员变量
  * `_methods` 即方法
  * `_constants` 即常量池
  * `_class_loader` 即类加载器
  * `_vtable` 虚方法表
  * `_itable` 接口方法表
* 如果这个类还有父类没有加载，先加载父类
* 加载和链接可能是交替运行的 

> 注意
> `instanceKlass` 这样的【元数据】是存储在方法区（1.8 后的元空间内），但 `_java_mirror`是存储在堆中可以通过前面介绍的 HSDB 工具查看 

![类加载](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624038.PNG)

### 4.2 链接 

#### 验证 

验证类是否符合 JVM规范，安全性检查

用 UE 等支持二进制的编辑器修改 HelloWorld.class 的魔数，在控制台运行：

```java
>> 视频中，将魔数字 CA FE BA BE  --修改为--> CA FE BA BA
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.ClassFormatError: Incompatible magic value 3405691578 in class file cn/itcast/jvm/t5/HelloWorld
```

#### 准备 

为 static 变量分配空间，设置默认值

* static 变量在 JDK 7 之前存储于 `instanceKlass` 末尾，从 JDK 7 开始，存储于 `_java_mirror` 末尾
* **static 变量分配空间和赋值是两个步骤**，**分配**空间在**准备阶段**完成，**赋值**在**初始化阶段**完成
* **如果 static 变量是 final 的基本类型，以及字符串常量，那么编译阶段值就确定了，赋值在准备阶段完成**
* **如果 static 变量是 final 的，但属于引用类型，那么赋值也会在初始化阶段完成**

**说明示例：**

```java
public class Load {
    static int a;
    static int b = 10;
    static final int c = 20;
    static final String d = "hello";
    static final Object e = new Object();
}
```

`Recompile`上述代码，再对字节码进行反编译`javap -v -p Load.class`，结果如下：

```java
{
  static int a;        // 只有对a的声明
    descriptor: I
    flags: ACC_STATIC

  static int b; 		// 声明b静态变量
    descriptor: I
    flags: ACC_STATIC
        
  static final int c;  // static + final + 基本类型
    descriptor: I
    flags: ACC_STATIC, ACC_FINAL
    ConstantValue: int 20  // 赋值在准备阶段完成,而不是在初始化阶段
        
  static final java.lang.String d;  // static + fianl + 引用类型：字符串常量
    descriptor: Ljava/lang/String;
    flags: ACC_STATIC, ACC_FINAL
    ConstantValue: String hello  // 字符串常量的赋值也是准备阶段完成
        
  static final java.lang.Object e;  // static + fianl + 引用类型
    descriptor: Ljava/lang/Object;
    flags: ACC_STATIC, ACC_FINAL  // 引用类型,赋值在初始化阶段完成
     
  public cn.xyc.Load();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1   // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcn/xyc/Load;
    
  static {};  // cinit 构造
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: bipush        10
         2: putstatic     #2   // Field b:I  完成了对b的赋值
         5: new           #3   // class java/lang/Object  
         8: dup
         9: invokespecial #1   // Method java/lang/Object."<init>":()V
        12: putstatic     #4   // Field e:Ljava/lang/Object;
        15: return
      LineNumberTable:
        line 6: 0
        line 9: 5
}
SourceFile: "Load.java"
```

#### 解析

将常量池中的符号引用解析为直接引用 

```java
/**
 * 解析的含义
 */
public class Load2 {
    public static void main(String[] args) throws ClassNotFoundException, IOException {
        ClassLoader classloader = Load2.class.getClassLoader();
        // loadClass方法不会导致类的解析和初始化-->类D也不会被加载解析初始化
        Class<?> c = classloader.loadClass("cn.xyc.C");
        // new C();  // 会导致C被加载解析初始化-->类D被加载解析初始化
        System.in.read();
    }
}

class C {
    D d = new D();
}

class D {
}
```

**类D未被解析的情况：`// new C();`**

1. 启动程序，通过jps查看进程ID；

2. 通过下述名带打开HSBS：

   ```
   cd C:\Software\Java\jdk1.8.0_241
   java -cp ./lib/sa-jdi.jar sun.jvm.hotspot.HSDB
   ```

   > 使用参考[JVM：类加载与字节码技术-1：2.10 多态的原理]()

3. 通过`Class Browser`查看类C的确在JVM中：`[class cn.xyc.C @0x00000007c0060a18](klass=0x00000007c0060a18)`

4. 但是类D却不在内存中，即没有被加载；

5. 进入类C的常量池中查看，发现了：

   ```java
   Index    Constant Type                  Constant Value
   2        JVM_CONSTANT_UnresolvedClass   cn/xyc/D
   ```

   类D只是一个符号，未经解析的类

**类D被解析的情况：`new C();`**

1. `new C();`重新运行代码，其他操作如上；

2. 通过`Class Browser`查看类D也在JVM中了：`       class cn.xyc.D @0x00000007c0060c10`

3. 再看类C的常量池如下：

   ```
   Index    Constant Type         Constant Value
   2        JVM_CONSTANT_Class    class cn.xyc.D @0x00000007c0060c10
   ```

   类D已经有地址了，即：将常量池中的符号引用解析为**直接引用**了

### 4.3 初始化 

#### `<cinit>()V` 方法 

初始化即调用 `<cinit>()V` ，虚拟机会保证这个类的『构造方法』的线程安全 

#### 发生的时机

概括得说，类初始化是【懒惰的】

1. main 方法所在的类，总会被首先初始化

2. 首次访问这个类的静态变量或静态方法时

3. 子类初始化，如果父类还没初始化，会引发

4. 子类访问父类的静态变量，只会触发父类的初始化

5. Class.forName，会导致类的初始化

6. new 会导致初始化 

不会导致类初始化的情况 

1. 访问类的 static final 静态常量（只有是基本类型和字符串）不会触发初始化 

2. 类对象.class 不会触发初始化

3. 创建该类的数组不会触发初始化 

4. 类加载的 loadClass 方法

5. Class.forName 的参数 2 为 false 时

#### 实验

**会导致初始化情况：**

```java
public class Load3 {
    static {
        System.out.println("main init");
        // 1. 输出：main init  --> 类的静态代码块运行，说明类被加载了
    }
    // 1. main 方法所在的类，总会被首先初始化
    public static void main(String[] args)  throws ClassNotFoundException {
		// 2. 首次访问这个类的静态变量或静态方法时
        // System.out.println(A.a);
        // 2. 输出：main init \n a init \n 0
        
        // 3. 子类初始化，如果父类还没初始化，会引发
        // System.out.println(B.c);
        // 3. 输出：main init \n a init \n b init \n false
        
        // 4. 子类访问父类静态变量，只触发父类初始化
        // System.out.println(B.a);
        // 4. 输出：main init \n a init \n 0
        
        // 5. 会初始化类 B，并先初始化类 A
        // Class.forName("cn.xyc.B");
		// 5. 输出：main init \n a init \n b init
    }
}

class A {
    static int a = 0;
    static {
        System.out.println("a init");
    }
}

class B extends A {
    final static double b = 5.0;
    static boolean c = false;
    static {
        System.out.println("b init");
    }
}
```

**不会导致类初始化的情况：**

```java
public class Load3 {
    public static void main(String[] args)  throws ClassNotFoundException {
		// 1. 静态常量不会触发初始化
        System.out.println(B.b);  
        // 1. 输出：只打印5.0，B类中的静态代码块没有执行，说明类B没被加载
        
        // 2. 类对象.class 不会触发初始化
        System.out.println(B.class);
        // 2. 输出：class cn.xyc.B，B类中的静态代码块没有执行，说明类B没被加载
        
        // 3. 创建该类的数组不会触发初始化
        System.out.println(new B[0]);
        // 3. 输出：[Lcn.xyc.B;@7f31245a，B类中的静态代码块没有执行，说明类B没被加载
        
        // 4. 不会初始化类B，但会加载 B、A
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        cl.loadClass("cn.xyc.B");
        // 4. 输出：无
        
        // 5. 不会初始化类 B，但会加载 B、A
        ClassLoader c2 = Thread.currentThread().getContextClassLoader();
        Class.forName("cn.xyc.B", false, c2);
        // 5. 输出：无
    }
}

class A {
    static int a = 0;
    static {
        System.out.println("a init");
    }
}

class B extends A {
    final static double b = 5.0;
    static boolean c = false;
    static {
        System.out.println("b init");
    }
}
```

### 4.4 练习 

从字节码分析，使用 a，b，c 这三个常量是否会导致 E 初始化 

```java
public class Load4 {
    public static void main(String[] args) {
        System.out.println(E.a);  // 不会
        System.out.println(E.b);  // 不会
        System.out.println(E.c);  // 会

    }
}

class E {
    public static final int a = 10;
    public static final String b = "hello";
    public static final Integer c = 20;  // Integer.valueOf(20)
    static {
        System.out.println("init E");
    }
}
```

典型应用 - 完成懒惰初始化单例模式

```java
public final class Singleton{
    private Singleton(){}
    // 内部类中保存单例
    private static class LazyHolder{
        static final Singleton INSTANCE = new Singleton();
        static{
            System.out.println("lazy holder init")
        }
    }
    
    // 第一次调用 getInstance 方法，才会导致内部类加载和初始化其静态成员
    public static Singleton getInstance(){
        return LazyHolder.INSTANCE;
    }
}
```

以上的实现特点是： 

* 懒惰实例化
* 初始化时的线程安全是有保障的，由类加载器保证其安全性

## 5. 类加载器 

以 JDK 8 为例： 

| 名称                                      | 负责加载哪的类        | 说明                          |
| ----------------------------------------- | --------------------- | ----------------------------- |
| 启动类加载器：Bootstrap ClassLoader       | JAVA_HOME/jre/lib     | 无法直接访问                  |
| 扩展类加载器：Extension ClassLoader       | JAVA_HOME/jre/lib/ext | 上级为 Bootstrap，显示为 null |
| 应用程序类加载器：Application ClassLoader | classpath             | 上级为 Extension              |
| 自定义类加载器                            | 自定义                | 上级为 Application            |

### 5.1 启动类加载器 

用 Bootstrap 类加载器加载类： 

```java
package cn.xyc;

public class F {
    static {
        System.out.println("bootstrap F init");
    }
}
```

执行：

```java
public class Load5_1 {
    public static void main(String[] args) throws ClassNotFoundException {
        // Class.forName 完成类的加载链接初始化操作
        Class<?> aClass = Class.forName("cn.xyc.F");
        System.out.println(aClass.getClassLoader());
        // 若类加载器是应用程序加载器输出：AppClassLoader
        // 若类加载器是扩展类加载器输出：ExtClassLoader
        // 但是启动类加载器类加载器java无法直接访问，因此会打印出null
    }
}
```

输出：

```java
E:\...这个路径是啥...?> java -Xbootclasspath/a:. cn.itcast.jvm.t3.load.Load5
bootstrap F init  // F类被加载
null  			  // 变成了启动类加载器加载
```

* `-Xbootclasspath` 表示设置 `bootclasspath`
* 其中 `/a:.` 表示将当前目录追加至 bootclasspath 之后
* 可以用这个办法替换核心类
  * `java -Xbootclasspath:<new bootclasspath>`
  * `java -Xbootclasspath/a:<追加路径>`
  * `java -Xbootclasspath/p:<追加路径> `

### 5.2 扩展类加载器 

```java
package cn.xyc;

public class G {
    static {
        System.out.println("classpath G init");
    }
}
```

执行：

```java
package cn.xyc;

public class Load5_2 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> aClass = Class.forName("cn.xyc.G");
        System.out.println(aClass.getClassLoader());
    }
}
```

输出：

```java
classpath G init  // G类被加载
sun.misc.Launcher$AppClassLoader@18b4aac2 // 应用程序类加载器
```

写一个同名的类 :

```java
package cn.xyc;

public class G {
    static {
        // System.out.println("classpath G init");
        System.out.println("Ext G init");
    }
}
```

打个 jar 包 

```java
C:\Users\ZhuCC\Desktop\Java\itcastbingfa\Code\JVM\target\classes>jar -cvf my.jar cn/xyc/G.class
已添加清单
正在添加: cn/xyc/G.class(输入 = 457) (输出 = 312)(压缩了 31%)
```

将 jar 包拷贝到 `JAVA_HOME/jre/lib/ext` 

重新执行 `Load5_2` 

输出：

```
Ext G init
sun.misc.Launcher$ExtClassLoader@29453f44
```

### 5.3 双亲委派模式 

所谓的双亲委派，就是指调用类加载器的 loadClass 方法时，查找类的规则：

先由上级完成类的加载，若上级没有这个类，则由本级的类加载器来完成类的加载

> 注意：
>
> 这里的双亲，翻译为上级似乎更为合适，因为它们并没有继承关系

```java
protected Class<?> loadClass(String name, boolean resolve) 
    throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // 1. 检查该类是否已经加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 2. 有上级的话，委派上级 loadClass
                    c = parent.loadClass(name, false);
                } else {
                    	// 3. 如果没有上级了（ExtClassLoader），则委派BootstrapClassLoader 
                        c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
            } 
            if (c == null) {
                long t1 = System.nanoTime();
                // 4. 每一层找不到，调用 findClass 方法（每个类加载器自己扩展）来加载
                c = findClass(name);
                // 5. 记录耗时
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        } 
        if (resolve) {
            resolveClass(c);
        } 
        return c;
    }
}
```

例如：

```java
public class Load5_3 {
    public static void main(String[] args) throws ClassNotFoundException {
        System.out.println(Load5_3.class.getClassLoader());
        Class<?> aClass = Load5_3.class.getClassLoader().loadClass("cn.xyc.H");
        System.out.println(aClass.getClassLoader());
    }
}
```

1. `sun.misc.Launcher$AppClassLoader //1` 处， 开始查看已加载的类，结果没有，即为null；
2. `sun.misc.Launcher$AppClassLoader // 2 `处，委派上级`sun.misc.Launcher$ExtClassLoader.loadClass() `
3. `sun.misc.Launcher$ExtClassLoader // 1` 处，查看已加载的类，结果没有；
4. `sun.misc.Launcher$ExtClassLoader // 2 `处，委派上级，但是其上级为null，表示已经到启动类加载器了，进入`findBootstrapClassOrNull(name);  // 3`处，委托启动类加载器去查找；
5. `BootstrapClassLoader` 是在 `JAVA_HOME/jre/lib` 下找 H 这个类，显然没有；
6. `sun.misc.Launcher$ExtClassLoader // 4` 处，调用自己的 findClass 方法，是在`JAVA_HOME/jre/lib/ext` 下找 H 这个类，显然没有， 回到 `sun.misc.Launcher$AppClassLoader的 // 2` 处
7. 继续执行到 `sun.misc.Launcher$AppClassLoader // 4` 处，调用它自己的 findClass 方法，在classpath 下查找，找到了 

### 5.4 线程上下文类加载器

> 后面的内容没有很好的了解，待后续再回来理解一下:dog2:

我们在使用 JDBC 时，都需要加载 Driver 驱动，不知道你注意到没有，不写

```java
Class.forName("com.mysql.jdbc.Driver")
```

也是可以让 `com.mysql.jdbc.Driver` 正确加载的，你知道是怎么做的吗？ 

让我们追踪一下源码：

```java
public class DriverManager {
    // 注册驱动的集合
    private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<>();
    // 初始化驱动
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    // ...
}
```

先不看别的，看看 DriverManager 的类加载器：

```java
System.out.println(DriverManager.class.getClassLoader());
```

打印 null，表示它的类加载器是 `Bootstrap ClassLoader`，会到 `JAVA_HOME/jre/lib` 下搜索类，但` JAVA_HOME/jre/lib` 下显然没有 `mysql-connector-java-5.1.47.jar` 包，这样问题来了，在`DriverManager` 的静态代码块中，怎么能正确加载 `com.mysql.jdbc.Driver `呢？ 

继续看 `loadInitialDrivers()` 方法： 

```java
private static void loadInitialDrivers() {
    String drivers;
    try {
        drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
        	public String run() {
                return System.getProperty("jdbc.drivers");
            }
		});
    } catch (Exception ex) {
        drivers = null;
    }
    // 1) 使用 ServiceLoader 机制加载驱动，即SPI
    AccessControler.doPrivileged(new PrivilegedAction<Void>)(){
        public Void run() {
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            Iterator<Driver> driversIterator = loadedDrivers.iterator();
            try{
                while(driversIterator.hasNext()) {
                    driversIterator.next();
                }
            } catch(Throwable t) {
                // Do nothing
            } 
            return null;
        }
    });
    println("DriverManager.initialize: jdbc.drivers = " + drivers);
    // 2）使用 jdbc.drivers 定义的驱动名加载驱动
    if (drivers == null || drivers.equals("")) {
        return;
    } 
    String[] driversList = drivers.split(":");
    println("number of Drivers:" + driversList.length);
    for (String aDriver : driversList) {
        try {
            println("DriverManager.Initialize: loading " + aDriver);
            // 这里的 ClassLoader.getSystemClassLoader() 就是应用程序类加载器
            Class.forName(aDriver, true, ClassLoader.getSystemClassLoader());
        } catch (Exception ex) {
            println("DriverManager.Initialize: load failed: " + ex);
        }
    }
}
```

先看 2）发现它最后是使用 `Class.forName` 完成类的加载和初始化，关联的是应用程序类加载器，因此可以顺利完成类加载 

再看 1）它就是大名鼎鼎的 Service Provider Interface （SPI） 

> 即打破了双亲委派机制

约定如下，在 jar 包的 META-INF/services 包下，以接口全限定名名为文件，文件内容是实现类名称 

![SPI](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624303.PNG)

这样就可以使用 

```java
ServiceLoader<接口类型> allImpls = ServiceLoader.load(接口类型.class);
Iterator<接口类型> iter = allImpls.iterator();
while(iter.hasNext()) {
    iter.next();
}
```

来得到实现类，体现的是【面向接口编程+解耦】的思想，在下面一些框架中都运用了此思想： 

* JDBC
* Servlet 初始化器
* Spring 容器
* Dubbo（对 SPI 进行了扩展） 

接着看 ServiceLoader.load 方法： 

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    // 获取线程上下文类加载器
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

线程上下文类加载器是当前线程使用的类加载器，默认就是应用程序类加载器，它内部又是由 `Class.forName` 调用了线程上下文类加载器完成类加载，具体代码在 `ServiceLoader` 的内部类 `LazyIterator` 中： 

```java
private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        // 这里：loader即线程上下文类加载器即应用程序类加载器
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service, "Provider" + cn + "not a subtype");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
             "Provider " + cn + " not a subtype");
    } 
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
             "Provider " + cn + " could not be instantiated",
             x);
    } 
    throw new Error(); // This cannot happen
}
```

### 5.5 自定义类加载器 

问问自己，什么时候需要自定义类加载器

* 想加载非 classpath 随意路径中的类文件
* 都是通过接口来使用实现，希望解耦时，常用在框架设计
* 这些类希望予以隔离，不同应用的同名类都可以加载，不冲突，常见于 tomcat 容器 

步骤： 

1. 继承 ClassLoader 父类
2. 要遵从双亲委派机制，重写 findClass 方法
   * 注意不是重写 loadClass 方法，否则不会走双亲委派机制
3. 读取类文件的字节码
4. 调用父类的 defineClass 方法来加载类
5. 使用者调用该类加载器的 loadClass 方法 

示例：

准备好两个类文件放入 `C:\Users\ZhuCC\Desktop`，它实现了 `java.util.Map` 接口：

```java
import java.util.AbstractMap;
import java.util.Map;
import java.util.Set;

public class MapImpl1  extends AbstractMap implements Map{
    @Override
    public Set<Entry> entrySet() {
        return null;
    }

    static {
        System.out.println("MapImpl1 init");
    }
}

import java.util.AbstractMap;
import java.util.Map;
import java.util.Set;

public class MapImpl2 extends AbstractMap implements Map {
    @Override
    public Set<Entry> entrySet() {
        return null;
    }

    static {
        System.out.println("MapImpl1 init");
    }
}
```

可以先反编译看一下： 

```java
C:\Users\ZhuCC\Desktop>javap MapImpl1.class
Compiled from "MapImpl1.java"
public class MapImpl1 extends java.util.AbstractMap implements java.util.Map {
  public MapImpl1();
  public java.util.Set<java.util.Map$Entry> entrySet();
  static {};
}

C:\Users\ZhuCC\Desktop>javap MapImpl2.class
Compiled from "MapImpl2.java"
public class MapImpl2 extends java.util.AbstractMap implements java.util.Map {
  public MapImpl2();
  public java.util.Set<java.util.Map$Entry> entrySet();
  static {};
}
```

自定义类加载器实现：

```java
public class Load7 {

    public static void main(String[] args) throws Exception{
        MyClassLoader classLoader = new MyClassLoader();
        Class<?> c1 = classLoader.loadClass("MapImpl1");
        Class<?> c2 = classLoader.loadClass("MapImpl1");  // 第二次加载时该类已经被放到了自定义类加载器的缓存中
        System.out.println(c1 == c2);  // true
        
        MyClassLoader classLoader2 = new MyClassLoader();
        Class<?> c3 = classLoader2.loadClass("MapImpl1");
        // 确定类相同的方式：包名/类名/类加载器为同一个
        System.out.println(c1 == c3);  // false
 
        // 创建一个MapImpl1对象
        // 静态代码块被执行，输出：MapImpl1 init
        c1.newInstance();
    }
}

class MyClassLoader extends ClassLoader{
	// 自定义类加载器
    @Override  // name 就是类名称
    protected Class<?> findClass(String name) throws ClassNotFoundException {

        String path = "C:\\Users\\ZhuCC\\Desktop\\" + name + ".class";

        try {
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            Files.copy(Paths.get(path), os);
            // 得到字节数组
            byte[] bytes = os.toByteArray();
            // byte[] -> *.class
            return defineClass(name, bytes, 0, bytes.length);
        } catch (IOException e) {
            e.printStackTrace();
            throw new ClassNotFoundException("类文件未找到", e);
        }
    }
}
```

## 6. 运行期优化 

### 6.1 即时编译 

#### 分层编译-TieredCompilation

先来个例子：

```java
public class JIT1 {
    // -XX:+PrintCompilation -XX:-DoEscapeAnalysis
    public static void main(String[] args) {
        for (int i = 0; i < 200; i++) {
            long start = System.nanoTime();
            for (int j = 0; j < 1000; j++) {
                new Object();
            }
            long end = System.nanoTime();
            System.out.printf("%d\t%d\n",i,(end - start));
        }
    }
}
```

输出结果：

```
0	119600
1	21200
2	20300
3	20400
4	20900
5	20400
6	20900
7	20800
8	19800
9	20800
10	20600
11	20800
12	19200
13	20600
14	20600
15	22600
16	23000
17	21100
18	20700
19	20500
20	20500
21	20700
22	22500
23	21200
24	21100
25	20700
26	22800
27	22100
28	21000
29	20900
30	21000
31	23700
32	22300
33	20900
34	21000
35	56900
36	21500
37	20700
38	20900
39	20600
40	20800
41	20800
42	20800
43	20800
44	18500
45	20600
46	21000
47	56700
48	20000
49	22700
50	21000
51	28200
52	19200
53	20000
54	20500
55	27400
56	20900
57	19700
58	20900
59	20100
60	24000
61	21700
62	27800
63	21500
64	21400
65	20800
66	28000
67	20500
68	24600
69	20200
70	20800
71	22600
72	21000
73	20900
74	15800
75	9500
76	10400
77	13200
78	11000
79	7600
80	9300
81	9900
82	9300
83	7400
84	9800
85	11400
86	9900
87	7900
88	9400
89	9200
90	9000
91	8400
92	28300
93	11500
94	8900
95	9100
96	9200
97	9300
98	9400
99	32900
100	111000
101	12500
102	9600
103	9600
104	8800
105	9800
106	10000
107	8300
108	11500
109	75800
110	9500
111	9900
112	8300
113	8500
114	8900
115	9100
116	9300
117	9400
118	10600
119	9300
120	10200
121	8700
122	10700
123	9500
124	9300
125	9100
126	57000
127	9900
128	300
129	400
// ....
197	300
198	300
199	300
```

原因是什么呢？ 

JVM 将执行状态分成了 5 个层次：

* 0 层，解释执行（Interpreter）
* 1 层，使用 C1 即时编译器编译执行（不带 profiling）
* 2 层，使用 C1 即时编译器编译执行（带基本的 profiling）
* 3 层，使用 C1 即时编译器编译执行（带完全的 profiling）
* 4 层，使用 C2 即时编译器编译执行 

> profiling 是指在运行过程中收集一些程序执行状态的数据，例如【方法的调用次数】，【循环的回边次数】等 

**即时编译器（JIT）与解释器的区别** 

* 解释器是将字节码解释为机器码，下次即使遇到相同的字节码，仍会执行重复的解释
* JIT 是将一些字节码编译为机器码，并存入 Code Cache，下次遇到相同的代码，直接执行，无需再编译
* 解释器是将字节码解释为针对所有平台都通用的机器码
* JIT 会根据平台类型，生成平台特定的机器码 

**一方面**：对于占据大部分的不常用的代码，我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行；**另一方面**：对于仅占据小部分的热点代码，我们则可以将其编译成机器码，以达到理想的运行速度。 

执行效率上简单比较一下 **Interpreter < C1 < C2**，总的目标是发现热点代码（hotspot 名称的由来），优化之。

刚才的一种优化手段称之为【逃逸分析】，发现新建的对象是否逃逸。可以使用 `-XX:-DoEscapeAnalysis` 关闭逃逸分析，再运行刚才的示例观察结果 

参考资料：https://docs.oracle.com/en/java/javase/12/vm/java-hotspot-virtual-machine-performance-enhancements.html#GUID-D2E3DC58-D18B-4A6C-8167-4A1DFB4888E4 

#### 方法内联-Inlining 

```java
private static int square(final int i) {
	return i * i;
}

// 调用：
System.out.println(square(9));
```

如果发现 square 是热点方法，并且长度不太长时，会进行内联，所谓的内联就是把方法内代码拷贝、粘贴到调用者的位置： 

```java
System.out.println(9*9);
```

还能够进行常量折叠（constant folding）的优化 

```java
System.out.println(81);
```

实验：

```java
public class JIT2 {
    // -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining -XX:CompileCommand=dontinline,*JIT2.square  // 不使用内敛的JVM参数
    // -XX:+PrintCompilation

    public static void main(String[] args) {
        int x = 0;
        for (int i = 0; i < 500; i++) {
            long start = System.nanoTime();
            for (int j = 0; j < 1000; j++) {
                x = square(9);
            }
            long end = System.nanoTime();
            System.out.printf("%d\t%d\t%d\n",i,x,(end - start));
        }
    }

    private static int square(final int i) {
        return i * i;
    }
}
```

结果：

```
0	81	26800
1	81	24400
2	81	13900
...
65	81	13300
66	81	6000
67	81	2500
...
494	81	0
495	81	0
496	81	0
497	81	0
498	81	0
499	81	100
```

#### 字段优化

> 没仔细看...:dog2:

JMH 基准测试请参考：http://openjdk.java.net/projects/code-tools/jmh/

创建 maven 工程，添加依赖如下 

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.21</version>
</dependency>

<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.21</version>
    <scope>provided</scope>
</dependency>
```

编写基准测试代码： 

```java
@Warmup(iterations = 2, time = 1)  // 使程序热身 JIT对代码进行优化
@Measurement(iterations = 5, time = 1)  // 对程序进行5轮测试
@State(Scope.Benchmark)
public class Benchmark1 {

    int[] elements = randomInts(1_000);

    private static int[] randomInts(int size) {
        Random random = ThreadLocalRandom.current();
        int[] values = new int[size];
        for (int i = 0; i < size; i++) {
            values[i] = random.nextInt();
        }
        return values;
    }

    @Benchmark
    public void test1() {
        for (int i = 0; i < elements.length; i++) {  // 直接操作成员变量
            doSum(elements[i]);
        }
    }

    @Benchmark
    public void test2() {
        int[] local = this.elements;  // 局部变量
        for (int i = 0; i < local.length; i++) {
            doSum(local[i]);
        }
    }

    @Benchmark
    public void test3() {
        for (int element : elements) {  // foreach方式
            doSum(element);
        }
    }

    static int sum = 0;
    @CompilerControl(CompilerControl.Mode.INLINE)  // 允许方法内联
    static void doSum(int x) {
        sum += x;
    }

    public static void main(String[] args) throws RunnerException {
        org.openjdk.jmh.runner.options.Options opt = new OptionsBuilder()
                .include(Benchmark1.class.getSimpleName())
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```

首先启用 doSum 的方法内联，测试结果如下（每秒吞吐量，分数越高的更好）： 

```java
//                            吞吐量得分        误差    单位：每s能调用的吞吐量
Benchmark          Mode  Cnt        Score        Error  Units
Benchmark1.test1  thrpt    5  3507240.120 ± 430356.848  ops/s
Benchmark1.test2  thrpt    5  3610461.567 ± 191846.685  ops/s
Benchmark1.test3  thrpt    5  3498577.056 ± 810891.901  ops/s
```

接下来禁用 doSum 方法内联：

```java
static int sum = 0;
@CompilerControl(CompilerControl.Mode.DONT_INLINE)  // 不允许方法内联
static void doSum(int x) {
    sum += x;
}
```

测试结果如下： 

```java
Benchmark          Mode  Cnt       Score       Error  Units
Benchmark1.test1  thrpt    5  443462.113 ± 62327.142  ops/s
Benchmark1.test2  thrpt    5  567664.188 ± 63040.278  ops/s
Benchmark1.test3  thrpt    5  567800.181 ± 24416.939  ops/s
```

分析：

在刚才的示例中，doSum 方法是否内联会影响 elements 成员变量读取的优化：

如果 doSum 方法内联了，刚才的 test1 方法会被优化成下面的样子（伪代码）： 

```java
@Benchmark
public void test1() {
    // elements.length 首次读取会缓存起来 -> int[] local
    for (int i = 0; i < elements.length; i++) { //后续999次求长度<-local
        sum += elements[i]; // 1000次取下标i的元素<-local
    }
}
```

可以节省 1999 次 Field 读取操作

但如果 doSum 方法没有内联，则不会进行上面的优化 

### 6.2 反射优化 

```java
public class Reflect1 {
    public static void foo() {
        System.out.println("foo...");
    }

    public static void main(String[] args) throws Exception {
        Method foo = Reflect1.class.getMethod("foo");
        for (int i = 0; i <= 16; i++) {
            System.out.printf("%d\t", i);
            foo.invoke(null);
        }
        System.in.read();
    }
}
```

foo.invoke 前面 0~15 次调用使用的是 `MethodAccessor` 的 `NativeMethodAccessorImpl` 实现 

```java
@CallerSensitive
public Object invoke(Object obj, Object... args) 
    throws IllegalAccessException, IllegalArgumentException, InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}

class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private final Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method var1) {
        this.method = var1;
    }

    public Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException {
    	// inflationThreshold 膨胀阈值，默认 15
        if (++this.numInvocations > ReflectionFactory.inflationThreshold() && !ReflectUtil.isVMAnonymousClass(this.method.getDeclaringClass())) {
            // 使用 ASM 动态生成的新实现代替本地实现，速度较本地实现快 20 倍左右
            MethodAccessorImpl var3 = (MethodAccessorImpl)(new MethodAccessorGenerator()).generateMethod(this.method.getDeclaringClass(), this.method.getName(), this.method.getParameterTypes(), this.method.getReturnType(), this.method.getExceptionTypes(), this.method.getModifiers());
            this.parent.setDelegate(var3);
        }
		// 调用本地实现
        return invoke0(this.method, var1, var2);
    }

    void setParent(DelegatingMethodAccessorImpl var1) {
        this.parent = var1;
    }

    private static native Object invoke0(Method var0, Object var1, Object[] var2);
}
```

当调用到第 16 次（从0开始算）时，会采用运行时生成的类代替掉最初的实现，可以通过 debug 得到类名为 `sun.reflect.GeneratedMethodAccessor1` 

![反射优化](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624728.PNG)

可以使用阿里的 arthas 工具：

```bash
C:\Users\ZhuCC\Desktop>java -jar arthas-boot.jar
[INFO] arthas-boot version: 3.3.9
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 18800
  [2]: 28352 cn.xyc.Reflect1
  [3]: 30272 org.jetbrains.jps.cmdline.Launcher
  [4]: 17736 sun.jvm.hotspot.HSDB
  [5]: 8568
  [6]: 29308 org.jetbrains.idea.maven.server.RemoteMavenServer
```

选择 2 回车表示分析该进程 

```java
[INFO] Download arthas success.
[INFO] arthas home: C:\Users\ZhuCC\.arthas\lib\3.4.1\arthas
[INFO] Try to attach process 28352
[INFO] Attach process 28352 success.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'


wiki      https://arthas.aliyun.com/doc
tutorials https://arthas.aliyun.com/doc/arthas-tutorials.html
version   3.4.1
pid       28352
time      2020-09-15 20:11:05
```

再输入【jad + 类名】来进行反编译 

```java
[arthas@28352]$ jad sun.reflect.GeneratedMethodAccessor1

ClassLoader:
+-sun.reflect.DelegatingClassLoader@6f94fa3e
  +-sun.misc.Launcher$AppClassLoader@18b4aac2
    +-sun.misc.Launcher$ExtClassLoader@61bbe9ba

Location:


/*
 * Decompiled with CFR.
 *
 * Could not load the following classes:
 *  cn.xyc.Reflect1
 */
package sun.reflect;

import cn.xyc.Reflect1;
import java.lang.reflect.InvocationTargetException;
import sun.reflect.MethodAccessorImpl;

public class GeneratedMethodAccessor1
extends MethodAccessorImpl {
    /*
     * Loose catch block
     * Enabled aggressive block sorting
     * Enabled unnecessary exception pruning
     * Enabled aggressive exception aggregation
     * Lifted jumps to return sites
     */
    public Object invoke(Object object, Object[] arrobject) throws InvocationTargetException {
        // 比较奇葩的做法，如果有参数，那么抛非法参数异常
        block4: {
            if (arrobject == null || arrobject.length == 0) break block4;
            throw new IllegalArgumentException();
        }
        try {
            // 可以看到，已经是直接调用了，而不是反射调用了
            Reflect1.foo();
            // 因为没有返回值
            return null;
        }
        catch (Throwable throwable) {
            throw new InvocationTargetException(throwable);
        }
        catch (ClassCastException | NullPointerException runtimeException) {
            throw new IllegalArgumentException(super.toString());
        }
    }
}

Affect(row-cnt:1) cost in 524 ms.
```

> **注意**
>
> * 通过查看 `ReflectionFactory` 源码可知 `sun.reflect.noInflation` 可以用来禁用膨胀（直接生成 `GeneratedMethodAccessor1`，但首次生成比较耗时，如果仅反射调用一次，不划算）
> * `sun.reflect.inflationThreshold` 可以修改膨胀阈值 