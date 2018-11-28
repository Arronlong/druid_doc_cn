# 近似直方图聚合器

确保[包含](http://druid.io/docs/0.12.3/operations/including-extensions.html) `druid-histogram`作为扩展名。

该聚合器基于 <http://jmlr.org/papers/volume11/ben-haim10a/ben-haim10a.pdf> 来计算近似直方图，并进行了以下修改： - 为了速度而对准确性进行了一些权衡（参见下面） - 只要不同数据点的数量小于分辨率（质心数），在数据点很少或处理离散数据点时提高准确度，草图就会保留准确的原始数据。你可以在[这篇文章中](https://metamarkets.com/2013/histograms/)找到一些细节。

由于某种原因，近似直方图草图仍然是实验性的，您应该在使用它们之前了解当前实现的局限性。近似值与数据有很大关系，这使得很难给出良好的通用指南，因此您应该试验并查看哪些参数适用于您的数据。

在使用它们之前，需要注意以下几点：

- 如原始论文所示，近似没有正式的误差界限。实际上，如果分布偏斜，则近似值会变差。
- 该算法依赖于顺序，因此，由于合并结果的顺序的变化，结果可能因同一查询而异。
- 一般来说，只有当数据随机分布时（即如果数据点最终按列排序，近似将是可怕的），该算法才能正常工作
- 我们交换聚合速度的准确性，在将直方图添加到一起时采用一些快捷方式，如果您的数据以某种方式订购，或者如果您的分布有长尾，则会导致病态情况。增加草图的分辨率以获得所需的精度应该更便宜。

话虽这么说，当平均值不够好时，这些草图对于获得一阶近似是有用的。假设您的段中的大多数行存储的数据点少于直方图的分辨率，您应该能够将它们用于监视目的并检测具有几百个质心的有意义的变化。为了获得具有数百万行数据的第95百分位数的良好准确度读数，您可能需要使用数千个质心，尤其是长尾，因为这是近似值更差的地方。

### 在摄取时创建近似直方图草图

要使用此功能，索引时必须包含“approxHistogram”或“approxHistogramFold”聚合器。摄取聚合器只能应用于数值。如果使用“approxHistogram”，那么缺少该值的任何输入行将被视为具有值0，而对于“approxHistogramFold”，将忽略这些行。

要查询结果，查询中必须包含“approxHistogramFold”聚合器。

```json
{
  "type" : "approxHistogram or approxHistogramFold (at ingestion time), approxHistogramFold (at query time)",
  "name" : <output_name>,
  "fieldName" : <metric_name>,
  "resolution" : <integer>,
  "numBuckets" : <integer>,
  "lowerLimit" : <float>,
  "upperLimit" : <float>
}
```

| 属性                      | 描述                                                         | 默认         |
| ------------------------- | ------------------------------------------------------------ | ------------ |
| `resolution`              | 要存储的质心数（数据点）。分辨率越高，结果越准确，但计算速度越慢。 | 50           |
| `numBuckets`              | 生成的直方图的输出桶数。根据底层数据的范围，桶间隔是动态的。使用后聚合器可以更好地控制存储方案 | 7            |
| `lowerLimit`/`upperLimit` | 限制给定范围的近似值。超出此范围的值将聚合为两个质心。超出此范围的值的计数仍然保持不变。 | -INF / + INF |

### 近似直方图后聚合器

后聚合器用于将不透明的近似直方图草图转换为分块直方图表示，以及计算各种分布度量，例如分位数，分钟和最大值。

#### 相同的桶后聚合器

使用给定数量的相等大小的箱计算近似直方图的直观表示。存储桶间隔基于基础数据的范围。

```json
{
  "type": "equalBuckets",
  "name": "<output_name>",
  "fieldName": "<aggregator_name>",
  "numBuckets": <count>
}
```

#### 桶后聚合器

给定初始断点，偏移量和桶大小，计算可视化表示。

存储桶大小决定了分箱间隔的宽度。

偏移确定这些间隔箱对齐的值。

```json
{
  "type": "buckets",
  "name": "<output_name>",
  "fieldName": "<aggregator_name>",
  "bucketSize": <bucket_size>,
  "offset": <offset>
}
```

#### 定制桶后聚合器

计算近似直方图的直观表示，其中根据给定的间隔布置了分箱。

```json
{ "type" : "customBuckets", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "breaks" : [ <value>, <value>, ... ] }
```

#### min post-aggregator

返回基础近似直方图聚合器的最小值

```json
{ "type" : "min", "name" : <output_name>, "fieldName" : <aggregator_name> }
```

#### 最大后聚合器

返回基础近似直方图聚合器的最大值

```json
{ "type" : "max", "name" : <output_name>, "fieldName" : <aggregator_name> }
```

#### 分位数后聚合器

基于基础近似直方图聚合器计算单个分位数

```json
{ "type" : "quantile", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "probability" : <quantile> }
```

#### 分位数后聚合器

基于底层近似直方图聚合器计算分位数数组

```json
{ "type" : "quantiles", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "probabilities" : [ <quantile>, <quantile>, ... ] }
```