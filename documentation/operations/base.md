## 6.1 `Kafka`的基本操作
本节将回顾您将在`Kafka`集群上执行的最常见操作。本节中介绍的所有工具都可以在`bin/Kafka`发行版的目录下找到，并且如果不带任何参数运行，则每个工具都将在所有可能的命令行选项中显示详细信息。

### 添加和删​​除`topic`

您可以选择手动添加`topic`，或在首次将数据发布到不存在的`topic`时自动创建`topic`。如果自动创建`topic`，则可能需要调整用于自动创建`topic`的默认`topic`配置。
使用`topic`工具添加和修改`topic`：
```bash
bin/kafka-topics.sh --bootstrap-server broker_host:port --create --topic my_topic_name \
      --partitions 20 --replication-factor 3 --config x=y
```
复制因子控制多少服务器将复制每个写入的消息。如果您的复制因子为3，则最多2台服务器可能会失败，然后您将无法访问数据。我们建议您使用2或3的复制因子，以便可以透明地反弹计算机而不会中断数据使用。

分区计数控制将`topic`分片到的日志数量。分区计数有几个影响。首先，每个分区必须完全适合一台服务器。因此，如果您有20个分区，则全部数据集（以及读写负载）将由不超过20台服务器（不计算副本）处理。最后，分区数会影响使用者的最大并行度。在[概念部分](https://kafka.apache.org/documentation/#intro_consumers)将对此进行更详细的讨论。

每个分片的分区日志都放置在`Kafka`日志目录下的自己的文件夹中。此类文件夹的名称由`topic`名称，破折号（-）和分区ID组成。由于典型的文件夹名称不能超过255个字符，因此`topic`名称的长度受到限制。我们假设分区的数量永远不会超过100,000。因此，`topic`名称不能超过249个字符。这在文件夹名称中只留了足够的空间用于破折号和可能为5位数字的长分区ID。

命令行中添加的配置将覆盖服务器为应保留数据时间长度之类的默认设置。[此处](https://kafka.apache.org/documentation/#topicconfigs)记录了完整的按`topic`配置集。

### 修改`topic`
您可以使用相同的`topic`工具来更改`topic`的配置或分区。
要添加分区，您可以执行

```bash
bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic my_topic_name \
      --partitions 40
```
请注意，分区的一种用例是在语义上对数据进行分区，添加分区不会更改现有数据的分区，因此如果消费者依赖该分区，则可能会打扰消费者。就是说，如果通过`hash(key) % number_of_partitions`该方法对数据进行了分区，则该分区可能会通过添加分区而被打乱，但是`Kafka`不会尝试以任何方式自动重新分配数据。

要添加配置：
```bash
bin/kafka-configs.sh --bootstrap-server broker_host:port --entity-type topics --entity-name my_topic_name --alter --add-config x=y
```
删除配置：
```bash
bin/kafka-configs.sh --bootstrap-server broker_host:port --entity-type topics --entity-name my_topic_name --alter --delete-config x
```
最后删除一个`topic`：
```bash
bin/kafka-topics.sh --bootstrap-server broker_host:port --delete --topic my_topic_name
```
`Kafka`当前不支持减少`topic`的分区数。

在[此处](https://kafka.apache.org/documentation/#basic_ops_increase_replication_factor)可以找到有关更改`topic`复制因子的说明。

### 优雅关机
`Kafka`群集将自动检测到任何代理关闭或故障，并为该计算机上的分区选择新的领导者。无论服务器发生故障还是为了维护或配置更改而有意将其关闭，都会发生这种情况。对于后一种情况，`Kafk`a支持一种更优雅的机制来停止服务器，而不仅仅是杀死服务器。当服务器正常停止时，它会进行两项优化：

1. 它将所有日志同步到磁盘，以避免在重新启动时进行任何日志恢复（即，验证日志尾部所有消息的校验和）。日志恢复需要时间，因此可以加快有意重启的速度。
2. 在关闭之前，它将把服务器所领导的所有分区迁移到其他副本。这将使领导层转移更快，并将每个分区不可用的时间减少到几毫秒。

只要服务器停止运行（不是通过强行终止），就将自动同步日志，但是受控的领导者迁移需要使用特殊设置：
```
controlled.shutdown.enable=true
```
请注意，只有在代理上托管的所有分区都具有副本（即复制因子大于1 并且这些副本中至少有一个处于活动状态）时，受控关闭才会成功。这通常是您想要的，因为关闭最后一个副本会使该`topic`分区不可用。

### 领导平衡
每当代理停止或崩溃时，该代理的分区就会转移到其他副本。这意味着默认情况下，重新启动代理时，它将仅是其所有分区的跟随者，这意味着它将不用于客户端读取和写入。
为了避免这种不平衡，`Kafka`提出了首选副本的概念。如果分区的副本列表为`1,5,9`，则首选节点1作为节点5或9的引导者，因为它在副本列表中较早。您可以让`Kafka`集群尝试通过运行以下命令来恢复对已还原副本的领导权：

```bash
bin/kafka-preferred-replica-election.sh --zookeeper zk_host:port/chroot
```
由于运行此命令可能很乏味，因此您还可以通过设置以下配置来配置`Kafka`以自动执行此操作：
```
auto.leader.rebalance.enable=true
```

### 在整个机架上平衡副本
机架感知功能可将同一分区的副本分布在不同机架上。这扩展了`Kafka`对代理故障提供的保证以涵盖机架故障，从而限制了机架上所有代理立即失效时数据丢失的风险。该功能还可以应用于其他代理分组，例如`EC2`中的可用性区域。
您可以通过在代理配置中添加属性来指定代理属于特定机架：
```
broker.rack=my-rack-id
```
创建 `topic`，修改`topic`或重新分发副本时，将遵循机架约束，确保副本尽可能跨尽可能多的机架（分区将跨最小（#racks，复制因子）个不同的机架）。
用于将副本分配给代理的算法可确保每个代理的领导者数量是恒定的，而不管代理如何在机架中分布。这样可以确保均衡的吞吐量。

但是，如果为机架分配了不同数量的代理，则副本的分配将不会是偶数。具有较少代理的机架将获得更多副本，这意味着它们将使用更多存储空间并将更多资源用于复制。因此，在每个机架上配置相等数量的代理是明智的。

### 在集群之间镜像数据
我们指的是在“镜像” `Kafka`群集之间复制数据的过程，以避免与单个群集中节点之间发生的复制产生混淆。`Kafka`带有用于在`Kafka`集群之间镜像数据的工具。该工具从源群集使用并生成到目标群集。这种镜像的常见用例是在另一个数据中心中提供副本。下一节将详细讨论这种情况。
您可以运行许多此类镜像过程以提高吞吐量并提高容错能力（如果一个进程死了，其他进程将接管额外的负载）。

将从源集群中的`topic`读取数据，并将数据写入目标集群中的同名`topic`。实际上，镜子制造商不过是卡夫卡的消费者和生产商联系在一起而已。

源集群和目标集群是完全独立的实体：它们可以具有不同数量的分区，并且偏移量将不同。出于这个原因，镜像群集实际上并不是要用作容错机制（因为用户位置会有所不同）。为此，我们建议使用正常的集群内复制。但是，镜像制作过程将保留并使用消息密钥进行分区，因此将按密钥保留顺序。

这是一个示例，显示了如何从输入集群中镜像单个`topic`（名为`my-topic`）：
```bash
bin/kafka-mirror-maker.sh
      --consumer.config consumer.properties
      --producer.config producer.properties --whitelist my-topic
```
请注意，我们使用`--whitelist`选项指定`topic`列表。此选项允许使用`Java`风格的正则表达式的任何正则表达式。所以，你可以参照两个名为`topic`一个和乙使用`--whitelist 'A|B'`。或者，您可以使用来镜像所有`topic``--whitelist '*'`。确保引用任何正则表达式，以确保外壳程序不会尝试将其扩展为文件路径。为了方便起见，我们允许使用`'`，`'`代替`'|'` 指定`topic`列表。通过将镜像与配置结合在一起，`auto.create.topics.enable=true`就可以拥有一个副本集群，即使添加了新`topic`，该副本集群也会自动在源集群中创建和复制所有数据。

### 检查消费者位置
有时查看消费者的位置很有用。我们有一个工具，可以显示所有使用者在使用者组中的位置以及他们在日志末尾之后的位置。要在名为`my-group`的使用者组上运行此工具，并使用名为`my-topic`的`topic`，则该过程将如下所示：
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
 
TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID                                       HOST                           CLIENT-ID
my-topic                       0          2               4               2          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-topic                       1          2               3               1          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-topic                       2          2               3               1          consumer-2-42c1abd4-e3b2-425d-a8bb-e1ea49b29bb2   /127.0.0.1                     consumer-2
```

### 管理消费者群体
使用`ConsumerGroupCommand`工具，我们可以列出，描述或删除使用者组。使用者组可以手动删除，也可以在该组的最后提交的偏移量过期时自动删除。仅当组没有任何活动成员时，手动删除才有效。例如，要列出所有`topic`中的所有消费者组：
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
 
test-consumer-group
```
如前所述，要查看偏移量，我们像这样“描述”消费者组：
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
 
TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                    HOST            CLIENT-ID
topic3          0          241019          395308          154289          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic2          1          520678          803288          282610          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic3          1          241018          398817          157799          consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2
topic1          0          854144          855809          1665            consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1
topic2          0          460537          803290          342753          consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1
topic3          2          243655          398812          155157          consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4
```
有许多附加的“描述”选项可用于提供有关消费者组的更详细的信息：

- `--members`：此选项提供使用者组中所有活动成员的列表。
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --members
 
CONSUMER-ID                                    HOST            CLIENT-ID       #PARTITIONS
consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1       2
consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4       1
consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2       3
consumer3-ecea43e4-1f01-479f-8349-f9130b75d8ee /127.0.0.1      consumer3       0
```

- `--members --verbose`：除了上述`--members`选项报告的信息之外，此选项还提供分配给每个成员的分区。
```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --members --verbose
 
CONSUMER-ID                                    HOST            CLIENT-ID       #PARTITIONS     ASSIGNMENT
consumer1-3fc8d6f1-581a-4472-bdf3-3515b4aee8c1 /127.0.0.1      consumer1       2               topic1(0), topic2(0)
consumer4-117fe4d3-c6c1-4178-8ee9-eb4a3954bee0 /127.0.0.1      consumer4       1               topic3(2)
consumer2-e76ea8c3-5d30-4299-9005-47eb41f3d3c4 /127.0.0.1      consumer2       3               topic2(1), topic3(0,1)
consumer3-ecea43e4-1f01-479f-8349-f9130b75d8ee /127.0.0.1      consumer3       0               -
```

- `--offsets`：这是默认的`describe`选项，并提供与`--describe`选项相同的输出。
- `--state`：此选项提供有用的组级别信息。
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group --state
 
COORDINATOR (ID)          ASSIGNMENT-STRATEGY       STATE                #MEMBERS
localhost:9092 (0)        range                     Stable               4
```

要手动删除一个或多个使用者组，可以使用`--delete`选项：
```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete --group my-group --group my-other-group
 
Deletion of requested consumer groups ('my-group', 'my-other-group') was successful.
```
要重置使用者组的偏移量，可以使用`--reset-offsets`选项。此选项当时支持一个消费者组。它需要定义以下范围：`--all-topics`或`--topic`。除非您使用`--from-file`方案，否则必须选择一个范围。另外，首先请确保使用者实例处于非活动状态。有关更多详细信息，请参见[`KIP-122`](https://cwiki.apache.org/confluence/display/KAFKA/KIP-122%3A+Add+Reset+Consumer+Group+Offsets+tooling)。

它具有3个执行选项：

- (default) 以显示要重置的偏移量。
- `--execute`：执行`--reset-offsets`过程。
- `--export`：将结果导出为`CSV`格式。

`--reset-offsets`还具有以下场景可供选择（必须选择至少一个场景）：

- `--to-datetime <String：datetime>`：将偏移量重置为与`datetime`的偏移量。格式："YYYY-MM-DDTHH：mm：SS.sss"
- `--to-earliest`：将偏移量重置为最早的偏移量。
- `--to-latest`：将偏移量重置为最新偏移量。
- `--shift-by` `<long：偏移数>`：重置偏移量，将当前偏移量偏移`n`，其中`n`可以为正或负。
- `--from-file`：将偏移量重置为`CSV`文件中定义的值。
- `--to-current`：将偏移量重置为当前偏移量。
- `--by-duration <String：duration>`：将偏移量重置为从当前时间戳记的持续时间偏移量。格式：`PnDTnHnMnS`
- `--to-offset`：将偏移量重置为特定偏移量。

请注意，超出范围的偏移量将调整为可用的偏移量结束。例如，如果偏移量结束为10，偏移量请求为15，则实际上将选择偏移量为10。
例如，要将使用者组的偏移量重置为最新的偏移量：
```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --reset-offsets --group consumergroup1 --topic topic1 --to-latest
 
TOPIC                          PARTITION  NEW-OFFSET
topic1                         0          0
```
如果您正在使用旧的高级使用者并将组元数据存储在`ZooKeeper`中（即`offsets.storage=zookeeper`），请通过`--zookeeper`而不是`bootstrap-server`：
```bash
bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list
```

### 扩展集群
将服务器添加到`Kafka`集群很容易，只需为其分配唯一的代理`ID`，然后在新服务器上启动`Kafka`。但是，不会为这些新服务器自动分配任何数据分区，因此，除非将分区移至它们，否则在创建新`topic`之前它们将不会做任何工作。因此，通常在将计算机添加到群集时，您将需要将一些现有数据迁移到这些计算机。
数据迁移过程是手动启动的，但是是完全自动化的。在幕后，`Kafka`会将新服务器添加为要迁移的分区的追随者，并允许其完全复制该分区中的现有数据。新服务器完全复制该分区的内容并加入同步副本后，现有副本之一将删除其分区的数据。

分区重新分配工具可用于在代理之间移动分区。理想的分区分配将确保所有代理之间的数据负载和分区大小均匀。分区重新分配工具没有能力自动研究`Kafka`群集中的数据分布，并四处移动分区以实现均匀的负载分布。因此，管理员必须弄清楚应该移动哪些`topic`或分区。

分区重新分配工具可以在3种互斥模式下运行：

- `--generate`：在此模式下，给定`topic`列表和代理列表，该工具会生成候选重新​​分配，以将指定`topic`的所有分区移至新的代理。给定`topic`和目标代理的列表，此选项仅提供一种方便的方法来生成分区重新分配计划。
- `--execute`：在这种模式下，该工具将根据用户提供的重新分配计划启动分区的重新分配。（使用`--reassignment-json-file`选项）。这可以是管​​理员手工制作的自定义重新分配计划，也可以使用`--generate`选项提供
- `--verify`：在此模式下，该工具会验证上一次`--execute`期间列出的所有分区的重新分配状态。状态可以是成功完成，失败或进行中

#### 自动将数据迁移到新计算机
分区重新分配工具可用于将某些`topic`从当前代理集移到新添加的代理。这在扩展现有集群时通常很有用，因为与一次移动一个分区相比，将整个`topic`移至新的代理集更容易。用于执行此操作时，用户应提供应移至新的一组代理的`topic`列表和新代理的目标列表。然后，该工具将给定`topic`列表中的所有分区平均分配到新的一组代理中。在此过程中，`topic`的复制因子保持不变。实际上，`topic`输入列表的所有分区的副本都已从旧的代理集移到新添加的代理。
例如，以下示例将`topic``foo1`，`foo2`的所有分区移动到新的代理集`5,6`。在此步骤结束时，`topic`foo1和foo2的所有分区仅存在于代理5,6上。

由于该工具将`topic`的输入列表作为`json`文件接受，因此您首先需要确定要移动的`topic`并按以下方式创建`json`文件：
```bash
cat topics-to-move.json
{"topics": [{"topic": "foo1"},
            {"topic": "foo2"}],
"version":1
}
```
`JSON`文件准备好后，使用分区重新分配工具生成候选分配：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --topics-to-move-json-file topics-to-move.json --broker-list "5,6" --generate
Current partition replica assignment
 
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
              {"topic":"foo1","partition":0,"replicas":[3,4]},
              {"topic":"foo2","partition":2,"replicas":[1,2]},
              {"topic":"foo2","partition":0,"replicas":[3,4]},
              {"topic":"foo1","partition":1,"replicas":[2,3]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}
 
Proposed partition reassignment configuration
 
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
              {"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":2,"replicas":[5,6]},
              {"topic":"foo2","partition":0,"replicas":[5,6]},
              {"topic":"foo1","partition":1,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[5,6]}]
}
```
该工具生成候选分配，该分配会将所有分区从`topic``foo1`，`foo2`移至代理5,6。但是请注意，此时分区移动尚未开始，它仅告诉您当前分配和建议的新分配。如果您要回滚到当前分配，则应将其保存。新的赋值应保存在`json`文件（例如，`expand-cluster-reassignment.json`）中，然后使用`--execute`选项输入到工具中，如下所示：

```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --execute
Current partition replica assignment
 
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
              {"topic":"foo1","partition":0,"replicas":[3,4]},
              {"topic":"foo2","partition":2,"replicas":[1,2]},
              {"topic":"foo2","partition":0,"replicas":[3,4]},
              {"topic":"foo1","partition":1,"replicas":[2,3]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}
 
Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
              {"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":2,"replicas":[5,6]},
              {"topic":"foo2","partition":0,"replicas":[5,6]},
              {"topic":"foo1","partition":1,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[5,6]}]
}
```
最后，`--verify`选项可与该工具一起使用，以检查分区重新分配的状态。请注意，应将相同的`expand-cluster-reassignment.json`（与`--execute`选项一起使用）与`--verify`选项一起使用：

```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [foo1,0] completed successfully
Reassignment of partition [foo1,1] is in progress
Reassignment of partition [foo1,2] is in progress
Reassignment of partition [foo2,0] completed successfully
Reassignment of partition [foo2,1] completed successfully
Reassignment of partition [foo2,2] completed successfully
```

#### 自定义分区分配和迁移
分区重新分配工具还可以用于将分区的副本选择性地移动到一组特定的代理。以这种方式使用时，假定用户知道重新分配计划，并且不需要工具生成候选重新​​分配，从而有效地跳过了`--generate`步骤并直接进入`--execute`步骤
例如，以下示例将`topic``foo1`的分区0移动到代理5,6，将`topic``foo2`的分区1移动到代理2,3：

第一步是在`json`文件中手工制作自定义重新分配计划：

```bash
cat custom-reassignment.json
{"version":1,"partitions":[{"topic":"foo1","partition":0,"replicas":[5,6]},{"topic":"foo2","partition":1,"replicas":[2,3]}]}
```
然后，将`json`文件与`--execute`选项一起使用以开始重新分配过程：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file custom-reassignment.json --execute
Current partition replica assignment
 
{"version":1,
"partitions":[{"topic":"foo1","partition":0,"replicas":[1,2]},
              {"topic":"foo2","partition":1,"replicas":[3,4]}]
}
 
Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}
```
`--verify`选项可与该工具一起使用，以检查分区重新分配的状态。请注意，应将相同的`expand-cluster-reassignment.json`（与`--execute`选项一起使用）与`--verify`选项一起使用：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file custom-reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [foo1,0] completed successfully
Reassignment of partition [foo2,1] completed successfully
```

### 退役经纪人
分区重新分配工具尚无法自动生成用于使经纪人退役的重新分配计划。因此，管理员必须提出重新分配计划，以将要停用的代理上托管的所有分区的副本移动到其余代理。这可能相对繁琐，因为重新分配需要确保所有副本都不会从退役经纪人转移到仅另一个经纪人。为了使此过程轻松进行，我们计划在将来为退役经纪人添加工具支持。

#### 复制因子增加
增加现有分区的复制因子很容易。只需在自定义重新分配`json`文件中指定额外的副本，然后将其与`--execute`选项一起使用即可增加指定分区的复制因子。
例如，以下示例将`topic``foo`的分区0的复制因子从1增加到3。在增加复制因子之前，该分区的唯一副本存在于代理5上。作为增加复制因子的一部分，我们将在经纪人6和7。

第一步是在`json`文件中手工制作自定义重新分配计划：
```bash
cat increase-replication-factor.json
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}
```
然后，将`json`文件与`--execute`选项一起使用以开始重新分配过程：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute
Current partition replica assignment
 
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5]}]}
 
Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}
```
`--verify`选项可与该工具一起使用，以检查分区重新分配的状态。请注意，应将`--verify`选项与同`一crement-replication-factor.json`（与`--execute`选项一起使用）一起使用：

```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --verify
Status of partition reassignment:
Reassignment of partition [foo,0] completed successfully
```
您还可以使用`kafka-topics`工具验证复制因子的增加：
```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic foo --describe
Topic:foo   PartitionCount:1    ReplicationFactor:3 Configs:
  Topic: foo    Partition: 0    Leader: 5   Replicas: 5,6,7 Isr: 5,6,7
```

#### 限制数据迁移期间的带宽使用
`Kafka`允许您对复制流量应用限制，在用于将副本在计算机之间移动的带宽上设置上限。当重新平衡群集，引导新的代理或添加或删除代理时，此功能很有用，因为它可以限制这些数据密集型操作对用户的影响。
有两个接口可用于接合油门。最简单，最安全的方法是在调用`kafka-reassign-partitions.sh`时应用油门，但`kafka-configs.sh`也可用于直接查看和更改油门值。
因此，例如，如果要执行重新平衡，请使用以下命令，它将以不超过50MB/s的速度移动分区。
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --execute --reassignment-json-file bigger-cluster.json —throttle 50000000
```
当您执行此脚本时，您将看到油门启动：
```bash
The throttle limit was set to 50000000 B/s
Successfully started reassignment of partitions.
```
如果您希望在重新平衡期间更改油门，比如说要增加吞吐量以使其更快地完成，则可以通过重新运行传递相同的`reassignment-json-file`的`execute`命令来执行此操作：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181  --execute --reassignment-json-file bigger-cluster.json --throttle 700000000
  There is an existing assignment running.
  The throttle limit was set to 700000000 B/s
```
重新平衡完成后，管理员可以使用`--verify`选项检查重新平衡的状态。如果重新平衡完成，则将通过`--verify`命令删除油门。重要的是，一旦重新平衡完成，管理员可以通过使用`--verify`选项运行命令来及时删除限制。否则，可能会限制常规复制流量。

当执行`--verify`选项并完成重新分配后，脚本将确认已取消节流阀：
```bash
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181  --verify --reassignment-json-file bigger-cluster.json
Status of partition reassignment:
Reassignment of partition [my-topic,1] completed successfully
Reassignment of partition [mytopic,0] completed successfully
Throttle was removed.
```
管理员还可以使用`kafka-configs.sh`验证分配的配置。有两对节气门配置，用于管理节流过程。油门值本身。这是使用动态属性在代理级别配置的：
```bash
leader.replication.throttled.rate
  follower.replication.throttled.rate
```
还有一组枚举的受限制的副本：
```bash
leader.replication.throttled.replicas
  follower.replication.throttled.replicas
```
每个`topic`都配置了哪些。所有四个配置值都由`kafka-reassign-partitions.sh`自动分配（在下面讨论）。

要查看油门极限配置：
```bash
bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type brokers
Configs for brokers '2' are leader.replication.throttled.rate=700000000,follower.replication.throttled.rate=700000000
Configs for brokers '1' are leader.replication.throttled.rate=700000000,follower.replication.throttled.rate=700000000
```
这显示了应用于复制协议的引导方和跟随方的限制。默认情况下，为双方分配相同的限制吞吐量值。

要查看限制副本的列表：
```bash
bin/kafka-configs.sh --describe --zookeeper localhost:2181 --entity-type topics
Configs for topic 'my-topic' are leader.replication.throttled.replicas=1:102,0:101,
    follower.replication.throttled.replicas=1:101,0:102
```
在这里，我们看到引导者限制应用于代理102上的分区1和代理101的分区0。同样，跟随者限制应用于代理101上的分区1和代理102上的分区0。

默认情况下，`kafka-reassign-partitions.sh`会将引导者限制应用于重新平衡之前存在的所有副本，其中任何一个都可能是领导者。它将从动油门应用于所有移动目的地。因此，如果在代理程序101,102上存在一个具有副本的分区，并将其重新分配给102,103，则该分区的前导调节器将应用于101,102，而后继调节器将仅应用于103。

如果需要，还可以使用kafka-configs.sh上的--alter开关手动更改油门配置。

#### 安全使用限制复制
使用限制复制时，应格外小心。特别是：

（1）节气门拆卸：

重新分配完成后，应及时卸下节流阀（通过运行`kafka-reassign-partitions -verify`）。
（2）确保进度：

如果将节流阀设置得太低，则与进入的写入速率相比，复制可能无法进行。在以下情况下会发生这种情况：

> max(BytesInPerSec)>throttle

其中`BytesInPerSec`是监视生产者到每个代理中的写入吞吐量的度量。

管理员可以使用度量标准来监视重新平衡期间复制是否在进行中：
```bash
kafka.server：type = FetcherLagMetrics，name = ConsumerLag，clientId =（[-。\ w] +），topic =（[-。\ w] +），partition =（[0-9] +）
```
在复制过程中，延迟应不断减少。如果度量标准未降低，则管理员应如上所述提高节流吞吐量。

### 设定配额
配额覆盖和默认值可以在（用户，客户端ID），用户或客户端ID水平中所述被配置在这里。默认情况下，客户端收到无限配额。可以为每个（用户，客户端ID），用户或客户端ID组设置自定义配额。
为（`user = user1`，`client-id = clientA`）配置自定义配额：

```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1 --entity-type clients --entity-name clientA
Updated config for entity: user-principal 'user1', client-id 'clientA'.
```
为`user = user1`配置自定义配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1
Updated config for entity: user-principal 'user1'.
```
为`client-id = clientA`配置自定义配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type clients --entity-name clientA
Updated config for entity: client-id 'clientA'.
```
通过指定`--entity-default`选项而不是`--entity-name`，可以为每个（用户，客户端ID），用户或客户端ID组设置默认配额。
为`user = userA`配置默认的客户端ID配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-name user1 --entity-type clients --entity-default
Updated config for entity: user-principal 'user1', default client-id.
```
配置用户的默认配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type users --entity-default
Updated config for entity: default user-principal.
```
配置客户端ID的默认配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --alter --add-config 'producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200' --entity-type clients --entity-default
Updated config for entity: default client-id.
```
以下是描述给定（用户，客户端ID）的配额的方法：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-name user1 --entity-type clients --entity-name clientA
Configs for user-principal 'user1', client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```
描述给定用户的配额：
```
bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-name user1
Configs for user-principal 'user1' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```
描述给定客户端ID的配额：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type clients --entity-name clientA
Configs for client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```
如果未指定实体名称，则将描述指定类型的所有实体。例如，描述所有用户：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users
Configs for user-principal 'user1' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
Configs for default user-principal are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```
对于（用户，客户端）类似：
```bash
bin/kafka-configs.sh  --zookeeper localhost:2181 --describe --entity-type users --entity-type clients
Configs for user-principal 'user1', default client-id are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
Configs for user-principal 'user1', client-id 'clientA' are producer_byte_rate=1024,consumer_byte_rate=2048,request_percentage=200
```
通过在代理上设置这些配置，可以设置适用于所有客户端ID的默认配额。仅当在`Zookeeper`中未配置配额替代或默认值时，才应用这些属性。默认情况下，每个client-id都会收到无限配额。以下将每个生产者和消费者客户端ID的默认配额设置为10MB/秒。
```bash
quota.producer.default=10485760
quota.consumer.default=10485760
```
请注意，这些属性已被弃用，并可能在将来的版本中删除。使用`kafka-configs.sh`配置的默认值优先于这些属性。