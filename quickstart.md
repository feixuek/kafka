# 快速开始
本教程假定您是从头开始的，并且没有现有的`Kafka`或`ZooKeeper`数据。由于`Kafka`控制台脚本在基于`Unix`的平台和`Windows`平台上有所不同，因此在`Windows`平台上使用`bin\windows\`代替`bin/`，并将脚本扩展名更改为`.bat`。

## 步骤1：下载程式码
下载`2.4.1`版本并解压缩。
```bash
tar -xzf kafka_2.12-2.4.1.tgz
cd kafka_2.12-2.4.1
```

## 步骤2：启动服务器
`Kafka`使用`ZooKeeper`，因此如果您还没有，请首先启动`ZooKeeper`服务器。您可以使用`kafka`随附的便利脚本来获取快速且肮脏的单节点`ZooKeeper`实例。
```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```
现在启动`kafka`服务器
```bash
bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

## 步骤3：建立主题
让我们用一个分区和一个副本创建一个名为"test"的`topic`：
```bash
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```
现在，如果我们运行`list topic`命令，我们可以看到该`topic`：
```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
test
```
或者，除了手动创建`topic`外，还可以将代理配置为在发布不存在的`topic`时自动创建`topic`。

## 第4步：发送一些消息
`Kafka`带有一个命令行客户端，它将从文件或标准输入中获取输入，并将其作为消息发送到`Kafka`集群。默认情况下，每行将作为单独的消息发送。

运行生产者，然后在控制台中键入一些消息以发送到服务器。
```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

## 步骤5：启动消费者
`Kafka`还有一个命令行使用者，它将消息转储到标准输出。
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```
如果上面的每个命令都在不同的终端上运行，那么您现在应该能够在生产者终端中键入消息，并看到它们出现在消费者终端中。

所有命令行工具都有其他选项。在不带参数的情况下运行该命令将显示用法信息，并对其进行详细记录。

## 步骤6：建立多`broker`集群
到目前为止，我们一直在与一个`broker`竞争，但这并不有趣。对于`Kafka`来说，单个代理只是一个大小为1的集群，因此除了启动更多的代理实例之外，没有什么太大的变化。但是，只是为了感受一下，让我们将集群扩展到三个节点（仍然全部在本地计算机上）。

首先，我们为每个代理创建一个配置文件（在`Windows`上，使用`copy`命令代替）：
```bash
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```
现在编辑这些新文件并设置以下属性：
```bash
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
```
该`broker.id`属性是集群中每个节点的唯一且永久的名称。我们只需要覆盖端口和日志目录，这是因为我们都在同一台计算机上运行它们，并且希望所有代理都不要试图在同一端口上注册或覆盖彼此的数据。

我们已经有`Zookeeper`并启动了单个节点，因此我们只需要启动两个新节点：
```bash
bin/kafka-server-start.sh config/server-1.properties &
...
bin/kafka-server-start.sh config/server-2.properties &
...
```
现在，创建一个具有三个复制因子的新`topic`：
```bash
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
好的，但是现在有了集群，我们如何知道哪个`broker`在做什么？要查看该命令，请运行"描述`topic`"命令：
```bash
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```
这是输出的说明。第一行给出了所有分区的摘要，每一行都给出了有关一个分区的信息。由于该`topic`只有一个分区，因此只有一行。

- “领导者”是负责给定分区的所有读取和写入的节点。每个节点将成为分区的随机选择部分的领导者。
- “副本”是为该分区复制日志的节点列表，无论它们是引导者还是当前处于活动状态。
- `isr`是“同步”副本的集合。这是副本列表的子集，当前仍处于活动状态并追随领导者。

请注意，在我的示例中，节点1是主题唯一分区的领导者。

我们可以在创建的原始`topic`上运行相同的命令，以查看其位置：
```bash
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```
因此，这里没有任何惊喜-原始`topic`没有副本，并且位于服务器0上，这是我们创建群集时集群中唯一的服务器。

让我们向我们的新`topic`发布一些消息：
```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```
现在让我们使用这些消息：
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```
现在让我们测试一下容错能力。`broker1`扮演领导者的角色，所以让我们杀死它：
```bash
ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
kill -9 7564
```
在`Windows`上使用：
```bash
wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
taskkill /pid 6016 /f
```
领导层已切换为关注者之一，并且节点1不再位于同步副本集中：
```bash
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```
但是，即使最初进行写操作的领导者已经下线，消息仍然可供使用：
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

## 步骤7：使用Kafka Connect导入/导出数据
从控制台写入数据并将其写回到控制台是一个方便的起点，但是您可能要使用其他来源的数据或将数据从`Kafka`导出到其他系统。对于许多系统，可以使用`Kafka Connect`导入或导出数据，而无需编写自定义集成代码。

`Kafka Connect`是`Kafka`附带的工具，用于将数据导入和导出到`Kafka`。它是运行连接器的可扩展工具，该 连接器实现用于与外部系统进行交互的自定义​​逻辑。在本快速入门中，我们将看到如何使用简单的连接器运行`Kafka Connect`，该连接器将数据从文件导入到`Kafka``topic`，并将数据从`Kafka``topic`导出到文件。

首先，我们将从创建一些种子数据开始进行测试：
```bash
echo -e "foo\nbar" > test.txt
```
或在`Windows`上：
```bash
echo foo> test.txt
echo bar>> test.txt
```
接下来，我们将启动两个以独立模式运行的连接器，这意味着它们将在单个本地专用进程中运行。我们提供了三个配置文件作为参数。第一个始终是`Kafka Connect`流程的配置，其中包含通用配置，例如要连接的`Kafka`代理和数据的序列化格式。其余的配置文件均指定要创建的连接器。这些文件包括唯一的连接器名称，要实例化的连接器类，以及连接器所需的任何其他配置。
```bash
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```
这些样本配置文件（随`Kafka`一起提供）使用您先前启动的默认本地集群配置并创建两个连接器：第一个是源连接器，该连接器从输入文件中读取行并将每个行生成到`Kafka``topic`，第二个是接收器连接器从`Kafka``topic`读取消息，并在输出文件中将它们作为一行显示。

在启动过程中，您将看到许多日志消息，其中包括一些表明正在实例化连接器的消息。`Kafka Connect`进程启动后，源连接器应开始从`test.txt`主题中读取行并将其生成到主题`connect-test`，而接收器连接器应开始从`topic`中读取消息`connect-test` 并将其写入文件`test.sink.txt`。我们可以通过检查输出文件的内容来验证数据已通过整个管道传递：

```bash
more test.sink.txt
foo
bar
```
请注意，数据存储在`Kafka``topic``connect-test`中，因此我们也可以运行控制台使用者以查看该`topic`中的数据（或使用自定义使用者代码进行处理）：
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```
连接器继续处理数据，因此我们可以将数据添加到文件中，并查看它在管道中的移动情况：
```bash
echo Another line>> test.txt
```
您应该看到该行出现在控制台使用者输出和接收器文件中。

## 步骤8：使用Kafka Streams处理数据
`Kafka Streams`是用于构建关键任务实时应用程序和微服务的客户端库，其中输入和/或输出数据存储在`Kafka`集群中。`Kafka Streams`结合了在客户端编写和部署标准`Java`和`Scala`应用程序的简便性以及`Kafka`服务器端集群技术的优势，使这些应用程序具有高度可伸缩性，弹性，容错性，分布式等等。此快速入门示例将演示如何运行此库中编码的流应用程序。