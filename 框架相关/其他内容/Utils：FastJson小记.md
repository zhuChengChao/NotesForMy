# Utils：FastJson小记

> Json作为日常开发过程中必面对的一种数据格式，而作为能将对象和Json相互转换的类库（FastJson），日常使用也是极为频繁；
>
> > 而FastJson确总是漏洞频出，最近就接了一个升级我们应用FastJson版本的诉求，想着就系统的学习一下FastJson，总结一些日常常用的操作云云。

## 0. 常用API

> 使用FastJson的前提条件：序列化的类符合java bean规范

```xml
<!--JOSN-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version> 1.2.76</version>
</dependency>
```

fastjosn入口类是：`import com.alibaba.fastjson.JSON;`其主要用于转换，即：将Java对象序列化为JSON字符串；将JSON字符串反序列化为Java对象

其中主要的API有如下：

* `public static final Object parse(String text)`：把JSON文本反序列化为JSONObject或者JSONArray 
* `public static final JSONObject parseObject(String text)`：把JSON文本反序列化成JSONObject    
* `public static final <T> T parseObject(String text, Class<T> clazz)`：把JSON文本反序列化为JavaBean
* `public static final JSONArray parseArray(String text)`：把JSON文本反序列化成JSONArray 
* `public static final <T> List<T> parseArray(String text, Class<T> clazz)`：把JSON文本反序列化成JavaBean集合 
* `public static final String toJSONString(Object object)`：将JavaBean序列化为JSON文本 
* `public static final String toJSONString(Object object, boolean prettyFormat)`：将JavaBean序列化为带格式的JSON文本 
* `public static final Object toJSON(Object javaObject)`：将JavaBean转换为JSONObject或者JSONArray。

使用案例：https://github.com/alibaba/fastjson/wiki/Samples-DataBind

> 有关类库的一些说明：
>
> * SerializeWriter：相当于StringBuffer
>
> * JSONArray：相当于`List<Object>`
>
> * JSONObject：相当于`Map<String, Object>`，可以使用get方法来获取相应的属性
>
> * JSON反序列化没有真正数组，本质类型都是`List<Object>`

## 1. 进一步说明

### 1.1 JSON/JSONObject/JSONArray类关系

`fastjson`的使用主要是三个对象：**JSON/JSONObject/JSONArray**，其中的类关系如下：

![JSON类关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207021132711.png)

**JSONObject**：JSON对象(JSONObject)中的数据都是以`key-value`形式出现，所以它实现了`Map`接口：

![JSONObject对象关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207021134364.png)

使用起来也很简单，**跟使用`Map`就没多大的区别**（因为它底层实际上就是操作`Map`)，常用的方法：

- `getString(String key)`：获取其中的属性；
- `remove(Object key)`：移除其中的属性；

**JSONArray**：是JSON数组，JSON数组对象中存储的是一个个JSON对象，所以类中的方法主要用于**直接操作JSON对象**：

![JSONArray对象](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202207021136240.png)

最常用的方法：`getJSONObject(int index)`

### 1.2 `parse()`及`parseObject()`区别

> https://blog.csdn.net/hosaos/article/details/106982555

先来看个例子：

```java
public class User implements Serializable{

    private String userName;

    public User() {
        System.out.println("call construct method");
    }

    public String getUserName() {
        System.out.println("call get method getUserName");
        return userName;
    }

    public void setUserName(String userName) {
        System.out.println("call set  method setUserName");
        this.userName = userName;
    }

}

@Test
public void test(){

    ParserConfig.getGlobalInstance().setAutoTypeSupport(true);

    String userJson = "{\"@type\":\"cn.xyc.domain.User\",\"userName\":\"testUserName\"}";

    System.out.println("===parseObject测试===");
    Object object1 = JSON.parseObject(userJson);
    System.out.println("===parse测试===");
    Object object2 = JSON.parse(userJson);

}

// 输出：
===parseObject测试===
call construct method
call set  method setUserName
call get method getUserName
===parse测试===
call construct method
call set  method setUserName
```

`parse()`及`parseObject()`进行反序列化时的细节区别在于，`parse()`会识别并调用目标类的 setter 方法，而 `parseObject()` 由于要将返回值转化为JSONObject，多执行了` JSON.toJSON(obj)`，**所以在处理过程中会调用反序列化目标类的getter方法**来将参数赋值给JSONObject

### 1.3 序列化&反序列化

fastjson的主要功能就是将Java Bean序列化成JSON字符串，这样得到字符串之后就可以通过数据库等方式进行持久化了；但是，fastjson在序列化以及反序列化的过程中**并没有使用Java自带的序列化机制，而是自定义了一套机制**。

其实，对于JSON框架来说，想要把一个Java对象转换成字符串，可以有两种选择：

1. 基于属性；
2. 基于setter/getter

而我们所常用的JSON序列化框架中，FastJson和jackson在把对象序列化成json字符串的时候，是通过遍历出该类中的所有getter方法进行的。

> Gson并不是这么做的，他是通过反射遍历该类中的所有属性，并把其值序列化成json。

```java
@Data
public class JsonDomain implements Serializable{

    private String field1;

    // 不存在的属性
    public String getField2() {
        return "field2";
    }

    public void setField2(String field2) {
        return;
    }
}

@Test
public void test(){

    JsonDomain jsonDomain = new JsonDomain();
    jsonDomain.setField1("1");
    jsonDomain.setField2("2");

    String str = JSON.toJSONString(jsonDomain);
    System.out.println(str);
    // {"field1":"1","field2":"field2"}  -- 可以看到有两个属性
}
```

## 2. SerializerFeature属性

如果我们希望我们的JSON对象在序列化和反序列化有一些特殊的要求，例如：使用单引号而不是双引号，此时可以通过配置来达到。

> 这里就介绍两个属性，其他属性不做过多介绍，可见：https://blog.csdn.net/u010246789/article/details/52539576

* `SerializerFeature.QuoteFieldNames`：输出key时是否使用双引号，默认为true

* `SerializerFeature.UseSingleQuotes`：使用单引号而不是双引号，默认为false

* `SerializerFeature.WriteMapNullValue`：是否输出值为null的字段，默认为false

* `SerializerFeature.WriteClassName`：即在序列化的时候，把原始类型记录下来，**但会引起风险**

  > 避免可见：https://github.com/alibaba/fastjson/wiki/fastjson_safemode

```java
@Data
public class User implements Serializable{
    private String userName;
}

@Test
public void test(){

    User user1 = new User();
    user1.setUserName("zhangsan");
    // 默认输出
    System.out.println(JSON.toJSONString(user1)); 
    // 添加属性：SerializerFeature.UseSingleQuotes
    System.out.println(JSON.toJSONString(user1, SerializerFeature.UseSingleQuotes));
    User user2 = new User();
    // 默认输出
    System.out.println(JSON.toJSONString(user2));
    // 添加属性：SerializerFeature.WriteMapNullValue
    System.out.println(JSON.toJSONString(user2, SerializerFeature.WriteMapNullValue));
}

// 输出结果：
{"userName":"zhangsan"}
{'userName':'zhangsan'}
{}
{"userName":null}
```

## 3. AutoType & SafeMode

Fastjson提供了autotype功能，**允许用户在反序列化数据中通过“@type”指定反序列化的Class类型**。

> 直接看以下两篇博文的介绍

[什么是FastJson中AutoType反序列化漏洞?](https://blog.csdn.net/hosaos/article/details/106982555)

[初探fastJson的AutoType](https://blog.csdn.net/qq_39208832/article/details/117233363)

## 4. 参考

https://www.w3cschool.cn/fastjson/

https://blog.csdn.net/rustwei/article/details/121162202

https://zhuanlan.zhihu.com/p/97222262

