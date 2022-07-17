# Java：Stream流式编程

> 不得不说，Java中的Stream流式编程结合lambda表达式，真的会使得代码简化很多；虽然说牺牲了一定的可读性，但只要用的熟悉了应该还是很容易就能一言看懂；就抽空稍微系统的学习一下常用的一些使用姿势。

## 1. 概述

Stream是Java8 API的新成员，它允许以声明性方式处理数据集合；

有以下特点：

1. **代码简洁**：函数式编程写出的代码简洁且意图明确，使用stream接口让你从此告别for循环。 

2. **多核友好**：Java函数式编程使得编写并行程序从未如此简单，你需要的全部就是调用一下方法。 

**使用流程：**

* 第一步：把集合转换为流stream
* 第二步：操作stream流，stream流在管道中经过**中间操作(intermediate operation)**的处理，最后由**最终操作(terminal operation)**得到前面处理的结果

**操作符：**中间操作符&终止操作符

## 2. 中间操作符

**filter**：用于通过设置的条件过滤出元素

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
//  功能描述:根据条件过滤集合数据
List<String> filtered = strings.stream().filter(s -> !s.isEmpty()).collect(Collectors.toList());
```

**distinct**：返回一个元素各异（根据流所生成元素的hashCode和equals方法实现）的流

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream().filter(i -> i % 2 == 0).distinct().forEach(System.out::println);

// 去除集合中重复数据
List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
List<String> distincted = strings.stream().distinct().collect(Collectors.toList());
```

**limit**：会返回一个不超过给定长度的流。

```java
List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
List<String> limited = strings.stream().limit(3).collect(Collectors.toList());
```

**skip**：返回一个扔掉了前n个元素的流。

```java
List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
List<String> skiped = strings.stream().skip(3).collect(Collectors.toList());
```

**sorted**：返回排序后的流。

```java
List<String> strings1 = Arrays.asList("abc", "abd", "aba", "efg", "abcd","jkl", "jkl");
List<String> sorted1 = strings1.stream().sorted().collect(Collectors.toList());

/**
 * 功能描述 : 对集合进行排序
 * @return : void
 */
@Test
public void sorted(){
    List<String> strings1 = Arrays.asList("abc", "abd", "aba", "efg", "abcd","jkl", "jkl");
    List<String> strings2 = Arrays.asList("张三", "李四", "王五", "赵柳", "张哥","李哥", "王哥");
    List<Integer> strings3 = Arrays.asList(10, 2, 30, 22, 1,0, -9);
    List<String> sorted1 = strings1.stream().sorted().collect(Collectors.toList());
    List<String> sorted2 = strings2.stream().sorted(
        Collections.reverseOrder(Collator.getInstance(Locale.CHINA))).collect(Collectors.toList());
    List<Integer> sorted3 = strings3.stream().sorted().collect(Collectors.toList());
    out.println(sorted1);
    out.println(sorted2);
    out.println(sorted3);
}
```

> ```xml
> <!--apache集合操作工具包-->
> <dependency>
>     <groupId>org.apache.commons</groupId>
>     <artifactId>commons-collections4</artifactId>
>     <version>4.4</version>
> </dependency>
> ```

**map**：接受一个函数作为参数。**这个函数会被应用到每个元素上**，并将其映射成一个新的元素（使用映射一词，是因为它和转换类似，但其中的细微差别在于它是“创建一个新版本”而不是去“修改”）。

```java
List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
List<String> mapped = strings.stream().map(str->str+"-itcast").collect(Collectors.toList());
```

**flatMap**：使用flatMap方法的效果是，各个数组并不是分别映射成一个流，而是映射成流的内容。所有使用`map(Arrays::stream)`时生成的**单个流都被合并起来，即扁平化为一个流**。

```java
List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
Stream<Character> flatMap = strings.stream().flatMap(Java8StreamTest::getCharacterByString);
```

> map 和 flatMap辨析：
>
> * map：对流中每一个元素进行处理
> * flatMap：流扁平化，让你把一个流中的“每个值”都换成另一个流，**然后把所有的流连接起来成为一个流** 
>
> **总结**：map是对一级元素进行操作，flatmap是对二级元素操作。
>
> **本质区别**：map返回一个值；flatmap返回一个流，多个值。
>
> **应用场景**：map对集合中每个元素加工,返回加工后结果；flatmap对集合中每个元素加工后，做扁平化处理后（拆分层级，放到同一层）然后返回
>
> ```java
> /**
>  * 功能描述:  通过使用map、flatMap把字符串转换为字符输出对比区别
>  * @return : void
>  */
> @Test
> public void flatMap2Map(){
>     List<String> strings = Arrays.asList("abc", "abc", "bc", "efg", "abcd","jkl", "jkl");
> 
>     Stream<Stream<Character>> mapStream = strings.stream().map(StreamDemo::getCharacterByString);
>     mapStream.forEach(System.out::print);
>     // mapStream.forEach(s -> s.forEach(System.out::print));
>     System.out.println("\n------------------------------------------------");
>     Stream<Character> flatMap = strings.stream().flatMap(StreamDemo::getCharacterByString);
>     flatMap.forEach(System.out::print);
> }
> 
> /**
> * 功能描述:字符串转换为字符流
> * @param str
> * @return : java.util.stream.Stream<java.lang.Character>
> */
> public static Stream<Character> getCharacterByString(String str) {
>     List<Character> characterList = new ArrayList<>();
>     for (Character character : str.toCharArray()) {
>         characterList.add(character);
>     }
>     return characterList.stream();
> }
> ```

## 3. 终止操作符

**anyMatch**：检查是否至少匹配一个元素，返回boolean。

```java
List<String> strings = Arrays.asList("abc", "abd", "aba", "efg", "abcd","jkl", "jkl");
boolean b = strings.stream().anyMatch(s -> s == "abc");
```

**allMatch**：  检查是否匹配所有元素，返回boolean。

```java
List<String> strings = Arrays.asList("abc", "abd", "aba", "efg", "abcd","jkl", "jkl");
boolean b = strings.stream().allMatch(s -> s == "abc");
```

**noneMatch**：检查是否没有匹配所有元素，返回boolean。

```java
List<String> strings = Arrays.asList("abc", "abd", "aba", "efg", "abcd","jkl", "jkl");
boolean b = strings.stream().noneMatch(s -> s == "abc");
```

**findAny**：将返回当前流中的任意元素。

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
Optional<String> any = strings.stream().findAny();
```

**findFirst**：返回第一个元素

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
Optional<String> first = strings.stream().findFirst();
```

> **findFirst & findAny辨析：**findAny并不是一定取的第一个值，在开启parallelStream的时候
>
> ```java
> @Test
> public void testFindFirstAndfindAny(){
>     List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
>     for (int i = 0; i < 10000; i++) {
>         String str1 = strings.parallelStream().findAny().get();
>         System.out.println(str1);
> 
>         /*String str2 = strings.parallelStream().findFirst().get();
> System.out.println(str2);*/
>     }
> }
> ```

**forEach**：遍历流。

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
strings.stream().forEach(s -> out.println(s));
```

**collect**：收集器，将流转换为其他形式。

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
// 转换成set
Set<String> set = strings.stream().collect(Collectors.toSet());
// 转换成list
List<String> list = strings.stream().collect(Collectors.toList());
// 转换成map
Map<String, String> map = strings.stream().collect(
    Collectors.toMap(v ->v.concat("_name"), v1 -> v1, (v1, v2) -> v1));
```

**reduce**：可以将流中元素反复结合起来，得到一个值。

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
Optional<String> reduce = strings.stream().reduce((acc,item) -> {return acc+item;});
if(reduce.isPresent()){
    System.out.println(reduce.get());
}
```

> ```java
> /**
>  * 功能描述 : 将流中元素反复结合起来，得到一个值
>  * @return : void
>  */
> @Test
> public void reduce(){
>     List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
>     // reduce方法一
>     Optional<String> reduce1 = strings.stream().reduce((acc, item) -> acc+item);
>     System.out.println(reduce1.isPresent() ? reduce1.get() : "");
>     // reduce方法二
>     String reduce2 = strings.stream().reduce("itcast", (acc, item) -> acc + item);
>     System.out.println(reduce2);
>     // reduce方法三
>     ArrayList<String> reduce3 = strings.stream().reduce(
>         new ArrayList<String>(),
>         new BiFunction<ArrayList<String>, String, ArrayList<String>>() {
>             @Override
>             public ArrayList<String> apply(ArrayList<String> acc, String item) {
>                 acc.add(item);
>                 return acc;
>             }
>         },
>         new BinaryOperator<ArrayList<String>>() {
>             @Override
>             public ArrayList<String> apply(ArrayList<String> acc, ArrayList<String> item) {
>                 return acc;
>             }
>         }
>     );
>     System.out.println(reduce3);
> }
> ```

**count**：返回流中元素总数。

```java
List<String> strings = Arrays.asList("cv", "abd", "aba", "efg", "abcd","jkl", "jkl");
long count = strings.stream().count();
```

