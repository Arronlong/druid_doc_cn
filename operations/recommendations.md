# 建议

# 一些一般准则

JVM标志：

```text
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=<something other than /tmp which might be mounted to volatile tmpfs file system>
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dorg.jboss.logging.provider=slf4j
-Dnet.spy.log.LoggerImpl=net.spy.memcached.compat.log.SLF4JLogger
-Dlog4j.shutdownCallbackRegistry=io.druid.common.config.Log4jShutdown
-Dlog4j.shutdownHookEnabled=true
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCApplicationConcurrentTime
-Xloggc:/var/logs/druid/historical.gc.log
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=50
-XX:GCLogFileSize=10m
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/logs/druid/historical.hprof
-XX:MaxDirectMemorySize=10240g
```

`ExitOnOutOfMemoryError`仅支持从JDK 8u92开始支持flag。对于旧版本，`-XX:OnOutOfMemoryError='kill -9 %p'`可以使用。

`MaxDirectMemorySize`限制jvm分配超过指定限制，通过将其设置为无限制jvm限制被取消并且OS级别内存限制仍然有效。确保Druid没有配置为分配比机器可用的更多堆外内存仍然很重要。这里的重要设置包括druid.processing.numThreads，druid.processing.numMergeBuffers和druid.processing.buffer.sizeBytes。

请注意，以上标志仅为一般准则。在必要时进行特定部署时要小心并随意更改它们。

此外，对于大型jvm堆，这里有一些垃圾收集效率指南，在某些情况下已知有帮助。 - 在tmpfs上挂载/ tmp（参见http://www.evanjones.ca/jvm-mmap-pause.html） - 在磁盘IO密集型节点（例如Historical和MiddleManager）上，GC和德鲁伊日志应写入不同的磁盘比写入数据的位置。- 禁用透明大页面（请参阅https://blogs.oracle.com/linux/performance-issues-with-transparent-huge-pages-thp） - 尝试使用`-XX:-UseBiasedLocking`jvm标志禁用偏向锁定。（参见https://dzone.com/articles/logging-stop-world-pauses-jvm）

# 使用UTC时区

我们建议对您的所有活动和节点使用UTC时区，不仅适用于德鲁伊，还适用于所有数据基础架构。这可以极大地缓解具有不一致时区的潜在查询问题。要在非UTC时区中[查询，](http://druid.io/docs/0.12.3/querying/granularities.html#period-granularities)请查看[查询粒度](http://druid.io/docs/0.12.3/querying/granularities.html#period-granularities)

# 固态硬盘

如果您没有运行完全在内存中的群集，则强烈建议将SSD用于历史和实时节点。SSD可以极大地减少在内存中分页数据所需的时间。

# JBOD vs RAID

历史节点在磁盘上存储大量段，并支持指定用于存储这些段的多个路径。通常，主机有多个配置了RAID的磁盘，这使得它们看起来像是操作系统的单个磁盘。如果RAID不是基于硬件控制器而是基于软件，则RAID可能会有特别的开销。因此，历史可能会通过JBOD提高磁盘吞吐量。

# 在可能的情况下使用Timeseries和TopN查询而不是GroupBy

Timeseries和TopN查询比其设计用例的groupBy查询更加优化并且明显更快。从应用程序发出多个topN或timeseries查询可能比单个groupBy查询更有效。

# 段大小很重要

细分通常应在300MB到700MB之间。太多的小段导致CPU利用效率低下，而太多的大段会影响查询性能，尤其是TopN查询。

# 阅读常见问题

你应该阅读人们在这里遇到的常见问题：

1）[摄取 - 常见问题](http://druid.io/docs/0.12.3/ingestion/faq.html)

2）[性能 - 常见问题](http://druid.io/docs/0.12.3/operations/performance-faq.html)