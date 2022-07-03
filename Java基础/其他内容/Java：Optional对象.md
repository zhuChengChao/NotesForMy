# Java：Optional对象

> 工作过程中碰到npe的问题可以说算是比较高频的情况了，当然解决方式也是多种多样，比如最为常用的方式直接用`==null`进行判断；其实这里可以用jDK8中提供的Optional对象实现更为优雅的判断。
>
> 其实的话在Optional的文档里已经讲的很清晰了，源码也写的比较清晰。

## 前期准备

这里构造了一个对象用于对后续的使用进行一定程度说明：

```java
@Data
public class User {

    private String name;
    private String sex;
    private Address address;

}

@Data
class Address {

    private String cityName;
    private Integer number;
}
```

## 成员&构造

### 成员属性

```java
// 作为一个空对象，后续会用到，使用默认构造函数进行构造
private static final Optional<?> EMPTY = new Optional<>();

// 用于保存使用Option时所传入的值
private final T value;
```

### 构造函数

构造函数全是私有的，不暴露出去使用。

```java
// 默认构造，给相应的成员value赋值为null，上述EMPTY就使用了该函数
private Optional() {
    this.value = null;
}

// 有参构造，且传入的参数不能为null
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}

// Objects#requireNonNull 方法
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

## 静态方法

静态方法可以说就是Options使用较为频繁的部分了，一般都是以静态函数作为起步，然后衔接Options中的其他方法来实现一些操作，比如：

```java
Optional.ofNullable(user).orElse(new User("zhangsan", "men"));
```

因为通过这些静态方法返回的都是Optional对象，因此可以衔接后续对象中的方法来完成一些操作。

### empty

作用：**直接返回上述的EMPTY成员**

```java
public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
```

### of&ofNullable

#### of 方法

直接调用有参构造，**构造一个Optional对象，然后返回**；

```java
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
```

> **需要注意的：从上面的有参构造来看，这个方法是会抛出NPE异常的**

#### ofNullable 方法

和上述 of 方法相比的，这个方法不会抛出异常，而是先对传入的 value 进行判断：

* 为null，调用`empty()`抛出成员EMPTY；
* 不为空才调用上述 `of` 方法，即构造一个Optional对象返回；

```java
public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}
```

> 使用频率No.1

## 非静态方法

使用静态方法作为起步，然后返回一个Option的对象，直接可以衔接Option对象中的方法来完成一些操作。

### orElse & orElseGet & orElseThrow

`orElse & orElseGet & orElseThrow` 三个函数的作用，当使用Option的静态方法`of() & ofNullable` 传入的值：

* 不为null时，直接返回value（通过Option构造函数传入的值）；
* 为null时进行相应的操作；

#### orElse

```java
public T orElse(T other) {
    return value != null ? value : other;
}

// eg：当user为null时，创建新的User
User user = null;
user = Optional.ofNullable(user).orElse(new User("zhangsan", "men"));
System.out.println(user.getName());  // zhagnsan

// eg:当user不为null时，不进行操作
User user = new User("zhangsan", "men");
user = Optional.ofNullable(user).orElse(new User("lisi", "male"));
System.out.println(user.getName());  // zhangsan
```

#### orElseGet

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

// eg:当address为null，调用orElseGet，之后不为null再调用orElseGet
user.setAddress(Optional.ofNullable(user.getAddress()).orElseGet(
    () -> new Address("hangzhou", 288)));
System.out.println(user.getAddress().getCityName());// hangzhou

user.setAddress(Optional.ofNullable(user.getAddress()).orElseGet(
    () -> new Address("shanghai", 288)));
System.out.println(user.getAddress().getCityName()); // hangzhou
```

#### orElseThrow

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}

// eg:
User user = null;

Optional.ofNullable(null).orElseThrow(
    () -> new NullPointerException("参数为null"));
```

### isPresent & ifPresent

#### isPresent

很好理解，就是判断Option中的value是否为null，不为null的话返回true；

```java
public boolean isPresent() {
    return value != null;
}

// eg:
User user = new User();
boolean isPresent = Optional.ofNullable(user).isPresent();
System.out.println(isPresent);  // true
isPresent = Optional.ofNullable(null).isPresent();
System.out.println(isPresent);  // false
```

#### ifPresent

当Option中的value不为null时，执行一个操作

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}

// eg:
User user = new User("zhangsan", "male");
System.out.println(user.getSex());  // male
Optional.ofNullable(user).ifPresent(
    u -> u.setSex("female"));
System.out.println(user.getSex());  // female
```

### map & flatMap

关于参数映射操作的两个函数，当对于的Optional中的value不为null时执行一个映射操作。

> 需要注意的是，**这个函数返回的是Optional对象**，也是为了方便能继续执行链式操作，即继续后续的map等操作。

#### map

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    // 先对传入的mapper进行一个非空判断
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        // 不为null时执行映射操作
        return Optional.ofNullable(mapper.apply(value));
    }
}

// eg:获取user中的name
User user = new User("zhangsan", "male");

String name = Optional.ofNullable(user).map(
    u -> u.getName()).get();

System.out.println(name);
```

#### flatMap

```java
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        // 不为null时执行映射操作
        return Objects.requireNonNull(mapper.apply(value));
    }
}

// eg:上面的例子
String name1 = Optional.ofNullable(user).flatMap(
                user1 -> Optional.ofNullable(user1)).get().getName();
```

**区别辨析：**这里可以看出map&flatMap的区别，其在于传入的参数：

* map：传入的Function，返回的是`? extends U`
* flatMap：传入的Function，返回的还是Optional类型。

### filter & get

这里还剩下最后两个函数，一起拿出来说说。

#### filter

顾名思义，过滤的一个函数，传入一个predicate，满足则返回，否则就返回空。

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}

// eg：
User user = new User("zhangsan", "male");

boolean passFilter = Optional.ofNullable(user).filter(
    u -> u.getSex().equals("male")).isPresent();
System.out.println(passFilter);  // true

passFilter = Optional.ofNullable(user).filter(
    u -> u.getSex().equals("female")).isPresent();
System.out.println(passFilter);  // false
```

#### get

为了完整性，这里提下get函数，就是获取一下Optional中的value：

```java
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}

// eg：获取Option中的值
User user01 = new User("zhangsan", "male");

User user02 = Optional.ofNullable(user01).get();
System.out.println(user01 == user02);
```

## 参考

https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html

https://www.cnblogs.com/rjzheng/p/9163246.html