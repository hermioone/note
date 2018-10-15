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



