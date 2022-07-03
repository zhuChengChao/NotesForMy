# Java：多线程计数

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 1. CountDownLatch

### 概念

让一些线程阻塞直到另一些线程完成一系列操作才被唤醒

CountDownLatch 主要有两个方法，当一个或多个线程调用 await 方法时，调用线程就会被阻塞。其它线程调用CountDown 方法会将计数器减1（**调用 CountDown 方法的线程不会被阻塞**），当计数器的值变成零时，因调用await 方法被阻塞的线程会被唤醒，继续执行。

### 场景

现在有这样一个场景，假设一个自习室里有7个人，其中有一个是班长，班长的主要职责就是在其它6个同学走了后，关灯，锁教室门，然后走人，因此班长是需要最后一个走的，那么有什么方法能够控制班长这个线程是最后一个执行，而其它线程是随机执行的。

### 解决方案

这个时候就用到了 CountDownLatch，计数器了。我们一共创建6个线程，然后计数器的值也设置成6

```java
// 计数器
CountDownLatch countDownLatch = new CountDownLatch(6);
```

然后每次学生线程执行完，就让计数器的值减1

```java
for (int i = 0; i <= 6; i++) {
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\t 上完自习，离开教室");
        countDownLatch.countDown();
    }, String.valueOf(i)).start();
}
```

最后我们需要通过 CountDownLatch 的 await 方法来控制班长主线程的执行，这里 `countDownLatch.await()` 可以想成是一道墙，只有当计数器的值为0的时候，墙才会消失，主线程才能继续往下执行

```java
countDownLatch.await();

System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
```

不加 CountDownLatch 的执行结果，我们发现 main 线程提前已经执行完成了

```bash
1	 上完自习，离开教室
0	 上完自习，离开教室
main	 班长最后关门
2	 上完自习，离开教室
3	 上完自习，离开教室
4	 上完自习，离开教室
5	 上完自习，离开教室
6	 上完自习，离开教室
```

引入 CountDownLatch 后的执行结果，我们能够控制住 main 方法的执行，这样能够保证前提任务的执行

```bash
0	 上完自习，离开教室
2	 上完自习，离开教室
4	 上完自习，离开教室
1	 上完自习，离开教室
5	 上完自习，离开教室
6	 上完自习，离开教室
3	 上完自习，离开教室
main	 班长最后关门
```

### 完整代码

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        
        // 计数器
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 0; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 上完自习，离开教室");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }

        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
    }
}
```

## 2. CyclicBarrier

### 概念

和 CountDownLatch 相反，需要集齐七颗龙珠，召唤神龙。也就是做加法，开始是0，加到某个值的时候就执行

CyclicBarrier 的字面意思就是**可循环**（cyclic）使用的**屏障**（Barrier）。它要求做的事情是，让**一组线程**到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过 CyclicBarrier 的 await 方法

### 案例

集齐7个龙珠，召唤神龙的 Demo，我们需要首先创建 CyclicBarrier

```java
/**
* 定义一个循环屏障，参数1：需要累加的值，参数2 需要执行的方法
*/
CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
	System.out.println("召唤神龙");
});
```

然后同时编写七个线程，进行龙珠收集，但一个线程收集到了的时候，我们需要让他执行await方法，等待到7个线程全部执行完毕后，我们就执行原来定义好的方法

```java
for (int i = 0; i < 7; i++) {
    final Integer tempInt = i;
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\t 收集到 第" + tempInt + "颗龙珠");
        try {
            // 先到的被阻塞，等全部线程完成后，才能执行方法
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }, String.valueOf(i)).start();
}
```

### 完整代码

```java
/**
 * CyclicBarrier循环屏障
 */
public class CyclicBarrierDemo {
    
    public static void main(String[] args) {
        /**
         * 定义一个循环屏障，参数1：需要累加的值，参数2 需要执行的方法
         */
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("召唤神龙");
        });

        for (int i = 0; i < 7; i++) {
            final Integer tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 收集到 第" + tempInt + "颗龙珠");

                try {
                    // 先到的被阻塞，等全部线程完成后，才能执行方法
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

运行结果：

```bash
1	 收集到 第1颗龙珠
0	 收集到 第0颗龙珠
2	 收集到 第2颗龙珠
3	 收集到 第3颗龙珠
4	 收集到 第4颗龙珠
5	 收集到 第5颗龙珠
6	 收集到 第6颗龙珠
召唤神龙
```

## 3. Semaphore

### 概念

信号量主要用于两个目的

- **一个是用于共享资源的互斥使用**
- **另一个用于并发线程数的控制**

### 代码

我们模拟一个抢车位的场景，假设一共有6个车，3个停车位

那么我们首先需要定义信号量为3，也就是3个停车位

```java
/**
* 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位
*/
Semaphore semaphore = new Semaphore(3, false);
```

然后我们模拟6辆车同时并发抢占停车位，但第一个车辆抢占到停车位后，信号量需要减1

```java
// 代表一辆车，已经占用了该车位
semaphore.acquire(); // 抢占
```

同时车辆假设需要等待3秒后，释放信号量

```java
// 每个车停3秒
try {
	TimeUnit.SECONDS.sleep(3);
} catch (InterruptedException e) {
	e.printStackTrace();
}
```

最后车辆离开，释放信号量

```java
// 释放停车位
semaphore.release();
```

完整代码

```java
/**
 * 信号量Demo
 */
public class SemaphoreDemo {

    public static void main(String[] args) {

        /**
         * 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位
         */
        Semaphore semaphore = new Semaphore(3, false);

        // 模拟6部车
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    // 代表一辆车，已经占用了该车位
                    semaphore.acquire(); // 抢占
                    System.out.println(Thread.currentThread().getName() + "\t 抢到车位");
                    // 每个车停3秒
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t 离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放停车位
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

运行结果：

```bash
0	 抢到车位
2	 抢到车位
1	 抢到车位
2	 离开车位
1	 离开车位
3	 抢到车位
0	 离开车位
4	 抢到车位
5	 抢到车位
4	 离开车位
3	 离开车位
5	 离开车位
```

看运行结果能够发现，0 2 1 车辆首先抢占到了停车位，然后等待3秒后，离开，然后后面 3 4 5 又抢到了车位