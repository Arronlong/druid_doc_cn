# 索引服务

索引服务是一种高度可用的分布式服务，可运行与索引相关的任务。

索引任务[任务](http://druid.io/docs/0.12.3/ingestion/tasks.html)创建（有时会破坏）德鲁伊[细分](http://druid.io/docs/0.12.3/design/segments.html)。索引服务具有类似主/从的体系结构。

索引服务由三个主要组件组成：可以运行单个任务的[Peon](http://druid.io/docs/0.12.3/design/peons.html)组件，管理peons 的[中间管理器](http://druid.io/docs/0.12.3/design/middlemanager.html)组件，以及管理向中间管理器分配任务的[Overlord](http://druid.io/docs/0.12.3/design/overlord.html)组件。宿主和中层管理人员可以在同一节点上运行，也可以跨多个节点运行，而中间管理人员和中间人经常在同一节点上运行。

使用Overlord服务上的API端点管理任务。有关更多信息，请参阅[Overlord Task API](http://druid.io/docs/0.12.3/operations/api-reference.html#overlord-tasks)。

![索引服务](http://druid.io/docs/img/indexing_service.png)

## 霸王

见[霸王](http://druid.io/docs/0.12.3/design/overlord.html)。

## 中层经理

见[中层经理](http://druid.io/docs/0.12.3/design/middlemanager.html)。

## 苦工

见[Peon](http://druid.io/docs/0.12.3/design/peons.html)。

## 任务

见[任务](http://druid.io/docs/0.12.3/ingestion/tasks.html)。