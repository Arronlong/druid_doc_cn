# 教程：查询数据

本教程将演示如何使用Druid的本机查询格式和Druid SQL查询Druid中的数据。

本教程假设您已经完成了4个摄取教程中的一个，因为我们将查询示例Wikipedia编辑数据。

- [教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)
- [教程：从Kafka加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-kafka.html)
- [教程：使用Hadoop加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch-hadoop.html)
- [教程：使用Tranquility加载流数据](http://druid.io/docs/0.12.3/tutorials/tutorial-tranquility.html)

## 本机JSON查询

Druid的本机查询格式以JSON表示。我们在下面包含了一个示例本机TopN查询`examples/wikipedia-top-pages.json`：

```json
{
  "queryType" : "topN",
  "dataSource" : "wikipedia",
  "intervals" : ["2015-09-12/2015-09-13"],
  "granularity" : "all",
  "dimension" : "page",
  "metric" : "count",
  "threshold" : 10,
  "aggregations" : [
    {
      "type" : "count",
      "name" : "count"
    }
  ]
}
```

此查询将检索2015-09-12页面编辑次数最多的10个维基百科页面。

让我们将此查询提交给德鲁伊经纪人：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-top-pages.json http://localhost:8082/druid/v2?pretty
```

您应该看到以下查询结果：

```json
[ {
  "timestamp" : "2015-09-12T00:46:58.771Z",
  "result" : [ {
    "count" : 33,
    "page" : "Wikipedia:Vandalismusmeldung"
  }, {
    "count" : 28,
    "page" : "User:Cyde/List of candidates for speedy deletion/Subpage"
  }, {
    "count" : 27,
    "page" : "Jeremy Corbyn"
  }, {
    "count" : 21,
    "page" : "Wikipedia:Administrators' noticeboard/Incidents"
  }, {
    "count" : 20,
    "page" : "Flavia Pennetta"
  }, {
    "count" : 18,
    "page" : "Total Drama Presents: The Ridonculous Race"
  }, {
    "count" : 18,
    "page" : "User talk:Dudeperson176123"
  }, {
    "count" : 18,
    "page" : "Wikipédia:Le Bistro/12 septembre 2015"
  }, {
    "count" : 17,
    "page" : "Wikipedia:In the news/Candidates"
  }, {
    "count" : 17,
    "page" : "Wikipedia:Requests for page protection"
  } ]
} ]
```

## 德鲁伊SQL查询

德鲁伊还支持用于查询的SQL方言。让我们运行一个SQL查询，它等同于上面显示的本机JSON查询：

```text
SELECT page, COUNT(*) AS Edits FROM wikipedia WHERE "__time" BETWEEN TIMESTAMP '2015-09-12 00:00:00' AND TIMESTAMP '2015-09-13 00:00:00' GROUP BY page ORDER BY Edits DESC LIMIT 10;
```

SQL查询通过HTTP提交为JSON。

### TopN查询示例

教程包中包含一个示例文件，其中包含上面显示的SQL查询`examples/wikipedia-top-pages-sql.json`。让我们将该查询提交给德鲁伊经纪人：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-top-pages-sql.json http://localhost:8082/druid/v2/sql
```

应返回以下结果：

```json
[
  {
    "page": "Wikipedia:Vandalismusmeldung",
    "Edits": 33
  },
  {
    "page": "User:Cyde/List of candidates for speedy deletion/Subpage",
    "Edits": 28
  },
  {
    "page": "Jeremy Corbyn",
    "Edits": 27
  },
  {
    "page": "Wikipedia:Administrators' noticeboard/Incidents",
    "Edits": 21
  },
  {
    "page": "Flavia Pennetta",
    "Edits": 20
  },
  {
    "page": "Total Drama Presents: The Ridonculous Race",
    "Edits": 18
  },
  {
    "page": "User talk:Dudeperson176123",
    "Edits": 18
  },
  {
    "page": "Wikipédia:Le Bistro/12 septembre 2015",
    "Edits": 18
  },
  {
    "page": "Wikipedia:In the news/Candidates",
    "Edits": 17
  },
  {
    "page": "Wikipedia:Requests for page protection",
    "Edits": 17
  }
]
```

### 漂亮的印刷

请注意，德鲁伊SQL不支持漂亮的结果打印。上面的输出使用[jq](https://stedolan.github.io/jq/) JSON处理工具格式化。这些教程中的其他SQL查询实例也将使用`jq`格式。

### 额外的德鲁伊SQL查询

以下部分提供了计划各种类型的本机Druid查询的其他SQL示例。

#### 时间序列

SQL时间序列查询的示例可在以下位置获得`examples/wikipedia-timeseries-sql.json`。此查询检索2015-09-12每小时从页面中删除的总行数。

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-timeseries-sql.json http://localhost:8082/druid/v2/sql
```

应返回以下结果：

```json
[
  {
    "HourTime": "2015-09-12T00:00:00.000Z",
    "LinesDeleted": 1761
  },
  {
    "HourTime": "2015-09-12T01:00:00.000Z",
    "LinesDeleted": 16208
  },
  {
    "HourTime": "2015-09-12T02:00:00.000Z",
    "LinesDeleted": 14543
  },
  {
    "HourTime": "2015-09-12T03:00:00.000Z",
    "LinesDeleted": 13101
  },
  {
    "HourTime": "2015-09-12T04:00:00.000Z",
    "LinesDeleted": 12040
  },
  {
    "HourTime": "2015-09-12T05:00:00.000Z",
    "LinesDeleted": 6399
  },
  {
    "HourTime": "2015-09-12T06:00:00.000Z",
    "LinesDeleted": 9036
  },
  {
    "HourTime": "2015-09-12T07:00:00.000Z",
    "LinesDeleted": 11409
  },
  {
    "HourTime": "2015-09-12T08:00:00.000Z",
    "LinesDeleted": 11616
  },
  {
    "HourTime": "2015-09-12T09:00:00.000Z",
    "LinesDeleted": 17509
  },
  {
    "HourTime": "2015-09-12T10:00:00.000Z",
    "LinesDeleted": 19406
  },
  {
    "HourTime": "2015-09-12T11:00:00.000Z",
    "LinesDeleted": 16284
  },
  {
    "HourTime": "2015-09-12T12:00:00.000Z",
    "LinesDeleted": 18672
  },
  {
    "HourTime": "2015-09-12T13:00:00.000Z",
    "LinesDeleted": 30520
  },
  {
    "HourTime": "2015-09-12T14:00:00.000Z",
    "LinesDeleted": 18025
  },
  {
    "HourTime": "2015-09-12T15:00:00.000Z",
    "LinesDeleted": 26399
  },
  {
    "HourTime": "2015-09-12T16:00:00.000Z",
    "LinesDeleted": 24759
  },
  {
    "HourTime": "2015-09-12T17:00:00.000Z",
    "LinesDeleted": 19634
  },
  {
    "HourTime": "2015-09-12T18:00:00.000Z",
    "LinesDeleted": 17345
  },
  {
    "HourTime": "2015-09-12T19:00:00.000Z",
    "LinesDeleted": 19305
  },
  {
    "HourTime": "2015-09-12T20:00:00.000Z",
    "LinesDeleted": 22265
  },
  {
    "HourTime": "2015-09-12T21:00:00.000Z",
    "LinesDeleted": 16394
  },
  {
    "HourTime": "2015-09-12T22:00:00.000Z",
    "LinesDeleted": 16379
  },
  {
    "HourTime": "2015-09-12T23:00:00.000Z",
    "LinesDeleted": 15289
  }
]
```

#### 通过...分组

可以使用示例SQL GroupBy查询`examples/wikipedia-groupby-sql.json`。此查询将检索2015-09-12中为每个维基百科语言类别（频道）添加的总行数。

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-groupby-sql.json http://localhost:8082/druid/v2/sql
```

应返回以下结果：

```json
[
  {
    "channel": "#en.wikipedia",
    "EXPR$1": 3045299
  },
  {
    "channel": "#it.wikipedia",
    "EXPR$1": 711011
  },
  {
    "channel": "#fr.wikipedia",
    "EXPR$1": 642555
  },
  {
    "channel": "#ru.wikipedia",
    "EXPR$1": 640698
  },
  {
    "channel": "#es.wikipedia",
    "EXPR$1": 634670
  }
]
```

#### 扫描

可以使用示例SQL扫描查询`examples/wikipedia-scan-sql.json`。此查询从数据源检索5个用户页面对。

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-scan-sql.json http://localhost:8082/druid/v2/sql
```

应返回以下结果：

```text
[
  {
    "user": "Thiago89",
    "page": "Campeonato Mundial de Voleibol Femenino Sub-20 de 2015"
  },
  {
    "user": "91.34.200.249",
    "page": "Friede von Schönbrunn"
  },
  {
    "user": "TuHan-Bot",
    "page": "Trĩ vàng"
  },
  {
    "user": "Lowercase sigmabot III",
    "page": "User talk:ErrantX"
  },
  {
    "user": "BattyBot",
    "page": "Hans W. Jung"
  }
]
```

#### 解释计划

通过预先添加`EXPLAIN PLAN FOR`到Druid SQL查询，可以查看SQL查询将计划的本机Druid查询。

解释前面显示的首页查询的示例查询已在以下位置提供`examples/wikipedia-explain-sql.json`：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/wikipedia-explain-top-pages-sql.json http://localhost:8082/druid/v2/sql
```

这将返回以下计划：

```json
[
  {
    "PLAN": "DruidQueryRel(query=[{\"queryType\":\"topN\",\"dataSource\":{\"type\":\"table\",\"name\":\"wikipedia\"},\"virtualColumns\":[],\"dimension\":{\"type\":\"default\",\"dimension\":\"page\",\"outputName\":\"d0\",\"outputType\":\"STRING\"},\"metric\":{\"type\":\"numeric\",\"metric\":\"a0\"},\"threshold\":10,\"intervals\":{\"type\":\"intervals\",\"intervals\":[\"2015-09-12T00:00:00.000Z/2015-09-13T00:00:00.001Z\"]},\"filter\":null,\"granularity\":{\"type\":\"all\"},\"aggregations\":[{\"type\":\"count\",\"name\":\"a0\"}],\"postAggregations\":[],\"context\":{},\"descending\":false}], signature=[{d0:STRING, a0:LONG}])\n"
  }
]
```

## 进一步阅读

该[查询文档](http://druid.io/docs/0.12.3/querying/querying.html)对德鲁伊的本地JSON查询的详细信息。

该[德鲁伊SQL文件](http://druid.io/docs/0.12.3/querying/sql.html)对使用德鲁伊SQL查询的详细信息。