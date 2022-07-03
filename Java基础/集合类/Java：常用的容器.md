# Java：常用的容器

> 对 Java 中的 **常用容器**，做一个微不足道的小小小小记

## 容器类概述

常见容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。

![Collection](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251612692.PNG)

![Map](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251612833.PNG)

### Collection

**Set**

| Set           | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| TreeSet       | 基于红黑树实现，支持有序性操作；<br />例如：根据一个范围查找元素的操作。<br />但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN) |
| HashSet       | 基于哈希表(HashMap)实现，支持快速查找，但不支持有序性操作。<br />并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的 |
| LinkedHashSet | 具有 HashSet 的查找效率，底层是链表+哈希表。<br />使用哈希表存储元素，再维护一个**双向链表**保存元素的插入信息 |

**List**

| List       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| ArrayList  | 基于动态数组实现，支持随机访问                               |
| Vector     | 和 ArrayList 类似，但它是线程安全的                          |
| LinkedList | 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。<br />不仅如此，LinkedList 还可以用作栈、队列和双向队列 |

**Queue**

| Queue         | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| LinkedList    | 基于链表实现的队列，可以用它来实现**双向队列**               |
| PriorityQueue | 基于堆结构实现，可以用它来实现**优先队列**                   |
| ArrayDeque    | 实现了Deque(双向队列)，Deque继承自Queue<br />使用了可变数组，可以做为队列来使用，也可以作为栈使用<br />ArrayDeque不支持`null`值 |

### Map

| Map           | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| TreeMap       | 基于红黑树实现                                               |
| HashMap       | 基于哈希表实现                                               |
| HashTable     | 和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。<br />它是遗留类，不应该去使用它。<br />现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁 |
| LinkedHashMap | 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU，Least Recently Used）顺序 |

## List & Set & Map 

List & Set & Map  之间的大致区别：

| 比较       | LIst                                                         | Set                                                          | Map                                                          |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 继承接口   | Collection                                                   | Collection                                                   | 是个接口                                                     |
| 常见实现类 | AbstractList<br />常用的子类有：<br />ArrayList<br />LinkedList<br />Vector | AbstractSet<br />常用的子类有：<br />HashSet<br />LinkedHashSet<br />TreeSet | 实现的类：<br />HashMap<br />HashTable<br />LinkedHashMap    |
| 常见方法   | add()<br />remove()<br />clear()<br />get()<br />contains()<br />size()<br />... | add()<br />remove()<br />clear()<br />get()<br />contains()<br />size()<br />... | put()<br />get()<br />remove()<br />clear()<br />containsKey()<br />containsValue()<br />keySet()<br />values()<br />size()<br />... |
| 元素       | 可重复                                                       | 不可重复                                                     | 不可重复                                                     |
| 顺序       | 有序                                                         | 一般无序(由hashcode决定)                                     | 一般无序                                                     |
| 线程安全   | Vector线程安全                                               |                                                              | HashTable线程安全                                            |

## Collection & Collections 

### Collection

是一个**集合接口**。它提供了对集合对象进行基本操作的通用接口方法。List，Set，Queue接口都继承Collection；直接实现该接口的类只有 AbstractCollection 类，该类也只是一个抽象类，提供了对集合类操作的一些基本实现。

### Collections

是不属于 Java 的集合框架的，它是**集合类的一个工具类/帮助类**。此类不能被实例化， 服务于 Java 的 Collection 框架。它包含有关集合操作的静态多态方法，实现对各种集合的搜索、排序、线程安全等操作。

**常用功能**

* `public static <T> boolean addAll(Collection<T> c, T... elements)`：往集合中添加一些元素。
* `public static void shuffle(List<?> list)` ：打乱顺序，打乱集合顺序。
* `public static <T> void sort(List<T> list) `：将集合中元素按照默认规则排序。
* `public static <T> void sort(List<T> list，Comparator<? super T> )` :将集合中元素按照指定规则排序。

```java
public class CollectionsDemo {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        // 原来写法
        // list.add(12);
        // list.add(14);
        // list.add(15);
        // list.add(1000);
        // 采用工具类 完成 往集合中添加元素
        Collections.addAll(list, 5, 222, 1，2);
        System.out.println(list);
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
} 
// 结果：
// [5, 222, 1, 2]
// [1, 2, 5, 222]
```

### Collections 中的 sort 方法

**Comparator比较器**

排序方法：`public static <T> void sort(List<T> list)` :将集合中元素按照默认规则排序。

如，对字符串类型进行比较：

```java
public class CollectionsDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
}

// 结果：
// [aba, cba, nba, sba]
```

说到排序了，简单的说就是两个对象之间比较大小，那么在 Java 中提供了两种比较实现的方式，一种是比较死板的采用 `java.lang.Comparable` 接口去实现，一种是灵活的当我需要做排序的时候在去选择的`java.util.Comparator` 接口完成。

那么我们采用的 `public static <T> void sort(List<T> list)` 这个方法完成的排序，**实际上要求了被排序的类型需要实现 Comparable 接口完成比较的功能**，在String类型上如下：

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    
    // 实现了Comparable<String>的方法，该接口内有抽象方法public int compareTo(T o);
    // 在String内对其进行了实现
    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
}
```

String 类实现了这个接口，并完成了比较规则的定义，但是这样就把这种规则写死了，那比如我想要字符串按照第一个字符降序排列，那么这样就要修改String的源代码，这是不可能的了。

```java
// 重写排序的规则
@Override
public int compareTo(Person o) {
    // return 0; // 认为元素都是相同的
    // 自定义比较的规则, 比较两个人的年龄(this,参数Person)
    // return this.getAge() - o.getAge();  // 年龄升序排序
    return o.getAge() - this.getAge();  // 年龄升序排序
}
```

那么这个时候我们可以使用 `public static <T> void sort(List<T> list，Comparator<? super T> )` 方法灵活的完成，这个里面就涉及到了 **Comparator这个接口**，位于位于 `java.util` 包下，排序是 comparator 能实现的功能之一，该接口代表一个比较器，比较器具有可比性！顾名思义就是做排序的，通俗地讲需要比较两个对象谁排在前谁排在后，那么比较的方法就是：`public int compare(String o1, String o2)` ：比较其两个参数的顺序。

两个对象比较的结果有三种：大于，等于，小于。

如果要按照升序排序，则o1 小于o2，返回（负数），相等返回0，01大于02返回（正数）;

如果要按照降序排序，则o1 小于o2，返回（正数），相等返回0，01大于02返回（负数）

**总结 Comparator 的排序规则**: o1-o2 为升序；o2-o1 为降序

操作如下:

```java
public class CollectionsDemo3 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法 按照第一个单词的降序
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.charAt(0) ‐ o1.charAt(0);
            }
        });
        System.out.println(list);
    }
}

// 结果如下：
// [sba, nba, cba, aba]
```

**Comparable 与 Comparator 两个接口的区别**

**Comparable**：强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的 compareTo 方法被称为它的自然比较方法。只能在类中实现 `compareTo()` 一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过`Collections.sort`（和 `Arrays.sort`）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**Comparator**：强行对某个对象进行整体排序。可以将 Comparator 传递给 sort 方法（如`Collections.sort` 或 `Arrays.sort`），从而允许在排序顺序上实现精确控制。还可以使用Comparator 来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象 collection 提供排序。

> 案例说明：
>
> 创建一个学生类，存储到ArrayList集合中完成指定排序操作。
>
> * Student 初始类
>
> ```java
> public class Student{
>     private String name;
>        private int age;
>        public Student() {
>        } 
>        public Student(String name, int age) {
>            this.name = name;
>            this.age = age;
>        } 
>        public String getName() {
>            return name;
>        } 
>        public void setName(String name) {
>            this.name = name;
>        } 
>        public int getAge() {
>            return age;
>        } 
>        public void setAge(int age) {
>            this.age = age;
>        } 
>        @Override
>        public String toString() {
>            return "Student{" +
>                "name='" + name + '\'' +
>                ", age=" + age +
>                '}';
>        }
>    }
> ```
> 
>* 测试类：
> 
>```java
> public class Demo {
>  public static void main(String[] args) {
>         // 创建四个学生对象 存储到集合中
>         ArrayList<Student> list = new ArrayList<Student>();
>         list.add(new Student("rose",18));
>         list.add(new Student("jack",16));
>         list.add(new Student("abc",16));
>         list.add(new Student("ace",17));
>         list.add(new Student("mark",16));
>         /*
>          	让学生 按照年龄排序 升序
>          */
>          Collections.sort(list);  // 要求 该list中元素类型 必须实现比较器Comparable接口
>         for (Student student : list) {
>             System.out.println(student);
>         }
>     }
>    }
> ```
> 
>发现，当我们调用 `Collections.sort()` 方法的时候 程序报错了。
> 
>原因：如果想要集合中的元素完成排序，那么必须要实现比较器 Comparable 接口。
> 
>于是我们就完成了Student类的一个实现，如下：
> 
>```java
> public class Student implements Comparable<Student>{
>     	// ....
>            @Override
>            public int compareTo(Student o) {
>            return this.age‐o.age;//升序
>        }
>    }
> ```
> 
>再次测试，代码就OK 了效果如下：
> 
>```java
> Student{name='jack', age=16}
> Student{name='abc', age=16}
> Student{name='mark', age=16}
> Student{name='ace', age=17}
> Student{name='rose', age=18}
> ```
> 
>如果在使用的时候，想要独立的定义规则去使用 可以采用 `Collections.sort(List list,Comparetor c)` 方式，自己定义规则：
> 
>```java
> Collections.sort(list, new Comparator<Student>() {
>     @Override
>        public int compare(Student o1, Student o2) {
>            return o2.getAge()‐o1.getAge();  // 以学生的年龄降序
>        }
>    });
> ```
> 
>效果：
> 
>```java
> Student{name='rose', age=18}
> Student{name='ace', age=17}
> Student{name='jack', age=16}
> Student{name='abc', age=16}
> Student{name='mark', age=16}
> ```
> 
>如果想要规则更多一些，可以参考下面代码：
> 
>```java
> Collections.sort(list, new Comparator<Student>() {
>     @Override
>        public int compare(Student o1, Student o2) {
>            // 年龄降序
>            int result = o2.getAge()‐o1.getAge();//年龄降序
>            if(result==0){//第一个规则判断完了 下一个规则 姓名的首字母 升序
>                result = o1.getName().charAt(0)‐o2.getName().charAt(0);
>            } 
>            return result;
>        }
>    });
> ```
> 
>效果如下：
> 
>```java
> Student{name='rose', age=18}
> Student{name='ace', age=17}
> Student{name='abc', age=16}
> Student{name='jack', age=16}
> Student{name='mark', age=16}
> ```

## 参考

https://mp.weixin.qq.com/s/q1r9Pno6ANUzZ9wMzA-JSg

https://blog.csdn.net/qq_41701956/article/details/103223461