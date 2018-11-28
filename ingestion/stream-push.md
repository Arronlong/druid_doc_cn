## 流推

德鲁伊可以通过[Tranquility](https://github.com/druid-io/tranquility/blob/master/README.md)连接到任何流数据源， [Tranquility](https://github.com/druid-io/tranquility/blob/master/README.md)是一个实时向德鲁伊推送流的软件包。德鲁伊没有与Tranquility捆绑在一起，你必须下载发行版。

如果您之前从未将流数据加载到带有Tranquility的Druid中，我们建议您首先尝试使用 [流加载教程](http://druid.io/docs/0.12.3/tutorials/tutorial-tranquility.html)，然后再返回此页面。

请注意，对于所有流提取选项，您必须确保传入数据足够新（在当前时间的可[配置windowPeriod](http://druid.io/docs/0.12.3/ingestion/stream-push.html#segmentgranularity-and-windowperiod)内）。旧邮件不会实时处理。最好使用[批量提取](http://druid.io/docs/0.12.3/ingestion/batch-ingestion.html)处理历史数据 。

### 服务器

德鲁伊可以使用[Tranquility Server](https://github.com/druid-io/tranquility/blob/master/docs/server.md)，它允许您在不开发JVM应用程序的情况下将数据发送到Druid。您可以运行与Druid middleManagers和历史流程共存的Tranquility服务器。

通过发出以下命令启动Tranquility服务器：

```bash
bin/tranquility server -configFile <path_to_config_file>/server.json
```

要自定义Tranquility Server：

- 在`server.json`，定制`properties`和`dataSources`。
- 如果您的服务器已经运行Tranquility，请将它们停止（CTRL-C）并再次启动它们。

有关自定义的提示`server.json`，请参阅 *编写提取规范*教程和 [Tranquility Server文档](https://github.com/druid-io/tranquility/blob/master/docs/server.md)。

### JVM应用程序和流处理器

Tranquility也可以作为库嵌入基于JVM的应用程序中。您可以使用[Core API](https://github.com/druid-io/tranquility/blob/master/docs/core.md)直接在自己的程序中执行此操作 ，也可以使用Tranquility中捆绑的连接器，用于流行的基于JVM的流处理器，如 [Storm](https://github.com/druid-io/tranquility/blob/master/docs/storm.md)， [Samza](https://github.com/druid-io/tranquility/blob/master/docs/samza.md)， [Spark Streaming](https://github.com/druid-io/tranquility/blob/master/docs/spark.md)和 [Flink](https://github.com/druid-io/tranquility/blob/master/docs/flink.md)。

### 卡夫卡（已弃用）

注意：Tranquility Kafka已弃用。请使用[Kafka Indexing Service](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)来加载Kafka的数据。

[Tranquility Kafka](https://github.com/druid-io/tranquility/blob/master/docs/kafka.md) 允许您在不编写任何代码的情况下将Kafka中的数据加载到Druid中。您只需要一个配置文件。

通过发出以下命令启动Tranquility服务器：

```bash
bin/tranquility kafka -configFile <path_to_config_file>/kafka.json
```

要在单机快速入门配置中自定义Tranquility Kafka：

- 在`kafka.json`，定制`properties`和`dataSources`。
- 如果您已经运行Tranquility，请将其停止（CTRL-C）并再次启动。

有关自定义的提示`kafka.json`，请参阅 [Tranquility Kafka文档](https://github.com/druid-io/tranquility/blob/master/docs/kafka.md)。

## 概念

### 任务创建

Tranquility可自动创建Druid实时索引任务，为您无缝地处理分区，复制，服务发现和架构翻转，无需停机。您永远不必编写代码来直接处理单个任务。但是，了解Tranquility如何创建任务会很有帮助。

宁静定期产生相对短暂的任务，每个人处理少量的 [德鲁伊片段](http://druid.io/docs/0.12.3/design/segments.html)。Tranquility通过ZooKeeper协调所有任务创建。您可以使用相同的配置启动任意数量的Tranquility实例，即使在不同的计算机上，它们也会发送到同一组任务。

有关[Tranquility](https://github.com/druid-io/tranquility/blob/master/docs/overview.md) 如何管理任务的更多详细信息，请参阅[Tranquility概述](https://github.com/druid-io/tranquility/blob/master/docs/overview.md)。

### segmentGranularity和windowPeriod

segmentGranularity是每个任务生成的段所涵盖的时间段。例如，“小时”的segmentGranularity将生成创建每个小时一小时的段的任务。

windowPeriod是事件允许的松弛时间。例如，windowPeriod为十分钟（默认值）意味着将删除过去时间超过十分钟或将来超过十分钟的任何事件。

这些是重要的配置，因为它们会影响任务的有效期，以及数据在传递到历史节点之前在实时系统中保留多长时间。例如，如果您的配置具有segmentGranularity“hour”和windowPeriod十分钟，则任务将保持在一小时十分钟内监听事件。因此，为了防止过多的任务累积，建议您的windowPeriod小于您的segmentGranularity。

### 仅附加

德鲁伊流媒体摄取仅*附加*，这意味着您无法使用流式摄取来在插入后更新或删除单个记录。如果需要更新或删除单个记录，则需要使用批量重建索引过程。有关 更多详细信息，请参阅*批量摄取*页面。

德鲁伊确实支持有效删除整个时间范围，而无需使用批量重建索引。这可以通过设置保留策略自动完成。

### 担保

Tranquility以尽力而为的设计运作。通过允许您设置副本以及在一段时间内重试失败的推送，它会非常难以保存您的数据，但它并不能保证您的事件只会被处理一次。在某些情况下，它可以删除或复制事件：

- 时间戳超出配置的windowPeriod的事件将被删除。
- 如果您遇到的Druid Middle Manager失败次数超过配置的副本数，则某些部分索引的数据可能会丢失。
- 如果存在阻止与德鲁伊索引服务进行通信的持续性问题，并且在此期间重试策略已用尽，或者该时间段持续时间超过windowPeriod，则会丢弃某些事件。
- 如果存在阻止Tranquility从索引服务接收确认的问题，它将重试批处理，这可能导致重复事件。
- 如果您在Storm或Samza中使用Tranquility，则两种架构的各个部分都至少具有一次设计，并且可能导致重复事件。

在正常操作下，这些风险很小。但如果您需要绝对100％的历史数据保真度，我们建议采用混合/批量流式架构，如下所述。

### 混合批/流

您可以在混合批处理/流式处理架构中组合批处理和流式处理方法。在混合体系结构中，您使用流式方法进行初始摄取，然后以批处理模式（通常每隔几小时或每晚）定期重新获取旧数据。当德鲁伊重新摄取时间范围内的数据时，新数据会自动替换先前摄取的数据。

德鲁伊目前支持的所有流式摄取方法确实在某些故障情况下引入了丢弃或重复消息的可能性，并且批量重新提取消除了历史数据的这种潜在错误源。

如果您因任何原因需要修改数据，批量重新提取还可以选择重新提取数据。

## 文档

可在[此处](https://github.com/druid-io/tranquility/blob/master/README.md)找到宁静文件。

## 组态

可在[此处](https://github.com/druid-io/tranquility/blob/master/docs/configuration.md)找到宁静配置。

Tranquility的tuningConfig可以在[这里](http://static.druid.io/tranquility/api/latest/#com.metamx.tranquility.druid.DruidTuning)找到。

