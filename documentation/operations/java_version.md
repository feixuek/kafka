## 6.4 Java版本

从安全角度来看，我们建议您使用`JDK 1.8`的最新发行版，因为较早的免费版本已披露了安全漏洞。`LinkedIn`当前正在使用G1收集器运行`JDK 1.8 u5`（希望升级到较新版本）。`LinkedIn`的调整如下所示：
```bash
-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC
-XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M
-XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80
```
供参考，以下是`LinkedIn`上最繁忙的集群之一（高峰时）的统计信息：
- 60个`broker`
- 50k分区（复制因子2）
- 每秒80万条消息
- 入站300MB/秒，出站1GB/秒以上

调优看起来相当激进，但是该集群中的所有代理都具有90％的GC暂停时间（约21ms），并且每秒执行的新`GC`少于1个。