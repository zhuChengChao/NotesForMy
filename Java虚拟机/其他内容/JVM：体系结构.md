# JVM：体系结构

> 本笔记是根据bilibili上 [尚硅谷](https://space.bilibili.com/302417610) 的课程 [Java大厂面试题第二季](https://www.bilibili.com/video/BV18b411M7xz?spm_id_from=333.788.b_636f6d6d656e74.29) 而做的学习笔记；因为最近也打算找下暑期实习...就针对性的学习一下:grimacing:
>

## 概览

![java程序内存结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633614.png)

Java GC 主要回收的是 **方法区** 和 **堆** 中的内容

![gc主要作用区域](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633475.png)

## 类加载器

- 类加载器是什么
- 双亲委派机制
- Java类加载的沙箱安全机制

## 常见的垃圾回收算法

### 引用计数

> 这是不是垃圾回收算法吧？？？:dog:

![引用计数](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633189.png)

在双端循环，互相引用的时候，容易报错，目前很少使用这种方式了

### 复制

复制算法在年轻代的时候，进行使用，复制时候有交换

![复制算法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633037.png)

**优点：没有产生内存碎片**

### 标记清除

先标记，后清除，缺点是会产生内存碎片，用于老年代多一些

![标记清除](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633762.png)

### 标记整理

标记清除+整理

![标记整理](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251633662.png)

**但是需要付出一定的代价，因为移动对象需要成本**