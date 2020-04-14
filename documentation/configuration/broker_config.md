# 3.1 `broker`配置

基本配置如下：
- `broker.id`
- `log.dirs`
- `zookeeper.connect`

`topic`级别的配置和默认值将在下面详细讨论。

> **`zookeeper.connect`**：以以下形式指定`ZooKeeper`连接`string`：`hostname:port`,`host`和`port`是`ZooKeeper`服务器的主机和端口。要允许在`ZooKeeper`计算机停机时通过其他`ZooKeeper`节点进行连接，您还可以在表单中指定多个主机`hostname1:port1,hostname2:port2,hostname3:port3`。服务器还可以在其`ZooKeeper`连接`string`中包含一个`ZooKeeper chroot`路径，该路径将其数据放在全局`ZooKeeper`命名空间中的某个路径下。例如，要给您一个`chroot`路径，`/chroot/path`请将连接`string`指定为`hostname1:port1,hostname2:port2,hostname3:port3/chroot/path`。
>
> **类型**：`string` - **默认值**：- **有效值**： - **重要性**：高 - **更新模式**：只读

> __`advertised.host.name`__：不推荐使用：仅在`advertised.listeners`或`listeners`未设置时使用。使用`advertised.listeners`代替。要发布给`ZooKeeper`的主机名，供客户端使用。在`IaaS`环境中，这可能需要与代理绑定的接口不同。如果未设置，它将使用`host.name`配置的值。否则，它将使用从`java.net.InetAddress.getCanonicalHostName()`返回的值。
>
> **类型**：`string` - **默认值**：`null` - **有效值**： - **重要性**：高 - **更新模式**：只读

> **`advertised.listeners`**：发布到`ZooKeeper`以便客户端使用的监听器（如果与`listenersconfi`g属性不同）。在`IaaS`环境中，这可能需要与代理绑定的接口不同。如果未设置，将使用`listeners`的值。`listeners`与之不同的是，广告`0.0.0.0`元地址无效。
>
> **类型**：`string` - **默认值**：`null` - **有效值**： - **重要性**：高 - **更新模式**：每个`broker`

> **`advertised.port`**：不推荐使用：仅在`advertised.listeners`或`listeners`未设置时使用。使用`advertised.listeners`代替。发布到`ZooKeeper`供客户端使用的端口。在`IaaS`环境中，这可能需要与代理绑定的端口不同。如果未设置，它将发布代理绑定到的相同端口。
>
> **类型**：`int` - **默认值**：`null` - **有效值**： - **重要性**：高 - **更新模式**：只读

> **`auto.create.topics.enable`**：启用服务器上`topic`的自动创建
>
> **类型**：`bool` - **默认值**：`true` - **有效值**： - **重要性**：高 - **更新模式**：只读

> **`auto.leader.rebalance.enable`**：启用自动领导者平衡。后台线程定期检查分区领导者的分布，可通过`leader.imbalance.check.interval.seconds`进行配置。如果领导者不平衡超过`leader.imbalance.per.broker.percentage`，则会导致领导者重新平衡到分区的首选领导者。
>
> **类型**：`bool` - **默认值**：`true` - **有效值**： - **重要性**：高 - **更新模式**：只读

> **`background.threads`**：用于各种后台处理任务的线程数
>
> **类型**：`int` - **默认值**：`10` - **有效值**：`[1，...]` - **重要性**：高 - **更新模式**：集群范围

> `broker.id`：此服务器的代理`ID`。如果未设置，将生成一个唯一的`broker``ID`。为避免`Zookeeper`生成的`broker``ID与`用户配置的`broker``ID`之间发生冲突，生成的`broker``ID`从`reserved.broker.max.id +1`开始。
类型：`int` - 默认值：`-1` - 有效值： - 重要性：高 - 更新模式：只读

> `compression.type`：指定给定`topic`的最终压缩类型。此配置接受标准压缩编解码器（`gzip`，`snappy`，`lz4`，`zstd`）。此外，它接受“未压缩”，等同于不压缩。和“生产者”，表示保留生产者设置的原始压缩编解码器。
类型：`string` - 默认值：生产者 - 有效值： - 重要性：高 - 更新模式：集群范围

> `control.plane.listener.name`：用于控制器和代理之间的通信的侦听器的名称。`Broker`将使用`control.plane.listener.name`在侦听器列表中定位端点，以侦听来自控制器的连接。例如，如果代理的配置为：`listeners = INTERNAL://192.1.1.8:9092，EXTERNAL://10.1.1.5：9093，CONTROLLER://192.1.1.8:9094` `listener.security.protocol.map = NTERNAL:PLAINTEXT，EXTERNAL:SSL，CONTROLLER:SSL``control.plane.listener.name = CONTROLLER`在启动时，代理将开始使用安全协议`SSL`侦听`192.1.1.8:9094`。在控制器端，当它通过`Zookeeper`发现代理发布的终结点时，它将使用`control.plane.listener.name`查找终结点，它将用于建立与代理的连接。例如，如果`broker`在`Zookeeper`上发布的端点是：`"endponints"：[" INTERNAL://broker1.example.com:9092","EXTERNAL://broker1.example.com:9093"，"CONTROLLER://broker1.example.com:9094"]`，控制器的配置为：`listener.security.protocol.map = INTERNAL:PLAINTEXT，EXTERNAL:SSL，CONTROLLER:SSL control.plane.listener.name = CONTROLLER`，然后控制器将使用"broker1.example.com:9094"使用安全协议"" SSL"连接到代理。如果未明确配置，则默认值将为`null`，并且将没有用于控制器连接的专用端点。`security.protocol.map = INTERNAL:PLAINTEXT，EXTERNAL:SSL，CONTROLLER:SSL control.plane.listener.name = CONTROLLER`，然后控制器将使用带有安全协议"SSL"的" broker1.example.com:9094"连接到代理。如果未明确配置，则默认值将为`null`，并且将没有用于控制器连接的专用端点。`security.protocol.map = INTERNAL:PLAINTEXT，EXTERNAL:SSL，CONTROLLER:SSL control.plane.listener.name = CONTROLLER`，然后控制器将使用带有安全协议"SSL"的"broker1.example.com:9094"连接到代理。如果未明确配置，则默认值将为`null`，并且将没有用于控制器连接的专用端点。
类型：`string` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：只读

> `delete.topic.enable`：启用删除`topic`。如果关闭此配置，则通过管理工具删除`topic`将无效。
类型：`bool` - 默认值：`true` - 有效值： - 重要性：高 - 更新模式：只读

> `host.name`：`listeners`不推荐使用：仅在未设置时使用。使用`listeners`代替。`broker`的主机名。如果设置此选项，它将仅绑定到该地址。如果未设置，它将绑定到所有接口
类型：`string` - 默认值："" - 有效值： - 重要性：高 - 更新模式：只读

> `leader.imbalance.check.interval.seconds`：控制器触发分区重新平衡检查的频率
类型：`long` - 默认值：300 - 有效值： - 重要性：高 - 更新模式：只读

> `leader.imbalance.per.broker.percentage`：每个`broker`允许的领导者不平衡比率。如果控制器超出每个`broker`的该值，则它将触发领导者余额。该值以百分比指定。
类型：`int` - 默认值：10 - 有效值： - 重要性：高 - 更新模式：只读

> `listeners`：侦听器列表-我们将在其上侦听的`URI`的逗号分隔列表和侦听器名称。如果侦听器名称不是安全协议，则还必须设置`listener.security.protocol.map`。将主机名指定为`0.0.0.0`以绑定到所有接口。将主机名保留为空以绑定到默认接口。合法侦听器列表的示例：`PLAINTEXT://myhost:9092，SSL://:9091 CLIENT://0.0.0.0:9092，REPLICATION://localhost:9093`
类型：`string` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：每个`broker`

> `log.dir`：保留日志数据的目录（`log.dirs`属性的补充）
类型：`string` - 默认值：`/tmp/kafka-logs` - 有效值： - 重要性：高 - 更新模式：只读

> `log.dirs`：保留日志数据的目录。如果未设置，则使用`log.dir`中的值
类型：`string` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：只读

> `log.flush.interval.messages`：将消息刷新到磁盘之前在日志分区上累积的消息数
类型：`long` - 默认值：9223372036854775807 - 有效值：`[1，...]` - 重要性：高 - 更新模式：集群范围

> `log.flush.interval.ms`：刷新到磁盘之前，任何`topic`中的消息在内存中保留的最长时间（以毫秒为单位）。如果未设置，则使用`log.flush.scheduler.interval.ms`中的值
类型：`long` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：集群范围

> `log.flush.offset.checkpoint.interval.ms`：我们更新上次刷新的持久记录的频率，该刷新充当日志恢复点
类型：`int` - 默认值：60000 - 有效值：`[0，...]` - 重要性：高 - 更新模式：只读

> `log.flush.scheduler.interval.ms`：日志刷新器检查是否需要将任何日志刷新到磁盘的频率（毫秒）
类型：`long` - 默认值：9223372036854775807 - 有效值： - 重要性：高 - 更新模式：只读

> `log.flush.start.offset.checkpoint.interval.ms`：更新日志开始偏移的持久记录的频率
类型：`int` - 默认值：60000 - 有效值：`[0，...]` - 重要性：高 - 更新模式：只读

> `log.retention.bytes`：删除日志之前的最大日志大小
类型：`long` - 默认值：`-1` - 有效值： - 重要性：高 - 更新模式：集群范围

> `log.retention.hours`：删除日志文件之前保留日志的小时数（以小时为单位），从`log.retention.ms`属性到第三级
类型：`int` - 默认值：`168` - 有效值： - 重要性：高 - 更新模式：只读

> `log.retention.minutes`：在删除日志文件之前保留日志文件的分钟数（以分钟为单位），仅次于`log.retention.ms`属性。如果未设置，则使用`log.retention.hours`中的值
类型：`int` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：只读

> `log.retention.ms` - ：删除日志文件之前保留日志文件的毫秒数（以毫秒为单位），如果未设置，则使用`log.retention.minutes`中的值。如果设置为`-1`，则不应用时间限制。
类型：`long` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：集群范围

> `log.roll.hours`：新日志段推出之前的最长时间（以小时为单位），属于`log.roll.ms`属性的第二时间
类型：`int` - 默认值：`168` - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `log.roll.jitter.hours`：从`logRollTimeMillis`中减去的最大抖动（以小时为单位），是`log.roll.jitter.ms`属性的第二个
类型：`int` - 默认值：`0` - 有效值：`[0，...]` - 重要性：高 - 更新模式：只读

> `log.roll.jitter.ms`：从`logRollTimeMillis`中减去的最大抖动（以毫秒为单位）。如果未设置，则使用`log.roll.jitter.hours`中的值
类型：`long` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：集群范围

> `log.roll.ms`：推出新的日志段之前的最长时间（以毫秒为单位）。如果未设置，则使用`log.roll.hours`中的值
类型：`long` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：集群范围

> `log.segment.bytes`：单个日志文件的最大大小
类型：`int` - 默认值：1073741824 - 有效值：[`14，...]` - 重要性：高 - 更新模式：集群范围

> `log.segment.delete.delay.ms`：从文件系统删除文件之前要等待的时间
类型：`long` - 默认值：60000 - 有效值：`[0，...]` - 重要性：高 - 更新模式：集群范围

> `message.max.bytes`：`Kafka`允许的最大记录批处理大小。如果增加了该数量，并且有一些使用方的年龄大于`0.10.2`，则还必须增加使用方的获取大小，以便他们可以获取如此大的记录批次。在最新的消息格式版本中，为了提高效率，始终将记录分组。在以前的消息格式版本中，未压缩的记录不会分组，并且在这种情况下，此限制仅适用于单个记录。可以使用主题级别`max.message.bytes`配置针对每个主题进行设置。
类型：`int` - 默认值：1000012 - 有效值：`[0，...]` - 重要性：高 - 更新模式：集群范围

> `min.insync.replicas`：当生产者将`acks`设置为`all`（或`-1`）时，`min.insync.replicas`指定必须确认写入才能使成功视为成功的最小副本数。如果无法满足此最小值，则生产者将引发异常（`NotEnoughReplicas`或`NotEnoughReplicasAfterAppend`）。
一起使用时，`min.insync.replicas`和`acks`可使您实施更大的耐用性保证。典型的情况是创建一个复制因子为3的主题，将`min.insync.replicas`设置为`2`，并产生"all"。如果大多数副本未收到写入，这将确保生产者引发异常。
类型：`int` - 默认值：1 - 有效值：`[1，...]` - 重要性：高 - 更新模式：集群范围

> `num.io.threads`：服务器用于处理请求的线程数，其中可能包括磁盘`I/O`
类型：`int` - 默认值：8 - 有效值：`[1，...]` - 重要性：高 - 更新模式：集群范围

> `num.network.threads`：服务器用于从网络接收请求并向网络发送响应的线程数
类型：`int` - 默认值：`3` - 有效值：`[1，...]` - 重要性：高 - 更新模式：集群范围

> `num.recovery.threads.per.data.dir`：每个数据目录在启动时用于日志恢复以及在关机时用于刷新的线程数
类型：`int` - 默认值：1 - 有效值：`[1，...]` - 重要性：高 - 更新模式：集群范围

> `num.replica.alter.log.dirs.threads`：可以在日志目录之间移动副本的线程数，其中可能包括磁盘`I/O`
类型：`int` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：只读

> `num.replica.fetchers`：用于复制来自源代理的消息的访存线程的数量。增大此值可以增加关注代理中的`I/O`并行度。
类型：`int` - 默认值：1 - 有效值： - 重要性：高 - 更新模式：集群范围

> `offset.metadata.max.bytes`：与偏移提交关联的元数据条目的最大大小
类型：`int` - 默认值：4096 - 有效值： - 重要性：高 - 更新模式：只读

> `offsets.commit.required.acks`：可以接受提交之前必需的确认。通常，不应覆盖默认值（`-1`）
类型：`short`  - 默认值：`-1` - 有效值： - 重要性：高 - 更新模式：只读

> `offsets.commit.timeout.ms`：偏移提交将被延迟，直到偏移`topic`的所有副本都收到提交或达到此超时为止。这类似于生产者请求超时。
类型：`int` - 默认值：5000 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.load.buffer.size`：将偏移量加载到缓存中时从偏移量段读取的批处理大小（软限制，如果记录太大，则覆盖）。
类型：`int` - 默认值：5242880 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.retention.check.interval.ms`：检查陈旧偏移的频率
类型：`long` - 默认值：600000 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.retention.minutes`：消费者组失去所有消费者（即变空）后，其偏移量将在此保留期内保留，然后丢弃。对于独立使用者（使用手动分配），偏移量将在上次提交时间加上此保留期后过期。
类型：`int` - 默认值：10080 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.topic.compression.codec`：用于偏移量`topic`的压缩编解码器-压缩可用于实现“原子”提交
类型：`int` - 默认值：0 - 有效值： - 重要性：高 - 更新模式：只读

> `offsets.topic.num.partitions`：偏移提交`topic`的分区数（部署后不应更改）
类型：`int` - 默认值：50 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.topic.replication.factor`：`offsets``topic`的复制因子（设置较高以确保可用性）。内部`topic`创建将失败，直到群集大小满足此复制因子要求为止。
类型：`short` - 默认值：3 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `offsets.topic.segment.bytes`：应该将`offsets``topic`段字节保持相对较小，以便于更快地压缩日志和缓存加载
类型：`int` - 默认值：104857600 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `port`：已弃用：仅在`listeners`未设置时使用。使用`listeners`代替。用于侦听和接受连接的端口
类型：`int` - 默认值：`9092` - 有效值： - 重要性：高 - 更新模式：只读

> `queued.max.requests`：在阻塞网络线程之前，数据平面允许的排队请求数
类型：`int` - 默认值：500 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `quota.consumer.default`：弃用：仅在未为以下项配置动态默认配额时使用 要么 在`Zookeeper`中。如果`clientId/consumer`组区分的任何使用者每秒获取的字节数超过此值，则将受到限制
类型：`long` - 默认值：9223372036854775807 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `quota.producer.default`：弃用：仅当未为以下项配置动态默认配额时使用， 要么 在`Zookeeper`中。如果由`clientId`区分的任何生产者每秒产生的字节数超过此值，则将受到限制
类型：`long` - 默认值：9223372036854775807 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `copy.fetch.min.bytes`：每个读取响应期望的最小字节。如果没有足够的字节，请等待直到`copyMaxMaxWaitTimeMs`
类型：`int` - 默认值：1 - 有效值： - 重要性：高 - 更新模式：只读

> `copy.fetch.wait.max.ms`：追随者副本发出的每个提取器请求的最大等待时间。此值应始终始终小于复制副本.`lag.time.max.ms`，以防止由于吞吐量低而导致ISR频繁缩小
类型：`int` - 默认值：500 - 有效值： - 重要性：高 - 更新模式：只读

> `copy.high.watermark.checkpoint.interval.ms`：高水印保存到磁盘的频率
类型：`long` - 默认值：5000 - 有效值： - 重要性：高 - 更新模式：只读

> `plicate.lag.time.max.ms`：如果至少在此时间之后某个跟随者未发送任何获取请求或未消耗直至领导者日志结束偏移，则领导者将从isr中删除跟随者
类型：`long` - 默认值：10000 - 有效值： - 重要性：高 - 更新模式：只读

> `copy.socket.receive.buffer.bytes`：套接字用于网络请求的接收缓冲区
类型：`int` - 默认值：65536 - 有效值： - 重要性：高 - 更新模式：只读

> `copy.socket.timeout.ms`：网络请求的套接字超时。它的值至少应为`copy.fetch.wait.max.ms`
类型：`int` - 默认值：30000 - 有效值： - 重要性：高 - 更新模式：只读

> `request.timeout.ms`：该配置控制客户端等待请求响应的最长时间。如果超时之前仍未收到响应，则客户端将在必要时重新发送请求，如果重试已用尽，则客户端将使请求失败。
类型：`int` - 默认值：30000 - 有效值： - 重要性：高 - 更新模式：只读

> `socket.receive.buffer.bytes`：套接字服务器套接字的`SO_RCVBUF`缓冲区。如果值为`-1`，则将使用操作系统默认值。
类型：`int` - 默认值：102400有 - 效值： - 重要性：高 - 更新模式：只读

> `socket.request.max.bytes`：套接字请求中的最大字节数
类型：`int` - 默认值：104857600 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `socket.send.buffer.bytes`：套接字服务器套接字的`SO_SNDBUF`缓冲区。如果值为`-1`，则将使用操作系统默认值。
类型：`int` - 默认值：102400 - 有效值： - 重要性：高 - 更新模式：只读

> `transaction.max.timeout.ms`：允许的最大事务超时。如果客户请求的交易时间超过此时间，则代理将在`InitProducerIdRequest`中返回错误。这样可以防止客户的超时过大，否则可能会使消费者无法阅读交易中包含的主题。
类型：`int` - 默认值：900000 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `transaction.state.log.load.buffer.size`：将生产者`ID`和事务加载到缓存中时从事务日志段读取的批处理大小（软限制，如果记录太大，则覆盖）。
类型：`int` - 默认值：5242880 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `transaction.state.log.min.isr`：覆盖交易主题的`min.insync.replicas`配置。
类型：`int` - 默认值：2 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `transaction.state.log.num.partitions`：交易`topic`的分区数（部署后不应更改）。
类型：`int` - 默认值：50 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `transaction.state.log.replication.factor`：事务`topic`的复制因子（设置为较高以确保可用性）。内部`topic`创建将失败，直到群集大小满足此复制因子要求为止。
类型：`short` - 默认值：3 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `transaction.state.log.segment.bytes`：事务主题段字节应保持相对较小，以便于更快地压缩日志和缓存加载
类型：`int` - 默认值：104857600 - 有效值：`[1，...]` - 重要性：高更新模式：只读

> `transactional.id.expiration.ms`：事务协调器将在其事务`ID`到期之前等待但未收到当前事务的任何事务状态更新的等待时间（以毫秒为单位）。此设置还会影响生产者`ID`的到期时间-在使用给定生产者`ID`进行最后一次写入之后，一旦经过此时间，生产者`ID`就会过期。请注意，由于主题的保留设置，如果删除生产者`ID`的最后一次写操作，生产者`ID`可能会更快过期。
类型：`int` - 默认值：604800000 - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `unclean.leader.election.enable`：指示是否允许将不在ISR集中的副本启用为万不得已的首长，即使这样做可能会导致数据丢失
类型：`bool` - 默认值：`false` - 有效值： - 重要性：高 - 更新模式：集群范围

> `zookeeper.connection.timeout.ms`：客户端等待与`Zookeeper`建立连接的最长时间。如果未设置，则使用`zookeeper.session.timeout.ms`中的值
类型：`int` - 默认值：`null` - 有效值： - 重要性：高 - 更新模式：只读

> `zookeeper.max.in.flight.requests`：客户端在阻止之前发送给`Zookeeper`的未确认请求的最大数量。
类型：`int` - 默认值：`10` - 有效值：`[1，...]` - 重要性：高 - 更新模式：只读

> `zookeeper.session.timeout.ms`：`Zookeeper`会话超时
类型：`int` - 默认值：6000 - 有效值： - 重要性：高 - 更新模式：只读

> `zookeeper.set.acl`：将客户端设置为使用安全`ACL`
类型：`bool` - 默认值：`false` - 有效值： - 重要性：高 - 更新模式：只读

> `broker.id.generation.enable`：在服务器上启用自动代理`ID`生成。启用后，应查看为`reserved.broker.max.id`配置的值。
类型：`bool` - 默认值：`true` - 有效值： - 重要性：中等 - 更新模式：只读

> `broker.rack`：`broker`的机架。这将用于机架感知复制分配中以实现容错功能。例如：`RACK1`，`us-east-1d`
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `connections.max.idle.ms`：空闲连接超时：服务器套接字处理器线程关闭空闲连接超过此数量的连接
类型：`long` - 默认值：600000 - 有效值： - 重要性：中等 - 更新模式：只读

> `connections.max.reauth.ms`：当显式设置为正数（默认为0，而不是正数）时，不超过配置值的会话生存期将在`v2.2.0`或更高版本的客户端进行身份验证时进行通信。代理将断开在会话生存期内未重新进行身份验证的任何此类连接，然后将该连接随后用于除重新身份验证以外的任何目的。配置名称可以选择以侦听器前缀和小写形式的SASL机制名称作为前缀。例如，`listener.name.sasl_ssl.oauthbearer.connections.max.reauth.ms = 3600000`
类型：`long` - 默认值：0 - 有效值：重要性：中等 - 更新模式：只读

> `control.shutdown.enable`：启用服务器的受控关闭
类型：`bool` - 默认值：`true` - 有效值： - 重要性：中等 - 更新模式：只读

> `control.shutdown.max.retries`：受控关闭可能由于多种原因而失败。这确定了发生此类故障时的重试次数
类型：`int` - 默认值：3 - 有效值： - 重要性：中等 - 更新模式：只读

> `control.shutdown.retry.backoff.ms`：在每次重试之前，系统需要时间从引起先前故障的状态中恢复（控制器故障转移，副本滞后等）。此配置确定重试之前要等待的时间。
类型：`long` - 默认值：5000 - 有效值： - 重要性：中等 - 更新模式：只读

> `controller.socket.timeout.ms`：控制器到`broker`通道的套接字超时
类型：`int` - 默认值：30000 - 有效值： - 重要性：中等 - 更新模式：只读

> `default.replication.factor`：自动创建的主题的默认复制因子
类型：`int` - 默认值：`1` - 有效值： - 重要性：中等 - 更新模式：只读

> `delegation.token.expiry.time.ms`：需要更新令牌之前的令牌有效时间（以毫秒为单位）。默认值1天。
类型：`long` - 默认值：86400000 - 有效值：`[1，...]` - 重要性：中等 - 更新模式：只读

> `delegation.token.master.key`：用于生成和验证委派令牌的主密钥/秘密密钥。必须在所有代理之间配置相同的密钥。如果未设置密钥或将密钥设置为空`string`，则代理将禁用委托令牌支持。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `delegation.token.max.lifetime.ms`：令牌具有最大生存期，超过该生存期将无法更新。默认值7天。
类型：`long` - 默认值：604800000 - 有效值：`[1，...]` - 重要性：中等 - 更新模式：只读

> `delete.records.purgatory.purge.interval.requests`：删除记录请求炼狱的清除间隔（以请求数为单位）
类型：`int` - 默认值：1 - 有效值： - 重要性：中等 - 更新模式：只读

> `fetch.purgatory.purge.interval.requests`：获取请求炼狱的清除间隔（以请求数为单位）
类型：`int` - 默认值：1000 - 有效值： - 重要性：中等 - 更新模式：只读

> `group.initial.rebalance.delay.ms`：组协调员在执行第一次重新平衡之前将等待更多消费者加入新组的时间。较长的延迟可能意味着较少的重新平衡，但会增加开始处理之前的时间。
类型：`int` - 默认值：`3000` - 有效值： - 重要性：中等 - 更新模式：只读

> `group.max.session.timeout.ms`：注册使用者的最大允许会话超时。较长的超时时间使消费者有更多的时间来处理心跳之间的消息，但要花费较长的时间来检测故障。
类型：`int` - 默认值：1800000 - 有效值： - 重要性：中等 - 更新模式：只读

> `group.max.size`：单个消费者组可以容纳的最大消费者数量。
类型：`int` - 默认值：2147483647 - 有效值：`[1，...]` - 重要性：中等 - 更新模式：只读

> `group.min.session.timeout.ms`：注册使用者的最小允许会话超时。较短的超时可以更快地进行故障检测，但需要付出更频繁的消费者心跳，这可能会使代理资源不堪重负。
类型：`int` - 默认值：6000 - 有效值： - 重要性：中等 - 更新模式：只读

> `inter.broker.listener.name`：用于代理之间进行通信的侦听器的名称。如果未设置，则侦听器名称由`security.inter.broker.protocol`定义。同时设置此属性和`security.inter.broker.protocol`属性是错误的。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `inter.broker.protocol.version`：指定将使用哪个版本的中间代理协议。在所有代理升级到新版本后，通常会遇到这种情况。一些有效值的示例是：`0.8.0、0.8.1、0.8.1.1、0.8.2、0.8.2.0、0.8.2.1、0.9.0.0、0.9.0.1`检查`ApiVersion`的完整列表。
类型：`string` - 默认值：2.4-IV1 - 有效值：`[0.8.0、0.8.1、0.8.2、0.9.0、0.10.0-IV0、0.10.0-IV1、0.10.1-IV0、0.10.1-IV1、0.10.1-IV2， 0.10.2-IV0、0.11.0-IV0、0.11.0-IV1、0.11.0-IV2、1.0-IV0、1.1-IV0、2.0-IV0、2.0-IV1、2.1-IV0、2.1-IV1、2.1- IV2、2.2-IV0、2.2-IV1、2.3-IV0、2.3-IV1、2.4-IV0、2.4-IV1]` - 重要性：中等 - 更新模式：只读

> `log.cleaner.backoff.ms`：没有日志可清除时的睡眠时间
类型：`long` - 默认值：15000 - 有效值：`[0，...]` - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.dedupe.buffer.size`：用于所有清理器线程中的日志重复数据删除的总内存
类型：`long` - 默认值：134217728 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.delete.retention.ms`：删除记录保留多长时间？
类型：`long` - 默认值：86400000 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.enable`：启用日志清除器进程以在服务器上运行。如果使用带有`cleanup.policy = compact`的任何`topic`（包括内部偏移量`topic`），应启用该选项。如果禁用，这些`topic`将不会被压缩，并且会不断增长。
类型：`bool` - 默认值：`true`有效值： - 重要性：中等 - 更新模式：只读

> `log.cleaner.io.buffer.load.factor`：日志清理程序重复数据删除缓冲区负载因子。重复数据删除缓冲区可以充满的百分比。较高的值将允许一次清除更多日志，但会导致更多哈希冲突
类型：`double` - 默认值：`0.9` - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.io.buffer.size`：用于所有清理器线程的日志清理器`I/O`缓冲区的总内存
类型：`int` - 默认值：524288 - 有效值：`[0，...]` - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.io.max.bytes.per.second`：将限制日志清除器，以使其读取和写入`I/O`的总和平均小于该值
类型：`double` - 默认值：1.7976931348623157E308 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.max.compaction.lag.ms`：消息将不符合压缩条件的最长时间。仅适用于正在压缩的日志。
类型：`long` - 默认值：9223372036854775807 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.min.cleanable.ratio`：符合清除条件的日志的脏日志与总日志的最小比率。如果还指定了`log.cleaner.max.compaction.lag.ms`或`log.cleaner.min.compaction.lag.ms`配置，则日志压缩器将在以下任一情况下立即认为该日志符合压缩条件：（i）已达到脏率阈值，并且日志至少在`log.cleaner.min.compaction.lag.ms`持续时间内具有脏（未压缩）记录，或者（ii）日志最多具有脏（未压缩）记录`log.cleaner.max.compaction.lag.ms`周期。
类型：`double` - 默认值：0.5 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.min.compaction.lag.ms`：消息在日志中保持未压缩的最短时间。仅适用于正在压缩的日志。
类型：`long` - 默认值：0 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.cleaner.threads`：用于日志清理的后台线程数
类型：`int` - 默认值：1 - 有效值：`[0，...]` - 重要性：中等 - 更新模式：集群范围

> `log.cleanup.policy`：保留窗口之外的段的默认清理策略。以逗号分隔的有效策略列表。有效策略为：“删除”和“紧凑”
类型：`list` - 默认值：`delete` - 有效值：[`compact`，`delete`] - 重要性：中等 - 更新模式：集群范围

> `log.index.interval.bytes`：我们向偏移量索引添加条目的间隔
类型：`int` - 默认值：4096 - 有效值：`[0，...]` - 重要性：中等 - 更新模式：集群范围

> `log.index.size.max.bytes`：偏移索引的最大大小（以字节为单位）
类型：`int` - 默认值：10485760 - 有效值：`[4，...]` - 重要性：中等 - 更新模式：集群范围

> `log.message.format.version`：指定代理用来将消息追加到日志的消息格式版本。该值应为有效的`ApiVersion`。一些示例是：`0.8.2、0.9.0.0、0.10.0`，请检查`ApiVersion`以获取更多详细信息。通过设置特定的消息格式版本，用户可以证明磁盘上的所有现有消息均小于或等于指定的版本。错误地设置此值将导致使用较旧版本的使用者中断，因为他们将收到他们不理解的格式的消息。
类型：`string` - 默认值：2.4-IV1 - 有效值：`[0.8.0、0.8.1、0.8.2、0.9.0、0.10.0-IV0、0.10.0-IV1、0.10.1-IV0、0.10.1-IV1、0.10.1-IV2， 0.10.2-IV0、0.11.0-IV0、0.11.0-IV1、0.11.0-IV2、1.0-IV0、1.1-IV0、2.0-IV0、2.0-IV1、2.1-IV0、2.1-IV1、2.1- IV2、2.2-IV0、2.2-IV1、2.3-IV0、2.3-IV1、2.4-IV0、2.4-IV1]` - 重要性：中等 - 更新模式：只读

> `log.message.timestamp.difference.max.ms`：代理接收消息时的时间戳与消息中指定的时间戳之间允许的最大差异。如果`log.message.timestamp.type = CreateTime`，则时间戳差异超过此阈值时，将拒绝一条消息。如果l`og.message.timestamp.type = LogAppendTime`，则忽略此配置。所允许的最大时间戳差应不大于`log.retention.ms`，以避免不必要的频繁滚动日志。
类型：`long` - 默认值：9223372036854775807 - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.message.timestamp.type`：定义消息中的时间戳是消息创建时间还是日志附加时间。该值应该是`CreateTime`或`LogAppendTime`
类型：`string` - 默认值：`CreateTime` - 有效值：[`CreateTime`，`LogAppendTime`] - 重要性：中等 - 更新模式：集群范围

> `log.preallocate`：创建新段时是否应该预分配文件？如果您在`Windows`上使用`Kafka`，则可能需要将其设置为`true`。
类型：`bool` - 默认值：`false` - 有效值： - 重要性：中等 - 更新模式：集群范围

> `log.retention.check.interval.ms`：日志清除器检查是否有任何日志可删除的频率（以毫秒为单位）
类型：`long` - 默认值：300000 - 有效值：[1，...] - 重要性：中等 - 更新模式：只读

> `max.connections`：我们随时允许在代理中允许的最大连接数。除了使用`max.connections.per.ip`配置的每个`IP`限制之外，还应用此限制。侦听器级别的限制也可以通过在配置名称前添加侦听器前缀来配置`listener.name.internal.max.connections`。应基于代理容量配置代理范围的限制，而应基于应用程序要求配置侦听器的限制。如果达到侦听器或代理限制，则新连接被阻止。即使已达到代理范围的限制，也允许在代理中间侦听器上进行连接。在这种情况下，将关闭另一个监听器上最近最少使用的连接。
类型：`int` - 默认值：2147483647 - 有效值：[0，...] - 重要性：中等 - 更新模式：集群范围

> `max.connections.per.ip`：每个`IP`地址允许的最大连接数。如果使用`max.connections.per.ip.overrides`属性配置了替代，则可以将其设置为0。如果达到限制，则将丢弃来自`IP`地址的新连接。
类型：`int` - 默认值：2147483647 - 有效值：[0，...] - 重要性：中等 - 更新模式：集群范围

> `max.connections.per.ip.overrides`：每个`IP`或主机名的逗号分隔列表将覆盖默认的最大连接数。一个示例值是"hostName:100,127.0.0.1:200"
类型：`string` - 默认值："" - 有效值： - 重要性：中等 - 更新模式：集群范围

> `max.incremental.fetch.session.cache.slots`：我们将维护的最大增量获取会话数。
类型：`int` - 默认值：1000 - 有效值：[0，...] - 重要性：中等 - 更新模式：只读

> `num.partitions`：每个`topic`的默认日志分区数
类型：`int` - 默认值：1 - 有效值：[1，...] - 重要性：中等 - 更新模式：只读

> `password.encoder.old.secret`：用于对动态配置的密码进行编码的旧机密。仅在更新机密时才需要。如果指定，则在代理启动时，将使用此旧机密对所有动态编码的密码进行解码，并使用`password.encoder.secret`重新编码。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `password.encoder.secret`：用于对此代理程序动态配置的密码进行编码的机密。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `principal.builder.class`：实现`KafkaPrincipalBuilder`接口的类的全限定名，该接口用于构建授权期间使用的`KafkaPrincipal`对象。此配置还支持不推荐使用的`PrincipalBuilder`接口，该接口以前用于通过`SSL`进行客户端身份验证。如果未定义任何主体构建器，则默认行为取决于所使用的安全协议。对于`SSL`身份验证，将使用ssl.principal.mapping.rules从客户端证书（如果提供的话）对可分辨名称应用的规则定义的主体派生；否则，如果不需要客户端身份验证，则主体名称将为匿名。对于`SASL`身份验证，将使用以下规则定义的主体派生主体`sasl.kerberos.principal.to.local.rules`是否正在使用`GSSAPI`，以及其他机制的`SASL`身份验证`ID`。对于`PLAINTEXT`，主体将是匿名的。
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `producer.purgatory.purge.interval.requests`：生产者请求炼狱的清除间隔（请求数）
类型：`int` - 默认值：1000 - 有效值： - 重要性：中等 - 更新模式：只读

> `queued.max.request.bytes`：在不再读取请求之前允许的排队字节数
类型：`long` - 默认值：-1 - 有效值： - 重要性：中等 - 更新模式：只读

> `copy.fetch.backoff.ms`：发生获取分区错误时的睡眠时间。
类型：`int` - 默认值：`1000` - 有效值：[0，...] - 重要性：中等 - 更新模式：只读

> `copy.fetch.max.bytes`：尝试为每个分区获取的消息的字节数。这不是绝对最大值，如果提取的第一个非空分区中的第一个记录批处理大于此值，那么仍将返回记录批处理以确保可以进行。代理接受的最大记录批处理大小是通过`message.max.bytes`（代理配置）或`max.message.bytes`（`topic`配置）定义的。
类型：`int` - 默认值：1048576 - 有效值：[0，...] - 重要性：中等 - 更新模式：只读

> `copy.fetch.response.max.bytes`：整个读取响应预期的最大字节数。记录是分批提取的，并且如果所提取的第一个非空分区中的第一个记录批处理大于此值，那么仍将返回记录批处理以确保可以进行。因此，这不是绝对最大值。代理接受的最大记录批处理大小是通过`message.max.bytes`（代理配置）或`max.message.bytes`（主题配置）定义的。
类型：`int` - 默认值：10485760 - 有效值：[0，...] - 重要性：中等 - 更新模式：只读

> `copy.selector.class`：实现`ReplicaSelector`的完全限定的类名。代理使用它来查找首选的只读副本。默认情况下，我们使用返回领导者的实现。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `reserved.broker.max.id`：可用于`broker.id`的最大数字
类型：`int` - 默认值：1000 - 有效值：[0，...] - 重要性：中等 - 更新模式：只读

> `sasl.client.callback.handler.class`：实现`AuthenticateCallbackHandler`接口的`SASL`客户端回调处理程序类的完全限定名称。
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `sasl.enabled.mechanisms`：`Kafka`服务器中启用的`SASL`机制的列表。该列表可以包含安全提供程序可用的任何机制。默认情况下，仅启用`GSSAPI`。
类型：`liast` - 默认值：`GSSAPI` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.jaas.config`：`SASL连`接的`JAAS`登录上下文参数，采用`JAAS`配置文件使用的格式。这里描述了`JAAS`配置文件格式。值的格式为：`loginModuleClass controlFlag (optionName=optionValue)*;`。对于代理，配置必须以侦听器前缀和小写的`SASL`机制名称作为前缀。例如，需要`listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config = com.example.ScramLoginModule`；
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.kinit.cmd`：`Kerberos kinit`命令路径。
类型：`string` - 默认值：`/usr/bin/kinit` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.min.time.before.relogin`：两次刷新尝试之间的登录线程睡眠时间。
类型：`long` - 默认值：60000 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.principal.to.local.rules`：从主体名称到简称（通常是操作系统用户名）的映射规则列表。将按顺序评估规则，并且使用与主体名称匹配的第一条规则将其映射为简称。列表中以后的所有规则都将被忽略。默认情况下，格式为`{username}/{hostname}@{REALM}`的主体名称映射到`{username}`。有关格式的更多详细信息，请参见安全授权和`acls`。请注意，如果`principal.builder.class`配置提供了`KafkaPrincipalBuilder`的扩展名，则将忽略此配置。
类型：`list` - 默认值：`default` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.service.name`：`Kafka`运行时使用的`Kerberos`主体名称。这可以在`Kafka`的`JAAS`配置或`Kafka`的配置中定义。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.ticket.renew.jitter`：添加到续订时间的随机抖动百分比。
类型：`double` - 默认值：0.05 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.kerberos.ticket.renew.window.factor`：登录线程将一直休眠，直到达到从上次刷新到票证到期的指定时间窗口因子为止，此时它将尝试续订票证。
类型：`double` - 默认值：0.8 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.login.callback.handler.class`：实现`AuthenticateCallbackHandler`接口的`SASL`登录回调处理程序类的全限定名。对于代理，登录回调处理程序配置必须以侦听器前缀和小写的SASL机制名称作为前缀。例如，`listener.name.sasl_ssl.scram-sha-256.sasl.login.callback.handler.class = com.example.CustomScramLoginCallbackHandler`
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `sasl.login.class`：实现`Login`接口的类的完全限定名称。对于代理，登录配置必须以侦听器前缀和小写的`SASL`机制名称作为前缀。例如，`listener.name.sasl_ssl.scram-sha-256.sasl.login.class = com.example.CustomScramLogin`
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `sasl.login.refresh.buffer.seconds`：刷新凭证时要保留的凭证过期前的缓冲区时间，以秒为单位。如果刷新将以比缓冲区秒数更接近到期的方式发生，则刷新将被上移以保持尽可能多的缓冲区时间。合法值介于0到3600（1小时）之间；如果未指定任何值，则使用默认值300（5分钟）。如果此值和`sasl.login.refresh.min.period.seconds`的总和超过凭据的剩余生存期，则两者都将被忽略。当前仅适用于`OAUTHBEARER`。
类型：`short` - 默认值：300 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.login.refresh.min.period.seconds`：登录刷新线程在刷新凭证之前等待的最短时间（以秒为单位）。合法值在0到900之间（15分钟）；如果未指定任何值，则使用默认值60（1分钟）。如果此值和`sasl.login.refresh.buffer.seconds`的总和超过凭据的剩余生存期，则两者都将被忽略。当前仅适用于`OAUTHBEARER`。
类型：`short` - 默认值：60 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.login.refresh.window.factor`：登录刷新线程将休眠，直到达到相对于凭据生存期的指定窗口因子为止，此时它将尝试刷新凭据。合法值介于0.5（50％）至1.0（100％）之间；如果未指定任何值，则使用默认值0.8（80％）。当前仅适用于`OAUTHBEARER`。
类型：`double` - 默认值：0.8 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.login.refresh.window.jitter`：相对于凭证生存期的最大随机抖动量，添加到登录刷新线程的睡眠时间中。合法值介于0到0.25（25％）之间（含）；如果未指定任何值，则使用默认值0.05（5％）。当前仅适用于`OAUTHBEARER`。
类型：`double` - 默认值：0.05 - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.mechanism.inter.broker.protocol`：用于`broker`之间通信的`SASL`机制。默认值为`GSSAPI`。
类型：`string` - 默认值：`GSSAPI` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `sasl.server.callback.handler.class`：实现`AuthenticateCallbackHandler`接口的`SASL`服务器回调处理程序类的全限定名。服务器回调处理程序必须使用小写的侦听器前缀和`SASL`机制名称作为前缀。例如，`listener.name.sasl_ssl.plain.sasl.server.callback.handler.class = com.example.CustomPlainCallbackHandler`。
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：只读

> `security.inter.broker.protocol`：用于在代理之间进行通信的安全协议。有效值为：`PLAINTEXT`，`SSL`，`SASL_PLAINTEXT`，`SASL_SSL`。同时设置此属性和`inter.broker.listener.name`属性是错误的。
类型：`string` - 默认值：`PLAINTEXT` - 有效值： - 重要性：中等 - 更新模式：只读

> `ssl.cipher.suites`：密码套件列表。这是认证，加密，MAC和密钥交换算法的命名组合，用于协商使用TLS或SSL网络协议的网络连接的安全设置。默认情况下，支持所有可用的密码套件。
类型：`list` - 默认值："" - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.client.auth`：配置`kafka`代理以请求客户端身份验证。共有以下设置：
- `ssl.client.auth=required` 如果设置为必需，则需要客户机认证。
- `ssl.client.auth=requested`这意味着客户端身份验证是可选的。与请求不同，如果设置了此选项，则客户端可以选择不提供有关其自身的身份验证信息
- `ssl.client.auth=none` 这意味着不需要客户端身份验证。

类型：`string` - 默认值：`none` - 有效值：[`required`，`requested`，`none`] - 重要性：中等 - 更新模式：每个`broker`

> `ssl.enabled.protocols`：为`SSL`连接启用的协议列表。
类型：`list` - 默认值：`TLSv1.2，TLSv1.1，TLSv1` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.key.password`：密钥存储文件中私钥的密码。这对于客户端是可选的。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.keymanager.algorithm`：密钥管理器工厂用于`SSL`连接的算法。缺省值是为`Java`虚拟机配置的密钥管理器工厂算法。
类型：`string` - 默认值：`SunX509` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.keystore.location`：密钥存储文件的位置。这对于客户端是可选的，并且可以用于客户端的双向身份验证。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.keystore.password`：密钥存储文件的存储密码。这对于客户端是可选的，并且仅在配置了`ssl.keystore.location`时才需要。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.keystore.type`：密钥存储文件的文件格式。这对于客户端是可选的。
类型：`string` - 默认值：`JKS` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.protocol`：用于生成`SSLContext`的`SSL`协议。默认设置为`TLS`，在大多数情况下都可以使用。最近的`JVM`中允许的值为`TLS`，`TLSv1.1`和`TLSv1.2`。较早的`JVM`中可能支持`SSL`，`SSLv2`和`SSLv3`，但由于已知的安全漏洞，因此不鼓励使用它们。
类型：`string` - 默认值：`TLS` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.provider`：用于`SSL`连接的安全提供程序的名称。缺省值是`JVM`的缺省安全提供程序。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.trustmanager.algorithm`：信任管理器工厂用于`SS`L连接的算法。默认值是为`Java`虚拟机配置的信任管理器工厂算法。
类型：`string` - 默认值：`PKIX` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.truststore.location`：信任库文件的位置。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.truststore.password`：信任存储文件的密码。如果未设置密码，对信任库的访问仍然可用，但是完整性检查被禁用。
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `ssl.truststore.type`：信任库文件的文件格式。
类型：`string` - 默认值：`JKS` - 有效值： - 重要性：中等 - 更新模式：每个`broker`

> `alter.config.policy.class.name`：应该用于验证的`alter configs`策略类。该类应实现`org.apache.kafka.server.policy.AlterConfigPolicy`接口。
类型：`class` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：只读

> `alter.log.dirs.replication.quota.window.num`：为更改日志目录复制配额而保留在内存中的样本数
类型：`int` - 默认值：11 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `alter.log.dirs.replication.quota.window.size.seconds`：更改日志目录复制配额的每个样本的时间跨度
类型：`int` - 默认值：1 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

`authorizer.class.name`：实现`sorg.apache.kafka.server.authorizer.Authorizer`接口的类的完全限定名称，该接口由代理用于授权。此配置还支持授权者，这些授权者实现了过时的`kafka.security.auth.Authorizer`特性，该特性以前用于授权。
类型：`string` - 默认值："" - 有效值： - 重要性：低 - 更新模式：只读

> `client.quota.callback.class`：实现`ClientQuotaCallback`接口的类的完全限定名称，该接口用于确定应用于客户端请求的配额限制。默认情况下，， 要么 应用存储在`ZooKeeper`中的配额。对于任何给定的请求，将应用与会话的用户主体和请求的客户端ID相匹配的最具体的配额。
类型：`class` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：只读

> `connection.failed.authentication.delay.ms`：身份验证失败时连接关闭延迟：这是在身份验证失败时延迟连接关闭的时间（以毫秒为单位）。必须将其配置为小于`connections.max.idle.ms`，以防止连接超时。
类型：`int` - 默认值：`100` - 有效值：[0，...] - 重要性：低 - 更新模式：只读

> `create.topic.policy.class.name`：用于验证的创建主题策略类。该类应实现`org.apache.kafka.server.policy.CreateTopicPolicy`接口。
类型：`class` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：只读

> `registration.token.expiry.check.interval.ms`：扫描间隔以删除过期的委托令牌。
类型：`long` - 默认值：3600000 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `kafka.metrics.polling.interval.secs`：度量轮询间隔（以秒为单位），可在`kafka.metrics.reporters`实现中使用。
类型：`int` - 默认值：10 - 有效值：[1，...] - 重要性：低 - 更新模式：只读
> `kafka.metrics.reporters`：用作`Yammer`指标定制报告程序的类的列表。记者要实行`kafka.metrics.KafkaMetricsReporter`特质。如果客户端希望在自定义报告程序上公开`JMX`操作，则自定义报告程序需要另外实现一个扩展了`kafka.metrics.KafkaMetricsReporterMBean`特征的`MBean`特征，以使注册的`MBean`符合标准`MBean`约定。
类型：`list` - 默认值："" - 有效值： - 重要性：低 - 更新模式：只读

> `listener.security.protocol.map`：侦听器名称和安全协议之间的映射。必须对同一安全协议进行定义，才能在多个端口或IP中使用同一安全协议。例如，即使两者都需要SSL，也可以将内部和外部流量分开。具体来说，用户可以使用名称`INTERNAL`和`EXTERNAL`定义侦听器，并将此属性定义为：`INTERNAL:SSL`，`EXTERNAL:SSL`。如图所示，键和值之间用冒号分隔，而地图项则用逗号分隔。每个侦听器名称在地图中只能出现一次。通过在配置名称中添加标准化的前缀（侦听器名称为小写），可以为每个侦听器配置不同的安全性（`SSL`和`SASL`）设置。例如，要为内部侦听器设置其他密钥库，请使用名称为`listener.name.internal.ssl.keystore.location`将被设置。如果未设置侦听器名称的配置，则该配置将回退到通用配置（即`ssl.keystore.location`）。
类型：`string` - 默认值：`PLAINTEXT:PLAINTEXT，SSL:SSL，SASL_PLAINTEXT:SASL_PLAINTEXT，SASL_SSL:SASL_SSL` - 有效值： - 重要性：低 - 更新模式：每个`broker`

> `log.message.downconversion.enable`：此配置控制是否启用消息格式的下转换以满足消费请求。设置false为时，对于希望使用较旧消息格式的使用者，代理将不执行降频转换。`UNSUPPORTED_VERSION`对于来自此类较旧客户端的消费请求，代理会以错误响应。此配置不适用于复制到关注者所需的任何消息格式转换。
类型：`bool` - 默认值：`true` - 有效值： - 重要性：低 - 更新模式：集群范围

> `metric.reporters`：用作指标报告者的类列表。实施该`org.apache.kafka.common.metrics.MetricsReporter`接口允许插入将通知新度量标准创建的类。始终包含`JmxReporter`来注册`JMX`统计信息。
类型：`list` - 默认值："" - 有效值： - 重要性：低 - 更新模式：集群范围

> `metrics.num.samples`：为计算指标而维护的样本数。
类型：`int` - 默认值：2 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `metrics.recording.level`：指标的最高记录级别。
类型：`string` - 默认值：`INFO` - 有效值： - 重要性：低 - 更新模式：只读

> `metrics.sample.window.ms`：度量样本被计算的时间窗口。
类型：`long` - 默认值：30000 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `password.encoder.cipher.algorithm`：用于对动态配置的密码进行编码的密码算法。
类型：`string` - 默认值：`AES/CBC/PKCS5Padding` - 有效值： - 重要性：低 - 更新模式：只读

> `password.encoder.iterations`：用于对动态配置的密码进行编码的迭代计数。
类型：`int` - 默认值：4096 - 有效值：[1024，...] - 重要性：低 - 更新模式：只读

> `password.encoder.key.length`：用于对动态配置的密码进行编码的密钥长度。
类型：`int` - 默认值：128 - 有效值：[8，...] - 重要性：低 - 更新模式：只读

> `password.encoder.keyfactory.algorithm`：用于对动态配置的密码进行编码的`SecretKeyFactory`算法。如果可用，默认值为`PBKDF2WithHmacSHA512`，否则为`PBKDF2WithHmacSHA1`。
类型：`string` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：只读

> `quota.window.num`：为客户端配额保留在内存中的样本数
类型：`int` - 默认值：11 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `quota.window.size.seconds`：每个客户端配额示例的时间跨度
类型：`int` - 默认值：1 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `replication.quota.window.num`：要保留在内存中用于复制配额的样本数
类型：`int` - 默认值：11 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `replication.quota.window.size.seconds`：复制配额的每个样本的时间跨度
类型：`int` - 默认值：1 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `security.providers`：可配置创建者类的列表，每个创建者类返回一个实现安全算法的提供者。这些类应实现`org.apache.kafka.common.security.auth.SecurityProviderCreator`接口。
类型：`string` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：只读

> `ssl.endpoint.identification.algorithm`：使用服务器证书验证服务器主机名的端点标识算法。
类型：`string` - 默认值：`https` - 有效值： - 重要性：低 - 更新模式：每个`broker`

> `ssl.principal.mapping.rules`：从客户端证书的专有名称到简称的映射规则列表。将按顺序评估规则，并且使用与主体名称匹配的第一条规则将其映射为简称。列表中以后的所有规则都将被忽略。默认情况下，X.500证书的专有名称将是主体。有关格式的更多详细信息，请参见安全授权和acls。请注意，如果`principal.builder.class`配置提供了`KafkaPrincipalBuilder`的扩展名，则将忽略此配置。
类型：`string` - 默认值：`default` - 有效值： - 重要性：低 - 更新模式：只读

> `ssl.secure.random.implementation`：用于`SSL`加密操作的`SecureRandom PRNG`实现。
类型：`string` - 默认值：`null` - 有效值： - 重要性：低 - 更新模式：每个`broker`

> `transaction.abort.timed.out.transaction.cleanup.interval.ms`：回滚已超时的事务的时间间隔
类型：`int` - 默认值：60000 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `transaction.remove.expired.transaction.cleanup.interval.ms`：删除由于`transactional.id.expiration.ms`传递而过期的事务的时间间隔
类型：`int` - 默认值：3600000 - 有效值：[1，...] - 重要性：低 - 更新模式：只读

> `zookeeper.sync.time.ms`：`ZK`追随者可以落后`ZK`领导者多远
类型：`int` - 默认值：`2000` - 有效值： - 重要性：低 - 更新模式：只读

可在`scala`类中找到有关代理配置的更多详细信息`kafka.server.KafkaConfig`。


## 3.1.1 更新`broker`配置

从`Kafka 1.1`版开始，可以在不重新启动`broker`的情况下更新某些`broker`配置。见`Dynamic Update Mode`列经纪人CONFIGS每个经纪人配置的更新模式。
- `read-only`：需要重新启动代理以进行更新
- `per-broker`：可以为每个`broker`动态更新
- `cluster-wide`：可以动态更新为群集范围的默认值。也可以作为每个`broker`的值进行更新以进行测试。

要更改代理`ID` 0（例如，日志清除器线程数）的当前代理配置：
```bash
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --alter --add-config log.cleaner.threads=2
```
要描述代理`ID` 0的当前动态代理配置：
```bash
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --describe
```
要删除配置替代并恢复为代理`ID` 0（例如，日志清理器线程数）的静态配置或默认值：
```bash
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 0 --alter --delete-config log.cleaner.threads
```
某些配置可以配置为群集范围的默认值，以在整个群集中保持一致的值。群集中的所有代理将处理群集默认更新。例如，要更新所有代理上的日志清理器线程：
```bash
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-default --alter --add-config log.cleaner.threads=2
```
要描述当前配置的动态集群范围内的默认配置：
```bash
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-default --describe
```
可以在群集级别配置的所有配置也可以在每个代理级别配置（例如，用于测试）。如果在不同级别定义了配置值，则使用以下优先级顺序：
- 存储在`ZooKeeper`中的动态每个`broker`配置
- 存储在`ZooKeeper`中的动态群集范围内的默认配置
- 来自的静态代理配置`server.properties`
- `Kafka`默认，请参阅代理配置

### 动态更新密码配置
动态更新的密码配置值在存储在`ZooKeeper`中之前已加密。`password.encoder.secret`必须将`Broker`配置 配置`server.properties`为启用动态更新密码配置。不同经纪人的秘密可能有所不同。

可以通过滚动重新启动代理来轮换用于密码编码的机密。当前在`ZooKeeper`中用于编码密码的旧机密必须在静态代理配置中提供`password.encoder.old.secret`，而新机密必须在中提供`password.encoder.secret`。当代理启动时，存储在`ZooKeeper`中的所有动态密码配置都将使用新密码重新编码。

在`Kafka 1.1.x`中，使用进行更新配置时，`kafka-configs.sh`即使未更改密码配置，也必须在每个更改请求中提供所有动态更新的密码配置。此约束将在以后的版本中删除。

### 在启动代理之前更新ZooKeeper中的密码配置
从`Kafka 2.0.0`起，`kafka-configs.sh`可以在启动代理进行引导之前使用`ZooKeeper`更新动态代理配置。这样可以将所有密码配置以加密形式存储，而无需在中清除密码`server.properties`。`password.encoder.secret`如果`alter`命令中包含任何密码配置，则还必须指定代理配置。还可以指定其他加密参数。密码编码器配置将不会保留在`ZooKeeper`中。例如，要将侦听器的`SSL`密钥密码存储`INTERNAL` 在代理0上：
```bash
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type brokers --entity-name 0 --alter --add-config
  'listener.name.internal.ssl.key.password=key-password,password.encoder.secret=secret,password.encoder.iterations=8192'
```
`listener.name.internal.ssl.key.password`使用提供的编码器配置 ，配置将以加密形式保存在`ZooKeeper`中。编码器机密和迭代不会保留在`ZooKeeper`中。

### 更新现有侦听器的SSL密钥库
可以为代理配置具有短有效期的`SSL`密钥库，以减少证书被泄露的风险。密钥库可以动态更新，而无需重新启动代理。配置名称必须以侦听器前缀为前缀， `listener.name.{listenerName}`.以便仅更新特定侦听器的密钥库配置。以下配置可以在每个代理级别的单个更改请求中更新：
- `ssl.keystore.type`
- `ssl.keystore.location`
- `ssl.keystore.password`
- `ssl.key.password`

如果侦听器是代理中间侦听器，则仅当新的密钥库被为此侦听器配置的信任库信任时，才允许更新。对于其他侦听器，代理不会对密钥库执行信任验证。证书必须由签署旧证书的同一证书颁发机构签名，以避免任何客户端身份验证失败。

### 更新现有侦听器的SSL信任库
代理信任库可以动态更新，而无需重新启动代理以添加或删除证书。更新的信任库将用于验证新的客户端连接。配置名称必须以侦听器前缀为前缀，`listener.name.{listenerName}`.以便仅更新特定侦听器的信任库配置。以下配置可以在每个代理级别的单个更改请求中更新：
- `ssl.truststore.type`
- `ssl.truststore.location`
- `ssl.truststore.password`

如果侦听器是代理中间侦听器，则仅当新的信任库信任该侦听器的现有密钥库时，才允许更新。对于其他侦听器，代理在更新之前不会执行信任验证。从新的信任库中删除用于签署客户端证书的CA证书可能会导致客户端身份验证失败。

### 更新默认主题配置
代理使用的默认主题配置选项可能会更新，而无需重新启动代理。该配置适用于主题，而没有按主题对等配置的主题配置覆盖。这些配置中的一个或多个可以在所有代理使用的群集默认级别被覆盖。
- `log.segment.bytes`
- `log.roll.ms`
- `log.roll.hours`
- `log.roll.jitter.ms`
- `log.roll.jitter.hours`
- `log.index.size.max.bytes`
- `log.flush.interval.messages`
- `log.flush.interval.ms`
- `log.retention.bytes`
- `log.retention.ms`
- `log.retention.minutes`
- `log.retention.hours`
- `log.index.interval.bytes`
- `log.cleaner.delete.retention.ms`
- `log.cleaner.min.compaction.lag.ms`
- `log.cleaner.max.compaction.lag.ms`
- `log.cleaner.min.cleanable.ratio`
- `log.cleanup.policy`
- `log.segment.delete.delay.ms`
- `unclean.leader.election.enable`
- `min.insync.replicas`
- `max.message.bytes`
- `compression.type`
- `log.preallocate`
- `log.message.timestamp.type`
- `log.message.timestamp.difference.max.ms`

从`Kafka 2.0.0`版开始，`unclean.leader.election.enable`动态更新配置时，控制器会自动启用不干净的领导者选举 。在`Kafka 1.1.x`版中，更改`unclean.leader.election.enable`仅在选择新控制器后才生效。可以通过运行以下命令来强制重新选择控制器：
```bash
bin/zookeeper-shell.sh localhost
rmr /controller
```

### 更新日志清理器配置
日志清除器配置可以在所有代理使用的群集默认级别上动态更新。这些更改将在日志清除的下一次迭代中生效。这些配置中的一个或多个可以更新：
- `log.cleaner.threads`
- `log.cleaner.io.max.bytes.per.second`
- `log.cleaner.dedupe.buffer.size`
- `log.cleaner.io.buffer.size`
- `log.cleaner.io.buffer.load.factor`
- `log.cleaner.backoff.ms`

### 更新线程配置
代理使用的各种线程池的大小可以在所有代理使用的群集默认级别上动态更新。更新被限制在一定范围内`currentSize / 2`，`currentSize * 2`以确保配置更新得到妥善处理。
- `num.network.threads`
- `num.io.threads`
- `num.replica.fetchers`
- `num.recovery.threads.per.data.dir`
- `log.cleaner.threads`
- `background.threads`

### 更新ConnectionQuota配置
代理为给定IP/主机允许的最大连接数可以在所有代理使用的群集默认级别上动态更新。所做的更改将应用​​于创建新的连接，新的限制将考虑现有的连接数。
- `max.connections.per.ip`
- `max.connections.per.ip.overrides`

### 添加和删​​除侦听器
侦听器可以动态添加或删除。添加新的侦听器时，必须将侦听器的安全性配置提供为带有侦听器前缀的侦听器配置`listener.name.{listenerName}`.。如果新的侦听器使用SASL，则必须使用`sasl.jaas.config `带有侦听器和机制前缀的JAAS配置属性来提供侦听器的JAAS配置。有关详细信息，请参见Kafka代理的JAAS配置。

在Kafka版本1.1.x中，中间经纪人侦听器使用的侦听器可能不会动态更新。要将代理中间的侦听器更新为新的侦听器，可以在所有代理上添加新的侦听器，而无需重新启动代理。然后需要滚动重启才能更新`inter.broker.listener.name`。

除了新侦听器的所有安全性配置之外，以下配置可以在每个代理级别动态更新：
- `listeners`
- `advertised.listeners`
- `listener.security.protocol.map`

代理间侦听器必须使用静态代理配置`inter.broker.listener.name` 或配置`inter.broker.security.protocol`。