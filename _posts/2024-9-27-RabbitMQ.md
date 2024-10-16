# 消息通信

## 通信方式——AMQP

应用程序和MQ建立TCP连接后，每次进行通信都是建立一个线程，通过AMQP信道通信

AMQP可以创建并行的传输层，不会被TCP链接约束

## 队列

### 多个消费者订阅

以轮询的方式分发

### 死信队列

服务端拒绝消息并设置不重新发送，消息将会投递到死信队列，

## 交换器

### 分类

- direct：路由键匹配投递到相应队列-
- fanout：收到消息后广播发送给所有绑定的队列
- topic：可以基于通配符匹配路由键
- headers：匹配AMQP消息的header而不是路由键

## 持久化

持久化消息要满足以下条件

- 将投递模式设置为2（持久）
- 发到持久化交换器
- 到达持久化队列

### 持久性日志

- 当发布一条持久化消息到持久化交换器上时，Rabbit会在消息提交到日志文件后发送响应
- 如果消息路由到非持久化队列，将会自动从持久性日志删除
- 如果从持久化队列消费消息，并且确认，将在持久化日志中标记为等待垃圾回收

### 发送方确认模式

将消息持久化是否成功告知发送方

- 异步确认，发送方会在一定时间内等待接收方的确认消息
- 没有回滚概念 (同事务相比)，更加轻量级

# 集群

## 架构

RabbitMQ会始终记录以下四种类型的内部元数据

- 队列元数据——队列名称和它的属性（是否可持久化，是否自动删除？）
- 交换器元数据——交换器名称、类型和属性
- 绑定元数据——保存如何将消息路由到消息队列
- vhost元数据——为vhost内的队列，交换器和绑定提供命名空间和安全属性

### 集群中的队列

每一个节点并不拥有所有队列的拷贝，原因：

- 存储空间——添加新的节点不会带来更多的存储空间
- 性能——消息的发布需要复制到每一个节点。对于需要持久化的消息还有磁盘活动。每次新增节点，都会增加网络和磁盘的活动

集群中只有唯一结点负责特定队列，其他节点需要将接收到的该队列的信息传递给该队列的所有者节点。

### 分布式交换器

- 交换器实质是一个名称和一个队列绑定列表

- 当消息发送到交换器时，实际上有你所连接的信道将消息上的路由键同交换器的绑定列表进行比较，然后路由信息。
- 因此交换器在集群的所有节点上

#### 作用

- 如果有节点故障时，不需要重新声明交换器，只需要让故障节点上的消费者重新连接到集群上，它们立即可以发送消息
- AMQP、发送方确认模式，需要判断是否可以路由到目标节点，交换器就可以检测是否可路由

### 内存节点和磁盘节点

集群的节点要么是内存节点，要么是磁盘节点

- 内存结点：所有元数据保存在内存。提供性能
- 磁盘节点：所有元数据保存在磁盘中，单节点只允许磁盘节点。保障集群的配置信息

要求集群中至少有一个磁盘节点。其他可以是内存节点

如果唯一一个磁盘节点故障，集群可以继续路由，但不能有以下操作：

- 创建队列、交换器、绑定关系
- 添加用户
- 更改权限
- 添加或删除集群节点                                                                                                                                                                                                                                                                                                                                              





