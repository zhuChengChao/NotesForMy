# JVM：类加载与字节码技术-1

> 在学习了 bilibili 上 **[黑马程序员]()** 的课程 [JVM完整教程](https://www.bilibili.com/video/BV1yE411Z7AP) 后做的学习笔记，总共分为5篇笔记，本文为 3/5 篇。

### 内容

1. 类文件结构
2. 字节码指令

> 下面的内容在后续笔记中：
>
> 3. 编译期处理
> 4. 类加载阶段
> 5. 类加载器
> 6. 运行期优化 

![类加载与字节码技术](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623233.PNG)

## 1. 类文件结构 

一个简单的 HelloWorld.java

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
```

执行 `javac -parameters -d . HellowWorld.java` 

>  参数-parameters会保存方法中参数的名称信息

编译为 HelloWorld.class 后是这个样子的：

```java
[root@localhost ~]# od -t xC HelloWorld.class
0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b 07
0000040 00 1c 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29
0000060 56 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e
0000100 75 6d 62 65 72 54 61 62 6c 65 01 00 12 4c 6f 63
0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 01
0000140 00 04 74 68 69 73 01 00 1d 4c 63 6e 2f 69 74 63
0000160 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f
0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e 01 00 16
0000220 28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72
0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 01 00 13
0000260 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69
0000300 6e 67 3b 01 00 10 4d 65 74 68 6f 64 50 61 72 61
0000320 6d 65 74 65 72 73 01 00 0a 53 6f 75 72 63 65 46
0000340 69 6c 65 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64
0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d 0c 00 1e
0000400 00 1f 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64
0000420 07 00 20 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74
0000440 63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c
0000460 6f 57 6f 72 6c 64 01 00 10 6a 61 76 61 2f 6c 61
0000500 6e 67 2f 4f 62 6a 65 63 74 01 00 10 6a 61 76 61
0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d 01 00 03 6f
0000540 75 74 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72
0000560 69 6e 74 53 74 72 65 61 6d 3b 01 00 13 6a 61 76
0000600 61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d
0000620 01 00 07 70 72 69 6e 74 6c 6e 01 00 15 28 4c 6a
0000640 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b
0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01
0000700 00 07 00 08 00 01 00 09 00 00 00 2f 00 01 00 01
0000720 00 00 00 05 2a b7 00 01 b1 00 00 00 02 00 0a 00
0000740 00 00 06 00 01 00 00 00 04 00 0b 00 00 00 0c 00
0000760 01 00 00 00 05 00 0c 00 0d 00 00 00 09 00 0e 00
0001000 0f 00 02 00 09 00 00 00 37 00 02 00 01 00 00 00
0001020 09 b2 00 02 12 03 b6 00 04 b1 00 00 00 02 00 0a
0001040 00 00 00 0a 00 02 00 00 00 06 00 08 00 07 00 0b
0001060 00 00 00 0c 00 01 00 00 00 09 00 10 00 11 00 00
0001100 00 12 00 00 00 05 01 00 10 00 00 00 01 00 13 00
0001120 00 00 02 00 14
```

根据 JVM 规范，类文件结构如下

```java
ClassFile{
    //  u4/u2 表示字节数
	u4 					magic;  		// 魔术			
	u2 					minor_version;  // 版本
	u2 					major_version;  // 版本
	u2 					constant_pool_count;  				   //常量池信息
	cp_info 			constant_pool[constant_pool_count-1];  //常量池信息
	u2 					access_flags;      // 访问修饰
	u2 					this_class;        // 类信息
	u2 					super_class;	   // 父类信息
	u2 					interfaces_count;  // 接口信息
	u2 					interfaces[interfaces_count];  // 接口信息
	u2 					fields_count;			// 类中的成员变量信息
	field_info  		fields[fields_count];	// 类中的成员变量信息
	u2 					methods_count;			// 类中的方法信息
	method_info 		methods[methods_count];	// 类中的方法信息
	u2 					attributes_count;			   // 类的附加属性信息
	attribute_info 		attributes[attributes_count];  // 类的附加属性信息
}
```

### 1.1 魔数magic

0~3 字节，表示它是否是【class】类型的文件 

0000000 **ca fe ba be** 00 00 00 34 00 23 0a 00 06 00 15 09

**ca fe ba be** ：表示这一个java类型的文件

### 1.2 版本 

4~7 字节，表示类的版本 00 34（52） 表示是 Java 8 

0000000 ca fe ba be **00 00 00 34** 00 23 0a 00 06 00 15 09

### 1.3 常量池 

| Constant Type               | Value    |
| --------------------------- | -------- |
| CONSTANT_Class              | 7        |
| CONSTANT_Fieldref           | 9        |
| CONSTANT_Methodref          | 10（0a） |
| CONSTANT_InterfaceMethodref | 11（0b） |
| CONSTANT_String             | 8        |
| CONSTANT_Integer            | 3        |
| CONSTANT_Float              | 4        |
| CONSTANT_Long               | 5        |
| CONSTANT_Double             | 6        |
| CONSTANT_NameAndType        | 12（0c） |
| CONSTANT_Utf8               | 1        |
| CONSTANT_MethodHandle       | 15（0f） |
| CONSTANT_MethodType         | 16（20） |
| CONSTANT_InvokeDynamic      | 18（22） |

8~9 字节，表示常量池长度，00 23 （35） 表示常量池有 #1~#34项，注意 #0 项不计入，也没有值 

0000000 ca fe ba be 00 00 00 34 **00 23** 0a 00 06 00 15 09

**第#1项** 0a 表示一个 Method 信息，00 06 和 00 15（21） 表示它引用了常量池中 #6 和 #21 项来获得这个方法的【所属类】和【方法名】 
0000000 ca fe ba be 00 00 00 34 00 23 **0a 00 06 00 15** 09 

**第#2项** 09 表示一个 Field 信息，00 16（22）和 00 17（23） 表示它引用了常量池中 #22 和 # 23 项来获得这个成员变量的【所属类】和【成员变量名】 
0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 **09**
0000020 **00 16 00 17** 08 00 18 0a 00 19 00 1a 07 00 1b 07 

**第#3项** 08 表示一个字符串常量名称，00 18（24）表示它引用了常量池中 #24 项
0000020 00 16 00 17 **08 00 18** 0a 00 19 00 1a 07 00 1b 07 

**第#4项** 0a 表示一个 Method 信息，00 19（25） 和 00 1a（26） 表示它引用了常量池中 #25 和 #26 项来获得这个方法的【所属类】和【方法名】
0000020 00 16 00 17 08 00 18 **0a 00 19 00 1a** 07 00 1b 07 

**第#5项** 07 表示一个 Class 信息，00 1b（27） 表示它引用了常量池中 #27 项 
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a **07 00 1b** 07 

**第#6项** 07 表示一个 Class 信息，00 1c（28） 表示它引用了常量池中 #28 项
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b **07**
0000040 **00 1c** 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29 

**第#7项** 01 表示一个 utf8 串，00 06 表示长度，3c 69 6e 69 74 3e 是【 `<init>` 】,表示了构造方法
0000040 00 1c **01 00 06 3c 69 6e 69 74 3e** 01 00 03 28 29 

**第#8项** 01 表示一个 utf8 串，00 03 表示长度，28 29 56 是【`()V`】其实就是表示无参、无返回值
0000040 00 1c 01 00 06 3c 69 6e 69 74 3e **01 00 03 28 29**
0000060 **56** 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e 

**第#9项** 01 表示一个 utf8 串，00 04 表示长度，43 6f 64 65 是【Code】
0000060 56 **01 00 04 43 6f 64 65** 01 00 0f 4c 69 6e 65 4e 

**第#10项** 01 表示一个 utf8 串，00 0f（15） 表示长度，4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65是【LineNumberTable】
0000060 56 01 00 04 43 6f 64 65 **01 00 0f 4c 69 6e 65 4e**
0000100 **75 6d 62 65 72 54 61 62 6c 65** 01 00 12 4c 6f 63 

**第#11项** 01 表示一个 utf8 串，00 12（18） 表示长度，4c 6f 63 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 是【LocalVariableTable】
0000100 75 6d 62 65 72 54 61 62 6c 65 **01 00 12 4c 6f 63**
0000120 **61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65** 01 

**第#12项** 01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【this】
0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 **01**
0000140 **00 04 74 68 69 73** 01 00 1d 4c 63 6e 2f 69 74 63 

**第#13项** 01 表示一个 utf8 串，00 1d（29） 表示长度，是【Lcn/itcast/jvm/t5/HelloWorld;】
0000140 00 04 74 68 69 73 01 **00 1d 4c 63 6e 2f 69 74 63**
0000160 **61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f** 
0000200 **57 6f 72 6c 64 3b** 01 00 04 6d 61 69 6e 01 00 16

**第#14项** 01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【main】
0000200 57 6f 72 6c 64 3b **01 00 04 6d 61 69 6e** 01 00 16

**第#15项** 01 表示一个 utf8 串，00 16（22） 表示长度，是【([Ljava/lang/String;)V】其实就是参数为字符串数组，无返回值
0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e **01 00 16**
0000220 **28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72**
0000240 **69 6e 67 3b 29 56** 01 00 04 61 72 67 73 01 00 13

**第#16项** 01 表示一个 utf8 串，00 04 表示长度，是【args】
0000240 69 6e 67 3b 29 56 **01 00 04 61 72 67 73** 01 00 13

**第#17项** 01 表示一个 utf8 串，00 13（19） 表示长度，是【[Ljava/lang/String;】
0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 **01 00 13** 
0000260 **5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69** 
0000300 **6e 67 3b** 01 00 10 4d 65 74 68 6f 64 50 61 72 61

**第#18项** 01 表示一个 utf8 串，00 10（16） 表示长度，是【MethodParameters】
0000300 6e 67 3b **01 00 10 4d 65 74 68 6f 64 50 61 72 61**
0000320 **6d 65 74 65 72 73** 01 00 0a 53 6f 75 72 63 65 46

**第#19项** 01 表示一个 utf8 串，00 0a（10） 表示长度，是【SourceFile】
0000320 6d 65 74 65 72 73 **01 00 0a 53 6f 75 72 63 65 46**
0000340 **69 6c 65** 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64

**第#20项** 01 表示一个 utf8 串，00 0f（15） 表示长度，是【HelloWorld.java】
0000340 69 6c 65 **01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64** 
0000360 **2e 6a 61 76 61** 0c 00 07 00 08 07 00 1d 0c 00 1e

**第#21项** 0c 表示一个 【名+类型】，00 07 00 08 引用了常量池中 #7 #8 两项
0000360 2e 6a 61 76 61 **0c 00 07 00 08** 07 00 1d 0c 00 1e

**第#22项** 07 表示一个 Class 信息，00 1d（29） 引用了常量池中 #29 项
0000360 2e 6a 61 76 61 0c 00 07 00 08 **07 00 1d** 0c 00 1e 

**第#23项** 0c 表示一个 【名+类型】，00 1e（30） 00 1f （31）引用了常量池中 #30 #31 两项
0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d **0c 00 1e**
0000400 **00 1f** 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64

**第#24项** 01 表示一个 utf8 串，00 0f（15） 表示长度，是【hello world】
0000400 00 1f **01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64**

**第#25项** 07 表示一个 Class 信息，00 20（32） 引用了常量池中 #32 项
0000420 **07 00 20** 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74

**第#26项** 0c 表示一个 【名+类型】，00 21（33） 00 22（34）引用了常量池中 #33 #34 两项
0000420 07 00 20 **0c 00 21 00 22** 01 00 1b 63 6e 2f 69 74

**第#27项** 01 表示一个 utf8 串，00 1b（27） 表示长度，是【cn/itcast/jvm/t5/HelloWorld】
0000420 07 00 20 0c 00 21 00 22 **01 00 1b 63 6e 2f 69 74**
0000440 **63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c**
0000460 **6f 57 6f 72 6c 64** 01 00 10 6a 61 76 61 2f 6c 61

**第#28项** 01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/Object】
0000460 6f 57 6f 72 6c 64 **01 00 10 6a 61 76 61 2f 6c 61**
0000500 **6e 67 2f 4f 62 6a 65 63 74** 01 00 10 6a 61 76 61

**第#29项** 01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/System】
0000500 6e 67 2f 4f 62 6a 65 63 74 **01 00 10 6a 61 76 61**
0000520 **2f 6c 61 6e 67 2f 53 79 73 74 65 6d** 01 00 03 6f

**第#30项** 01 表示一个 utf8 串，00 03 表示长度，是【out】
0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d **01 00 03 6f**
0000540 **75 74** 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72

**第#31项** 01 表示一个 utf8 串，00 15（21） 表示长度，是【Ljava/io/PrintStream;】
0000540 75 74 **01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72**
0000560 **69 6e 74 53 74 72 65 61 6d 3b** 01 00 13 6a 61 76 

**第#32项** 01 表示一个 utf8 串，00 13（19） 表示长度，是【java/io/PrintStream】
0000560 69 6e 74 53 74 72 65 61 6d 3b **01 00 13 6a 61 76**
0000600 **61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d**

**第#33项** 01 表示一个 utf8 串，00 07 表示长度，是【println】
0000620 **01 00 07 70 72 69 6e 74 6c 6e** 01 00 15 28 4c 6a

**第#34项** 01 表示一个 utf8 串，00 15（21） 表示长度，是【(Ljava/lang/String;)V】
0000620 01 00 07 70 72 69 6e 74 6c 6e **01 00 15 28 4c 6a**
0000640 **61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b**
0000660 **29 56** 00 21 00 05 00 06 00 00 00 00 00 02 00 01 

### 1.4 访问标识与继承信息 

| Flag Name      | Value  | Interpretation                                               |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | Declared public ; may be accessed from outside its package.  |
| ACC_FINAL      | 0x0010 | Declared final ; no subclasses allowed.                      |
| ACC_SUPER      | 0x0020 | Treat superclass methods specially when invoked by the *invokespecial* instruction. |
| ACC_INTERFACE  | 0x0200 | Is an interface, not a class.                                |
| ACC_ABSTRACT   | 0x0400 | Declared abstract ; must not be instantiated.                |
| ACC_SYNTHETIC  | 0x1000 | Declared synthetic; not present in the source code.          |
| ACC_ANNOTATION | 0x2000 | Declared as an annotation type.                              |
| ACC_ENUM       | 0x4000 | Declared as an enum type.                                    |

接上方的字节码继续：

21 表示该 class 是一个类，公共的
0000660 29 56 **00 21** 00 05 00 06 00 00 00 00 00 02 00 01 

05 表示根据常量池中 #5 找到本类全限定名
0000660 29 56 00 21 **00 05** 00 06 00 00 00 00 00 02 00 01 

06 表示根据常量池中 #6 找到父类全限定名
0000660 29 56 00 21 00 05 **00 06** 00 00 00 00 00 02 00 01 

表示接口的数量，本类为 0
0000660 29 56 00 21 00 05 00 06 **00 00** 00 00 00 02 00 01 

### 1.5 Field 信息 

| FieldType     | Type      | Interpretation                                               |
| ------------- | --------- | ------------------------------------------------------------ |
| B             | byte      | signed byte                                                  |
| C             | char      | Unicode character code point in the Basic Multilingual Plane, encoded with UTF-16 |
| D             | double    | double-precision floating-point value                        |
| F             | float     | single-precision floating-point value                        |
| I             | int       | integer                                                      |
| J             | long      | long integer                                                 |
| L ClassName ; | reference | an instance of class ClassName                               |
| S             | short     | signed short                                                 |
| Z             | boolean   | true or false                                                |
| [             | reference | one array dimension                                          |

接上方的字节码继续：

表示成员变量数量，本类为 0
0000660 29 56 00 21 00 05 00 06 00 00 **00 00** 00 02 00 01 

### 1.6 Method 信息 

接上方的字节码继续：

表示方法数量，本类为 2
0000660 29 56 00 21 00 05 00 06 00 00 00 00 **00 02** 00 01 

一个方法由 访问修饰符，名称，参数描述，方法属性数量，方法属性组成

第一个方法的字节码如下：

0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 **00 01**
0000700 **00 07 00 08 00 01 00 09 00 00 00 2f 00 01 00 01**
0000720 **00 00 00 05 2a b7 00 01 b1 00 00 00 02 00 0a 00**
0000740 **00 00 06 00 01 00 00 00 04 00 0b 00 00 00 0c 00**
0000760 **01 00 00 00 05 00 0c 00 0d 00 00** 00 09 00 0e 00 

上述字节码，按照顺序解释如下：

- 00 01代表访问修饰符（本类中是 public）
- 00 07代表引用了常量池 #07 项作为方法名称 
- 00 08代表引用了常量池 #08 项作为方法参数描述 
- 00 01代表方法属性数量，本方法是 1 
- 方法属性 
  - 00 09 表示引用了常量池 #09 项，发现是【Code】属性
  - 00 00 00 2f 表示此属性的长度是 47  
  - 00 01 表示【操作数栈】最大深度 
  - 00 01 表示【局部变量表】最大槽（slot）数 
  - 00 00 00 05 表示字节码程度，本例是 5
  - 2a b7 00 01 b1 是字节码指令 
  - 00 00 00 02 表示方法细节属性数量，本例是 2 
  - 00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性
    - 00 00 00 06 表示此属性的总长度，本例是 6
    - 00 01 表示【LineNumberTable】长度
    - 00 00 表示【字节码】行号 00 04 表示【java 源码】行号
  - 00 0b 表示引用了常量池 #11 项，发现是【LocalVariableTable】属性
    - 00 00 00 0c 表示此属性的总长度，本例是 12
    - 00 01 表示【LocalVariableTable】长度
    - 00 00 表示局部变量生命周期开始，相对于字节码的偏移量
    - 00 05 表示局部变量覆盖的范围长度
    - 00 0c 表示局部变量名称，本例引用了常量池 #12 项，是【this】
    - 00 0d 表示局部变量的类型，本例引用了常量池 #13 项，是【Lcn/itcast/jvm/t5/HelloWorld;】
    - 00 00 表示局部变量占有的槽位（slot）编号，本例是 0 

第二个方法的字节码如下：

0000760 01 00 00 00 05 00 0c 00 0d 00 00 **00 09 00 0e 00**
0001000 **0f 00 02 00 09 00 00 00 37 00 02 00 01 00 00 00**
0001020 **09 b2 00 02 12 03 b6 00 04 b1 00 00 00 02 00 0a**
0001040 **00 00 00 0a 00 02 00 00 00 06 00 08 00 07 00 0b**
0001060 **00 00 00 0c 00 01 00 00 00 09 00 10 00 11 00 00** 
0001100 **00 12 00 00 00 05 01 00 10 00 00** 00 01 00 13 00

* 00 09 代表访问修饰符（本类中是 public static）
* 00 0e 代表引用了常量池 #14 项作为方法名称
* 00 0f 代表引用了常量池 #15 项作为方法参数描述
* 00 02 代表方法属性数量，本方法是 2
* 方法属性（属性1）
  * 00 09 表示引用了常量池 #09 项，发现是【Code】属性
  * 00 00 00 37 表示此属性的长度是 55
  * 00 02 表示【操作数栈】最大深度
  * 00 01 表示【局部变量表】最大槽（slot）数
  * 00 00 00 05 表示字节码长度，本例是 9
  * b2 00 02 12 03 b6 00 04 b1 是字节码指令
  * 00 00 00 02 表示方法细节属性数量，本例是 2
  * 00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性
    * 00 00 00 0a 表示此属性的总长度，本例是 10
    * 00 02 表示【LineNumberTable】长度
    * 00 00 表示【字节码】行号 00 06 表示【java 源码】行号
    * 00 08 表示【字节码】行号 00 07 表示【java 源码】行号
  * 00 0b 表示引用了常量池 #11 项，发现是【LocalVariableTable】属性
    * 00 00 00 0c 表示此属性的总长度，本例是 12
    * 00 01 表示【LocalVariableTable】长度 
    * 00 00 表示局部变量生命周期开始，相对于字节码的偏移量
    * 00 09 表示局部变量覆盖的范围长度
    * 00 10 表示局部变量名称，本例引用了常量池 #16 项，是【args】
    * 00 11 表示局部变量的类型，本例引用了常量池 #17 项，是【[Ljava/lang/String;】
    * 00 00 表示局部变量占有的槽位（slot）编号，本例是 0 

* 方法属性（属性2）
  * 00 12 表示引用了常量池 #18 项，发现是【MethodParameters】属性
    * 00 00 00 05 表示此属性的总长度，本例是 5
    * 01 参数数量
    * 00 10 表示引用了常量池 #16 项，是【args】
    * 00 00 访问修饰符 

### 1.7 附加属性 

接上方字节码：

0001100 00 12 00 00 00 05 01 00 10 00 00 **00 01 00 13 00**
0001120 **00 00 02 00 14** 

* 00 01 表示附加属性数量
* 00 13 表示引用了常量池 #19 项，即【SourceFile】
* 00 00 00 02 表示此属性的长度
* 00 14 表示引用了常量池 #20 项，即【HelloWorld.java】 

参考文献 

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html 

## 2. 字节码指令 

### 2.1 入门 

接着上一节，研究一下两组字节码指令，一个是

`public cn.itcast.jvm.t5.HelloWorld();` 构造方法的字节码指令 

```
2a b7 00 01 b1
```

1. `2a => aload_0` 加载 `slot 0` 的局部变量，即 `this`，做为下面的 `invokespecial` 构造方法调用的参数
2. `b7 => invokespecial` 预备调用构造方法，哪个方法呢？
3. 00 01 引用常量池中 #1 项，即【 `Method java/lang/Object."<init>":()V` 】
4. b1 表示返回 

另一个是 `public static void main(java.lang.String[]);` 主方法的字节码指令 

```
b2 00 02 12 03 b6 00 04 b1
```

1. `b2 => getstatic` 用来加载静态变量，哪个静态变量呢？
2. 00 02 引用常量池中 #2 项，即【`Field java/lang/System.out:Ljava/io/PrintStream;`】
3. `12 => ldc` 加载参数，哪个参数呢？
4. 03 引用常量池中 #3 项，即 【`String hello world`】
5. `b6 => invokevirtual` 预备调用成员方法，哪个方法呢？
6. 00 04 引用常量池中 #4 项，即【`Method java/io/PrintStream.println:(Ljava/lang/String;)V`】
7. b1 表示返回 

请参考：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5 

### 2.2 javap 工具 

自己分析类文件结构太麻烦了，Oracle 提供了 javap 工具来反编译 class 文件

```java
C:\...>javap -v HelloWorld.class
Classfile /C:/.../HelloWorld.class
  Last modified 2020-9-9; size 547 bytes         // 最后修改时间  文件大小
  MD5 checksum 72b642df992eec6e9c33e4a8b09d4022  // MD5的签名
  Compiled from "HelloWorld.java"  // 对应的原文件
public class cn.xyc.HelloWorld     // 类的全路径名称
  minor version: 0
  major version: 52                // 52 -> JDK8
  flags: ACC_PUBLIC, ACC_SUPER     // 访问修饰符
Constant pool:  // 常量池
   #1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #23            // hello world
   #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #26            // cn/xyc/HelloWorld
   #6 = Class              #27            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcn/xyc/HelloWorld;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               SourceFile
  #19 = Utf8               HelloWorld.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = Class              #28            // java/lang/System
  #22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #23 = Utf8               hello world
  #24 = Class              #31            // java/io/PrintStream
  #25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #26 = Utf8               cn/xyc/HelloWorld
  #27 = Utf8               java/lang/Object
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{  // 方法信息
  public cn.xyc.HelloWorld();  // 构造方法
    descriptor: ()V            // 构造方法参数信息
    flags: ACC_PUBLIC          // 访问修饰符
    Code:                      // 具体代码
      stack=1, locals=1, args_size=1  // 栈深度 局部变量表长度 参数长度
         0: aload_0			   // 将第一个引用类型本地变量推送至栈顶
         1: invokespecial #1   // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:      // 局部变量表
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcn/xyc/HelloWorld;

  public static void main(java.lang.String[]);  // 主方法
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2    // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3    // String hello world
         5: invokevirtual #4    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
}
SourceFile: "HelloWorld.java"
```

### 2.3 图解方法执行流程 

#### 1）原始 java 代码 

```java
/**
 * 演示 字节码指令 和 操作数栈、常量池的关系
 */
public class Demo3_1 {
    public static void main(String[] args) {
        int a = 10;
        int b = Short.MAX_VALUE + 1;
        int c = a + b;
        System.out.println(c);
    }
}
```

#### 2）编译后的字节码文件 

```java
C:\...>javap -v Demo3_1.class
Classfile /C:/.../Demo3_1.class
  Last modified 2020-9-13; size 597 bytes
  MD5 checksum bb8d90abbce7a3caa70d1f589389774f
  Compiled from "Demo3_1.java"
public class cn.xyc.Demo3_1
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#25         // java/lang/Object."<init>":()V
   #2 = Class              #26            // java/lang/Short
   #3 = Integer            32768
   #4 = Fieldref           #27.#28        // java/lang/System.out:Ljava/io/PrintStream;
   #5 = Methodref          #29.#30        // java/io/PrintStream.println:(I)V
   #6 = Class              #31            // cn/xyc/Demo3_1
   #7 = Class              #32            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               LocalVariableTable
  #13 = Utf8               this
  #14 = Utf8               Lcn/xyc/Demo3_1;
  #15 = Utf8               main
  #16 = Utf8               ([Ljava/lang/String;)V
  #17 = Utf8               args
  #18 = Utf8               [Ljava/lang/String;
  #19 = Utf8               a
  #20 = Utf8               I
  #21 = Utf8               b
  #22 = Utf8               c
  #23 = Utf8               SourceFile
  #24 = Utf8               Demo3_1.java
  #25 = NameAndType        #8:#9          // "<init>":()V
  #26 = Utf8               java/lang/Short
  #27 = Class              #33            // java/lang/System
  #28 = NameAndType        #34:#35        // out:Ljava/io/PrintStream;
  #29 = Class              #36            // java/io/PrintStream
  #30 = NameAndType        #37:#38        // println:(I)V
  #31 = Utf8               cn/xyc/Demo3_1
  #32 = Utf8               java/lang/Object
  #33 = Utf8               java/lang/System
  #34 = Utf8               out
  #35 = Utf8               Ljava/io/PrintStream;
  #36 = Utf8               java/io/PrintStream
  #37 = Utf8               println
  #38 = Utf8               (I)V
{
  public cn.xyc.Demo3_1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcn/xyc/Demo3_1;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        10  // 常数10入栈-->该值不是从常量池中取来的
         2: istore_1		  // 存入第一个slot_1中
         3: ldc           #3  // int 32768
         5: istore_2  	      // 存入第二个slot_2中
         6: iload_1		      // 加载slot_1中的值到操作栈中
         7: iload_2		      // 加载slot_2中的值到操作栈中
         8: iadd
         9: istore_3		  // 存入第三个slot_3中
        10: getstatic     #4  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: iload_3			  // 加载slot_3中的值到操作栈中
        14: invokevirtual #5  // Method java/io/PrintStream.println:(I)V
        17: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 10
        line 9: 17
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      18     0  args   [Ljava/lang/String;
            3      15     1     a   I
            6      12     2     b   I
           10       8     3     c   I
}
SourceFile: "Demo3_1.java"
```

#### 3）常量池载入运行时常量池

![常量池载入运行时常量池](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623282.PNG)

可见 `int a=10` 这些小的数组并不是存储运行时常量池中的，而是与方法的字节码存在一起；

但一旦**数值超过了Short的最大值**，`Short.MAX_VALUE + 1;`，值就存在常量池中了

#### 4）方法字节码载入方法区 

![方法字节码载入方法区](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623938.PNG)

#### 5）main 线程开始运行，分配栈帧内存 

![main线程开始运行，分配栈帧内存](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623312.PNG)

> 绿色：局部变量表，对应字节码中 locals=4，分别存放args，a，b，c
>
> 浅蓝色：操作数栈，stack=2

#### 6）执行引擎开始执行字节码 

##### bipush 10

* 将一个 byte 压入操作数栈（其长度会补齐 4 个字节），类似的指令还有：
  * sipush 将一个 short 压入操作数栈（其长度会补齐 4 个字节）
  * ldc 将一个 int 压入操作数栈
  * ldc2_w 将一个 long 压入操作数栈（分两次压入，因为 long 是 8 个字节）
* 这里小的数字都是和字节码指令存在一起，超过 short 范围的数字存入了常量池 

![bipush10](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623580.PNG)

##### istore_1 

* 将操作数栈顶数据弹出，存入局部变量表的 slot 1 

![istore-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623702.PNG)

##### ldc #3

* 从常量池加载 #3 数据到操作数栈
* **注意** Short.MAX_VALUE 是 32767，所以 32768 = Short.MAX_VALUE + 1 实际是在编译期间计算好的 

![ldc](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623301.PNG)

##### istore_2

![istore-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623646.PNG)

##### iload_1

![iload-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623041.PNG)

##### iload_2

![iload-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623658.PNG)

##### iadd

![iadd](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623295.PNG)

##### istore_3

![istore-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623968.PNG)

##### getstatic #4

![getstatic](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251623510.PNG)

##### iload_3

![iload-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624195.PNG)

##### invokevirtual #5

* 找到常量池 #5 项
* 定位到方法区 java/io/PrintStream.println:(I)V 方法
* 生成新的栈帧（分配 locals、stack等）
* 传递参数，执行新栈帧中的字节码 

![invokevirtual](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624387.PNG)

* 执行完毕，弹出栈帧
* 清除 main 操作数栈内容 

![invokevirtual2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624999.PNG)

##### return

* 完成 main 方法调用，弹出 main 栈帧
* 程序结束 

### 2.4 练习 - 分析 i++ 

目的：从字节码角度分析 a++ 相关题目 

源码： 

```java
/**
 * 从字节码角度分析　a++  相关题目
 */
public class Demo3_2 {
    public static void main(String[] args) {
        int a = 10;
        int b = a++ + ++a + a--;
        System.out.println(a);  // 11
        System.out.println(b);  // 34
    }
}
```

字节码：

```java
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: bipush        10
         2: istore_1
         3: iload_1
         4: iinc          1, 1
         7: iinc          1, 1
        10: iload_1
        11: iadd
        12: iload_1
        13: iinc          1, -1
        16: iadd
        17: istore_2      // 下方分析到此
        18: getstatic     #2  // Field java/lang/System.out:Ljava/io/PrintStream;
        21: iload_1
        22: invokevirtual #3  // Method java/io/PrintStream.println:(I)V
        25: getstatic     #2  // Field java/lang/System.out:Ljava/io/PrintStream;
        28: iload_2
        29: invokevirtual #3  // Method java/io/PrintStream.println:(I)V
        32: return
      LineNumberTable:
        line 6: 0
        line 7: 3
        line 8: 18
        line 9: 25
        line 10: 32
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      33     0  args   [Ljava/lang/String;
            3      30     1     a   I
           18      15     2     b   I
```

分析：

* **注意 iinc 指令是直接在局部变量 slot 上进行运算**
* a++ 和 ++a 的区别是先执行 iload 还是 先执行 iinc 

![iinc](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624307.PNG)

### 2.5 条件判断指令 

| 指令 | 助记符    | 含义             |
| ---- | --------- | ---------------- |
| 0x99 | ifeq      | 判断是否 == 0    |
| 0x9a | ifne      | 判断是否 != 0    |
| 0x9b | iflt      | 判断是否 < 0     |
| 0x9c | ifge      | 判断是否 >= 0    |
| 0x9d | ifgt      | 判断是否 > 0     |
| 0x9e | ifle      | 判断是否 <= 0    |
| 0x9f | if_icmpeq | 两个int是否 ==   |
| 0xa0 | if_icmpne | 两个int是否 !=   |
| 0xa1 | if_icmplt | 两个int是否 <    |
| 0xa2 | if_icmpge | 两个int是否 >=   |
| 0xa3 | if_icmpgt | 两个int是否 >    |
| 0xa4 | if_icmple | 两个int是否 <=   |
| 0xa5 | if_acmpeq | 两个引用是否 ==  |
| 0xa6 | if_acmpne | 两个引用是否 !=  |
| 0xc6 | ifnull    | 判断是否 == null |
| 0xc7 | ifnonnull | 判断是否 != null |

几点说明：

* **byte，short，char 都会按 int 比较，因为操作数栈都是 4 字节**
* goto 用来进行跳转到指定行号的字节码 

源码：

```java
public class Demo3_3 {
    public static void main(String[] args) {
        int a = 0;
        if(a == 0) {
            a = 10;
        } else {
            a = 20;
        }
    }
}
```

字节码：

```java
Code:
      stack=1, locals=2, args_size=1
         0: iconst_0             // 分配了0
         1: istore_1             // 把0存入a变量中
         2: iload_1				 // 把a加载进入栈中进行后续比较
         3: ifne          12     // 栈中的数不等于0？，若是则跳至12
         6: bipush        10     // 若等于0，则将10放入栈
         8: istore_1			 // 将10赋值给a
         9: goto          15     // 跳转到15行
        12: bipush        20     // 将20放入栈
        14: istore_1             // 将20赋值给a
        15: return
```

> 思考
>
> 细心的同学应当注意到，以上比较指令中没有 long，float，double 的比较，那么它们要比较怎么办？
>
> 参考:https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.lcmp 

### 2.6 循环控制指令

其实循环控制还是前面介绍的那些指令，例如 while 循环：

```java
public class Demo3_4 {
    public static void main(String[] args) {
        int a = 0;
        while (a < 10) {
            a++;
        }
    }
}
```

字节码：

```java
Code:
  stack=2, locals=2, args_size=1
     0: iconst_0             // 常量0
     1: istore_1             // 赋值给a 
     2: iload_1              // 把a加载进操作栈中
     3: bipush        10     // 把10加载进操作栈中
     5: if_icmpge     14     // 栈中，两个整数进行比较
     8: iinc          1, 1   // 自增
    11: goto          2      // 调回第2行代码
    14: return
```

再比如 do while 循环： 

```java
public class Demo3_5 {
    public static void main(String[] args) {
        int a = 0;
        do {
            a++;
        } while (a < 10);
    }
}
```

字节码：

```java
Code:
  stack=2, locals=2, args_size=1
     0: iconst_0
     1: istore_1
     2: iinc          1, 1  // 先自增
     5: iload_1             // 把自增后的值放入栈
     6: bipush        10    // 把10放入栈
     8: if_icmplt     2     // 进行比较
    11: return
```

最后再看看 for 循环： 

```java
public class Demo3_6 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
        }
    }
}
```

字节码：

```java
Code:
  stack=2, locals=2, args_size=1
     0: iconst_0
     1: istore_1
     2: iload_1
     3: bipush        10
     5: if_icmpge     14
     8: iinc          1, 1
    11: goto          2
    14: return
```

> 注意
>
> 比较 while 和 for 的字节码，你发现它们是一模一样的，殊途也能同归 

### 2.7 练习 - 判断结果 

请从字节码角度分析，下列代码运行的结果： 

```java
public class Demo3_6_1 {
    public static void main(String[] args) {
        int i = 0;
        int x = 0;
        while (i < 10) {
            x = x++;
            i++;
        } 
        System.out.println(x); // 结果是 0
    }
}
```

字节码：只与x有关的

```java
2: iconst_0
3: istore_2
10: iload_2             // 把x的值放入栈中，x此时为0
11: iinc          2, 1  // x自增，这个自增在slot中自增的
14: istore_2            // 赋值操作，把栈中的x值赋值给x，x被覆盖了
```

### 2.8 构造方法 

#### 1）`<cinit>()V`

```java
public class Demo3_8_1 {
    static int i = 10;
    static {
        i = 20;
    } 
    static {
        i = 30;
    }
}
```

编译器会按从上至下的顺序，收集所有 static 静态代码块和静态成员赋值的代码，合并为一个特殊的方法 `<cinit>()V` ： 

```java
0: bipush 10    	// 把10放入栈中   
2: putstatic #2 	// Field i:I，常量池中找到i变量
5: bipush 20		// 把20放入栈中
7: putstatic #2 	// Field i:I
10: bipush 30		// 把20放入栈中
12: putstatic #2 	// Field i:I
15: return
```

`<cinit>()V` 方法会在类加载的初始化阶段被调用 

#### 2）`<init>()V`

```java
public class Demo3_8_2 {
    private String a = "s1";
    
    {
        b = 20;
    } 
    
    private int b = 10;
    
    {
        a = "s2";
    } 
    
    public Demo3_8_2(String a, int b) {
        this.a = a;
        this.b = b;
    }
    
    public static void main(String[] args){
        Demo3_8_2 d = new Demo3_8_2("s3", 30);
        System.out.println(d.a);  // "s3"
        System.out.println(d.b);  // 30
    }
}
```

编译器会按从上至下的顺序，收集所有 {} 代码块和成员变量赋值的代码，形成新的构造方法，但原始构造方法内的代码总是在最后 

```java
Code:
   stack=2, locals=3, args_size=3
       0: aload_0			 // this
       1: invokespecial #1   // super.<init>()V
       4: aload_0
       5: ldc #2 			 // <- "s1"
       7: putfield #3 		 // -> this.a
       10: aload_0
       11: bipush 20 		 // <- 20
       13: putfield #4 		 // -> this.b
       16: aload_0
       17: bipush 10		 // <- 10
       19: putfield #4		 // -> this.b
       22: aload_0
       23: ldc #5			 // <- "s2"
       25: putfield #3 		 // -> this.a
       28: aload_0 			 // ------------------------------
       29: aload_1 			 // <- slot_1(a) "s3" 			 |
       30: putfield #3		 // -> this.a 				     |
       33: aload_0			 //								 |
       34: iload_2			 // <- slot 2(b) 30 			 |
       35: putfield #4		 // -> this.b --------------------
       38: return
```

### 2.9 方法调用

```java
public class Demo3_9 {
    public Demo3_9() { }

    private void test1() { }

    private final void test2() { }

    public void test3() { }

    public static void test4() { }

    @Override
    public String toString() {
        return super.toString();
    }

    public static void main(String[] args) {
        Demo3_9 d = new Demo3_9();
        d.test1();
        d.test2();
        d.test3();
        d.test4();
        Demo3_9.test4();
        d.toString();
    }
}
```

字节码：

```java
Code:
  stack=2, locals=2, args_size=1
     0: new           #3    // class cn/xyc/Demo3_9
     3: dup					// 复制一份引用
     4: invokespecial #4    // Method "<init>":()V 构造方法，调用完成后，引用被消耗
     7: astore_1            // 把对象引用赋值给d
     8: aload_1             // 把对象引用加载至栈中
     9: invokespecial #5    // Method test1:()V  私有方法
    12: aload_1
    13: invokespecial #6    // Method test2:()V  final方法
    16: aload_1
    17: invokevirtual #7    // Method test3:()V  普通方法-->运行时确定，动态绑定
    20: aload_1
    21: pop                 // 由于静态方法的调用不需要对象的引用
    22: invokestatic  #8    // Method test4:()V  静态方法
    25: invokestatic  #8    // Method test4:()V  静态方法
    28: aload_1
    29: invokevirtual #9    // Method toString:()Ljava/lang/String;
    32: pop
    33: return
```

* new 是创建【对象】，给对象分配堆内存，执行成功会将【对象引用】压入操作数栈
* dup 是复制操作数栈栈顶的内容，本例即为【对象引用】，为什么需要两份引用呢，一个是要配合 invokespecial 调用该对象的构造方法 `"<init>":()V` （会消耗掉栈顶一个引用），另一个要配合 astore_1 赋值给局部变量
* 最终方法（final），私有方法（private），构造方法都是由 invokespecial 指令来调用，属于**静态绑定**
* 普通成员方法是由 invokevirtual 调用，属于**动态绑定**，即支持多态
* 成员方法与静态方法调用的另一个区别是，执行方法前是否需要【对象引用】
* 比较有意思的是 d.test4(); 是通过【对象引用】调用一个静态方法，可以看到在调用invokestatic 之前执行了 pop 指令，把【对象引用】从操作数栈弹掉了
* 还有一个执行 invokespecial 的情况是通过 super 调用父类方法 

### 2.10 多态的原理 

```java
/**
 * 演示多态原理，注意加上下面的 JVM 参数，禁用指针压缩
 * -XX:-UseCompressedOops -XX:-UseCompressedClassPointers
 */
public class Demo3_10 {

    public static void test(Animal animal) {
        animal.eat();
        System.out.println(animal.toString());
    }

    public static void main(String[] args) throws IOException {
        test(new Cat());
        test(new Dog());
        System.in.read();
    }
}

abstract class Animal {
    public abstract void eat();

    @Override
    public String toString() {
        return "我是" + this.getClass().getSimpleName();
    }
}

class Dog extends Animal {

    @Override
    public void eat() {
        System.out.println("啃骨头");
    }
}

class Cat extends Animal {

    @Override
    public void eat() {
        System.out.println("吃鱼");
    }
}
```

#### 1）运行代码

停在 `System.in.read()` 方法上，这时运行 jps 获取进程 id

#### 2）运行 HSDB 工具 

进入 JDK 安装目录，执行 

```bash
cd C:\Software\Java\jdk1.8.0_241
java -cp ./lib/sa-jdi.jar sun.jvm.hotspot.HSDB
```

进入图形界面 `File -> Attach To HotSport process -> 输入进程 id`

#### 3）查找某个对象

打开 `Tools -> Find Object By Query `寻找对象

输入 `select d from cn.xyc.Dog d` 点击 Execute 执行 

![HSDB工具](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624573.PNG)

#### 4）查看对象内存结构 

点击超链接可以看到对象的内存结构，此对象没有任何属性，因此只有对象头的 16 字节，前 8 字节是 MarkWord，后 8 字节就是对象的 Class 指针 

但目前看不到它的实际地址 

![HSDB工具2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624101.PNG)

#### 5）查看对象Class的内存地址

可以通过 `Windows -> Console` 进入命令行模式，执行 

```
mem 0x000000012a19b1e0 2
```

mem 有两个参数，参数 1 是对象地址，参数 2 是查看 2 行（即 16 字节）

结果中第一行为对象的MarkWord

结果中第二行 0x000000001bc04028 即为 Class 的内存地址 

![HSDB工具3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624903.PNG)

#### 6）查看类的 vtable

vtable：虚方法表

**方法1**：`Tool -> inspector`进入 Inspector 工具，输入刚才的 Class 内存地址，看到如下界面 

![HSDB工具4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624202.PNG)

**方法2**：或者 `Tools -> Class Browser` 输入 Dog 查找，可以得到相同的结果 

![HSDB工具5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624586.PNG)

无论通过哪种方法，都可以找到 Dog Class 的 vtable 长度为 6，意思就是 Dog 类有 6 个虚方法（多态相关的，final，static 不会列入）

那么这 6 个方法都是谁呢？从 Class 的起始地址开始算，偏移 0x1b8 就是 vtable 的起始地址，进行计算得到： 

```
0x000000001bc04028
			 1b8 +
---------------------
0x000000001bc041e0
```

通过 Windows -> Console 进入命令行模式，执行 

```
hsdb> mem 0x000000001bc041e0 6
0x000000001bc041e0: 0x000000001b801b10 
0x000000001bc041e8: 0x000000001b8015e8 
0x000000001bc041f0: 0x000000001bc035e8 
0x000000001bc041f8: 0x000000001b801540 
0x000000001bc04200: 0x000000001b801678 
0x000000001bc04208: 0x000000001bc03fa8 
```

就得到了 6 个虚方法的入口地址，因为上方显示了vtable中有6个虚方法

#### 7）验证方法地址 

通过 `Tools -> Class Browser` 查看每个类的方法定义，比较可知 

```
Dog    --> public void eat() @0x000000001bc03fa8;
Animal --> public java.lang.String toString() @0x000000001bc035e8;
Object --> protected void finalize() @0x000000001b801b10;
Object --> public boolean equals(java.lang.Object) @0x000000001b8015e8;
Object --> public native int hashCode() @0x000000001b801540;
Object --> protected native java.lang.Object clone() @0x000000001b801678;
```

对号入座，发现：

eat() 方法是 Dog 类自己的，而Animal中的eat方法为：`public abstract void eat() @0x000000001bc03530;`

![HSDB工具6](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251624271.PNG)

#### 8）小结 

当执行 invokevirtual 指令时，
1. 先通过栈帧中的对象引用找到对象
2. 分析对象头，找到对象的实际 Class
3. Class 结构中有 vtable，它在类加载的链接阶段就已经根据方法的重写规则生成好了
4. 查表得到方法的具体地址
5. 执行方法的字节码 

### 2.11 异常处理

#### try-catch 

```java
public class Demo3_11_1 {

    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (Exception e) {
            i = 20;
        }
    }
}
```

字节码：

```java
Code:
  stack=1, locals=3, args_size=1
     0: iconst_0
     1: istore_1          // 将0复制给i
     2: bipush        10  // 把10放入栈中
     4: istore_1          // 把10复制给i   即try块中的代码
     5: goto          12  // 运行到12行
     8: astore_2          // 从局部变量表中可知，astore_2->slot2->e,把异常对象存入slot2中
     9: bipush        20
    11: istore_1
    12: return
Exception table:  // 异常表
  from    to  target   type  // 监视第[2,5)行的代码  发生异常后和Exception进行匹配进入第8行
    2      5       8   Class java/lang/Exception  
LocalVariableTable:  // 局部变量表
  Start  Length  Slot  Name   Signature
      9       3     2     e   Ljava/lang/Exception;
      0      13     0  args   [Ljava/lang/String;
      2      11     1     i   I
```

* 可以看到多出来一个 **Exception table** 的结构，[from, to) 是前闭后开的检测范围，一旦这个范围内的字节码执行出现异常，则通过 type 匹配异常类型，如果一致，进入 target 所指示行号
* 8 行的字节码指令 astore_2 是将异常对象引用存入局部变量表的 slot 2 位置 

#### 多个 single-catch 块的情况 

```java
public class Demo3_11_2 {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (ArithmeticException e) {
            i = 30;
        } catch (NullPointerException e) {
            i = 40;
        } catch (Exception e) {
            i = 50;
        }
    }
}
```

字节码：

```java
Code:
  stack=1, locals=3, args_size=1
     0: iconst_0
     1: istore_1
     2: bipush        10
     4: istore_1
     5: goto          26
     8: astore_2            // 异常放到slot2中的e变量
     9: bipush        30
    11: istore_1
    12: goto          26
    15: astore_2            // 异常放到slot2中的e变量
    16: bipush        40
    18: istore_1
    19: goto          26
    22: astore_2            // 异常放到slot2中的e变量
    23: bipush        50
    25: istore_1
    26: return
  Exception table:
     from    to  target type  // target 变多了
         2     5     8   Class java/lang/ArithmeticException
         2     5    15   Class java/lang/NullPointerException
         2     5    22   Class java/lang/Exception
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
        9       3     2     e   Ljava/lang/ArithmeticException;
       16       3     2     e   Ljava/lang/NullPointerException;
       23       3     2     e   Ljava/lang/Exception;
        0      27     0  args   [Ljava/lang/String;
        2      25     1     i   I
```

因为异常出现时，只能进入 Exception table 中一个分支，所以局部变量表 slot 2 位置**被共用** 

#### multi-catch 的情况 

```java
public class Demo3_11_3 {
    public static void main(String[] args) {
        try {
            Method test = Demo3_11_3.class.getMethod("test");
            test.invoke(null);
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    public static void test() {
        System.out.println("ok");
    }
}
```

字节码：

```java
Code:
  stack=3, locals=2, args_size=1
     0: ldc           #2  // class cn/xyc/Demo3_11_3
     2: ldc           #3  // String test
     4: iconst_0
     5: anewarray     #4  // class java/lang/Class
     8: invokevirtual #5  // Method java/lang/Class.getMethod:(Ljava/lang/String;[Ljava/lang/Class;)Ljava/lang/reflect/Method;
    11: astore_1
    12: aload_1
    13: aconst_null
    14: iconst_0
    15: anewarray     #6  // class java/lang/Object
    18: invokevirtual #7  // Method java/lang/reflect/Method.invoke:(Ljava/lang/Object;[Ljava/lang/Object;)Ljava/lang/Object;
    21: pop
    22: goto          30
    25: astore_1          // 异常对象的引用存储于slot1
    26: aload_1           // 把异常对象的引用加载到操作栈
    27: invokevirtual #1  // Method java/lang/ReflectiveOperationException.printStackTrace:()V
    30: return
  Exception table:
     from    to  target  type  // 监视[0， 22)行代码，出现异常进行匹配，异常的入口相同
         0    22    25   Class java/lang/NoSuchMethodException
         0    22    25   Class java/lang/IllegalAccessException
         0    22    25   Class java/lang/reflect/InvocationTargetException
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
       12      10     1  test   Ljava/lang/reflect/Method;
       26       4     1     e   Ljava/lang/ReflectiveOperationException;
        0      31     0  args   [Ljava/lang/String;
```

#### finally 

```java
public class Demo3_11_4 {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (Exception e) {
            i = 20;
        } finally {
            i = 30;
        }
    }
}
```

字节码：

```java
Code:
  stack=1, locals=4, args_size=1
     0: iconst_0
     1: istore_1          // 0 -> i
     2: bipush        10  // try --------------------------------------
     4: istore_1          // 10 -> i                                  |
     5: bipush        30  // finally                                  |
     7: istore_1          // 30 -> i                                  |
     8: goto          27  // return -----------------------------------
    11: astore_2          // catch Exceptin -> e ----------------------
    12: bipush        20  //                                          |
    14: istore_1          // 20 -> i                                  |
    15: bipush        30  // finally                                  |
    17: istore_1		  // 30 -> i                                  |
    18: goto          27  // return -----------------------------------
    21: astore_3          // catch any -> slot 3 ----------------------
    22: bipush        30  // finally                                  |
    24: istore_1          // 30 -> i                                  |
    25: aload_3           // <- slot 3  有这个slot3                    |
    26: athrow            // throw 抛出---------------------------------
    27: return
  Exception table:
     from    to  target type
         2     5    11   Class java/lang/Exception
         2     5    21   any  // 剩余的异常类型，比如 Error，监测[2, 5)中的其他异常/error
        11    15    21   any  // 剩余的异常类型，比如 Error，监测[11, 15)->catch块中的其他异常/error
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
       12       3     2     e   Ljava/lang/Exception;
        0      28     0  args   [Ljava/lang/String;
        2      26     1     i   I
```

可以看到 finally 中的代码被复制了 3 份，分别放入 try 流程，catch 流程以及 catch 剩余的异常类型流程 

### 2.12 练习 - finally 面试题

#### finally 出现了 return 

先问问自己，下面的题目输出什么？ 

```java
public class Demo3_12_1 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);  // 20
    }

    public static int test() {
        try {
            return 10;
        } finally {
            return 20;
        }
    }
}
```

字节码：

```java
public static int test();
  descriptor: ()I
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=1, locals=2, args_size=0
       0: bipush        10  // <- 10 放入栈顶
       2: istore_0          // 10 -> slot 0(从栈顶移除了)
       3: bipush        20  // < - 20 放入栈顶
       5: ireturn           // 返回栈顶 int(20)
       6: astore_1          // catch any -> slot 1
       7: bipush        20  // <- 20 放入栈顶
       9: ireturn           // 返回栈顶 int(20)
    Exception table:
       from    to  target type
           0     3     6   any
```

* 由于 finally 中的 ireturn 被插入了所有可能的流程，因此返回结果肯定以 finally 的为准
* 至于字节码中第 2 行，似乎没啥用，且留个伏笔，看下个例子
* 跟上例中的 finally 相比，**发现没有 athrow 了**，这告诉我们：如果在 finally 中出现了 return，会吞掉异常！可以试一下下面的代码

```java
public class Demo3_12_1 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);  // 20
    }

    public static int test() {
        try {
            int i = 1/0;  // 加不加结果都一样
            return 10;
        } finally {
            return 20;
        }
    }
}
```

很危险的操作，异常被吞掉了，即使发生了异常也不知道了

#### finally 对返回值影响 

同样问问自己，下面的题目输出什么？ 

```java
public class Demo3_12_2 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);  // 10
    }

    public static int test() {
        int i = 10;
        try {
            return i;
        } finally {
            i = 20;
        }
    }
}
```

字节码：

```java
public static int test();
  descriptor: ()I
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=1, locals=3, args_size=0
       0: bipush        10  // <- 10 放入栈顶
       2: istore_0			// 10 -> i
       3: iload_0			// <- i(10)
       4: istore_1			// 10 -> slot1，暂存至slot1，目的是为了固定返回值 ☆
       5: bipush        20  // <- 20 放入栈顶
       7: istore_0          // 20 -> i
       8: iload_1		    // <- slot 1(10) 载入 slot 1 暂存的值 ☆
       9: ireturn           // 返回栈顶的 int(10)
      10: astore_2          // 期间出现异常，存储异常到slot2中
      11: bipush        20  // <- 20 放入栈顶
      13: istore_0          // 20 -> i
      14: aload_2           // 异常加载进栈
      15: athrow            // athrow出异常
    Exception table:
       from    to  target type
           3     5    10   any
```

### 2.13 synchronized 

```java
public class Demo3_13 {
    public static void main(String[] args) {
        Object lock = new Object();
        synchronized (lock) {
            System.out.println("ok");
        }
    }
}
```

字节码：

```java
public static void main(java.lang.String[]);
  descriptor: ([Ljava/lang/String;)V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=2, locals=4, args_size=1
       0: new           #2  // class java/lang/Object
       3: dup               // 将对象的引用复制两次放入栈中，在invokespecial构造方法中被消耗一次
       4: invokespecial #1  // Method java/lang/Object."<init>":()V
       7: astore_1          // 把lock引用->slot_1(lock),给lock赋值时又消耗了一次
       8: aload_1           // 把lock的引用加载到操作栈，<- lock （synchronized开始）
       9: dup
      10: astore_2          // lock引用 -> slot 2
      11: monitorenter		// monitorenter(lock引用)
      12: getstatic     #3  // Field java/lang/System.out:Ljava/io/PrintStream;
      15: ldc           #4  // String ok
      17: invokevirtual #5  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      20: aload_2           // <- slot 2(lock引用) 
      21: monitorexit		// monitorexit(lock引用)
      22: goto          30
      25: astore_3			// any -> slot 3
      26: aload_2			// <- slot 2(lock引用)
      27: monitorexit		// monitorexit(lock引用)		
      28: aload_3			// <- slot 3(lock引用)
      29: athrow			// 抛出
      30: return
    Exception table:
       from    to  target type
          12    22    25   any    // synchronized代码块内部出现问题，进入25
          25    28    25   any    // [25, 28)行代码出现问题，进入25
    LineNumberTable: ...
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      31     0  args   [Ljava/lang/String;
          8      23     1  lock   Ljava/lang/Object;
    StackMapTable: ...
```

> 注意
>
> 方法级别的 synchronized 不会在字节码指令中有所体现 
