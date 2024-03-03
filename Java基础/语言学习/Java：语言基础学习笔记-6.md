# Java：语言基础学习笔记-6

> 时间紧任务重，此文作为后端学习基础笔记的第六篇

## 24. Stream流&方法引用

### 24.1 Stream流

说到Stream便容易想到I/O Stream，而实际上，谁规定“流”就一定是“IO流”呢？在Java 8中，得益于Lambda所带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库既有的弊端。

#### 24.1.1 引言

**传统集合的多步遍历代码**

几乎所有的集合（如 Collection 接口或 Map 接口等）都支持直接或间接的遍历操作。而当我们需要对集合中的元素进行操作的时候，除了必需的添加、删除、获取外，最典型的就是集合遍历。例如：

```java
import java.util.ArrayList;
import java.util.List;
public class Demo01ForEach {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
        for (String name : list) {
            System.out.println(name);
        }
    }
}
```

这是一段非常简单的集合遍历操作：对集合中的每一个字符串都进行打印输出操作。

**循环遍历的弊端**

Java 8的Lambda让我们可以更加专注于做什么（What），而不是怎么做（How），这点此前已经结合内部类进行了对比说明。现在，我们仔细体会一下上例代码，可以发现：

* for循环的语法就是“怎么做“
* for循环的循环体才是“做什么”

为什么使用循环？因为要进行遍历。但循环是遍历的唯一方式吗？遍历是指每一个元素逐一进行处理，而并不是从第一个到最后一个顺次处理的循环。前者是目的，后者是方式。

试想一下，如果希望对集合中的元素进行筛选过滤：

1. 将集合A根据条件一过滤为子集B；
2. 然后再根据条件二过滤为子集C。

那怎么办？在Java 8之前的做法可能为：

```java
import java.util.ArrayList;
import java.util.List;
public class Demo02NormalFilter {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
        List<String> zhangList = new ArrayList<>();
        for (String name : list) {
            if (name.startsWith("张")) {
                zhangList.add(name);
            }
        } 
        List<String> shortList = new ArrayList<>();
        for (String name : zhangList) {
            if (name.length() == 3) {
                shortList.add(name);
            }
        } 
        for (String name : shortList) {
            System.out.println(name);
        }
    }
}
```

这段代码中含有三个循环，每一个作用不同：

1. 首先筛选所有姓张的人；
2. 然后筛选名字有三个字的人；
3. 最后进行对结果进行打印输出。

每当我们需要对集合中的元素进行操作的时候，总是需要进行循环、循环、再循环。这是理所当然的么？不是。循环是做事情的方式，而不是目的。另一方面，使用线性循环就意味着只能遍历一次。如果希望再次遍历，只能再使 用另一个循环从头开始。 那Lambda的衍生物Stream能给我们带来怎样更加优雅的写法呢？

**Stream的更优写法**

下面来看一下借助Java 8的Stream API，什么才叫优雅： 

```java
import java.util.ArrayList;
import java.util.List;
public class Demo03StreamFilter {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
        list.stream()
            .filter(s -> s.startsWith("张"))
            .filter(s -> s.length() == 3)
            .forEach(System.out::println);
    }
}
```

直接阅读代码的字面意思即可完美展示无关逻辑方式的语义：获取流、过滤姓张、过滤长度为3、逐一打印。代码中并没有体现使用线性循环或是其他任何算法进行遍历，我们真正要做的事情内容被更好地体现在代码中。

#### 24.1.2 流式思想概述

> **注意：请暂时忘记对传统IO流的固有印象！**

整体来看，流式思想类似于工厂车间的“**生产流水线**”。

当需要对多个元素进行操作（特别是多步操作）的时候，考虑到性能及便利性，我们应该首先拼好一个“模型”步骤方案，然后再按照方案去执行它。

![流式思想](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516233.PNG)

这张图中展示了过滤、映射、跳过、计数等多步操作，这是一种集合元素的处理方案，而方案就是一种“函数模型”。图中的每一个方框都是一个“流”，调用指定的方法，可以从一个流模型转换为另一个流模型。而最右侧的数字3是最终结果。

这里的 filter 、 map 、 skip 都是在对函数模型进行操作，集合元素并没有真正被处理。只有当终结方法 count 执行的时候，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。

> 备注：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何元素（或其地址值）。

Stream（流）是一个来自数据源的元素队列

* 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
* **数据源**流的来源。 可以是集合，数组等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

* **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路(shortcircuiting)。
* **内部迭代**： 以前对集合遍历都是通过Iterator或者增强for的方式，显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

当使用一个流的时候，通常包括三个基本步骤：

1. 获取一个数据源（source）
2. 数据转换
3. 执行操作获取想要的结果，

每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道。

#### 24.1.3 获取流方式

`java.util.stream.Stream<T>` 是Java 8新加入的最常用的流接口。（这并不是一个函数式接口。）

获取一个流非常简单，有以下几种常用的方式：

* 所有的 Collection 集合都可以通过 stream 默认方法获取流；
* Stream 接口的静态方法 of 可以获取数组对应的流。

**根据Collection获取流**

首先， `java.util.Collection` 接口中加入了default方法 stream 用来获取流，所以其所有实现类均可获取流。

```java
import java.util.*;
import java.util.stream.Stream;
public class Demo04GetStream {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        // ...
        Stream<String> stream1 = list.stream();
        
        Set<String> set = new HashSet<>();
        // ...
        Stream<String> stream2 = set.stream();
        
        Vector<String> vector = new Vector<>();
        // ...
        Stream<String> stream3 = vector.stream();
    }
}
```

**根据Map获取流**

`java.util.Map` 接口不是 Collection 的子接口，且其 K-V 数据结构不符合流元素的单一特征，所以获取对应的流 需要分key、value或entry等情况：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Stream;
public class Demo05GetStream {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        // ...
        Stream<String> keyStream = map.keySet().stream();
        Stream<String> valueStream = map.values().stream();
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream();
    }
}
```

**根据数组获取流**

如果使用的不是集合或映射而是数组，由于数组对象不可能添加默认方法，所以 Stream 接口中提供了静态方法 of ，使用很简单:

```java
import java.util.stream.Stream;
public class Demo06GetStream {
    public static void main(String[] args) {
        String[] array = { "张无忌", "张翠山", "张三丰", "张一元" };
        Stream<String> stream = Stream.of(array);
    }
}
```

> 备注： of 方法的参数其实是一个可变参数，所以支持数组。

#### 24.1.4 常用方法

![常用方法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516040.PNG)

流模型的操作很丰富，这里介绍一些常用的API。这些方法可以被分成两种：

* **延迟方法**：返回值类型仍然是 Stream 接口自身类型的方法，因此支持链式调用。（除了终结方法外，其余方法均为延迟方法。）
* **终结方法**：返回值类型不再是 Stream 接口自身类型的方法，因此不再支持类似 StringBuilder 那样的链式调用。

本小节中，终结方法包括 count 和 forEach 方法。

> 备注：本小节之外的更多方法，请自行参考API文档。

**逐一处理：forEach**

* 虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。

  ```java
  void forEach(Consumer<? super T> action);
  ```

  该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理。

* **复习Consumer接口**

  ```java
  java.util.function.Consumer<T>接口是一个消费型接口。
  Consumer接口中包含抽象方法void accept(T t)，意为消费一个指定泛型的数据。
  ```

* **基本使用：**

```java
import java.util.stream.Stream;
public class Demo12StreamForEach {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("张无忌", "张三丰", "周芷若");
        stream.forEach(name-> System.out.println(name));
    }
}
```

**过滤：filter**

* 可以通过 filter 方法将一个流转换成另一个子集流。方法签名：

  ```java
  Stream<T> filter(Predicate<? super T> predicate);
  ```

  该接口接收一个 Predicate 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件。

  ![filter](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516393.PNG)

* **复习Predicate接口**

  此前我们已经学习过 `java.util.stream.Predicate` 函数式接口，其中唯一的抽象方法为：

  ```java
  boolean test(T t);
  ```

  该方法将会产生一个boolean值结果，代表指定的条件是否满足。如果结果为true，那么Stream流的 filter 方法将会留用元素；如果结果为false，那么 filter 方法将会舍弃元素。

* **基本使用**

  Stream流中的 filter 方法基本使用的代码如下，在这里通过Lambda表达式来指定了筛选的条件：必须姓张。

```java
import java.util.stream.Stream;
public class Demo07StreamFilter {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若");
        Stream<String> result = original.filter(s -> s.startsWith("张"));
    }
}
```

**映射：map**

* 如果需要将流中的元素映射到另一个流中，可以使用 map 方法。方法签名:

  ```java
  <R> Stream<R> map(Function<? super T, ? extends R> mapper)
  ```

  该接口需要一个 Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

  ![map-stream](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516349.PNG)

* **复习Function接口**

  此前我们已经学习过 `java.util.stream.Function` 函数式接口，其中唯一的抽象方法为：

  ```java
  R apply(T t);
  ```

  这可以将一种T类型转换成为R类型，而这种转换的动作，就称为“映射”。

* **基本使用**

  Stream流中的 map 方法基本使用的代码如下，这段代码中， map 方法的参数通过方法引用，将字符串类型转换成为了int类型（并自动装箱为 Integer 类对象）。

```java
import java.util.stream.Stream;
public class Demo08StreamMap {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("10", "12", "18");
        Stream<Integer> result = original.map(str->Integer.parseInt(str));
    }
}
```

**统计个数：count**

* 正如旧集合 Collection 当中的 size 方法一样，流提供 count 方法来数一数其中的元素个数：

  ```java
  long count();
  ```

  该方法返回一个long值代表元素个数（不再像旧集合那样是int值）。

* 基本使用：

```java
import java.util.stream.Stream;
public class Demo09StreamCount {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若");
        Stream<String> result = original.filter(s -> s.startsWith("张"));
        System.out.println(result.count()); // 2
    }
}
```

**取用前几个：limit**

* limit 方法可以对流进行截取，只取用前n个。方法签名：

  ```java
  Stream<T> limit(long maxSize);
  ```

  参数是一个long型，如果集合当前长度大于参数则进行截取；否则不进行操作。基本使用：

  ![limit-stream](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516659.PNG)

**跳过前几个：skip**

* 如果希望跳过前几个元素，可以使用 skip 方法获取一个截取之后的新流：

  ```java
  Stream<T> skip(long n);
  ```

  如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

  ![skip-stream](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516601.PNG)

* 基本使用：

```java
import java.util.stream.Stream;
public class Demo11StreamSkip {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若");
        Stream<String> result = original.skip(2);
        System.out.println(result.count()); // 1
    }
}
```

**组合：concat**

* 如果有两个流，希望合并成为一个流，那么可以使用 Stream 接口的静态方法 concat ：

  ```java
  static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
  ```

  > 备注：这是一个静态方法，与 `java.lang.String` 当中的 concat 方法是不同的。

* 该方法的基本使用代码如：

```java
import java.util.stream.Stream;
public class Demo12StreamConcat {
    public static void main(String[] args) {
        Stream<String> streamA = Stream.of("张无忌");
        Stream<String> streamB = Stream.of("张翠山");
        Stream<String> result = Stream.concat(streamA, streamB);
    }
}
```

#### 24.1.5 练习：集合元素处理(传统方式)

**题目**

现在有两个 ArrayList 集合存储队伍当中的多个成员姓名，要求使用传统的for循环（或增强for循环）依次进行以下若干操作步骤：

1. 第一个队伍只要名字为3个字的成员姓名；存储到一个新集合中。
2. 第一个队伍筛选之后只要前3个人；存储到一个新集合中。
3. 第二个队伍只要姓张的成员姓名；存储到一个新集合中。
4. 第二个队伍筛选之后不要前2个人；存储到一个新集合中。
5. 将两个队伍合并为一个队伍；存储到一个新集合中。
6. 根据姓名创建Person对象；存储到一个新集合中。
7. 打印整个队伍的Person对象信息。

两个队伍（集合）的代码如下：

```java
import java.util.ArrayList;
import java.util.List;
public class DemoArrayListNames {
    public static void main(String[] args) {
        //第一支队伍
        ArrayList<String> one = new ArrayList<>();
        one.add("迪丽热巴");
        one.add("宋远桥");
        one.add("苏星河");
        one.add("石破天");
        one.add("石中玉");
        one.add("老子");
        one.add("庄子");
        one.add("洪七公");
        //第二支队伍
        ArrayList<String> two = new ArrayList<>();
        two.add("古力娜扎");
        two.add("张无忌");
        two.add("赵丽颖");
        two.add("张三丰");
        two.add("尼古拉斯赵四");
        two.add("张天爱");
        two.add("张二狗");
        // ....
    }
}
```

而 Person 类的代码为：

```java
public class Person {
    private String name;
    public Person() {}
    public Person(String name) {
        this.name = name;
    } 
    @Override
    public String toString() {
        return "Person{name='" + name + "'}";
    } 
    public String getName() {
        return name;
    } 
    public void setName(String name) {
        this.name = name;
    }
}
```

**解答**

既然使用传统的for循环写法，那么：

```java
public class DemoArrayListNames {
    public static void main(String[] args) {
        List<String> one = new ArrayList<>();
        // ...
        List<String> two = new ArrayList<>();
        // ...
        // 第一个队伍只要名字为3个字的成员姓名；
        List<String> oneA = new ArrayList<>();
        for (String name : one) {
            if (name.length() == 3) {
                oneA.add(name);
            }
        } 
        // 第一个队伍筛选之后只要前3个人；
        List<String> oneB = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            oneB.add(oneA.get(i));
        } 
        // 第二个队伍只要姓张的成员姓名；
        List<String> twoA = new ArrayList<>();
        for (String name : two) {
            if (name.startsWith("张")) {
                twoA.add(name);
            }
        } 
        // 第二个队伍筛选之后不要前2个人；
        List<String> twoB = new ArrayList<>();
        for (int i = 2; i < twoA.size(); i++) {
            twoB.add(twoA.get(i));
        } 
        // 将两个队伍合并为一个队伍；
        List<String> totalNames = new ArrayList<>();
        totalNames.addAll(oneB);
        totalNames.addAll(twoB);
        // 根据姓名创建Person对象；
        List<Person> totalPersonList = new ArrayList<>();
        for (String name : totalNames) {
            totalPersonList.add(new Person(name));
        } 
        // 打印整个队伍的Person对象信息。
        for (Person person : totalPersonList) {
            System.out.println(person);
        }
    }
}
```

运行结果为：

```java
Person{name='宋远桥'}
Person{name='苏星河'}
Person{name='石破天'}
Person{name='张天爱'}
Person{name='张二狗'}
```

#### 24.1.6 练习：集合元素处理(Stream方式)

**题目**

将上一题当中的传统for循环写法更换为Stream流式处理方式。两个集合的初始内容不变， Person 类的定义也不变。

**解答**

等效的Stream流式处理代码为：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Stream;
public class DemoStreamNames {
    public static void main(String[] args) {
        List<String> one = new ArrayList<>();
        // ...
        List<String> two = new ArrayList<>();
        // ...
        // 第一个队伍只要名字为3个字的成员姓名；
        // 第一个队伍筛选之后只要前3个人；
        Stream<String> streamOne = one.stream().
            filter(s -> s.length() == 3).limit(3);
        // 第二个队伍只要姓张的成员姓名；
        // 第二个队伍筛选之后不要前2个人；
        Stream<String> streamTwo = two.stream().
            filter(s -> s.startsWith("张")).skip(2);
        // 将两个队伍合并为一个队伍；
        // 根据姓名创建Person对象；
        // 打印整个队伍的Person对象信息。
        Stream.concat(streamOne,streamTwo).
            map(Person::new).forEach(System.out::println);
    }
}
```

运行效果完全一样：

```java
Person{name='宋远桥'}
Person{name='苏星河'}
Person{name='石破天'}
Person{name='张天爱'}
Person{name='张二狗'}
```

### 24.2 方法引用

在使用Lambda表达式的时候，我们实际上传递进去的代码就是一种解决方案：拿什么参数做什么操作。那么考虑一种情况：如果我们在Lambda中所指定的操作方案，已经有地方存在相同方案，那是否还有必要再写重复逻辑？

#### 24.2.1 冗余的Lambda场景

来看一个简单的函数式接口以应用Lambda表达式：

```java
@FunctionalInterface
public interface Printable {
    void print(String str);
}
```

在 Printable 接口当中唯一的抽象方法 print 接收一个字符串参数，目的就是为了打印显示它。那么通过Lambda来使用它的代码很简单：

```java
public class Demo01PrintSimple {
    private static void printString(Printable data) {
        data.print("Hello, World!");
    } 
    public static void main(String[] args) {
        printString(s -> System.out.println(s));
    }
}
```

其中 printString 方法只管调用 Printable 接口的 print 方法，而并不管 print 方法的具体实现逻辑会将字符串打印到什么地方去。而 main 方法通过Lambda表达式指定了函数式接口 Printable 的具体操作方案为：拿到String(类型可推导，所以可省略)数据后，在控制台中输出它。

#### 24.2.2 问题分析

这段代码的问题在于，对字符串进行控制台打印输出的操作方案，明明已经有了现成的实现，那就是**System.out对象中的`println(String)`方法**。既然Lambda希望做的事情就是调用`println(String)` 方法，那何必自己手动调用呢？

#### 24.2.3 用方法引用改进代码

能否省去Lambda的语法格式（尽管它已经相当简洁）呢？只要“引用”过去就好了：

```java
public class Demo02PrintRef {
    private static void printString(Printable data) {
        data.print("Hello, World!");
    } 
    public static void main(String[] args) {
        printString(System.out::println);
    }
}
```

请注意其中的双冒号 `::` 写法，这被称为“**方法引用**”，而双冒号是一种新的语法。

#### 24.2.4 方法引用符

双冒号 `::` 为引用运算符，而它所在的表达式被称为**方法引用**。如果Lambda要表达的函数方案已经存在于某个方法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。

**语义分析**

例如上例中， System.out 对象中有一个重载的 `println(String)` 方法恰好就是我们所需要的。那么对于 printString 方法的函数式接口参数，对比下面两种写法，完全等效：

* Lambda表达式写法： `s -> System.out.println(s);`
* 方法引用写法： `System.out::println`

第一种语义是指：拿到参数之后经Lambda之手，继而传递给 System.out.println 方法去处理。

第二种等效写法的语义是指：直接让 System.out 中的 println 方法来取代Lambda。

两种写法的执行效果完全一样，而第二种方法引用的写法复用了已有方案，更加简洁。

> 注：Lambda中传递的参数一定是方法引用中的那个方法可以接收的类型，否则会抛出异常

**推导与省略**

如果使用Lambda，那么根据“**可推导就是可省略**”的原则，无需指定参数类型，也无需指定的重载形式——它们都将被自动推导。而如果使用方法引用，也是同样可以根据上下文进行推导。

函数式接口是Lambda的基础，而方法引用是Lambda的孪生兄弟。

下面这段代码将会调用 println 方法的不同重载形式，将函数式接口改为int类型的参数：

```java
@FunctionalInterface
public interface PrintableInteger {
    void print(int str);
}
```

由于上下文变了之后**可以自动推导出唯一对应的匹配重载**，所以方法引用没有任何变化：

```java
public class Demo03PrintOverload {
    private static void printInteger(PrintableInteger data) {
        data.print(1024);
    } 
    public static void main(String[] args) {
        printInteger(System.out::println);
    }
}
```

这次方法引用将会自动匹配到 `println(int)` 的重载形式。

#### 24.2.5 通过对象名引用成员方法

这是最常见的一种用法，与上例相同。如果一个类中已经存在了一个成员方法：

```java
public class MethodRefObject {
    public void printUpperCase(String str) {
        System.out.println(str.toUpperCase());
    }
}
```

函数式接口仍然定义为：

```java
@FunctionalInterface
public interface Printable {
    void print(String str);
}
```

那么当需要使用这个 printUpperCase 成员方法来替代 Printable 接口的Lambda的时候，**已经具有了`MethodRefObject` 类的对象实例，则可以通过对象名引用成员方法**，代码为：

```java
public class Demo04MethodRef {
    private static void printString(Printable lambda) {
        lambda.print("Hello");
    } 
    public static void main(String[] args) {
        MethodRefObject obj = new MethodRefObject();
        printString(obj::printUpperCase);
    }
}
```

#### 24.2.6 通过类名称引用静态方法

由于在 `java.lang.Math` 类中已经存在了静态方法 abs ，所以当我们需要通过Lambda来调用该方法时，有两种写法。首先是函数式接口：

```java
@FunctionalInterface
public interface Calcable {
	int calc(int num);
}
```

第一种写法是使用Lambda表达式：

```java
public class Demo05Lambda {
    private static void method(int num, Calcable lambda) {
        System.out.println(lambda.calc(num));
    } 
    public static void main(String[] args) {
        method(-10, n -> Math.abs(n));
    }
}
```

但是使用方法引用的更好写法是：

```java
public class Demo06MethodRef {
    private static void method(int num, Calcable lambda) {
        System.out.println(lambda.calc(num));
    } 
    public static void main(String[] args) {
        method(-10, Math::abs);
    }
}
```

在这个例子中，下面两种写法是等效的：

* Lambda表达式： `n -> Math.abs(n)`
* 方法引用： `Math::abs`

#### 24.2.7 通过super引用成员方法

如果存在继承关系，当Lambda中需要出现super调用时，也可以使用方法引用进行替代。首先是函数式接口：

```java
@FunctionalInterface
public interface Greetable {
    void greet();
}
```

然后是父类 Human 的内容：

```java
public class Human {
    public void sayHello() {
        System.out.println("Hello!");
    }
}
```

最后是子类 Main 的内容，其中使用了Lambda的写法：

```java
public class Man extends Human {
    @Override
    public void sayHello() {
        System.out.println("大家好,我是Man!");
    } 
    // 定义方法method,参数传递Greetable接口
    public void method(Greetable g){
        g.greet();
    } 
    public void show(){
        //调用method方法,使用Lambda表达式
        method(()->{
            //创建Human对象,调用sayHello方法
            new Human().sayHello();
        });
        //简化Lambda
        method(()->new Human().sayHello());
        //使用super关键字代替父类对象
        method(()->super.sayHello());
    }
}
```

但是如果使用方法引用来调用父类中的 sayHello 方法会更好，例如另一个子类 Woman ：

```java
public class Woman extends Human {
    @Override
    public void sayHello() {
        System.out.println("大家好,我是Woman!");
    } 
    //定义方法method,参数传递Greetable接口
    public void method(Greetable g){
        g.greet();
    } 
    public void show(){
        method(super::sayHello);
    }
}
```

在这个例子中，下面两种写法是等效的：

* Lambda表达式： `() -> super.sayHello()`
* 方法引用： `super::sayHello`

#### 24.2.8 通过this引用成员方法

this代表当前对象，如果需要引用的方法就是当前类中的成员方法，那么可以使用“**this::成员方法**”的格式来使用方法引用。首先是简单的函数式接口：

```java
@FunctionalInterface
public interface Richable {
    void buy();
}
```

下面是一个丈夫 Husband 类：

```java
public class Husband {
    private void marry(Richable lambda) {
        lambda.buy();
    } 
    public void beHappy() {
        marry(() -> System.out.println("买套房子"));
    }
}
```

开心方法 beHappy 调用了结婚方法 marry ，后者的参数为函数式接口 Richable ，所以需要一个Lambda表达式。但是如果这个Lambda表达式的内容已经在本类当中存在了，则可以对 Husband 丈夫类进行修改：

```java
public class Husband {
    private void buyHouse() {
        System.out.println("买套房子");
    } 
    private void marry(Richable lambda) {
        lambda.buy();
    } 
    public void beHappy() {
        marry(() -> this.buyHouse());
    }
}
```

如果希望取消掉Lambda表达式，用方法引用进行替换，则更好的写法为：

```java
public class Husband {
    private void buyHouse() {
        System.out.println("买套房子");
    } 
    private void marry(Richable lambda) {
        lambda.buy();
    } 
    public void beHappy() {
        marry(this::buyHouse);
    }
}
```

在这个例子中，下面两种写法是等效的：

* Lambda表达式： `() -> this.buyHouse()`
* 方法引用： `this::buyHouse`

#### 24.2.9 类的构造器引用

由于构造器的名称与类名完全一样，并不固定。所以构造器引用使用 **类名称::new** 的格式表示。首先是一个简单的 Person 类：

```java
public class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    } 
    public String getName() {
        return name;
    } 
    public void setName(String name) {
        this.name = name;
    }
}
```

然后是用来创建 Person 对象的函数式接口：

```java
public interface PersonBuilder {
    Person buildPerson(String name);
}
```

要使用这个函数式接口，可以通过Lambda表达式：

```java
public class Demo09Lambda {
    public static void printName(String name, PersonBuilder builder) {
        System.out.println(builder.buildPerson(name).getName());
    } 
    public static void main(String[] args) {
        printName("赵丽颖", name -> new Person(name));
    }
}
```

但是通过构造器引用，有更好的写法：

```java
public class Demo10ConstructorRef {
    public static void printName(String name, PersonBuilder builder) {
        System.out.println(builder.buildPerson(name).getName());
    } 
    public static void main(String[] args) {
        printName("赵丽颖", Person::new);
    }
}
```

在这个例子中，下面两种写法是等效的：

* Lambda表达式： `name -> new Person(name)`
* 方法引用： `Person::new`

#### 24.2.10 数组的构造器引用

数组也是 Object 的子类对象，所以同样具有构造器，只是语法稍有不同。如果对应到Lambda的使用场景中时，需要一个函数式接口：

```java
@FunctionalInterface
public interface ArrayBuilder {
    int[] buildArray(int length);
}
```

在应用该接口的时候，可以通过Lambda表达式：

```java
public class Demo11ArrayInitRef {
    private static int[] initArray(int length, ArrayBuilder builder) {
        return builder.buildArray(length);
    } 
    public static void main(String[] args) {
        int[] array = initArray(10, length -> new int[length]);
    }
}
```

但是更好的写法是使用数组的构造器引用：

```java
public class Demo12ArrayInitRef {
    private static int[] initArray(int length, ArrayBuilder builder) {
        return builder.buildArray(length);
    } 
    public static void main(String[] args) {
        int[] array = initArray(10, int[]::new);
    }
}
```

在这个例子中，下面两种写法是等效的：

* Lambda表达式： `length -> new int[length]`
* 方法引用： `int[]::new`

## 25. 基础加强

### 25.1 Junit单元测试

#### 25.1.1 测试分类

1. 黑盒测试：不需要写代码，给输入值，看程序是否能够输出期望的值。
2. 白盒测试：需要写代码的。关注程序具体的执行流程。

#### 25.1.2 Junit使用：白盒测试

**步骤**：

1. 定义一个测试类(测试用例)

   建议测试类名：被测试的类名+Test，如`CalculatorTest`

   包名：`xxx.xxx.xx.test`，如`cn.itcast.test`

2. 定义测试方法：可以独立运行

   方法名：test测试的方法名，如`testAdd()` 

   返回值：void

   参数列表：空参

3. 给方法加`@Test`
4. 导入junit依赖环境

**判定结果**：

* 红色：失败

* 绿色：成功

一般我们会使用断言操作来处理结果

* `Assert.assertEquals(期望的结果,运算的结果);`

**补充：**

`@Before`: 修饰的方法会在测试方法之前被自动执行

`@After`: 修饰的方法会在测试方法执行之后自动被执行


### 25.2 反射：框架设计的灵魂

**框架**：半成品软件。可以在框架的基础上进行软件开发，简化编码

**反射**：将类的各个组成部分封装为其他对象，这就是反射机制

#### 25.2.1 反射好处

1. 可以在程序运行过程中，操作这些对象。
2. 可以解耦，提高程序的可扩展性。

#### 25.2.2 获取Class对象的方式

1. `Class.forName("全类名")`：将字节码文件加载进内存，返回Class对象

   多用于配置文件，将类名定义在配置文件中。读取文件，加载类
2. `类名.class`：通过类名的属性获取class

   多用于参数的传递
3. `对象.getClass()`：`getClass()`方法在Object类中定义着。

   多用于对象的获取字节码的方式

**结论：**同一个字节码文件(*.class)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。

#### 25.2.3 Class对象功能

**获取功能：**

* **获取成员变量们**
  * `Field[] getFields()` ：获取所有public修饰的成员变量
  * `Field getField(String name)`：获取指定名称的public修饰的成员变量

  * `Field[] getDeclaredFields()`：获取所有的成员变量，不考虑修饰符
  * `Field getDeclaredField(String name) ` ：获取指定名称的成员变量，不考虑修饰符

* **获取构造方法们**
  * `Constructor<?>[] getConstructors()` 
  * `Constructor<T> getConstructor(类<?>... parameterTypes) ` 
  * `Constructor<T> getDeclaredConstructor(类<?>... parameterTypes)`
  * `Constructor<?>[] getDeclaredConstructors()`

* **获取成员方法们：**
  * `Method[] getMethods()  `
  * `Method getMethod(String name, 类<?>... parameterTypes)  `
  * `Method[] getDeclaredMethods()  `
  * `Method getDeclaredMethod(String name, 类<?>... parameterTypes)  `

* **获取全类名**	
  * `String getName()  `

**Field：成员变量**

* 设置值：`void set(Object obj, Object value)  `

* 获取值：`get(Object obj) `

* 忽略访问权限修饰符的安全检查(暴力反射)：`setAccessible(true)`

**Constructor:构造方法**

* **创建对象：**
  * `T newInstance(Object... initargs)  `
  * 如果使用空参数构造方法创建对象，操作可以简化：`Class.newInstance()`方法

**Method：方法对象**

* **执行方法：**`Object invoke(Object obj, Object... args)  `

* **获取方法名称：**`String getName()`:获取方法名

#### 25.2.4 反射案例

**需求**：写一个"框架"，不能改变该类的任何代码的前提下，可以帮我们创建任意类的对象，并且执行其中任意方法

**实现：**

1. 配置文件
2. 反射

**步骤：**

1. 将需要创建的对象的全类名和需要执行的方法定义在配置文件中
2. 在程序中加载读取配置文件
3. 使用反射技术来加载类文件进内存
4. 创建对象
5. 执行方法

**代码**

1. 在src同级的目录下建立配置文件`.properties`，文件内容：

   ```
   className=cn.itcast.domain.Student
   methodName=sleep
   ```
   
2. 建立ReflectTest类，进行测试：

   ```java
   package cn.itcast.reflect;
   
   import cn.itcast.domain.Person;
   import cn.itcast.domain.Student;
   
   import java.io.IOException;
   import java.io.InputStream;
   import java.lang.reflect.Method;
   import java.util.Properties;
   /**
     * 框架类
     */
   public class ReflectTest {
       public static void main(String[] args) throws Exception {
           //可以创建任意类的对象，可以执行任意方法
           /* 前提：不能改变该类的任何代码。可以创建任意类的对象，可以执行任意方法 */
           //1.加载配置文件
           //1.1创建Properties对象
           Properties pro = new Properties();
           //1.2加载配置文件，转换为一个集合
           //1.2.1获取class目录下的配置文件
           ClassLoader classLoader = ReflectTest.class.getClassLoader();
           InputStream is = classLoader.getResourceAsStream("pro.properties");
           pro.load(is);
           //2.获取配置文件中定义的数据
           String className = pro.getProperty("className");
           String methodName = pro.getProperty("methodName");
           //3.加载该类进内存
           Class cls = Class.forName(className);
           //4.创建对象
           Object obj = cls.newInstance();
           //5.获取方法对象
           Method method = cls.getMethod(methodName);
           //6.执行方法
           method.invoke(obj);
       }
   }
   ```

### 25.3 注解

**注解**：说明程序的。给计算机看的

**注释**：用文字描述程序的。给程序员看的

#### 25.3.1 定义

注解（Annotation），也叫**元数据**。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

**概念描述：**

* JDK1.5之后的新特性
* 说明程序的
* 使用注解：@注解名称

**作用分类：**

1. 编写文档：通过代码里标识的注解生成文档【生成文档doc文档】
2. 代码分析：通过代码里标识的注解对代码进行分析【使用反射】
3. 编译检查：通过代码里标识的注解让编译器能够实现基本的编译检查【Override】

#### 25.3.2 JDK中预定义的一些注解

`@Override` ：检测被该注解标注的方法是否是继承自父类(接口)的

`@Deprecated` ：该注解标注的内容，表示已过时

`@SuppressWarnings`：压制警告，一般传递参数 all，即 `@SuppressWarnings("all")`

#### 25.3.4 自定义注解

**格式：**

```java
// 元注解
public @interface 注解名称{
	属性列表;
}
```

**本质**：注解本质上就是一个接口，该接口默认继承Annotation接口

`public interface MyAnno extends java.lang.annotation.Annotation {}`

**属性**：接口中的抽象方法

**要求**：属性的返回值类型有下列取值

* 基本数据类型
* String
* 枚举
* 注解
* 以上类型的数组

定义了属性，在使用时需要给属性赋值

1. 如果定义属性时，使用default关键字给属性默认初始化值，则使用注解时，可以不进行属性的赋值。
2. 如果只有一个属性需要赋值，并且属性的名称是value，则value可以省略，直接定义值即可。
3. 数组赋值时，值使用`{}`包裹。如果数组中只有一个值，则`{}`可以省略

#### 25.3.5 元注解

**元注解：用于描述注解的注解**

`@Target`：描述注解能够作用的位置

* ElementType取值：
  * `TYPE`：可以作用于类上
  * `METHOD`：可以作用于方法上
  * `FIELD`：可以作用于成员变量上

`@Retention`：描述注解被保留的阶段

* `@Retention(RetentionPolicy.RUNTIME)`：当前被描述的注解，会保留到class字节码文件中，并被JVM读取到

* `@Documented`：描述注解是否被抽取到api文档中
* `@Inherited`：描述注解是否被子类继承

#### 25.3.6 在程序使用(解析)注解

获取注解中定义的属性值

1. 获取注解定义的位置的对象（Class，Method，Field）

2. 获取指定的注解

   ```java
   getAnnotation(Class)
       
   //其实就是在内存中生成了一个该注解接口的子类实现对象
   public class ProImpl implements Pro{
       public String className(){
           return "cn.itcast.annotation.Demo1";
       }
       public String methodName(){
           return "show";
       }
   }
   ```

3. 调用注解中的抽象方法获取配置的属性值

#### 25.3.7 注解小结

以后大多数时候，我们会使用注解，而不是自定义注解

注解给谁用？
1. 编译器
2. 给解析程序用

注解不是程序的一部分，可以理解为注解就是一个标签

### 25.4 JDK8新特性

#### 25.4.1. 前言

JDK8 已经发布很久了，在很多企业中都已经在使用。并且Spring5、SpringBoot2.0都推荐使用JDK1.8以上版本。所以我们必须与时俱进，拥抱变化。

Jdk8这个版本包含语言、编译器、库、工具和JVM等方面的十多个新特性。在本文中我们将学习以下方面的新特性：

- [Lambda表达式](#2. Lambda表达式)
- [函数式接口](#3. 函数式接口)
- [方法引用](#4. 方法引用)
- [接口的默认方法和静态方法](#5. 接口的默认方法和静态方法)
- [Optional](#6. Optional)
- [Streams](#7. Streams)
- [并行数组](#8. 并行数组)

#### 25.4.2 Lambda表达式

函数式编程

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。可以使代码变的更加简洁紧凑。

**基本语法：**`(参数列表) -> {代码块}`

需要注意：

- 参数类型可省略，编译器可以自己推断
- 如果只有一个参数，圆括号可以省略
- 代码块如果只是一行代码，大括号也可以省略
- 如果代码块是一行，且是有结果的表达式，`return`可以省略

> **注意：**事实上，把Lambda表达式可以看做是匿名内部类的一种简写方式。当然，前提是这个匿名内部类对应的必须是接口，而且接口中必须只有一个函数！Lambda表达式就是直接编写函数的：参数列表、代码体、返回值等信息，**`用函数来代替完整的匿名内部类`**！

**用法示例**

* 示例1：多个参数

准备一个集合：

```java
// 准备一个集合
List<Integer> list = Arrays.asList(10, 5, 25, -15, 20);
```

假设我们要对集合排序，我们先看JDK7的写法，需要通过匿名内部类来构造一个`Comparator`：

```java
// Jdk1.7写法
Collections.sort(list,new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 - o2;
    }
});
System.out.println(list);// [-15, 5, 10, 20, 25]
```

如果是jdk8，我们可以使用新增的集合API：`sort(Comparator c)`方法，接收一个比较器，我们用Lambda来代替`Comparator` 的匿名内部类：

```java
// Jdk1.8写法，参数列表的数据类型可省略：
list.sort((i1,i2) -> { return i1 - i2;});

System.out.println(list);// [-15, 5, 10, 20, 25]
```

对比一下`Comparator`中的`compare()`方法，你会发现：这里编写的Lambda表达式，恰恰就是`compare()`方法的简写形式，JDK8会把它编译为匿名内部类。是不是简单多了！

别着急，我们发现这里的代码块只有一行代码，符合前面的省略规则，我们可以简写为：

```java
// Jdk8写法
// 因为代码块是一个有返回值的表达式，可以省略大括号以及return
list.sort((i1,i2) -> i1 - i2);
```

* **示例2：单个参数**

还以刚才的集合为例，现在我们想要遍历集合中的元素，并且打印。

先用jdk1.7的方式：

```java
// JDK1.7遍历并打印集合
for (Integer i : list) {
    System.out.println(i);
}
```

jdk1.8给集合添加了一个方法：`foreach()` ，接收一个对元素进行操作的函数：

```java
// JDK1.8遍历并打印集合，因为只有一个参数，所以我们可以省略小括号:
list.forEach(i -> System.out.println(i));
```

* **实例3：把Lambda赋值给变量**

Lambda表达式的实质其实还是匿名内部类，所以我们其实可以把Lambda表达式赋值给某个变量。

```java
// 将一个Lambda表达式赋值给某个接口：
Runnable task = () -> {
    // 这里其实是Runnable接口的匿名内部类，我们在编写run方法。
    System.out.println("hello lambda!");
};
new Thread(task).start();
```

不过上面的用法很少见，一般都是直接把Lambda作为参数。

* **示例4：隐式final**

Lambda表达式的实质其实还是匿名内部类，而匿名内部类在访问外部局部变量时，要求变量必须声明为`final`！不过我们在使用Lambda表达式时无需声明`final`，这并不是说违反了匿名内部类的规则，因为Lambda底层会隐式的把变量设置为`final`，在后续的操作中，一定不能修改该变量：

正确示范：

```java
// 定义一个局部变量
int num = -1;
Runnable r = () -> {
    // 在Lambda表达式中使用局部变量num，num会被隐式声明为final
    System.out.println(num);
};
new Thread(r).start();// -1
```

错误案例：

```java
// 定义一个局部变量
int num = -1;
Runnable r = () -> {
    // 在Lambda表达式中使用局部变量num，num会被隐式声明为final，不能进行任何修改操作
    System.out.println(num++);
};
new Thread(r).start();//报错
```

#### 25.4.3 函数式接口

经过前面的学习，相信大家对于Lambda表达式已经有了初步的了解。总结一下：

- Lambda表达式是接口的匿名内部类的简写形式
- 接口必须满足：内部只有一个函数

其实这样的接口，我们称为函数式接口，我们学过的`Runnable`、`Comparator`都是函数式接口的典型代表。但是在实践中，函数接口是非常脆弱的，只要有人在接口里添加多一个方法，那么这个接口就不是函数接口了，就会导致编译失败。Java 8提供了一个特殊的注解`@FunctionalInterface`来克服上面提到的脆弱性并且显示地表明函数接口。而且jdk8版本中，对很多已经存在的接口都添加了`@FunctionalInterface`注解，例如`Runnable`接口：

![runnable](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251516536.png)

另外，Jdk8默认提供了一些函数式接口供我们使用：

* **Function类型接口**

```java
@FunctionalInterface
public interface Function<T, R> {
	// 接收一个参数T，返回一个结果R
    R apply(T t);
}
```

Function代表的是有参数，有返回值的函数。还有很多类似的Function接口：

| 接口名                 | 描述                                            |
| :--------------------- | ----------------------------------------------- |
| `BiFunction<T,U,R>`    | 接收两个T和U类型的参数，并且返回R类型结果的函数 |
| `DoubleFunction<R>`    | 接收double类型参数，并且返回R类型结果的函数     |
| `IntFunction<R>`       | 接收int类型参数，并且返回R类型结果的函数        |
| `LongFunction<R>`      | 接收long类型参数，并且返回R类型结果的函数       |
| `ToDoubleFunction<T>`  | 接收T类型参数，并且返回double类型结果           |
| `ToIntFunction<T>`     | 接收T类型参数，并且返回int类型结果              |
| `ToLongFunction<T>`    | 接收T类型参数，并且返回long类型结果             |
| `DoubleToIntFunction`  | 接收double类型参数，返回int类型结果             |
| `DoubleToLongFunction` | 接收double类型参数，返回long类型结果            |

看出规律了吗？这些都是一类函数接口，在Function基础上衍生出的，要么明确了参数不确定返回结果，要么明确结果不知道参数类型，要么两者都知道。

* **Consumer系列**

```java
@FunctionalInterface
public interface Consumer<T> {
	// 接收T类型参数，不返回结果
    void accept(T t);
}
```

Consumer系列与Function系列一样，有各种衍生接口，这里不一一列出了。不过都具备类似的特征：那就是不返回任何结果。

* **Predicate系列**

```java
@FunctionalInterface
public interface Predicate<T> {
	// 接收T类型参数，返回boolean类型结果
    boolean test(T t);
}
```

Predicate系列参数不固定，但是返回的一定是boolean类型。

* **Supplier系列**

```java
@FunctionalInterface
public interface Supplier<T> {
	// 无需参数，返回一个T类型结果
    T get();
}
```

Supplier系列，英文翻译就是“供应者”，顾名思义：只产出，不收取。所以不接受任何参数，返回T类型结果。

#### 25.4.4 方法引用

方法引用使得开发者可以将已经存在的方法作为变量来传递使用。方法引用可以和Lambda表达式配合使用。

* **语法**

总共有四类方法引用：

| 语法                   | 描述                             |
| ---------------------- | -------------------------------- |
| 类名::静态方法名       | 类的静态方法的引用               |
| 类名::非静态方法名     | 类的非静态方法的引用             |
| 实例对象::非静态方法名 | 类的指定实例对象的非静态方法引用 |
| 类名::new              | 类的构造方法引用                 |

* **示例**

首先我们编写一个集合工具类，提供一个方法：

```java
public class CollectionUtil{
    /**
      * 利用function将list集合中的每一个元素转换后形成新的集合返回
      * @param list 要转换的源集合
      * @param function 转换元素的方式
      * @param <T> 源集合的元素类型
      * @param <R> 转换后的元素类型
      * @return
      */
    public static <T,R> List<R> convert(List<T> list, Function<T,R> function){
        List<R> result = new ArrayList<>();
        list.forEach(t -> result.add(function.apply(t)));
        return result;
    }
}

```

可以看到这个方法接收两个参数：

- `List<T> list`：需要进行转换的集合
- `Function<T,R>`：函数接口，接收T类型，返回R类型。用这个函数接口对list中的元素T进行转换，变为R类型

接下来，我们看具体案例：

1. **类的静态方法引用**

```java
List<Integer> list = Arrays.asList(1000, 2000, 3000);
```

我们需要把这个集合中的元素转为十六进制保存，需要调用`Integer.toHexString()`方法：

```java
public static String toHexString(int i) {
    return toUnsignedString0(i, 4);
}
```

这个方法接收一个 i 类型，返回一个`String`类型，可以用来构造一个`Function`的函数接口：

我们先按照Lambda原始写法，传入的Lambda表达式会被编译为`Function`接口，接口中通过`Integer.toHexString(i)`对原来集合的元素进行转换：

```java
// 通过Lambda表达式实现
List<String> hexList = CollectionUtil.convert(list, i -> Integer.toHexString(i));
System.out.println(hexList);// [3e8, 7d0, bb8]
```

上面的Lambda表达式代码块中，只有对`Integer.toHexString()`方法的引用，没有其它代码，因此我们可以直接把方法作为参数传递，由编译器帮我们处理，这就是静态方法引用：

```java
// 类的静态方法引用
List<String> hexList = CollectionUtil.convert(list, Integer::toHexString);
System.out.println(hexList);// [3e8, 7d0, bb8]
```

2. **类的非静态方法引用**

接下来，我们把刚刚生成的`String`集合`hexList`中的元素都变成大写，需要借助于String类的`toUpperCase()`方法：

```java
public String toUpperCase() {
    return toUpperCase(Locale.getDefault());
}
```

这次是非静态方法，不能用类名调用，需要用实例对象，因此与刚刚的实现有一些差别，我们接收集合中的每一个字符串`s`。但与上面不同然后`s`不是`toUpperCase()`的参数，而是调用者：

```java
// 通过Lambda表达式，接收String数据，调用toUpperCase()
List<String> upperList = CollectionUtil.convert(hexList, s -> s.toUpperCase());
System.out.println(upperList);// [3E8, 7D0, BB8]
```

因为代码体只有对`toUpperCase()`的调用，所以可以把方法作为参数引用传递，依然可以简写：

```java
// 类的成员方法
List<String> upperList = CollectionUtil.convert(hexList, String::toUpperCase);
System.out.println(upperList);// [3E8, 7D0, BB8]
```

3. **指定实例的非静态方法引用**

下面一个需求是这样的，我们先定义一个数字`Integer num = 2000`，然后用这个数字和集合中的每个数字进行比较，比较的结果放入一个新的集合。比较对象，我们可以用`Integer`的`compareTo`方法:

```java
public int compareTo(Integer anotherInteger) {
    return compare(this.value, anotherInteger.value);
}
```

先用Lambda实现，

```java
List<Integer> list = Arrays.asList(1000, 2000, 3000);

// 某个对象的成员方法
Integer num = 2000;
List<Integer> compareList = CollectionUtil.convert(list, i -> num.compareTo(i));
System.out.println(compareList);// [1, 0, -1]
```

与前面类似，这里Lambda的代码块中，依然只有对`num.compareTo(i)`的调用，所以可以简写。但是，需要注意的是，这次方法的调用者不是集合的元素，而是一个外部的局部变量`num`，因此不能使用 `Integer::compareTo`，因为这样是无法确定方法的调用者。要指定调用者，需要用 `对象::方法名`的方式：

```java
// 某个对象的成员方法
Integer num = 2000;
List<Integer> compareList = CollectionUtil.convert(list, num::compareTo);
System.out.println(compareList);// [1, 0, -1]
```

4. **构造函数引用**

最后一个场景：把集合中的数字作为毫秒值，构建出`Date`对象并放入集合，这里我们就需要用到Date的构造函数：

```java
/**
  * @param   date   the milliseconds since January 1, 1970, 00:00:00 GMT.
  * @see     java.lang.System#currentTimeMillis()
  */
public Date(long date) {
    fastTime = date;
}
```

我们可以接收集合中的每个元素，然后把元素作为`Date`的构造函数参数：

```java
// 将数值类型集合，转为Date类型
List<Date> dateList = CollectionUtil.convert(list, i -> new Date(i));
// 这里遍历元素后需要打印，因此直接把println作为方法引用传递了
dateList.forEach(System.out::println);
```

上面的Lambda表达式实现方式，代码体只有`new Date()`一行代码，因此也可以采用方法引用进行简写。但问题是，构造函数没有名称，我们只能用`new`关键字来代替：

```java
// 构造方法
List<Date> dateList = CollectionUtil.convert(list, Date::new);
dateList.forEach(System.out::println);
```

注意两点：

- 上面代码中的 `System.out::println` 其实是 指定对象System.out的非静态方法println的引用
- 如果构造函数有多个，可能无法区分导致传递失败

### 25.5 接口的默认方法和静态方法

Java 8使用两个新概念扩展了接口的含义：默认方法和静态方法。

#### 25.5.1 默认方法

默认方法使得开发者可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。

默认方法和抽象方法之间的区别在于抽象方法需要实现，而默认方法不需要。接口提供的默认方法会被接口的实现类继承或者覆写，例子代码如下：

```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or 
    // may not implement (override) them.
    default String notRequired() { 
        return "Default implementation"; 
    }        
}

private static class DefaultableImpl implements Defaulable {
}

private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```

Defaulable接口使用关键字default定义了一个默认方法`notRequired()`。DefaultableImpl类实现了这个接口，同时默认继承了这个接口中的默认方法；OverridableImpl类也实现了这个接口，但覆写了该接口的默认方法，并提供了一个不同的实现。

#### 25.5.2 静态方法

Java 8带来的另一个有趣的特性是在接口中可以定义静态方法，我们可以直接用接口调用这些静态方法。例子代码如下：

```java
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}
```

下面的代码片段整合了默认方法和静态方法的使用场景：

```java
public static void main( String[] args ) {
    // 调用接口的静态方法，并且传递DefaultableImpl的构造函数引用来构建对象
    Defaulable defaulable = DefaulableFactory.create( DefaultableImpl::new );
    System.out.println( defaulable.notRequired() );
	// 调用接口的静态方法，并且传递OverridableImpl的构造函数引用来构建对象
    defaulable = DefaulableFactory.create( OverridableImpl::new );
    System.out.println( defaulable.notRequired() );
}
```

这段代码的输出结果如下：

```
Default implementation
Overridden implementation
```

由于JVM上的默认方法的实现在字节码层面提供了支持，因此效率非常高。**默认方法允许在不打破现有继承体系的基础上改进接口**。该特性在官方库中的应用是：给`java.util.Collection`接口添加新方法，如`stream()`、`parallelStream()`、`forEach()`和`removeIf()`等等。

尽管默认方法有这么多好处，但在实际开发中应该谨慎使用：在复杂的继承体系中，默认方法可能引起歧义和编译错误。如果你想了解更多细节，可以参考官方文档。

### 25.6 Optional

Java应用中最常见的bug就是空值异常。

`Optional`仅仅是一个容器，可以存放`T类型`的值或者`null`。它提供了一些有用的接口来避免显式的`null`检查，可以参考Java 8官方文档了解更多细节。

接下来看一点使用Optional的例子：可能为空的值或者某个类型的值：

```java
Optional< String > fullName = Optional.ofNullable( null );
System.out.println( "Full Name is set? " + fullName.isPresent() );        
System.out.println( "Full Name: " + fullName.orElseGet( () -> "[none]" ) ); 
System.out.println( fullName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
```

* 如果`Optional`实例持有一个非空值，则`isPresent()`方法返回`true`，否则返回`false`；
* 如果`Optional`实例持有`null`，`orElseGet()`方法可以接受一个lambda表达式生成的默认值；
* `map()`方法可以将现有的`Optional`实例的值转换成新的值；
* `orElse()`方法与`orElseGet()`方法类似，但是在持有null的时候返回传入的默认值，而不是通过Lambda来生成。

上述代码的输出结果如下：

```
Full Name is set? false
Full Name: [none]
Hey Stranger!
```

再看下另一个简单的例子：

```java
Optional< String > firstName = Optional.of( "Tom" );
System.out.println( "First Name is set? " + firstName.isPresent() );        
System.out.println( "First Name: " + firstName.orElseGet( () -> "[none]" ) ); 
System.out.println( firstName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
System.out.println();
```

这个例子的输出是：

```
First Name is set? true
First Name: Tom
Hey Tom!
```

如果想了解更多的细节，请参考官方文档。

### 25.7 Streams

新增的Stream API（`java.util.stream`）将生成环境的函数式编程引入了Java库中。**这是目前为止最大的一次对Java库的完善**，以便开发者能够写出更加有效、更加简洁和紧凑的代码。

Steam API 极大得简化了集合操作（后面我们会看到不止是集合），首先看下这个叫Task的类：

```java
public class Streams  {
    private enum Status {
        OPEN, CLOSED
    };

    private static final class Task {
        private final Status status;
        private final Integer points;

        Task( final Status status, final Integer points ) {
            this.status = status;
            this.points = points;
        }

        public Integer getPoints() {
            return points;
        }

        public Status getStatus() {
            return status;
        }

        @Override
        public String toString() {
            return String.format( "[%s, %d]", status, points );
        }
    }
}
```

Task类有一个points属性，另外还有两种状态：OPEN或者CLOSED。现在假设有一个task集合：

```java
final Collection< Task > tasks = Arrays.asList(
    new Task( Status.OPEN, 5 ),
    new Task( Status.OPEN, 13 ),
    new Task( Status.CLOSED, 8 ) 
);
```

首先看一个问题：在这个task集合中一共有多少个OPEN状态的？计算出它们的points属性和。在Java 8之前，要解决这个问题，则需要使用foreach循环遍历task集合；但是在Java 8中可以利用steams解决：包括一系列元素的列表，并且支持顺序和并行处理。

```java
// Calculate total points of all active tasks using sum()
final long totalPointsOfOpenTasks = tasks
    .stream()
    .filter( task -> task.getStatus() == Status.OPEN )
    .mapToInt( Task::getPoints )
    .sum();

System.out.println( "Total points: " + totalPointsOfOpenTasks );
```

运行这个方法的控制台输出是：

```
Total points: 18
```

这里有很多知识点值得说。

* 首先，`tasks`集合被转换成`steam`表示；
* 其次，在`steam`上的`filter`操作会过滤掉所有`CLOSED`的`task`；
* 第三，`mapToInt`操作基于`tasks`集合中的每个`task`实例的`Task::getPoints`方法将`task`流转换成`Integer`集合；
* 最后，通过`sum`方法计算总和，得出最后的结果。

在学习下一个例子之前，还需要记住一些steams（点此更多细节）的知识点。Steam之上的操作可分为中间操作和晚期操作。

* **中间操作（Pipelining）**会返回一个新的steam——执行一个中间操作（例如filter）并不会执行实际的过滤操作，而是创建一个新的steam，并将原steam中符合条件的元素放入新创建的steam。

* 晚期操作（例如forEach或者sum），会遍历steam并得出结果或者附带结果；在执行晚期操作之后，steam处理线已经处理完毕，就不能使用了。在几乎所有情况下，晚期操作都是立刻对steam进行遍历。

steam的另一个价值是创造性地**支持并行处理（parallel processing）**。对于上述的tasks集合，我们可以用下面的代码计算所有task的points之和：

```java
// Calculate total points of all tasks
final double totalPoints = tasks
   .stream()
   .parallel()
   .map( task -> task.getPoints() ) // or map( Task::getPoints ) 
   .reduce( 0, Integer::sum );

System.out.println( "Total points (all tasks): " + totalPoints );
```

这里我们使用parallel方法并行处理所有的task，并使用reduce方法计算最终的结果。控制台输出如下：

```
Total points（all tasks）: 26.0
```

对于一个集合，经常需要根据某些条件对其中的元素分组。利用steam提供的API可以很快完成这类任务，代码如下：

```java
// Group tasks by their status
final Map< Status, List< Task > > map = tasks
    .stream()
    .collect( Collectors.groupingBy( Task::getStatus ) );
System.out.println( map );
```

控制台的输出如下：

```java
{CLOSED=[[CLOSED, 8]], OPEN=[[OPEN, 5], [OPEN, 13]]}
```

最后一个关于tasks集合的例子问题是：如何计算集合中每个任务的点数在集合中所占的比重，具体处理的代码如下：

```java
// Calculate the weight of each tasks (as percent of total points) 
final Collection< String > result = tasks
    .stream()                                        // Stream< String >
    .mapToInt( Task::getPoints )                     // IntStream
    .asLongStream()                                  // LongStream
    .mapToDouble( points -> points / totalPoints )   // DoubleStream
    .boxed()                                         // Stream< Double >
    .mapToLong( weigth -> ( long )( weigth * 100 ) ) // LongStream
    .mapToObj( percentage -> percentage + "%" )      // Stream< String> 
    .collect( Collectors.toList() );                 // List< String > 

System.out.println( result );
```

控制台输出结果如下：

```
[19%, 50%, 30%]
```

最后，正如之前所说，Steam API 不仅可以作用于Java集合，传统的IO操作（从文件或者网络一行一行得读取数据）可以受益于steam处理，这里有一个小例子：

```java
final Path path = new File( filename ).toPath();
try( Stream< String > lines = Files.lines(path, StandardCharsets.UTF_8) ) {
    lines.onClose(() -> System.out.println("Done!"))
        .forEach( System.out::println );
}
```

Stream的方法`onClose()` 返回一个等价的有额外句柄的Stream，当Stream的`close()`方法被调用的时候这个句柄会被执行。Stream API、Lambda表达式还有接口默认方法和静态方法支持的方法引用，是Java 8对软件开发的现代范式的响应。

### 25.8 并行数组

Java8版本新增了很多新的方法，用于支持并行数组处理。最重要的方法是`parallelSort()`，可以显著加快多核机器上的数组排序。下面的例子论证了 parallexXxx 系列的方法：

```java
package com.javacodegeeks.java8.parallel.arrays;

import java.util.Arrays;
import java.util.concurrent.ThreadLocalRandom;

public class ParallelArrays {
    public static void main( String[] args ) {
        long[] arrayOfLong = new long [ 20000 ];        

        Arrays.parallelSetAll( arrayOfLong, 
            index -> ThreadLocalRandom.current().nextInt( 1000000 ) );
        Arrays.stream( arrayOfLong ).limit( 10 ).forEach( 
            i -> System.out.print( i + " " ) );
        System.out.println();

        Arrays.parallelSort( arrayOfLong );        
        Arrays.stream( arrayOfLong ).limit( 10 ).forEach( 
            i -> System.out.print( i + " " ) );
        System.out.println();
    }
}
```

上述这些代码使用`parallelSetAll()`方法生成20000个随机数，然后使用`parallelSort()`方法进行排序。这个程序会输出乱序数组和排序数组的前10个元素。上述例子的代码输出的结果是：

```
Unsorted: 591217 891976 443951 424479 766825 351964 242997 642839 119108 552378 
Sorted: 39 220 263 268 325 607 655 678 723 793
```
