# 保留或自动删除数据

协调器节点使用规则来确定应从群集加载或删除哪些数据。规则用于数据保留和查询执行，并在协调器控制台（http：// coordinator_ip：port）上设置。

有三种类型的规则，即加载规则，丢弃规则和广播规则。加载规则指示应如何将段分配给不同的历史节点层以及每个层中应存在多少个段的副本。删除规则指示何时应从群集中完全删除段。最后，广播规则指示不同数据源的片段应如何共存于历史节点中。

协调器从元数据存储加载一组规则。规则可以特定于某个数据源和/或可以配置默认规则集。规则按顺序阅读，因此规则的排序很重要。协调器将遍历所有可用的段，并将每个段与适用的第一个规则进行匹配。每个细分可能只匹配一个规则。

注意：建议使用协调器控制台配置规则。但是，协调器节点确实具有HTTP端点，以编程方式配置规则。

更新规则时，可能不会在协调器下次运行时反映更改。这将在不久的将来修复。

## 加载规则

加载规则指示服务器层中应存在多少个段的副本。

### 永远负载规则

永久加载规则的形式如下：

```json
{
  "type" : "loadForever",  
  "tieredReplicants": {
    "hot": 1,
    "_default_tier" : 1
  }
}
```

- `type` - 这应该始终是“loadForever”
- `tieredReplicants` - JSON对象，其中键是层名称，值是该层的副本数。

### 区间加载规则

间隔加载规则的形式如下：

```json
{
  "type" : "loadByInterval",
  "interval": "2012-01-01/2013-01-01",
  "tieredReplicants": {
    "hot": 1,
    "_default_tier" : 1
  }
}
```

- `type` - 这应该始终是“loadByInterval”
- `interval` - 表示ISO-8601间隔的JSON对象
- `tieredReplicants` - JSON对象，其中键是层名称，值是该层的副本数。

### 期间负荷规则

期间加载规则的形式如下：

```json
{
  "type" : "loadByPeriod",
  "period" : "P1M",
  "tieredReplicants": {
      "hot": 1,
      "_default_tier" : 1
  }
}
```

- `type` - 这应该始终是“loadByPeriod”
- `period` - 表示ISO-8601周期的JSON对象
- `tieredReplicants` - JSON对象，其中键是层名称，值是该层的副本数。

段的间隔将与指定的时段进行比较。如果周期与间隔重叠，则规则匹配。

## 删除规则

删除规则指示何时应从群集中删除段。

### 永远放弃规则

永远丢弃规则的形式如下：

```json
{
  "type" : "dropForever"  
}
```

- `type` - 这应该总是“dropForever”

符合此规则的所有段都将从群集中删除。

### 区间丢弃规则

间隔丢弃规则的形式如下：

```json
{
  "type" : "dropByInterval",
  "interval" : "2012-01-01/2013-01-01"
}
```

- `type` - 这应该总是“dropByInterval”
- `interval` - 表示ISO-8601周期的JSON对象

如果间隔包含段的间隔，则删除段。

### 期间掉落规则

期间下降规则的形式如下：

```json
{
  "type" : "dropByPeriod",
  "period" : "P1M"
}
```

- `type` - 这应该总是“dropByPeriod”
- `period` - 表示ISO-8601周期的JSON对象

段的间隔将与指定的时段进行比较。这段时间是从过去的某个时间到现在的时间。如果句点包含间隔，则规则匹配。

## 广播规则

广播规则指示不同数据源的段应如何共存于历史节点中。一旦为数据源配置了广播规则，就将数据源的所有段广播到保存共同定位数据源的*任何段*的服务器。

### 永远的广播规则

永久广播规则的形式如下：

```json
{
  "type" : "broadcastForever",
  "colocatedDataSources" : [ "target_source1", "target_source2" ]
}
```

- `type` - 这应该始终是“broadcastForever”
- `colocatedDataSources` - 包含要共存的数据源名称的JSON列表。`null`空列表表示广播到集群中的每个节点。

### 区间广播规则

间隔广播规则的形式如下：

```json
{
  "type" : "broadcastByInterval",
  "colocatedDataSources" : [ "target_source1", "target_source2" ],
  "interval" : "2012-01-01/2013-01-01"
}
```

- `type` - 这应该始终是“broadcastByInterval”
- `colocatedDataSources` - 包含要共存的数据源名称的JSON列表。`null`空列表表示广播到集群中的每个节点。
- `interval` - 表示ISO-8601周期的JSON对象。仅广播区间的片段。

### 期间广播规则

期间广播规则的形式如下：

```json
{
  "type" : "broadcastByPeriod",
  "colocatedDataSources" : [ "target_source1", "target_source2" ],
  "period" : "P1M"
}
```

- `type` - 这应该始终是“broadcastByPeriod”
- `colocatedDataSources` - 包含要共存的数据源名称的JSON列表。`null`空列表表示广播到集群中的每个节点。
- `period` - 表示ISO-8601周期的JSON对象

段的间隔将与指定的时段进行比较。这段时间是从过去的某个时间到现在的时间。如果句点包含间隔，则规则匹配。

广播规则不保证数据源的各个段始终位于同一位置，因为共置数据源的段不是以原子方式加载在一起的。如果要始终将某些数据源的段放在一起，建议将colocatedDataSources留空。

# 永久删除数据

Druid可以从群集中完全删除数据，擦除元数据存储条目，并从深存储中删除标记为未使用的任何段的数据（通过规则从群集中删除的段始终标记为未使用）。您可以提交一个[击杀任务](http://druid.io/docs/0.12.3/ingestion/tasks.html)的[索引服务](http://druid.io/docs/0.12.3/design/indexing-service.html)来做到这一点。

# 重新下载丢弃的数据

仅使用规则无法重新加载从德鲁伊群集中删除的数据。要在德鲁伊重新加载已删除的数据，您必须首先设置保留期（即将保留期从1个月更改为2个月），然后在德鲁伊协调器控制台或德鲁伊协调器端点中启用数据源。