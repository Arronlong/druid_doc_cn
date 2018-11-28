# 德鲁伊警报

德鲁伊会在遇到意外情况时发出警报。

警报作为JSON对象发布到运行时日志文件或HTTP（发送到Apache Kafka等服务）。默认情况下禁用警报发射。

所有德鲁伊警报共享一组共同的字段：

- `timestamp` - 创建警报的时间
- `service` - 发出警报的服务名称
- `host` - 发出警报的主机名
- `severity` - 警报的严重性，例如异常，组件故障，服务故障等。
- `description` - 警报的描述
- `data`- 如果有异常，那么带有字段的JSON对象`exceptionType`，`exceptionMessage`和`exceptionStackTrace`