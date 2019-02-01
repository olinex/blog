---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 快速开始

这个教程假设你是一个初学者, 并且没有安装Kafka或ZooKeeper. 因为在Windows和Unix系统中, Kafka的命令行脚本不同, 所以在Windows系统中, 使用 `bin\windows\` 替代 `bin/` , 并且将脚本后缀改成 `.bat`

## 第一步: 安装

[下载](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.1.0/kafka_2.11-2.1.0.tgz)并解压安装Kafka-2.1.0

```bash
> tar -xzf kafka_2.11-2.1.0.tgz
> cd kafka_2.11-2.1.0
```

## 第二步: 启动服务

Kafka使用了 [ZooKeeper](https://zookeeper.apache.org/) , 因此你需要先启动 ZooKeeper服务. 你可以使用和 Kafka 一起打包的便利脚本来安装一个粗糙的单节点 ZooKeeper 实例.

```bash
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```

开启Kafka服务

```bash
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

## 第三步: 创建主题

让我们来创建一个名叫 "test" 的分区, 它只有一个分区和一个副本:

```bash
> bin/kafka-topices.sh --create --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 --topic test
```

现在我们可以在主题列表内看到新创建的主题:

```bash
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

又或者, 你可以通过向未创建的主题推送数据, 可以自动创建主题并配置你的管道, 而不是手动创建.

## 第四步: 发送消息

Kafka提供了一个命令行终端, 可以从文件或标准输入中读取数据并作为消息发送给Kafka集群. 一般情况下, 每一行都将作为一条独立的消息.

运行生产者并在终端输入一些消息, 将其发送给服务.

```bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

## 第五步: 启动消费者

Kafka还提供了命令行终端作为消费者, 它会导出消息至标准输出.

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
--topic test --from-beginning
This is a message
This is another message
```

如果你在不同地界面运行以上的命令, 那么你现在将消息从生产者输入, 就可以看到新的消息出现在消费者界面.

所有的命令行工具都有附加地可选参数, 不加任何参数运行命令将会打印命令地使用文档.

## 第六步: 设置多通道集群

现在我们已经运行了一个管道, 不过这并不怎么有趣. 对于Kafka, 一个单通道只是一个单位集群, 运行更多的管道并不会改变什么. 只是为了体验一下, 让我们将集群扩展至三个节点 \( 依然在同一台本地机器 \).

首先让我们制作一个用于所有管道的配置文件 \( 在Windows, 使用 `copy` 命令 \) :

```bash
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
```

现在, 按照以下配置修改文件:

```text
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
```

`broker.id` 是节点在集群内唯一且永久的名称. 我们不得不重写日志目录和端口, 因为我们希望所有地节点都在运行在同一台服务器, 并且我们希望在同一个端口下注册或重写其他数据时, 都能保持管道不变.

现在我们已经有了ZooKeeper和一个单节点, 因此只需要启动另外两个节点:

```bash
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```

创建一个名为 `my-replicated-topic` 带有三个副本的主题:

```bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 \
--replication-factor 3 --partitions 1 --topic my-replicated-topic
```

要想查看集群中, 哪个管道正在做什么, 可以通过 "查看主题" 命令:

```text
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```

第一行给出了相关主题的所有分区的摘要信息, 后续的每一行代表了一个分区的信息. 在这里我们只有一个分区和 `my-replicated-topic` 相关, 因此只有一行. 

* leader - 主节点会响应对分区的读写请求. 每个节点将会是某个分区的主节点
* replicas - 是备份节点id的列表, 包括主节点和当前不可用的节点, 这些节点备份分区的日志
* isr - 是备份节点id列表的子集. 这些节点当前可用且被主节点控制

{% hint style="info" %}
在上述示例中, 节点1是仅有的一个分区的主节点
{% endhint %}

我们同样可以对 test 主题执行该命令:

```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test    PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test    Partition: 0    Leader: 0    Replicas: 0    Isr: 0
```

如我们所料, 这个主题没有备份并且处于服务 0 上, 我们只在集群上创建了这一个服务.

发送一些消息给我们的新主题:

```bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```



