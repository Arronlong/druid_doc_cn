# 将德鲁伊与其他技术相结合

本页讨论了如何将德鲁伊与其他技术相结合。

## 与开源流技术集成

事件流可以存储在分布式消息总线（如Kafka）中，并通过分布式流
处理器系统（如Storm，Samza或Spark Streaming）进一步处理。流处理器处理的数据可以使用[Tranquility](https://github.com/druid-io/tranquility)库提供给Druid 。

![img](http://druid.io/docs/img/druid-production.png)

## 与SQL-on-Hadoop技术集成

理论上，德鲁伊应该与SQL-on-Hadoop技术很好地集成，例如Apache Drill，Spark SQL，Presto，Impala和Hive。