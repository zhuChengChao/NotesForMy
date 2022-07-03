# Java：抽象类和接口

> 对 Java 中的 **抽象类和接口**，做一个微不足道的小小小小记

**抽象类**：使用 abstract 修饰，子类用 extends 继承；

**接口**：使用 interface 修饰，采用 implements 实现；

## 构造函数

1. 抽象类中可以定义构造函数（但是抽象类不能被实例化）；
2. 接口不能定义构造函数；

> 演示一下：
>
> ```java
> // 抽象类
> public abstract class AbstractTest {
>        // 正常
>        public AbstractTest() {
>            System.out.println("abstract class");
>        }
> }
> 
> /******************分割线*********************/
> 
> // 接口
> public interface InterfaceTest {
>        // error：Interface abstract method cannot have body
>        public InterfaceTest(){}
> }
> ```

## 成员变量

1. 抽象类中的成员权限可以是 public、默认、protected、private

   > 注：抽象类中**抽象方法就是为了重写**，所以**不能被 private 修饰**；

2. 而接口中的成员**只可以是 public**（方法默认：public abstract 、成员变量默认：public static final）；

> 演示一下
>
> ```java
> // 抽象类：
> public abstract class AbstractTest {
>        public AbstractTest() {
>            System.out.println("abstract class");
>        }
> 
>        public int a = 0;
>        int b = 0;
>        protected int c = 0;
>        private int d = 0;
>        // 成员变量是可以用private修饰的
>        private String e = "hello";
> 
>        public void out(){
>            System.out.println(d);
>            System.out.println(e);
>        }
> }
> 
> public class Demo extends AbstractTest {
> 
>        public static void main(String[] args) {
>            Demo demo = new Demo();
>            demo.out();
>        }
> }
> 
> // 输出：
> // abstract class
> // 0
> // hello
> 
> /******************分割线*********************/
> 
> // 接口：
> public interface InterfaceTest {
> 
>        int a = 10;
>        // error:Modifier 'protected' not allowed here
>        protected int b = 10;
>        // error:Modifier 'default' not allowed here
>        default int c = 20;
>        // info:Modifier 'public' is redundant for interface field
>        public int d = 30;
>        // public static final可以不加，是默认的
>        public static final int e = 40;
> }
> ```

## 成员方法

1. 抽象类中可以有抽象方法和具体方法，
2. 而接口中只能有抽象方法（public abstract），**但在 JDK1.8中，允许在接口中包含带有具体实现的方法，使用 default 修饰，这类方法就是默认方法。**
3. 抽象类中可以包含静态方法；
4. 接口中不可以包含静态方法，**同样在JDK1.8 以后可以包含**，之前不能包含是因为，接口不可以实现方法，只可以定义方法，所以不能使用静态方法（因为静态方法必须实现）。现在可以包含了，**只能直接用接口调用静态方法**。
5. JDK1.8中，接口仍然不可以包含静态代码块。

> 演示一下：
>
> ```java
> // 抽象类
> public abstract class AbstractTest {
>        public AbstractTest() {
>            System.out.println("abstract class");
>        }
> 
>        public int a = 0;
>        int b = 0;
>        protected int c = 0;
>        private int d = 0;
>        // 成员变量是可以用private修饰的
>        private String e = "hello";
> 
>        // 具体方法，在继承AbstractTest的类中可以直接调用
>        public void method1(){
>            System.out.println(d);
>            System.out.println(e);
>        }
> 
>        // 静态方法，可以通过 AbstractTest.method2()进行调用
>        public static String f = "world";
>        public static void method2(){
>            System.out.println(f);
>        }
> 
>        // 抽象方法，后续抽象方法必须是继承类中实现，否则继承类也是个抽象类
>        public abstract void method3();
>        abstract void method4();
>        protected abstract void method5();
>        // error: illegal combination of midifiers: 'abstract' and 'private'
>        // private abstract void method6();
> }
> 
> /******************分割线*********************/
> 
> // 接口
> public interface InterfaceTest {
> 
>        int a = 10;
>        // public static final可以不加，可以不加
>        public static final int b = 20;
> 
>        // 抽象方法：public abstract可以不加，是默认，且只能用这个修饰
>        public abstract void method1();
> 
>        // 默认方法：必须只能用default修饰
>        default void method2(){
>            System.out.println("默认方法");
>        }
> 
>        // 静态方法: 通过InterfaceTest.method3()调用
>        // 只能用public去修饰，可加可不加，其他的 protected/private/default都是不允许的
>        public static void method3(){
>            System.out.println("静态方法");
>        }
> }
> 
> 
> public class Demo implements InterfaceTest {
> 
>        public static void main(String[] args) {
>            Demo demo = new Demo();
>            // 实现了抽象方法调用
>            demo.method1();
>            // 调用默认方法
>            demo.method2();
>            // 调用静态方法
>            // demo.method3();        // 和抽象类不同，不能这样调用
>            InterfaceTest.method3();  // 只能这样调用
>        }
> 
>        @Override
>        public void method1() {
>            System.out.println();
>        }
> }
> ```

## 总结

|                  | 抽象类                                                       | 接口                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **类修饰符**     | public/不加                                                  | public/不加                                                  |
| **构造方法**     | 可以有                                                       | 无                                                           |
| **普通成员变量** | 可以有，类的普通成员变量相同                                 | 不能有，即使定义了也是被默认修饰了<br />public static final  |
| **普通成员方法** | 可以有，和类的普通成员方法相同<br />即可以用 public/protected/static/final进行修饰 | JDK9 之后可以有<br />JKD8 中可以有用单用default修饰的有方法体的方法 |
| **抽象方法**     | 可以用 public/protected/不加                                 | 只能是 public 的<br />且默认是 public abstract               |
| **静态方法**     | 可以有                                                       | JDK9 之后可以有<br />JDK8 中可以有单用static修饰的有方法体的方法 |
| **静态成员**     | 有                                                           | 有，定义的变量会被默认加上<br />public static fianl          |
| **继承/实现**    | 只能继承一个抽象类<br />                                     | 可以实现多个接口<br />接口之间还可以多继承                   |

其中表中的**不加**，即表示默认的访问修饰符，只在本包中可见，在外包中不可见；如果外包中继承/实现就会无法找到相应的类/接口/抽象方法；而在本包中使用无恙。

> JDK 1.8 前，抽象类的方法默认访问权限为 protected
>
> JDK 1.8 时，抽象类的方法默认访问权限变为 default

