## 苦工

### 组态

对于Peon配置，请参阅[Peon查询配置](http://druid.io/docs/0.12.3/configuration/index.html#peon-query-configuration)和[其他Peon配置](http://druid.io/docs/0.12.3/configuration/index.html#additional-peon-configuration)。

### HTTP端点

有关Peon支持的API端点列表，请参阅[Peon API参考](http://druid.io/docs/0.12.3/operations/api-reference.html#peon)。

Peons在单个JVM中运行单个任务。MiddleManager负责为运行任务创建Peons。Peons应该很少（如果用于测试目的）自己运行。

### 运行

除非出于开发目的，否则peon应该很少独立于中层经理。

```text
io.druid.cli.Main internal peon <task_file> <status_file>
```

任务文件包含任务JSON对象。状态文件指示将输出任务状态的位置。