# Java：CompletableFuture

> 工作过程中发现有大哥使用了CompletableFuture类来执行一些操作，之前没遇见过故找了些资料学习一波其适用，顺带做下记录。

## 前言

首先的话CompletableFuture类实现了接口Future：

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
```

显然其出现就是解决了Future在使用上的痛点，即通过Future在获取异步执行结果时，一般通过的是`get()`方法或者是`isDone()`方法，这样就造成了一个主线程的等待，那CompletableFuture的对其进行了一个改善。

## Future

通过Future获取线程执行结果：

```java
// 创建线程池
ExecutorService executor = Executors.newFixedThreadPool(2);

// 创建任务Callable
Callable task = () -> {
    System.out.println(Thread.currentThread() + ": Runnable");
    return "task1";
};

// 提交任务
Future taskFuture = executor.submit(task);

// 阻塞获取：即当任务没执行完成的话会一直阻塞在这里
String res = (String)taskFuture.get();

// 这里肯定是在线程输出完毕之后再进行一个输出
System.out.println("主线程输出：" + res);

// 关闭线程池
executor.shutdown();
```

**Future中常用方法：**

1. `boolean cancel(boolean mayInterruptIfRunning);`

   取消当前任务；

2. `boolean isCancelled();`

   判断当前方法是否取消；

3. `boolean isDone();`

   判断当前方法是否完成

4. `V get() throws InterruptedException, ExecutionException;`

   阻塞式获取任务执行结果；

5. ` get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;`

   阻塞式获取任务执行结果，但到了设定的时间**默认就抛出异常**。

## CompletableFuture

从上述可以发现，仅使用Future的话当想要获取结果时，如果提交的任务还未执行完成，则需要进行等待；

这是 Java8 就引入了类：`java.util.concurrent.CompletableFuture`

它针对`Future`做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。

### 创建任务

**使用`CompletableFuture`执行任务**

* `public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)`
* `public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,                                                   Executor executor)`
* `public static CompletableFuture<Void> runAsync(Runnable runnable)`
* `public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)`

> 对于上述4个方法其区别就是两点：
>
> 1. 是否需要返回值，需要就用supplyAsync，不需要就用runAsync；
>
> 2. 是否加入了自定义的执行器Executor；不加的话就用默认的：
>
>    `private static final Executor asyncPool = useCommonPool ? ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();`

其中关于 Supper 接口：

```java
public interface Supplier<T> {
    T get();
}
```

### 直接获取结果

当通过上述方法将任务提交后，`CompletableFuture`已经被提交给默认的线程池执行了，我们需要定义的是`CompletableFuture`完成时和异常时需要回调的实例。

完成时，`CompletableFuture`会调用`Consumer`对象：

```java
public interface Consumer<T> {
    void accept(T t);
}
```

异常时，`CompletableFuture`会调用`Function`对象：

```java
public interface Function<T, R> {
    R apply(T t);
}
```

可见`CompletableFuture`的优点是：

- 异步任务结束时，会自动回调某个对象的方法；
- 异步任务出错时，会自动回调某个对象的方法；
- 主线程设置好回调后，不再关心异步任务的执行。

**基础使用Demo：**由于提交的任务是异步执行的，因此这里会先输出主线程的内容，然后等任务执行完成后再回调相应的方法。

```java
public static void main(String[] args) throws InterruptedException {

    // 创建异步任务：提供supplier接口
    CompletableFuture<String> cf =
        CompletableFuture.supplyAsync(
        new Supplier<String>() {
            @Override
            public String get() {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "线程执行任务完成";
            }
        }
    );

    // 如果执行成功:提供consumer接口，会回调这部分内容
    cf.thenAccept(
        new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        }
    );

    // 如果执行异常:
    cf.exceptionally(
        new Function<Throwable, String>() {
            @Override
            public String apply(Throwable throwable) {
                throwable.printStackTrace();
                return null;
            }
        }
    );

    // 主线程输出：
    System.out.println("主线程执完毕");
    // 主线程必须等待，防守护线程的退出
    Thread.sleep(500);
}
```

> 注意点：completableFuture这套使用异步任务的操作都是创建成了**守护线程**，因此要防止非守护线程的结束从而导致非守护线程的自动退出。（所有上面主线程中要进行一个等待操作）

### 串行执行任务

如果只是实现了异步回调机制，我们还看不出`CompletableFuture`相比`Future`的优势。`CompletableFuture`更强大的功能是，多个`CompletableFuture`可以串行执行；

例如：

- 定义两个`CompletableFuture`，第一个`CompletableFuture`根据证券名称查询证券代码；
- 第二个`CompletableFuture`根据证券代码查询证券价格，
- 这两个`CompletableFuture`实现串行操作如下：

**实现方式1：**

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 第一个任务:根据证券名称查询证券代码
        CompletableFuture<String> cfQuery = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油");
        });
        
        // cfQuery成功后继续执行下一个任务:根据证券代码查询证券价格
        CompletableFuture<Double> cfFetch = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice(code);
        });
        
        // cfFetch成功后打印结果:
        cfFetch.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(2000);
    }

    static String queryCode(String name) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        return "601857";
    }

    static Double fetchPrice(String code) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        return 5 + Math.random() * 20;
    }
}
```

**实现方式2：**

这里使用方法`public <U> CompletableFuture<U> thenCompose`来实现上述结果，通过thenCompose来串行连接任务；

> 这里还通过 `join()` 来获取最终执行结果。

```java
// 将上面的任务1+任务2合并写
CompletableFuture<Double> futrue = CompletableFuture.supplyAsync(
    () -> queryCode("中国石油")
).thenCompose(code -> CompletableFuture.supplyAsync(
    () -> fetchPrice(code)
));

// 通过Join方式来获取结果
System.out.println("price: " + futrue.join());

// 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
Thread.sleep(2000);
```

**说明：这里可以我们注意下`CompletableFuture`的命名规则：**

- `xxx()`：表示该方法将继续在已有的线程中执行；
- `xxxAsync()`：表示将异步在线程池中执行。

也就是说，上面的串行方法，如果使用的是`thenApply()/thenCompose()`就是两个线程去执行，而如果是`thenApplyAsync()/thenComposeAsync()`还是在同一线程里执行

> 而至于上述thenCompose代码，因为是return了`code -> CompletableFuture.supplyAsync`，在这里创建了新的线程，所以存在等效，进一步说明：
>
> ```java
> CompletableFuture<Double> futrue = CompletableFuture.supplyAsync(
>     () -> queryCode("中国石油")  // 线程1去执行
> ).thenCompose(code -> CompletableFuture.supplyAsync(
>     () -> fetchPrice(code)  // 线程2执行
> ));
> 
> CompletableFuture<Double> futrue = CompletableFuture.supplyAsync(
>     () -> queryCode("中国石油")  // 线程1去执行
> ).thenCompose(code -> {
>     System.out.println("...");  // 线程1去执行
>     return CompletableFuture.supplyAsync(
>         () -> fetchPrice(code));  // 线程2执行
> });
> ```

### 并行执行任务

**方式1：**

除了串行执行外，多个`CompletableFuture`还可以并行执行。

* `CompletableFuture.anyOf()`：可以实现“任意个`CompletableFuture`只要一个成功"
* `CompletableFuture.allOf()`：可以实现“所有`CompletableFuture`都必须成功”

例如，我们考虑这样的场景：同时从新浪和网易查询证券代码，只要任意一个返回结果，就进行下一步查询价格，查询价格也同时从新浪和网易查询，只要任意一个返回结果，就完成操作：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 两个CompletableFuture执行异步查询:
        CompletableFuture<String> cfQueryFromSina = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油", "https://finance.sina.com.cn/code/");
        });
        CompletableFuture<String> cfQueryFrom163 = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油", "https://money.163.com/code/");
        });

        // 用anyOf合并为一个新的CompletableFuture:
        CompletableFuture<Object> cfQuery = CompletableFuture.anyOf(cfQueryFromSina, cfQueryFrom163);

        // 两个CompletableFuture执行异步查询:
        CompletableFuture<Double> cfFetchFromSina = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice((String) code, "https://finance.sina.com.cn/price/");
        });
        CompletableFuture<Double> cfFetchFrom163 = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice((String) code, "https://money.163.com/price/");
        });

        // 用anyOf合并为一个新的CompletableFuture:
        CompletableFuture<Object> cfFetch = CompletableFuture.anyOf(cfFetchFromSina, cfFetchFrom163);

        // 最终结果:
        cfFetch.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(200);
    }

    static String queryCode(String name, String url) {
        System.out.println("query code from " + url + "...");
        try {
            Thread.sleep((long) (Math.random() * 100));
        } catch (InterruptedException e) {
        }
        return "601857";
    }

    static Double fetchPrice(String code, String url) {
        System.out.println("query price from " + url + "...");
        try {
            Thread.sleep((long) (Math.random() * 100));
        } catch (InterruptedException e) {
        }
        return 5 + Math.random() * 20;
    }
}
```

**方式2：**

使用方法`public <U,V> CompletableFuture<V> applyToEither`，实现上述第一个选择功能；

```java
// 两个CompletableFuture执行异步查询:
CompletableFuture<String> queryCode = CompletableFuture.supplyAsync(() -> {
    return queryCode("中国石油", "https://finance.sina.com.cn/code/");
}).applyToEither(CompletableFuture.supplyAsync(() -> {
    return queryCode("中国石油", "https://money.163.com/code/");
}), code -> code);  // 这里的code就是谁先执行完成的返回内容

// 最终结果:
queryCode.thenAccept((code) -> {
    System.out.println("res: " + code);
});
// 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
Thread.sleep(200);
```

**尝试一下都需要执行完成的：**

使用方法`public <U,V> CompletableFuture<V> thenCombine`，将任务融合到一起执行

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "你";
}).thenCombine(CompletableFuture.supplyAsync(() -> {
    return "好";
}), new BiFunction<String, String, String>() {
    @Override
    public String apply(String s1, String s2) {
        return s1 + s2;
    }
});

System.out.println("reuslt: " + future.join());

Thread.sleep(1000);
```

## 总结

`CompletableFuture`可以指定异步处理流程：

- `thenAccept()`处理正常结果；
- `exceptional()`处理异常结果；
- `thenApplyAsync()` / `thenComposeAsync()` 用于串行化另一个`CompletableFuture`；
- `anyOf()` / `allOf()` / `applyToEither()` / ` thenCombine()` 用于并行化多个`CompletableFuture`。

其他方法的话基本上也差不多，按照需求去看官网文档吧。

## 参考

[廖雪峰：使用CompletableFuture](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650#0)

https://www.bilibili.com/video/BV1nA411g7d2

https://blog.csdn.net/finalheart/article/details/87615546