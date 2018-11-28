# API参考

此页面记录了每种德鲁伊服务类型的所有API端点。

## 目录

- [共同](http://druid.io/docs/0.12.3/operations/api-reference.html#common)
- [协调员](http://druid.io/docs/0.12.3/operations/api-reference.html#coordinator)
- [霸王](http://druid.io/docs/0.12.3/operations/api-reference.html#overlord)
- [中层经理](http://druid.io/docs/0.12.3/operations/api-reference.html#middlemanager)
- [日工](http://druid.io/docs/0.12.3/operations/api-reference.html#peon)
- [经纪人](http://druid.io/docs/0.12.3/operations/api-reference.html#broker)
- [历史的](http://druid.io/docs/0.12.3/operations/api-reference.html#historical)

## 共同

所有节点都支持以下端点。

### 得到

- `/status`

返回Druid版本，加载的扩展，使用的内存，总内存和有关节点的其他有用信息。

- `/status/health`

始终返回布尔“true”值且具有200 OK响应的端点，对自动运行状况检查很有用。

## 协调员

### 领导

#### 得到

- `/druid/coordinator/v1/leader`

返回集群的当前领导协调者。

- `/druid/coordinator/v1/isLeader`

返回带有字段“leader”的JSON对象，无论是true还是false，指示此服务器是否是群集的当前领导协调者。此外，如果服务器是当前的领导者，则返回HTTP 200，否则返回HTTP 404。如果您只希望在负载均衡器中考虑使用活动的负责人，则适合用作负载均衡器状态检查。

### 分段加载

#### 得到

- `/druid/coordinator/v1/loadstatus`

返回群集中实际加载的段的百分比与应在群集中加载的段的百分比。

- `/druid/coordinator/v1/loadstatus?simple`

返回要加载的段数，直到应在集群中加载的段可用于查询。这不包括复制。

- `/druid/coordinator/v1/loadstatus?full`

返回每个层中要加载的段数，直到应在集群中加载的段全部可用。这包括复制。

- `/druid/coordinator/v1/loadqueue`

返回要为每个历史节点加载和删除的段的ID。

- `/druid/coordinator/v1/loadqueue?simple`

返回要加载和删除的段数，以及每个历史节点的总段加载和丢弃大小（以字节为单位）。

- `/druid/coordinator/v1/loadqueue?full`

返回要为每个历史节点加载和删除的段的序列化JSON。

### 元数据存储信息

#### 得到

- `/druid/coordinator/v1/metadata/datasources`

返回集群中已启用数据源的名称列表。

- `/druid/coordinator/v1/metadata/datasources?includeDisabled`

返回集群中已启用和已禁用数据源的名称列表。

- `/druid/coordinator/v1/metadata/datasources?full`

返回所有已启用数据源的列表，其中包含存储在元数据存储中的有关这些数据源的所有元数据。

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}`

返回存储在元数据存储中的数据源的完整元数据。

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments`

返回存储在元数据存储中的数据源的所有段的列表。

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments?full`

返回数据源的所有段的列表，其中包含存储在元数据存储中的完整段元数据。

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments/{segmentId}`

返回存储在元数据存储中的特定段的完整段元数据。

#### POST

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments`

对于存储在元数据存储中的数据源，返回与任何给定间隔重叠的所有段的列表。请求体是字符串间隔的数组，例如[interval1，interval2，...]，例如[“2012-01-01T00：00：00.000 / 2012-01-03T00：00：00.000”，“2012-01-05T00：00 ：00.000 / 2012-01-07T00：00：00.000" ]

- `/druid/coordinator/v1/metadata/datasources/{dataSourceName}/segments?full`

对于具有存储在元数据存储中的完整段元数据的数据源，返回与任何给定间隔重叠的所有段的列表。请求体是字符串间隔的数组，例如[interval1，interval2，...]，例如[“2012-01-01T00：00：00.000 / 2012-01-03T00：00：00.000”，“2012-01-05T00：00 ：00.000 / 2012-01-07T00：00：00.000" ]

### 数据源信息

#### 得到

- `/druid/coordinator/v1/datasources`

返回在集群中找到的数据源名称列表。

- `/druid/coordinator/v1/datasources?simple`

返回JSON对象的列表，其中包含在群集中找到的数据源的名称和属性。属性包括段计数，总段字节大小，minTime和maxTime。

- `/druid/coordinator/v1/datasources?full`

返回在集群中找到的数据源名称列表，其中包含有关这些数据源的所有元数据。

- `/druid/coordinator/v1/datasources/{dataSourceName}`

返回包含数据源的名称和属性的JSON对象。属性包括段计数，总段字节大小，minTime和maxTime。

- `/druid/coordinator/v1/datasources/{dataSourceName}?full`

返回数据源的完整元数据。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals`

返回一组段间隔。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals?simple`

返回JSON对象的间隔映射，该对象包含段的总字节大小和该间隔的段数。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals?full`

将区段元数据映射的区间映射返回到包含该区间的段的一组服务器名称。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals/{interval}`

返回ISO8601间隔的一组段ID。请注意，{interval}参数由a `_`而不是a `/`（例如，2016-06-27_2016-06-28）分隔。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals/{interval}?simple`

返回指定时间间隔内包含的段间隔映射到JSON对象，该对象包含段的总字节大小和间隔的段数。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals/{interval}?full`

返回指定时间间隔内包含的段间隔映射到段元数据映射到包含间隔段的服务器名称集。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals/{interval}/serverview`

返回指定时间间隔内包含的段间隔的映射，以获取有关包含间隔段的服务器的信息。

- `/druid/coordinator/v1/datasources/{dataSourceName}/segments`

返回集群中数据源的所有段的列表。

- `/druid/coordinator/v1/datasources/{dataSourceName}/segments?full`

返回具有完整段元数据的群集中数据源的所有段的列表。

- `/druid/coordinator/v1/datasources/{dataSourceName}/segments/{segmentId}`

返回群集中特定段的完整段元数据。

- `/druid/coordinator/v1/datasources/{dataSourceName}/tiers`

返回数据源所在的层。

#### POST

- `/druid/coordinator/v1/datasources/{dataSourceName}`

启用所有未被其他人黯然失色的数据源段。

- `/druid/coordinator/v1/datasources/{dataSourceName}/segments/{segmentId}`

启用细分。

#### 删除

- `/druid/coordinator/v1/datasources/{dataSourceName}`

禁用数据源。

- `/druid/coordinator/v1/datasources/{dataSourceName}/intervals/{interval}`
- `@Deprecated. /druid/coordinator/v1/datasources/{dataSourceName}?kill=true&interval={myISO8601Interval}`

为给定的间隔和数据源运行[Kill任务](http://druid.io/docs/0.12.3/ingestion/tasks.html)。

请注意，{interval}参数由a `_`而不是a `/`（例如，2016-06-27_2016-06-28）分隔。

- `/druid/coordinator/v1/datasources/{dataSourceName}/segments/{segmentId}`

禁用细分。

### 保留规则

#### 得到

- `/druid/coordinator/v1/rules`

将所有规则作为JSON对象返回到集群中的所有数据源，包括默认数据源。

- `/druid/coordinator/v1/rules/{dataSourceName}`

返回指定数据源的所有规则。

- `/druid/coordinator/v1/rules/{dataSourceName}?full`

返回指定数据源的所有规则，并包含默认数据源。

- `/druid/coordinator/v1/rules/history?interval=<interval>`

返回所有数据源的规则的审计历史记录。可以通过`druid.audit.manager.auditHistoryMillis`在coordinator runtime.properties中设置（如果未配置，则为1周）来指定interval的缺省值

- `/druid/coordinator/v1/rules/history?count=<n>`

最后返回 所有数据源规则的审计历史记录条目。

- `/druid/coordinator/v1/rules/{dataSourceName}/history?interval=<interval>`

返回指定数据源的规则的审计历史记录。可以通过`druid.audit.manager.auditHistoryMillis`在coordinator runtime.properties中设置（如果未配置，则为1周）来指定interval的缺省值

- `/druid/coordinator/v1/rules/{dataSourceName}/history?count=<n>`

最后返回 指定数据源的规则的审计历史记录的条目。

#### POST

- `/druid/coordinator/v1/rules/{dataSourceName}`

使用JSON格式的规则列表POST以更新规则。

还可以指定用于审核配置更改的可选标头参数。

| 标题参数名称      | 描述                   | 默认 |
| ----------------- | ---------------------- | ---- |
| `X-Druid-Author`  | 作者进行配置更改       | “”   |
| `X-Druid-Comment` | 评论描述正在进行的更改 | “”   |

### 间隔

#### 得到

请注意，{interval}参数由a `_`而不是a `/`（例如，2016-06-27_2016-06-28）分隔。

- `/druid/coordinator/v1/intervals`

返回具有总大小和计数的所有数据源的所有间隔。

- `/druid/coordinator/v1/intervals/{interval}`

返回与给定isointerval相交的所有间隔的聚合总大小和计数。

- `/druid/coordinator/v1/intervals/{interval}?simple`

返回给定isointerval内每个时间间隔的总大小和计数。

- `/druid/coordinator/v1/intervals/{interval}?full`

返回给定isointerval内每个时间间隔的每个数据源的总大小和计数。

## 霸王

### 领导

#### 得到

- `/druid/indexer/v1/leader`

返回集群的当前领导者霸主。如果你有多个霸主，那么任何时候只有一个领先者。其他人待命。

- `/druid/indexer/v1/isLeader`

这将返回一个带有字段“leader”的JSON对象，可以是true或false。此外，如果服务器是当前的领导者，则此调用返回HTTP 200，否则返回HTTP 404。如果您只希望在负载均衡器中考虑使用活动的负责人，则适合用作负载均衡器状态检查。

### 任务

#### 得到

- `/druid/indexer/v1/task/{taskId}/status`

检索任务的状态。

- `/druid/indexer/v1/task/{taskId}/segments`

检索有关任务段的信息。

#### POST

- `/druid/indexer/v1/task`

用于向霸主提交任务和主管规范的端点。返回已提交任务的taskId。

- `druid/indexer/v1/task/{taskId}/shutdown`

关闭任务。

## 中层经理

MiddleManager没有[公共端点](http://druid.io/docs/0.12.3/operations/api-reference.html#common)之外的任何API [端点](http://druid.io/docs/0.12.3/operations/api-reference.html#common)。

## 日工

Peon没有[公共端点](http://druid.io/docs/0.12.3/operations/api-reference.html#common)之外的任何API [端点](http://druid.io/docs/0.12.3/operations/api-reference.html#common)。

## 经纪人

### 数据源信息

#### 得到

- `/druid/v2/datasources`

返回可查询数据源的列表。

- `/druid/v2/datasources/{dataSourceName}`

返回数据源的维度和指标。（可选）您可以提供请求参数“完整”以获取服务时间间隔列表，其中包含为这些时间间隔提供的维度和指标。您还可以显式提供请求参数“interval”以引用特定时间间隔。

如果未指定间隔，则将使用跨越当前时间之前的可配置时间段的默认间隔。此间隔的持续时间以ISO8601格式指定：

druid.query.segmentMetadata.defaultHistory

- `/druid/v2/datasources/{dataSourceName}/dimensions`

返回数据源的维度。

- `/druid/v2/datasources/{dataSourceName}/metrics`

返回数据源的度量标准。

- `/druid/v2/datasources/{dataSourceName}/candidates?intervals={comma-separated-intervals-in-ISO8601-format}&numCandidates={numCandidates}`

返回段信息列表，包括给定数据源和间隔的服务器位置。如果未指定“numCandidates”，则将返回每个间隔的所有服务器。

### 加载状态

#### 得到

- `/druid/broker/v1/loadstatus`

返回一个标志，指示代理是否知道Zookeeper中的所有段。这可用于了解何时可以在重新启动后查询代理节点。

### 查询

#### POST

- `/druid/v2/candidates/`

返回段信息列表，包括给定查询的服务器位置。

## 历史的

### 分段加载

#### 得到

- `/druid/historical/v1/loadstatus`

返回表单的JSON `{"cacheInitialized":<value>}`，其中value是`true`或者`false`指示是否已加载本地缓存中的所有段。这可用于了解重启后历史节点何时可供查询。

- `/druid/historical/v1/readiness`

类似于`/druid/historical/v1/loadstatus`，但不是返回带有标志的JSON，如果已加载本地缓存中的段，则响应200 OK，如果尚未加载，则返回503 SERVICE UNAVAILABLE。