## 6.3 配置

### 重要的客户端配置
最重要的生产者配置是：
- `cks`
- 压缩
- 批量

最重要的使用者配置是访存大小。

所有配置都记录在配置部分中。

### 生产服务器配置
这是生产服务器配置示例：
```bash
# ZooKeeper
zookeeper.connect=[list of ZooKeeper servers]
 
# Log configuration
num.partitions=8
default.replication.factor=3
log.dir=[List of directories. Kafka should have its own dedicated disk(s) or SSD(s).]
 
# Other configurations
broker.id=[An integer. Start with 0 and increment by 1 for each new broker.]
listeners=[list of listeners]
auto.create.topics.enable=false
min.insync.replicas=2
queued.max.requests=[number of concurrent requests]
```
我们的客户配置在不同的用例之间变化很大。