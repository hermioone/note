# Spring AMQP

## 3 参考

### 3.1 使用Spring AMQP

#### 3.1.1 AMQP抽象

Spring AMQP有几个模块组成，包括：

* spring-amqp

  不依赖任何具体的AMQP的实现，具有可移植性

* spring-rabbit

#### 3.1.2 连接和资源管理

spring-rabbit模块中的核心组件是```org.springframework.amqp.rabbit.connection.Connection ```接口，它是```com.rabbitmq.client.Connection ```的封装。模块中提供的唯一具体实现是```CachingConnectionFactory```。默认情况下，会建立一个被应用***共享***的Connection，因为AMQP中的工作单元是Channel。

当创建```CachingConnectionFactory```的实例，应该提供hostname、port、username、password。

```java
connectionFactory.setChannelCacheSize(25);        
//调整缓存的大小，如果超过25个信道被释放，那么25个信道会进入缓存，剩下的会被关掉。
//默认值是25

connectionFactory.setCacheMode(CachingConnectionFactory.CacheMode.CONNECTION);            
//设置缓存Connection
//默认值是CachingConnectionFactory.CacheMode.CHANNEL

connectionFactory.setConnectionLimit(25);
/*
设置总共允许的连接总数，如果连接数量达到限制，channelCheckoutTimeLimit is used to wait for a connection to become 空闲
如果超时，则抛出AmqpTimeoutException。
*/

connectionFactory.setChannelCheckoutTimeout(10);
/*
如果这个值大于0，那么信道数量如果超过缓存大小就不会创建，阻塞这个值的时间，如果超过这个时间，则抛出AmqpTimeoutException。
*/
```

> 当缓存模式为时`CONNECTION`，自动声明队列等不受支持。 

> **注：**
>
> 在框架内创建的信道（例如：RabbitTemplate）会被可靠地返还给缓存，但是如果框架之外创建通道（例如，通过直接访问连接并调用`createChannel()`），则必须可靠地返回它们（通过关闭）`finally`，以避免用尽通道。 

> **重要：**
>
> 当应用只有一个单例的CachingConnectionFactory时，当connection被阻塞时，应用程序会停止工作。

##### Connection and Channel Listeners

```ConnectionFactory```支持注册```ConnectionListener```和```ChannelListener```。```ConnectionListener```被```RabbitAdmin```用来在连接建立时进行声明（exchange，queue，binding）。

##### 日志记录频道关闭事件

```java
connectionFactory.setCloseExceptionLogger(new ConditionalExceptionLogger() {
    @Override
    public void log(Log log, String s, Throwable throwable) {
        ///
    }
});
```

在`CachingConnectionFactory`使用默认策略登录通道封闭如下：

- 正常通道关闭（200 OK）不记录。
- 如果通道由于被动队列声明失败而关闭，则会在DEBUG级别进行记录。
- 如果某个频道`basic.consume`由于排他性消费者条件而被拒绝，它将在INFO级别登录。
- 所有其他人都记录在ERROR级别。

要修改此行为，`ConditionalExceptionLogger`请将自定义注入到 `CachingConnectionFactory`其`closeExceptionLogger`属性中。

##### RabbitMQ自动重连

Spring AMQP的自动重连机制和amqp-client提供的重启机制兼容。

#### 3.1.4 AmqpTemplate

Spring AMQP提供了一个核心抽象模版接口：AmqpTemplate。它定义所有的发送和接口的基础操作。目前，只有一个具体的实现：RabbitTemplate。

##### 添加重试功能

可以给``` RabbitTemplate ```配置``` RetryTemplate ```来处理连接问题

```java
RabbitTemplate template = new RabbitTemplate（connectionFactory（））;
RetryTemplate retryTemplate = new RetryTemplate（）;
ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy（）;
backOffPolicy.setInitialInterval（500）;
backOffPolicy.setMultiplier（10.0）;
backOffPolicy.setMaxInterval（10000）;
retryTemplate.setBackOffPolicy（backOffPolicy）;
template.setRetryTemplate（retryTemplate）;
```

##### 发布是异步的 - 如何检测成功还是失败

考虑两种失败的情景：

* 发布到一个exchange，但是没有匹配的queue
* 发布到一个不存在的exchange

第一种情况稍后讨论。（发布确认和返回）

第二种情况消息会被drop，没有返回，对应的信道因为异常而关闭。默认情况下，异常会被记录，但是可以通过在```connection```上注册ConnectionListener来获取这些事情的详情。

```java
this.connectionFactory.addConnectionListener(new ConnectionListener() {
    @Override
    public void onCreate(Connection connection) {
    }
    @Override
    public void onShutDown(ShutdownSignalException signal) {
    	...
    }
});
```

You can examine the signal’s reason property to determine the problem that occurred.

To detect the exception on the sending thread, you can ```setChannelTransacted(true)``` on the ```RabbitTemplate``` and the exception will be detected on the txCommit(). However, **transactions significantly impede performance** so consider this carefully before enabling transactions for just this one use case.

##### 发布确认和返回

对于消息返回，template的 ```mandatory``` 必须为true或者 ```mandtory-expression``` 对于一个特定消息为true。

```java
template.setMandatory(true);

template.setReturnCallback(new RabbitTemplate.ReturnCallback() {
    @Override
    public void returnedMessage(Message message, int i, String s, String s1, String s2) {
		...
    }
});
```

一个RabbitTemplate只能有一个 ```ReturnCallback``` 。

对于发布确认，```CachingConnectionFactory``` 的 ```publisherConfirms``` 属性需要设置为true。

```java
connectionFactory.setPublisherConfirms(true);

template.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {

    }
});
```

```CorrelationData``` 是客户端发送原始信息时提供的对象。The ```ack``` is true for an ack and false for a nack.

