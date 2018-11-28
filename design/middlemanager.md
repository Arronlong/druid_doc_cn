## 中间管理器节点

### 组态

对于Middlemanager节点配置，请参阅[索引服务配置](http://druid.io/docs/0.12.3/configuration/index.html#middlemanager-and-peons)。

### HTTP端点

有关MiddleManager支持的API端点列表，请参阅[API参考](http://druid.io/docs/0.12.3/operations/api-reference.html#middlemanager)。

### 概观

中间管理器节点是执行提交的任务的工作节点。Middle Managers将任务转发给在不同JVM中运行的peons。我们为任务分别使用JVM的原因是资源和日志隔离。每个[Peon](http://druid.io/docs/0.12.3/design/peons.html)一次只能运行一个任务，但是，中层经理可能有多个人。

### 运行

```text
io.druid.cli.Main server middleManager
```