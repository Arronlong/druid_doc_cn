# 排序订单

这些排序顺序由[TopNMetricSpec](http://druid.io/docs/0.12.3/querying/topnmetricspec.html)，[SearchQuery](http://druid.io/docs/0.12.3/querying/searchquery.html)，GroupByQuery的[LimitSpec](http://druid.io/docs/0.12.3/querying/limitspec.html)和[BoundFilter使用](http://druid.io/docs/0.12.3/querying/filters.html#bound-filter)。

## 辞书

通过将字符串转换为其UTF-8字节数组表示并按字节顺序逐字节进行比较来对值进行排序。

## 字母数字

适用于包含数字和非数字内容的字符串，例如：“file12在文件2之后排序”

有关此排序如何对值进行排序的详细信息，请参阅https://github.com/amjjd/java-alphanum。

此排序不适用于带小数点或负数的数字。*例如，“1.3”在此排序中位于“1.15”之前，因为“15”的有效数字多于“3”。*负数在正数后排序（因为数字字符在负数中位于“ - ”之前）。

## 数字

将值排序为数字，支持整数和浮点值。支持负值。

此排序顺序将尝试将所有字符串值解析为数字。不可解析的值被视为空值，并且空值在数字之前。

当比较两个不可解析的值（例如，“hello”和“world”）时，此排序将通过按字典顺序比较未解析的字符串进行排序。

## STRLEN

按字符串长度对值进行排序。当存在平局时，该比较器将回退到使用String compareTo方法。