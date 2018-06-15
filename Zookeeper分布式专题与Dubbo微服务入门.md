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

下载加压```zookeeper-x.tar.gz```，配置环境变量

```shell
export JAVA_HOME=/usr/jdk8
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin
```

目录结构：

* bin：主要的一些运行命令
* conf：存放配置文件，其中需要修改```zk.cfg```
* contrib：附加的一些功能
* dist-maven：mvn编译后的目录
* docs：文档
* lib：需要依赖的jar包
* recipes：案例demo代码
* src：源码

### 2-4 zookeeper配置文件介绍，运行zk

把```conf/zoo_sample.cfg```复制为```zoo.cfg```，修改```zoo.cfg```

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10    # 用于集群，允许从节点连接并同步到master节点的初始化连接时间，以tickTime的倍数来表示
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5		# 用于集群，master主节点与从节点之间的心跳机制，以tickTime的倍数来表示
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/usr/local/zookeeper/dataDir			# 必须配置
dataLogDir=/usr/local/zookeeper/dataLogDir		 # 日志目录，如果不配置会和dataDir共用
# the port at which the clients will connect
clientPort=2181				# 客户端连接服务器的端口，默认2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

```shell
./zkServer.sh start
./zkServer.sh status
./zkServer.sh stop
./zkServer.sh restart
```

## 第3章 Zookeeper的基本数据模型

### 3-1 zk数据模型介绍

* 是一个树形结构。
* zk的数据模型也可以理解为linux\unix的文件目录。
* 每一个节点都称之为znode，它可以有子节点，也可以有数据。
* 每个节点分为临时节点和永久节点，临时节点在客户端断开后消失。
* 每个zk节点都有各自的版本号，可以通过命令行来显示节点信息。
* 每当节点数据发生变化，那么该节点的版本号会累加（和数据库中的乐观锁相似）。
* 删除/修改过时节点，版本号不匹配则会报错。
* 每个zk节点存储的数据不宜过大，几k即可。
* 节点可以设置权限acl，可以通过权限来限制用户的访问。

### 3-2 zk客户端连接/关闭服务端，查看znode

#### 客户端连接

```bin/zkCli.sh```

#### 查看znode结构

```ls /```

#### 关闭客户端连接

```quit```

### 3-3 zookeeper的作用

1. **master节点选举**：当主节点挂了后，从节点就会接手工作，并且保证这个节点是唯一的，这也是所谓的首脑模式，从而保证我们的集群是高可用的。

2. **统一配置文件管理**：只需要部署一台服务器，则可以把相同的配置文件同步更新到其他所有服务器，此操作在云计算机中用的特别多。

3. **发布与订阅**：类似消息队列MQ。dubbo发布者把数据存在znode上，订阅者会读取这个数据。

4. **提供分布式锁**：分布式环境中不同进程之间争夺资源，类似于多线程中的锁。

   ![](http://static.zybuluo.com/vermouth9/4fc2h2ndvtwuhmlq960p475h/image.png)

5. **集群管理**：集群中保证数据的强一致性，客户端不管链接任何节点的时候，都可以读到一致的数据。

   ![](http://static.zybuluo.com/vermouth9/a8inbok73203dhojgy9n6c8z/image.png)

## 第4章 ZK基本特性与基于Linux的ZK客户端命令行

### 4-2 session的基本原理与create命令的使用

#### session的基本原理

* 客户端与服务端之间的连接存在会话
* 每个会话都可以设置一个超时时间
* 心跳结束，session则过期
* session过期，则临时节点znode会被抛弃
* 心跳机制：客户端向服务端的ping包请求

#### zk常用命令行操作

* ```create [-s] [-e] path data acl```：

  * ```create /imooc imooc-data```：默认创建的节点，非顺序，持久化

  * ```get /imooc```：

    ```properties
    imooc-data
    cZxid = 0x4
    ctime = Fri Jun 15 13:49:52 CST 2018
    mZxid = 0x4
    mtime = Fri Jun 15 13:49:52 CST 2018
    pZxid = 0x4
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 10
    numChildren = 0
    ```

  * ```create -e /imooc imooc-data```：创建临时的节点

  * ```get /imooc```：

    ```properties
    imooc-data
    cZxid = 0x4
    ctime = Fri Jun 15 13:49:52 CST 2018
    mZxid = 0x4
    mtime = Fri Jun 15 13:49:52 CST 2018
    pZxid = 0x5
    cversion = 1			# 版本号做了一次更新
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 10
    numChildren = 1
    ```

  * ```get /imooc/tmp```：

    ```properties
    imooc-data
    cZxid = 0x5
    ctime = Fri Jun 15 13:52:45 CST 2018
    mZxid = 0x5
    mtime = Fri Jun 15 13:52:45 CST 2018
    pZxid = 0x5
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x100004196e80001			# 说明是临时节点
    dataLength = 10
    numChildren = 0
    ```

  * ```create -s /imooc/sec sec```：创建顺序的节点

    ```shell
    Created /imooc/sec0000000001
    
    # 如果再执行一次create -s /imooc/sec se
    Created /imooc/sec0000000002
    ```

### 4-4 zk特性-理解watcher机制

在zkCli中可以通过help查看有哪些命令可以设置watcher

```shell
stat path [watch]
set path data [version]
ls path [watch]
delquota [-n|-b] path
ls2 path [watch]
setAcl path acl
setquota -n|-b val path
history 
redo cmdno
printwatches on|off
delete path [version]
sync path
listquota path
rmr path
get path [watch]
create [-s] [-e] path data acl
addauth scheme auth
quit 
getAcl path
close 
connect host:port
```



* 针对每个节点的操作，都会有一个监督者->watcher
* 当监控的某个对象（znode）发生了变化，则触发watcher事件
* zk中的watcher是一次性的，触发后立即销毁
* 父节点、子节点的增删改都能够触发其watcher
* 针对不同类型的操作，触发的watcher事件也不同
  * （子）节点创建事件
  * （子）节点删除事件
  * （子）节点数据变化事件

### 4-5 父节点watcher事件

* 创建父节点触发：NodeCreated

  * ```stat /imooc watch```

  * ```create /imooc 123```

    ```shell
    WATCHER::
    
    WatchedEvent state:SyncConnected type:NodeCreated path:/imooc
    Created /imooc
    ```

* 修改父节点数据触发：NodeDataChanged

  * ```get /imooc watch```：也可以用```stat /imooc watch```

  * ```set /imooc 789```

    ```shell
    WATCHER::
    
    WatchedEvent state:SyncConnected type:NodeDataChanged path:/imooc
    cZxid = 0x14
    ctime = Fri Jun 15 14:10:56 CST 2018
    mZxid = 0x16
    mtime = Fri Jun 15 14:14:11 CST 2018
    pZxid = 0x14
    cversion = 0				
    dataVersion = 2				# 修改数据的版本
    aclVersion = 0				# acl的版本
    ephemeralOwner = 0x0
    dataLength = 3
    numChildren = 0
    ```

* 删除父节点触发：NodeDeleted

  * ```stat /imooc watch```

  * ```delete /imooc```

    ```shell
    WATCHER::
    
    WatchedEvent state:SyncConnected type:NodeDeleted path:/imooc
    ```

### 4-6 子节点watcher事件

* ```ls```为父节点设置watcher，创建子节点触发：NodeChildrenChanged

  * 如果父节点不存在，首先创建父节点（create /imooc leihou）

  * ```ls /imooc watch```：注意必须用```ls```

  * ```create /imooc/abc alielie```：

    ```shell
    WATCHER::
    
    WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/imooc
    Created /imooc/abc
    ```

* ```ls```为父节点设置watcher，删除子节点触发：NodeChildrenChanged

  * ```ls /imooc watch```

  * ```delete /imooc/abc```：

    ```shell
    WATCHER::
    
    WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/imooc
    ```

* **创建和删除子节点都只会触发父节点的NodeChildrenChanged事件**，因为父节点不关心子节点究竟怎么变化。

* ```ls```为父节点设置watcher，修改子节点不触发事件：客户端

  * ```create /imooc/xyz leihou```

  * ```ls /imooc watch```

  * ```set /imooc/xyz 9090```

    ```shell
    cZxid = 0x21
    ctime = Fri Jun 15 14:26:24 CST 2018
    mZxid = 0x22
    mtime = Fri Jun 15 14:26:43 CST 2018
    pZxid = 0x21
    cversion = 0
    dataVersion = 1
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 4
    numChildren = 0
    ```

### 4-7 watcher常用使用场景

***统一资源配置***

假设有一个zookeeper集群，主机更新了客户端的数据库配置信息（/imooc/sqlConfig），其他的节点也都跟着改变，因此都触发watcher，这样可以通知每个主机相连的客户端去更新配置信息。

### 4-8 权限acl详解，acl的构成：schema与id

* 针对节点可以设置相关读写等权限，目的是为了保障数据安全性
* 权限permissions可以指定不同的权限范围以及角色

ACL命令行

* ```getAcl path```：获取某个节点的acl权限信息
* ```setAcl path acl```：设置某个节点的acl权限信息
* ```addauth scheme auth```：输入认证授权信息，注册时输入明文密码（登录），但是在zk的系统里，密码是以加密的形式存在的。

ACL的构成

* zk的acl通过```[scheme:id:permissions]```来构成权限列表

  * scheme：代表采用的某种权限机制

    * world：world下只有一个id：anyone，那么组合的写法就是```world:anyone:[permissions]```

    * auth：代表认证登录，需要注册用户有权限就可以，形式为```auth:user:password:[permissions]```

    * digest：需要对密码加密才能访问，与auth的区别：就是auth明文，digest密文。组合为：

      ```digest:username:BASE64(SHA1(password)):[permissions]```

    * ip：当设置为ip指定的ip地址，此时限制ip进行访问，比如：```ip:192.168.1.1:[permissions]```

    * super：代表超级管理员，拥有所有的权限

  * id：代表允许访问的用户

  * permissions：权限组合字符串

    权限字符串缩写：c、r、d、w、a

    * CREATE：创建子节点
    * READ：获取节点/子节点
    * DELETE：删除子节点
    * WRITE：设置节点数据
    * ADMIN：设置权限

### 4-10 acl命令行讲解

#### 1. world

```create /imooc/abc leihou```

```getAcl /imooc/abc```：

```shell
'world,'anyone
: cdrwa
```

```setAcl /imooc/abc world:anyone:crwa```：

```shell
cZxid = 0x29
ctime = Fri Jun 15 14:58:55 CST 2018
mZxid = 0x29
mtime = Fri Jun 15 14:58:55 CST 2018
pZxid = 0x29
cversion = 0		
dataVersion = 0	
aclVersion = 1				# 权限版本发生变化
ephemeralOwner = 0x0
dataLength = 2
numChildren = 0
```

```getAcl /imooc/abc```：

```shell
'world,'anyone
: crwa
```

```delete /imooc/abc/123```：

```shell
Authentication is not valid : /imooc/abc/123
```

#### 2. auth

使用auth的话必须先注册

```setAcl /imooc/abc auth:imooc:imooc:cdrwa```

```shell
Acl is not valid : /names/imooc
```

```addauth digest imooc:imooc```

```setAcl /imooc/abc auth:imooc:imooc:cdrwa```：

* 这里也可以匿名设置：
  * ```setAcl /imooc/abc auth::cdrwa```
  * 因为这个设置的用户名和密码其实和addauth中的一样的，也就是说即使这里的用户名和密码随便设置，但是getAcl的时候，得到的还是```imooc:XwEDaL3J0JQGkRQzM0DpO6zMzZs=```，因为addauth中设置的就是```imooc:XwEDaL3J0JQGkRQzM0DpO6zMzZs=```

```getAcl /imooc/abc```

```shell
'digest,'sherry:XwEDaL3J0JQGkRQzM0DpO6zMzZs=				# 数据库中存的是密文
: cdrwa
```

#### 3. digest

假设已经注册好了用户imooc:imooc

```setAcl /names/test digest:imooc:XwEDaL3J0JQGkRQzM0DpO6zMzZs=:cdra```

```getAcl /names/test```

```shell
'digest,'imooc:XwEDaL3J0JQGkRQzM0DpO6zMzZs=
: cdra
```

```get /names/test```

```shell
Authentication is not valid : /names/test
```

```addauth digest imooc:imooc```

```get /names/test```

```shell
leihou
cZxid = 0x36
ctime = Fri Jun 15 15:20:44 CST 2018
mZxid = 0x36
mtime = Fri Jun 15 15:20:44 CST 2018
pZxid = 0x36
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

#### 4. super

1. 修改```zkServer.sh```增加super管理员
2. 重启```zkServer.sh```

### 4-15 acl的使用场景

* 开发/测试环境分离，开发者无权操作测试库的节点，只能看。
* 生产环境上控制指定ip的服务可以访问相关节点，防止混乱。

### 4-16 zk的四字命令

* zk可以通过它自身提供的简写命令来和服务器进行交互
* 需要使用到nc命令，安装：```yum install nc```
* ```echo [command] | nc [ip] [port]```

```echo conf | nc localhost 2181```

```properties
clientPort=2181
dataDir=/usr/local/zookeeper/dataDir/version-2
dataLogDir=/usr/local/zookeeper/dataLogDir/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=0
```

## 第5章 选举模式和zookeeper的集群安装

![](http://static.zybuluo.com/vermouth9/8g4bvixi9id65ky7rct0jme8/image.png)

当主节点挂掉后，zz这个从节点经过选举称为主节点，此时原来的主节点如果恢复了，就会变成从节点。

选举模式要求集群有奇数个节点，所以集群中最少有3个节点。

### 5-2 搭建伪分布式集群

1. 把zookeeper1复制两份：zookeeper2，zookeeper3

2. 修改zookeeper1下的conf中的```zoo.cfg```：

   ```properties
   # The number of milliseconds of each tick
   tickTime=2000
   # The number of ticks that the initial
   # synchronization phase can take
   initLimit=10
   # The number of ticks that can pass between
   # sending a request and getting an acknowledgement
   syncLimit=5
   # the directory where the snapshot is stored.
   # do not use /tmp for storage, /tmp here is just
   # example sakes.
   dataDir=/usr/local/zookeeper1/dataDir					# 需要修改
   dataLogDir=/usr/local/zookeeper1/dataLogDir				# 需要修改
   # the port at which the clients will connect
   clientPort=2181										  # 需要修改
   # the maximum number of client connections.
   # increase this if you need to handle more clients
   #maxClientCnxns=60
   #
   # Be sure to read the maintenance section of the
   # administrator guide before turning on autopurge.
   #
   # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
   #
   # The number of snapshots to retain in dataDir
   #autopurge.snapRetainCount=3
   # Purge task interval in hours
   # Set to "0" to disable auto purge feature
   #autopurge.purgeInterval=1
   
   
   server.1=192.168.1.103:2888:3888				# 288x是集群中数据同步用的端口号
   server.2=192.168.1.103:2889:3889				# 388x是选举模式用的端口号
   server.3=192.168.1.103:2890:3890
   ```

3. 进入zookeeper1下的dataDir目录，创建一个文件```myid```，在```myid```中输入1

4. 修改zookeeper2和zookeeper3下的conf中的```zoo.cfg```，把对应的1分别换成2和3，也在zookeeper2和zookeeper3下重复步骤3，只是分别输入2和3

### 5-3 搭建真实的集群

和搭建伪分布式集群过程一样，只不过3个server的ip不相同，端口相同。

## 第6章 使用ZooKeeper原生Java API

* 会话连接与恢复
* 节点的增删改查
* watch与acl的相关操作

























## 备注

* 这种较短的命令行是输入的命令：

  ```create /imooc/abc leihou```

* 这种整行的命令行是输入命令后的显示结果

  ```shell
  'world,'anyone
  : cdrwa
  ```







