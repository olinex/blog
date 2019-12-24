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

现在让我们测试一下错误处理. 管道 1 现在是主节点, 将其进程杀死:

```bash
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
```

在Windows系统下:

```bash
> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f
```

其中一个附属节点已经自动切换为主节点, 节点 1 已经不再处于 `Isr` 节点集内了:

```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```

虽然主节点已经不再可用, 但是消息依然可以被消费:

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
--from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

## 第七步: 使用Kafka链接来导入/导出数据

在终端输入输出数据在一开始是非常便利的, 但是你更加有可能希望通过其他来源来输入数据, 或导出数据到其他的系统或服务中. 对于很多系统而言, 你可以使用Kafka的链接来导入/导出数据.

Kafka链接工具与Kafka集成. 它易于扩展并且可以执行自定义的逻辑. 在这里我们会示范如何通过Kafka链接工具运行一个简单的链接. 我们会将数据从文件中导入到Kafka主题中, 并将数据从主题中导出到文件内.

首先, 我们创建一些数据来进行测试:

```bash
> echo -e "foo\nbar" > test.txt
```

在Windows系统中:

```bash
> echo foo> test.txt
> echo bar>> test.txt
```

 接着, 我们创建两个单例模式的链接, 这意味着他们将只会运行一个独立的本地进程. 我们提供了三个配置文件的路径作为参数, 第一个文件必须是Kafka链接进程的配置文件, 它包含一些常见配置, 例如需要连接的管道和数据的序列化格式. 后续的配置文件每个都创建一个链接. 这些配置文件包含了一个唯一的连接名, 需要实例化的连接类, 和任何其他的配置信息.

```bash
> bin/connect-standalone.sh config/connect-standalone.properties \
config/connect-file-source.properties config/connect-file-sink.properties
```

 以下的示例配置文件, 使用了你早先启动的默认本地集群配置, 并创建了两个链接: 

* 第一个连接是输入来源连接, 它会从数据文件中读取每一行数据, 并将数据按行输入到Kafka主题中
* 第二个连接是输出接收连接, 它会从Kafka主题中读取消息, 并将每一行消息按行输出到文件中.

在执行的时候, 你会看到一些日志消息, 包含了一些链接的实例化信息. 当Kafka链接进程开始时, 来源连接应该从test.txt按行读取数据, 并将它们推送至 connect-test 主题中, 然后接收连接将会从 connect-test 主题中读取消息并将他们写入 test.sink.txt 文件内. 我们可以查看输出文件来检查测试结果:

```bash
> more test.sink.txt
foo
bar
```

你应该注意到, 数据已经被保存在了Kafka主题 connect-test 内, 因此你可以运行消费者终端来查看主体内的数据:

```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
... 
```

链接会持续处理数据, 因此如果我们向文件内添加数据, 你会看到它被推送至管道:

```bash
> echo Another line>> test.txt
```

你会发现这一行数据出现在了消费者终端和接收文件内.

## 第八步: 使用Kafka流来处理数据

Kafka流是为了构建实时业务处理应用和微服务而存在的客户端库, 当输入输出数据保存在Kafka集群后, Kafka流在客户端整合了标准的Java/Scala应用, 使它们受益于Kafka的服务端集群技术, 使得应用可扩展, 有弹性的, 容错的, 分布式的. 

