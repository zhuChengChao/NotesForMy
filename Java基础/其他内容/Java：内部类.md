# Java：内部类

> 对 Java 中的 **内部类**，做一个微不足道的小小小小记

首先：内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。

## 成员内部类

1. 成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括 private 成员和 static 成员）

2. 如果要访问外部类同名的成员，需要以下面的形式进行访问：

   * `外部类.this.成员变量`
   * `外部类.this.成员方法`

3. 在外部类中如果要访问内部类的成员，必须先创建一个成员内部类对象，再通过指向这个对象的引用来访问

4. 要创建成员内部类的对象，前提是必须存在一个外部类的对象：

   `Outer outer = new Outer(); Outer.Inner inner= outer.new Inner();`

5. 内部类可以拥有 private、protected、public 访问权限及包访问权限；

6. 成员内部类中不能存在任何 static 的变量和方法，可以定义常量。

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

## 局部内部类

1. 在方法中定义的内部类称为局部内部类；

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
            // Static int i = 100;
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

## 静态内部类

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

## 匿名内部类

简单地说：匿名内部类就是没有名字的内部类。

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
6. 因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。

## 参考

https://www.cnblogs.com/ldl326308/p/9477566.html

