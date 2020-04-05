## 1.1 介绍
`Apache``Kafka`®是一个分布式流平台。这到底是什么意思呢？
流平台具有三个关键功能：

- 发布和订阅记录流，类似于消息队列或企业消息传递系统。
- 以容错的持久方式存储记录流。
- 处理记录流。

`Kafka`通常用于两大类应用程序：

- 建立实时流数据管道，以可靠地在系统或应用程序之间获取数据
- 构建实时流应用程序，以转换或响应数据流

要了解`Kafka`如何执行这些操作，让我们从头开始深入研究`Kafka`的功能。

首先几个概念：

- `Kafka`在一个或多个可以跨越多个数据中心的服务器上作为集群运行。
- `Kafka`集群将记录流存储在称为`topic`的类别中。
- 每个记录由一个键，一个值和一个时间戳组成。

`Kafka`具有四个核心`API`：

- [生产者`API`](https://kafka.apache.org/documentation.html#producerapi)允许应用程序发布的记录流至一个或多个`Kafaka`的`topic`。
- [消费者`API`](https://kafka.apache.org/documentation.html#consumerapi)允许应用程序订阅一个或多个`topic`，并处理所产生的对他们记录的数据流。
- [流`API`](https://kafka.apache.org/documentation/streams/)允许应用程序充当流处理器，从一个或多个`topic`消费的输入流，并产生一个输出流至一个或多个输出的`topic`，有效地将输入数据流转换成输出流。
- [连接器`API`](https://kafka.apache.org/documentation.html#connect)允许构建和运行可重复使用的生产者或消费者连接`kafaka``topic`到现有的应用程序或数据系统。例如，关系数据库的连接器可能会捕获对表的所有更改。

![](images/kafka-apis.png)

在`Kafka`中，客户端和服务器之间的通信是通过简单，高性能，与语言无关的`TCP`协议完成的。该协议已版本化，并与旧版本保持向后兼容性。我们为`Kafka`提供了`Java`客户端，但是客户端支持[多种语言](https://cwiki.apache.org/confluence/display/KAFKA/Clients)。

### Topic和日志
首先，让我们深入探讨`Kafka`提供的记录`topic`的核心抽象。

`topic`是将记录发布到的类别或订阅源名称。`Kafka`中的`topic`始终是多用户的；也就是说，一个`topic`可以有零个，一个或多个消费者来订阅写入该`topic`的数据。

对于每个`topic`，`Kafka`集群都会维护一个分区日志，如下所示：

![](images/log_anatomy.png)

每个分区都是有序的，不变的记录序列，这些记录连续地附加到结构化的提交日志中。分别为分区中的记录分配了一个顺序`ID`号，称为`offset`，该`ID`号唯一标识分区中的每个记录。

`Kafka`集群使用可配置的保留期限持久保留所有已发布的记录（无论是否已使用它们）。例如，如果将保留策略设置为两天，则在发布记录后的两天内，该记录可供使用，之后将被丢弃以释放空间。`Kafka`的性能相对于数据大小实际上是恒定的，因此长时间存储数据不是问题。

![](images/log_anatomy.png)

实际上，基于每个消费者保留的唯一元数据是该消费者在日志中的偏移量或位置。此偏移量由消费者控制：通常，消费者在读取记录时会线性地推进其偏移量，但是实际上，由于位置是由消费者控制的，因此它可以按喜欢的任何顺序使用记录。例如，消费者可以重置到较旧的偏移量以重新处理过去的数据，或者跳到最近的记录并从“现在”开始使用。

这些功能的组合意味着`Kafka`的消费者非常`cheap`-他们来来去去对集群或其他消费者没有太大影响。例如，您可以使用我们的命令行工具来“尾部”任何`topic`的内容，而无需更改任何现有消费者所消耗的内容。

日志中的分区有多种用途。首先，它们允许日志扩展到超出单个服务器所能容纳的大小。每个单独的分区都必须适合承载它的服务器，但是一个`topic`可能有很多分区，因此它可以处理任意数量的数据。其次，它们充当并行性的单元-稍有更多。

### 分布
日志的分区分布在`Kafka`群集中的服务器上，每台服务器处理数据并要求共享分区。每个分区都跨可配置数量的服务器复制，以实现容错功能。

每个分区都有一个充当"leader"的服务器和零个或多个充当"followers"的服务器。领导者处理对分区的所有读写请求，而跟随者则被动地复制领导者。如果领导者失败，则跟随者之一将自动成为新领导者。每个服务器充当其某些分区的领导者，而充当其他分区的跟随者，因此群集是负载均衡的。

### 地理复制
`Kafka` `MirrorMaker`为您的集群提供地理复制支持。使用`MirrorMaker`，可以在多个数据中心或云区域中复制消息。您可以在主动/被动方案中使用它进行备份和恢复。或在主动/主动方案中将数据放置在离您的用户更近的位置，或支持数据位置要求。

### 生产者
生产者将数据发布到他们选择的`topic`。生产者负责选择将哪个记录分配给`topic`中的哪个分区。可以以循环方式完成此操作，仅是为了平衡负载，也可以根据某些语义分区功能（例如基于记录中的某些键）进行此操作。一秒钟就可以了解更多有关分区的信息！

### 消费者
消费者使用消费者组名称标记自己，并且发布到`topic`的每条记录都会传递到每个订阅消费者组中的一个消费者实例。消费者实例可以在单独的进程中或在单独的机器上。

如果所有消费者实例都具有相同的消费者组，那么将在这些消费者实例上有效地平衡记录。

如果所有消费者实例具有不同的消费者组，则每个记录将广播到所有消费者进程。

![](images/consumer-groups.png)

由两台服务器组成的`Kafka`群集，其中包含四个带有两个消费者组的分区（`P0-P3`）。消费者组`A`有两个消费者实例，组`B`有四个。

但是，更常见的是，我们发现`topic`具有少量的消费者组，每个“逻辑订户”一个。每个组均由许多消费者实例组成，以实现可伸缩性和容错能力。这无非就是发布-订阅语义，其中订阅者是消费者的集群而不是单个进程。

在`Kafka`中实现使用的方式是通过在使用方实例上划分日志中的分区，以便每个实例在任何时间点都是分区“公平份额”的排他使用方。`Kafka`协议动态处理了维护组成员身份的过程。如果新实例加入该组，它们将接管该组其他成员的某些分区；如果实例死亡，则其分区将分配给其余实例。

`Kafaka`只提供了记录的总订单中的一个分区，而不是一个`topic`的不同分区之间。对于大多数应用程序，按分区排序以及按键对数据进行分区的能力就足够了。但是，如果您需要记录的总订单量，则可以使用只有一个分区的`topic`来实现，尽管这将意味着每个消费者组只有一个消费者。

### 多租户
您可以将`Kafka`部署为多租户解决方案。通过配置哪些`topic`可以产生或使用数据来启用多租户。配额也有运营支持。管理员可以在请求上定义和实施配额，以控制客户端使用的代理资源。有关更多信息，请参阅[安全性文档](https://kafka.apache.org/documentation/#security)。

### 保证
在较高级别上，`Kafka`提供以下保证：

- 生产者发送到特定`topic`分区的消息将按其发送顺序附加。也就是说，如果记录`M1`由与记录`M2`相同的生产者发送，并且首先发送`M1`，则`M1`的偏移量将小于`M2`，并在日志中更早出现。
- 消费者实例按记录在日志中的存储顺序查看记录。
- 对于具有复制因子`N`的`topic`，我们最多可以容忍`N-1`个服务器故障，而不会丢失任何提交给日志的记录。

在文档的设计部分中提供了有关这些保证的更多详细信息。

### `Kafka`作为消息传递系统
`Kafka`的流概念与传统的企业消息传递系统相比如何？

传统上，消息传递具有两种模型：队列和发布-订阅。在队列中，一组消费者可以从服务器中读取内容，并且每条记录都将转到其中一个。在发布-订阅记录中广播给所有消费者。这两个模型中的每一个都有优点和缺点。队列的优势在于，它允许您将数据处理划分到多个消费者实例上，从而扩展处理量。不幸的是，队列不是多用户的—一次进程读取了丢失的数据。发布-订阅允许您将数据广播到多个进程，但是由于每条消息都传递给每个订阅者，因此无法扩展处理。

`Kafaka`的消费者群体概念概括了这两个概念。与队列一样，消费者组允许您将处理划分为一组进程（消费者组的成员）。与发布订阅一样，`Kafka`允许您将消息广播到多个消费者组。

`Kafka`模型的优势在于，每个`topic`都具有这两个属性-可以扩展处理范围，并且是多订阅者-无需选择其中一个。

与传统的消息传递系统相比，`Kafka`还具有更强的订购保证。

传统队列将记录按顺序保留在服务器上，如果有多个消费者从队列中消费，则服务器将按记录的存储顺序分发记录。但是，尽管服务器按顺序分发记录，但是这些记录是异步传递给消费者的，因此它们可能在不同的消费者上乱序到达。这实际上意味着在并行使用的情况下会丢失记录的顺序。消息传递系统通常通过“专有消费者”的概念来解决此问题，该概念仅允许一个进程从队列中使用，但是，这当然意味着在处理中没有并行性。

`Kafaka`做得更好。通过在`topic`内具有并行性（即分区）的概念，Kafka能够在用户进程池中提供排序保证和负载均衡。这是通过将`topic`中的分区分配给消费者组中的消费者来实现的，以便每个分区都由组中的一个消费者完全消费。通过这样做，我们确保消费者是该分区的唯一读取器，并按顺序使用数据。由于存在许多分区，因此仍然可以平衡许多消费者实例上的负载。但是请注意，消费者组中的消费者实例不能超过分区。

### `Kafka`作为存储系统
任何允许发布与使用无关的消息发布的消息队列都有效地充当了运行中消息的存储系统。`Kafka`的不同之处在于它是一个非常好的存储系统。

写入`Kafka`的数据将写入磁盘并进行复制以实现容错功能。`Kafka`允许生产者等待确认，以便直到完全复制并确保即使写入服务器失败的情况下写入也不会完成。

`Kafka`的磁盘结构可以很好地扩展使用-无论服务器上有`50KB`还是`50TB`的持久数据，`Kafka`都将执行相同的操作。

认真对待存储并允许客户端控制其读取位置的结果是，您可以将`Kafka`视为一种专用于高性能，低延迟提交日志存储，复制和传播的专用分布式文件系统。

有关`Kafka`的提交日志存储和复制设计的详细信息，请阅读[此页面](https://kafka.apache.org/documentation/#design)。

### `Kafka`用于流处理
仅读取，写入和存储数据流是不够的，目的是实现对流的实时处理。

在`Kafka`中，流处理器是指从输入`topic`中获取连续数据流，对该输入进行一些处理并生成连续数据流以输出`topic`的任何东西。

例如，零售应用程序可以接受销售和装运的输入流，并输出根据此数据计算出的重新订购和价格调整流。

可以直接使用生产者和消费者`API`进行简单处理。但是，对于更复杂的转换，`Kafka`提供了完全集成的`Streams API`。这允许构建执行非重要处理的应用程序，这些应用程序计算流的聚合或将流连接在一起。

该功能有助于解决此类应用程序所面临的难题：处理无序数据，在代码更改时重新处理输入，执行状态计算等。

流`API`建立在`Kafka`提供的核心原语之上：它使用生产者和消费者`API`进行输入，使用`Kafka`进行状态存储，并使用相同的组机制来实现流处理器实例之间的容错。

### 拼凑在一起
消息，存储和流处理的这种组合看似不寻常，但这对于`Kafka`作为流平台的角色至关重要。

像`HDFS`这样的分布式文件系统允许存储静态文件以进行批处理。实际上，像这样的系统可以存储和处理过去的历史数据。

传统的企业消息传递系统允许处理将来的消息，这些消息将在您订阅后到达。以这种方式构建的应用程序会在将来的数据到达时对其进行处理。

`Kafka`结合了这两种功能，对于将`Kafka`用作流应用程序平台和流数据管道平台而言，这种结合至关重要。

通过结合存储和低延迟订阅，流应用程序可以以相同的方式处理过去和将来的数据。那是一个单一的应用程序可以处理历史数据，存储的数据，而不是在到达最后一条记录时结束，而是可以在将来的数据到达时继续进行处理。这是流处理的通用概念，它包含批处理以及消息驱动的应用程序。

同样，对于流数据管道，对实时事件的订阅组合使得可以将`Kafka`用于非常低延迟的管道。但是可靠地存储数据的能力使得可以将其用于必须保证数据传输的关键数据，或与仅定期加载数据或可能停机很长时间进行维护的脱机系统集成。流处理设施使得可以在数据到达时对其进行转换。

有关`Kafka`提供的担保，`API`和功能的更多信息，请参阅[本文档](https://kafka.apache.org/documentation.html)的其余部分。

## 1.2 用例
这是对`ApacheKafka®`的一些流行用例的描述。有关这些领域的概述，请参阅此[博客文章](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)。

### 消息传递
`Kafka`可以很好地替代传统邮件代理。消息代理的使用有多种原因（将处理与数据生产者分离，缓冲未处理的消息等）。与大多数邮件系统相比，`Kafka`具有更好的吞吐量，内置的分区，复制和容错能力，这使其成为大规模邮件处理应用程序的理想解决方案。
根据我们的经验，消息传递的使用通常吞吐量较低，但是可能需要较低的端到端延迟，并且通常取决于`Kafka`提供的强大的持久性保证。

在这个领域，`Kafka`与`ActiveMQ`或`RabbitMQ`等传统消息传递系统相当。

### 网站活动跟踪
`Kafka`最初的用例是能够将用户活动跟踪管道重建为一组实时的发布-订阅供稿。这意味着将网站活动（页面浏览，搜索或用户可能采取的其他操作）发布到中心`topic`，每种活动类型只有一个`topic`。这些提要可用于一系列用例的订阅，包括实时处理，实时监控，以及加载到`Hadoop`或脱机数据仓库系统中以进行脱机处理和报告。
活动跟踪通常量很大，因为每个用户页面视图都会生成许多活动消息。

### 指标
`Kafka`通常用于操作监控数据。这涉及汇总来自分布式应用程序的统计信息，以生成集中的运行数据提要。

### 日志汇总
许多人使用`Kafka`代替日志聚合解决方案。日志聚合通常从服务器收集物理日志文件，并将它们放在中央位置（也许是文件服务器或`HDFS`）以进行处理。`Kafka`提取文件的详细信息，并将日志或事件数据作为消息流进行更清晰的抽象。这允许较低延迟的处理，并且更容易支持多个数据源和分布式数据消耗。与以日志为中心的系统（如`Scribe`或`Flume`）相比，`Kafka`具有同样出色的性能，由于复制而提供的更强的耐用性保证以及更低的端到端延迟。

### 流处理
`Kafka`的许多用户在由多个阶段组成的处理管道中处理数据，其中原始输入数据从`Kafka``topic`中使用，然后进行汇总，充实或以其他方式转换为新`topic`，以供进一步使用或后续处理。例如，用于推荐新闻文章的处理管道可能会从`RSS`提要中检索文章内容，并将其发布到“文章”`topic`中。进一步的处理可能会使该内容规范化或重复数据删除，并将清洗后的文章内容发布到新`topic`中；最后的处理阶段可能会尝试向用户推荐此内容。这样的处理管道基于各个`topic`创建实时数据流的图形。从`0.10.0.0`开始，一个轻量但功能强大的流处理库称为`Kafka Streams`,可以在`Apache Kafka`中使用来执行上述数据处理。除了`Kafka Streams`之外，其他开源流处理工具还包括`Apache Storm`和`Apache Samza`。

### 事件源
[事件源](https://martinfowler.com/eaaDev/EventSourcing.html)是一种应用程序设计样式，其中状态更改以时间顺序记录记录。`Kafka`对大量存储的日志数据的支持使其成为使用这种样式构建的应用程序的绝佳后端。

### 提交日志
`Kafka`可以用作分布式系统的一种外部提交日志。该日志有助于在节点之间复制数据，并充当故障节点恢复其数据的重新同步机制。`Kafka`中的日志压缩功能有助于支持此用法。在这种用法中，`Kafka`与`Apache BookKeeper`项目相似。

## 1.3 快速开始
本教程假定您是从头开始的，并且没有现有的`Kafka`或`ZooKeeper`数据。由于`Kafka`控制台脚本在基于`Unix`的平台和`Windows`平台上有所不同，因此在`Windows`平台上使用`bin\windows\`代替`bin/`，并将脚本扩展名更改为`.bat`。

### 步骤1：下载程式码
下载`2.4.1`版本并解压缩。
```bash
tar -xzf kafka_2.12-2.4.1.tgz
cd kafka_2.12-2.4.1
```

### 步骤2：启动服务器
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

### 步骤3：建立主题
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

### 第4步：发送一些消息
`Kafka`带有一个命令行客户端，它将从文件或标准输入中获取输入，并将其作为消息发送到`Kafka`集群。默认情况下，每行将作为单独的消息发送。

运行生产者，然后在控制台中键入一些消息以发送到服务器。
```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

### 步骤5：启动消费者
`Kafka`还有一个命令行使用者，它将消息转储到标准输出。
```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```
如果上面的每个命令都在不同的终端上运行，那么您现在应该能够在生产者终端中键入消息，并看到它们出现在消费者终端中。

所有命令行工具都有其他选项。在不带参数的情况下运行该命令将显示用法信息，并对其进行详细记录。

### 步骤6：建立多`broker`集群
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

### 步骤7：使用Kafka Connect导入/导出数据
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

### 步骤8：使用Kafka Streams处理数据
`Kafka Streams`是用于构建关键任务实时应用程序和微服务的客户端库，其中输入和/或输出数据存储在`Kafka`集群中。`Kafka Streams`结合了在客户端编写和部署标准`Java`和`Scala`应用程序的简便性以及`Kafka`服务器端集群技术的优势，使这些应用程序具有高度可伸缩性，弹性，容错性，分布式等等。此快速入门示例将演示如何运行此库中编码的流应用程序。

## 1.4 生态系统
在主发行版之外，有很多工具可以与`Kafka`集成。该[生态系统](https://cwiki.apache.org/confluence/display/KAFKA/Ecosystem)页面列出的许多信息，包括流处理系统，`Hadoop`的集成，监控和部署工具。

## 1.5 从旧版本升级