# 删除数据

永久删除德鲁伊片段有两个步骤：

1. 该段必须首先标记为“未使用”。当按照保留规则删除段时，以及用户通过Coordinator API手动禁用段时，会发生这种情况。
2. 在段被标记为“未使用”之后，Kill Task将删除Druid元数据存储中的任何“未使用”段以及深度存储。

有关保留规则的文档，请参阅[数据保留](http://druid.io/docs/0.12.3/operations/rule-configuration.html)。

有关使用Coordinator API禁用段的文档，请参阅[协调器删除API](http://druid.io/docs/0.12.3/operations/api-reference.html#coordinator-delete)

[教程：删除数据中](http://druid.io/docs/0.12.3/tutorials/tutorial-delete-data.html)提供了数据删除教程

## 杀死任务

Kill任务删除有关段的所有信息并将其从深层存储中删除。必须在德鲁伊段表中禁用（使用== 0）可填充段。可用的语法是：

```json
{
    "type": "kill",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "interval" : <all_segments_in_this_interval_will_die!>,
    "context": <task context>
}
```