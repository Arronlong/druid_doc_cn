# 任务概述

任务在中间管理器上运行，并始终在单个数据源上运行。

使用POST请求向Overlord提交任务。有关API详细信息，请参阅[Overlord Task API](http://druid.io/docs/0.12.3/operations/api-reference.html#overlord-tasks)。

有几种不同类型的任务。

## 细分创建任务

### 本地批量索引任务

请参阅本[机批量提取](http://druid.io/docs/0.12.3/ingestion/native-batch.html)。

### Hadoop索引任务

请参阅[Hadoop批量提取](http://druid.io/docs/0.12.3/ingestion/hadoop.html)。

### 卡夫卡索引任务

Kafka索引任务由Kafka Supervisor自动创建，负责从Kafka流中提取数据。这些任务不是由用户直接创建/提交的。有关详细信息，请参阅[Kafka Indexing Service](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html)。

### 流推任务（Tranquility）

Tranquility Server使用[EventReceiverFirehose](http://druid.io/docs/0.12.3/ingestion/firehose.html#eventreceiverfirehose)自动创建通过HTTP接收事件的“实时”任务。这些任务不是由用户直接创建/提交的。有关详细信息，请参阅[Tranquility Stream Push](http://druid.io/docs/0.12.3/ingestion/stream-push.html)。

## 压实任务

压缩任务合并给定间隔的所有段。有关详细信息，请参阅[压缩](http://druid.io/docs/0.12.3/ingestion/compaction.html)。

## 细分合并任务

附加任务，合并任务和相同时间间隔合并任务的文档已移至[其他任务](http://druid.io/docs/0.12.3/ingestion/misc-tasks.html)。

## 杀死任务

Kill任务删除有关段的所有信息并将其从深层存储中删除。

有关详细信息，请参阅[删除数据](http://druid.io/docs/0.12.3/ingestion/delete-data.html)。

## 杂项。任务

请参阅[其他任务](http://druid.io/docs/0.12.3/ingestion/misc-tasks.html)。

## 任务锁定和优先级

请参阅[任务锁定和优先级](http://druid.io/docs/0.12.3/ingestion/locking-and-priority.html)。