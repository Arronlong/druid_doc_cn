# 性能常见问题

## 我无法匹配您的基准测试结果

不正确的配置是我们看到人们试图部署德鲁伊的最大问题。教程中列出的示例配置是针对少量数据而设计的，其中所有节点都在一台机器上。对于实际生产使用来说，配置非常差。

## 我应该如何设置JVM堆？

JVM堆的大小实际上取决于您运行的Druid节点的类型。以下是一些注意事项。

[Broker节点](http://druid.io/docs/0.12.3/design/broker.html)主要使用JVM堆来合并历史和实时的结果。经纪人还使用堆外内存和处理groupBy查询的线程。我们在这里推荐20G-30G的堆。

[历史节点](http://druid.io/docs/0.12.3/design/historical.html)使用堆外内存来存储中间结果，默认情况下，所有段都可以在查询之前进行内存映射。通常，历史节点上可用的内存越多，可以提供的段越多，而不会将数据分页到磁盘上。在历史记录中，JVM堆用于[GroupBy查询](http://druid.io/docs/0.12.3/querying/groupbyquery.html)，一些用于中间计算的数据结构和一般处理。计算段的空间大小的一种方法是：memory_for_segments = total_memory - heap - direct_memory - jvm_overhead。请注意，total_memory在这里指的是cgroup可用的内存（如果在Linux上运行），默认情况下将是所有系统内存。

我们建议堆的250mb *（processing.numThreads）。

[协调器节点](http://druid.io/docs/0.12.3/design/coordinator.html)不需要堆外内存，堆用于加载有关所有段的信息，以确定需要加载，删除，移动或复制哪些段。

## 德鲁伊使用多少直接记忆？

处理查询的任何Druid节点（代理，摄取工作者和历史节点）都使用两种具有可配置大小的直接内存缓冲区：处理缓冲区和合并缓冲区。

每个处理线程分配一个处理缓冲区。此外，还有一个共享缓冲区共享池（目前仅用于GroupBy V2查询）。

直接内存使用的其他来源包括： - 加载列进行读取时，会为解压缩分配64KB直接缓冲区。 - 在摄取期间合并一组段时，为每个String类型列分配直接缓冲区，用于要合并的集合中的每个段。这些缓冲区的大小等于其段内String列的基数，乘以4个字节（缓冲区存储整数）。例如，如果合并了两个段，第一个段具有基数为1000的单个String列，而第二个段具有带基数500的String列，则合并步骤将分配（1000 + 500）* 4 = 6000个字节直接记忆。这些缓冲区用于跨段合并String列的值字典。这些“字典合并缓冲区”`druid.processing.numMergeBuffers`。

估算直接内存使用量的有用公式如下：

```
druid.processing.buffer.sizeBytes * (druid.processing.numMergeBuffers + druid.processing.numThreads + 1)
```

这`+1`是一个模糊参数，用于解释解压缩和字典合并缓冲区，可能需要根据被摄取/查询的数据的特征进行调整。操作员可以通过`-XX:MaxDirectMemorySize=<VALUE>`在命令行提供至少这一数量的直接内存。

## 什么是中间计算缓冲区？

中间计算缓冲区指定用于存储中间结果的缓冲区大小。Historical和Realtime节点中的计算引擎将使用此大小的暂存缓冲区在堆外执行所有中间计算。较大的值允许在数据上单次传递更多聚合，而较小的值可能需要更多传递，具体取决于正在执行的查询。默认大小为1073741824字节（1GB）。

## 什么是服务器maxSize？

Server maxSize设置节点可以容纳的最大累积段大小（以字节为单位）。更改此参数将通过控制节点上的内存/磁盘比率来影响性能。将此参数设置为大于节点上的总内存容量的值，并可能导致发生磁盘分页。该寻呼时间引入了查询延迟延迟。

## 我的日志真的很健谈，我可以将它们设置为异步写入吗？

是的，使用`log4j2.xml`类似于以下内容导致一些更繁琐的类异步写入：

```text
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <AsyncLogger name="io.druid.curator.inventory.CuratorInventoryManager" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <AsyncLogger name="io.druid.client.BatchServerInventoryView" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <!-- Make extra sure nobody adds logs in a bad way that can hurt performance -->
    <AsyncLogger name="io.druid.client.ServerInventoryView" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <AsyncLogger name ="io.druid.java.util.http.client.pool.ChannelResourceFactory" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```