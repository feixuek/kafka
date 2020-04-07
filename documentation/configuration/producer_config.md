# 3.3 Producer配置

下面是生产者的配置：

> `key.serializer`：实现`org.apache.kafka.common.serialization.Serializer`接口的密钥的序列化程序类。
类型：`class` - 默认值： - 有效值： - 重要性：高

> `value.serializer`：实现`org.apache.kafka.common.serialization.Serializer`接口的`value`的`Serializer`类。
类型：`class` - 默认值： - 有效值： - 重要性：高

> `acks`：生产者要求领导者在确认请求完成之前已收到的确认数。这控制了发送记录的持久性。允许以下设置：
- `acks=0`如果设置为零，那么生产者将根本不等待服务器的任何确认。该记录将立即添加到套接字缓冲区中并视为已发送。在这种情况下，`retries`不能保证服务器已收到记录，并且配置将不会生效（因为客户端通常不会知道任何故障）。为每条记录提供的偏移量将始终设置为-1。
- `acks=1`这意味着领导者会将记录写入其本地日志，但会在不等待所有关注者的完全确认的情况下做出响应。在这种情况下，如果领导者在确认记录后立即失败，但是在跟随者复制记录之前，则记录将丢失。
- `acks=all`这意味着领导者将等待全套同步副本确认记录。这保证了只要至少一个同步副本仍处于活动状态，记录就不会丢失。这是最有力的保证。这等效于`acks = -1`设置。

类型：`string` - 默认值：1 - 有效值：[`all`，-1、0、1] - 重要性：高

> `bootstrap.servers`：用于建立与`Kafka`集群的初始连接的主机/端口对列表。客户端将使用所有服务器，而与此处指定用于引导的服务器无关—该列表仅影响用于发现整套服务器的初始主机。此列表应采用形式`host1:port1,host2:port2,...`。由于这些服务器仅用于初始连接以发现完整的集群成员身份（可能会动态更改），因此此列表不必包含完整的服务器集合（但是，如果服务器关闭，则可能需要多个服务器）。 。
类型：`list` - 默认值："" - 有效值：非空字符串 - 重要性：高

> `buffer.memory`：生产者可以用来缓冲等待发送到服务器的记录的总内存字节。如果记录的发送速度超过了将记录发送到服务器的速度，则生产者将对其进行阻止，`max.block.ms`然后引发异常。
此设置应大致对应于生产者将使用的总内存，但不是硬性约束，因为并非生产者使用的所有内存都用于缓冲。一些额外的内存将用于压缩（如果启用了压缩）以及维护进行中的请求。

类型：`long` - 默认值：33554432 - 有效值：[0，...] - 重要性：高

> `compression.type`：生产者生成的所有数据的压缩类型。默认值为无（即不压缩）。有效值为`none`，`gzip`，`snappy`，`lz4`，或`zstd`。压缩涉及全部批次的数据，因此，批处理的效率也会影响压缩率（批处理越多意味着压缩效果越好）。
类型：`string` - 默认值：`none` - 有效值： - 重要性：高

> `retries`：设置大于零的值将导致客户端重新发送其发送失败并带有潜在的瞬时错误的任何记录。请注意，此重试与客户端在收到错误后重新发送记录没有什么不同。允许重试而不将其设置`max.in.flight.requests.per.connection`为1可能会更改记录的顺序，因为如果将两个批次发送到单个分区，并且第一个失败并被重试，但是第二个成功，则第二个批次中的记录可能会首先出现。另外请注意，如果`delivery.timeout.ms`成功配置的超时时间在确认之前首先到期，则在重试次数用完之前，生产请求将失败。用户通常应该更喜欢保留此配置未设置，而用于`delivery.timeout.ms`控制重试行为。
类型：`int` - 默认值：214748364 - 7有效值：[0，...，2147483647] - 重要性：高

> `ssl.key.password`：密钥存储文件中私钥的密码。这对于客户端是可选的。
类型：`password` - 默认值：`null` - 有效值： - 重要性：高

> `ssl.keystore.location`：密钥存储文件的位置。这对于客户端是可选的，并且可以用于客户端的双向身份验证。
类型：`string` - 默认值：`null` - 有效值： - 重要性：高

> `ssl.keystore.password`：密钥存储文件的存储密码。这对于客户端是可选的，并且仅在配置了`ssl.keystore.location`时才需要。
类型：`password` - 默认值：`null` - 有效值： - 重要性：高

> `ssl.truststore.location`：信任库文件的位置。
类型：`string` - 默认值：`null` - 有效值： - 重要性：高

> `ssl.truststore.password`：信任存储文件的密码。如果未设置密码，对信任库的访问仍然可用，但是完整性检查被禁用。
类型：`password` - 默认值：`null` - 有效值： - 重要性：高

> `batch.size`：每当将多个记录发送到同一分区时，生产者将尝试将记录一起批处理成更少的请求。这有助于提高客户端和服务器的性能。此配置控制默认的批处理大小（以字节为单位）。
不会尝试批处理大于此大小的记录。

发送给代理的请求将包含多个批次，每个分区一个，并包含可发送的数据。

较小的批处理量将使批处理变得不那么普遍，并且可能会降低吞吐量（批处理量为零将完全禁用批处理）。非常大的批处理大小可能会浪费内存，因为我们总是在预期其他记录的情况下分配指定批处理大小的缓冲区。

类型：`int` - 默认值：`16384` - 有效值：`[0，...]` - 重要性：中等

> `client.dns.lookup`：控制客户端如何使用`DNS`查找。如果设置为`use_all_dns_ips`那么，当查找返回一个主机名的多个`IP`地址时，将在连接失败之前尝试全部连接。同时适用于引导服务器和公告服务器。如果值为，`resolve_canonical_bootstrap_servers_only`则将解析每个条目并将其扩展为规范名称列表。
类型：`string` - 默认值：`default` - 有效值：[`default`，`use_all_dns_ips，resolve_canonical_bootstrap_servers_only`] - 重要性：中等

> `client.id`：发出请求时传递给服务器的`ID`字符串。其目的是通过允许将逻辑应用程序名称包含在服务器端请求日志中，从而能够跟踪`IP/端口`以外的请求源。
类型：`string` - 默认值："" - 有效值： - 重要性：中等

> `connections.max.idle.ms`：在此配置指定的毫秒数后关闭空闲连接。
类型：`long` - 默认值：540000 - 有效值： - 重要性：中等

> `delivery.timeout.ms`：在调用`return`之后报告成功或失败的时间的上限`send()`。这限制了记录在发送之前将被延迟的总时间，等待来自代理的确认的时间（如果期望）以及允许可重发的发送失败的时间。如果遇到不可恢复的错误，重试已用尽，或将记录添加到已达到较早的交付到期期限的批次，则生产者可能会报告未能在此配置之前发送记录失败。此配置的值应大于或等于`request.timeout.ms`和`linger.ms`。
类型：`int` - 默认值：120000 - 有效值：[0，...] - 重要性：中等

> `linger.ms`：生产者将在请求传输之间到达的所有记录归为一个批处理请求。通常，只有在记录到达速度快于记录发送速度时，才在负载下发生这种情况。但是，在某些情况下，即使在中等负载下，客户端也可能希望减少请求的数量。此设置通过添加少量的人为延迟来实现此目的，也就是说，生产者将立即等待直到给定的延迟以允许其他记录被发送，以便发送者可以被分批处理，而不是立即发送记录。可以认为这类似于`TCP`中的Nagle算法。此设置给出了批处理延迟的上限：一旦获得`batch.size`不论此设置如何，都会立即发送一个分区的大量记录，但是，如果我们为该分区积累的字节数少于该字节数，我们将在指定的时间“徘徊”以等待更多的记录显示。此设置默认为0（即无延迟）。`linger.ms=5`例如，设置将具有减少发送请求的数量的效果，但是在没有负载的情况下，发送记录的延迟将增加5毫秒。
类型：`long` - 默认值：0 - 有效值：[0，...] - 重要性：中等

> `max.block.ms`：配置控制阻塞的时间`KafkaProducer.send()`和`KafkaProducer.partitionsFor()`长度。由于缓冲区已满或元数据不可用，可以阻塞这些方法。用户提供的序列化器或分区器中的阻塞将不计入此超时。
类型：`long` - 默认值：60000 - 有效值：[0，...] - 重要性：中等

> `max.request.size`：请求的最大大小（以字节为单位）。此设置将限制生产者将在单个请求中发送的记录批数，以避免发送大量请求。这实际上也是最大记录批次大小的上限。请注意，服务器对记录批大小有自己的上限，该上限可能与此不同。
类型：`int` - 默认值：1048576 - 有效值：[0，...] - 重要性：中等

> `partitioner.class`：实现`org.apache.kafka.clients.producer.Partitioner`接口的`Partitioner`类。
类型：`class` - 默认值：`org.apache.kafka.clients.producer.internals.DefaultPartitioner` - 有效值： - 重要性：中等

> `receive.buffer.bytes`：读取数据时要使用的`TCP`接收缓冲区（`SO_RCVBUF`）的大小。如果值为-1，则将使用操作系统默认值。
类型：`int` - 默认值：32768 - 有效值：[-1，...] - 重要性：中等

> `request.timeout.ms`：该配置控制客户端等待请求响应的最长时间。如果超时之前仍未收到响应，则客户端将在必要时重新发送请求，如果重试已用尽，则客户端将使请求失败。它应该大于`replica.lag.time.max.ms`（代理配置），以减少由于不必要的生产者重试而导致消息重复的可能性。
类型：`int` - 默认值：30000 - 有效值：[0，...] - 重要性：中等

> `sasl.client.callback.handler.class`：实现`AuthenticateCallbackHandler`接口的SASL客户端回调处理程序类的完全限定名称。
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等

> `sasl.jaas.config`：`SAS`L连接的`JAAS`登录上下文参数，采用`JAAS`配置文件使用的格式。这里描述了 `JAAS`配置文件格式。值的格式为：“`loginModuleClass controlFlag (optionName=optionValue)*;`。对于代理，配置必须以侦听器前缀和小写的SASL机制名称作为前缀。例如，需要`listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config = com.example.ScramLoginModule`；
类型：`password` - 默认值：`null` - 有效值： - 重要性：中等

> `sasl.kerberos.service.name`：`Kafka`运行时使用的`Kerberos`主体名称。这可以在`Kafka`的`JAAS`配置或`Kafka`的配置中定义。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等

> `sasl.login.callback.handler.class`：实现`AuthenticateCallbackHandler`接口的SASL登录回调处理程序类的全限定名。对于代理，登录回调处理程序配置必须以侦听器前缀和小写的SASL机制名称作为前缀。例如，`listener.name.sasl_ssl.scram-sha-256.sasl.login.callback.handler.class = com.example.CustomScramLoginCallbackHandler`
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等

> `sasl.login.class`：实现`Login`接口的类的完全限定名称。对于代理，登录配置必须以侦听器前缀和小写的SASL机制名称作为前缀。例如，`listener.name.sasl_ssl.scram-sha-256.sasl.login.class = com.example.CustomScramLogin`
类型：`class` - 默认值：`null` - 有效值： - 重要性：中等

> `sasl.mechanism`：用于客户端连接的`SASL`机制。这可以是安全提供程序可用的任何机制。`GSSAPI`是默认机制。
类型：`string` - 默认值：`GSSAPI` - 有效值： - 重要性：中等

> `security.protocol`：用于与代理进行通信的协议。有效值为：`PLAINTEXT`，`SSL`，`SASL_PLAINTEXT`，`SASL_SSL`。
类型：`string` - 默认值：`PLAINTEXT` - 有效值： - 重要性：中等

> `send.buffer.bytes`：发送数据时要使用的`TCP`发送缓冲区（`SO_SNDBUF`）的大小。如果值为-1，则将使用操作系统默认值。
类型：`int` - 默认值：131072 - 有效值：[-1，...] - 重要性：中等

> `ssl.enabled.protocols`：为`SSL`连接启用的协议列表。
类型：`list` - 默认值：`TLSv1.2`，`TLSv1.1`，`TLSv1` - 有效值： - 重要性：中等

> `ssl.keystore.type`：密钥存储文件的文件格式。这对于客户端是可选的。
类型：`string` - 默认值：`JKS` - 有效值： - 重要性：中等

> `ssl.protocol`：用于生成`SSLContext`的SSL协议。默认设置为TLS，在大多数情况下都可以使用。最近的JVM中允许的值为`TLS`，`TLSv1.1`和`TLSv1.2`。较早的JVM中可能支持`SSL`，`SSLv2`和`SSLv3`，但由于已知的安全漏洞，因此不鼓励使用它们。
类型：`string` - 默认值：`TLS` - 有效值： - 重要性：中等

> `ssl.provider`：用于`SSL`连接的安全提供程序的名称。缺省值是JVM的缺省安全提供程序。
类型：`string` - 默认值：`null` - 有效值： - 重要性：中等

> `ssl.truststore.type`：信任库文件的文件格式。
类型：`string` - 默认值：`JKS` - 有效值： - 重要性：中等

> `enable.idempotence`：设置为`true`时，生产者将确保每个消息的确切副本被写入流中。如果为`false`，则生产者由于代理失败等原因而重试，可能会在流中写入重试消息的副本。请注意，启用幂等性必须`max.in.flight.requests.per.connection`小于或等于5，`retries`大于0且acks必须为`all`。如果用户未明确设置这些值，则将选择合适的值。如果设置了不兼容的值，`ConfigException`将抛出。
类型：`bool` - 默认值：`false` - 有效值： - 重要性：低

> `interceptor.classes`：用作拦截器的类的列表。通过实现该`org.apache.kafka.clients.producer.ProducerInterceptor`接口，您可以在生产者收到的记录发布到Kafka集群之前对其进行拦截（并可能对其进行突变）。默认情况下，没有拦截器。
类型：`list` - 默认值："" - 有效值：非空字符串 - 重要性：低

> `max.in.flight.requests.per.connection`：客户端在阻塞之前将通过单个连接发送的未确认请求的最大数量。请注意，如果将此设置设置为大于1并且发送失败，则存在由于重试而导致消息重新排序的风险（即，如果启用了重试）。
类型：`int` - 默认值：5 - 有效值：[1，...] - 重要性：低

> `metadata.max.age.ms`：以毫秒为单位的时间段，在此之后我们强制刷新元数据，即使我们没有看到任何分区领导更改也可以主动发现任何新的代理或分区。
类型：`long` - 默认值：300000 - 有效值：[0，...] - 重要性：低

> `metric.reporters`：用作指标报告者的类列表。实施该`org.apache.kafka.common.metrics.MetricsReporter`接口允许插入将通知新度量标准创建的类。始终包含`JmxReporter`来注册JMX统计信息。
类型：`list` - 默认值："" - 有效值：非空字符串 - 重要性：低

> `metrics.num.samples`：为计算指标而维护的样本数。
类型：`int` - 默认值：2 - 有效值：[1，...] - 重要性：低

> `metrics.recording.level`：指标的最高记录级别。
类型：`string` - 默认值：`INFO` - 有效值：[`INFO`，`DEBUG`] - 重要性：低

> `metrics.sample.window.ms`：度量样本被计算的时间窗口。
类型：`long` - 默认值：30000 - 有效值：[0，...] - 重要性：低

> `reconnect.backoff.max.ms`：当重新连接到反复连接失败的代理时，要等待的最长时间（以毫秒为单位）。如果提供此选项，则对于每个连续的连接失败，每个主机的退避量将成倍增加，直至达到此最大值。在计算退避增量之后，添加20％的随机抖动以避免连接风暴。
类型：`long` - 默认值：1000 - 有效值：[0，...] - 重要性：低

>`reconnect.backoff.ms`：尝试重新连接到给定主机之前要等待的基本时间。这样可以避免以紧密的循环重复连接到主机。此补偿适用于客户端到代理的所有连接尝试。
类型：`long` - 默认值：50 - 有效值：[0，...] - 重要性：低

> `retry.backoff.ms`：尝试重试对给定主题分区的失败请求之前要等待的时间。在某些故障情况下，这避免了在紧密循环中重复发送请求。
类型：`long` - 默认值：100 - 有效值：[0，...] - 重要性：低

> `sasl.kerberos.kinit.cmd`：`Kerberos kinit`命令路径。
类型：`string` - 默认值：`/usr/bin/kinit` - 有效值： - 重要性：低

> `sasl.kerberos.min.time.before.relogin`：两次刷新尝试之间的登录线程睡眠时间。
类型：`long` - 默认值：60000 - 有效值： - 重要性：低

> `sasl.kerberos.ticket.renew.jitter`：添加到续订时间的随机抖动百分比。
类型：`long` - 默认值：0.05 - 有效值： - 重要性：低

> `sasl.kerberos.ticket.renew.window.factor`：登录线程将一直休眠，直到达到从上次刷新到票证到期的指定时间窗口因子为止，此时它将尝试续订票证。
类型：`double` - 默认值：0.8 - 有效值： - 重要性：低

> `sasl.login.refresh.buffer.seconds`：刷新凭证时要保留的凭证过期前的缓冲区时间，以秒为单位。如果刷新将以比缓冲区秒数更接近到期的方式发生，则刷新将被上移以保持尽可能多的缓冲区时间。合法值介于0到3600（1小时）之间；如果未指定任何值，则使用默认值300（5分钟）。如果此值和`sasl.login.refresh.min.period.seconds`的总和超过凭据的剩余生存期，则两者都将被忽略。当前仅适用于`OAUTHBEARER`。
类型：`short` - 默认值：300 - 有效值：[0，...，3600] - 重要性：低

> `sasl.login.refresh.min.period.seconds`：登录刷新线程在刷新凭证之前等待的最短时间（以秒为单位）。合法值在0到900之间（15分钟）；如果未指定任何值，则使用默认值60（1分钟）。如果此值和`sasl.login.refresh.buffer.seconds`的总和超过凭据的剩余生存期，则两者都将被忽略。当前仅适用于`OAUTHBEARER`。
类型：`short` - 默认值：60 - 有效值：[0，...，900] - 重要性：低

> `sasl.login.refresh.window.factor`：登录刷新线程将休眠，直到达到相对于凭据生存期的指定窗口因子为止，此时它将尝试刷新凭据。合法值介于0.5（50％）至1.0（100％）之间；如果未指定任何值，则使用默认值0.8（80％）。当前仅适用于`OAUTHBEARER`。
类型：`double` - 默认值：0.8 - 有效值：[0.5，...，1.0] - 重要性：低

> `sasl.login.refresh.window.jitter`：相对于凭证生存期的最大随机抖动量，添加到登录刷新线程的睡眠时间中。合法值介于0到0.25（25％）之间（含）；如果未指定任何值，则使用默认值0.05（5％）。当前仅适用于`OAUTHBEARER`。
类型：`double` - 默认值：0.05 - 有效值：[0.0，...，0.25] - 重要性：低

> `security.providers`：可配置创建者类的列表，每个创建者类返回一个实现安全算法的提供者。这些类应实现`org.apache.kafka.common.security.auth.SecurityProviderCreator`接口。
类型：`string` - 默认值：`null` - 有效值： - 重要性：低

> `ssl.cipher.suites`：密码套件列表。这是认证，加密，MAC和密钥交换算法的命名组合，用于协商使用TLS或SSL网络协议的网络连接的安全设置。默认情况下，支持所有可用的密码套件。
类型：`list` - 默认值：`null` - 有效值： - 重要性：低

> `ssl.endpoint.identification.algorithm`：使用服务器证书验证服务器主机名的端点标识算法。
类型：`string` - 默认值：`https` - 有效值： - 重要性：低

> `ssl.keymanager.algorithm`：密钥管理器工厂用于SSL连接的算法。缺省值是为`Java`虚拟机配置的密钥管理器工厂算法。
类型：`string` - 默认值：`SunX509` - 有效值： - 重要性：低

> `ssl.secure.random.implementation`：用于`SSL`加密操作的`SecureRandom PRNG`实现。
类型：`string` - 默认值：`null` - 有效值： - 重要性：低

> `ssl.trustmanager.algorithm`：信任管理器工厂用于`SSL`连接的算法。默认值是为`Java`虚拟机配置的信任管理器工厂算法。
类型：`string` - 默认值：`PKIX` - 有效值： - 重要性：低

> `transaction.timeout.ms`：事务协调器在主动中止正在进行的事务之前等待生产者更新事务状态的最长时间（以毫秒为单位）。如果该值大于`int`中的`transaction.max.timeout.ms`设置代理，请求将失败并显示`InvalidTransactionTimeout`错误。
类型：`int` - 默认值：60000 - 有效值： - 重要性：低

> `transactional.id`：用于事务传递的`TransactionalId`。这使跨越多个生产者会话的可靠性语义成为可能，因为它允许客户端保证使用相同`TransactionalId`的事务在开始任何新事务之前已经完成。如果未提供`TransactionalId`，则生产者将限于幂等传递。请注意，`enable.idempotence`如果配置了`TransactionalId` ，则必须启用它。默认值为`null`，这表示无法使用事务。请注意，默认情况下，交易需要至少三个经纪人的集群，这是建议的生产设置。对于开发，您可以通过调整代理设置来更改此设置`transaction.state.log.replication.factor`。
类型：`string` - 默认值：`null` - 有效值：非空字符串 - 重要性：低