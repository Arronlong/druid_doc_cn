# 查询

查询使用HTTP REST风格的请求查询的一系列节点（做[经纪人](http://druid.io/docs/0.12.3/design/broker.html)， [历史](http://druid.io/docs/0.12.3/design/historical.html)。[苦工](http://druid.io/docs/0.12.3/design/peons.html)正在运行的流食入的任务也可以接受查询）。查询以JSON表示，并且每个节点类型都公开相同的REST查询接口。对于正常的德鲁伊操作，应该向代理节点发出查询。查询可以像这样发布到可查询节点 -

```bash
 curl -X POST '<queryable_host>:<port>/druid/v2/?pretty' -H 'Content-Type:application/json' -d @<query_json_file>
```

Druid的原生查询语言是基于HTTP的JSON，尽管社区的许多成员已经使用其他语言提供了不同的 [客户端库](http://druid.io/docs/0.12.3/development/libraries.html)来查询德鲁伊。

德鲁伊的原生查询相对较低，与内部执行计算的方式密切相关。德鲁伊查询被设计为轻量级并且非常快速地完成。这意味着，对于更复杂的分析或构建更复杂的可视化，可能需要多个德鲁伊查询。

## 可用查询

德鲁伊有各种用例的查询类型。查询由各种JSON属性组成，而Druid针对不同的用例具有不同类型的查询。各种查询类型的文档描述了可以设置的所有JSON属性。

### 聚合查询

- [时间序列](http://druid.io/docs/0.12.3/querying/timeseriesquery.html)
- [TOPN](http://druid.io/docs/0.12.3/querying/topnquery.html)
- [通过...分组](http://druid.io/docs/0.12.3/querying/groupbyquery.html)

### 元数据查询

- [时间界限](http://druid.io/docs/0.12.3/querying/timeboundaryquery.html)
- [细分元数据](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html)
- [数据源元数据](http://druid.io/docs/0.12.3/querying/datasourcemetadataquery.html)

### 搜索查询

- [搜索](http://druid.io/docs/0.12.3/querying/searchquery.html)

## 我应该使用哪个查询？

Where possible, we recommend using [Timeseries](http://druid.io/docs/0.12.3/querying/querying.html) and [TopN](http://druid.io/docs/0.12.3/querying/querying.html) queries instead of [GroupBy](http://druid.io/docs/0.12.3/querying/querying.html). GroupBy is the most flexible Druid query, but also has the poorest performance. Timeseries are significantly faster than groupBy queries for aggregations that don't require grouping over dimensions. For grouping and sorting over a single dimension, topN queries are much more optimized than groupBys.

## Query Cancellation

Queries can be cancelled explicitly using their unique identifier. If the query identifier is set at the time of query, or is otherwise known, the following endpoint can be used on the broker or router to cancel the query.

```sh
DELETE /druid/v2/{queryId}
```

For example, if the query ID is `abc123`, the query can be cancelled as follows:

```sh
curl -X DELETE "http://host:port/druid/v2/abc123"
```

## Query Errors

If a query fails, you will get an HTTP 500 response containing a JSON object with the following structure:

```json
{
  "error" : "Query timeout",
  "errorMessage" : "Timeout waiting for task.",
  "errorClass" : "java.util.concurrent.TimeoutException",
  "host" : "druid1.example.com:8083"
}
```

The fields in the response are:

| field        | description                                                  |
| ------------ | ------------------------------------------------------------ |
| error        | A well-defined error code (see below).                       |
| errorMessage | A free-form message with more information about the error. May be null. |
| errorClass   | The class of the exception that caused this error. May be null. |
| host         | The host on which this error occurred. May be null.          |

Possible codes for the *error* field include:

| code                      | description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `Query timeout`           | The query timed out.                                         |
| `Query interrupted`       | The query was interrupted, possibly due to JVM shutdown.     |
| `Query cancelled`         | The query was cancelled through the query cancellation API.  |
| `Resource limit exceeded` | The query exceeded a configured resource limit (e.g. groupBy maxResults). |
| `Unknown exception`       | 发生了一些其他异常。检查errorMessage和errorClass以获取详细信息，但请记住，这些字段的内容是自由格式的，并且可能会在发行版之间发生变化。 |