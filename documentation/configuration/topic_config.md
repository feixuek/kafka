# 3.2 `Topic`配置

与`topic`相关的配置具有服务器默认值和可选的按`topic`覆盖。如果未提供按`topic`的配置，则使用服务器默认值。可以在`topic`创建时通过提供一个或多个`--config`选项来设置替代值。本示例使用自定义的最大邮件大小和刷新率创建一个名为`my-topic`的`topic`：
```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-topic --partitions 1 \
    --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
```
覆盖也可以在以后使用`alter configs`命令进行更改或设置。本示例更新`my-topic`的最大邮件大小：
```bash
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic
    --alter --add-config max.message.bytes=128000
```
要检查在`topic`上设置的替代，您可以执行
```bash
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name my-topic --describe
```
要删除替代，您可以执行
```bash
bin/kafka-configs.sh --zookeeper localhost:2181  --entity-type topics --entity-name my-topic
    --alter --delete-config max.message.bytes
```
以下是`topic`级别的配置。在“服务器默认属性”标题下给出了该属性的服务器默认配置。给定的服务器默认配置值仅在没有显式`topic`配置替代的情况下才适用于`topic`。

> `cleanup.policy`：`delete`或`compact`或两者兼有的字符串。该字符串指定要在旧日志段上使用的保留策略。当达到旧段的保留时间或大小限制时，默认策略（`delete`）将丢弃旧段。`compact`设置将启用有关该`topic`的日志压缩。
类型：`list` - 默认值：`delete` - 有效值：[`compact`,`delete`] - 服务器默认属性：`log.cleanup.policy` - 重要性：中等

> `compression.type`：指定给定`topic`的最终压缩类型。此配置接受标准压缩编解码器（"gzip"，"snappy"，"lz4"，"zstd"）。此外，它接受`uncompressed`，等同于不压缩。和`producer`，表示保留生产者设置的原始压缩编解码器。
类型：`string` - 默认值：`producer` - 有效值：[`uncompressed`，`zstd`，`lz4`，`snappy`，`gzip`，`producer`] - 服务器默认属性：`compression.type` - 重要性：中等

> `delete.retention.ms`：保留日志压缩`topic`的删除逻辑删除标记的时间。如果使用者从偏移量0开始，以确保他们获得最后阶段的有效快照，此设置还限制了使用者必须完成读取的时间（否则，在他们完成扫描之前，可能已收集了删除逻辑删除）。
类型：`long` - 默认值：86400000 - 有效值：[0，...] - 服务器默认属性：`log.cleaner.delete.retention.ms` - 重要性：中等

> `file.delete.delay.ms`：从文件系统中删除文件之前要等待的时间
类型：`long` - 默认值：60000 - 有效值：[0，...] - 服务器默认属性：`log.segment.delete.delay.ms` - 重要性：中等

> `flush.messages`：此设置允许指定一个间隔，在此间隔我们将强制对日志数据进行`fsyn`c。例如，如果将其设置为1，我们将在每条消息之后进行`fsync`；如果是`5`，则每隔5条消息就会进行`fsync`。通常，我们建议您不要设置此选项，而应使用复制来提高持久性，并允许操作系统的后台刷新功能，因为这样做效率更高。可以按`topic`覆盖此设置（请参阅按`topic`配置部分）。
类型：`long` - 默认值：9223372036854775807 - 有效值：[0，...] - 服务器默认属性：`log.flush.interval.messages` - 重要性：中等

> `flush.ms`：此设置允许指定一个时间间隔，在该时间间隔我们将强制将数据同步写入日志。例如，如果将其设置为`1000`，我们将在经过1000毫秒后进行`fsync`。通常，我们建议您不要设置此选项，而应使用复制来提高持久性，并允许操作系统的后台刷新功能，因为这样做效率更高。
类型：`long` - 默认值：9223372036854775807 - 有效值：[0，...] - 服务器默认属性：`log.flush.interval.ms` - 重要性：中等

> `follower.replication.throttled.replicas`：应在关注者端限制其日志复制的副本的列表。该列表应以`[PartitionId]：[BrokerId]，[PartitionId]：[BrokerId]：...`的形式描述一组副本，或者通配符`*`可用于限制该`topic`的所有副本。
类型：`list` - 默认值："" - 有效值：`[partitionId]：[brokerId]，[partitionId]：[brokerId]，...` - 服务器默认属性：`follower.replication.throttled.replicas` - 重要性：中等

> `index.interval.bytes`：此设置控制`Kafka`多久将一次索引条目添加到其偏移索引中。默认设置可确保我们大约每4096个字节为一条消息编制索引。索引越多，读取越接近日志中的确切位置，但索引越大。您可能不需要更改此设置。
类型：`int` - 默认值：4096 - 有效值：[0，...] - 服务器默认属性：`log.index.interval.bytes` - 重要性：中等

> `leader.replication.throttled.replicas`：应在引导者端限制其日志复制的副本的列表。该列表应以`[PartitionId]：[BrokerId]，[PartitionId]：[BrokerId]：...`的形式描述一组副本，或者通配符`*`可用于限制该`topic`的所有副本。
类型：`list` - 默认值："" - 有效值：`[partitionId]：[brokerId]，[partitionId]：[brokerId]，...` - 服务器默认属性：`Leader.replication.throttled.replicas` - 重要性：中等

> `max.compaction.lag.ms`：一条消息将保持不资格进行日志压缩的最长时间。仅适用于正在压缩的日志。
类型：`long` - 默认值：9223372036854775807 - 有效值：[1，...] - 服务器默认属性：`log.cleaner.max.compaction.lag.ms` - 重要性：中等

> `max.message.bytes`：`Kafka`允许的最大记录批处理大小。如果增加了该数量，并且有一些使用方的年龄大于`0.10.2`，则还必须增加使用方的获取大小，以便他们可以获取如此大的记录批次。在最新的消息格式版本中，为了提高效率，始终将记录分组。在以前的消息格式版本中，未压缩的记录不会分组，并且在这种情况下，此限制仅适用于单个记录。
类型：`int` - 默认值：1000012 - 有效值：[0，...] - 服务器默认属性：`message.max.bytes` - 重要性：中等

`message.format.version`：指定代理将消息附加到日志时所使用的消息格式版本。该值应为有效的`ApiVersion`。一些示例是：`0.8.2、0.9.0.0、0.10.0`，请检查`ApiVersion`以获取更多详细信息。通过设置特定的消息格式版本，用户可以证明磁盘上的所有现有消息均小于或等于指定的版本。错误地设置此值将导致使用较旧版本的使用者中断，因为他们将收到他们不理解的格式的消息。
类型：`string` - 默认值：2.4-IV1 - 有效值：`[0.8.0、0.8.1、0.8.2、0.9.0、0.10.0-IV0、0.10.0-IV1、0.10.1-IV0、0.10.1-IV1、0.10.1-IV2， 0.10.2-IV0、0.11.0-IV0、0.11.0-IV1、0.11.0-IV2、1.0-IV0、1.1-IV0、2.0-IV0、2.0-IV1、2.1-IV0、2.1-IV1、2.1- IV2、2.2-IV0、2.2-IV1、2.3-IV0、2.3-IV1、2.4-IV0、2.4-IV1]` - 服务器默认属性：`log.message.format.version` - 重要性：中等

> `message.timestamp.difference.max.ms`：代理接收消息时的时间戳与消息中指定的时间戳之间允许的最大差异。如果`message.timestamp.type = CreateTime`，则如果时间戳差异超过此阈值，则将拒绝邮件。如果`message.timestamp.type = LogAppendTime`，则忽略此配置。
类型：`long` - 默认值：9223372036854775807 - 有效值：[0，...] - 服务器默认属性：`log.message.timestamp.difference.max.ms` - 重要性：中等

> `message.timestamp.type`：定义消息中的时间戳是消息创建时间还是日志追加时间。该值应该是`CreateTime`或`LogAppendTime`
类型：`string` - 默认值：`CreateTime` - 有效值：[`CreateTime`，`LogAppendTime`] - 服务器默认属性：`log.message.timestamp.type` - 重要性：中等

> `min.cleanable.dirty.ratio`：此配置控制日志压缩器尝试清除日志的频率（假设日志压缩）已启用）。默认情况下，我们将避免清除已压缩超过50％的日志的日志。此比率限制了重复项在日志中浪费的最大空间（最多50％的日志可以重复）。更高的比率将意味着更少，更有效的清洁，但意味着更多的原木浪费空间。如果还指定了`max.compaction.lag.ms或min.compaction.lag.ms`配置，则日志压紧工具将在满足以下任一条件时立即认为日志有资格进行压紧：（i）达到脏率阈值并且日志至少在`min.compaction.lag.ms`持续时间内具有脏（未压缩）记录，或者（ii）如果日志最多在`max.compaction.lag.ms`期间内具有脏（未压缩）记录。
类型：`double` - 默认值：0.5 - 有效值：[0，...，1] - 服务器默认属性：`log.cleaner.min.cleanable.ratio` - 重要性：中等

> `min.compaction.lag.ms`：消息在日志中保持未压缩的最短时间。仅适用于正在压缩的日志。
类型：`long` - 默认值：0 - 有效值：[0，...] - 服务器默认属性：`log.cleaner.min.compaction.lag.ms` - 重要性：中等

> `min.insync.replicas`：当生产者将`acks`设置为`all`（或“ -1”）时，此配置指定必须确认写入才能使成功写入的最小副本数。如果无法满足此最小值，则生产者将引发异常（`NotEnoughReplicas`或`NotEnoughReplicasAfterAppend`）。
当一起使用，`min.insync.replicas`并且`acks`允许你强制执行更持久耐用的保证。典型的场景是创建一个复制因子3（设置`min.insync.replicas`为2）并产生acks`all`的`topic`。如果大多数副本未收到写入，这将确保生产者引发异常。
类型：`int` - 默认值：1 - 有效值：[1，...] - 服务器默认属性：`min.insync.replicas` - 重要性：中等

> `preallocate`：如果在创建新的日志段时应该在磁盘上预分配文件，则为`true`。
类型：`bool` - 默认值：`false` - 有效值： - 服务器默认属性：`log.preallocate` - 重要性：中等

> `retention.bytes`：此配置控制分区（包括日志段）可以增加到的最大大小，如果我们使用“删除”保留策略，则在我们丢弃旧日志段以释放空间之前。默认情况下，没有大小限制，只有时间限制。由于此限制是在分区级别上强制执行的，因此请将其乘以分区数以计算`topic`保留（以字节为单位）。
类型：`long` - 默认值：-1 - 有效值： - 服务器默认属性：`log.retention.bytes` - 重要性：中等

> `reserved.ms`：此配置控制如果我们使用“删除”保留策略，则在丢弃旧日志段以释放空间之前，我们将保留日志的最长时间。这表示有关消费者必须多长时间读取数据的SLA。如果设置为-1，则不应用时间限制。
类型：`long` - 默认值：604800000 - 有效值：[-1，...] - 服务器默认属性：`log.retention.ms` - 重要性：中等

> `segment.bytes`：此配置控制日志的段文件大小。保留和清除操作总是一次完成一个文件，因此，较大的段大小意味着更少的文件，但对保留的粒度控制较少。
类型：`int` - 默认值：1073741824 - 有效值：[14，...] - 服务器默认属性：`log.segment.bytes`重要性：中等

> `segment.index.bytes`：此配置控制将偏移量映射到文件位置的索引的大小。我们预分配该索引文件，并仅在日志滚动后收缩它。通常，您不需要更改此设置。
类型：`int` - 默认值：10485760 - 有效值：[0，...] - 服务器默认属性：`log.index.size.max.bytes` - 重要性：中等

> `segment.jitter.ms`：从计划的分段滚动时间中减去的最大随机抖动，以免分段滚动产生雷声
类型：`long` - 默认值：0 - 有效值：[0，...] - 服务器默认属性：`log.roll.jitter.ms` - 重要性：中等

> `segment.ms`：此配置控制`Kafka`将迫使日志滚动的时间段，即使段文件未满也可以确保保留可以删除或压缩旧数据。
类型：`long` - 默认值：604800000 - 有效值：[1，...] - 服务器默认属性：`log.roll.ms` - 重要性：中等

> `unclean.leader.election.enable`：指示是否将不在ISR集中的副本启用作为最后选择的领导者，即使这样做可能会导致数据丢失。
类型：`bool` - 默认值：`false`有效值： - 服务器默认属性：`unclean.leader.election.enable` - 重要性：中等

> `message.downconversion.enable`：此配置控制是否启用消息格式的下转换以满足消费请求。设置`false`为时，对于希望使用较旧消息格式的使用者，代理将不执行降频转换。`UNSUPPORTED_VERSION`对于来自此类较旧客户端的消费请求，代理会以错误响应。此配置不适用于复制到关注者所需的任何消息格式转换。
类型：`bool` - 默认值：`true` - 有效值： - 服务器默认属性：`log.message.downconversion.enable` - 重要性：低