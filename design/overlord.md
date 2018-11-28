## 霸王节点

### 组态

对于Overlord节点配置，请参阅[Overlord配置](http://druid.io/docs/0.12.3/configuration/index.html#overlord)。

### HTTP端点

有关Overlord支持的API端点列表，请参阅[API参考](http://druid.io/docs/0.12.3/operations/api-reference.html#overlord)。

### 概观

霸王节点负责接受任务，协调任务分配，围绕任务创建锁定以及将状态返回给调用者。Overlord可以配置为以两种模式之一运行 - 本地或远程（本地默认）。在本地模式中，霸主还负责创建用于执行任务的工具。在本地模式下运行霸主时，还必须提供所有中间管理器和peon配置。本地模式通常用于简单的工作流程。在远程模式下，霸主和中间管理器在不同的进程中运行，您可以在不同的服务器上运行每个进程。如果您打算将索引服务用作所有德鲁伊索引的单一端点，则建议使用此模式。

### 霸主控制台

霸王控制台可用于查看待处理任务，运行任务，可用工作人员以及最近的工作人员创建和终止。可以通过以下方式访问控制台：

```text
http://<OVERLORD_IP>:<port>/console.html
```

### 黑名单工人

如果工人失败超过阈值的任务，霸主将把这些工人列入黑名单。不超过20％的节点可以列入黑名单。列入黑名单的节点将定期列入白名单。

以下vairables可用于设置阈值和黑名单超时。

```text
druid.indexer.runner.maxRetriesBeforeBlacklist
druid.indexer.runner.workerBlackListBackoffTime
druid.indexer.runner.workerBlackListCleanupPeriod
druid.indexer.runner.maxPercentageBlacklistWorkers
```

### 自动缩放

当前使用的自动调节机制与我们的部署基础架构紧密耦合，但框架应该适用于其他实现。我们对现有机制的新实现或扩展非常开放。在我们自己的部署中，中间管理器节点是Amazon AWS EC2节点，它们被配置为在[galaxy](https://github.com/ning/galaxy)环境中注册自己。

如果启用了自动调节，则当任务处于暂挂状态太长时间时，可能会添加新的中间管理器。如果中间管理人员在一段时间内没有执行任何任务，他们可能会被终止。