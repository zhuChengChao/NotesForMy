# Utils：Test For Groovy&Spock&Mock

> 之前大致看了下Mockito的使用，但在工作中使用更多的却是Groovy&Spock&PowerMock这一套单侧模式；那为了能快速上手，在周末就粗略的学下Groovy&Spock的基础使用姿势。

## Groovy快速上手

> 基本参考自：[Groovy for Java developer: learning Groovy-specific features](https://sysgears.com/articles/groovy-differences-java/)

### Groovy对象

Groovy bean 应该只包含字段，而 getter 和 setter 将被隐式创建：

创建一个User类：`User.groovy`

```groovy
class User {
    String firstName
    String lastName
}
```

对其使用：`GroovyDemo.groovy`

```groovy
class GroovyDemo {

    static void main(String[] args) {

        def user = new User()
        user.firstName = "John"
        user.lastName = "Doe"
    }
}
```

当然也可以显示的设置相应的 getter&setting 方法：

```groovy
class User {

    String firstName
    String lastName

    String getFirstName() {
        return firstName
    }

    void setFirstName(String firstName) {
        this.firstName = firstName
    }

    String getLastName() {
        return lastName
    }

    void setLastName(String lastName) {
        this.lastName = lastName
    }
}
```

对上述代码进行测试

```groovy
def user = new User()
user.firstName = "John"
user.lastName = "Doe"

println(user.firstName)
// 输出：The first name is [John]
```

### Groovy 闭包

Groovy 闭包：Groovy 提供了一种方法来创建作为第一类对象的函数；通过闭包，可以定义一个代码块，然后将其作为常规变量传递：

```groovy
// the simplest closure
def hello = {"Hello, $it!"}
assert hello('Chris') == 'Hello, Chris!'

// the closure that do not take any params
def bye = {->'Bye!'}
assert bye() == 'Bye!'

// the closure with several params
def niceHello = {firstName, lastName -> "Hello, $firstName $lastName!"}
assert niceHello('Chris', 'Bennett') == 'Hello, Chris Bennett!'
```

上述可见闭包默认采用一个 `it` 参数，可以通过指定命名参数或使用 `{-> ...}` 声明完全省略参数来更改此行为。

### Groovy lists&maps

Groovy 中 list和map类型的使用：

**List使用：**

* 定义：

```groovy
// an empty list
def emptyList = []

// predefined list
def list = ['One', 'Two', 'Three']
```

* 取值：

```groovy
def list = ['One', 'Two', 'Three']

// gets the first element from the list
assert list[0] == 'One'

// gets a range of elements from the list
assert list[1..2] == ['Two', 'Three']

// gets another range
assert list[-1..-2] == ['Three', 'Two']
```

* 迭代和转换列表：

```groovy
def list = [3,1,2]
// iterates the list
def emptyList = []

list.each {emptyList << "$it!"}
assert emptyList == ['3!', '1!', '2!']

// iterates the list and transforms each entry into a new value
// using the closure
assert list.collect {it * 2} == [6, 2, 4]

// sorts using the closure as a comparator
assert list.sort {it1, it2 -> it1 <=> it2} == [1, 2, 3]

// gets min or max using closure as comparator
assert list.min {it1, it2 -> it1 <=> it2} == 1
```

**Map使用：**

* 定义：

```groovy
// an empty map
def emptyMap = [:]

// predefined map
def map = [John: 10, Mark: 20, Peter: 'Not defined']
```

* 取值：

```groovy
def map = [John: 10, Mark: 20, Peter: 'Not defined']

// 方式1：数组方式取值
assert map['Peter'] == 'Not defined'

// 方式2：对象方式取值
assert map.Mark == 20

// 方式3：also you can preset default value that will be returned by
// the get method if key does not exist，用方法取值
assert map.get('Michael', 100) == 100
```

* 借助闭包实现数据的转换

```groovy
// iterates the map
def emptyMap = [:]
def map = [John: 10, Mark: 20, Peter: 'Not defined']
map.each { key, value ->
    emptyMap.put(key, "$key: $value" as String)
}
assert emptyMap == [John: 'John: 10', Mark: 'Mark: 20',
                    Peter: 'Peter: Not defined']

// iterates the map and transforms each entry using
// the closure, returns a list of transformed values
assert map.collect { key, value ->
    "$key: $value"
} == ['John: 10', 'Mark: 20', 'Peter: Not defined']

// sorts map elements using the closure as a comparator
map.put('Chris', 15)
assert map.sort { e1, e2 ->
    e1.key <=> e2.key
} == [John: 10, Chris: 15, Mark: 20, Peter: 'Not defined']
```

### 其他操作

* 对于所有类型，布尔运算符 `==` 与 Java equals 的工作方式相同（Java中 `==` 意味着原始类型的相等和对象的标识），若要按身份进行比较，请改用 is 方法：

  ```groovy
  assert [] == []
  assert ![].is([])
  ```

* `.*` 运算符用于对集合的所有成员执行操作：

  ```groovy
  assert [1, 2, 3] == ['a', 'ab', 'abc']*.size()
  ```

* `return` 可有可无

  ```groovy
  def foo() {
      // do something
      return something
  }
  // is equals to
  def bar() {
      // do something
      something
  }
  ```

> 很皮毛的记录了一些使用，更多内容可见：[Groovy 教程](https://www.w3cschool.cn/groovy/)

## Spock快速上手

> 主要参考自：[使用Spock框架进行单元测试](http://blog.2baxb.me/archives/1398)

### 概述

简单地说，spock是一个测试框架，它的核心特性有以下几个：

1. 可以应用于java或groovy应用的单元测试框架。
2. 测试代码使用基于groovy语言扩展而成的规范说明语言（specification language）。
3. 通过junit runner调用测试，兼容绝大部分junit的运行场景（ide，构建工具，持续集成等）。
4. 框架的设计思路参考了JUnit，jMock，RSpec，Groovy，Scala，Vulcans……

### 环境搭建

Maven导包

```xml
<!--groovy依赖-->
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.15</version>
    <scope>test</scope>
</dependency>

<!--spock依赖-->
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>1.3-groovy-2.4</version>
    <scope>test</scope>
</dependency>
```

> 上述的版本配置我是可用的...很坑找了很多都用不来:rage:

创建测试目录

1. 在IDEA的test目录下创建名为groovy的目录；
2. 右键目录→`Mark Directory As`→`Test Source Root`

### 概念说明

#### Specification

在Spock中，待测系统(system under test; SUT) 的行为是由规格(specification) 所定义的。在使用Spock框架编写测试时，测试类需要继承自Specification类。

```groovy
import spock.lang.Specification

class xxxTest extends Specification {}
```

#### Fields

Specification类中可以定义字段，这些字段在运行每个测试方法前会被重新初始化，跟放在`setup()`里是一个效果。

```groovy
def obj = new ClassUnderSpecification()
def coll = new Collaborator()
```

#### Fixture Methods

预先定义的几个固定的函数，与junit或testng中类似：

```groovy
// run before every feature method 每个feature方法执行前执行
def setup() {}   
// run after every feature method 每个feature方法执行后执行
def cleanup() {}       
// run before the first feature method 在第一个feature方法执行前执行
def setupSpec() {}      
// run after the last feature method 在最后一个feature方法执行后执行
def cleanupSpec() {}    
```

#### Feature methods

这是Spock规格(Specification)的核心，其描述了SUT应具备的各项行为；每个Specification都会包含一组相关的Feature methods，如要测试1+1是否等于2，可以编写一个函数：

```groovy
def "sum should return param1+param2"() {
    expect:
    sum.sum(1,1) == 2
}
```

#### Blocks

每个feature method又被划分为不同的block，不同的block处于测试执行的不同阶段，在测试运行时，各个block按照不同的顺序和规则被执行，如下图：

![Blocks2Phases](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261426325.png)

对上述的Blocks进一步说明。

##### setup Blocks

setup也可以写成given，在这个block中会放置与这个测试函数相关的初始化程序，如：

```groovy
setup:
def stack = new Stack()
def elem = "push me"
```

一般会在这个block中定义局部变量，定义mock函数等。

##### When ... Then Blocks

when与then需要搭配使用，在when中执行待测试的函数，在then中判断是否符合预期，如：

```groovy
// ...衔接上部分setup内容...
when:
stack.push(elem)  

then:
!stack.empty
stack.size() == 1
stack.peek() == elem
```

在then中判断是否符合预期：

* **断言：**条件类似junit中的assert，就像上面的例子，在then或expect中会默认assert所有返回值是boolean型的顶级语句。如果要在其它地方增加断言，需要显式增加assert关键字，如：

  ```groovy
  def setup() {
      stack = new Stack()
      assert stack.empty
  }
  ```

* **异常断言**：如果要验证有没有抛出异常，可以用`thrown()`，如下：

  ```groovy
  // ...衔接上部分setup内容...
  when:
  stack.pop()  
   
  then:
  thrown(EmptyStackException)
  stack.empty
  ```

  要获取抛出的异常对象，可以用以下语法：

  ```groovy
  // ...衔接上部分setup内容...
  when:
  stack.pop()  
   
  then:
  def e = thrown(EmptyStackException)
  e.cause == null
  ```

  如果要验证没有抛出某种异常，可以用notThrown()：

  ```groovy
  def "HashMap accepts null key"() {
    setup:
    def map = new HashMap()  
   
    when:
    map.put(null, "elem")  
   
    then:
    notThrown(NullPointerException)
  }
  ```

##### Expect Blocks

expect可以看做精简版的when+then，如：

```groovy
when:
def x = Math.max(1, 2)  
 
then:
x == 2
```

可以简化为：

```groovy
expect:
Math.max(1, 2) == 2
```

##### Cleanup Blocks

函数退出前做一些清理工作，如关闭资源等。

##### Where Blocks

做测试时最复杂的事情之一就是准备测试数据，尤其是要测试边界条件、测试异常分支等，这些都需要在测试之前规划好数据。但是传统的测试框架很难轻松的制造数据，要么依赖反复调用，要么用xml或者data provider函数之类难以理解和阅读的方式。比如说：

```groovy
def "maximum of two numbers"() {
    expect:
    // exercise math method for a few different inputs
    Math.max(1, 3) == 3
    Math.max(7, 4) == 7
    Math.max(0, 0) == 0
}
```

而在spock中，通过where block可以让这类需求实现起来变得非常优雅：

```groovy
def "maximum of two numbers"() {
    expect:
        Math.max(a, b) == c

    where:
        a | b || c
    3 | 5 || 5
    7 | 0 || 7
    0 | 0 || 0
}
```

上述例子实际会跑三次测试，相当于在for循环中执行三次测试，a/b/c的值分别为3/5/5，7/0/7和0/0/0。如果在方法前声明`@Unroll`，则会当成三个方法运行。

> 更进一步，可以为标记@Unroll的方法声明动态的spec名：

```groovy
    @Unroll
    def "maximum of #a and #b should be #c"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b || c
        3 | 5 || 5
        7 | 0 || 7
        0 | 0 || 0
    }
```

![@Unroll注解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261426512.PNG)

运行时，名称会被替换为实际的参数值。

除此之外，where block还有两种数据定义的方法，并且可以结合使用，如：

```groovy
def "maximum of #a and #b should be #c"() {
    expect:
    Math.max(a, b) == c

    where:
    a | _
    3 | _
    7 | _
    0 | _

    b << [5, 0, 0]

    c = a > b ? a : b
}
```

### 入门案例

> 首先创建一个简单的Java类：两数之和
>
> ```java
> public class Sum {
> 
>     public int sum(int first, int second){
>         return first + second;
>     }
> }
> ```

使用Spock进行编写测试代码：`SumTest.grovvy`

```groovy
package cn.xyc

import spock.lang.Specification

class SumTest extends Specification {

    def sum = new Sum()

    def "sum should return param1+param2"(){
        expect:
        sum.sum(1, 1) == 2
    }
}
```

至此，一个最简单的spock测试就写完了。

### 正常使用

比如日常结合Mock等测试工具来实现单测。

#### with Mock

在spock中创建一个mock对象非常简单：

```groovy
class PublisherTest extends Specification {

    Publisher publisher = new Publisher()
    // Mock两个对象
    Subscriber subscriber1 = Mock()
    Subscriber subscriber2 = Mock()

    def setup(){
        publisher.subscribers.add(subscriber1)
        publisher.subscribers.add(subscriber2)
    }
}
```

而创建了mock对象之后就可以对它的交互做验证了：

```groovy
class PublisherTest extends Specification {

    Publisher publisher = new Publisher()
    // Mock两个对象
    Subscriber subscriber1 = Mock()
    Subscriber subscriber2 = Mock()

    def setup(){
        publisher.subscribers.add(subscriber1)
        publisher.subscribers.add(subscriber2)
    }

    def "should send messages to all subscribers"() {
        when:
        publisher.send("hello")

        then:
        1 * subscriber1.receive("hello")
        1 * subscriber2.receive("hello")
    }

}
```

上面的例子里验证了：在publisher调用send时，两个subscriber都应该被调用一次`receive(“hello”)`。

> 示例中，表达式中的次数、对象、函数和参数部分都可以灵活定义：
>
> ```groovy
> // 调用1次receive("hello")   
> 1 * subscriber.receive("hello")   
> // 调用0次receive("hello")  
> 0 * subscriber.receive("hello") 
> // 调用1-3次receive("hello")  
> (1..3) * subscriber.receive("hello")
> // 至少调用1次receive("hello")  
> (1.._) * subscriber.receive("hello") 
> // 至多3次调用receive("hello") 
> (_..3) * subscriber.receive("hello") 
> // 任何次数调用（包括0次）
> _ * subscriber.receive("hello")   
> // 调用1次receive，且入参是hello
> 1 * subscriber.receive("hello")   
> // 调用1次receive，且入参不是hello
> 1 * subscriber.receive(!"hello")    
> // 调用1次receive，且入参为空
> 1 * subscriber.receive()     
> // 调用1次receive，且入参随意（可以为空）
> 1 * subscriber.receive(_)   
> // 同上？
> 1 * subscriber.receive(*_)
> // 调用1次receive，且入参非空
> 1 * subscriber.receive(!null)     
> // 调用1次receive，且入参必为string
> 1 * subscriber.receive(_ as String) 
> // an argument that satisfies the given predicate
> // (here: message length is greater than 3)
> 1 * subscriber.receive({ it.size() > 3 }) 
> // any method on subscriber, with any argument list
> 1 * subscriber._(*_)     
> // shortcut for and preferred over the above
> 1 * subscriber._         
> // any method call on any mock object
> 1 * _._                  
> // shortcut for and preferred over the above
> 1 * _                    
> ```

#### Stubbing

对mock对象定义函数的返回值可以用如下方法：

```groovy
subscriber.receive(_) >> "ok"  // 接受任意参数，然后返回"ok"
```

符号代表函数的返回值，执行上面的代码后，再调用`subscriber.receice`方法将返回ok。

如果要每次调用返回不同结果，可以使用：

```groovy
subscriber.receive(_) >>> ["ok", "error", "error", "ok"]
```

如果要做额外的操作，如抛出异常，可以使用：

```groovy
subscriber.receive(_) >> { throw new InternalError("ouch") }
```

而如果要每次调用都有不同的结果，可以把多次的返回连接起来：

```groovy
subscriber.receive(_) >>> ["ok", "fail", "ok"] >> { throw new InternalError() } >> "ok"
```

## 参考

[Groovy for Java developer: learning Groovy-specific features](https://sysgears.com/articles/groovy-differences-java/)

[使用Spock框架进行单元测试](http://blog.2baxb.me/archives/1398)

## 推荐阅读

[Spock单元测试框架实战指南一Spock是什么？它和JUnit有什么区别？](https://www.cnblogs.com/maoyx/p/14028040.html)

[Spock单元测试框架实战指南二-mock第三方依赖 ](https://www.cnblogs.com/maoyx/p/14040079.html)

[Spock单元测试框架实战指南三-If esle 多分支场景测试](https://www.cnblogs.com/maoyx/p/14045396.html)

[Spock单元测试框架实战指南四 - 异常测试 ](https://www.cnblogs.com/maoyx/p/14065793.html)

[Spock单元测试框架实战指南五 - void方法测试](https://www.cnblogs.com/maoyx/p/14071423.html)

[Spock单元测试框架实战指南六 - 静态方法测试](https://www.cnblogs.com/maoyx/p/14077254.html)

[Spock单元测试框架实战指南七 - 动态Mock](https://www.cnblogs.com/maoyx/p/14083700.html)

[Spock单元测试框架实战指南八 - 常用mock封装成基类](https://www.cnblogs.com/maoyx/p/14094917.html)

[Spock单元测试框架实战指南九 - 模拟抽象类方法](https://www.cnblogs.com/maoyx/p/14100565.html)

[Spock单元测试框架实战指南十 - 注意事项](https://www.cnblogs.com/maoyx/p/14106439.html)

