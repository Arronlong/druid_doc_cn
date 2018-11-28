# 聚合粒度

粒度字段确定数据如何在时间维度上进行分段，或者如何按小时，天，分钟等进行汇总。

它既可以指定为简单粒度的字符串，也可以指定为任意粒度的对象。

### 简单的粒度

简单粒度按其UTC时间指定为字符串和桶时间戳（例如，从00:00 UTC开始的天数）。

支持粒度字符串是：`all`，`none`，`second`，`minute`，`fifteen_minute`，`thirty_minute`，`hour`，`day`，`week`，`month`，`quarter`和`year`。

- `all` 将所有东西都装进一个桶里
- `none`不存储数据（它实际上使用索引的粒度 - 这里最小值是`none`指毫秒粒度）。使用`none`在[TimeseriesQuery](http://druid.io/docs/0.12.3/querying/timeseriesquery.html)目前也不建议（该系统将尝试生成0值全部毫秒不存在的，这往往是很多）。

#### 例：

假设您有下面存储在Druid中的数据，具有毫秒的摄取粒度，

```json
{"timestamp": "2013-08-31T01:02:33Z", "page": "AAA", "language" : "en"}
{"timestamp": "2013-09-01T01:02:33Z", "page": "BBB", "language" : "en"}
{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}
{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}
```

在提交具有`hour`粒度的groupBy查询后，

```json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":"hour",
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "2000-01-01T00:00Z/3000-01-01T00:00Z"
   ]
}
```

你会得到

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T01:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T01:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T23:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T03:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

请注意，所有空桶都将被丢弃。

如果您将粒度更改为`day`，您将获得

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

如果将粒度更改为`none`，则将获得与将其设置为摄取粒度相同的结果。

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T01:02:33.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T01:02:33.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T23:32:45.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T03:32:45.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

具有小于摄取粒度的查询粒度没有意义，因为关于该较小粒度的信息不存在于索引数据中。因此，如果查询粒度小于摄取粒度，那么德鲁伊产生的结果等同于将查询粒度设置为摄取粒度。见`queryGranularity`在[食入规格](http://druid.io/docs/0.12.3/ingestion/ingestion-spec.html#granularityspec)。

如果您将粒度更改为`all`，您将获得在1个桶中聚合的所有内容，

```json
[ {
  "version" : "v1",
  "timestamp" : "2000-01-01T00:00:00.000Z",
  "event" : {
    "count" : 4,
    "language" : "en"
  }
} ]
```

### 持续时间粒度

持续时间粒度指定为精确持续时间（以毫秒为单位），时间戳以UTC格式返回。持续时间粒度值以毫秒为单位。

它们还支持指定可选原点，该原点定义从何时开始计算时间桶（默认为1970-01-01T00：00：00Z）。

```javascript
{"type": "duration", "duration": 7200000}
```

这每2个小时就有一次。

```javascript
{"type": "duration", "duration": 3600000, "origin": "2012-01-01T00:30:00Z"}
```

在半小时内每小时都有一个小块。

#### 例：

在提交24小时持续时间的groupBy查询后，重用上一示例中的数据，

```json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":{"type": "duration", "duration": "86400000"},
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "2000-01-01T00:00Z/3000-01-01T00:00Z"
   ]
}
```

你会得到

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

如果您将粒度的原点设置为`2012-01-01T00:30:00Z`，

```javascript
   "granularity":{"type": "duration", "duration": "86400000", "origin":"2012-01-01T00:30:00Z"}
```

你会得到

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

请注意，每个存储桶的时间戳从第30分钟开始。

### 期间粒度

期间粒度被指定为[ISO8601](https://en.wikipedia.org/wiki/ISO_8601)格式的年，月，周，小时，分钟和秒（例如P2W，P3M，PT1H30M，PT0.750S）的任意周期组合。它们支持指定确定周期边界开始位置的时区以及返回时间戳的时区。默认情况下，年份从1月1日开始，月份从月初开始，周数从星期一开始，除非指定了原点。

时区是可选的（默认为UTC）。Origin是可选的（在给定时区内默认为1970-01-01T00：00：00）。

```javascript
{"type": "period", "period": "P2D", "timeZone": "America/Los_Angeles"}
```

这将在太平洋时区以两天为一块。

```javascript
{"type": "period", "period": "P3M", "timeZone": "America/Los_Angeles", "origin": "2012-02-01T00:00:00-08:00"}
```

这将在太平洋时区推出3个月的季度，其中三个月的季度定义为从2月开始。

#### 例

重复使用上一个示例中的数据，如果您在太平洋时区提交1天的groupBy查询，

```json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":{"type": "period", "period": "P1D", "timeZone": "America/Los_Angeles"},
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "1999-12-31T16:00:00.000-08:00/2999-12-31T16:00:00.000-08:00"
   ]
}
```

你会得到

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-30T00:00:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000-07:00",
  "event" : {
    "count" : 2,
    "language" : "en"
  }
} ]
```

请注意，每个存储桶的时间戳已转换为太平洋时间。行`{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}`和 `{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}`放在同一个桶中，因为它们在太平洋时间的同一天。

另请注意，`intervals`in groupBy查询不会转换为指定的时区，粒度指定的时区仅应用于查询结果。

如果您将粒度的原点设置为`1970-01-01T20:30:00-08:00`，

```javascript
   "granularity":{"type": "period", "period": "P1D", "timeZone": "America/Los_Angeles", "origin": "1970-01-01T20:30:00-08:00"}
```

你会得到

```json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-29T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-08-30T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

请注意，`origin`您指定的时区与时区无关，它仅用作查找第一个粒度存储区的起点。在这种情况下，Row `{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}`和`{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}` 不在同一个桶中。

#### 支持的时区

时区支持由[Joda Time库](http://www.joda.org/)提供，该[库](http://www.joda.org/)使用标准的IANA时区。请参阅[Joda Time支持的时区](http://joda-time.sourceforge.net/timezones.html)。