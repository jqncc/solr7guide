## 导出Solr结果集 
<div class="content-intro view-box ">可以使用特殊的秩查询解析器和[响应编写器](https://www.w3cschool.cn/solr_doc/solr_doc-rmnk2h9y.html)来导出完全排序的结果集，这种解析器和响应编写器专门设计用于处理涉及排序和导出数百万条记录的场景。  
  
此功能使用 stream 排序技术，在毫秒内开始发送记录，并继续对结果进行 stream 处理，直到整个结果集被排序并且导出为止。  
此功能可能有用的情况包括：会话分析、分布式合并联接、时间序列汇总、高基数字段上的聚合、完全分布式字段合并和基于排序的统计信息。  

## 字段要求

字段要求所有正在排序和导出的字段必须将 docValues 设置为 true。有关详细信息, 请参阅关于 DocValues 的一节。  

## / export RequestHandler

/export 请求处理程序具有适当配置，是 Solr 的现成请求处理程序之一，有关更多信息，请参阅隐式 RequestHandlers。  
  
请注意，这个请求处理程序的属性被定义为“不变量”，这意味着它们不能被其他时间传递的其他属性（例如在查询时）重写。  

## 请求结果导出

您可以使用 /export 来请求导出查询的结果集。  
  
所有查询都必须包括 sort 和 fl 参数，否则查询将返回一个错误。过滤器查询也被支持。  
受支持的响应编写器是 json 和 javabin。由于向后兼容性的原因，wt=xsort也被支持作为输入，但是 wt=xsort 与 wt=json 的行为相同。默认的输出格式是 json。  
以下是一些索引日志数据的导出请求示例：  
```
http://localhost:8983/solr/core_name/export?q=my-query&amp;sort=severity+desc,timestamp+desc&amp;fl=severity,timestamp,msg
```

### 指定排序标准

sort 属性定义了文档在导出的结果集中的排序方式。结果可以通过字段类型为 int，long，float，double，string 的任何字段进行排序。排序字段必须是单值字段。  
  
最多可以为每个请求指定四个排序字段，使用 “asc” 或 “desc” 属性。  

### 指定字段列表

fl 属性定义将将随结果集导出的字段。任何可以排序的字段类型（即 int，long，float，double，string，date，boolean）都可以在字段列表中使用。这些字段可以是单值或多值的。但是，目前还不支持返回分数和通配符。  

## 分布式支持

请参见用于分布式支持的部分流表达式。  
