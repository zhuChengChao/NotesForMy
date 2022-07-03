# Java：创建对象

> 对 Java 中的**创建对象的内容**，做一个微不足道的小小小小记

## 创建对象的方式概述

1. 使用 new 关键字：`Person person = new Person();`

2. 反射创建：使用 Class 类的 newInstance 方法，该方法调用 **无参的构造器** 创建对象

   ```java
   // 使用 Class 类的 newInstance 方法
   String str2 = (String) Class.forName("java.lang.String").newInstance();
   String str3 = String.class.newInstance();
   
   // 使用 Constructor 类的 newInstance 方法
   Constructor<String> constructor = String.class.getConstructor();
   String str4 = constructor.newInstance();
   ```

   > [Java：反射小记]()

3. 使用 clone() 方法：

   ```java
   // 要使用 clone 方法，我们需要先实现 Cloneable 接口并实现其定义的clone方法。
   Person person1 = new Person("刘桂香");
   Person person2 = (Person) person1.clone();
   ```

4. 反序列化，比如调用 ObjectInputStream 类的 readObject() 方法。

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

## 构造函数

> 创建对象方式1：使用 new 关键字：`Person person = new Person();`

**构造函数特性：**

1. 名字与类名相同；
2. 没有返回值，但不能用 void 声明构造函数；
3. 生成类的对象时自动执行，无需调用。

**默认构造方法：**

Java 程序在执行子类的构造方法之前，如果没有用 super() 来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法/无参构造”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super() 来显示调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。

解决办法是：在父类里加上一个不做事且没有参数的构造方法，即**默认构造方法**。

**对 super 的进一步展开：** 

1. 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。

2. 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

3. `this()` 和 `super()` 不能同时出现在一个构造函数里面，**因为 this(...) 必然会调用其它的构造函数，其它的构造函数必然也会有 super(...) 语句的存在**，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。

   > 简而言之：`super()` 和 `this()` 同时出现的话，会出现初始化父类两次的不安全操作

## 克隆对象

> 创建对象方式3：使用 `clone()` 方法，来创建对象

###  实现对象的克隆

1. 将A对象的值分别通过set方法加入B对象中；

   ```java
   Student stu1 = new Student();
   stu1.setName("张三");
   Student stu2 = new Student();
   stu2.setName(stu1.getName());
   ```

2. 实现 Cloneable 接口并重写 Object 类中的 `clone()` 方法；

   * **浅克隆**：

     * 需要复制的类实现 Cloneable 接口（该接口为标记接口，接口中无任何方法）
     * 覆盖 `clone()` 方法，方法中调用 `super.clone()` 方法得到需要的复制对象。

     ```java
     import java.util.Date;
     
     public class Student implements Cloneable {
         private String name;
         private Date birthday;
     
         public String getName() {
             return name;
         }
     
         public void setName(String name) {
             this.name = name;
         }
     
         public Date getBirthday() {
             return birthday;
         }
     
         public void setBirthday(Date birthday) {
             this.birthday = birthday;
         }
     
         @Override
         protected Object clone() throws CloneNotSupportedException {
             return super.clone();
         }
     
     
         public static void main(String[] args) throws CloneNotSupportedException {
             //测试
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

   * **深克隆**

     * 通过覆盖 Object 类的 `clone()` 方法可以实现深克隆，将 `clone()` 方法的修饰符修改为 public，总而言之：深克隆把要复制的对象所引用的对象都复制了一遍。

     ```java
     // 覆盖clone方法
     @Override
     public Object clone() throws CloneNotSupportedException {
         Student student = (Student) super.clone();
         // 拷贝对象和原始对象的引用类型引用不同对象
         student.setBirthday(new Date());  
         return student;
     }
     
     //测试
     Student stu1 = new Student();
     stu1.setName("张三");
     stu1.setBirthday(new Date());
     Student stu2 = (Student) stu1.clone();
     System.out.println(stu2.getName());
     System.out.println(stu2.getBirthday());
     
     // 被拷贝对象中引用对象 
     System.out.println(stu1.getBirthday() == stu2.getBirthday());  // false
     ```

3. 实现 Serializable 接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深克隆;

   * 序列化就是将对象写到流的过程，写到流中的对象是原有对象的一个拷贝，而原对象仍然存在于内存中。
   * 通过序列化实现的拷贝不仅可以复制对象本身，而且可以复制其引用的成员对象，因此通过序列化将对象写到一个流中，再从流里将其读出来，**可以实现深克隆**。

4. 工具类 BeanUtils 和 PropertyUtils 进行对象复制

   > 有待验证

### 深克隆和浅克隆的区别

1. 浅克隆：拷贝对象和原始对象的引用类型引用同一个对象。**浅克隆只是复制了对象的引用地址，两个对象指向同一个内存地址**，所以修改其中任意的值，另一个值都会随之变化，这就是浅克隆。

   ```java
   @Override  
   public Object clone() {  
       Student stu = null;  
       try{ 
           // 浅复制  
           stu = (Student)super.clone();   
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

**克隆的补充：**

深克隆的实现就是在引用类型所在的类实现 Cloneable 接口，**并使用 public 访问修饰符重写 clone 方法**。

Java 中定义的 clone 没有深浅之分，都是统一的调用 Object 的 clone 方法。为什么会有深克隆的概念？是由于我们在实现的过程中刻意的嵌套了 clone 方法的调用。也就是说深克隆就是在需要克隆的对象类型的类中重新实现克隆方法 clone()。

## 序列化克隆对象

### 概念

**概念：**

序列化：把Java对象转换为字节序列的过程。

反序列化：把字节序列恢复为Java对象的过程。

**解释：**

对象序列化是一个用于将对象状态转换为字节流的过程，可以将其保存到磁盘文件中或通过网络发送到任何其他程序。从字节流创建对象的相反的过程称为反序列化。而创建的字节流是与平台无关的，在一个平台上序列化的对象可以在不同的平台上反序列化。序列化是为了解决在对象流进行读写操作时所引发的问题。

序列化的实现：将需要被序列化的类实现 Serializable 接口，该接口没有需要实现的方法，只是用于标注该对象是可被序列化的，然后使用一个输出流（如：FileOutputStream）来构造一个 ObjectOutputStream 对象，接着使用 ObjectOutputStream 对象的 `writeObject(Object obj)` 方法可以将参数为 obj 的对象写出，要恢复的话则使用输入流。

### 操作

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
        //反序列化对象
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("./data.obj"));
        System.out.println("obj1:" + (String)in.readObject());  //读取字面值常量
        System.out.println("obj2:" + (Date)in.readObject());  //读取匿名Date对象
        System.out.println("obj3:" + (Person)in.readObject());  //读取person对象
        in.close();
    }
}

// 结果：
// obj1:你好!
// obj2:Thu Aug 20 22:02:51 CST 2020
// obj3:Person{name='王麻子', age=40}
```

### 使用场景

**什么情况下需要序列化？**

1. 当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；

2. 当你想用套接字在网络上传送对象的时候；

   * 当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以**二进制序列**的形式在网络上传送。发送方需要把这个 Java 对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为 Java 对象。
   * 只能将支持 `java.io.Serializable` 接口的对象写入流中。每个 serializable 对象的类都被编码，编码内容包括类名和类签名、对象的字段值和数组值，以及从初始对象中引用的其他所有**对象的闭包**。

3. 当你想通过 RMI 传输对象的时候：

   > Remote Method Invocation，远程方法调用：https://www.cnblogs.com/grefr/p/5046313.html

### transient

补充一个关键字：**transient** 

对于**不想进行序列化的变量**，使用 transient 关键字修饰。

transient 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化。当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。**transient 只能修饰变量**，不能修饰类和方法。

## 参考

https://www.cnblogs.com/wxd0108/p/5685817.html

https://blog.csdn.net/ztchun/article/details/79110096

https://www.cnblogs.com/shoshana-kong/p/10538661.html

https://www.cnblogs.com/wshuage/archive/2018/11/13/9955130.html