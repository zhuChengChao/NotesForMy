# 中间件：RabbitMQ学习笔记-2

> 很早之前学过 RabbitMQ，做学习项目时学的，学的很粗糙。这次实习过程中再次用到了消息队列，用的是公司内部的消息队列的实现，从使用层面来讲的话很容易上手。然后抽空就再重新找了波课程重新学习了一下，课程地址：[黑马程序员RabbitMQ全套教程](https://www.bilibili.com/video/BV15k4y1k7Ep)，同样跟着视频同步做了些笔记，分为上下两篇，此为 1/2 篇。

## 6. 死信队列  

### 6.1 死信的概念

先从概念解释上搞清楚这个定义，死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说， producer 将消息投递到 broker 或者直接到queue 里了， consumer 从 queue 取出消息进行消费，但某些时候由于特定的原因**导致 queue 中的某些消息无法被消费**，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。

应用场景：为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中。还有比如说：用户在商城下单成功并点击去支付后在指定时间未支付时自动失效  

### 6.2 死信的来源

消息 TTL 过期

队列达到最大长度(队列满了，无法再添加数据到 MQ 中)

消息被拒绝（basic.reject 或 basic.nack）并且 requeue=false.  

### 6.3 死信实战

#### 6.3.1 代码架构图  

![死信队列实战架构](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357519.PNG)

#### 6.3.2 消息 TTL 过期

生产者代码

```java
package cn.xyc.rabbitmq.eight;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

public class Producer {

    private static final String NORMAL_EXCHANGE = "normal_exchange";
    private static final String NORMAL_QUEUE = "normal_queue";


    public static void main(String[] args) throws Exception{

        Channel channel = RabbitMqUtils.getChannel();
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 设置消息的 TTL 时间
        AMQP.BasicProperties basicProperties = new AMQP.BasicProperties().builder().expiration("10000").build();
        // 该信息是用作演示队列个数限制
        for (int i = 1; i <= 10; i++) {
            String message = "info"+i;
            channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", basicProperties, message.getBytes());
            System.out.println("生产者发送消息:"+message);
        }
    }
}
```

消费者 C1 代码（启动之后关闭该消费者 模拟其接收不到消息）

```java
package cn.xyc.rabbitmq.eight;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class Consumer01 {

    // 普通交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    // 死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";
    // 普通队列名称
    private static final String NORMAL_QUEUE = "noraml_queue";
    // 死信队列名称
    private static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();
        // 声明死信和普通交换机 类型为 direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
        // 死信队列绑定死信交换机与 routingkey
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        // 正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        // 正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, params);
        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");

        System.out.println("等待接收消息........... ");
        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                String str = new String(message.getBody(), "UTF-8");
                System.out.println("Consumer01收到消息: " + str);
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            }

        };
        channel.basicConsume(NORMAL_QUEUE, true, deliverCallback, consumerTag -> { });
    }
}
```

消费者 C2 代码（以上步骤完成后 启动 C2 消费者 它消费死信队列里面的消息） 

```java
package cn.xyc.rabbitmq.eight;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;
import com.rabbitmq.client.Delivery;

import java.io.IOException;

public class Consumer02 {

    // 死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";
    // 死信队列名称
    private static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        System.out.println("等待接收死信队列消息........... ");
        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                String str = new String(message.getBody(), "UTF-8");
                System.out.println("Consumer01收到消息: " + str);
            }

        };
        channel.basicConsume(DEAD_QUEUE, true, deliverCallback, consumerTag -> {});

    }
}
```

生产者未发送消息：

![死信队列-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357163.PNG)

生产者发送10条消息，此时正常消息队列中存在10条数据

![死信队列-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357761.PNG)

过了10s后，正常队列中的消息没有被消费，消息进入死信队列中

![死信队列-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357809.PNG)

启动消费者 C2 消费死信队列中的消息

```bash
等待接收死信队列消息........... 
Consumer01收到消息: info1
Consumer01收到消息: info2
Consumer01收到消息: info3
Consumer01收到消息: info4
Consumer01收到消息: info5
Consumer01收到消息: info6
Consumer01收到消息: info7
Consumer01收到消息: info8
Consumer01收到消息: info9
Consumer01收到消息: info10
```

#### 6.3.3 队列达到最大长度

消息生产者代码去掉 TTL 属性

```java
package cn.xyc.rabbitmq.eight;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

public class Producer {

    private static final String NORMAL_EXCHANGE = "normal_exchange";
    private static final String NORMAL_QUEUE = "normal_queue";


    public static void main(String[] args) throws Exception{

        Channel channel = RabbitMqUtils.getChannel();
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 设置消息的 TTL 时间，这里去掉了
        // AMQP.BasicProperties basicProperties = new AMQP.BasicProperties().builder().expiration("10000").build();
        //该信息是用作演示队列个数限制
        for (int i = 1; i <= 10; i++) {
            String message = "info"+i;
            // channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", basicProperties, message.getBytes());
            channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", null, message.getBytes());
            System.out.println("生产者发送消息:"+message);
        }
    }
}
```

C1 消费者修改以下代码（启动之后关闭该消费者 模拟其接收不到消息）

```java
//正常队列绑定死信队列信息
Map<String, Object> params = new HashMap<>();
//正常队列设置死信交换机 参数 key 是固定值
params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
//正常队列设置死信 routing-key 参数 key 是固定值
params.put("x-dead-letter-routing-key", "lisi");
// 设置正常队列长度的限制
params.put("x-max-length", 6);

channel.queueDeclare(NORMAL_QUEUE, false, false, false, params);
channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
```

注意此时需要把原先队列删除 因为参数改变了

C2 消费者代码不变（启动 C2 消费者）

![死信队列-4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357984.PNG)

正常队列中长度限制为 6，因此只有 6 条消息，还有 4 条放到了死信队列中

```bash
# 生产者
生产者发送消息:info1
生产者发送消息:info2
生产者发送消息:info3
生产者发送消息:info4
生产者发送消息:info5
生产者发送消息:info6
生产者发送消息:info7
生产者发送消息:info8
生产者发送消息:info9
生产者发送消息:info10

# 消费者C1
等待接收消息........... 
Consumer01收到消息: info5
Consumer01收到消息: info6
Consumer01收到消息: info7
Consumer01收到消息: info8
Consumer01收到消息: info9
Consumer01收到消息: info10

# 消费者C2
等待接收死信队列消息........... 
Consumer01收到消息: info1
Consumer01收到消息: info2
Consumer01收到消息: info3
Consumer01收到消息: info4
```

### 6.3.4 消息被拒

消息生产者代码同上生产者一致

C1 消费者代码（启动之后关闭该消费者 模拟其接收不到消息）

```java
package cn.xyc.rabbitmq.eight;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class Consumer01 {

    //普通交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    //死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";
    //普通队列名称
    private static final String NORMAL_QUEUE = "noraml_queue";
    //死信队列名称
    private static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();
        //声明死信和普通交换机 类型为 direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
        //死信队列绑定死信交换机与 routingkey
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        //正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        //正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");
        // 设置正常队列长度的限制
        // params.put("x-max-length", 6);

        channel.queueDeclare(NORMAL_QUEUE, false, false, false, params);
        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");

        System.out.println("等待接收消息........... ");
        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                String str = new String(message.getBody(), "UTF-8");

                if("info5".equals(str)){
                    System.out.println("Consumer01 接收到消息" + str + "并拒绝签收该消息");
                    // requeue 设置为 false 代表拒绝重新入队 该队列如果配置了死信交换机将发送到死信队列中
                    channel.basicReject(message.getEnvelope().getDeliveryTag(), false);

                }else{
                    System.out.println("Consumer01收到消息: " + str);
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                }
            }

        };

        // channel.basicConsume(NORMAL_QUEUE, true, deliverCallback, consumerTag -> { });
        // 需要关闭自动确认
        channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, consumerTag -> { });
    }
}
```

C2 消费者代码不变

启动消费者 1 然后再启动消费者 2  

![死信队列-5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357276.PNG)

```bash
# 生产者
生产者发送消息:info1
生产者发送消息:info2
生产者发送消息:info3
生产者发送消息:info4
生产者发送消息:info5
生产者发送消息:info6
生产者发送消息:info7
生产者发送消息:info8
生产者发送消息:info9
生产者发送消息:info10

# 消费者1
等待接收消息........... 
Consumer01收到消息: info1
Consumer01收到消息: info2
Consumer01收到消息: info3
Consumer01收到消息: info4
Consumer01 接收到消息info5并拒绝签收该消息
Consumer01收到消息: info6
Consumer01收到消息: info7
Consumer01收到消息: info8
Consumer01收到消息: info9
Consumer01收到消息: info10

# 消费者2
等待接收死信队列消息........... 
Consumer02收到消息: info5
```

## 7. 延迟队列

### 7.1 延迟队列概念

延时队列，队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理，简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列。

### 7.2 延迟队列使用场景

1. 订单在十分钟之内未支付则自动取消

2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。

3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。

4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。

5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议  

这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如：发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；看起来似乎使用定时任务，一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？如果数据量比较少，确实可以这样做，比如：对于“如果账单一周内未支付则进行自动结算” 这样的需求，如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。但对于数据量比较大，并且时效性较强的场景，如： “订单十分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。  

<img src="https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357407.PNG" alt="延迟队列使用场景" style="zoom:50%;" />

### 7.3 RabbitMQ 中的 TTL

TTL 是什么呢？ **TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的最大存活时间**，单位是毫秒。换句话说，如果一条消息设置了 TTL 属性或者消息进入了设置了 TTL 属性的队列，那么这条消息如果在 TTL 设置的时间内没有被消费，则会成为"死信"。如果同时配置了队列的 TTL 和消息的 TTL，那么较小的那个值将会被使用，有两种方式设置 TTL。

#### 7.3.1 消息设置 TTL

一种方式便是针对每条消息设置 TTL  

```java
rabbitTemplate.convertAndSend("X", "XC", message, new MessagePostProcessor() {
    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        // 设置TTL时间
        message.getMessageProperties().setExpiration(ttlTime);
        return message;
    }
});
```

#### 7.3.2 队列设置TTL

另一种是在创建队列的时候设置队列的 “x-message-ttl” 属性

```java
// 声明队列的 TTL
args.put("x-message-ttl", 40000);
return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
```

#### 7.3.3 两者的区别

如果设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃（如果配置了死信队列被丢到死信队列中)，而**第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的**，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；另外，还需要注意的一点是，如果不设置 TTL，表示消息永远不会过期，如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。

前一小节我们介绍了死信队列，刚刚又介绍了 TTL，至此利用 RabbitMQ 实现延时队列的两大要素已经集齐，接下来只需要将它们进行融合，再加入一点点调味料，延时队列就可以新鲜出炉了。想想看，延时队列，不就是想要消息延迟多久被处理吗， **TTL 则刚好能让消息在延迟多久之后成为死信，另一方面，成为死信的消息都会被投递到死信队列里**，这样只需要消费者一直消费死信队列里的消息就完事了，因为里面的消息都是希望被立即处理的消息。  

### 7.4 整合 SpringBoot

#### 7.4.1 创建项目

略

#### 7.4.2 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--RabbitMQ 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.47</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!--swagger-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
    <!--RabbitMQ 测试依赖-->
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

#### 7.4.3 修改配置文件

```properties
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

#### 7.4.4 添加Swagger 配置类

```java
package cn.xyc.mqspringboot.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket webApiConfig(){

        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                .build();
    }

    private ApiInfo webApiInfo(){
        return new ApiInfoBuilder()
                .title("rabbitmq 接口文档")
                .description("本文档描述了 rabbitmq 微服务接口定义")
                .version("1.0")
                .contact(new Contact("enjoy6288", "http://atguigu.com", "1551388580@qq.com"))
                .build();
    }
}
```

### 7.5 队列 TTL

#### 7.5.1 代码架构图

创建两个队列 QA 和 QB，两者队列 TTL 分别设置为 10S 和 40S，然后在创建一个交换机 X 和死信交换机 Y，它们的类型都是 direct，创建一个死信队列 QD，它们的绑定关系如下：

![队列TTL](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357460.PNG)

#### 7.5.2 配置文件类代码

```java
package cn.xyc.mqspringboot.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class TTLQueueConfig {

    // 交换机X
    public static final String X_EXCHANGE = "X";
    // 队列QA，ttl=10
    public static final String QUEUE_A = "QA";
    // 队列QB，ttl=40
    public static final String QUEUE_B = "QB";

    // 死信交换机Y
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    // 队列QD
    public static final String DEAD_LETTER_QUEUE = "QD";


    // 声明 xExchange
    @Bean("xExchange")
    public DirectExchange xExchange(){
        return new DirectExchange(X_EXCHANGE);
    }

    // 声明 死信交换机Y
    @Bean("yExchange")
    public DirectExchange yExchange(){
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    // 声明队列 A ttl 为 10s 并绑定到对应的死信交换机
    @Bean("queueA")
    public Queue queueA(){
        Map<String, Object> args = new HashMap<>(3);
        // 声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        // 声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        // 声明队列的 TTL
        args.put("x-message-ttl", 10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
    }

    // 声明队列 A 绑定 X 交换机
    @Bean
    public Binding queueABindingX(@Qualifier("queueA") Queue queueA,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }

    // 声明队列 B ttl 为 40s 并绑定到对应的死信交换机
    @Bean("queueB")
    public Queue queueB(){
        Map<String, Object> args = new HashMap<>(3);
        // 声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        // 声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        // 声明队列的 TTL
        args.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
    }

    // 声明队列 B 绑定 X 交换机
    @Bean
    public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }

    //声明死信队列 QD
    @Bean("queueD")
    public Queue queueD(){
        return new Queue(DEAD_LETTER_QUEUE);
    }

    //声明死信队列 QD 绑定关系
    @Bean
    public Binding deadLetterBindingQAD(@Qualifier("queueD") Queue queueD,
                                        @Qualifier("yExchange") DirectExchange yExchange){
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}

```

#### 7.5.3 消息生产者代码

```java
package cn.xyc.mqspringboot.controller;


import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

@Slf4j
@RequestMapping("/ttl")
@RestController
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable String message){
        log.info("当前时间： {},发送一条信息给两个 TTL 队列:{}", new Date(), message);
        rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: "+message);
        rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: "+message);
    }
}
```

#### 7.5.4 消息消费者代码

```java
package cn.xyc.mqspringboot.service;


import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Date;

@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "QD")
    public void receiveD(Message message, Channel channel) throws IOException{
        String msg = new String(message.getBody());
        log.info("当前时间： {},收到死信队列信息{}", new Date().toString(), msg);
    }
}
```

发起一个请求 http://localhost:8080/ttl/sendMsg/嘻嘻嘻

```bash
..[..] x.x.x.class : 当前时间： Tue Aug 24 00:25:32 CST 2021,发送一条信息给两个 TTL 队列:嘻嘻嘻
..[..] x.x.x.class : 交换机已经收到 id 为:的消息
..[..] x.x.x.class : 交换机已经收到 id 为:的消息
..[..] x.x.x.class : 当前时间： Tue Aug 24 00:25:42 CST 2021,收到死信队列信息消息来自 ttl 为 10S 的队列: 嘻嘻嘻
..[..] x.x.x.class : 当前时间： Tue Aug 24 00:26:12 CST 2021,收到死信队列信息消息来自 ttl 为 40S 的队列: 嘻嘻嘻
```

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息，然后被消费掉，这样一个延时队列就打造完成了。

不过，如果这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有 10S 和 40S 两个时间选项，如果需要一个小时后处理，那么就需要增加 TTL 为一个小时的队列，如果是预定会议室然后提前通知这样的场景，岂不是要增加无数个队列才能满足需求？  

### 7.6 延时队列优化

#### 7.6.1 代码架构图

在这里新增了一个队列 QC，绑定关系如下,该队列不设置TTL 时间  

![延迟队列优化架构图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261358663.PNG)

#### 7.6.2 配置文件类代码

```java
package cn.xyc.mqspringboot.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

@Component
public class MsgTTLQueueConfig {

    // 死信交换机Y
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    // 队列QC， TTL有生产者决定
    public static final String QUEUE_C = "QC";

    // 声明队列 C 死信交换机
    @Bean("queueC")
    public Queue queueC(){
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //没有声明 TTL 属性
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build();
    }

    // 声明队列 B 绑定 X 交换机
    @Bean
    public Binding queueCBindingX(@Qualifier("queueC") Queue queueC,
                                  @Qualifier("xExchange")DirectExchange xExchange){
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }
}
```

#### 7.6.3 消息生产者代码

```java
// 在类：SendMsgController 中
@GetMapping("sendExpirationMsg/{message}/{ttlTime}")
public void sendMsg(@PathVariable String message, @PathVariable String ttlTime) {

    rabbitTemplate.convertAndSend("X", "XC", message, new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            message.getMessageProperties().setExpiration(ttlTime);
            return message;
        }
    });

    log.info("当前时间： {},发送一条时长{}毫秒 TTL 信息给队列 C:{}", new Date(), ttlTime, message);
}
```

发起请求

* http://localhost:8080/ttl/sendExpirationMsg/你好1/20000
* http://localhost:8080/ttl/sendExpirationMsg/你好2/2000

```bash
: 当前时间： Tue Aug 24 00:29:52 CST 2021,发送一条时长20000毫秒 TTL 信息给队列 C:你好1
: 交换机已经收到 id 为:的消息
: 当前时间： Tue Aug 24 00:29:57 CST 2021,发送一条时长2000毫秒 TTL 信息给队列 C:你好2
: 交换机已经收到 id 为:的消息
: 当前时间： Tue Aug 24 00:30:12 CST 2021,收到死信队列信息你好1
: 当前时间： Tue Aug 24 00:30:12 CST 2021,收到死信队列信息你好2
```

看起来似乎没什么问题，但是在最开始的时候，就介绍过如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“，**因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列，如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。**  

### 7.7 RabbitMQ 插件实现延迟队列  

上文中提到的问题，确实是一个问题，**如果不能实现在消息粒度上的 TTL，并使其在设置的TTL 时间及时死亡，就无法设计成一个通用的延时队列**。那如何解决呢，接下来我们就去解决该问题。  

插件名：rabbitmq_delayed_message_exchange

> 没有实操，略了，具体可看对于文档中的章节，后续有空再补上

### 7.8 总结

延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用 RabbitMQ 的特性，如：消息可靠发送、消息可靠投递，死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为单个节点挂掉导致延时队列不可用或者消息丢失。

当然，延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz 或者利用 kafka 的时间轮，这些方式各有特点，看需要适用的场景  

## 8. 发布确认高级  

在生产环境中由于一些不明原因，导致 RabbitMQ 重启，在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。于是，我们开始思考，如何才能进行 RabbitMQ 的消息可靠投递呢？ 特别是在这样比较极端的情况，RabbitMQ 集群不可用的时候，无法投递的消息该如何处理呢:  

### 8.1 发布确认 SpringBoot 版本

> 接上面的 SpringBoot 代码

#### 8.1.1 确认机制方案  

![发布确认-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357078.PNG)

#### 8.1.2 代码架构图

![发布确认-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357091.PNG)

#### 8.1.3 配置文件  

在配置文件当中需要添加

```properties
spring.rabbitmq.publisher-confirm-type=correlated
```

* NONE：禁用发布确认模式，是默认值
* CORRELATED：发布消息成功到交换器后会触发回调方法
* SIMPLE：
  * 经测试有两种效果，其一效果和 CORRELATED 值一样会触发回调方法；
  * 其二在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是 waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker

#### 8.1.4 添加配置类  

```java
package cn.xyc.mqspringboot.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    // 声明业务 Exchange
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    // 声明确认队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue confirmQueue,
                                @Qualifier("confirmExchange") DirectExchange confirmExchange){
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with("key1");
    }
}
```

#### 8.1.5 消息生产者

```java
package cn.xyc.mqspringboot.controller;


import cn.xyc.mqspringboot.service.MyCallBack;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;

@Slf4j
@RestController
@RequestMapping("/confirm")
public class Producer {

    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";

    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;

    // 依赖注入 rabbitTemplate 之后再设置它的回调对象
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(myCallBack);
    }

    @GetMapping("/sendMsg/{message}")
    public void sendMessange(@PathVariable("message") String message){
        //指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        String routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, routingKey + "--->" + message, correlationData1);

        CorrelationData correlationData2 = new CorrelationData("2");
        routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME + "?", routingKey, routingKey + "--->" + message, correlationData2);

        CorrelationData correlationData3 = new CorrelationData("3");
        routingKey = "key2";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, routingKey + "--->" + message, correlationData3);

        log.info("发送消息内容:{}", message);
    }
}
```

#### 8.1.6 回调接口  

```java
package cn.xyc.mqspringboot.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ReturnedMessage;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback{

    /**
     * 交换机不管是否收到消息的一个回调方法
     * CorrelationData
     * 消息相关数据
     * ack
     * 交换机是否收到消息
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";

        if(ack){
            log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
            log.info("交换机还未收到 id 为:{}消息,由于原因:{}", id, cause);
        }
    }
}
```

#### 8.1.7 消息消费者  

```java
package cn.xyc.mqspringboot.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class ConfirmConsumer {

    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    @RabbitListener(queues = CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message){
        String msg = new String(message.getBody());
        log.info("接受到队列 confirm.queue 消息:{}", msg);
    }
}
```

#### 8.1.8 结果分析

发送请求：`http://localhost:8080/confirm/sendMsg/这是一条消息`

可以看到，发送了三条消息：

* 第一条消息的 RoutingKey 为 "key1"，交换机为 confirm.exchange，交换机成功接收，消费者接收
* 第二条消息的 RoutingKey 为 "key1"，交换机为 confirm.exchange + "?"，交换机就接收不到了，被丢弃
* 第三条消息的 RoutingKey 为 "key2"，交换机为 confirm.exchange，交换机成功接收，消息丢弃
* 消息 1，3 都成功被交换机接收，也收到了交换机的确认回调，但消费者只收到了一条消息，因为第三条消息的 RoutingKey 与队列的 BindingKey 不一致，也没有其它队列能接收这个消息，所有第三条消息被直接丢弃了。 

```bash
# 交换机能接收到消息1
交换机已经收到 id 为:1的消息
# 消息1被消费了，执行回调方法
发送消息内容:这是一条消息
# 交换机未能接收到 消息2
交换机还未收到 id 为:2消息,由于原因:channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'confirm.exchange?' in vhost '/', class-id=60, method-id=40)
# 交换机能接收到消息3
交换机已经收到 id 为:3的消息
```

### 8.2 回退消息

#### 8.2.1 Mandatory 参数

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，**如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的**。那么如何让无法被路由的消息帮我想办法处理一下？最起码通知我一声，我好自己处理啊。通过设置 mandatory 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者

开启配置：`spring.rabbitmq.template.mandatory=true`，或者在代码中设置

#### 8.2.2 消息生产者代码  

```java
package cn.xyc.mqspringboot.controller;

import cn.xyc.mqspringboot.service.MyCallBack;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;

@Slf4j
@RestController
@RequestMapping("/confirm")
public class Producer {

    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";

    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;

    // 依赖注入 rabbitTemplate 之后再设置它的回调对象
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(myCallBack);

        /**
         * true：
         * 交换机无法将消息进行路由时，会将该消息返回给生产者
         * false：
         * 如果发现消息无法进行路由，则直接丢弃
         */
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnsCallback(myCallBack);
    }

    @GetMapping("/sendMsg/{message}")
    public void sendMessange(@PathVariable("message") String message){
        //指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        String routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, routingKey + "--->" + message, correlationData1);

        CorrelationData correlationData2 = new CorrelationData("2");
        routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME + "?", routingKey, routingKey + "--->" + message, correlationData2);

        CorrelationData correlationData3 = new CorrelationData("3");
        routingKey = "key2";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, routingKey + "--->" + message, correlationData3);

        log.info("发送消息内容:{}", message);
    }

}
```

#### 8.2.3 回调接口

```java
package cn.xyc.mqspringboot.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ReturnedMessage;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback{


    /**
     * 交换机不管是否收到消息的一个回调方法
     * CorrelationData
     * 消息相关数据
     * ack
     * 交换机是否收到消息
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";

        if(ack){
            log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
            log.info("交换机还未收到 id 为:{}消息,由于原因:{}", id, cause);
        }
    }

    // 当消息无法路由的时候的回调方法
    @Override
    public void returnedMessage(ReturnedMessage returned) {
        log.info("消息:{}被服务器退回，退回原因:{}, 交换机是:{}, 路由 key:{}",
                new String(returned.getMessage().getBody()), returned.getReplyText(), returned.getExchange(), returned.getRoutingKey());
    }
}
```

#### 8.2.4 消费者

无需改变

#### 8.2.4 结果分析  

```bash
# 消息1交换机能接收到，且能被消费
: 交换机已经收到 id 为:1的消息
# 消息2交换机不能接收到，因为路径交换机路径错误
: 交换机还未收到 id 为:2消息,由于原因:channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'confirm.exchange?' in vhost '/', class-id=60, method-id=40)
# 消息1交换机能接收到，但是路由错误，不能消费
: 交换机已经收到 id 为:3的消息
# 消息1被消费
: 发送消息内容:这是一条消息
# 消息2回退
: 消息:key2--->这是一条消息被服务器退回，退回原因:NO_ROUTE, 交换机是:confirm.exchange, 路由 key:key2
```

### 8.3. 备份交换机

**有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力**，有机会在生产者的消息无法被投递时发现并处理。但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢？前面在设置死信队列的文章中，我们提到，**可以为队列设置死信交换机来存储那些处理失败的消息**，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。

在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。什么是备份交换机呢？**备份交换机可以理解为 RabbitMQ 中交换机的“备胎”** ，当我们为某一个交换机声明一个对应的备份交换机时， 就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。  

#### 8.3.1 代码架构图

![备份交换机](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261358271.PNG)

#### 8.3.2 修改配置类  

```java
package cn.xyc.mqspringboot.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    public static final String BACKUP_EXCHANGE_NAME = "backup.exchange";
    public static final String BACKUP_QUEUE_NAME = "backup.queue";
    public static final String WARNING_QUEUE_NAME = "warning.queue";

    // 声明业务 Exchange
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        // return new DirectExchange(CONFIRM_EXCHANGE_NAME);
        // 修改：
        ExchangeBuilder exchangeBuilder =
                ExchangeBuilder
                        .directExchange(CONFIRM_EXCHANGE_NAME)
                        .durable(true)
                        .withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);  // 设置备份交换机
        return exchangeBuilder.build();
    }

    // 声明备份 Exchange
    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    // 声明确认队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    // 声明备份队列
    @Bean("backupQueue")
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    // 声明告警队列
    @Bean("warningQueue")
    public Queue warningQueue(){
        return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }

    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue confirmQueue,
                                @Qualifier("confirmExchange") DirectExchange confirmExchange){
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with("key1");
    }

    // 声明备份队列绑定关系
    @Bean
    public Binding backupQueueBinding(@Qualifier("backupQueue") Queue backupQueue,
                                @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    //  声明报警队列绑定关系
    @Bean
    public Binding warningQueueBinding(@Qualifier("warningQueue") Queue warningQueue,
                                      @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(warningQueue).to(backupExchange);
    }

}
```

#### 8.3.3 报警消费者

```java
package cn.xyc.mqspringboot.service;


import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class WarningConsumer {

    public static final String WARNING_QUEUE_NAME = "warning.queue";

    @RabbitListener(queues = WARNING_QUEUE_NAME)
    public void receiveWarningMsg(Message message){
        String msg = new String(message.getBody());
        log.info("接受到队列 warning.queue 消息:{}", msg);
    }
}
```

#### 8.3.4 备份消费者

```java
package cn.xyc.mqspringboot.service;


import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class BackupConsumer {

    public static final String BACKUP_QUEUE_NAME = "backup.queue";

    @RabbitListener(queues = BACKUP_QUEUE_NAME)
    public void receiveMsg(Message message){
        String msg = new String(message.getBody());
        log.info("接受到队列 backup.queue 消息:{}", msg);
    }
}
```

#### 8.3.5 结果分析

**测试注意事项**：重新启动项目的时候需要把原来的 confirm.exchange 删除因为我们修改了其绑定属性，不然报以下错:  

发送消息：`http://localhost:8080/confirm/sendMsg/这是一条消息`

```bash
: 交换机已经收到 id 为:1的消息
: 交换机还未收到 id 为:2消息,由于原因:channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'confirm.exchange?' in vhost '/', class-id=60, method-id=40)
: 交换机已经收到 id 为:3的消息
: 发送消息内容:这是一条消息
: 接受到队列 confirm.queue 消息:key1--->这是一条消息
: 接受到队列 warning.queue 消息:key2--->这是一条消息
: 接受到队列 backup.queue 消息:key2--->这是一条消息
```

mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，消息究竟何去何从？谁优先级高，经过上面结果显示答案是**备份交换机优先级高**。  

## 9. RabbitMQ 其他知识点

### 9.1. 幂等性

#### 9.1.1. 概念

用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等。  

#### 9.1.2. 消息重复消费

消费者在消费 MQ 中的消息时， MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断，故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

#### 9.1.3. 解决思路

MQ 消费者的幂等性的解决一般使用**全局 ID** 或者写个**唯一标识**比如时间戳、UUID、订单号，消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。

#### 9.1.4. 消费端的幂等性保障

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性，这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。

业界主流的幂等性有两种操作：

1. **唯一ID+指纹码机制，利用数据库主键去重；**
2. **利用 redis 的原子性去实现**

#### 9.1.5. 唯一ID+指纹码机制

指纹码：我们的一些规则或者时间戳加别的服务给到的唯一信息码，它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中，优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。

#### 9.1.6. Redis 原子性

**利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费。**

### 9.2. 优先级队列

#### 9.2.1. 使用场景

在我们系统中有一个**订单催付**的场景，我们的客户在天猫下的订单，淘宝会及时将订单推送给我们，如果在用户设定的时间内未付款那么就会给用户推送一条短信提醒，很简单的一个功能对吧，但是， tmall 商家对我们来说，肯定是要分大客户和小客户的对吧，比如像苹果，小米这样大商家一年起码能给我们创造很大的利润，所以理应当然，他们的订单必须得到优先处理，而曾经我们的后端系统是使用 redis 来存放的定时轮询，大家都知道 redis 只能用 List 做一个简简单单的消息队列，并不能实现一个优先级的场景，  所以订单量大了后采用 RabbitMQ 进行改造和优化，如果发现是大客户的订单给一个相对比较高的优先级，否则就是默认优先级。  

#### 9.2.2. 如何添加

方式1：控制台页面添加

![优先队列](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357232.PNG)

方式2：队列中代码添加优先级 

```java
// 设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU
Map<String, Object> params = new HashMap();
params.put("x-max-priority", 10);
channel.queueDeclare(QUEUE_NAME, true, false, false, params);
```

消息中添加优先级  

```java
// 给消息赋予一个 priority 属性
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
```

![优先队列-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261357678.PNG)

注意事项：要让队列实现优先级需要做如下事情：

1. 队列需要设置为优先级队列；
2. 消息需要设置消息的优先级，消费者需要等待消息已经发送到队列中才去消费因为，这样才有机会对消息进行排序

#### 9.2.3. 实战

消息生产者

```java
package cn.xyc.rabbitmq.nine;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;

public class Producer {

    private static final String QUEUE_NAME="hello";

    public static void main(String[] args) throws Exception{

        Channel channel = RabbitMqUtils.getChannel();

        //给消息赋予一个 priority 属性
        AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();

        for (int i = 1; i <= 10; i++) {
            String message = "info"+i;

            if(i==5){
                channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());
            }else{
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            }
            System.out.println("发送消息完成:" + message);
        }
    }
}
```

消息消费者  

```java
package cn.xyc.rabbitmq.nine;

import cn.xyc.rabbitmq.RabbitMqUtils;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;
import com.rabbitmq.client.Delivery;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class Consumer {

    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();
        // 设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU
        Map<String, Object> params = new HashMap();
        params.put("x-max-priority", 10);
        channel.queueDeclare(QUEUE_NAME, true, false, false, params);
        System.out.println("消费者启动等待消费..............");

        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                String receivedMessage = new String(message.getBody());
                System.out.println("接收到消息:" + receivedMessage);
            }
        };

        channel.basicConsume(QUEUE_NAME, true, deliverCallback,(consumerTag)->{
            System.out.println("消费者无法消费消息时调用，如队列被删除");
        });
    }
}
```

### 9.3. 惰性队列

#### 9.3.1. 使用场景

RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因（比如消费者下线、宕机亦或者是由于维护而关闭等）而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。

默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当 RabbitMQ 需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。虽然 RabbitMQ 的开发者们一直在升级相关的算法，但是效果始终不太理想，尤其是在消息量特别大的时候。  

#### 9.3.2. 两种模式

队列具备两种模式： default 和 lazy。

默认的为 default 模式，在 3.6.0 之前的版本无需做任何变更。 lazy 模式即为惰性队列的模式，可以通过调用 `channel.queueDeclare` 方法的时候在参数中设置，也可以通过 Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

在队列声明的时候可以通过 `“x-queue-mode”` 参数来设置队列的模式，取值为 “default” 和 “lazy”。下面示例中演示了一个惰性队列的声明细节：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);  
```

#### 9.3.3. 内存开销对比

![惰性队列开销对比](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261358267.PNG)

在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅占用 1.5MB  

## 10. RabbitMQ 集群

### 10.1. clustering

#### 10.1.1. 使用集群的原因

最开始我们介绍了如何安装及运行 RabbitMQ 服务，不过这些是单机版的，无法满足目前真实应用的要求。如果 RabbitMQ 服务器遇到内存崩溃、机器掉电或者主板故障等情况，该怎么办？单台 RabbitMQ 服务器可以满足每秒 1000 条消息的吞吐量，那么如果应用需要 RabbitMQ 服务满足每秒 10 万条消息的吞吐量呢？购买昂贵的服务器来增强单机 RabbitMQ 务的性能显得捉襟见肘，搭建一个 RabbitMQ 集群才是解决实际问题的关键。

#### 10.1.2. 搭建步骤  

> 略...

### 10.2. 镜像队列

#### 10.2.1. 使用镜像的原因

如果 RabbitMQ 集群中只有一个 Broker 节点，那么该节点的失效将导致整体服务的临时性不可用，并且也可能会导致消息的丢失。可以将所有消息都设置为持久化，并且对应队列的 durable 属性也设置为 true， 但是这样仍然无法避免由于缓存导致的问题：因为消息在发送之后和被写入磁盘井执行刷盘动作之间存在一个短暂却会产生问题的时间窗。通过 publisher confirm 机制能够确保客户端知道哪些消息己经存入磁盘， 尽管如此，一般不希望遇到因单点故障导致的服务不可用。  

引入镜像队列（Mirror Queue）的机制，可以将队列镜像到集群中的其他 Broker 节点之上，如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的可用性。  

#### 10.2.2. 搭建步骤  

> 略...

