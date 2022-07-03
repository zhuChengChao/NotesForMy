# Utils：Test For Mockito

> 开发过程中对于码出来的代码自测当然是不可避免环节，即所谓的单测，好的单侧能极大的避免bug的出现；
>
> 但对于处处存在着依赖的应用，测试环境发布再测试显然是比较麻烦，因而就出现了较多的测试框架，比如Mockito，也因在工作过程中需要使用到该框架对自己写的代码进行测试，故找了写资料来学习一波，做些笔记方便后续回顾。
>
> > 官方文档传送门：https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html

## Mockito概述

Mockito 是一个模拟测试框架，主要功能是**模拟类/对象的行为**。

通俗地讲就是我们在对某个类测试时，涉及到一些RPC调用，数据库，缓存的操作等，这时候就可以将对应的依赖Mock掉（模拟），让其返回我们自己期望其出现的结果。

使用了 Mockito ，使得我们不用再对待测代码的外部依赖进行关注，直接对需要进行依赖的操作通过打桩的方式来处理需要依赖外部调用的业务逻辑，然后进关注和测试自己的业务逻辑。

**打桩：**打桩也叫 stub，存根。就是把所需的**测试数据塞进对象**中，关注点在于入参和出参。在Mockito中使用 `when(…).thenReturn(…)`等方式描述了对象以某个入参调用某个方法时，返回`thenReturn`中指定的出参。这个过程就叫**Stub打桩**。只要这个方法被stub了，每次调用，就会一直返回这个stub的值。

Mockito的**使用过程**很简单：

1. 将被测试接口相关的外部依赖对象转换为Mock对象，然后打桩。
2. 执行测试接口代码。
3. 校验返回的关键结果是否正确

## 环境准备

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 使用姿势

### Mock进行模拟

`org.mockito.Mockito` 的 mock 方法可以模拟类和接口。

**Mock类：**

```java
import org.junit.Assert;
import org.junit.Test;
import java.util.*;
import static org.mockito.Mockito.*;

@Test
public void testMock01() {

    // Random类的对象利用Mock来进行模拟
    Random mockRandom = mock(Random.class);
    // 指定调用 nextInt 方法时，永远返回 100
    when(mockRandom.nextInt()).thenReturn(100);

    Assert.assertEquals(100, mockRandom.nextInt());
    Assert.assertEquals(100, mockRandom.nextInt());
}
```

**Mock接口：**

```java
@Test
public void testMock02() {
    List mockList = mock(List.class);

    Assert.assertEquals(0, mockList.size());
    Assert.assertEquals(null, mockList.get(0));

    // 调用 mock 对象的写方法，是没有效果的
    mockList.add("a");  
    // 没有指定 size() 方法返回值，这里结果是默认值
    Assert.assertEquals(0, mockList.size());  
    // 没有指定 get(0) 返回值，这里结果是默认值
    Assert.assertEquals(null, mockList.get(0));   

    // 指定 get(0)时返回 a
    when(mockList.get(0)).thenReturn("a");          
    // 没有指定 size() 方法返回值，这里结果是默认值
    Assert.assertEquals(0, mockList.size());    
    // 因为上面指定了 get(0) 返回 a，所以这里会返回 a
    Assert.assertEquals("a", mockList.get(0));     
    // 没有指定 get(1) 返回值，这里结果是默认值
    Assert.assertEquals(null, mockList.get(1));     

}
```

### @Mock注解

`@Mock` 注解可以理解为对 mock 方法的一个替代。

> **注意：**使用该注解时，要使用`MockitoAnnotations.initMocks` 方法，让注解生效。

```java
@Mock
private Random random;

@Test
public void testMock03() {

    // MockitoAnnotations.initMocks(this);  // 已经废弃用下一个
    MockitoAnnotations.openMocks(this);

    when(random.nextInt()).thenReturn(100);

    Assert.assertEquals(100, random.nextInt());
}
```

这里的话`MockitoAnnotations.initMocks` 放在 junit 的 `@Before` 注解修饰的函数中更合适：

```java
@Mock
private Random random;   

@Before
public void before() {
    // 让注解生效
    MockitoAnnotations.openMocks(this);
}

@Test
public void testMock04() {

    when(random.nextInt()).thenReturn(100);

    Assert.assertEquals(100, random.nextInt());
}
```

`MockitoAnnotations.initMocks` 的一个替代方案是使用 MockitoJUnitRunner 。

```java
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;

import java.util.*;

import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class MockTest {

    @Mock
    private Random random;

    @Test
    public void test() {
        when(random.nextInt()).thenReturn(100);
        Assert.assertEquals(100, random.nextInt());
    }
}
```

### Mockito spy&@Spy

spy 和 mock不同，不同点是：

- spy 的参数是对象示例，mock 的参数是 class。
- 被 spy 的对象，调用其方法时默认会走真实方法；但mock 对象不会。

```java
public class MockTest {
    
    // 测试 spy
    @Test
    public void testSpy() {

        ExampleService spyExampleService = spy(new ExampleService());

        // 默认会走真实方法
        Assert.assertEquals(3, spyExampleService.add(1, 2));

        // 打桩后，不会走了
        when(spyExampleService.add(1, 2)).thenReturn(10);
        Assert.assertEquals(10, spyExampleService.add(1, 2));

        // 但是参数比匹配的调用，依然走真实方法
        Assert.assertEquals(3, spyExampleService.add(2, 1));
    }

    // 测试 mock
    @Test
    public void testMock() {

        ExampleService mockExampleService = mock(ExampleService.class);

        // 默认返回结果是返回类型int的默认值
        Assert.assertEquals(0, mockExampleService.add(1, 2));
    }
}

class ExampleService {

    int add(int a, int b) {
        return a+b;
    }
}
```

> 注意：mock 对象的方法的返回值默认都是返回类型的默认值。
>
> 例如，返回类型是 int，默认返回值是 0；返回类型是一个类，默认返回值是 null。

spy 对应注解 @Spy，和 @Mock 是一样用的：

```java
@Spy
private ExampleService spyExampleService;

@Test
public void testspyAnno() {

    MockitoAnnotations.openMocks(this);

    Assert.assertEquals(3, spyExampleService.add(1, 2));

    when(spyExampleService.add(1, 2)).thenReturn(10);
    Assert.assertEquals(10, spyExampleService.add(1, 2));

}
```

> 对于`@Spy`，如果发现修饰的变量是 null，会自动调用类的无参构造函数来初始化：
>
> ```java
> // 写法1
> @Spy
> private ExampleService spyExampleService;
> 
> // 写法2
> @Spy
> private ExampleService spyExampleService = new ExampleService();
> ```
>
> 当然前提就是要有默认的无参构造。

### @InjectMocks 注解

mockito 会将 `@Mock`、`@Spy` 修饰的对象自动注入到 `@InjectMocks` 修饰的对象中。注入方式有多种，mockito 会按照下面的顺序尝试注入：

1. 构造函数注入
2. 设值函数注入（set函数）
3. 属性注入

```java
package cn.xyc.demo;

import cn.xyc.demo.service.HttpService;
import cn.xyc.demo.service.ExampleService;
import org.junit.Assert;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.mockito.Mockito.when;

public class ExampleServiceTest {

    @Mock
    private HttpService httpService;

    @InjectMocks  // 会将httpService注入到exampleService中去
    private ExampleService exampleService = new ExampleService();

    @Test
    public void testInjectMocks() {

        MockitoAnnotations.openMocks(this);

        when(httpService.queryStatus()).thenReturn(0);
        Assert.assertEquals("你好", exampleService.hello());
    }
}
```

> 其中涉及到的业务类：
>
> ```java
> // 业务类1：
> import java.util.Random;
> 
> public class HttpService {
> 
>     public int queryStatus() {
>         // 发起网络请求，提取返回结果
>         // 这里用随机数模拟结果
>         return new Random().nextInt(2);
>     }
> }
> 
> // 业务类2
> public class ExampleService {
> 
>     private HttpService httpService;
> 
>     public String hello() {
>         int status = httpService.queryStatus();
>         if (status == 0) {
>             return "你好";
>         }
>         else if (status == 1) {
>             return "Hello";
>         }
>         else {
>             return "未知状态";
>         }
>     }
> }
> ```

### 参数匹配

#### 精准匹配

```java
List mockList = mock(List.class);
// 匹配姿势1：当mockList调用get(0)时，就返回0，这个 0 就是参数的精确匹配。
when(mockList.get(0)).thenReturn("a");

// 匹配姿势2：对于精确匹配，还可以用 eq，
when(mockStringList.get(eq(0))).thenReturn("a");
```

#### 模糊匹配

```java
// Mockito.anyInt() 匹配所有的 int
when(mockStringList.get(anyInt())).thenReturn("a");
```

> 还有很多的any类型，这里不列举了。

#### 匹配顺序

如果参数匹配即声明了精确匹配，也声明了模糊匹配；又或者同一个值的精确匹配出现了两次，使用时会匹配哪一个？会匹配符合匹配条件的最新声明的匹配。

```java
// 精确匹配 0
when(testList.get(0)).thenReturn("a");
Assert.assertEquals("a", testList.get(0));

// 精确匹配 0
when(testList.get(0)).thenReturn("b");
Assert.assertEquals("b", testList.get(0));

// 模糊匹配
when(testList.get(anyInt())).thenReturn("c");
Assert.assertEquals("c", testList.get(0));
Assert.assertEquals("c", testList.get(1));

// 精确匹配 0
when(testList.get(0)).thenReturn("a");
Assert.assertEquals("a", testList.get(0));
Assert.assertEquals("c", testList.get(1));
```

### 验证行为

Verify主要用于对Mock方法**入参**的校验，而Assert针对**返回值**的校验。

#### verify校验

> verify 可以校验 mock 对象是否发生过某些操作

**基础使用**

```java
// 静态导入会使代码更简洁
import static org.mockito.Mockito.*;  

public class MockTest {

    @Test
    public void testVerify(){
        //mock creation 创建mock对象
        List mockedList = mock(List.class);

        //using mock object 使用mock对象
        mockedList.add("one");
        mockedList.clear();

        //verification 验证是否执行过某些行为
        verify(mockedList).add("one");
        verify(mockedList).clear();
    }
}
```

**验证函数的确切调用次数**

```java
List mockedList = mock(List.class);

//using mock
mockedList.add("once");
mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

//following two verifications work exactly the same - times(1) is used by default
verify(mockedList).add("once");
verify(mockedList, times(1)).add("once");

//exact number of invocations verification
verify(mockedList, times(2)).add("twice");
verify(mockedList, times(3)).add("three times");

//verification using never(). never() is an alias to times(0)
verify(mockedList, never()).add("never happened");

//verification using atLeast()/atMost()
verify(mockedList, atLeastOnce()).add("three times");
verify(mockedList, atLeast(2)).add("three times");
verify(mockedList, atMost(5)).add("three times");
```

#### 断言校验

```
import org.junit.Assert;

// 校验值的姿势：
Assert.assertEquals("a", testList.get(0));
Assert.assertEquals("c", testList.get(1));

// 校验异常姿势：
Assert.assertTrue(ex instanceof RuntimeException);

// 直接校验失败：
Assert.fail();

// ...其他内容间相应的函数
```

#### 异常校验*

**校验方式1：**使用try...catch

```java
@Test
public void testExceptionWithTryCatch(){
	// 使用try...catch的方式来验证所抛出的异常
    try {
        Assert.fail("npe!");
    }catch (AssertionError e){
        // e.printStackTrace();
        Assert.assertEquals(e.getMessage(), "npe!");
    }
}
```

**校验方式2：**JUnit的@Test注解

```java
@Test(expected = ArithmeticException.class)
public void testExceptionWithAnnotation(){
    int i = 1/0;
}
```

**校验方式3：**Junit的@Rule和ExpectedException

```java
@Rule
public ExpectedException thrown = ExpectedException.none();

@Test
public void testExceptionWithRule() {
    thrown.expect(ArithmeticException.class);
    // thrown.expectMessage("/ by zero");

    int i = 1/0;
}
```

> 更详细内容见：[使用JUnit的ExpectedException和@Rule测试自定义异常](https://blog.csdn.net/dnc8371/article/details/106706650)

### 打桩使用

#### thenReturn&doReturn 设置方法的返回值

thenReturn设置方法的返回值：

```java
@Test
public void testThenReturn1() {

    Random mockRandom = mock(Random.class);
    // thenReturn 用来指定特定函数和参数调用的返回值
    when(mockRandom.nextInt()).thenReturn(1);

    Assert.assertEquals(1, mockRandom.nextInt());

}

@Test
public void testThenReturn2() {
    Random mockRandom = mock(Random.class);

    // thenReturn中可以指定多个返回值
    when(mockRandom.nextInt()).thenReturn(1, 2, 3);

    // 在调用时返回值依次出现。若调用次数超过返回值的数量，再次调用时返回最后一个返回值。
    Assert.assertEquals(1, mockRandom.nextInt());
    Assert.assertEquals(2, mockRandom.nextInt());
    Assert.assertEquals(3, mockRandom.nextInt());
    Assert.assertEquals(3, mockRandom.nextInt());
    Assert.assertEquals(3, mockRandom.nextInt());
}
```

doReturn 的作用和 thenReturn 相同，但使用方式不同：

```java
@Test
public void testDoReturn() {

    Random random = mock(Random.class);
    doReturn(1).when(random).nextInt();

    Assert.assertEquals(1, random.nextInt());
}
```

#### thenThrow&doThrow 让方法抛出异常

`when .. thenThrow`只能让有返回值的方法抛出异常，但不能让返回值是 void 的方法抛出异常：

```java
@Test
public void testThenThrow1() {

    Random mockRandom = mock(Random.class);

    // thenThrow 用来让函数调用抛出异常。
    when(mockRandom.nextInt()).thenThrow(new RuntimeException("异常"));
    // 等价于：
    // doThrow(new RuntimeException("异常")).when(random).nextInt();

    try {
        mockRandom.nextInt();
        Assert.fail();  // 上面会抛出异常，所以不会走到这里
    } catch (Exception ex) {
        Assert.assertTrue(ex instanceof RuntimeException);
        Assert.assertEquals("异常", ex.getMessage());
    }
}


@Test
public void testThenThrow2() {

    Random mockRandom = mock(Random.class);

    // thenThrow 中可以指定多个异常，在调用时异常依次出现。
    // 若调用次数超过异常的数量，再次调用时抛出最后一个异常。
    when(mockRandom.nextInt()).thenThrow(new RuntimeException("异常1"), new RuntimeException("异常2"));

    try {
        mockRandom.nextInt();
        Assert.fail();
    } catch (Exception ex) {
        Assert.assertTrue(ex instanceof RuntimeException);
        Assert.assertEquals("异常1", ex.getMessage());
    }

    try {
        mockRandom.nextInt();
        Assert.fail();
    } catch (Exception ex) {
        Assert.assertTrue(ex instanceof RuntimeException);
        Assert.assertEquals("异常2", ex.getMessage());
    }
}
```

> **错误用法：**当对应返回类型是 void 的函数，thenThrow 是无效的，要使用 doThrow
>
> ```java
> static class ExampleService {
> 
>     public void hello() {
>         System.out.println("Hello");
>     }
> 
> }
> 
> @Mock
> private ExampleService exampleService;
> 
> @Test
> public void test() {
> 
>     MockitoAnnotations.initMocks(this);
> 
>     // 这句编译不通过，IDE 也会提示错误，原因很简单，when 的参数是非 void
>     when(exampleService.hello()).thenThrow(new RuntimeException("异常"));
> 
> }
> ```
>
> 下面是正确用法。

用 doThrow 可以让返回void的函数抛出异常，同样也可以让返回非void的函数抛出异常：

```java
@Test
public void test() {

    MockitoAnnotations.openMocks(this);

    // 这种写法可以达到效果
    doThrow(new RuntimeException("异常")).when(exampleService).hello();

    try {
        exampleService.hello();
        Assert.fail();
    } catch (RuntimeException ex) {
        Assert.assertEquals("异常", ex.getMessage());
    }
}
```

#### then&thenAnswer&doAnswer 自定义方法处理逻辑

then 和 thenAnswer 的效果是一样的；它们的参数是实现 Answer 接口的对象，在该对象中可以获取调用参数，自定义返回值。

```java
@Mock
private ExampleService exampleService;

@Test
public void testThenAnswer() {

    MockitoAnnotations.openMocks(this);

    // 这里的话then和thenAnswer的效果是一样的
    when(exampleService.add(anyInt(),anyInt())).thenAnswer(new Answer<Integer>() {
        @Override
        public Integer answer(InvocationOnMock invocation) throws Throwable {
            Object[] args = invocation.getArguments();
            // 获取参数
            Integer a = (Integer) args[0];
            Integer b = (Integer) args[1];

            // 根据第1个参数，返回不同的值
            if (a == 1) {
                return 9;
            }
            if (a == 2) {
                return 99;
            }
            if (a == 3) {
                throw new RuntimeException("异常");
            }
            return 999;
        }
    });

    Assert.assertEquals(9, exampleService.add(1, 100));
    Assert.assertEquals(99, exampleService.add(2, 100));

    try {
        exampleService.add(3, 100);
        Assert.fail();
    } catch (RuntimeException ex) {
        Assert.assertEquals("异常", ex.getMessage());
    }
}

// class ExampleService {
//     int add(int a, int b) {
//         return a+b;
//     }
// }
```

doAnswer 的作用和 thenAnswer 相同，但使用方式不同：

```java
@Test
public void testDoAnswer() {

    MockitoAnnotations.openMocks(this);

    Random random = mock(Random.class);
    doAnswer(new Answer() {
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
            return 1;
        }
    }).when(random).nextInt();

    Assert.assertEquals(1, random.nextInt());

}
```

#### doNothing 让 void 函数什么都不做

doNothing 用于让 void 函数什么都不做；因为 mock 对象中，void 函数就是什么都不做，所以该方法更适合 spy 对象。

```java
@Test
public void testDoNothing() {

    ExampleService exampleService = spy(new ExampleService());
    exampleService.hello();  // 会输出 Hello

    // 让 hello 什么都不做
    doNothing().when(exampleService).hello();
    exampleService.hello(); // 什么都不输出
}
```

### 其他方法

#### reset 重置对象

使用 reset 方法，可以重置之前自定义的返回值和异常：

* **reset mock 对象示例**

```java
@Test
public void testResetMock() {

    ExampleService exampleService = mock(ExampleService.class);

    // mock 对象方法的默认返回值是返回类型的默认值
    Assert.assertEquals(0, exampleService.add(1, 2));

    // 设置让 add(1,2) 返回 100
    when(exampleService.add(1, 2)).thenReturn(100);
    Assert.assertEquals(100, exampleService.add(1, 2));

    // 重置 mock 对象，add(1,2) 返回 0
    reset(exampleService);
    Assert.assertEquals(0, exampleService.add(1, 2));

}
```

* **reset spy 对象示例**

```java
@Test
public void testResetSpy() {

    ExampleService exampleService = spy(new ExampleService());

    // spy 对象方法调用会用真实方法，所以这里返回 3
    Assert.assertEquals(3, exampleService.add(1, 2));

    // 设置让 add(1,2) 返回 100
    when(exampleService.add(1, 2)).thenReturn(100);
    Assert.assertEquals(100, exampleService.add(1, 2));

    // 重置 spy 对象，add(1,2) 返回 3
    reset(exampleService);
    Assert.assertEquals(3, exampleService.add(1, 2));

}
```

#### thenCallRealMethod 调用 spy 对象的真实方法

thenCallRealMethod 可以用来重置 spy 对象的**特定方法特定参数调用**。

```java
@Test
public void test() {

    ExampleService exampleService = spy(new ExampleService());

    // spy 对象方法调用会用真实方法，所以这里返回 3
    Assert.assertEquals(3, exampleService.add(1, 2));

    // 设置让 add(1,2) 返回 100
    when(exampleService.add(1, 2)).thenReturn(100);
    when(exampleService.add(2, 2)).thenReturn(100);
    Assert.assertEquals(100, exampleService.add(1, 2));
    Assert.assertEquals(100, exampleService.add(2, 2));

    // 重置 spy 对象，让 add(1,2) 调用真实方法，返回 3
    when(exampleService.add(1, 2)).thenCallRealMethod();
    Assert.assertEquals(3, exampleService.add(1, 2));

    // add(2, 2) 还是返回 100
    Assert.assertEquals(100, exampleService.add(2, 2));
}
```

#### mockingDetails 判断对象是否为 mock对象、spy 对象

Mockito 的 mockingDetails 方法会返回 MockingDetails 对象，它的 isMock 方法可以判断对象是否为 mock 对象，isSpy 方法可以判断对象是否为 spy 对象。

```java
@Test
public void testMockingDetails() {

    ExampleService exampleService = mock(ExampleService.class);

    // 判断 exampleService 是否为 mock 对象
    System.out.println( mockingDetails(exampleService).isMock() );     // true

    // 判断 exampleService 是否为 spy 对象
    System.out.println( mockingDetails(exampleService).isSpy() );      // false

}
```

### PowerMock 强化 Mockito

PowerMock 是一个增强库，用来增加 Mockito 、EasyMock 等测试库的功能。

Mockito 默认是不支持静态方法，比如我们在 ExampleService 类中定义静态方法 add：

```java
public class ExampleService {

    public static int add(int a, int b) {
        return a+b;
    }

}
```

尝试给静态方法打桩，会报错：

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.junit.MockitoJUnitRunner;

import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class MokitorTest {

    @Test
    public void testMock() {
        // 会报错
        when(ExampleService.add(1, 2)).thenReturn(100);
    }
}
```

**使用 Powermock 弥补 Mockito 缺失的静态方法 mock 功能**，且PowerMockRunner 支持 Mockito 的 @Mock 等注解

> 该部分内容后续再了解吧。

## 参考

https://zhuanlan.zhihu.com/p/101026707

https://www.letianbiji.com/java-mockito/mockito-hello-world.html

https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html

https://github.com/hehonghui/mockito-doc-zh#0