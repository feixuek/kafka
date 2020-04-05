## 2. API
`Kafka`包含五个核心`api`：
1. 生产者`API`允许应用程序向`kafka`集群中的`topic`发送数据流。
2. 消费者`API`允许应用程序从`kafka`集群中的`topic`读取数据流。
3. 流`API`允许将来自输入`topic`的数据流输出到`topic`。
4. `Connect API`允许实现连接器，这些连接器可以不断地从某些源系统或应用程序拉入`Kafka`或从`Kafka`推入某些接收器系统或应用程序。
5. 管理员`API`允许管理和检查`topics`，`brokers`和其他`kafka`对象。

`Kafka`通过与语言无关的协议公开其所有功能，该协议具有可用于多种编程语言的客户端。但是，只有`Java`客户端是作为`Kafka`主项目的一部分维护的，其他`Java`客户端则可以作为独立的开源项目使用。[此处](https://cwiki.apache.org/confluence/display/KAFKA/Clients)提供了非`Java`客户端列表。

### 2.1生产者API
生产者`API`允许应用程序将数据流发送到`Kafka`集群中的`topic`。
显示如何使用生产者的示例在`javadocs`中给出 。

要使用生产者，可以使用以下`Maven`依赖项：
```maven
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.4.1</version>
</dependency>
```

### 2.2消费者API
消费者`API`允许应用程序从`Kafka`集群中的`topic`读取数据流。
显示如何使用使用者的示例在`javadocs`中给出 。

要使用使用者，可以使用以下`Maven`依赖项：
```maven
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.4.1</version>
</dependency>
```

### 2.3流API
流`API`允许将来自输入`topic`数据流输出到`topic`。
显示如何使用此库的示例在`javadocs`中给出

有关使用`Streams API`的其他文档，请参见[此处](https://kafka.apache.org/24/documentation/streams/)。

要使用`Kafka Streams`，可以使用以下`Maven`依赖项：
```maven
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>2.4.1</version>
</dependency>
```  
使用`Scala`时，您可以选择包括该`kafka-streams-scala`库。开发者指南中提供了有关使用`Kafka Streams DSL for Scala`的其他文档。

要将`Kafka Streams DSL for Scala`用于`Scala 2.12`，可以使用以下`maven`依赖项：

```maven
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams-scala_2.12</artifactId>
    <version>2.4.1</version>
</dependency>
```

### 2.4连接API
`Connect API`允许实现连接器，这些连接器不断地从某些源数据系统拉入`Kafka`或从`Kafka`推入某些接收器数据系统。
尽管`Connect`的许多用户并不需要直接使用此`API`，但是他们可以使用预构建的连接器而无需编写任何代码。有关使用`Connect`的其他信息，请参见[此处](https://kafka.apache.org/24/javadoc/index.html?org/apache/kafka/connect)。

那些想要实现自定义连接器的人可以参见`javadoc`。

### 2.5管理员API
`Admin API`支持管理和检查`topic`，`broker`，`ACL`和其他`Kafka`对象。
要使用`Admin API`，请添加以下`Maven`依赖项：

```maven
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.4.1</version>
</dependency>
```
有关`Admin API`的更多信息，请参见`javadoc`。