# Java：Javers进行对象比较

> 记录一种能方便比较两个Java对象异同的方法。

日常开发中，不可避免的存在要对两个对象进行比较，当然可以通过遍历对象中的所有值进行一个比较，最近发现了一个较为快捷的比较方式，通过借助于一个package：**JaVers**。

## 基础使用

**导包**

```xml
<dependency>
    <groupId>org.javers</groupId>
    <artifactId>javers-core</artifactId>
    <version>6.6.3</version>
</dependency>
```

**最基础的进行对象之间的比较：**

> 使用 `javers.compare(Object, Object)` 进行比较，实则使用的是对应的 equal 方法进行比较。

```java
// 定义类：
@Builder
@AllArgsConstructor
public class Address {
    private String city;
    private String street;
}

// 比较两个类
Address address1 = new Address("New York","5th Avenue");
Address address2 = new Address("New York","6th Avenue");

Javers javers = JaversBuilder.javers().build();
Diff diff = javers.compare(address1, address2);
ValueChange change = diff.getChangesByType(ValueChange.class).get(0);
System.out.println(diff);
```

**输出比较结果：**

```java
Diff:
* changes on cn.xyc.domain.Address/ :
  - 'street' changed: '5th Avenue' -> '6th Avenue'
```

## [Examples — Diff](https://javers.org/documentation/diff-examples)

这部分内容就摘录一些官网文档中的一些内容

### [Compare two Entity objects](https://javers.org/documentation/diff-examples/#compare-entities)

> 先定义一下对象：`Employee.java`
>
> ```java
> package cn.xyc.domain;
> 
> import lombok.AllArgsConstructor;
> import lombok.Builder;
> import org.javers.core.metamodel.annotation.Id;
> import org.javers.core.metamodel.annotation.TypeName;
> 
> import java.util.ArrayList;
> import java.util.List;
> import java.util.Set;
> 
> @Builder
> @AllArgsConstructor
> @TypeName("Employee")
> public class Employee {
> 
>     @Id
>     private String name;
>     private PositionEnum position;
>     private int salary;
>     private int age;
>     private Employee boss;
>     private List<Employee> subordinates;
>     private Address primaryAddress;
>     private Set<String> skills;
> 
>     public Employee(String name) {
>         this.name = name;
>     }
> 
>     public Employee(String name, int salary) {
>         this.name = name;
>         this.salary = salary;
>     }
> 
>     public Employee addSubordinate(Employee employee) {
>         employee.boss = this;
> 
>         if(subordinates == null){
>             subordinates = new ArrayList<>();
>         }
>         subordinates.add(employee);
>         return this;
>     }
> 
>     public Employee addSubordinates(Employee... employees) {
>         for (Employee e : employees){
>             addSubordinate(e);
>         }
>         return this;
>     }
> 
>     @Override
>     public String toString() {
>         return "Employee{" +
>                 "name='" + name + '\'' +
>                 '}';
>     }
> }
> ```
>
> 定义类：`Address.java`
>
> ```java
> package cn.xyc.domain;
> 
> import lombok.AllArgsConstructor;
> import lombok.Builder;
> 
> @Builder
> @AllArgsConstructor
> public class Address {
> 
>     private String city;
>     private String street;
>     
>     public Address(String city) {
>         this.city = city;
>     }
> }
> ```
>
> 定义枚举：`PositionEnum.java`
>
> ```java
> package cn.xyc.domain;
> 
> public enum  PositionEnum {
>     Assistant,
>     Secretary,
>     Developer,
>     Specialist,
>     Saleswoman,
>     ScrumMaster,
>     Townsman,
>     Hero,
>     Trainee
> }
> ```

**正式开始比较：**

```java
// 两个实体类，这里表示同一个人
Employee employeeOld = Employee.builder()
    .name("zhangsan")
    .age(20)
    .salary(1000)
    .position(PositionEnum.Townsman)
    .primaryAddress(new Address("hangzhou"))
    .skills(Sets.asSet("management"))
    .subordinates(Lists.asList(new Employee("lisi")))
    .build();

Employee employeeNew = Employee.builder()
    .name("zhangsan")
    .age(21)
    .salary(1500)
    .position(PositionEnum.Hero)
    .boss(new Employee("wangwu"))
    .position(PositionEnum.Townsman)
    .primaryAddress(new Address("beijing"))
    .skills(Sets.asSet("management", "agile coaching"))
    .subordinates(Lists.asList(new Employee("lisi"), new Employee("zhaoliu")))
    .build();

// 创建Javer
Javers javers = JaversBuilder.javers().build();
// 比较不同
Diff diff = javers.compare(employeeOld, employeeNew);
// 判断是否改变
System.out.println("是否改变：" + diff.hasChanges());
// 输出改变的内容
System.out.println("改变内容：" + diff);
```

输出结果：

```bash
是否改变：true
改变内容：Diff:
* new object: Employee/zhaoliu
  - 'name' = 'zhaoliu'
* changes on Employee/zhangsan :
  - 'age' changed: '20' -> '21'   # 属性 changed 原值 -> 新值
  - 'boss' = 'Employee/wangwu'
  - 'primaryAddress.city' changed: 'hangzhou' -> 'beijing'
  - 'salary' changed: '1000' -> '1500'
  - 'skills' collection changes :
     · 'agile coaching' added
  - 'subordinates' collection changes :
     1. 'Employee/zhaoliu' added
* new object: Employee/wangwu
  - 'name' = 'wangwu'
```

> 不同类型的输出结果：
>
> * 遍历修改结果：`diff.getChanges().forEach(change -> System.out.println("- " + change));`
>
> ```bash
> iterating over changes:
>     - NewObject{ new object: Employee/wangwu }
>     - NewObject{ new object: Employee/zhaoliu }
>     - InitialValueChange{ property: 'name', left:'',  right:'zhaoliu' }
>     - InitialValueChange{ property: 'name', left:'',  right:'wangwu' }
>     - ValueChange{ property: 'salary', left:'1000',  right:'1500' }
>     - ValueChange{ property: 'age', left:'20',  right:'21' }
>     - ReferenceChange{ property: 'boss', left:'',  right:'Employee/wangwu' }
>     - ListChange{ property: 'subordinates', elementChanges:1, left.size: 1, right.size: 2}
>     - SetChange{ property: 'skills', elementChanges:1, left.size: 1, right.size: 2}
>     - ValueChange{ property: 'city', left:'hangzhou',  right:'beijing' }
> ```
>
> * 按照修改对象分组：
>
> ```java
> System.out.println("iterating over changes grouped by objects");
> diff.groupByObject().forEach(byObject -> {
>     System.out.println("* changes on " +byObject.getGlobalId().value() + " : ");
>     byObject.get().forEach(change -> System.out.println("  - " + change));
> });
> ```
>
> ```bash
> iterating over changes grouped by objects
> * changes on Employee/zhaoliu : 
>   - NewObject{ new object: Employee/zhaoliu }
>   - InitialValueChange{ property: 'name', left:'',  right:'zhaoliu' }
> * changes on Employee/zhangsan : 
>   - ValueChange{ property: 'salary', left:'1000',  right:'1500' }
>   - ValueChange{ property: 'age', left:'20',  right:'21' }
>   - ReferenceChange{ property: 'boss', left:'',  right:'Employee/wangwu' }
>   - ListChange{ property: 'subordinates', elementChanges:1, left.size: 1, right.size: 2}
>   - SetChange{ property: 'skills', elementChanges:1, left.size: 1, right.size: 2}
>   - ValueChange{ property: 'city', left:'hangzhou',  right:'beijing' }
> * changes on Employee/wangwu : 
>   - NewObject{ new object: Employee/wangwu }
>   - InitialValueChange{ property: 'name', left:'',  right:'wangwu' }
> ```
>
> * 输出JSON格式：`System.out.println(javers.getJsonConverter().toJson(diff));`
>
> ```json
> {
>   "changes": [
>     {
>       "changeType": "NewObject",
>       "globalId": {
>         "entity": "Employee",
>         "cdoId": "wangwu"
>       }
>     },
> 	// ...
>   ]
> }
> ```

### [Compare graphs](https://javers.org/documentation/diff-examples/#compare-graphs)

这里还是用员工类进行说明

* **新增对象（员工雇佣）**

```java
@Test
public void shouldDetectHired() {

    Employee oldBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Great Developer"));

    Employee newBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Great Developer"),
        new Employee("Hired One"),
        new Employee("Hired Second"));

    Javers javers = JaversBuilder.javers().build();
    Diff diff = javers.compare(oldBoss, newBoss);
    List changes = diff.getObjectsByChangeType(NewObject.class);
    changes.stream().forEach(System.out::println);
}

// 输出:
// Employee{name='Hired Second'}
// Employee{name='Hired One'}
```

* **减少对象（无情优化）**

```java
@Test
public void shouldDetectFired() {

    Employee oldBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Great Developer"),
        new Employee("Team Lead").addSubordinates(
            new Employee("Another Dev"),
            new Employee("To Be Fired")
        ));

    Employee newBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Great Developer"),
        new Employee("Team Lead").addSubordinates(
            new Employee("Another Dev")
        ));

    Javers javers = JaversBuilder.javers().build();
    Diff diff = javers.compare(oldBoss, newBoss);

    List<ObjectRemoved> changes = diff.getChangesByType(ObjectRemoved.class);
    System.out.println(changes);
}

// 输出：[ObjectRemoved{ object removed: Employee/To Be Fired }]
```

- **薪水修改**

```java
@Test
public void shouldDetectSalaryChange(){

    Employee oldBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Noisy Manager"),
        new Employee("Great Developer", 10000));

    Employee newBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Noisy Manager"),
        new Employee("Great Developer", 20000));

    Javers javers = JaversBuilder.javers().build();
    Diff diff = javers.compare(oldBoss, newBoss);
    ValueChange change =  diff.getChangesByType(ValueChange.class).get(0);
    System.out.println(change);
}

// 输出：ValueChange{ property: 'salary', left:'10000',  right:'20000' }
```

- **换个老板**

```java
@Test
public void shouldDetectBossChange() {

    Employee oldBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Manager One")
        .addSubordinate(new Employee("Great Developer")),
        new Employee("Manager Second"));

    Employee newBoss = new Employee("Big Boss").addSubordinates(
        new Employee("Manager One"),
        new Employee("Manager Second")
        .addSubordinate(new Employee("Great Developer")));

    Javers javers = JaversBuilder.javers().build();
    Diff diff = javers.compare(oldBoss, newBoss);

    ReferenceChange change = diff.getChangesByType(ReferenceChange.class).get(0);
    System.out.println(change);
    System.out.println(diff);
}

// 输出：
// ReferenceChange{ property: 'boss', left:'Employee/Manager One',  right:'Employee/Manager Second' }
// Diff:
// * changes on Employee/Manager One :
//   - 'subordinates' collection changes :
//      0. 'Employee/Great Developer' removed
// * changes on Employee/Great Developer :
//   - 'boss' reference changed: 'Employee/Manager One' -> 'Employee/Manager Second'
// * changes on Employee/Manager Second :
//   - 'subordinates' collection changes :
//      0. 'Employee/Great Developer' added
```

### [Compare top-level collections](https://javers.org/documentation/diff-examples/#compare-valueobjects)

```java
// Person中有两个String，第一个值上加了@Id的注解
List<Person> oldList = Lists.asList( new Person("tommy", "Tommy Smart") );
List<Person> newList = Lists.asList( new Person("tommy", "Tommy C. Smart") );
```

* 不指定类型：`javers.compare(Object, Object)`

  ```java
  Diff diff = javers.compare(oldList, newList);
  System.out.println(diff);
  
  // # 输出：
  // Diff:
  // * changes on org.javers.core.graph.LiveGraphFactory$ListWrapper/ :
  //   - 'list' collection changes :
  //      0. 'Person{login='tommy', name='Tommy Smart', position=null}' changed to 'Person{login='tommy', name='Tommy C. Smart', position=null}'
  ```

* 指定类型：`javers.compareCollections(Collection, Collection, Class)`

  ```java
  Diff diff = javers.compareCollections(oldList, newList, Person.class);
  //there should be one change of type {@link ValueChange}
  List<ValueChange> changesByType = diff.getChangesByType(ValueChange.class);
  System.out.println(changesByType.size());  // 1
  System.out.println(changesByType.get(0).getPropertyName());  // name
  System.out.println(changesByType.get(0).getLeft());  // Tommy Smart
  System.out.println(changesByType.get(0).getRight());  // Tommy C. Smart
  
  
  System.out.println(diff);
  // Diff:
  // * changes on cn.xyc.domain.Person/tommy :
  // 	- 'name' changed: 'Tommy Smart' -> 'Tommy C. Smart'
  ```

可以对比一下上面的不同点：

* 第一个使用了`javers.compare`，直接把List中的Person作为一个整体进行比较，即按照List的维度去比较；
* 第二使用了`javers.compareCollections`，按照Person维度进行比较；

### [Groovy diff example](https://javers.org/documentation/diff-examples/#groovy-diff-example)

JaVers 很好的适配了 Groovy

```groovy
package cn.xyc

import org.javers.core.JaversBuilder
import groovy.transform.TupleConstructor
import org.javers.core.metamodel.annotation.Id
import spock.lang.Specification

class GroovyDiffExample extends Specification {

    @TupleConstructor
    class Person {
        @Id login
        String lastName
    }

    def "should calculate diff for GroovyObjects"(){
        given:
        def javers = JaversBuilder.javers().build()

        when:
        def diff = javers.compare(
                new Person('bob','Uncle'),
                new Person('bob','Martin')
        )

        then:
        diff.changes.size() == 1
        diff.changes[0].left == 'Uncle'
        diff.changes[0].right == 'Martin'
    }
}
```

> 使用 Groovy 记得导包：
>
> ```xml
> <!--groovy依赖-->
> <dependency>
>     <groupId>org.codehaus.groovy</groupId>
>     <artifactId>groovy-all</artifactId>
>     <version>2.4.15</version>
>     <scope>test</scope>
> </dependency>
> 
> <!--spock依赖-->
> <dependency>
>     <groupId>org.spockframework</groupId>
>     <artifactId>spock-core</artifactId>
>     <version>1.3-groovy-2.4</version>
>     <scope>test</scope>
> </dependency>
> ```

### [Custom Value comparator example](https://javers.org/documentation/diff-examples/#custom-value-comparator-example)

这里给了一个对于BigDecimal的比较，直接使用 `javers.compare` 比较BigDecimal是无法比较的，会报错的...不支持的，这里给出了两个比较方法：

* 四舍五入式的：

  ```groovy
  class ValueObject {
      BigDecimal value
      List<BigDecimal> values
  }
  
  def "should compare BigDecimal properties with desired precision"(){
      given:
          def javers = JaversBuilder.javers()
          .registerValue(BigDecimal, new CustomBigDecimalComparator(2)).build()
  
      expect:
          javers.compare(new ValueObject(value: 1.123), new ValueObject(value: 1.124)).changes.size() == 0
      javers.compare(new ValueObject(value: 1.12),  new ValueObject(value: 1.13)).changes.size()  == 1
  }
  ```

  > 在类`CustomBigDecimalComparator.java`内部
  >
  > ```java
  > public class CustomBigDecimalComparator implements CustomValueComparator<BigDecimal>{
  >     
  > 	// ...
  >     
  >     @Override
  >     public boolean equals(BigDecimal a, BigDecimal b) {
  >         return round(a).equals(round(b));
  >     }
  >     
  >     // ...
  > }
  > ```

* 真实比较

  ```groovy
  def "should compare BigDecimal with fixed equals"() {
      when: "comparing using BigDecimal.equals()"
      def javers = JaversBuilder.javers().build()
  
      then:
      javers.compare(new ValueObject(value: 1.000), new ValueObject(value: 1.00)).changes.size() == 1
  
      when: "comparing using arithmetical equals()"
      javers = JaversBuilder.javers()
              .registerValue(BigDecimal, new BigDecimalComparatorWithFixedEquals())
              .build()
  
      then:
      javers.compare(new ValueObject(value: 1.000), new ValueObject(value: 1.00)).changes.size() == 0
      javers.compare(new ValueObject(value: 1.100), new ValueObject(value: 1.20)).changes.size() == 1
  }
  ```

  > 在类`BigDecimalComparatorWithFixedEquals.java`内部
  >
  > ```java
  > public class BigDecimalComparatorWithFixedEquals implements CustomValueComparator<BigDecimal> {
  >     @Override
  >     public boolean equals(BigDecimal a, BigDecimal b) {
  >         return a.compareTo(b) == 0;
  >     }
  > 
  >     @Override
  >     public String toString(BigDecimal value) {
  >         return value.stripTrailingZeros().toString();
  >     }
  > }
  > ```

### [Custom Property comparator example](https://javers.org/documentation/diff-examples/#custom-property-comparator-example)

自定义比较器来实现一个比较操作：

* 自定义比较类的实现，代码中很容易看出其是干啥

```groovy
package cn.xyc

import org.javers.common.collections.Sets
import org.javers.core.diff.changetype.PropertyChangeMetadata
import org.javers.core.diff.changetype.container.ContainerElementChange
import org.javers.core.diff.changetype.container.SetChange
import org.javers.core.diff.changetype.container.ValueAdded
import org.javers.core.diff.changetype.container.ValueRemoved
import org.javers.core.diff.custom.CustomPropertyComparator
import org.javers.core.metamodel.property.Property

class FunnyStringComparator implements CustomPropertyComparator<String, SetChange> {

    @Override
    Optional<SetChange> compare(String left, String right, PropertyChangeMetadata metadata, Property property) {

        if (equals(left, right)) {
            return Optional.empty()
        }

        Set leftSet = left.toCharArray().toSet()
        Set rightSet = right.toCharArray().toSet()

        List<ContainerElementChange> changes = []
        Sets.difference(leftSet, rightSet).forEach{ c -> changes.add(new ValueRemoved(c))}
        Sets.difference(rightSet, leftSet).forEach{c -> changes.add(new ValueAdded(c))}

        return Optional.of(new SetChange(metadata, changes))
    }

    @Override
    boolean equals(String a, String b) {
        a.toCharArray().toSet() == b.toCharArray().toSet()
    }

    @Override
    String toString(String value) {
        return value
    }
}
```

* 对上述自定义比较器进行一个测试

```groovy
package cn.xyc

import org.javers.core.JaversBuilder
import org.javers.core.diff.changetype.container.SetChange
import spock.lang.Specification

class FunnyStringComparatorTest extends Specification {

    class ValueObject {
        String value
    }

    def "should use FunnyStringComparator to compare String properties"(){
        given:
        def javers = JaversBuilder.javers()
                .registerCustomType(String, new FunnyStringComparator()).build()

        when:
        def diff = javers.compare(new ValueObject(value: "aaa"), new ValueObject(value: "a"))
        println "first diff: "+ diff

        then:
        diff.changes.size() == 0

        when:
        diff = javers.compare(new ValueObject(value: "aaa"), new ValueObject(value: "b"))
        println "second diff: "+ diff

        then:
        diff.changes.size() == 1
        diff.changes[0] instanceof SetChange
        diff.changes[0].changes.size() == 2 // two item changes in this SetChange
    }
}
```

至此使用 JaVer 进行对象比较差不多就结束了 Over，当然还可以借助于 SpringBoot 等框架来实现比较之类的，但这里就不再展开了。

## 参考

Javers官网：https://javers.org/

https://javers.org/documentation/getting-started/

https://javers.org/documentation/diff-examples/