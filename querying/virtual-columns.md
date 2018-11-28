# 虚拟列

虚拟列是在查询期间从一组列创建的可查询列“视图”。

虽然虚拟列始终将自身显示为单个列，但虚拟列可能会从多个基础列中进行绘制。

虚拟列可用作维度或聚合器的输入。

每个Druid查询都可以接受虚拟列列表作为参数。以下扫描查询作为示例提供：

```text
{
 "queryType": "scan",
 "dataSource": "page_data",
 "columns":[],
 "virtualColumns": [
    {
      "type": "expression",
      "name": "fooPage",
      "expression": "concat('foo' + page)",
      "outputType": "STRING"
    },
    {
      "type": "expression",
      "name": "tripleWordCount",
      "expression": "wordCount * 3",
      "outputType": "LONG"
    }
  ],
 "intervals": [
   "2013-01-01/2019-01-02"
 ] 
}
```

## 虚拟列类型

### 表达式虚拟列

表达式虚拟列具有以下语法：

```text
{
  "type": "expression",
  "name": <name of the virtual column>,
  "expression": <row expression>,
  "outputType": <output value type of expression>
}
```

| 属性     | 描述                                                         | 需要？          |
| -------- | ------------------------------------------------------------ | --------------- |
| 名称     | 虚拟列的名称。                                               | 是              |
| 表达     | 一个[表达式](http://druid.io/docs/0.12.3/misc/math-expr.html)，它将一行作为输入并输出虚拟列的值。 | 是              |
| 输出类型 | 表达式的输出将被强制转换为此类型。可以是LONG，FLOAT，DOUBLE或STRING。 | 不，默认是FLOAT |