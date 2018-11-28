# DumpSegment工具

DumpSegment工具可用于转储段的元数据或内容以进行调试。请注意，转储不一定是段的完全保真转换。特别是，并非包含所有元数据，并且复杂度量值可能不完整。

要运行该工具，请将其指向段目录并提供用于写入输出的文件：

```text
java io.druid.cli.Main tools dump-segment \
  --directory /home/druid/path/to/segment/ \
  --out /home/druid/output.txt
```

### 输出格式

#### 数据转储

默认情况下，或者使用`--dump rows`此工具，该工具将段的行转储为换行符分隔的JSON对象，每行一个对象，使用每列的默认序列化。通常包括所有列，但如果您愿意，可以使用转储将转储限制为特定列`--column name`。

例如，漂亮打印时，一行可能看起来像这样：

```text
{
  "__time": 1442018818771,
  "added": 36,
  "channel": "#en.wikipedia",
  "cityName": null,
  "comment": "added project",
  "count": 1,
  "countryIsoCode": null,
  "countryName": null,
  "deleted": 0,
  "delta": 36,
  "isAnonymous": "false",
  "isMinor": "false",
  "isNew": "false",
  "isRobot": "false",
  "isUnpatrolled": "false",
  "iuser": "00001553",
  "metroCode": null,
  "namespace": "Talk",
  "page": "Talk:Oswald Tilghman",
  "regionIsoCode": null,
  "regionName": null,
  "user": "GELongstreet"
}
```

#### 元数据转储

使用`--dump metadata`此工具转储元数据而不是行。此工具生成的元数据转储格式与[SegmentMetadata查询](http://druid.io/docs/0.12.3/querying/segmentmetadataquery.html)返回的格式相同。

#### 位图转储

使用`--dump bitmaps`此工具转储位图索引而不是行。此工具生成的位图转储仅包括字典编码的字符串列。输出包含一个字段“bitmapSerdeFactory”，用于描述段中使用的位图类型，以及一个字段“bitmaps”，其中包含每列每个值的位图。这些是默认的base64编码，但您也可以将它们作为行号列表转储`--decompress-bitmaps`。

通常包括所有列，但如果您愿意，可以使用转储将转储限制为特定列`--column name`。

样本输出：

```text
{
  "bitmapSerdeFactory": {
    "type": "concise"
  },
  "bitmaps": {
    "isRobot": {
      "false": "//aExfu+Nv3X...",
      "true": "gAl7OoRByQ..."
    }
  }
}
```

### 命令行参数

| 争论                | 描述                                                         | 需要？ |
| ------------------- | ------------------------------------------------------------ | ------ |
| - 目录文件          | 包含段数据的目录。这可以通过从深存储中解压缩“index.zip”来生成。 | 是     |
| - 输出文件          | 要写入的文件，或省略写入stdout的文件。                       | 是     |
| --dump TYPE         | 转储“行”（默认），“元数据”或“位图”                           | 没有   |
| --column columnName | 列包括。为多列指定多次，或省略包括所有列。                   | 没有   |
| --filter json       | JSON编码的[查询过滤器](http://druid.io/docs/0.12.3/querying/filters.html)。省略包括所有行。仅在转储行时使用。 | 没有   |
| --time-ISO8601      | 格式化ISO8601格式的__time列而不是长。仅在转储行时使用。      | 没有   |
| --decompress位图    | 将位图转储为数组而不是base64编码的压缩位图。仅在转储位图时使用。 | 没有   |