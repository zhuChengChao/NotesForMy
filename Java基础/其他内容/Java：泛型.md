# Java：泛型

> 对 Java 中的 **泛型**，做一个微不足道的小小小小记

## 泛型实现

### 概述

**开篇：**

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();		
System.out.println(l1.getClass() == l2.getClass());  // 存在类型擦除：true
```

泛型是通过类型擦除来实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息。

例如：`List<String>` 在运行时仅用一个 List 来表示。这样做的目的，是确保能和 Java 5 之前的版本开发二进制类库进行兼容。

### 类型擦除：

泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除。

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
> 1、两个 String 其实只有第一个起作用，后面一个没什么卵用，只不过 JDK7 才开始支持 `List<String>list = new ArrayList<>` 这种写法。
>
> 2、第一个 String 就是告诉编译器，List 中存储的是 String 对象，也就是**起类型检查的作用**，之后编译器会擦除泛型占位符，以保证兼容以前的代码。

## 通配符

**通配符 `？`**：除了用 `<T>` 表示泛型外，还有 `<?>`这种形式。**`?` 被称为通配符**。

> 为何需要引入通配符 `?`的概念：
>
> ```java
> class Base{}
> class Sub extends Base{}
> 
> // 以下代码无问题：
> Sub sub = new Sub();
> Base base = sub;
> List<Sub> lsub = new ArrayList<>();
> // error! 无法编译通过
> List<Base> lbase = lsub;
> ```
>
> 编译器不会让它通过的。Sub 是 Base 的子类，不代表 `List<Sub>`和 `List<Base>`有继承关系。
>
> 但是，在现实编码中，确实有这样的需求，希望泛型能够处理某一范围内的数据类型，比如某个类和它的子类，对此 Java 引入了通配符这个概念。
>
> 因此，**通配符的出现是为了指定泛型中的类型范围**。

**通配符的三种形式：**

1. `<?>`被称作无限定的通配符。
2. `<? extends T>`被称作有上限的通配符。
3. `<? super T>`被称作有下限的通配符。

**无限定的通配符`？`：**它其中的 `?` 其实代表的是未知类型，所以涉及到 `?` 时的操作，一定与具体类型无关；

**有上限的通配符`<? extends T>`**：

确保类型必须是 T 的子类来设定类型的上界，例如 `List<? extends Number>` 可以接受 `List<Integer>` 或 `List<Float>`。

**有下限的通配符`<? super T>`**它通过确保类型必须是 T 的父类来设定类型的下界

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

class Student extends Person{
    public Student() {
    }

    public Student(String name) {
        super(name);
    }
}

class Animal{
    private String name;

    public Animal(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Animal{" +
            "name='" + name + '\'' +
            '}';
    }
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

## 参考

https://blog.csdn.net/briblue/article/details/76736356