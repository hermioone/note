# RabbitMQ消息中间件技术精讲 

## 第一章 课程介绍

### 1.3 业界主流消息中间件介绍

#### ActiveMQ

* 性能不好

ActiveMQ集群模式

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-18-51.png)

#### Kafka

* 主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输。不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务。

**关注于性能，对数据可靠性要求不高**

Kafka集群模式

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-24-23.png)

#### RocketMQ

* 具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。起源于Kafka，对消息的可靠传输及事务性做了优化
* 不易维护
* 商业版收费

RocketMQ集群模式

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-30-33.png)

#### RabbitMQ

* 开源，性能好，稳定性好
* 提供可靠性消息投递模式、返回模式
* 与SpringAMQP完美地整合，API丰富
* 集群模式丰富，表达式配置，HA模式，镜像队列模型
* 保证数据不丢失的前提下做到高可靠性，可用性
* 性能和吞吐量不如Kafka，但是比ActiveMQ高

RabbitMQ集群模式

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-34-55.png)

## 第二章 RabbitMQ核心概念

### 2.3 RabbitMQ高性能的原因

* Erlang语言最初在于交换机领域，这样使得RabbitMQ在Broker之间进行数据交互的性能是非常优秀的
* Erlang的优点：Erlang有着和原生Socket一样的延迟

### 2.4 AMQP高级消息队列协议与模型

AMQP定义：是具有现代特征的二进制协议。是应用层协议的一个开放标准，为面向消息的中间件设计。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-51-31.png)

> 生产者投递到Exchange上，消费者从Message Queue中取数据

### 2.5 AMQP核心概念讲解

* ```Server```：又称broker，接受客户端的连接，实现AMQP实体服务
* ````Connection```：应用程序与broker的连接
* ```Channel```：网络信道，**几乎所有的操作都在Channel中进行**。Channel是进行消息读写的通道，客户端可建立多个Channel，每个Channel代表一个会话任务。类似与数据库中的Session
* ```Message```：服务器和应用程序之间传送的数据。由**Properties**和**Bidy**组成。
  * Properties：可以对消息进行修饰，比如消息的优先级、延迟等高级特性
  * Body：消息体内容
* ```Virtual Host```：虚拟地址，用于进行**逻辑隔离**，最上层的消息路由。一个Virtual Host里面可以有若干个Exchange和Queue，**同一个Virtual Host里面不能有相同名称的Exchange或Queue**。Vhost之于RabbitMQ就像虚拟机之于物理服务器一样：它既能将同一个RabbitMQ的众多客户区分开来，又可以避免队列和交换器的命名冲突
* ```Exchange```：交换机，接收消息，根据路由键转发消息到绑定的队列
* ```Binding```：Exchange和Queue之间的虚拟链接，binding中可以包含routing key
* ```Routing Key```：路由规则，虚拟机可用它来确定如何路由一个特定消息
* ```Queue```：消息队列，保存消息并将它们转发给消费者

### 2.6 RabbitMQ整体架构与消息流转

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-59-07.png)

### 2.7 RabbitMq环境安装

* 服务的启动：```rabbitmq-server start &```
* 服务的停止：```rabbitmqctl stop_app```
* 管理插件：```rabbitmq-plugins enable rabbitmq_management```

5672是java端进行通信的端口号；15672是管控台的端口号；25672是集群通信的端口号。

### 2.9 命令行与管理台结合讲解

凡是可以在管理台执行的操作都可以通过命令行来操作。

#### 高级操作

* ```rabbitmqctl reset```：移除所有数据，要在```rabbitmqctl stop_app```之后使用
* ```rabbitmqctl change_cluster_node_type disc | ram```：修改集群节点的存储形式
* ```rabbitmqctl forget_cluster_node [--offline]```：忘记节点（摘除节点）

### 2.10 生产者消费者模型构建

需要先运行消费者，因为**在消费者中声明了队列**

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-44-54.png)

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-43-34.png)

如图所示，有一个问题：

***为什么生产者绑定的routingKey和消费者监听的queue名字一样时（“test001”），消息也可以路由过去？***

> RabbitMQ的生产者在发送消息时，必须要指定一个exchange，如果exchange为空的话（""），会走第一个exchange（```AMQP default```），这个exchange的路由规则就是按照routingKey的名字来寻找同名的queue，如果有，就把消息路由过去；如果没有，就把消息删除。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-51-18.png)

### 2.12 交换机详解

Exchange：接收消息，并根据路由键转发消息到所绑定的队列

#### 交换机的属性

* Name：交换机名称
* Type：交换机类型：direct、topic、fanout、headers
* Durability：是否需要持久化
* Auto Delete：当最后一个绑定到Exchange上的队列删除后，自动删除该Exchange
* Internal：当前Exchange是否用于RabbitMQ内部使用，默认为False（不常使用）
* Arguments：扩展参数，用于扩展AMQP协议自定制化

#### Direct Exchange

所有发送到Direct Exchange的消息被转发到RoutingKey中指定的Queue

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_22-05-56.png)

##### Producer

```java
// 3. 通过connection创建一个Channel
Channel channel = connection.createChannel();

// 4. 声明
String exchangeName = "test_direct_exchange";
String routingKey = "test.direct";

// 5. 发送
String msg = "Direct Exchange Message";
channel.basicPublish(exchangeName, routingKey, null, msg.getBytes());
```

##### Consumer

```java
// 3. 通过connection创建一个Channel
Channel channel = connection.createChannel();

// 4. 声明
String exchangeName = "test_direct_exchange";
String exchangeType = "direct";
String routingKey = "test.direct";
String queueName = "test_direct_queue";
// 声明一个交换机
channel.exchangeDeclare(exchangeName, exchangeType, true, false, false, null);
// 声明一个队列
channel.queueDeclare(queueName, false, false, false, null);
// 建立一个绑定关系
channel.queueBind(queueName, exchangeName, routingKey);

// 5. 创建消费者
QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
```

#### Topic Exchange

* 所有发送到Topic Exchange的消息被转发到订阅了RoutingKey中指定Topic的所有的Queue上

* Exchange将RoutingKey和某Topic进行模糊匹配，此时队列需要绑定一个Topic

  模糊匹配可以使用通配符

  * #：匹配一个或多个词：“log.#”可以匹配到```log.info.oa```
  * *：匹配不多不少一个词：“log.\*”只会匹配到```log.erro```

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_22-35-41.png)

##### Producer

```java
// 4. 声明
String exchangeName = "test_topic_exchange";
String routingKey1 = "user.save";
String routingKey2 = "user.delete.abc";

// 5. 发送
String msg = "Direct Exchange Message";
channel.basicPublish(exchangeName, routingKey1, null, msg.getBytes());
channel.basicPublish(exchangeName, routingKey2, null, msg.getBytes());
```

##### Consumer

```java
// 4. 声明
String exchangeName = "test_topic_exchange";
String exchangeType = "topic";
String routingKey = "user.#";
String queueName = "test_topic_queue";
// 声明一个交换机
channel.exchangeDeclare(exchangeName, exchangeType, true, false, false, null);
// 声明一个队列
channel.queueDeclare(queueName, false, false, false, null);
// 建立一个绑定关系
channel.queueBind(queueName, exchangeName, routingKey);
```

Consumer可以收到两条消息。

> 注意，如果要修改Consumer中的绑定关系（例如修改成：```channel.queueBind(queueName, exchangeName, "user.*);```，需要先将原来的绑定关系解绑，否则test_topic_queue和test_topic_exchange会有两个绑定关系（ “user.#” 和 “user.*” ）

#### Fanout Exchange

* 不处理任何路由键，只需要简单地将队列绑定到交换机上
* 发送到交换机的消息都会被转发到与该交换机绑定的所有队列上
* 转发消息的速度是最快的

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_22-57-27.png)

##### Producer

```java
// 4. 声明
String exchangeName = "test_topic_exchange";

// 5. 发送
String msg = "Direct Exchange Message";
channel.basicPublish(exchangeName, "", null, msg.getBytes());
```

##### Consumer

```java
// 4. 声明
String exchangeName = "test_fanout_exchange";
String exchangeType = "fanout";
String routingKey = "";			// 绑定关系为空串
String queueName = "test_fanout_queue";
// 声明一个交换机
channel.exchangeDeclare(exchangeName, exchangeType, true, false, false, null);
// 声明一个队列
channel.queueDeclare(queueName, false, false, false, null);
// 建立一个绑定关系
channel.queueBind(queueName, exchangeName, routingKey);
```

这里Producer和Consumer中的routingKey可以随便设置，不会有任何影响。

> 总结来说，对于不同的交换机类型，Producer的代码几乎相同，只是Consumer端的exchangeType不同

### 2.15 绑定、队列、消息、虚拟主机详解

* 绑定：Exchange和Exchange、Queue之间的连接关系。绑定中可以包含routingKey或者参数

* 队列：实际存储消息数据

  * Durability：是否持久化
  * Auto Delete：为true时，当最后一个监听被移除后，该Queue会自动被删除

* 消息：服务器和应用程序之间传送的数据。由Properties和Payload组成

  * 常用属性：delivery mode、headers（自定义属性）

  * 其它属性：

    * content_type、content_encoding、priority

    * correlation_id、reply_to、expiration、message_id

      correlation_id 通常和 reply_to 一起使用：**用于RPC服务**

      * `replyTo`：通常用来设置一个回调队列

      * `correlationId`：用来关联请求和RPC调用后的恢复

        ![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2019-01-28_10-43-25.png)

    * timestamp、type、user_id、app_id、cluster_id

    ```java
    Map<String, Object> headers = new HashMap<>();
    headers.put("蓉儿", "郭靖");
    headers.put("盈盈", "令狐冲");
    headers.put("赵敏", "张无忌");
    
    AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
        .deliveryMode(2)                    // 持久化消息
        .contentEncoding("UTF-8")           // 字符集
        .expiration("10000")                // 10s 后如果这个消息没有被消费，则被自动移除
        .headers(headers)                   // 自定义属性
        .build();
    
    // 4. 通过Channel发送数据
    for (int i = 0; i < 5; i++) {
        String msg = "Leihou RabbitMq";
        channel.basicPublish("", "test001", props, msg.getBytes());
    }
    ```

    ```java
    while (true) {
        QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
        String msg = new String(delivery.getBody());
        Map<String, Object> headers = delivery.getProperties().getHeaders();
        System.out.println(msg + "------" + headers);
        // Envelope envelope = delivery.getEnvelope();
    }
    ```

* 虚拟主机：虚拟地址，用于进行逻辑隔离，最上层的消息路由

## 第三章 深入RabbitMQ高级特性

### 3.2 消息如何保障100%的投递成功

什么是生产端的可靠性投递？

* 保障消息的成功发出
* 保障MQ节点的成功接收
* 发送端收到MQ节点（Broker）确认应答
* 完善的消息进行补偿机制

互联网大厂的解决方案：

* 消息落库，对消息状态进行打标（消息状态的变化）
* 消息的延迟投递，做二次确认，回调检查

#### 方案一：消息落库，对消息状态进行打标

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-22_20-55-42.png)

* step1：业务数据入库（BIZ DB），生成消息也入库（MSG DB），这个数据库都要做持久化
* step2：发送消息
* step3：Producer异步监听Broker返回的确认消息
  * step4：从数据库中把这条消息抓取出来，对status进行更新（从0更新成1，消息投递成功）
  * step5：有一个初始timeout，当超过了这个timeout后status仍为0，就从数据库中把该条消息抽取出来
  * step6：把消息再次投递，重复step3、4、5
  * step7：重试次数大于3时，把status设为2，进行一些补偿机制

#### 方案二：消息的延迟投递，做二次确认，回调检查

方案一在高并发的场景下不太合适，因为需要做两次数据库的同步操作。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-22_21-44-32.png)

* step1：**先把订单入库后，再发送消息**
* step2：发送延迟消息（比如2分钟后消息才到）
* step3：消费端监听并处理消息
* step4：消费端生成一条消息（Send Confirm），投递到MQ
* step5：callback服务监听（step4发送的服务），对消息进行持久化的存储（MSG DB）
* step6：2分钟后，step2的延迟消息发送到MQ，Callback服务监听延迟投递的队列，去检查DB数据库，查看是否有Confirm
  * 如果没有Confirm，Callback给Upstream发送一个RPC Resend命令。Upstream去查BIZ，再把这个消息重新发出去

### 3.3 幂等性概念及业界主流解决方案

#### 幂等性

借鉴数据库的乐观锁机制：比如执行一个更新库存的SQL语句

```sql
UPDATE T_REPS SET count=count-1, version=version+1 where version=1
```

先查出来version在数据库中的当前值，更新库存时也更新version值，这样如果有并发的更新请求过来，仍旧用的old version，就查不到数据了。

> 对一件事情进行操作，操作一次的结果和操作多次的结果是一样的。

#### 消费端-幂等性保障

在海量订单产生的业务高峰期，如何避免消息的重复消费问题

***消费端实现幂等性，就意味着，我们的消息永远不会消费多次，即使我们收到了多条一样的消息***

业界主流的幂等性操作：

* 唯一ID + 指纹码 机制，利用数据库主键去重
* 利用Redis的原子性去实现

#### 唯一ID + 指纹码

```sql
SELECT COUNT(1) FROM T_ORDER WHERE ID = 唯一ID + 指纹码
```

* 如果数量为0，就执行insert操作
* 如果数量为1，就返回失败

好处：实现简单

坏处：高并发下有数据库写入的性能瓶颈

解决方案：跟进ID进行分库分表进行算法路由

#### 利用Redis原子性实现

利用Redis进行幂等，需要考虑的问题：

* 是否要进行数据落库，如果落库的话，关键解决的问题是数据库和缓冲如何做到原子性
* 如果不进行落库，那么都存储到缓存中，如何设置定时同步的策略

### 3.4 持久化

RabbitMQ的持久化分为三个部分：

* 交换器的持久化：在声明交换器时将 `durable` 参数设置为 true
  * 否则交换器的元数据会丢失，但消息不会丢失，只是不能将消息发送到这个交换器
* 队列的持久化：声明队列时将 `durable` 参数设置为 true
  * 队列的元数据会丢失，消息也会丢失
* 消息的持久化：将消息的投递模式（`BasicProperties` 中的 `deliveryMode` 属性）设置为2
  * 确保RabbitMQ发生异常情况时，消息不会丢失

> 单单只设置队列持久化，重启之后消息会丢失;单单只设置消息的持久化，重启之后队列消失，继而消息也
> 丢失。单单设置消息持久化而不设置队列的持久化显得毫无意义。

***将交换器、队列、消息都设置了持久化之后就能百分之百保证数据不丢失了吗 ？*** 答案是否定的。

* 从消费者来说，如果在订阅消费队列时将autoAck 参数设置为true ，那么当消费者接收到相关消息之后，还没来得及处理就看机了，这样也算数据丢失。

  * 这种情况很好解决，将 `autoAck` 参数设置为false ， 并进行手动确认。

* 在持久化的消息正确存入RabbitMQ 之后，还需要有一段时间（虽然很短，但是不可忽视）才能存入磁盘之中。RabbitMQ 并不会为每条消息都进行同步存盘(调用内核的fsyncl方法)的处理，可能仅仅保存到操作系统缓存之中而不是物理磁盘之中。如果在这段时间内RabbitMQ 服务节点发生了异常情况，消息保存还没来得及落盘，那么这些消息将会丢失。

  > fsync 在Linux 中的意义在于同步数据到存储设备上。大多数块设备的数据都是通过缓存进行的，将数据写到文件上通常将该数据由内核复制到缓存中，如果缓存尚未写满，则不将其排入输出队列上，而是等待其写满或者当内核需要重用该缓存时，再将该缓存排入输出队列，进而同步到设备上。内核提供了fsync接口，用户可以根据自己的需要通过此接口更新数据到存储设备上。

  * 可以使用RabbitMQ的镜像队列机制来解决这个问题

* 在发送端引入事务机制或者发送方确认机制保证消息已经正确地发送并存储到RabbitMQ中。



### 3.5 生产者确认

默认情况下生产者是不知道消息有没有正确地到达服务器。如果在消息到达服务器之前己经丢失，持久化操作也解决不了这个问题。RabbitMQ 针对这个问题，提供了两种解决方式:

* 事务机制
* 发送方确认机制（publisher confirm)

#### 3.5.1 事务机制

```java
try {
    channel.txSelect();  			// 将当前信道设置为事务模式
    for(int i = 0; i < LOOP_TIMES; i++) { // 如果发送多条消息，把发送和提交事务包进循环中
        channel.basicPublish("exchange", 
                         "routingKey", 
                         MessageProperties.PERSISTENT_TEXT_PLAIN, 
                         ("msg:" + i).getBytes());
    	channel.txCommit();				// 提交事务
    }
}catch(Exception e) {
    e.printStackTrace();
    channel.txRollback();			// 事务回滚
}
```

> 使用事务机制会极大影响RabbitMQ的性能，降低消息吞吐量。

#### 3.5.2 发送方确认机制 

事务机制在一条消息发送之后会 **使发送端阻塞**，以等待RabbitMQ 的回应，之后才能继续发送下一条消息。相比之下， 发送方确认机制最大的好处在于**它是异步的**，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用程序便可以通过回调方法来处理该确认消息，如果RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条nack CBas i c . Nack) 命令，生产者应用程序同样可以在回调方法中处理该nack 命令。

> 如果队列和消息是持久化的，那么确认消息会在消息写入磁盘之后发出。

如何实现Confirm确认消息

1. 在channel上开启确认模式：```channel.confirmSelect()```
2. 在channel上添加监听：```addConfirmListener```，监听成功和失败的返回结果，根据具体的结果对消息进行重新发送、或记录日志等后续处理

```java
// 3. 通过Connection创建一个新的Channel
Channel channel = connection.createChannel();

// 4. 指定我们的消息投递模式：消息的确认模式
channel.confirmSelect();

String exchangeName = "test_confirm_exchange";
String routingKey = "confirm.save";

// 5. 发送一条消息
String msg = "hello, RabbitMQ send confirm message";
channel.basicPublish(exchangeName, routingKey, null, msg.getBytes());

// 6. 添加一个确认监听
channel.addConfirmListener(new ConfirmListener() {
    /**
             *
             * @param deliveryTag 消息的唯一标签
             * @param multiple
             * @throws IOException
             */
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("----------------ack----------------");
    }

    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("----------------no ack----------------");
    }
});
```

Consumer没有变化

#### 3.5.3 注意

* 事务机制和 Publisher Confirm 机制时互斥的
* 事务机制和 Publisher Confirm 机制能保证正确发送至RabbitMQ的交换器，如果此交换器没有匹配的队列，那么消息也会丢失。此时需要发送方使用 mandatory 参数，详见【3.6 Return返回消息详解】

### 3.6 Return返回消息详解

Return消息机制：

* Return Listener用于处理一些不可路由的消息
* 某些情况下，在发送消息时，当前的exchange不存在或者指定的路由key路由不到，这个时候我们需要监听这种不可达的消息，就要使用Return Listener
* 有一个关键的配置项 **mandatory**：如果为true，则监听器会接收到路由不可达的消息，然后进行后续处理；如果为false，那么broker端自动删除该消息

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-24_22-26-27.png)

### 3.7 自定义消费者

之前的消费端使用while循环

```java
// 5. 创建消费者
QueueingConsumer consumer = new QueueingConsumer(channel);
channel.basicConsume(queueName, true, consumer);

while (true) {
    QueueingConsumer.Delivery delivery = consumer.nextDelivery();
    System.out.println("消费端接收" + new String(delivery.getBody()));
}
```

我们可以用一种更优雅的方式来实现消费者

```java
// 5. 创建消费者
channel.basicConsume(queueName, true, new MyConsumer(channel));


class MyConsumer extends DefaultConsumer {

    /**
     * Constructs a new instance and records its association to the passed-in channel.
     *
     * @param channel the channel to which this consumer is attached
     */
    public MyConsumer(Channel channel) {
        super(channel);
    }

    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("----------------------consume message");
        System.err.println("consumerTag: " + consumerTag);
        System.err.println("envelope: " + envelope);
        System.err.println("properties: " + properties);
        System.err.println("body: " + new String(body));
    }
}
```

### 3.8 消费端的限流策略

> 什么是消费端限流？

假设我们的RabbitMQ服务器有上万条未处理的消息，我们随便打开一个消费者客户端，巨量的消息瞬间全部推送过来，但是我们**单个客户端无法同时处理这么多数据**，可能导致服务器崩溃。

> RabbitMQ提供了一种qos（服务质量保证）功能，即在**非自动确认消息**的前提下，如果一定数目的消息（通过基于consume或者channel设置Qos的值）未被确认前，不进行消费新的消息。

***所以autoack一定要设置为false***

```java
/*
 * prefetchSize: 0，
 * prefetchCount: 1，一次最多推送给consumer 1 条消息，收到回复后再推送下1条
 * global: false，这个限流作用在queue上，true表示作用再channel上
 */
channel.basicQos(0, 1, false);

// autoAck设置为false
channel.basicConsume(queueName, false, new MyConsumer(channel));



public class MyConsumer extends DefaultConsumer {

    private Channel channel;

    public MyConsumer(Channel channel) {
        super(channel);
        this.channel = channel;
    }

    @Override
    public void handleDelivery(String consumerTag, 
                               Envelope envelope, 
                               AMQP.BasicProperties properties, 
                               byte[] body) throws IOException {
        System.out.println("----------------------consume message");
        System.err.println("consumerTag: " + consumerTag);
        System.err.println("envelope: " + envelope);
        System.err.println("properties: " + properties);
        System.err.println("body: " + new String(body));

        
        // 注意：这里一定要ack，否则消费者会被block住
        // false表示不支持 批量签收
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
}
```

### 3.10 消费端ACK与重回队列机制

#### 消费端的手工ACK和NACK

* 消费端进行消费的时候，如果由于业务异常我们可以进行日志的记录，然后进行补偿
* 如果由于服务器宕机等严重问题，那我们就需要手工进行ACK保障消费端消费成功

#### 消费端的重回队列

消费端重回队列是为了对没有处理成功的消息，把消息重新回递给Broker。在实际应用中，一半都会***关闭重回队列***。

#### Producer

```java
for (int i = 0; i < 5; i++) {
    String msg = "hello, RabbitMQ send ack message " + i;

    Map<String, Object> headers = new HashMap<>();
    headers.put("num", i);

    AMQP.BasicProperties props = new AMQP.BasicProperties().builder()
        .deliveryMode(2)
        .contentEncoding("UTF-8")
        .headers(headers)
        .build();

    channel.basicPublish(exchangeName, routingKey2, false, props, msg.getBytes());
}

channel.close();
connection.close();
```

#### Consumer

```java
// autoAck设置为false
channel.basicConsume(queueName, false, new MyConsumer(channel));


class MyConsumer extends DefaultConsumer {

    private Channel channel;

    public MyConsumer(Channel channel) {
        super(channel);
        this.channel = channel;
    }

    @Override
    public void handleDelivery(String consumerTag, 
                               Envelope envelope, 
                               AMQP.BasicProperties properties, 
                               byte[] body) throws IOException {
        System.out.println("----------------------consume message----------------------");
        System.err.println("body: " + new String(body));

        if((Integer)properties.getHeaders().get("num") == 1) {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            /*
             * 最后一个参数：requeue=true：消费失败的这个消息重回队列，并且在队列的尾部
             */
            channel.basicNack(envelope.getDeliveryTag(), false, true);
        }else {
            channel.basicAck(envelope.getDeliveryTag(), false);
        }
    }
}
```

### 3.11 TTL消息详解

TTL：Time To Live的缩写，也就是生存时间。RabbitMQ支持消息的过期时间，在消息发送时可以进行指定。也支持队列的过期时间，从消息入队开始，只要超过了队列的超时时间配置，那么消息会自动清除

如下代码是作用于消息上的

```java
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
                .expiration("10000")                // 10s 后如果这个消息没有被消费，则被自动移除
                .build();
```

如下代码是作用在队列上，队列上的所有消息10s未被消费，都会被自动移除

```java
Map<String, Object> arguments = new HashMap<>();
arguments.put("x-message-ttl", 10000);
channel.queueDeclare(queueName, true, false, false, arguments);
```

### 3.12 死信队列DLX

利用DLX，**当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个Exchange（DLX）上**。

消息变成死信有以下几种情况：

* 消息被拒绝（basic.reject / basic.nack），并且 requeue=false
* 消息TTL过期
* 队列达到最大长度

DLX和一般的Exchange没有任何区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性。当这个队列中有死信时，RabbitMQ会自动地将这个死信消息重新发布到设置的Exchange上去，进而路由到另一个队列。可以监听这个队列中消息做相应的处理，这个特性可以弥补RabbitMQ 3.0之前支持的immediate参数的功能。

#### Producer

```java
String msg = "hello, RabbitMQ send dlx message";
for (int i = 0; i < 1; i++) {
    AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
        .deliveryMode(2)                    // 持久化消息
        .contentEncoding("UTF-8")           // 字符集
        .expiration("10000")                // 10s 后如果这个消息没有被消费，则被自动移除
        .build();
    channel.basicPublish(exchangeName, routingKey2, false, props, msg.getBytes());
}

channel.close();
connection.close();
```

#### Consumer

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-29_22-14-08.png)

### 3.13 消息传输保障

### 4-2 SpringAMQP用户管理组件：RabbitAdmin

```java
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;		// 注意这里和之前的ConnectionFactory不同，之前的是：com.rabbitmq.client.ConnectionFactory
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
//@ComponentScan(value = {"com.vermouth.*"})
public class RabbitMQConfig {

    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();

        connectionFactory.setAddresses(RabbitMQConstant.ADDRESS);
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setVirtualHost("/");

        return connectionFactory;
    }

    /**
     * @param connectionFactory 使用@Bean注解可自动注入其它的bean，但是名字(connectionFactory)必须和Bean的名字(connectionFactory())一样
     */
    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        rabbitAdmin.setAutoStartup(true);           // 这个一定要配置成true，否则Spring容器不会加载RabbitAdmin类
        return rabbitAdmin;
    }
}
```

可以使用RabbitAdmin来管理Exchange，Queue

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RabbitmqSpringApplicationTests {

    @Autowired
    private RabbitAdmin rabbitAdmin;

    @Test
    public void testAdmin() throws Exception {
        Assert.assertNotNull(rabbitAdmin);

        rabbitAdmin.declareExchange(new DirectExchange("test.direct", false, false));
        rabbitAdmin.declareQueue(new Queue("test.direct.queue", false));

        rabbitAdmin.declareBinding(new Binding("test.direct.queue", Binding.DestinationType.QUEUE,
                "test.direct", "direct", new HashMap<>()));

        // 必须要先declareExchange和declareQueue，否则后面直接绑定会报错
        rabbitAdmin.declareExchange(new TopicExchange("test.topic", false, false));
        rabbitAdmin.declareQueue(new Queue("test.topic.queue", false));
        rabbitAdmin.declareBinding(BindingBuilder
                .bind(new Queue("test.topic.queue", false))		// name, durable
                .to(new TopicExchange("test.topic", false, false))	// name, durable, autoDelete
                .with("user.#"));
    }

}
```

### 4-4 RabbitAdmin分析

RabbitAdmin底层实现就是从Spring容器中获取Exchange、Binding、RoutingKey以及Queue的@Bean声明，然后使用RabbitTemplate的```execute()```执行对应的声明、修改、删除等一系列RabbitMQ基础功能操作。

### 4-5 SpringAMQP声明式配置使用

在Rabbit基础API里声明一个Exchange、绑定、队列：

```java
channel.exchangeDeclare(exchangeName, exchangeType, true, false, false, null);
channel.queueDeclare(queueName, fals,e false, false, null);
channel.queueBind(queueName, exchangeName, routingKey);
```

使用SpringAMQP去声明，只需使用@Bean方式

```java
@Bean
public TopicExchange exchange001() {
    return new TopicExchange("topic001", true, false);
}

@Bean
public Queue queue001() {
    return new Queue("queue001", true);
}

@Bean
public Binding binding001() {
    return BindingBuilder.bind(queue001()).to(exchange001()).with("mq.*");
}
```

### 4-6 RabbitTemplate实战

在与SpringAMQP整合的时候发送消息的关键类。该类提供了丰富的发送消息方法，包括可靠性投递消息方法、回调监听消息接口```ConfirmCallback```，返回值确认接口```ReturnCallback```等等。也需要先注入到Spring容器中才能使用。

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    return new RabbitTemplate(connectionFactory);
}
```

```java
@Test
public void testSendMessage2() throws Exception {
    MessageProperties props = new MessageProperties();
    props.setContentType("text/plain");
    Message message = new Message("Leihou, RabbitMQ".getBytes(), props);
    rabbitTemplate.send("topic001", "spring.amqp", message);

    rabbitTemplate.convertAndSend("topic001", "spring.amqp", "Leihou, RabbitMQ");
}
```

### 4-7 SimpleMessageListenerContainer

* 监听队列，自动启动，自动声明功能
* 设置事务特性
* 设置消费者数量
* 设置消息确认和自动确认模式
* 设置消费者标签生成策略

```java
@Bean
public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = 
        new SimpleMessageListenerContainer(connectionFactory);

    container.setQueues(queue001());
    container.setConcurrentConsumers(1);
    container.setMaxConcurrentConsumers(5);
    container.setDefaultRequeueRejected(false);         // false不会requeue
    container.setAcknowledgeMode(AcknowledgeMode.AUTO);     // 自动签收
    container.setConsumerTagStrategy(queue -> queue + "_" + UUID.randomUUID().toString());
    container.setMessageListener(new ChannelAwareMessageListener() {
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            String msg = new String(message.getBody());
            System.out.println("-------------消费者：" + msg);
        }
    });

    return container;
}
```

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-31_20-12-10.png)

SimpleMessageListenerContainer相当于声明了一个消费者

### 4-8 MessageListenerAdaper使用

消息监听适配器

```java
@Bean
public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = 
        new SimpleMessageListenerContainer(connectionFactory);

    container.setQueues(queue001());
    container.setConcurrentConsumers(1);
    container.setMaxConcurrentConsumers(5);
    container.setDefaultRequeueRejected(false);         // false不会requeue
    container.setAcknowledgeMode(AcknowledgeMode.AUTO);     // 自动签收
    container.setConsumerTagStrategy(queue -> queue + "_" + UUID.randomUUID().toString());

    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    adapter.setDefaultListenerMethod("consumeMessage");	// 把默认方法名改为consumeMessage
    // adapter.setMessageConverter();   传给consumeMessage的默认参数是String，通过这个方法可以修改
    container.setMessageListener(adapter);

    return container;
}


public class MessageDelegate {
    /**
     * MessageListenerAdapter默认调用的方法名：handleMessage
     */
    public void handleMessage(String msg) {
        System.err.println("默认方法，消息内容：" + msg);
    }

    public void consumeMessage(String msg) {
        System.err.println("自定义方法，消息内容：" + msg);
    }
}
```

或者可以将队列和方法绑定

```java
@Bean
public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);

    // ..

    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    Map<String, String> map = new HashMap<>();
    map.put("queue001", "method001");         // 将queue001这个队列和method001这个方法绑定，凡是路由到queue001队列的消息都会被method001处理
    adapter.setQueueOrTagToMethodName(map);
    container.setMessageListener(adapter);

    return container;
}

public class MessageDelegate {
    public void handleMessage(String msg) {
        System.err.println("默认方法，消息内容：" + msg);
    }

    public void consumeMessage(String msg) {
        System.err.println("自定义方法，消息内容：" + msg);
    }
    
    public void method001(String msg) {
        System.err.println("method001方法，消息内容：" + msg);
    }
}
```

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-31_20-46-20.png)

### 4-13 RabbitMQ与SpringBoot整合

```properties
spring.rabbitmq.addresses=localhost:5672

# 默认就是guest
spring.rabbitmq.username=guest
# 默认就是guest
spring.rabbitmq.password=guest
# 默认就是/
spring.rabbitmq.virtual-host=/
#spring.rabbitmq.connection-timeout=15000

# 实现一个监听器用于监听Broker端给我们返回的确认请求:rabbitTemplate.setConfirmCallback();
spring.rabbitmq.publisher-confirms=true
# spring.rabbitmq.listener.simple.acknowledge-mode=auto 2.0.2版本不需要加这个配置，但是2.0.6需要加上这个配置

# 对不可达的消息进行后续的处理（需要mandatory=true），保证消息的路由成功:rabbitTemplate.setReturnCallback();
spring.rabbitmq.publisher-returns=true
spring.rabbitmq.template.mandatory=true
```

```java
@Component
public class RabbitSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    private final RabbitTemplate.ConfirmCallback confirmCallback = 
        (correlationData, ack, cause) -> {
            System.err.println("correlationData: " + correlationData);
            System.err.println("ack: " + ack);
            System.err.println("cause: " + cause);
            if (!ack) {
                System.err.println("异常处理...");
            }
    	};

    private final RabbitTemplate.ReturnCallback returnCallback = 
        (message, replyCode, replyText, exchange, routingKey) -> {
        	System.err.println("return exchange: " + exchange 
                               + ", routingKey: " + routingKey 
                               + ", replyCode: " + replyCode
                               + ", replyText: " + replyText);
    	};

    public void send(Object message, Map<String, Object> props) throws Exception {
        MessageHeaders mhs = new MessageHeaders(props);
        Message msg = MessageBuilder.createMessage(message, mhs);
        rabbitTemplate.setConfirmCallback(confirmCallback);
        rabbitTemplate.setReturnCallback(returnCallback);

        CorrelationData cd = new CorrelationData();
        cd.setId("vermouth");       // id + 时间戳 应该全局唯一
        rabbitTemplate.convertAndSend("exchange-1", "springboot.hello", msg, cd);

    }
}
```

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-31_22-48-41.png)

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-31_22-51-26.png)

## 第五章 高可靠，构建RabbitMQ集群架构

### 5-2 RabbitMQ集群架构模式

#### 主备模式

***主备模式***：（主节点提供读写，从节点不提供任何读写，只是备用；主从是主节点读写，从节点只读）实现RabbitMQ的高可用集群，一般在并发和数据量不高的情况下，这种模型非常地好用且简单。主备模式也成为Warren模式。从节点的目的是主节点宕机的时候，能够自动切换到从节点，此时从节点变成主节点继续提供读写。和activemq利用zookeeper做主备一样。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-11-05_20-08-53.png)

#### 远程模式

***远程模式***：远程模式可以实现双活的一种模式，简称Shovel模式，所谓Shovel就是我们可以把消息进行不同数据中心的复制工作，我们可以跨地域地让两个mq集群互联。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-11-05_20-18-29.png)

这种模式用得不多。

#### 镜像模式

***镜像模式***：保证100%数据不丢失，在实际工作中也是用的最多的。并且实现集群非常简单。

镜像队列，目的是为了保证rabbitmq数据的高可靠性解决方案，主要就是实现数据的同步，一般来讲实验2-3个节点。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-11-05_22-23-31.png)

#### 多活模式

***多活模式***：这种模式也是实现异地数据复制的主流模式，因为Shovel模式配置比较复杂。这种模型需要依赖rabbitmq的federation插件。采用多中心模式，那么在多个数据中心各部署一套RabbitMQ集群，各中心的RabbitMQ服务除了需要为业务提供正常地消息服务外，中心之间还需要实现部分队列消息共享。

### 5-7 RabbitMQ集群整合负载均衡基础组件HaProxy

HAProxy是一款提供高可用性，负载均衡以及基于TCP（第四层）和HTTP（第七层）应用的代理软件，支持虚拟主机，免费快速并且可靠。HAProxy特别适用于那些负载特大的web站点，这些站点通常又需要会话保持七层处理。HAProxy运行在时下的硬件上，完全可以支持数以万计的并发连接。

### 5-11 RabbitMQ集群恢复与故障转移的5中解决方案

节点A和B组成一个镜像队列，B是Master。

#### 场景一：A先停，B后停

**解决**：只要先启动B，再启动A即可。或者先启动A，30秒之后启动B即可回复镜像队列。

#### 场景二：A、B同时停机

**解决**：30s之内连续启动A和B即可

#### 场景三：A先停、B后停、且A无法回复

因为B是Master，所以等B起来之后，在B节点上调用控制台命令：```rabbitmqctl forget_cluster_node A``` 解除与A的Cluster关系，再将新的Slave节点加入B即可恢复镜像队列

#### 场景四：A先停、B后停、且B无法恢复

在3.1.x之前没有什么好的解决方案。因为B是主节点，所以直接启动A是不行的。在3.4.2以后，forget_cluster_node 支持 ```--offline``` 参数，就以为着当在A节点执行 ```rabbitmqctl forget_cluster_node --offline B``` 时，RabbitMQ会mock一个节点代表A，执行```forget_cluster_node```命令将B剔除cluster，然后A就可以正常启动了，最后将新的slave节点加入A即可重新恢复镜像队列。

#### 场景五：A先停。B后停，且A、B均无法恢复，但是能得到A或B的磁盘文件

只能通过恢复数据的方式去尝试恢复，将A或B的数据库文件默认在$RABBIT_HOME/var/lib/目录中，把它拷贝到新节点的对于mulxia，再将新节点的hostname改成A或B的hostname，如果是A节点（Slave）的磁盘文件，择按照场景4处理即可，如果是B节点（Master）的磁盘文件，择按照场景3处理，最后将新的Slave加入到新节点后完成恢复

## 第六章 SET化架构衍化与设计

### 6-2 衍变之路

对于庞大的大型分布式集群来说，会面临以下问题：

* 容灾问题
* 资源扩展问题
* 大集群拆分问题

#### 1. 容灾问题

核心服务（比如订单服务）挂掉，会影响全网所有用户，导致整个业务不可用。数据库主库集中在一个IDC，主机房挂掉，会影响全网所有用户，整个业务无法快速切换和恢复。

#### 2. 资源扩展问题

单IDC的资源（机器、网络带宽等）以及没法满足，扩展IDC时，存在跨机房访问时延问题（增加异地机房时，时延问题更加严重）；数据库主库单点，连接数有限，不能支持应用程序的持续扩展。

#### 3. 大集群拆分问题

分布式集群闺蜜扩大后，会相应地带来资源扩展、大集群拆分以及容灾问题

所以处于业务扩展性和容灾需求的考虑，需要一套从底层架构彻底解决问题的方案，业界主流解决方案：

***单元化架构方案***

#### 同城“双活”架构介绍

业务层面上以及做到真正的双活（或者多活），分别承担部分流量；

存储层面比如定时任务、缓存、持久层、数据分析等都是主从架构，会有跨机房写；

一个数据中心故障，可以手动切换流量，部分组件可以自动切换

#### 两地三中心结构介绍

使用灾备的思想，在同城"双活"的基础上，在异地部署一套灾备数据中心，每个中心都具有完备的数据处理能力，只有当主节点故障需要容灾的时候才会紧急启动备用数据中心。

**问题**：异地备份中心不工作，关键时刻不敢切；本质上数据仍然单点写，数据库瓶颈无法解

#### SET化方案介绍











