# Java：反射

> 对 Java 中的 **反射**，做一个微不足道的小小小小记

## 概念

Java 反射指的是在 Java 程序运行状态中，对于任何一个类，都可以获得这个类的所有属性和方法；对于给定的一个对象，都能够调用它的任意一个属性和方法。这种**动态获取类的内容以及动态调用对象的方法称为反射机制**。

**进一步**

首先，**每个类都有一个 Class 对象**，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。类加载相当于 Class 对象的加载，**类在第一次使用时才动态加载到 JVM 中**。也可以使用 `Class.forName("全限定类名")` 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，**并且这个类可以在运行时才加载进来**，甚至在编译时期该类的 .class 不存在也可以加载进来。

## 实现

**`Class类` 和 `java.lang.reflect` 一起对反射提供了支持**，`java.lang.reflect` 类库主要包含了以下三个类：

1. **Field** ：可以使用 `get()` 和 `set()` 方法读取和修改 Field 对象关联的字段；

   ```java
   // 定义类
   class Person{
       private String name;
   
       public Person() {
       }
   
       public Person(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "Person{" +
               "name='" + name + '\'' +
               '}';
       }
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

应用举例：工厂模式，使用反射机制，根据全限定类名获得某个类的 Class 实例。

## 反射的优缺点

**优点：**运行期类型的判断，`class.forName()` 动态加载类，提高代码的灵活度；

**缺点：**尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

1. **性能开销 ：**反射涉及了动态类型的解析，**所以 JVM 无法对这些代码进行优化**。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
2. 安全限制 ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
3. **内部暴露：**由于反射允许代码执行一些在正常情况下不被允许的操作（比如：访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

## 参考

https://www.cnblogs.com/nerxious/archive/2012/12/24/2829446.html

https://mp.weixin.qq.com/s/4E3xRXOVUQzccmP0yahlqA