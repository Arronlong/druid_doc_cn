# 在德鲁伊上发展

德鲁伊的代码库由几个主要组件组成。对于有兴趣学习代码的开发人员，本文档提供了构成Druid的主要组件的高级概述以及从学习代码开始的相关类。

## 存储格式

德鲁伊中的数据以称为[段](http://druid.io/docs/0.12.3/design/segments.html)的自定义列格式存储。细分由不同类型的列组成。`Column.java`扩展它的类是查看存储格式的好地方。

## 细分创建

摄取原始数据`IncrementalIndex.java`，并在其中创建分段`IndexMerger.java`。

## 存储引擎

德鲁伊段是内存映射的，`IndexIO.java`用于查询。

## 查询引擎

可以在Query *类中找到与Druid查询相关的大多数逻辑。Druid利用查询运行器来运行查询。查询运行程序通常嵌入其他查询运行程序，每个查询运行程序添加一层逻辑。跟踪查询逻辑的一个很好的起点是从头开始`QueryResource.java`。

## 协调

历史节点的大多数协调逻辑都在德鲁伊协调员身上。这里的出发点是`DruidCoordinator.java`。
（实时）摄取的大多数协调逻辑都在德鲁伊索引服务中。这里的出发点是`OverlordResource.java`。

## 实时摄取

德鲁伊通过`FirehoseFactory.java`课程加载数据。Firehoses经常包裹其他firehoses，其中，类似于
查询运行器的设计，每个firehose添加一层逻辑。大部分核心管理逻辑都`RealtimeManager.java`存在，并且存在持久性和切换逻辑`RealtimePlumber.java`。

## 基于Hadoop的批量摄取

两个主要的Hadoop索引类`HadoopDruidDetermineConfigurationJob.java`用于确定要创建多少德鲁伊段，并`HadoopDruidIndexerJob.java`创建德鲁伊段。

在未来的某个时刻，我们可能会将Hadoop摄取代码移出核心德鲁伊。

## 内部UI

德鲁伊目前有两个内部UI。一个是协调员，一个是霸王。

在未来的某个时刻，我们可能会将内部UI代码移出核心德鲁伊。

## 客户端库

我们欢迎新客户图书馆与德鲁伊互动的贡献。查看现有客户端 [库](http://druid.io/docs/0.12.3/development/libraries.html)的客户端库。