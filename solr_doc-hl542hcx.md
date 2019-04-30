## Solr：流表达式 
<div class="content-intro view-box ">流表达式为 Solr Cloud 提供了一种简单而强大的流处理语言。  
  
流式表达式是一组可以组合起来执行许多不同的并行计算任务的函数。这些函数是并行 SQL 接口的基础。  
有越来越多的函数库可以结合起来实现：  

    - 请求/响应流处理
    - 批量流处理
    - 快速交互式 MapReduce
    - 聚合（两者都推下了分面和洗牌的 MapReduce）  

    - 并行关系代数（分布式连接，交集，联合，互补）
    - 发布/订阅消息
    - 分布图遍历
    - 机器学习和并行迭代模型训练
    - 异常检测
    - 推荐系统
    - 检索和排序服务
    - 文本分类和特征提取
    - 流式 NLP
    - 统计编程

来自外部系统的流可以与来自 Solr 的流结合，用户可以通过遵循 Solr 的 Java streaming API 来添加自己的流函数。  
  
注意：流表达式和流 API 都被认为是实验性的，并且 API 是可以改变的。  

## 流语言基础<a href="http://lucene.apache.org/solr/guide/7_0/streaming-expressions.html#stream-language-basics"/>

流表达式由与 Solr 集合一起使用的流式函数组成。它们发出一串元组（键/值映射）。  
许多提供的流函数被设计用来处理整个结果集，而不是像普通搜索那样的前 N 个结果。这由 /export 处理程序支持。  
一些流函数作为流源来发起流式流。其他流函数充当流修饰符来包装其他流函数并在元组流上执行操作。许多流函数可以跨工作集合并行化。这对于关系代数函数来说可能特别强大。  

### 流请求和响应<a href="http://lucene.apache.org/solr/guide/7_0/streaming-expressions.html#streaming-requests-and-responses"/>

Solr 有一个 /stream 请求处理程序，它接受流表达式请求并将这些元组作为 JSON 流返回。这个请求处理程序是隐式定义的，这意味着在 solrconfig.xml 中没有任何定义，请参阅隐式RequestHandlers。  
  
/stream 请求处理程序有一个参数：expr，它是用来指定流表达式。例如，这个 curl 命令将一个简单的 search() 表达式编码并发布到 /stream 处理程序：  
```
curl --data-urlencode 'expr=search(enron_emails,
                                   q="from:1800flowers*",
                                   fl="from, to",
                                   sort="from asc",
                                   qt="/export")' http://localhost:8983/solr/enron_emails/stream
```
每个函数的参数详情如下。  
对于上面的例子，/stream 处理程序用下面的 JSON 响应进行响应：  
```
{"result-set":{"docs":[
   {"from":"1800flowers.133139412@s2u2.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers.93690065@s2u2.com","to":"jtholt@ect.enron.com"},
   {"from":"1800flowers.96749439@s2u2.com","to":"alewis@enron.com"},
   {"from":"1800flowers@1800flowers.flonetwork.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@1800flowers.flonetwork.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@1800flowers.flonetwork.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@1800flowers.flonetwork.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@1800flowers.flonetwork.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"ebass@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"lcampbel@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"ebass@enron.com"},
   {"from":"1800flowers@shop2u.com","to":"ebass@enron.com"},
   {"EOF":true,"RESPONSE_TIME":33}]}
}
```
注意上面的示例流中的最后一个元组是：{"EOF":true,"RESPONSE_TIME":33}。该 EOF 指示流的结束。要处理 JSON 响应，您需要使用流式 JSON 实现，因为流表达式旨在返回可能有数百万条记录的整个结果集。在您的 JSON 客户端中，您需要遍历每个文档（元组），并检查 EOF 元组来确定流的结束。  
  
org.apache.solr.client.solrj.io 软件包提供了将流表达式编译到流 API 对象中的 Java 类。这些类可以用来从 Java 应用程序中执行流表达式。例如：  
```
StreamFactory streamFactory = new StreamFactory().withCollectionZkHost("collection1", zkServer.getZkAddress())
    .withStreamFunction("search", CloudSolrStream.class)
    .withStreamFunction("unique", UniqueStream.class)
    .withStreamFunction("top", RankStream.class)
    .withStreamFunction("group", ReducerStream.class)
    .withStreamFunction("parallel", ParallelStream.class);
ParallelStream pstream = (ParallelStream)streamFactory.constructStream("parallel(collection1, group(search(collection1, q=\"*:*\", fl=\"id,a_s,a_i,a_f\", sort=\"a_s asc,a_f asc\", partitionKeys=\"a_s\"), by=\"a_s asc\"), workers=\"2\", zkHost=\""+zkHost+"\", sort=\"a_s asc\")");
```

### 数据要求<a href="http://lucene.apache.org/solr/guide/7_0/streaming-expressions.html#data-requirements"/>

由于流表达式依赖于 /export 处理程序，所以使用 /export 的许多字段和字段类型要求也是 /stream 的要求，特别是对于 sort 和 fl 参数。有关详细信息，请参阅导出结果集部分。  
  

## 流表达式的类型<a href="http://lucene.apache.org/solr/guide/7_0/streaming-expressions.html#types-of-streaming-expressions"/>

### 关于流源
流源来源于流。最常用的是 search 查询。  
对所有可用源表达式的完整引用在流源引用中可用。  
  

### 关于流修饰符

流修饰符包装其他流函数或在流上执行操作。  
Stream Decorator Reference 中提供了对所有可用修饰符表达式的完整引用。  

### 关于 Stream 评估器
流计算器可以用来计算基于元组中其他值的新值。新计算的值可以放入元组（作为 select(…​) 子句的一部分），用于过滤流（作为 having(…​) 子句的一部分），以及用于其他事情。计算器可以包含字段名称、原始值或其他计算器，从而使您能够创建复杂的计算逻辑，包括有条件的 if/then 选项。  
如果您想将原始值用作评估的一部分，则需要考虑计算器的解析顺序。  
1 <li>如果参数可以被解析成一个有效的数字，那么它被认为是一个数字。例如，add(3,4.5)</li>2 <li>如果参数可以被解析成一个有效的布尔值，那么它被认为是一个布尔值。例如，eq(true,false)</li>3 <li>如果参数可以被分析成有效的计算器，那么它被认为是一个计算器。例如，eq(add(10,4),add(7,7))</li>4 <li>即使引用了该参数，也会将其视为字段名称。例如，eq(fieldA,"fieldB")</li>
如果您希望使用原始字符串作为计算的一部分，则您将需要考虑使用 raw(string) 计算程序。无论输入什么内容，都将始终返回原始值。  
Stream Evaluator Reference 中提供了对所有可用计算程序表达式的完整引用。  
