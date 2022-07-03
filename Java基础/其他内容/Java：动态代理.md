# Java：动态代理

> 对 Java 中的 **动态代理**，做一个微不足道的小小小小记

## 概述

**动态代理：**当想要给实现了某个接口的类中的方法，加一些额外的处理。比如说加日志，加事务等。可以给这个类创建一个代理，故名思议就是创建一个新的类，这个类不仅包含原来类方法的功能，而且还在原来的基础上添加了额外处理的新功能。**这个代理类并不是定义好的，是动态生成的**。具有解耦意义，灵活，扩展性强。

**动态代理的应用：**Spring 的 AOP、加事务、加权限、加日志。

## 实现

**JDK原生动态代理：动态代理类和被代理类必须继承同一个接口。动态代理只能对接口中声明的方法进行代理。**

> 之所以实现相同的接口，目的是为了尽可能保证代理对象内部结构和目标对象一致，这样我们对代理对象的操作最终都可以转移到目标对象身上，**代理对象只需专注增强代码的编写**。

1. **Proxy**

   `java.lang.reflect.Proxy` 是所有动态代理的父类。它通过静态方法 `newProxyInstance()` 来创建动态代理的 class 对象和实例；

   ```java
   public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler handler)  throws IllegalArgumentException
   
   // loader：一个 ClassLoader 对象，定义了由哪个 ClassLoader 对象来对生成的代理对象进行加载；
   
   // interfaces：一个 Interface 对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了 
   
   // handler：一个 InvocationHandler 对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个 InvocationHandler 对象上。
   
   // 通过 Proxy.newProxyInstance 创建的代理对象是在 Jvm 运行时动态生成的一个对象，它并不是我们的 InvocationHandler 类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象。
   ```

2. **InvocationHandler**

   每一个动态代理实例都有一个关联的InvocationHandler。通过代理实例调用方法，方法调用请求会被转发给InvocationHandler的invoke方法；

   ```java
   Object invoke(Object proxy, Method method, Object[] args) throws Throwable
   // proxy: 指代我们所代理的那个真实对象
   // method: 指代的是我们所要调用真实对象的某个方法的 Method 对象
   // args: 指代的是调用真实对象某个方法时接受的参数
   ```

**举个例子：**

```java
/* 动态代理实现方式小案例：*/

// 定义一个卖电脑的接口：
package cn.itcast.proxy;
public interface SaleComputer {
    public String sale(double money);
    public void show();
}

// 实现该接口，定义一个真实类
package cn.itcast.proxy;

public class Lenovo implements SaleComputer {
    @Override
    public String sale(double money) {
        System.out.println("花了" + money + "元买了一台联想电脑...");
        return "联想电脑";
    }
    @Override
    public void show() {
        System.out.println("展示电脑....");
    }
}

package cn.itcast.proxy;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyTest {

    public static void main(String[] args) {
        // 1.创建真实对象
        Lenovo lenovo = new Lenovo();

        // 2.动态代理增强lenovo对象
        /*
			三个参数：
				1. 类加载器：真实对象.getClass().getClassLoader()
				2. 接口数组：真实对象.getClass().getInterfaces()
				3. 处理器：new InvocationHandler()
      */
        SaleComputer proxy_lenovo = (SaleComputer) Proxy.newProxyInstance(lenovo.getClass().getClassLoader(), lenovo.getClass().getInterfaces(), new InvocationHandler() {
            /*
         	代理逻辑编写的方法：代理对象调用的所有方法都会触发该方法执行
         		参数：
                	1. proxy:代理对象
                	2. method：代理对象调用的方法，被封装为的对象
                	3. args:代理对象调用的方法时，传递的实际参数
            */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                /*
             	 System.out.println("该方法执行了....");
                 System.out.println(method.getName());
                 System.out.println(args[0]);
                */
                //判断是否是sale方法
                if(method.getName().equals("sale")){
                    // 1.增强参数
                    double money = (double) args[0];
                    money = money * 0.85;
                    System.out.println("专车接你....");
                    // 使用真实对象调用该方法
                    String obj = (String) method.invoke(lenovo, money);
                    System.out.println("免费送货...");
                    // 2.增强返回值
                    return obj+"_鼠标垫";
                }else{
                    Object obj = method.invoke(lenovo, args);
                    return obj;
                }
            }
        });

        // 3.调用方法
        String computer = proxy_lenovo.sale(8000);
        System.out.println(computer);
        proxy_lenovo.show();
    }
}
```

## 参考

https://blog.csdn.net/woshichuanqihan/article/details/52205137

bravo1988的回答：https://www.zhihu.com/question/20794107

https://www.cnblogs.com/ynzj123/p/12921288.html