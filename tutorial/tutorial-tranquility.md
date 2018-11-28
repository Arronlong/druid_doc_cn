## 运行Tranquility服务器

运行以下命令：

```bash
bin/tranquility server -configFile ../examples/conf/tranquility/wikipedia-server.json -Ddruid.extensions.loadList=[]
```

## 发送数据

让我们将样本维基百科编辑数据发送给Tranquility：

```bash
curl -XPOST -H'Content-Type: application/json' --data-binary @quickstart/wikiticker-2015-09-12-sampled.json http://localhost:8200/v1/post/wikipedia
```

这将打印如下：

```json
{"result":{"received":39244,"sent":39244}}
```

这表明HTTP服务器收到了39,244个事件，并向德鲁伊发送了39,244个事件。如果在启用Tranquility Server后运行太快，此命令可能会生成“连接被拒绝”错误，这意味着服务器尚未启动。它应该在几秒钟内启动。该命令也可能需要几秒钟才能完成第一次运行它，在此期间德鲁伊资源被分配给摄取任务。完成后，后续POST将快速完成。

一旦将数据发送到德鲁伊，您就可以立即查询它。

如果您看到`sent`计数为0，请重试send命令，直到`sent`计数也显示为39244：

```json
{"result":{"received":39244,"sent":0}}
```

## 查询您的数据

请按照[查询教程](http://druid.io/docs/0.12.3/tutorials/tutorial-query.html)对新加载的数据运行一些示例查询。

## 清理

如果您希望浏览任何其他摄取教程，则需要重置群集并按照这些[重置说明进行操作](http://druid.io/docs/0.12.3/tutorials/index.html#resetting-cluster-state)，因为其他教程将写入相同的“维基百科”数据源。

## 进一步阅读

有关Tranquility的更多信息，请参阅[Tranquility文档](https://github.com/druid-io/tranquility)。