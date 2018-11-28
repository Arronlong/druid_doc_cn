# 加载流

可以使用[Tranquility](https://github.com/druid-io/tranquility)（德鲁伊知晓客户端）或[Kafka索引服务](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)在德鲁伊摄取流。

## 宁静（流推）

如果您有一个生成流的程序，那么您可以将该流直接推送到Druid中。通过这种方法，Tranquility嵌入在您的数据生成应用程序中。Tranquility附带了Storm和Samza流处理器的绑定。它还有一个直接的API，可以在任何基于JVM的程序中使用，例如Spark Streaming或Kafka使用者。

Tranquility为您无缝地处理分区，复制，服务发现和架构翻转，无需停机。您只需要定义您的德鲁伊模式。

有关示例和更多信息，请参阅[Tranquility README](https://github.com/druid-io/tranquility)。

教程中还提供了一个[教程：使用HTTP推送加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-tranquility.html)。

## 卡夫卡索引服务（Stream Pull）

德鲁伊可以使用[卡夫卡索引服务](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)从卡夫卡流中提取数据。

Kafka索引服务可以在Overlord上配置*主管*，通过管理Kafka索引任务的创建和生命周期来促进Kafka的摄取。这些索引任务使用Kafka自己的分区和偏移机制读取事件，因此能够提供完全一次摄取的保证。他们还能够从Kafka读取非近期事件，并且不受其他摄取机制强加的窗口期限的影响。主管监督索引任务的状态，以协调切换，管理故障并确保维护可伸缩性和复制要求。

教程中提供了一个[教程：从Kafka加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-kafka.html)。