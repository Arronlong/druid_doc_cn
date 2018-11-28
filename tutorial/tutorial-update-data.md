# 教程：更新现有数据

本教程演示了如何更新现有数据，同时显示覆盖和追加。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)，[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)和[教程：汇总](http://druid.io/docs/0.12.3/tutorials/tutorial-rollup.html)也很有帮助。

## 覆盖

本教程的这一部分将介绍如何覆盖现有的数据间隔。

### 加载初始数据

让我们加载一个初始数据集，我们将覆盖并附加到该数据集。

我们将在本教程中使用的规范位于`examples/updates-init-index.json`。此规范创建`updates-tutorial`从`examples/updates-data.json`输入文件调用的数据源。

我们提交任务：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-init-index.json http://localhost:8090/druid/indexer/v1/task
```

我们有三个包含“动物”维度和“数字”指标的初始行：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-select-sql.json http://localhost:8082/druid/v2/sql
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "animal": "tiger",
    "count": 1,
    "number": 100
  },
  {
    "__time": "2018-01-01T03:01:00.000Z",
    "animal": "aardvark",
    "count": 1,
    "number": 42
  },
  {
    "__time": "2018-01-01T03:01:00.000Z",
    "animal": "giraffe",
    "count": 1,
    "number": 14124
  }
]
```

### 覆盖初始数据

要覆盖此数据，我们可以在相同的时间间隔内提交另一个任务，但输入数据不同。

该`examples/updates-overwrite-index.json`规范将在执行重写`updates-tutorial`的数据源。

请注意，此任务从中读取输入`examples/updates-data2.json`，并`appendToExisting`设置为`false`（表示这是覆盖）。

我们提交任务：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-overwrite-index.json http://localhost:8090/druid/indexer/v1/task
```

当Druid从这个覆盖任务完成加载新段时，“tiger”行现在具有值“lion”，“aardvark”行具有不同的数字，并且“giraffe”行已被替换。更改可能需要几分钟才能生效：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-select-sql.json http://localhost:8082/druid/v2/sql
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "animal": "lion",
    "count": 1,
    "number": 100
  },
  {
    "__time": "2018-01-01T03:01:00.000Z",
    "animal": "aardvark",
    "count": 1,
    "number": 9999
  },
  {
    "__time": "2018-01-01T04:01:00.000Z",
    "animal": "bear",
    "count": 1,
    "number": 111
  }
]
```

## 附加到数据

我们来试试附加数据吧。

该`examples/updates-append-index.json`任务从规格读取输入`examples/updates-data3.json`，并将其数据追加到`updates-tutorial`数据源。请注意，此规范中`appendToExisting`设置`true`了此项。

我们提交任务：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-append-index.json http://localhost:8090/druid/indexer/v1/task
```

加载新数据后，我们可以在“章鱼”之后看到另外两行。请注意，编号为222的新“bear”行尚未与现有的bear-111行汇总，因为新数据保存在单独的段中。这同样适用于两个“狮子”行。

```json
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "animal": "lion",
    "count": 1,
    "number": 100
  },
  {
    "__time": "2018-01-01T03:01:00.000Z",
    "animal": "aardvark",
    "count": 1,
    "number": 9999
  },
  {
    "__time": "2018-01-01T04:01:00.000Z",
    "animal": "bear",
    "count": 1,
    "number": 111
  },
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "animal": "lion",
    "count": 1,
    "number": 300
  },
  {
    "__time": "2018-01-01T04:01:00.000Z",
    "animal": "bear",
    "count": 1,
    "number": 222
  },
  {
    "__time": "2018-01-01T05:01:00.000Z",
    "animal": "mongoose",
    "count": 1,
    "number": 737
  },
  {
    "__time": "2018-01-01T06:01:00.000Z",
    "animal": "snake",
    "count": 1,
    "number": 1234
  },
  {
    "__time": "2018-01-01T07:01:00.000Z",
    "animal": "octopus",
    "count": 1,
    "number": 115
  },
  {
    "__time": "2018-01-01T09:01:00.000Z",
    "animal": "falcon",
    "count": 1,
    "number": 1241
  }
]
```

如果我们运行GroupBy查询而不是a `select *`，我们可以看到单独的“lion”和“bear”行将在查询时组合在一起：

```sql
select __time, animal, SUM("count"), SUM("number") from "updates-tutorial" group by __time, animal;
```

```json
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/updates-groupby-sql.json http://localhost:8082/druid/v2/sql
[
  {
    "__time": "2018-01-01T01:01:00.000Z",
    "animal": "lion",
    "EXPR$2": 2,
    "EXPR$3": 400
  },
  {
    "__time": "2018-01-01T03:01:00.000Z",
    "animal": "aardvark",
    "EXPR$2": 1,
    "EXPR$3": 9999
  },
  {
    "__time": "2018-01-01T04:01:00.000Z",
    "animal": "bear",
    "EXPR$2": 2,
    "EXPR$3": 333
  },
  {
    "__time": "2018-01-01T05:01:00.000Z",
    "animal": "mongoose",
    "EXPR$2": 1,
    "EXPR$3": 737
  },
  {
    "__time": "2018-01-01T06:01:00.000Z",
    "animal": "snake",
    "EXPR$2": 1,
    "EXPR$3": 1234
  },
  {
    "__time": "2018-01-01T07:01:00.000Z",
    "animal": "octopus",
    "EXPR$2": 1,
    "EXPR$3": 115
  },
  {
    "__time": "2018-01-01T09:01:00.000Z",
    "animal": "falcon",
    "EXPR$2": 1,
    "EXPR$3": 1241
  }
]
```