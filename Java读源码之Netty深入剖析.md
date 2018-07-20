# Java读源码之Netty深入剖析

## 第三章 Netty服务端启动

两个问题：

- 服务端的socket在哪里初始化？（netty在哪里调用jdk底层的socket API？）
- 在哪里accept连接？

Netty服务端启动的过程：

1. 创建服务端Channel
2. 初始化服务端Channel
3. 注册selector
4. 端口绑定

### 3-2 服务端Channel的创建







