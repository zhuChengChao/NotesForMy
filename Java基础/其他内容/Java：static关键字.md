# Java：static关键字

> 对 Java 中的 **static** 关键字，做一个微不足道的小小小小记

## static 修饰变量

**静态变量：**是被 static 修饰的变量，也称为类变量，它属于类，因此不管创建多少个对象，静态变量在内存中有且仅有一个拷贝；静态变量可以实现让多个对象共享内存；可以直接通过 `类名.静态变量名` 来访问它。

> **静态的注意事项**：
>
> 1. 静态方法只能访问静态成员（包括静态成员变量和静态成员方法），不能访问非静态成员或方法；非静态方法可以访问静态也可以访问非静态方法或成员。
> 2. 静态方法中不能出现 this，super 关键字。因为静态是优先于对象存在的，所以不能出现 this，super 关键字。
> 3. 主函数是静态的。

**实例变量：**属于某一实例，需要先创建对象，然后通过对象才能访问到它。

总之：**静态变量属于类的级别，而实例变量属于对象的级别**。

## static 修饰方法

**静态方法**：被 static 修饰的方法，静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字；

> 再次注意：
>
> 1. 被 static 修饰的静态变量与静态方法为静态资源，**静态资源是类初始化的时候加载的，而非静态资源是类new的时候加载的**；
> 2. 静态方法不能引用非静态资源：非静态资源为new的时候才会产生的东西，对于初始化后就存在的静态资源来说，根本不认识它。
> 3. 静态方法里面可以引用静态资源；
> 4. 非静态方法里面可以引用静态资源：非静态方法就是实例方法，那是 new 之后才产生的，那么属于类的内容它都认识。

## static 修饰代码块

**静态代码块**：静态代码块在类初始化时运行一次；

* 初始化顺序：静态变量和静态代码块优先于实例变量和普通语句块，静态变量和静态代码块的初始化顺序取决于它们在代码中的顺序。

  ```java
  public class InitialOrderTest {
  	
      // 1：被调用
      public static String staticField = "静态变量";
      // 2：被调用
      public static String staticMethod = staticMethod();
  	
      // 4:只有new的时候才会输出
      {
          System.out.println("普通语句块");
      }
  	
      // 3:被调用
      static {
          System.out.println(staticField);
          System.out.println("静态语句块"); 
      }
  
      public String field = "实例变量";
  
      public static String staticMethod(){
          System.out.println("静态变量调用静态方法");
          return "staticMethod";
      }
  
      // 最后才是构造函数的初始化
      public InitialOrderTest() {
          // 5:只有new的时候才会输出
          System.out.println("构造函数");    
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

* 静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问

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

  > 对上述现象做进一步解释：
  >
  > 对上述代码查看静态代码块的字节码如下：
  >
  > ```java
  > {
  >  // 类加载的链接中的 **准备阶段**，为static变量分配空间，即staticField已经分配好了空间
  >  public static java.lang.String staticField;  
  >    descriptor: Ljava/lang/String;
  >    flags: ACC_PUBLIC, ACC_STATIC
  > 
  >  public cn.xyc.other.InitialOrderTest();
  >  // ... 略
  >  public static void main(java.lang.String[]);
  >  // ... 略
  > 
  >  // 在类的初始化阶段调用<cinit>()V完成对静态变量的赋值
  >  static {};
  >    descriptor: ()V
  >    flags: ACC_STATIC
  >    Code:
  >      stack=2, locals=0, args_size=0
  >         0: ldc           #4  // String ?????
  >         2: putstatic     #5  // Field staticField:Ljava/lang/String;
  >         5: getstatic     #6  // Field java/lang/System.out:Ljava/io/PrintStream;
  >         8: ldc           #7  // String 静态语句块
  >        10: invokevirtual #8  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
  >        13: getstatic     #6  // Field java/lang/System.out:Ljava/io/PrintStream;
  >        16: getstatic     #5  // Field staticField:Ljava/lang/String;
  >        19: invokevirtual #8  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
  >        22: return
  >      LineNumberTable:
  >        line 9: 0
  >        line 10: 5
  >        line 16: 13
  >        line 17: 22
  > }
  > SourceFile: "InitialOrderTest.java"
  > ```

* 静态代码块是严格按照：父类静态代码块->子类静态代码块的顺序加载的，且只加载一次。

  **对初始化顺序的补充：**存在继承的情况下，初始化顺序为：

  1. 父类（静态变量、静态代码块）
  2. 子类（静态变量、静态代码块）
  3. 父类（实例变量、普通代码块）
  4. 父类（构造函数）
  5. 子类（实例变量、普通代码块）
  6. 子类（构造函数）

## static 修饰内部类

静态内部类：非静态内部类依赖于外部类的实例，**而静态内部类不需要**。但静态内部类不能访问外部类的非静态的变量和方法；

静态内部类是不需要依赖于外部类的，通常称为嵌套类（nested class）；

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

        //静态内部类非静态方法
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

> 通过静态内部类还可以实现安全的单例模式，后续再写

## 参考

https://www.cnblogs.com/jasonboren/p/11052500.html

https://www.cnblogs.com/xrq730/p/4820992.html

https://www.cnblogs.com/GrimMjx/p/10105626.html