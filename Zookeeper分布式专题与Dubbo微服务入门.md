# Zookeeper分布式专题与Dubbo微服务入门

## 第1章 分布式系统概念与Zookeeper简介

### 1-2 什么是分布式系统

![](http://static.zybuluo.com/vermouth9/rypfwkuo3jt37cxu2ub0kds6/image.png)

面向服务式的开发

### 1-3 分布式系统的瓶颈以及zk的相关特性

* 一致性：数据一致性，数据按照顺序分批入库
* 原子性：事务要么成功要么失败，不会局部化
* 单一视图：客户端连接集群中的任一zk节点，数据都是一致的
* 可靠性：每次对zk的操作状态都会保存在服务端
* 实时性：客户端可以读取到zk服务端的最新数据

## 第2章 zookeeper的安装

