# Java：TransmittableThreadLocal

> 关于：ThreadLocal → InheritableThreadLocal → TransmittableThreadLocal
>
> > TODO：TransmittableThreadLocal不太明白其实现原理，后续有一定了解了再回来补充。

## ThreadLocal

> 关于ThreadLocal的详细介绍可见之前的笔记
>

一言蔽之，ThreadLocal的出现就是为了使得**不同的线程都可以拥有自己专属的本地变量**，即：**ThreadLocal 类主要解决的就是让每个线程绑定自己的值**。

这里可以用一个例子来进行说明：

```java
/**
 * 需求：线程隔离
 *      在多线程并发的场景下，每个线程中的变量都是相互独立
 *      线程A： 设置（变量1）   获取（变量1）
 *      线程B： 设置（变量2）   获取（变量2）
 */
public class Demo01 {

    private String content;

    private String getContent() {
        return content;
    }

    private void setContent(String content) {
        this.content = content;
    }

    public static void main(String[] args) {
        Demo01 demo = new Demo01();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                // 每个线程：存一个变量，过一会儿取出这个变量
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------");
                System.out.println(Thread.currentThread().getName() + "--->" + demo.getContent());
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```

打印结果:

```bash
-----------------------
-----------------------
线程1--->线程1的数据
线程0--->线程1的数据
-----------------------
线程2--->线程3的数据
-----------------------
线程3--->线程3的数据
-----------------------
线程4--->线程4的数据
```

从结果可以看出多个线程在访问同一个变量的时候出现的异常，线程间的数据没有隔离。下面我们来看下采用 ThreadLocal 的方式来解决这个问题的例子。

```java
public class Demo01 {

    // 创建一个 ThreadLocal 对象
    ThreadLocal<String> tl = new ThreadLocal<>();

    // private String content;
    private String getContent() {
        // return content;
        return tl.get();
    }

    private void setContent(String content) {
        // this.content = content;
        // 将变量绑定到ThreadLocal中
        tl.set(content);
    }

    public static void main(String[] args) {
        Demo01 demo = new Demo01();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                // 每个线程：存一个变量，过一会儿取出这个变量
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------");
                System.out.println(Thread.currentThread().getName() + "--->" + demo.getContent());
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```

打印结果：

```bash
-----------------------
线程0--->线程0的数据
-----------------------
线程1--->线程1的数据
-----------------------
线程2--->线程2的数据
-----------------------
线程3--->线程3的数据
-----------------------
线程4--->线程4的数据
```

从结果来看，这样很好的解决了多线程之间数据隔离的问题，十分方便。

## InheritableThreadLocal

### 基本使用

上述借助于 ThreadLocal 很好的解决了多线程之间数据隔离的问题，如下：

> 原先 ThreadLocal 无法传递变量的例子：
>
> ```java
> private static ThreadLocal tl = new ThreadLocal();
> 
> public static void main(String[] args) throws Exception {
> 
>     tl.set("hello");
> 
>     print(Thread.currentThread().getName(), tl.get());
> 
>     new Thread(() -> {
>         print(Thread.currentThread().getName(), tl.get());
>     }).start();
> 
>     Thread.sleep(1000);
>     print(Thread.currentThread().getName(), tl.get());
> }
> 
> private static void print(String threadName, Object value){
>     System.out.println(
>         "thread name: " + threadName +
>         "  tl value: " + value);
> }
> ```
>
> 输出结果：
>
> ```bash
> thread name: main  tl value: hello
> # 可以看到子线程无法获取到父线程中的变量，当然这是正常的
> thread name: Thread-0  tl value: null  
> thread name: main  tl value: hello
> ```

那有没有一种情况？

**子线程需要获取一下父线程的本地变量呢？**这时候就可以借助ThreadLocal的一个子类：InheritableThreadLocal，来实现，只需要将上述的

`private static ThreadLocal tl = new ThreadLocal();`

修改为：`private static ThreadLocal tl = new InheritableThreadLocal();`

输出结果：

```bash
thread name: main  tl value: hello
# 可以看到这次子线程获取到了父线程的本地变量
thread name: Thread-0  tl value: hello  
thread name: main  tl value: hello
```

且子线程可以修改父线程中的本地变量：

```java
// private static ThreadLocal tl = new InheritableThreadLocal();

User user = new User("zhangsan", "male");
tl.set(user);

print(Thread.currentThread().getName(), tl.get());

new Thread(() -> {
    User tmpUser = (User)tl.get();
    tmpUser.setName("lisi");
    print(Thread.currentThread().getName(), tl.get());
}).start();

Thread.sleep(1000);
print(Thread.currentThread().getName(), tl.get());
```

输出：

```java
thread name: main  tl value: User{name='zhangsan', sex='male', address=null}
thread name: Thread-0  tl value: User{name='lisi', sex='male', address=null}
// 可以看到主线中的本地变量被子线程给修改掉了
thread name: main  tl value: User{name='lisi', sex='male', address=null}
```

### 源码浅析

**分析1：首先了解inheritableThreadLocals，基于此实现了子线程能获取到父线程的本地变量**

```java
// 继承自ThreadLocal，重写了其中的三个方法 childValue & getMap & createMap
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
	
    // 这个方法在ThreadLocal中直接抛的异常
    protected T childValue(T parentValue) {
        return parentValue;
    }

    // 这个方法在ThreadLocal中返回的是threadLocals
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }
	
    // 这个方法在ThreadLocal中执行的是t.threadLocals = new ThreadLocalMap(..)
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

**分析2：查看 Thread 类，了解一下InheritableThreadLocal是如何实现父线程将本地值传入到子线程中的**

> 总结：主线程创建类InheritableThreadLocal，通过set方式往其中放入本地变量；然后在主线程中创建子线程时，在子线程初始化时会将主线程中InheritableThreadLocal存在的本地变量放入到子线程的InheritableThreadLocal中。

1. 通过InheritableThreadLocal的set方法进行赋值；

```java
//  ThreadLocal中的set方法
public void set(T value) {
    Thread t = Thread.currentThread();
    // 从上述可知，InheritableThreadLocal重写了getMap方法，这里就返回的是inheritableThreadLocal
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // 有值就返回
        map.set(this, value);  
    else
        // 没值创建，在InheritableThreadLocal重写了createMap方法，
        // 即创建了ThreadLocal后赋值给t.inheritableThreadLocals
        createMap(t, value);  
}
```

2. 在父线程中初始化子线程时传入，即在Thread初始化init方法中，将父线程的本地变量放入到子线程中，**从而实现了一个传递；**

```java
public class Thread implements Runnable {
    
    // ...
    
    // ThreadLocal使用的Map
    ThreadLocal.ThreadLocalMap threadLocals = null;

    // InheritableThreadLocal使用的Map，关注这个
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    
    // ...
    
    // 初始化线程方法：Initializes a Thread.
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
		
        // 线程初始化时，我的理解是这里应该是父线程里去初始化子线程，所以这里的线程应该是父线程
        Thread parent = currentThread();
		
        // ....
        
        // 当父线程存在inheritableThreadLocals时，则在子线程init时
        // 将父线程的inheritableThreadLocals的内容赋值给子线程
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            // 利用父线程的inheritableThreadLocals进行inheritableThreadLocals的创建，实现值传递
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        // ...
    }
    
}
```

> 对于上述的`ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);`方法：
>
> ```java
> /**
>  * Factory method to create map of inherited thread locals.
>  * Designed to be called only from Thread constructor.
>  *
>  * @param  parentMap the map associated with parent thread
>  * @return a map containing the parent's inheritable bindings
>  */
> static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
>     // 初始化ThreadLocal.ThreadLocalMap对象
>     return new ThreadLocalMap(parentMap);
> }
> 
> /**
>  * Construct a new map including all Inheritable ThreadLocals
>  * from given parent map. Called only by createInheritedMap.
>  *
>  * @param parentMap the map associated with parent thread.
>  */
> private ThreadLocalMap(ThreadLocalMap parentMap) {
>     // 取父线程里的哈希表
>     Entry[] parentTable = parentMap.table;
>     int len = parentTable.length;
>     setThreshold(len);
>     table = new Entry[len];
> 
>     for (int j = 0; j < len; j++) {
>         // 遍历父hash表中的元素
>         Entry e = parentTable[j];
>         if (e != null) {
>             @SuppressWarnings("unchecked")
>             // 取key，key当然就是ThreadLocal
>             ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
>             if (key != null) {
>                 // 取value，在InheritableThreadLocal中对其进行了重写
>                 Object value = key.childValue(e.value);
>                 Entry c = new Entry(key, value);
>                 int h = key.threadLocalHashCode & (len - 1);
>                 while (table[h] != null)  // 注意，这里没有用链式结构而是寻找下一个为null的槽位
>                     h = nextIndex(h, len);
>                 // 把值放入，这里直接用父hash表中的数据放入了子哈希表，即引用的对象相同
>                 table[h] = c;
>                 size++;
>             }
>         }
>     }
> }
> ```
>
> > 可以看到这里这里的放入是直接那了主线程的本地变量放入到子线程中，即在子线程中修改也同时会影响到主线程。

**分析3：再查看类inheritableThreadLocals：子线程如何获取到父线程的本地变量**

当子线程在获取本地变量时：

```java
// 父线程定义：private static ThreadLocal tl = new InheritableThreadLocal();
// 父线程设置：tl.set("hello");

// 子线程中获取值：tl.get()
// 在类ThreadLocal中：
public T get() {
    Thread t = Thread.currentThread();
    // 用到了上述InheritableThreadLocal重写的getMap方法，即返会了当前线程（子线程）的inheritableThreadLocals
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

### 存在问题

**线程不安全：**

由于 ThreadLocal 的出现本来就是为了实现线程之间变量的隔离，而 InheritableThreadLocal 的引入会导致子线程可以修改父线程中的引用类型，上面的基本使用中就体现了。

**线程池失效：**

按照上述实现，在使用线程池的时候，InheritableThreadLocal 存在失效的情况，**因为父线程的InheritableThreadLocalMap是通过init一个Thread的时候进行赋值给子线程的**，而线程池在执行异步任务时可能不再需要创建新的线程了，因此也就不会再传递父线程的InheritableThreadLocalMap给子线程了。

代码说明：

```java
// 线程池中仅有一个核心线程
private static ExecutorService executorService = Executors.newFixedThreadPool(1);
// 来一个InheritableThreadLocal
private static ThreadLocal tl = new InheritableThreadLocal<>();

public static void main(String[] args) {

    print(Thread.currentThread().getName(), tl.get());
	
    // 先让主线程去创建一个线程放入到线程池中，这时线程在init时，由于主线程的InheritableThreadLocal没内容，因此创建的线程不会拿着主线程的本地变量
    executorService.execute(()->{
        print(Thread.currentThread().getName(), tl.get());
    });
	
    // 往主线程中放入本地成员变量
    tl.set(1);
	
    // 这时创建的子线程是可以拿到主线程中的本地成员变量的
    new Thread(() -> {
        print(Thread.currentThread().getName(), tl.get());
    }).start();
	
    // 但线程池就不行了，因为这个线程不是新创建的，而是被复用的原因
    executorService.execute(()->{
        print(Thread.currentThread().getName(), tl.get());
    });

    print(Thread.currentThread().getName(), tl.get());

}

private static void print(String threadName, Object value){
    System.out.println(
        "thread name: " + threadName +
        "  tl value: " + value);
}
```

输出结果：

```bash
thread name: main  tl value: null
thread name: pool-1-thread-1  tl value: null
thread name: Thread-0  tl value: 1
thread name: main  tl value: 1
# 可以看到，线程池中没拿到主线程的本地变量，变量传递失败。
thread name: pool-1-thread-1  tl value: null
```

## TransmittableThreadLocal

针对上述 InheritableThreadLocal 存在的问题，阿里实现了[transmittable-thread-local](https://github.com/alibaba/transmittable-thread-local)（TTL）来解决线程池异步值传递问题。

贴一个官网文档的User Guide：

![UserGuide](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603130.PNG)

### 使用指导

> 这部分内容全部摘自：https://github.com/alibaba/transmittable-thread-local#-user-guide

#### 简单使用

**父线程给子线程传递值：**

```java
TransmittableThreadLocal<String> context = new TransmittableThreadLocal<>();

// 在父线程中设置
context.set("value-set-in-parent");
System.out.println(Thread.currentThread().getName() + ":" +context.get());

// 在子线程中可以读取，值是"value-set-in-parent"
new Thread(()->{
    System.out.println(Thread.currentThread().getName() + ":" +context.get());
}).start();
```

> 这其实是`InheritableThreadLocal`的功能，应该使用`InheritableThreadLocal`来完成。
>
> 但对于使用线程池等会池化复用线程的执行组件的情况，线程由线程池创建好，并且线程是池化起来反复使用的；这时父子线程关系的`ThreadLocal`值传递已经没有意义，应用需要的实际上是把 **任务提交给线程池时**的`ThreadLocal`值传递到 **任务执行时**。

#### 结合线程池

**修饰`Runnable`和`Callable`**

```java
TransmittableThreadLocal<String> context = new TransmittableThreadLocal<>();
ExecutorService executor = Executors.newCachedThreadPool();

// 在父线程中设置
context.set("value-set-in-parent");
System.out.println(Thread.currentThread().getName() + ":" +context.get());

Runnable task = () -> System.out.println(Thread.currentThread().getName() + ":" +context.get());
// 额外的处理，生成修饰了的对象ttlRunnable
Runnable ttlRunnable = TtlRunnable.get(task);
executor.submit(ttlRunnable);
```

> **注意**：
> 即使是同一个`Runnable`任务多次提交到线程池时，每次提交时都需要通过修饰操作（即`TtlRunnable.get(task)`）以抓取这次提交时的`TransmittableThreadLocal`上下文的值；即如果同一个任务下一次提交时不执行修饰而仍然使用上一次的`TtlRunnable`，则提交的任务运行时会是之前修饰操作所抓取的上下文。示例代码如下：
>
> ```java
> TransmittableThreadLocal<String> context = new TransmittableThreadLocal<>();
> ExecutorService executor = Executors.newCachedThreadPool();
> // 在父线程中设置
> context.set("value-set-in-parent");
> 
> // 第一次提交
> Runnable task = () -> System.out.println(Thread.currentThread().getName() + ":" +context.get());
> executor.submit(TtlRunnable.get(task));
> 
> // ...业务逻辑代码，
> // 并且修改了 TransmittableThreadLocal上下文 ...
> context.set("value-modified-in-parent");
> 
> // 再次提交
> // 重新执行修饰，以传递修改了的 TransmittableThreadLocal上下文
> executor.submit(TtlRunnable.get(task));
> ```

上面演示了`Runnable`，`Callable`的处理类似：

```java
TransmittableThreadLocal<String> context = new TransmittableThreadLocal<>();
ExecutorService executor = Executors.newCachedThreadPool();

// 在父线程中设置
context.set("value-set-in-parent");
System.out.println(Thread.currentThread().getName() + ":" +context.get());

Callable call = () -> {
    System.out.println(Thread.currentThread().getName() + ":" +context.get());
    return context.get();
};
// 额外的处理，生成修饰了的对象ttlRunnable
Callable ttlCallable = TtlCallable.get(call);
executor.submit(ttlCallable);
```

#### 整个过程的完整时序图

![TransmittableThreadLocal-sequence-diagram](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251603833.png)

#### 修饰线程池

省去每次`Runnable`和`Callable`传入线程池时的修饰，这个逻辑可以在线程池中完成。

通过工具类`com.alibaba.ttl.threadpool.TtlExecutors`完成，有下面的方法：

- `getTtlExecutor`：修饰接口`Executor`
- `getTtlExecutorService`：修饰接口`ExecutorService`
- `getTtlScheduledExecutorService`：修饰接口`ScheduledExecutorService`

示例代码：

```java
ExecutorService executor = Executors.newCachedThreadPool();
executor = TtlExecutors.getTtlExecutorService(executor);

TransmittableThreadLocal<String> context = new TransmittableThreadLocal<>();

// 在父线程中设置
context.set("value-set-in-parent");
System.out.println(Thread.currentThread().getName() + ":" +context.get());

Runnable task = () ->
    System.out.println(Thread.currentThread().getName() + ":" +context.get());
Callable call = () -> {
    System.out.println(Thread.currentThread().getName() + ":" +context.get());
    return context.get();
};

// 额外的处理，生成修饰了的对象ttlRunnable
// Runnable ttlRunnable = TtlRunnable.get(task);
// Callable ttlCallable = TtlCallable.get(call);

// 这里不需要上述的额外处理了
executor.submit(task);
executor.submit(call);
```

#### ~~使用`Java Agent`来修饰`JDK`线程池实现类~~

> 略。

### ~~实现浅析~~

从InheritableThreadLocal不支持线程池的根本原因是InheritableThreadLocal是在父线程创建子线程时复制的，由于线程池的复用机制，“子线程”只会复制一次。

要支持线程池中能访问提交任务线程的本地变量，其实**只需要在父线程向线程池提交任务时复制父线程的上下环境，那在子线程中就能够如愿访问到父线程中的本地变量**，实现本地环境变量在线程池调用中的透传，这也就是TransmittableThreadLocal最本质的实现原理。

> 内功不够，后续再了解 :angry::angry::angry:

## 参考

https://github.com/alibaba/transmittable-thread-local#-user-guide

https://www.cnblogs.com/hama1993/p/10400265.html

https://zhuanlan.zhihu.com/p/146124826

https://www.cnblogs.com/hama1993/p/10409740.html#!comments

https://mp.weixin.qq.com/s?__biz=MzIzNzgyMjYxOQ==&mid=2247484380&idx=1&sn=ff2e2aaf9cfc63ae60cedd07dea26733