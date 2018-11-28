# JavaScript编程指南

本页讨论如何使用JavaScript扩展Druid。

## 例子

JavaScript可用于以各种方式扩展德鲁伊：

- [聚合器](http://druid.io/docs/0.12.3/querying/aggregations.html#javascript-aggregator)
- [提取功能](http://druid.io/docs/0.12.3/querying/dimensionspecs.html#javascript-extraction-function)
- [过滤器](http://druid.io/docs/0.12.3/querying/filters.html#javascript-filter)
- [后聚合](http://druid.io/docs/0.12.3/querying/post-aggregations.html#javascript-post-aggregator)
- [输入解析器](http://druid.io/docs/0.12.3/ingestion/data-formats.html#javascript)
- [路由器策略](http://druid.io/docs/0.12.3/development/router.html#javascript)
- [工人选择策略](http://druid.io/docs/0.12.3/configuration/indexing-service.html#javascript)

JavaScript可以在运行时动态注入，使得快速创建新功能原型变得方便，而无需编写和部署Druid扩展。

Druid在优化级别9使用Mozilla Rhino引擎来编译和执行JavaScript。

## 安全

Druid不在沙箱中执行JavaScript功能，因此他们可以完全访问机器。所以Javascript函数允许用户在德鲁伊进程中执行任意代码。因此，默认情况下，Javascript被禁用。但是，在开发/暂存环境或安全生产环境中，您可以通过设置[配置属性](http://druid.io/docs/0.12.3/configuration/index.html#javascript) 来启用它们`druid.javascript.enabled = true`。

## 全局变量

避免使用全局变量。德鲁伊可能在多个线程之间共享全局范围，如果使用全局变量，这可能导致不可预测的结果。

## 性能

简单的JavaScript函数通常会对本机速度造成轻微的性能损失。更复杂的JavaScript函数可能会有更陡峭的性能损失。Druid每个查询每个节点编译一次JavaScript函数。

在大量使用JavaScript函数时，您可能需要特别注意垃圾收集，尤其是编译类本身的垃圾收集。确保使用支持及时收集未使用类的垃圾收集器配置（这在使用Metaspace的JDK8上比在JDK7上更容易）。

## JavaScript与原生扩展

通常，我们建议在安全性不成问题时使用JavaScript，并且当开发速度比性能或内存使用更重要时。如果安全性是一个问题，或者性能和内存使用是最重要的，我们建议开发本机德鲁伊扩展。

此外，本机德鲁伊扩展比JavaScript功能更灵活。由于需要自定义数据格式，因此必须将某些类型的扩展（如草图）编写为本机Druid扩展。