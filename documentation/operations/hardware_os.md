## 6.5 硬件和操作系统

我们正在使用具有`24GB`内存的双四核`Intel Xeon`机器。
您需要足够的内存来缓冲活动的读取器和写入器。您可以通过假设您希望能够缓冲30秒钟并以`write_throughput * 30`计算您的内存需求来对内存需求进行估算。

磁盘吞吐量很重要。我们有`8x7200 rpm SATA`驱动器。通常，磁盘吞吐量是性能瓶颈，并且磁盘越多越好。根据配置刷新行为的方式，您可能会也可能不会从更昂贵的磁盘中受益（如果您经常强制执行刷新，则较高的`RPM SAS`驱动器可能会更好）。

### 操作系统
`Kafka`应该可以在任何`Unix`系统上良好运行，并且已经在`Linux`和`Solaris`上进行了测试。
我们已经看到了在`Windows`上运行的一些问题，尽管我们很乐意对此进行更改，但`Windows`目前不是受良好支持的平台。

不太可能需要太多的OS级调整，但是存在三种潜在的重要OS级配置：

- 文件描述符限制：`Kafka`将文件描述符用于日志段和打开的连接。如果代理程序托管许多分区，则除了代理程序建立的连接数之外，还要考虑代理程序至少需要`(number_of_partitions)*(partition_size/segment_size)`来跟踪所有日志段。我们建议代理程序至少使用100000个允许的文件描述符作为起点。注意：`mmap()`函数为与文件描述符`fildes`关联的文件添加了额外的引用，该引用不会被该文件描述符上的后续`close()`删除。没有更多的文件映射时，将删除此引用。
- 最大套接字缓冲区大小：能够增加，以使数据中心之间的高性能数据传输这里描述。
- 一个进程可能具有的最大内存映射区域数（即`vm.max_map_count`）。请参阅`Linux`内核文档。在考虑代理可能具有的最大分区数量时，应注意此操作系统级别的属性。默认情况下，在许多`Linux`系统上，`vm.max_map_count`的值大约为65535。每个分区分配的每个日志段都需要一对索引/时间索引文件，并且每个文件占用1个映射区域。换句话说，每个日志段使用2个地图区域。因此，每个分区至少需要2个地图区域，只要它承载单个日志段即可。也就是说，在代理上创建50000分区将导致分配100000个地图区域，并可能在默认`vm.max_map_count`的系统上导致`OutOfMemoryError`（映射失败）导致代理崩溃。请记住，每个分区的日志段数取决于段大小，负载强度，保留策略，并且通常，

### 磁盘和文件系统
我们建议使用多个驱动器以获得良好的吞吐量，并且不要与应用程序日志或其他OS文件系统活动共享用于Kafka数据的相同驱动器，以确保良好的延迟。您可以将这些驱动器一起`RAID`到单个卷中或格式化，然后将每个驱动器安装为自己的目录。由于`Kafka`具有复制功能，因此`RAID`提供的冗余也可以在应用程序级别提供。这个选择有几个权衡。
如果配置了多个数据目录，则会将分区循环分配给数据目录。每个分区将完全位于数据目录之一中。如果分区之间的数据平衡不佳，则可能导致磁盘之间的负载不平衡。

`RAID`在平衡磁盘之间的负载方面可能会做得更好（尽管并非总是如此），因为它可以在较低级别上平衡负载。RAID的主要缺点是，它通常会严重影响写入吞吐量并减少可用磁盘空间。

RAID的另一个潜在好处是可以容忍磁盘故障。但是，我们的经验是，重建RAID阵列的I / O占用大量资源，从而有效地禁用了服务器，因此这并不能提供太多的实际可用性改进。

### 应用程序与操作系统刷新管理
`Kafka`始终会立即将所有数据写入文件系统，并支持配置刷新策略的功能，该策略控制何时使用刷新将数据从OS缓存中强制出到磁盘上。可以控制此刷新策略，以在一段时间后或在写入一定数量的消息之后将数据强制到磁盘。在此配置中有几种选择。
`Kafka`必须最终调用`fsync`才能知道数据已刷新。当从崩溃中恢复任何未知的日志段时，`Kafka`将通过检查其消息的CRC来检查每条消息的完整性，并在启动时执行的恢复过程中重建附带的偏移索引文件。

请注意，`Kafka`中的持久性不需要将数据同步到磁盘，因为发生故障的节点将始终从其副本中恢复。

我们建议使用默认刷新设置，该设置将完全禁用应用程序`fsync`。这意味着依靠操作系统和`Kafka`自己的后台刷新来完成后台刷新。这为所有用途提供了最佳的解决方案：无需调节旋钮，提高吞吐量和延迟，并提供完全恢复保证。通常，我们认为复制提供的保证要强于同步到本地磁盘，但是偏执狂仍然更愿意同时拥有两者，并且仍然支持应用程序级`fsync`策略。

使用应用程序级刷新设置的缺点是，它在磁盘使用模式上效率较低（它给操作系统减少了重新排序写操作的余地），并且由于大多数`Linux`文件系统中的`fsync`阻止了对文件的写操作，因此引入了延迟。后台刷新进行更精细的页面级锁定。

通常，您不需要对文件系统进行任何低级调整，但是在接下来的几节中，我们将对其中的一些内容进行详细介绍，以防它有用。

### 了解Linux OS刷新行为
在Linux中，写入文件系统的数据将保留在页面缓存中，直到必须将其写出到磁盘为止（由于应用程序级`fsyn`c或操作系统自身的刷新策略）。数据刷新是通过一组称为`pdflush`的后台线程完成的（或在2.6.32版的内核“冲洗线程”中）。
`Pdflush`具有可配置的策略，该策略控制可以在缓存中维护多少脏数据以及必须将多脏数据写回到磁盘的时间。此处描述了此策略。当`Pdflush`无法跟上写入数据的速度时，它将最终导致写入过程阻塞写入中的延迟，从而减慢数据的累积。

您可以通过以下方式查看操作系统内存使用情况的当前状态：
```bash
cat /proc/meminfo
```
这些值的含义在上面的链接中描述。
与用于存储将要写到磁盘的数据的进程内缓存相比，使用页面缓存具有多个优点：

- `I/O`调度程序会将连续的小写操作批量处理为更大的物理写操作，从而提高了吞吐量。
- `I/O`调度程序将尝试对写入进行重新排序，以最小化磁盘头的移动，从而提高了吞吐量。
- 它会自动使用机器上的所有可用内存

### 文件系统选择
`Kafka`使用磁盘上的常规文件，因此它对特定文件系统没有硬性依赖。但是，使用最多的两个文件系统是`EXT4`和`XFS`。从历史上讲，`EXT4`的使用率更高，但是最近对`XFS`文件系统的改进表明，它对`Kafka`的工作负载具有更好的性能特征，而不会影响稳定性。

使用各种文件系统创建和安装选项，在具有大量消息负载的群集上执行了比较测试。`Kafka`中受监控的主要指标是“请求本地时间”，指示附加操作花费的时间。XFS带来了更好的本地时间（最佳EXT4配置为160ms与250ms +），并且平均等待时间更短。XFS性能还显示磁盘性能的可变性较小。

#### 通用文件系统说明
对于用于数据目录的任何文件系统，在`Linux`系统上，建议在安装时使用以下选项：
- `noatime`：读取文件时，此选项禁用文件`atime`（上次访问时间）属性的更新。这可以消除大量的文件系统写操作，尤其是在引导使用方的情况下。`Kafka`根本不依赖`atime`属性，因此可以安全地禁用它。

#### XFS注释
`XFS`文件系统具有大量的自动调整功能，因此，无论在文件系统创建时还是在安装时，都不需要更改默认设置。值得考虑的唯一调整参数是：

- `largeio`：这会影响`stat`调用报告的首选`I/O`大小。尽管这可以在较大的磁盘写入上实现更高的性能，但实际上对性能的影响很小或没有影响。
- `nobarrier`：对于具有电池后备缓存的基础设备，此选项可以通过禁用定期写刷新来提供更高的性能。但是，如果基础设备的行为良好，它将向文件系统报告它不需要刷新，并且此选项将无效。

#### EXT4注意事项
EXT4是Kafka数据目录的文件系统的一种可服务选择，但是要获得最佳性能，则需要调整几个安装选项。另外，这些选项在故障情况下通常是不安全的，并且将导致更多的数据丢失和损坏。对于单个代理故障，这不是什么大问题，因为可以擦除磁盘并从群集重建副本。在多故障情况下（例如断电），这可能意味着不容易恢复的基础文件系统（以及数据）损坏。可以调整以下选项：

- `data = writeback`：`Ext4`默认为`data = ordered`，这在某些写入操作中具有很强的顺序。`Kafka`不需要此排序，因为它对所有未刷新的日志执行非常偏执的数据恢复。此设置消除了排序约束，并且似乎大大减少了延迟。
- 禁用日志记录：日志记录是一个折衷方案：它使服务器崩溃后重新启动速度更快，但是它引入了很多附加锁定，从而增加了写入性能。那些不关心重新启动时间并希望减少写入延迟峰值的主要来源的人可以完全关闭日记功能。
- `commit = num_secs`：调整ext4提交到其元数据日志的频率。将此值设置为较低的值可以减少崩溃期间未刷新数据的丢失。将此值设置为较高的值将提高吞吐量。
- `nob`h：使用`data = writeback`模式时，此设置控制其他排序保证。对于Kafka，这应该是安全的，因为我们不依赖于写入顺序，并且可以提高吞吐量和延迟。
- `delalloc`：延迟分配意味着文件系统避免在物理写入发生之前分配任何块。这使ext4可以分配较大的范围而不是较小的页面，并有助于确保按顺序写入数据。此功能非常适合吞吐量。它似乎确实涉及文件系统中的某些锁定，这增加了一些延迟差异。