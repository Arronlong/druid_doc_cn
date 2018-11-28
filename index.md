# 什么是Druid？

Druid是一个专为大型数据集上的高性能切片和骰子分析（“ [OLAP](http://en.wikipedia.org/wiki/Online_analytical_processing) ”风格）而设计的数据存储。Druid最常用作为GUI分析应用程序提供动力的数据存储，或者用作需要快速聚合的高度并发API的后端。Druid的常见应用领域包括：

- 点击流分析
- 网络流量分析
- 服务器指标存储
- 应用性能指标
- 数字营销分析
- 商业智能/ OLAP

Druid的主要特点是：

1. **列式存储格式。**Druid使用面向列的存储，这意味着它只需要加载特定查询所需的精确列。这为只查看几列的查询提供了巨大的速度提升。此外，每列都针对其特定数据类型进行了优化，支持快速扫描和聚合。
2. **可扩展的分布式系** Druid通常部署在数十到数百台服务器的集群中，并且可以提供数百万条记录/秒的摄取率，保留数万亿条记录，以及亚秒级到几秒钟的查询延迟。
3. **大规模并行处理。**Druid可以在整个集群中并行处理查询。
4. **Realtime or batch ingestion.** Druid can ingest data either realtime (ingested data is immediately available for querying) or in batches.
5. **Self-healing, self-balancing, easy to operate.** As an operator, to scale the cluster out or in, simply add or remove servers and the cluster will rebalance itself automatically, in the background, without any downtime. If any Druid servers fail, the system will automatically route around the damage until those servers can be replaced. Druid is designed to run 24/7 with no need for planned downtimes for any reason, including configuration changes and software updates.
6. **云原生，容错的架构，不会丢失数据。**一旦Druid摄取了您的数据，副本就会安全地存储在[深层存储](http://druid.io/docs/0.12.3/design/#deep-storage)（通常是云存储，HDFS或共享文件系统）中。即使每个Druid服务器都出现故障，您的数据也可以从深层存储中恢复。对于仅影响少数Druid服务器的更有限的故障，复制可确保在系统恢复时仍可进行查询。
7. **用于快速过滤的索引。**Druid使用[CONCISE](https://arxiv.org/pdf/1004.0403)或 [Roaring](https://roaringbitmap.org/)压缩位图索引来创建索引，这些索引可以跨多个列快速过滤和搜索。
8. **近似算法。**Druid包括用于近似计数 - 不同，近似排序以及近似直方图和分位数的计算的算法。这些算法提供有限的内存使用，并且通常比精确计算快得多。对于精确度比速度更重要的情况，Druid还提供精确计数 - 不同且精确的排名。
9. **摄取时自动汇总。**Druid可选择在摄取时支持数据汇总。此摘要部分预先汇总了您的数据，可以节省大量成本并提高性能。

# 我什么时候应该使用Druid？

如果您的用例符合以下几个描述符，Druid可能是一个不错的选择：

- 插入率非常高，但更新不常见。
- 您的大多数查询都是聚合和报告查询（“分组依据”查询）。您可能还有搜索和扫描查询。
- 您将查询延迟定位为100毫秒到几秒钟。
- 您的数据有一个时间组件（Druid包括与时间特别相关的优化和设计选择）。
- 您可能有多个表，但每个查询只能访问一个大的分布式表。查询可能会触发多个较小的“查找”表。
- 您有高基数数据列（例如URL，用户ID），需要对它们进行快速计数和排名。
- 您希望从Kafka，HDFS，平面文件或对象存储（如Amazon S3）加载数据。

情况下，您可能会*不*希望使用Druid包括：

- 您需要使用主键对*现有*记录进行低延迟更新。Druid支持流式插入，但不支持流式更新（使用后台批处理作业进行更新）。
- 您正在构建一个脱机报告系统，其中查询延迟不是很重要。
- 你想做“大”连接（将一个大事实表连接到另一个大事实表）。

# 建筑

Druid拥有一个多进程，分布式架构，旨在实现云友好且易于操作。每个Druid流程类型都可以独立配置和扩展，为您的群集提供最大的灵活性。此设计还提供增强的容错能力：一个组件的中断不会立即影响其他组件。

Druid的过程类型是：

- [**历史**](http://druid.io/docs/0.12.3/design/historical.html)进程是处理存储和查询“历史”数据（包括系统中已经存在足够长时间以提交的任何流数据）的主要工具。历史进程从深层存储中下载段并响应有关这些段的查询。他们不接受写作。
- [**MiddleManager**](http://druid.io/docs/0.12.3/design/middlemanager.html)进程处理将新数据提取到集群中。他们负责从外部数据源阅读并发布新的Druid片段。
- [**代理**](http://druid.io/docs/0.12.3/design/broker.html)进程从外部客户端接收查询，并将这些查询转发给Historicals和MiddleManagers。当Brokers从这些子查询中收到结果时，它们会合并这些结果并将它们返回给调用者。最终用户通常会查询Brokers，而不是直接查询Historicals或MiddleManagers。
- [**协调员**](http://druid.io/docs/0.12.3/design/coordinator.html)流程监视历史流程。他们负责将段分配给特定服务器，并确保段在历史记录之间保持平衡。
- [**霸王**](http://druid.io/docs/0.12.3/design/overlord.html)进程监视MiddleManager进程，并且是数据摄入Druid的控制器。他们负责将提取任务分配给MiddleManagers并协调段发布。
- [**路由器**](http://druid.io/docs/0.12.3/development/router.html)进程是*可选的*进程，在Druid Brokers，Overlords和Coordinator之前提供统一的API网关。它们是可选的，因为您也可以直接联系Druid经纪人，宿主和协调员。

Druid进程可以单独部署（每个物理服务器，虚拟服务器或容器一个），也可以在共享服务器上共存。一个常见的托管计划是三种类型的计划：

1. “数据”服务器运行历史和中间管理程序。
2. “查询”服务器运行Broker和（可选）路由器进程。
3. “Master”服务器运行Coordinator和Overlord进程。他们也可以运行ZooKeeper。

除了这些流程类型，Druid还有三个外部依赖项。这些旨在能够利用现有的基础设施。

- [**深度存储**](http://druid.io/docs/0.12.3/design/#deep-storage)，每个Druid服务器都可以访问共享文件存储。这通常是分布式对象存储，如S3或HDFS，或网络安装的文件系统。Druid使用它来存储已被摄入系统的任何数据。
- [**元数据存储**](http://druid.io/docs/0.12.3/design/#metadata-storage)，共享元数据存储。这通常是传统的RDBMS，如PostgreSQL或MySQL。
- [**ZooKeeper**](http://druid.io/docs/0.12.3/design/#zookeeper)用于内部服务发现，协调和领导者选举。

这种架构背后的想法是使Druid集群在生产中大规模运作变得简单。例如，深度存储和元数据存储与集群其余部分的分离意味着Druid进程具有极强的容错能力：即使每个Druid服务器都出现故障，您仍然可以从存储在深存储中的数据和元数据中重新启动集群。商店。

下图显示了查询和数据如何流经此体系结构：

![img](http://druid.io/docs/img/druid-architecture.png)

# 数据源和细分

Druid数据存储在“数据源”中，类似于传统RDBMS中的表。每个数据源按时间划分，并可选择进一步按其他属性划分。每个时间范围都称为“块”（例如，如果您的数据源按天分区，则为一天）。在块中，数据被划分为一个或多个“段”。每个段都是单个文件，通常包含多达几百万行数据。由于段被组织成时间块，因此将段视为生活在时间轴上有时会有所帮助，如下所示：

![img](http://druid.io/docs/img/druid-timeline.png)

数据源可能只有几个段，最多可达数十万甚至数百万个段。每个段都开始在MiddleManager上创建生命，并且在那时，它是可变的和未提交的。分段构建过程包括以下步骤，旨在生成紧凑的数据文件并支持快速查询：

- 转换为柱状格式
- 使用位图索引进行索引
- 使用各种算法进行压缩
  - 字符串编码，使用String列的id存储最小化
  - 位图索引的位图压缩
  - 所有列的类型感知压缩

定期提交和发布细分。此时，它们被写入[深层存储](http://druid.io/docs/0.12.3/design/#deep-storage)，变为不可变，并从MiddleManagers迁移到历史进程（有关详细信息，请参阅上面的[体系结构](http://druid.io/docs/0.12.3/design/#architecture)）。关于该段的条目也被写入[元数据存储](http://druid.io/docs/0.12.3/design/#metadata-storage)。此条目是有关该段的自描述元数据，包括段的架构，其大小以及它在深层存储上的位置。这些条目是协调器用于了解群集上*应该*有哪些数据的用途。

# 查询处理

查询首先进入Broker，Broker将识别哪些段具有可能与该查询相关的数据。段列表始终按时间进行修剪，也可能会被其他属性修剪，具体取决于数据源的分区方式。然后，代理将识别哪些Historicals和MiddleManagers正在为这些段提供服务，并向每个进程发送重写的子查询。Historical / MiddleManager进程将接受查询，处理它们并返回结果。Broker接收结果并将它们合并在一起以获得最终答案，并将其返回给原始调用者。

经纪人修剪是Druid限制每个查询必须扫描的数据量的重要方式，但这不是唯一的方法。对于比Broker可用于修剪更细粒度的过滤器，每个段内的索引结构允许Druid在查看任何数据行之前确定哪些（如果有）行匹配过滤器集。一旦Druid知道哪些行与特定查询匹配，它只会访问该查询所需的特定列。在这些列中，Druid可以从一行跳到另一行，避免读取与查询过滤器不匹配的数据。

所以Druid使用三种不同的技术来最大化查询性能：

- 修剪为每个查询访问哪些段。
- 在每个段内，使用索引来标识必须访问的行。
- 在每个段中，仅读取与特定查询相关的特定行和列。

# 外部依赖

## 深度存储

Druid仅将深度存储用作数据的备份，并将其作为在Druid进程之间在后台传输数据的一种方式。要响应查询，历史进程不会从深层存储读取，而是在提供任何查询之前从其本地磁盘读取预取的段。这意味着Druid在查询期间永远不需要访问深层存储，从而帮助它提供最佳的查询延迟。这也意味着您必须在深度存储和整个历史进程中拥有足够的磁盘空间来存储您计划加载的数据。

有关更多详细信息，请参阅[深层存储依赖性](http://druid.io/docs/0.12.3/dependencies/deep-storage.html)。

## 元数据存储

元数据存储保存各种系统元数据，例如段可用性信息和任务信息。

有关更多详细信息，请参阅[元数据存储依赖性](http://druid.io/docs/0.12.3/dependencies/metadata-storage.html)

## 动物园管理员

Druid使用[ZooKeeper](http://zookeeper.apache.org/)（ZK）来管理当前的集群状态。

有关更多详细信息，请参阅[Zookeeper依赖项](http://druid.io/docs/0.12.3/dependencies/zookeeper.html)。