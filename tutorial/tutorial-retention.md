# 教程：配置数据保留

本教程演示如何在数据源上配置保留规则，以设置将保留或删除的数据的时间间隔。

对于本教程，我们假设您已经按照[单机快速入门](http://druid.io/docs/0.12.3/tutorials/index.html)中的描述下载了Druid，并让它在本地计算机上运行。

完成[教程：加载文件](http://druid.io/docs/0.12.3/tutorials/tutorial-batch.html)和[教程：查询数据](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)也很有帮助。

## 加载示例数据

在本教程中，我们将使用Wikipedia编辑样本数据，其中包含摄取任务规范，该规范将为输入数据中的每个小时创建一个单独的段。

摄取规范可以在`examples/retention-index.json`。让我们提交该规范，它将创建一个名为的数据源`retention-tutorial`：

```bash
curl -X 'POST' -H 'Content-Type:application/json' -d @examples/retention-index.json http://localhost:8090/druid/indexer/v1/task
```

摄取完成后，在浏览器中转到http：// localhost：8081以访问协调器控制台。

在协调器控制台中，转到`datasources`页面顶部的选项卡。

此选项卡显示可用的数据源以及每个数据源的保留规则摘要：

![摘要](http://druid.io/docs/0.12.3/tutorials/img/tutorial-retention-00.png)

目前，没有为`retention-tutorial`数据源设置规则。请注意，有默认规则，当前设置为`load Forever 2 in _default_tier`。

这意味着无论时间戳如何，都将加载所有数据，并且每个段将被复制到默认层中的两个节点。

在本教程中，我们暂时忽略分层和冗余概念。

让我们点击`retention-tutorial`左边的数据源。

下一页（http：// localhost：8081 /＃/ datasources / retention-tutorial）提供有关数据源包含哪些段的信息。在左侧，页面显示有24个段，每个段包含2015-09-12特定小时的数据：

![原始细分](http://druid.io/docs/0.12.3/tutorials/img/tutorial-retention-01.png)

## 设置保留规则

假设我们想要删除2015-09-12的前12个小时的数据，并保留2015-09-12后12个小时的数据。

单击`edit rules`页面左上角带有铅笔图标的按钮。

将出现规则配置窗口。输入`tutorial`user和changelog注释字段。

现在单击`+ Add a rule`按钮两次。

在`rule #1`顶部的框中，单击`Load`,, 在间隔框中`Interval`输入`2015-09-12T12:00:00.000Z/2015-09-13T00:00:00.000Z`，然后单击`+ _default_tier replicant`。

在`rule #2`底部的框中，单击`Drop`和`Forever`。

规则应如下所示：

![设定规则](http://druid.io/docs/0.12.3/tutorials/img/tutorial-retention-02.png)

现在单击`Save all rules`，等待几秒钟，然后刷新页面。

2015-09-12前12小时的细分现已消失：

![新细分](http://druid.io/docs/0.12.3/tutorials/img/tutorial-retention-03.png)

生成的保留规则链如下：

- loadByInterval 2015-09-12T12 / 2015-09-13（12小时）
- dropForever
- loadForever（默认规则）

规则链从上到下进行评估，默认规则链始终添加在底部。

我们刚刚创建的教程规则链加载数据，如果它在指定的12小时间隔内。

如果数据不在12小时间隔内，则规则链将进行`dropForever`下一步评估，这将删除所有数据。

该`dropForever`终止规则链，有效地覆盖默认的`loadForever`规则，这将永远不会在这个规则链到达。

请注意，在本教程中，我们定义了特定时间间隔的加载规则。

相反，如果您希望根据数据的大小来保留数据（例如，保留过去3个月到当前时间范围内的数据），则应定义“期间”加载规则。

## 进一步阅读

- [加载规则](http://druid.io/docs/0.12.3/operations/rule-configuration.html)