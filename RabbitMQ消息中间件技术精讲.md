# RabbitMQ消息中间件技术精讲 

## 第一章 课程介绍

### 1-3 业界主流消息中间件介绍

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

### 2-3 RabbitMQ高性能的原因

* Erlang语言最初在于交换机领域，这样使得RabbitMQ在Broker之间进行数据交互的性能是非常优秀的
* Erlang的优点：Erlang有着和原生Socket一样的延迟

### 2-4 AMQP高级消息队列协议与模型

AMQP定义：是具有现代特征的二进制协议。是应用层协议的一个开放标准，为面向消息的中间件设计。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-15_22-51-31.png)

> 生产者投递到Exchange上，消费者从Message Queue中取数据

#### 2-5 AMQP核心概念讲解

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

### 2-6 RabbitMQ整体架构与消息流转

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-59-07.png)

### 2-7 RabbitMq环境安装

* 服务的启动：```rabbitmq-server start &```
* 服务的停止：```rabbitmqctl stop_app```
* 管理插件：```rabbitmq-plugins enable rabbitmq_management```

5672是java端进行通信的端口号；15672是管控台的端口号；25672是集群通信的端口号。

### 2-9 命令行与管理台结合讲解

凡是可以在管理台执行的操作都可以通过命令行来操作。

#### 高级操作

* ```rabbitmqctl reset```：移除所有数据，要在```rabbitmqctl stop_app```之后使用
* ```rabbitmqctl change_cluster_node_type disc | ram```：修改集群节点的存储形式
* ```rabbitmqctl forget_cluster_node [--offline]```：忘记节点（摘除节点）

### 2-10 生产者消费者模型构建

需要先运行消费者，因为**在消费者中声明了队列**

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-44-54.png)

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-43-34.png)

如图所示，有一个问题：

***为什么生产者绑定的routingKey和消费者监听的queue名字一样时（“test001”），消息也可以路由过去？***

> RabbitMQ的生产者在发送消息时，必须要指定一个exchange，如果exchange为空的话（""），会走第一个exchange（```AMQP default```），这个exchange的路由规则就是按照routingKey的名字来寻找同名的queue，如果有，就把消息路由过去；如果没有，就把消息删除。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-18_21-51-18.png)

### 2-12 交换机详解

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

### 2-15 绑定、队列、消息、虚拟主机详解

* 绑定：Exchange和Exchange、Queue之间的连接关系。绑定中可以包含routingKey或者参数

* 队列：实际存储消息数据

  * Durability：是否持久化
  * Auto Delete：为true时，当最后一个监听被移除后，该Queue会自动被删除

* 消息：服务器和应用程序之间传送的数据。由Properties和Payload组成

  * 常用属性：delivery mode、headers（自定义属性）

  * 其它属性：

    * content_type、content_encoding、priority
    * correlation_id、replay_to、expiration、message_id
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











