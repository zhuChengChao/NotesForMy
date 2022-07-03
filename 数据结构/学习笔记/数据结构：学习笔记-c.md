# 数据结构：学习笔记-c

> 虽然之前已经是学了一波数据结构了，过程中突然又发现了一个视频教程，就顺带给看了，算是相互参考吧，并也做了一波笔记，总共分了 5 篇笔记，此为 3/5 篇。

## 1. 课程目标

1. 了解散列表的定义
2. 掌握散列表的要求和特点
3. 理解散列函数的设计方法
4. 掌握散列冲突的解决方案
5. 掌握散列表的应用：HashMap 源码分析
6. 掌握哈希算法的应用场景
7. 理解树的定义
8. 掌握二叉树的定义及遍历方式
9. 掌握二叉查找树的插入，删除，查询等相关操作
10. 掌握 AVL 树的特点及实现

## 2. 散列表

### 2.1 散列表概述

#### 2.1.1 散列表概念

散列表(Hash Table)又名哈希表 / Hash 表，是根据键（Key）直接访问在内存存储位置的数据结构，它是由数组演化而来的，利用了数组支持按照下标进行随机访问数据的特性；接下来我们以一个具体的例子来说明散列的思想及原理。

![散列表](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106925.PNG)

### 2.2 散列函数

#### 2.2.1 散列函数的要求及特点

散列函数就是一个函数(方法)，能够将给定的 key 转换为特定的散列值，我们可以表示为：hashValue = hash(key)

散列函数要满足的几个基本要求：

**1：散列函数计算得到的散列值必须是大于等于0的正整数，因为hash值需要作为数组的下标。**

**2：如果key1==key2，那么经过hash后得到的哈希值也必相同即：hash(key1)== hash(key2)** 

**3：如果key1 != key2，那么经过hash后得到的哈希值也必不相同即：hash(key1) != hash(key2)**

好的散列函数应该满足以下特点：

**1：散列函数不能太复杂，因为太复杂度势必要消耗很多的时间在计算哈希值上，也会间接影响散列表性能。**

**2：散列函数计算得出的哈希值尽可能的能随机并且均匀的分布，这样能够将散列冲突最小化。**

#### 2.2.2 散列函数的设计方法

实际工作中，我们还需要综合考虑各种因素。这些因素有关键字的长度、特点、分布、还有散列表的大小等。散列函数各式各样，下方举几个常用的、简单的散列函数的设计方法。

##### **2.2.2.1 直接寻址法：**

比如我们现在要对 0-100 岁的人口数字统计表，那么我们对年龄这个关键字 key就可以直接用年龄的数字作为地址。此时 hash(key) = key。这个时候，我们可以得出这么个哈希函数：hash(0) = 0，hash(1) = 1，……，hash(20) = 20。

| 地址 | 年龄 | 人数  |
| ---- | ---- | ----- |
| 00   | 0    | 500w  |
| 01   | 1    | 600w  |
| 02   | 2    | 450w  |
| ...  | ...  | ...   |
| 20   | 20   | 1500w |
| ...  | ...  | ...   |

如果我们现在要统计的是1980 年后出生年份的人口数，那么我们对出生年份这个关键字可以用年份减去 1980 来作为地址。此时 hash(key) =key-1980

| 地址 | 出生年份 | 人数  |
| ---- | -------- | ----- |
| 00   | 1980     | 1500w |
| 01   | 1981     | 1600w |
| 02   | 1982     | 1300w |
| ...  | ...      | ...   |
| 20   | 2000     | 500w  |
| ...  | ...      | ...   |

也就是说，我们可以**取关键字key的某个线性函数值为散列地址，即：**

**hash(key) = a x key + b，其中a,b为常量**

这样的散列函数优点就是简单、均匀，也不会产生冲突，但问题是这需要事先知道关键字 key 的分布情况，适合査找表较小且连续的情况。由于这样的限制，在现实应用中，直接寻址法虽然简单，但却并不常用。

##### **2.2.2.2 除留余数法**

除留余数法此方法为最常用的构造散列函数方法。**对于散列表长为m的散列函数公式为：**

**hash( key ) = key mod p ( p ≤ m )**

本方法的关键就在于选择合适的 p, p 如果选得不好，就可能会容易产生哈希冲突，比如：有 12 个关键字 key，现在我们要针对它设计一个散列表。如果采用除留余数法，那么可以先尝试将散列函数设计为 hash(key) = key mod 12 的方法。比如29 mod 12 = 5，所以它存储在下标为 5 的位置。

| 下标   | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 关键字 | 12   | 25   | 38   | 15   | 16   | 29   | 78   | 67   | 56   | 21   | 22   | 47   |

不过这也是存在冲突的可能的，因为 12 = 2×6 = 3×4。如果关键字中有像 18(3×6)、30(5×6)、42(7×6)等数字，它们的余数都为 6，这就和 78 所对应的下标位置冲突了。此时如果我们不选用 p=12 而是选用 p=11 则结果如下：

| 下标   | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 0    | 1    |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 关键字 | 12   | 24   | 36   | 48   | 60   | 72   | 84   | 86   | 108  | 120  | 132  | 144  |

使用除留余数法的一个经验是，若散列表表长为 m，通常 p 为小于或等于表长（最好接近 m）的最大质数或不包含小于 20 质因子的合数。总之实践结果证明：**当P取小于哈希表长的最大质数时，产生的哈希函数较好**。

##### **2.2.2.3 平方取中法**

这是一种常用的哈希函数构造方法。这个方法是先取关键字的平方，然后根据可使用空间的大小，选取平方数是中间几位为哈希地址。

**hash(key) = key平方的中间几位**

这种方法的原理是通过取平方扩大差别，平方值的中间几位和这个数的每一位都相关，则对不同的关键字得到的哈希函数值不易产生冲突，由此产生的哈希地址也较为均匀。

| 关键字 | 关键字的平方 | 哈希函数值 |
| ------ | ------------ | ---------- |
| 1234   | 1,522,756    | 227        |
| 2143   | 4,592,449    | 924        |
| 4132   | 17,073,424   | 734        |
| 3214   | 10,329,796   | 297        |

##### **2.2.2.4 折叠法**

有时关键码所含的位数很多，采用平方取中法计算太复杂，则可将关键码分割成位数相同的几部分（最后一部分的位数可以不同），然后取这几部分的叠加和（舍去进位）作为散列地址，这方法称为折叠法，折叠法可分为两种：

移位叠加：将分割后的几部分低位对齐相加。

边界叠加：从一端沿分割界来回折叠，然后对齐相加。

比如关键字为:12320324111220，分为 5 段，123，203，241，112，20，两种方式如下：

移位叠加：123+203+241+112+20=879

边界叠加：321+302+142+211+02=978

当然了散列函数的设计方法不仅仅只有这些方法，对于这些方法我们无需全部掌握也不需要死记硬背，我们理解其设计原理即可。

#### 2.2.3 散列冲突

两个不同的关键字（**key**），由于散列函数值相同，因而被映射到同一表位置上。该现象称为散列冲突或哈希碰撞。

#### 2.2.4 散列冲突的解决方案

前面我们讲到即使再好的散列函数我们也无法避免不了散列冲突(哈希冲突，哈希碰撞)，那如果真的出现了散列冲突我们应该如何来解决散列冲突呢？在本节中我们来介绍**两类方法解决散列冲突：开放寻址法，链表法**

##### **2.2.4.1 开放寻址法**

开放寻址法的核心思想是：一旦出现了散列冲突，我们就重新去寻址一个空的散列地址。

1. **线性检测**

   我们往散列表中插入数据时，如果某个数据经过散列函数散列之后，存储位置已经被占用了，我们就从当前位置开始，依次往后查找，看是否有空闲位置，直到找到为止。

   ![线性检测](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106273.PNG)

   如：散列表的大小为 7，在元素 X 插入之前已经有 a，b，c，d 四个元素插入到散列表中了，元素 X 经过 hash(X)计算之后得到的哈希值为 4，但是 4 这个位置已经有数据了，所以产生了冲突，于是我们需要按照顺序依次向后查找，一直查找到数组的尾部都没有空闲位置了，所以再从头开始查找，直到找到空闲位置下标为 1 的位置，至此将元素 X 插入下标为 1 的位置。

   我们刚刚所讲的是向散列表中插入数据，如果要从散列表中查找是否存在某个元素，这个过程跟插入类似，先根据散列函数求出要查找元素的 key 的散列值，然后比较数组中下标为其散列值的元素和要查找的元素，如果相等则表明该元素就是我们想要的元素，如果不等还要继续向后寻找遍历，如果遍历到数组中的空闲位置还没有找到则说明我们要找的元素并不在散列表中。

   当然了散列表跟数组一样，不仅支持插入、查找操作，还支持删除操作。其中删除操作稍微有点特殊，删除操作不能简单的将要删除的位置设置为空，为什么呢？

   上面刚讲到从散列表中查找是否存在某个元素一旦在对应 hash 值下标下的元素不是我们想要的就会继续在散列表中向后遍历，直到找到数组中的空闲位置，但如果这个空闲位置是我们刚刚删除的，那就会中断向后查找的过程，那这样的话查找的算法就会失效，本来应该认定为存在的元素会被认定为不存在，那删除的问题如何解决呢？我们可以将删除的元素特殊标记为 deleted，当线性检测遇到标记 deleted的时候并不停下来而是继续向后检测，如下图所示：

   ![线性检测2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106141.PNG)

   使用线性检测的方式存在很大的问题：那就是当散列表中的数据越来越多的时候，散列冲突发生的可能性就越来越大，空闲的位置越来越少，那线性检测的时间就会越来越长，在极端情况下我们可能需要遍历整个数组，所以最坏的情况下时间复杂度为 O(n)，因此对于开放寻址解决冲突还有另外两种比较经典的的检测方式：**二次检测，双重散列**。

2. **二次检测**

   所谓的二次检测跟线性检测的原理一样，只不过线性检测每次检测的步长是 1，每次检测的下标依次是： 
   $$
   hash(key)+0，hash(key)+1，hash(key)+2，hash(key)+3，...，
   $$
   所谓的二次检测指的是每次检测的步长变为原来的二次方，即每次检测的下标为：
   $$
   hash(key)+0，hash(key)+1^2，hash(key)+2^2，hash(key)+3^2，...，
   $$

3. **双重散列**

   所谓的双重散列，意思就是不仅要使用一个散列函数。我们使用一组散列函数hash1(key)，hash2(key)，hash3(key)…我们先用第一个散列函数，如果计算得到的存储位置已经被占用，再用第二个散列函数，依次类推，直到找到空闲的存储位置。

**装载因子：**

总之不管采用哪种探测方法，当散列表中空闲位置不多的时候，散列冲突的概率就会大大提高。为了尽可能保证散列表的操作效率，一般情况下，我们会尽可能保证散列表中有一定比例的空闲位置。我们用**装载因子(load factor)**来表示空位的多少。散列表装载因子的计算公式为：

**装载因子 = 散列表中元素的个数/散列表的长度**

装载因子越大，说明空闲位置越少，冲突越多，散列表的性能会下降。那**如果装载因子过大了怎么办**？装载因子过大不仅插入的过程中要多次寻址，查找的过程也会变得很慢。当装载因子过大时，进行**动态扩容**，重新申请一个更大的散列表，将数据搬移到这个新散列表中。假设每次扩容我们都申请一个原来散列表大小两倍的空间。如果原来散列表的装载因子是 0.8，那经过扩容之后，新散列表的装载因子就下降为原来的一半，变成了 0.4。针对数组的扩容，数据搬移操作比较简单。但是，针对散列表的扩容，数据搬移操作要复杂很多。因为散列表的大小变了，数据的存储位置也变了，所以我们需要通过散列函数重新计算每个数据的存储位置。

插入一个数据，最好情况下，不需要扩容，最好时间复杂度是 O(1)。最坏情况下，散列表装载因子过高，启动扩容，我们需要重新申请内存空间，重新计算哈希位置，并且搬移数据，所以时间复杂度是 O(n)。但是这个动态扩容的过程在 n 次操作中会遇见一次，因此平均下来时间复杂度接近最好情况，就是 O(1)。

当散列表的装载因子超过某个阈值时，就需要进行扩容。装载因子阈值需要选择得当。如果太大，会导致冲突过多；如果太小，会导致内存浪费严重。装载因子阈值的设置要权衡时间、空间复杂度。如果内存空间不紧张，对执行效率要求很高，可以降低负载因子的阈值；相反，如果内存空间紧张，对执行效率要求又不高，可以增加负载因子的值，甚至可以大于 1。

**总结一下，当数据量比较小、装载因子小的时候，适合采用开放寻址法**。这也是 Java 中的 ThreadLocalMap 使用开放寻址法解决散列冲突的原因。

##### **2.2.4.2 链表法**

相比开放寻址法，它要简单很多。我们来看这个图，在散列表中，数组的每个下标位置我们可以称之为“桶（bucket）”或者“槽（slot）”，每个桶(槽)会对应一条链表，所有散列值相同的元素我们都放到相同槽位对应的链表中。

![链表法](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106728.PNG)

**基于链表的散列冲突处理方法比较适合存储大对象、大数据量的散列表，而且，比起开放寻址法，它更加灵活，支持更多的优化策略，比如用红黑树代替链表。**

##### 2.2.4.3 **总结**

综合本章节所学的知识点，我们知道作为一个企业级的散列表应该具有如下特点：

* **支持快速的查询、插入、删除操作；**
* **内存占用合理，不能浪费过多的内存空间；**
* **性能稳定，极端情况下，散列表的性能也不会退化到无法接受的情况。**

我们要实现这样一个散列表应该从如下几个方面来考虑设计思路：

* **设计一个合适的散列函数；**

* **定义装载因子阈值，并且设计动态扩容策略；**

* **选择合适的散列冲突解决方法。**

### 2.3 散列表的应用

HashMap 的数据结构图如下图所示：

![HashMap](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106492.PNG)

**jdk1.8 中关于 HashMap 的实现跟 jdk1.7 的几点差别：**

**1：数据结构引入了红黑树，好处是可以提高查询效率(jdk1.7中极端情况下查询是O(n)，如果引入红黑树在极端情况下的查询可以降低为O(log n))，当散列表某一桶内链表节点数>=8时链表树化成红黑树，红黑树太小时退化成链表，退化的阈值为6**

**2：计算key的hash值的方式不一样，但是思路和原理一样都是对key的hashCode进行扰动让高位和低位一起参与运算计算出更加均匀的hash码，降低hash冲突的概率。**

**3：插入数据时如果发送了hash冲突，优先判断该位置上是否是红黑树，如果是则存入红黑树中，如果是链表则插入链表尾节点上(jdk1.7是插入到链表头节点上)，插入完成后还判断链表的节点数是否大于等于设定好的链表转红黑树的阈值，如果满足则将链表转换为红黑树。**

**4：两个版本都会产生扩容操作，只不过jdk1.8中扩容涉及到对红黑树的操作以及优化了在hash冲突时计算元素新下标的代码，使其非常简单高效！**

### 2.4 哈希算法

#### 2.4.1 定义

哈希算法又称为摘要算法，它可以将任意数据通过一个函数转换成长度固定的数据串，这个映射转换的规则就是哈希算法，而通过原始数据映射之后得到的二进制值串就是哈希值。 **可见，摘要算法就是通过摘要函数f()对任意长度的数据data计算出固定长度的摘要digest，目的是为了发现原始数据是否被人篡改过。**

 摘要算法之所以能指出数据是否被篡改过，就是因为摘要函数是一个**单向函数**，计算 f(data)很容易，但通过 digest 反推 data 却非常困难。而且，对原始数据做一个 bit 的修改，都会导致计算出的摘要完全不同。

那有没有可能两个不同的数据通过某个摘要算法得到了相同的摘要呢？完全有可能！因为任何摘要算法都是把无限多的数据集合映射到一个**有限的集合**中。这种情况就是我们说的**碰撞**。

#### 2.4.2 要求

我们要想设计出一个优秀的哈希算法并不是很容易，一个优秀的哈希算法一般要满足如下几点要求：

1. **将任何一条不论长短的信息，计算出唯一的一摘要(哈希值)与它相对应，对输入数据非常敏感，哪怕原始数据只修改了一个Bit，最后得到的哈希值也大不相同**
2.         **摘要的长度必须固定，散列冲突的概率要很小，对于不同的原始数据，哈希值相同的概率非常小**
3.         **摘要不可能再被反向破译。也就是说，我们只能把原始的信息转化为摘要，而不可能将摘要反推回去得到原始信息，即哈希算法是单向的**
4.          **哈希算法的执行效率要尽量高效，针对较长的文本，也能快速地计算出哈希值**

这些要求都是比较理论的说法，我们那一种企业常用的哈希算法 MD5 来说明：现使用 MD5 对三段数据分别进行哈希求值：

1. MD5('数据结构和算法') = 31ea1cbbe72095c3ed783574d73d921e

2. MD5('数据结构和算法很好学')=0fba5153bc8b7bd51b1de100d5b66b0a 

3. MD5('数据结构和算法不好学')=85161186abb0bb20f1ca90edf3843c72

从其结果我们可以看出：MD5 的摘要值(哈希值)是固定长度的，是 16 进制的 32 位即128 Bit 位，无论要哈希的数据有多长，多短，哈希之后的数据长度是固定的，另外哈希值是随机的无规律的，无法根据哈希值反向推算文本信息，其次 2，3 表明尽管只有一字之差得到的结果也是千差万别，最后哈希的速度和效率是非常高的，这一点我们可能还体会不到，因为我们哈希的只是很短的一串数据，即便我们哈希的是整个这段文本，用 MD5 计算哈希值，速度也是非常的快，总之 MD5 基本满足了我们前面所讲解的这几个要求。

### 2.5 总结

在本章节中我们学习了散列表数据结构，掌握了散列函数的特点及设计要求，明确了其中几种设计方案，知道了散列冲突的原理以及解决散列冲突的方案，对于散列表在企业中的应用我们重点分析了 HashMap 和 HashTable 的源码，最后我们介绍了哈希算法，重点是阐述了哈希算法的应用场景。

## 3. 树

在前面章节的中我们学习了线性表数据结构：数组，链表，栈，队列；在这章中我们来学习一种非线性表叫做：树。我们先来看树的定义及相关概念

### 3.1 树的定义及相关概念

#### 3.1.1 树的定义

树在维基百科中的定义为：树（英语：Tree）是一种无向图（undirected graph），其中任意两个顶点间存在唯一一条路径。或者说，只要没有回路的连通图就是树。在计算机科学中，**树**（英语：tree）是一种抽象数据类型（Abstract Data Type）或是实现这种抽象数据类型的数据结构，用来模拟具有树状结构性质的数据集合。它是由 n（n>0）个有限节点组成一个具有层次关系的集合。把它叫做“树”是因为它看起来像一棵倒挂的树，也就是说它是根朝上，而叶朝下的。它具有以下的特点：

* 每个节点都只有有限个子节点或无子节点；

* 没有父节点的节点称为根节点；

* 每一个非根节点有且只有一个父节点；

* 除了根节点外，每个子节点可以分为多个不相交的子树；

* 树里面没有环路(cycle)

![树形结构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106183.PNG)

以上这些都是树，下面我们再看几个**不是树**的情况

![不是树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106850.PNG)

“树”这种数据结构真的很像我们现实生活中的“树”，这里面每个元素我们叫作“节点”；用来连线相邻节点之间的关系，我们叫作“父子关系”。

比如在下方这副图中：

![树2.0](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106401.PNG)

其中：节点 B 是节点 C D E 的**父节点**，C D E 就是 B 的**子节点**，C D E 之间称为**兄弟节点**，我们把没有父节点的 A 节点叫做**根节点**，我们把没有子节点的节点称为**叶子节点**如：F G H I K L 均是叶子节点。

#### 3.1.2 高度、深度及层

理解了树的定义之后我们来学习几个跟树相关的概念：**高度(Heigh)，深度(Depth)，层(Level)**，我们依次来看这几个概念：

节点的高度：节点到叶子节点的最长路径(边数)，所有叶子节点的高度为 0。

节点的深度：根节点到这个节点所经历的边的个数，根的深度为 0。

节点的层数：节点的深度+1

树的高度：根节点的高度

我们用一幅图来继续说明如下：

![树的高度深度](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106999.PNG)

### 3.2 二叉树

树这种数据结构形式结构是多种多样的，但是在实际企业开发中用的最多的还是二叉树，接下来我们学习二叉树

#### 3.2.1 二叉树的定义

二叉树，顾名思义，每个节点最多有两个“叉”，也就是两个子节点，分别是**左子节点**和**右子节点**。不过，二叉树并不要求每个节点都有两个子节点，有的节点只有左子节点，有的节点只有右子节点，如下图所示均是二叉树

![二叉树2.0](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106695.PNG)

当然了在这三棵树中，有两棵比较特殊的二叉树，分别是 T2 和 T3

T2：叶子节点全都在最底层，除了叶子节点之外，每个节点都有左右两个子节点，这种二叉树就叫作**满二叉树**。

T3：叶子节点都在最底下两层，最后一层的叶子节点都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大，这种二叉树叫作**完全二叉树**。

满二叉树我们特别容易理解，完全二叉树我们可能就不是特别能够分清楚，下面我画几棵树，你分析一下看哪些是完全二叉树？

![判断完全二叉树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106470.PNG)

通过对比这几棵树，你分析出来哪些是完全二叉树哪些不是吗？

答案是：T1 是完全二叉树，T2,T3,T4 均不是完全二叉树。

你可能会说，满二叉树的特征非常明显，但是完全二叉树的特征不怎么明显啊，单从长相上来看，完全二叉树并没有特别特殊的地方啊，那我们为什么还要特意把它拎出来讲呢？为什么偏偏把最后一层的叶子节点靠左排列的叫完全二叉树？如果靠右排列就不能叫完全二叉树了吗？这个定义的由来或者说目的在哪里？

要理解完全二叉树定义的由来，我们需要先了解，如何表示（或者存储）一棵二叉树？想要存储一棵二叉树，我们有两种方法，一种是基于指针或者引用的二叉链式存储法，一种是基于数组的顺序存储法。

我们先来看比较简单、直观的链式存储法。从我们画的图中你应该可以很清楚地看到，每个节点有三个字段，其中一个存储数据，另外两个是指向左右子节点的指针。我们只要找到根节点，就可以通过左右子节点的指针，把整棵树都串起来。这种存储方式我们比较常用。大部分二叉树代码都是通过这种结构来实现的。

![链式树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106709.PNG)

我们再来看，基于数组的顺序存储法。我们把根节点存储在下标 i = 1 的位置，那左子节点存储在下标 2 × i = 2 的位置，右子节点存储在 2 × i + 1 = 3 的位置。以此类推，B 节点的左子节点存储在 2i = 2 × 2 = 4 的位置，右子节点存储在 2 × i + 1 = 2 × 2 + 1 = 5 的位置。如下图所示：

![数组树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106326.PNG)

像我刚刚图中这棵树其实是一个完全二叉树，我们只是浪费了数组下标为 0 的位置，但如果对于如下这棵树，我们浪费的存储空间可就多了，

![非完全二叉树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106872.PNG)

总结一下，如果节点 X 存储在数组中下标为 i 的位置，下标为 2 × i 的位置存储的就是左子节点，下标为 2 × i + 1 的位置存储的就是右子节点。反过来，下标为 i/2 的位置存储就是它的父节点。通过这种方式，我们只要知道根节点存储的位置（一般情况下，为了方便计算子节点，根节点会存储在下标为 1 的位置），这样就可以通过下标计算，把整棵树都串起来。

#### 3.2.2 二叉树的遍历

前面我们讲述了二叉树的存储结构接下来我们来学习二叉树的遍历方式，经典的三种遍历方式：**前序遍历，中序遍历，后续遍历**，我们依次来看

前序遍历：对于树中的任意节点来说，先打印这个节点，然后再打印它的左子树，最后打印它的右子树。

中序遍历：对于树中的任意节点来说，先打印它的左子树，然后再打印它本身，最后打印它的右子树。

后序遍历：对于树中的任意节点来说，先打印它的左子树，然后再打印它的右子树，最后打印这个节点本身。

我们还是以图示的方式来表述这个过程：

![二叉树遍历](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106961.PNG)

实际上从遍历的过程我们可以总结出：二叉树的前序，中序，后序遍历是一个递归的过程，比如前序遍历，就是先打印根节点，然后递归的打印其左子树，然后递归的打印其右子树，那我们之前也分析过递归代码的编写，主要是要找出递推公式以及递归终止条件，那你能自行编写出二叉树前序，中序，后序遍历的代码吗？

另外遍历二叉树遍历的时间复杂度是多少呢？通过我们分析的二叉树的遍历流程我们可以发现，遍历二叉树的时间复杂度跟二叉树节点的个数 n 成正比，因此，**二叉树遍历的时间复杂度是O(n)**。

#### 3.2.3 二叉查找树

二叉查找树又名二叉搜索树，有序二叉树或者排序二叉树，是二叉树中比较常用的一种类型，我们先来看二叉查找树的结构定义

##### 3.2.3.1 结构及特点

**二叉查找树要求，在树中的任意一个节点，其左子树中的每个节点的值，都要小于这个节点的值，而右子树节点的值都大于这个节点的值。**

详细可以分为以下 4点：

1. **若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；**

2.  **若任意节点的右子树不空，则右子树上所有节点的值均大于它的根节点的值；**

3.  **任意节点的左、右子树也分别为二叉查找树；**

4. **没有键值相等的节点。**

 我们以一副图示表示一下该树的结构

![二叉查找树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106818.PNG)

我们从二叉查找树这个名称就能够体会到该树的一个特点，能够支持快速查找，那除此之外还有什么特点呢？包括我们为什么还要来学习这种树呢？

**二叉查找树支持动态数据的快速插入，删除，查找操作**。这是它的特点，也正是我们要来学习它的原因，我们之前也学习过散列表这种数据结构，它也支持这几个操作，并且散列表实现这几个操作更加的高效，时间复杂度是 O(1),那既然有了如此高效的散列表为什么还要来学习二叉查找树，是不是有某些情况我们必须使用它？带着这几个问题我们依次展开二叉查找树这几个特点的相关学习。

##### 3.2.3.2 查找操作

我们来看如何在一棵二叉查找树中查询某一个值的节点：我们从根节点开始，如果它等于我们要查找的数据，那就返回。如果要查找的数据比根节点的值小，那就在左子树中递归查找；如果要查找的数据比根节点的值大，那就在右子树中递归查找。如图

![二叉查找树查找](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106537.PNG)

代码实现如下：

```java
/**
 *	二叉搜索树
 */
public class SimpleBinarySearchTree {

    // 二叉查找树,指向根节
    private Node root;

    private static class Node{
        public int value;
        public Node left;
        public Node right;

        public Node(int value, Node left, Node right) {
            this.value = value;
            this.left = left;
            this.right = right;
        }

        // 输出节点的值+左右孩子的值(如果有的话)
        @Override
        public String toString() {
            return "Node{" +
                    "value=" + value +
                    ", left=" + (left==null?"null":left.value) +
                    ", right=" + (right==null?"null":right.value) +
                    '}';
        }
    }

    //根据指定的值查找对应的节点
    public Node find(int value){
        // 从根节点开始遍历
        Node parent = root;

        while (parent != null){
            if(parent.value > value){
                // 当查找值大于该节点值，则往查找该节点的左子节点
                parent = parent.left;
            }else if(parent.value < value){
                // 当查找值小于该节点值，则往查找该节点的右子节点
                parent = parent.right;
            }else{
                // 相等时返回该节点
                return parent;
            }
        }
        return null;
    }
}
```

因为暂时还没写数据的插入方法，因此目前还没办法判断查询方法的有效性，接下来我们分析数据的插入。

##### 3.2.3.3 插入操作

二叉查找树的插入操作和查询操作有点类似，也是要先从根节点开始依次比较要插入的数据和节点的数据的大小并以此来判断是将数据插入其左子树还是右子树，如果节点的右子树为空，就将新数据直接插到右子节点的位置；如果不为空，就再递归遍历右子树，查找插入位置。同理，如果要插入的数据比节点数值小，并且节点的左子树为空，就将新数据插入到左子节点的位置；如果不为空，就再递归遍历左子树，查找插入位置。

![二叉树插入](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252106844.PNG)

```java
/**
 *	二叉搜索树
 */
public class SimpleBinarySearchTree {

    // 二叉查找树,指向根节
    private Node root;

    private static class Node{
		// ...
    }

    //根据指定的值查找对应的节点
    public Node find(int value){
		//...
    }

    //将 value 值存入容器
    public boolean put(int value){
        if(root == null){
            // 根节点为空，创建树
            root = new Node(value, null, null);
            return true;
        }

        Node parent = root;
        while (parent!=null){
            if(parent.value > value){
                // 当查找值大于该节点值：
                // 左子节点为空，直接插入
                if(parent.left == null){
                    parent.left = new Node(value, null, null);
                    return true;
                }
                // 左子节点不为空，继续查找该节点的左子节点
                parent = parent.left;
            }else if(parent.value < value){
                if(parent.right == null){
                    parent.right = new Node(value, null, null);
                    return true;
                }
                parent = parent.right;
            }else{
                parent.value = value;
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        SimpleBinarySearchTree tree = new SimpleBinarySearchTree();
        //向容器中添加值
        tree.put(8);
        tree.put(6);
        tree.put(5);
        tree.put(7);
        tree.put(4);
        tree.put(11);
        tree.put(10);
        tree.put(13);
        tree.put(9);
        tree.put(12);
        tree.put(14);
        tree.put(15);

        int value = 8;
        SimpleBinarySearchTree.Node node = tree.find(value);
        System.out.println("节点值为"+value+"的节点为:"+node);
    }
}
```

##### 3.2.3.4 删除操作

二叉查找树的插入和查询相对来说比较简单易懂，但是删除操作相对复杂，总结下来有三种情况：

1：要删除的节点是叶子节点即没有子节点，我们只需将父节点中指向该节点的指针置为 null 即可，这是最简单的一种形式。比如删除图中的节点 10

2：要删除的节点只有一个子节点(只有左子节点或者只有右子节点)，我们只需要更新父节点中，指向要删除节点的指针，让它指向要删除节点的子节点就可以了。比如删除图中的节点 38

3：要删除的节点有两个子节点，这是最复杂的一种情况，我们需要找到这个节点的右子树中的最小节点，把它替换到要删除的节点上。然后再删除掉这个最小节点，因为最小节点肯定没有左子节点（如果有左子结点，那就不是最小节点了），所以，我们可以应用上面两条规则来删除这个最小节点。比如删除图中的节点 25

![二叉树删除1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107929.PNG)

按照规则删除之后的树结构为：

![二叉树删除2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107643.PNG)

代码实现如下：

```java
/**
 *	二叉搜索树
 */
public class SimpleBinarySearchTree {

    // 二叉查找树,指向根节
    private Node root;

    private static class Node{...}

    //根据指定的值查找对应的节点
    public Node find(int value){...}

    //将 value 值存入容器
    public boolean put(int value){...}

    // 删除节点
    public void remove(int value){
        // 记录要删除的节点
        Node current = root;
        // 记录要删除节点的父节点
        Node parent = null;
        // 先找到要删除的元素及其父元素
        while (current != null){
            if(current.value > value){
                // 当查找值大于该节点值，则往查找该节点的左子节点
                parent = current;
                current = current.left;
            }else if(current.value < value){
                // 当查找值小于该节点值，则往查找该节点的右子节点
                parent = current;
                current = current.right;
            }else{
                // 相等时跳出循环
                break;
            }
        }

        //如果没有找到则返回
        if(current == null){
            return;
        }

        //要删除的节点有两个子节点 这种情况要用右子树中最小节点的值替换当前要删除元素的值，然后删除右侧最小节点
        if(current.left != null && current.right != null){
            // 找到该节点右子树的最小节点----->最左侧的叶子节点
            Node rightTree = current.right;
            // rigthTree 的父节
            Node rightTree_parent = current;
            // 遍历找到要删除节点右子树的最左节点
            while (rightTree.left != null){
                rightTree_parent = rightTree;
                rightTree = rightTree.left;
            }

            // 用右子树中最小的节点替换当前要删除的节点
            current.value = rightTree.value;
            
            // 要删除的节点与其父节点复制给current与parent，那么后续只需对current和parent进行删除操作即可
            current = rightTree;
            parent = rightTree_parent;
        }

        //删除节点是叶子节点或者仅有一个子节点,都是要删除该节点，将父节点的指针指向当前节点的子节点
        Node child = null;
        if(current.right != null){
            child = current.right;
        }else if(current.left != null){
            child = current.left;
        }else{
            child = null;
        }

        //执行删除
        if(parent == null){
            root = child;
        }else if(parent.left == current){
            parent.left = child;
        }else if(parent.right == current){
            parent.right = child;
        }
    }

    public static void main(String[] args) {
        SimpleBinarySearchTree tree = new SimpleBinarySearchTree();
        //向容器中添加值
        tree.put(16);
        tree.put(14);
        tree.put(12);
        tree.put(15);
        tree.put(10);
        tree.put(35);
        tree.put(25);
        tree.put(20);
        tree.put(27);
        tree.put(26);
        tree.put(30);
        tree.put(40);
        tree.put(38);
        tree.put(41);
        tree.put(39);

        int indexValue = 26;
        int deleteValue = 25;
        System.out.println("删除前节点为:"+tree.find(indexValue));
        tree.remove(deleteValue);
        System.out.println("删除后节点为:"+tree.find(indexValue));
    }
}
```

##### 3.2.3.5 查找最大值/最小值

对于查找二叉查找树的最小值，我们只需要从根节点开始依次查找其左子节点直到最后的叶子节点，最后的叶子节点就是其最小值，同理查找最大值只需要从根节点开始依次查找其右子节点直到最后的叶子节点即为最大值。

代码实现如下

```java
public Node getMax(){
    // 从根节点开始遍历
    Node parent = root;

    if(parent == null){
        return null;
    }

    while (parent.right != null){
        parent = parent.right;
    }
    return parent;
}

public Node getMin(){
    // 从根节点开始遍历
    Node parent = root;

    if(parent == null){
        return null;
    }

    while (parent.left != null){
        parent = parent.left;
    }
    return parent;
}
```

当然了对于二叉查找树我们还可以找到节点的前驱节点和后继节点，对于这两个概念我们解释说明如下：

**后继节点：该节点右子树中最小的节点**

**前驱节点：该节点左子树中最大的节点**

理解了这两个概念之后，留一个课后思考题：如何通过编程的方式获取某一个节点的前驱和后继节点呢？

##### 3.2.3.6 其他操作

二叉查找树除了上述的相关操作外，还有一个重要的特性，就是**如果中序遍历二叉查找树，能够得到一个有序的数据序列，时间复杂度是O(n)**，非常的高效，因此二叉查找树也叫**二叉排序树**，

我们前面讲解的时候存储的都是 int 类型的数字，但是在实际的软件开发中我们一般都是存储的包含很多属性的对象，判断的时候利用对象中的某一个属性进行判断，对象中的其他属性我们称为卫星数据，对于有重复数据的二叉查找树，我们应该如何存储和查找呢？有两种方案：

1. 二叉查找树中每一个节点不仅会存储一个数据，因此我们通过链表和支持动态扩容的数组等数据结构，把值相同的数据都存储在同一个节点上。

2. 每个节点仍然只存储一个数据。在查找插入位置的过程中，如果碰到一个节点的值，与要插入数据的值相同，我们就将这个要插入的数据放到这个节点的右子树，也就是说，把这个新插入的数据当作大于这个节点的值来处理，当要查找数据的时候，遇到值相同的节点，我们并不停止查找操作，而是继续在右子树中查找，直到遇到叶子节点，才停止。这样就可以把键值等于要查找值的所有节点都找出来。对于删除操作，我们也需要先查找到每个要删除的节点，然后再按前面讲的删除操作的方法，依次删除。

   ![二叉树中有相同的值](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107698.PNG)

##### 3.2.3.7 时间复杂度分析

在这一节中我们分析一下二叉查找树的查找，插入，删除的相关操作的时间复杂度。

实际上由于二叉查找树的形态各异，时间复杂度也不尽相同，我画了几棵树我们来看一下插入，查找，删除的时间复杂度：

![不同形态的二叉树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107244.PNG)

对于图中第一种情况属于最坏的情况，二叉查找树已经退化成了链表，左右子树极度不平衡，此时查找的时间复杂度肯定是 O(n)。

对于图中第二种或者第三种情况是属于一个比较理想的情况，我们代码的实现逻辑以及图中所示表明插入，查找，删除的时间复杂度其实和树的高度成正比，那也就是说时间复杂度为**O(height)**

那如何求一棵完全二叉树的高度？即求一棵包含 n 个节点的完全二叉树的高度？

对于一棵满二叉树而言：树的高度就等于最大层数减一，为了方便计算，我们转换成层来表示。从上图中可以看出，包含 n 个节点的完全二叉树中，第一层包含 1 个节点，第二层包含 2 个节点，第三层包含 4 个节点，依次类推，下面一层节点个数是上一层的 2 倍，第 k 层包含的节点个数就是 2^(k-1)。

但是对于完全二叉树来说，最后一层的节点个数有点儿不遵守上面的规律了。它包含的节点个数在 1 个到 2^(k-1)个之间（我们假设最大层数是 k）。如果我们把每一层的节点个数加起来就是总的节点个数 n。也就是说，如果节点的个数是 n，那么 n 满足这样一个关系：
$$
1+2+4+8+...+2^{(k-2)} +1 =< n <= 1+2+4+8+...+2^{(k-2)} +2^{(k-1)}
$$
这是一个等比数列，根据等比数列求和公式
$$
S= \frac{a_1-a_nq}{1-q},其中q是公比,a_n为数列的第n项，a_1为首项
$$
所以：我们利用求和公式对上述式子进行计算后得知，k 的取值范围为：
$$
log_2(n+1) \leq k \leq log_2(n) + 1
$$
推导得，也就是说完全二叉树的高度小于等于：[log2n]+1

> 还需要看书斟酌一番

通过我们的分析我们发现一棵极度不平衡的二叉查找树，它的查找性能和单链表一样。我们需要构建一种不管怎么删除、插入数据，在任何时候，都能保持任意节点左右子树都比较平衡的二叉查找树，这种特殊的二叉查找树也可以叫做**平衡二叉查找树。平衡二叉查找树的高度接近logn，所以插入、删除、查找操作的时间复杂度也比较稳定，是O(logn)**。

##### 3.2.3.8 二叉树和散列表的对比

之前我们学习过散列表的插入、删除、查找操作的时间复杂度可以做到常量级的O(1)，非常高效。而二叉查找树在比较平衡的情况下，插入、删除、查找操作时间复杂度才是 O(logn)，相对散列表，好像并没有什么优势，那我们为什么还要用二叉查找树呢？个人认为原因如下：

1. 散列表中的数据是无序存储的，如果要输出有序的数据，需要先进行排序。而对于二叉查找树来说，我们只需要中序遍历，就可以在 O(n) 的时间复杂度内，输出有序的数据序列。
2. 散列表扩容耗时很多，而且当遇到散列冲突时，性能不稳定，尽管二叉查找树的性能也不稳定，但是在工程中，我们最常用的平衡二叉查找树的性能非常稳定，时间复杂度稳定在 O(logn)。
3. 尽管散列表的查找等操作的时间复杂度是常量级的，但因为哈希冲突的存在，这个常量不一定比 logn 小，所以实际的查找速度可能不一定比 O(logn) 快。加上哈希函数的耗时，也不一定就比平衡二叉查找树的效率高。
4. 散列表的构造比二叉查找树要复杂，需要考虑的东西很多。比如散列函数的设计、冲突解决办法、扩容、缩容等。平衡二叉查找树只需要考虑平衡性这一个问题，而且这个问题的解决方案比较成熟、固定。
5. 为了避免过多的散列冲突，散列表装载因子不能太大，特别是基于开放寻址法解决冲突的散列表，不然会浪费一定的存储空间。

综合这几点，平衡二叉查找树在某些方面还是优于散列表的，所以，这两者的存在并不冲突。我们在实际的开发过程中，需要结合具体的需求来选择。

#### 3.2.4 平衡二叉树(AVL)

上一节我们讲到，二叉查找树只有在比较平衡的情况下，插入、删除、查找操作时间复杂度才是 O(logn)，不过在二叉查找树频繁的动态更新过程中，会逐渐退化直至最坏的情况变为链表，时间复杂度退化为 O(n)，所以我们要解决这种复杂度退化的问题就要找到一种平衡二叉树，平衡二叉查找树中“平衡”的意思，其实就是让整棵树左右看起来比较“对称”、比较“平衡”，不要出现左子树很高、右子树很矮的情况。这样就能让整棵树的高度相对来说低一些，相应的插入、删除、查找等操作的效率高一些。

##### 3.2.4.1 定义

**平衡二叉查找树**：简称平衡二叉树。由前苏联的数学家 Adelse-Velskil 和 Landis 在 1962 年提出的高度平衡的二叉树，根据科学家的英文名也称为 AVL 树。它具有如下几个性质：

1. 可以是空树。

2. 假如不是空树，任何一个结点的左子树与右子树都是平衡二叉树，并且高度之差的绝对值不超过 1

   ![平衡二叉树](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107011.PNG)

上图中的两棵树，左边的是AVL 树，它的任何节点的两个子树的高度差别都<=1；而右边的不是 AVL 树，因为 7 的两颗子树的高度相差为 2(以 2 为根节点的树的高度是 2，而以 8 为根节点的树的高度是 0)。

**对于给定结点数为n的AVL树，最大高度为O(log2n).**也就说，从 n 个数中，查找一个特定值时，最多需要 log2n 次。因此，AVL 是一种特别适合进行查找操作的树。

##### 3.2.4.2 失衡的四种情况及调整方法

在平衡二叉树中，当我们插入新的元素时，为了保证二叉搜索树的特性，很容易导致某些结点**失衡**，即该结点的平衡因子大于 1。

而在二叉树中，任意结点孩子最多只有左右两个，而且导致失去平衡的必要条件就是当前结点的两颗子树的高度差等于 2。因此，致使一个结点失衡的插入操作有以下 4 种

1. 在结点的左子树的左子树插入元素,LL 插入；
2. 在结点的左子树的右子树插入元素,LR 插入；
3. 在结点的右子树的左子树插入元素,RL 插入；
4. 在结点的右子树的右子树插入元素,RR 插入。

![平衡二叉树失衡](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107421.PNG)

> LL(一）中结点 1 的插入导致结点 8 失衡，而插入的位置是在其左子树的左子树上，同样 LL(二）中，结点 3 插入同样导致结点 8 失衡，**这里需要注意子树是从受影响的结点算起，虽然3 插在了右边，但他依旧是在 8（失衡结点）左子树的左子树上，因此属于 LL 插入。**
>
> LR(一）,结点 5 插入导致结点 8 失衡，插入位置是在其左子树的右子树上，同样 LR(二)结点 7 的插入也是同理，因此这二者都属于 LR 插入。

**面对以上4种失衡的情况，在AVL树中将采用LL(左左)，LR(左右)，RR(右右)和RL(右左)四种旋转方式进行调整。**

为了对这四种情况进行说明，我们先来定义 AVL 树的结构：

AVL 树首先是二叉查找树，因此它的结点也必须是可比较。同时为了方便，加入一个表示当前结点高度的 height 字段，同时定义了返回节点高度和树的高度的方法，也定了一个返回两个高度中最大高度的方法。

```java
public class AVLTree <T extends Comparable<T>>{

    //AVL树根节点
    private AVLNode tree;

    //获取某一节点的高度
    public int height(AVLNode node){
        // 特殊说明：这里的高度对于叶子节点而且，其高度其实为1
        return node==null? 0: node.height;
    }

    // 获取 avl 树的高度
    public int height(){
        return tree.height;
    }

    // 返回两个高度中的最大值
    private int getMax(int height1, int height2){
        return height1>height2 ? height1 : height2;
    }

    // 方法我们使用前中后序遍历的方式打印树
    @Override
    public String toString() {
        System.out.println("前序遍历的结果:");
        preOrder(tree);
        System.out.println();
        System.out.println("中序遍历的结果:");
        inOrder(tree);
        System.out.println();
        System.out.println("后续遍历的结果:");
        inOrder(tree);
        System.out.println();

        return null;
    }

    // 前序遍历
    public void preOrder(AVLNode node){
        if(node == null){
            return;
        }
        System.out.print(node.data + "->");
        preOrder(node.left);
        preOrder(node.right);
    }

    //中序遍历
    public void inOrder(AVLNode node){
        if(node == null){
            return;
        }
        preOrder(node.left);
        System.out.print(node.data + "->");
        preOrder(node.right);
    }

    //后续遍历
    public void postOrder(AVLNode node){
        if(node == null){
            return;
        }
        preOrder(node.left);
        preOrder(node.right);
        System.out.print(node.data + "->");
    }

    private static class AVLNode<T extends Comparable<T>>{
        // 节点中存储的数据
        private T data;
        // 左子树节点
        private AVLNode<T> left;
        // 右子树节点
        private AVLNode<T> right;
        //节点的高度
        private int height;

        public AVLNode(T data, AVLNode<T> left, AVLNode<T> right, int height) {
            this.data = data;
            this.left = left;
            this.right = right;
            this.height = height;
        }

        public AVLNode(T data, AVLNode<T> left, AVLNode<T> right) {
            this.data = data;
            this.left = left;
            this.right = right;
        }
    }
}
```

1. **LL-左左旋转**

![左左旋转](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107020.PNG)

代码实现如下：

```java
// LL旋转
public AVLNode leftRotate(AVLNode node){
    // 定义临时变量保存 失衡节点的左子树 该节点也是左旋后的根节点
    AVLNode node_left = node.left;
    // 将失衡节点左子树的右子树作为失衡节点的左子树
    node.left = node_left.right;
    // 将失衡节点作为旋转后根节点的右子树
    node_left.right = node;
    // 重新计算失衡节点和旋转后根节点的高度
    node.height = getMax(height(node.left), height(node.right)) + 1;
    node_left.height = getMax(height(node_left.left), height(node_left.right)) + 1;
    return node_left;
}
```

2. **RR-右右旋转**

![右右旋转](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107908.PNG)

代码实现如下：

```java
// RR旋转
public AVLNode rightRotate(AVLNode node){
    //	定义临时变量保存右旋后的根节点 也就是失衡节点的右子树
    AVLNode node_right = node.right;
    //	将新的根节点的左子树当作失衡节点的右子树
    node.right = node_right.left;
    //	将失衡节点当作新的根节点的左子树
    node_right.left = node;
    //	重新计算失衡节点和新的根节点的高度
    node.height = getMax(height(node.left), height(node.right)) + 1;
    node_right.height = getMax(height(node_right.left), height(node_right.right)) + 1;
    return node_right;
}
```

3. **LR-左右旋转**

![左右旋转](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107511.PNG)

代码实现如下：

```java
// LR-左右旋转
public AVLNode leftRightRotate(AVLNode node){
    node.left = rightRotate(node.left);
    return leftRotate(node);
}
```

4. **RL-右左旋转**

![右左旋转](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206252107691.PNG)

代码实现如下：

```java
// RL-右左旋转
public AVLNode rightLeftRotate(AVLNode node){
    node.right = leftRotate(node.right);
    return rightRotate(node);
}
```

##### 3.2.4.3 插入操作

上一节中我们分析了平衡二叉树的几种失衡情况以及对应的调整策略，下面我们来分析一下如何向平衡二叉树中插入数据，

 代码实现如下：

```java
public class AVLTree <T extends Comparable<T>>{

    //AVL树根节点
    private AVLNode tree;

    //获取某一节点的高度
    public int height(AVLNode node){
        return node==null? 0: node.height;
    }

    // 获取 avl 树的高度
    public int height(){
        return tree.height;
    }

    // 返回两个高度中的最大值
    private int getMax(int height1, int height2){
        return height1>height2 ? height1 : height2;
    }

    // 方法我们使用前中后序遍历的方式打印树
    @Override
    public String toString() {
        System.out.println("前序遍历的结果:");
        preOrder(tree);
        System.out.println();
        System.out.println("中序遍历的结果:");
        inOrder(tree);
        System.out.println();
        System.out.println("后续遍历的结果:");
        postOrder(tree);
        System.out.println();

        return null;
    }

    // 前序遍历
    public void preOrder(AVLNode node){
        if(node == null){
            return;
        }
        System.out.print(node.data + "->");
        preOrder(node.left);
        preOrder(node.right);
    }

    //中序遍历
    public void inOrder(AVLNode node){
        if(node == null){
            return;
        }
        inOrder(node.left);
        System.out.print(node.data + "->");
        inOrder(node.right);
    }

    //后续遍历
    public void postOrder(AVLNode node){
        if(node == null){
            return;
        }
        postOrder(node.left);
        postOrder(node.right);
        System.out.print(node.data + "->");
    }

    // LL旋转
    public AVLNode leftRotate(AVLNode node){
        // 定义临时变量保存 失衡节点的左子树 该节点也是左旋后的根节点
        AVLNode node_left = node.left;
        // 将失衡节点左子树的右子树作为失衡节点的左子树
        node.left = node_left.right;
        // 将失衡节点作为旋转后根节点的右子树
        node_left.right = node;
        // 重新计算失衡节点和旋转后根节点的高度
        node.height = getMax(height(node.left), height(node.right));
        node_left.height = getMax(height(node_left.left), height(node_left.right));
        return node_left;
    }

    // RR旋转
    public AVLNode rightRotate(AVLNode node){
        //	定义临时变量保存右旋后的根节点 也就是失衡节点的右子树
        AVLNode node_right = node.right;
        //	将新的根节点的左子树当作失衡节点的右子树
        node.right = node_right.left;
        //	将失衡节点当作新的根节点的左子树
        node_right.left = node;
        //	重新计算失衡节点和新的根节点的高度
        node.height = getMax(height(node.left), height(node.right));
        node_right.height = getMax(height(node_right.left), height(node_right.right));
        return node_right;
    }

    // LR-左右旋转
    public AVLNode leftRightRotate(AVLNode node){
        node.left = rightRotate(node.left);
        return leftRotate(node);
    }

    // RL-右左旋转
    public AVLNode rightLeftRotate(AVLNode node){
        node.right = leftRotate(node.right);
        return rightRotate(node);
    }

    // 插入操作
    public void insert(T value){
        this.tree = insert(tree, value);
    }

    private AVLNode insert(AVLNode<T> node, T data){
        //将 data 添加到 node 节点的子节
        if(node == null){
            node = new AVLNode<T>(data, null, null);
        }else{
            int compare = data.compareTo(node.data);
            if(compare > 0){
                // 要添加的值大于当前节点的值,将 data 存储到当前节点的右子树上
                // 递归的插入
                node.right = insert(node.right, data);
                //	如插入后 avl 树变得不平衡,则应该重新调节该树的结构---旋转
                if(height(node.right) - height(node.left) >= 2){
                    //	判断是 RR 还是 RL
                    if(data.compareTo(node.right.data) > 0){
                        // RR
                        node = rightRotate(node);
                    }else{
                        node = rightLeftRotate(node);
                    }
                }

            }else if(compare < 0){
                // 要添加的值小于当前节点的值,将 data 存储到当前节点的左子树上
                // 递归的插入
                node.left = insert(node.left, data);
                //	如插入后 avl 树变得不平衡,则应该重新调节该树的结构---旋转
                if(height(node.left) - height(node.right) >= 2){
                    if(data.compareTo(node.left.data) < 0){
                        // LL
                        node = leftRotate(node);
                    }else{
                        node = leftRightRotate(node);
                    }
                }
            }else{
                //要添加的值和该节点的值相同，不做处理
            }
        }

        //计算节点 node 的高度
        // +1解释：hight为左右子树的双亲节点，故此处需要+1
        node.height = getMax(height(node.left), height(node.right)) + 1;
        return node;
    }


    private static class AVLNode<T extends Comparable<T>>{
        // 节点中存储的数据
        private T data;
        // 左子树节点
        private AVLNode<T> left;
        // 右子树节点
        private AVLNode<T> right;
        //节点的高度
        private int height;

        public AVLNode(T data, AVLNode<T> left, AVLNode<T> right, int height) {
            this.data = data;
            this.left = left;
            this.right = right;
            this.height = height;
        }

        public AVLNode(T data, AVLNode<T> left, AVLNode<T> right) {
            this.data = data;
            this.left = left;
            this.right = right;
        }
    }

    public static void main(String[] args) {
        AVLTree<Integer> tree = new AVLTree<>();
        //	添加节点
        tree.insert(10);
        tree.insert(8);
        tree.insert(3);
        tree.insert(12);
        tree.insert(9);
        tree.insert(4);
        tree.insert(5);
        tree.insert(7);
        tree.insert(1);
        tree.insert(11);
        tree.insert(17);
        System.out.println(tree);
    }
}
```

在这里我们分析了 AVL 树的插入操作，那对于 AVL 树的查询操作和删除操作应该如何实现呢？你可以试着去分析和实现一下。

#### 3.2.5 小结

本章内容我们学习了树这种数据结构，最主要是学习了二叉树，掌握了二叉树的定义，二叉树的遍历方式，介绍了二叉查找树，对二叉查找树的查找，插入，删除等相关操作做了实现，接着又学习了 AVL 树，实现了 AVL 树的部分功能，最后分析了一下红黑树及其平衡的过程。对于我们日常的软件编程而言，我们知道了树对于动态数据的添加，删除和查询操作是非常友好的，基本上其时间复杂度为 O(n)，在某些情况下甚至于要好于 O(1)的散列表。

## 4. 总结

### 4.1 散列表

散列函数的特点及要求：

1. 散列函数计算得到的散列值必须是大于等于 0 的正整数，因为 hash 值需要作为数组的下标。
2. 如果 key1==key2，那么经过 hash 后得到的哈希值也必相同即：hash(key1) == hash(key2)
3. 如果 key1 != key2，那么经过 hash 后得到的哈希值也必不相同即：hash(key1) != hash(key2)

散列函数设计的方法：

1. 直接寻址法
2. 除留余数发
3. 数字分析法
4. 平方取中法
5. 折叠法

 散列冲突的解决方案：

1. 开放寻址-线性探测
2. 开放寻址-二次检测
3. 开放寻址-双重散列
4. 链表法

散列表的应用：

1. HashMap 的源码解析
2. HashTable 的源码解析

哈希算法的应用场景：

1. 安全加密
2. 唯一 ID
3. 数据校验
4. 散列函数
5. 负载均衡
6. 数据分片
7. 分布式存储

### 4.2 树

二叉树的定义 

二叉树的遍历：前序，中序，后序遍历 

二叉查找树

AVL 树