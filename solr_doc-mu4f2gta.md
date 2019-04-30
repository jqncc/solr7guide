## Solr文档转换器 
<div class="content-intro view-box ">Solr文档转换器可用于修改查询结果中返回的有关每个文档的信息。
      
  

## 使用文件转换器

在执行请求时，可以使用方括号将文档转换器包含在 fl 参数中，例如：  
```
fl=id,name,score,[shard]
```

一些转换器允许或要求可以在括号内指定为键值对的本地参数：  
```
fl=id,name,score,[explain style=nl]
```

与常规字段一样，您可以更改 Transformer 通过前缀向文档添加字段时使用的键：  
```
fl=id,name,score,my_val_a:[value v=42 t=int],my_val_b:[value v=7 t=float]
```

以下各节将详细讨论这些转换器的功能。  

## 可用转换器


### [Value] - ValueAugmenterFactory

修改每个文档以包含完全相同的值，就像它是每个文档中的存储字段一样：  
```
q=*:*&amp;fl=id,greeting:[value v='hello']
```

上面的查询会产生如下结果：  
```
&lt;result name="response" numFound="32" start="0"&gt;
  &lt;doc&gt;
    &lt;str name="id"&gt;1&lt;/str&gt;
    &lt;str name="greeting"&gt;hello&lt;/str&gt;&lt;/doc&gt;
  &lt;/doc&gt;
  ...
```
默认情况下，值以字符串的形式返回，但是可以使用 int、float、double 或 date 的值指定 “t” 参数来强制指定返回类型：  
```
q=*:*&amp;fl=id,my_number:[value v=42 t=int],my_string:[value v=42]
```

除了使用这些请求参数之外，还可以配置 ValueAugmenterFactory 的其他命名实例，或覆盖 solrconfig.xml 文件中现有 [value] 转换器的默认行为：  
```
&lt;transformer name="mytrans2" class="org.apache.solr.response.transform.ValueAugmenterFactory" &gt;
  &lt;int name="value"&gt;5&lt;/int&gt;
&lt;/transformer&gt;
&lt;transformer name="value" class="org.apache.solr.response.transform.ValueAugmenterFactory" &gt;
  &lt;double name="defaultValue"&gt;5&lt;/double&gt;
&lt;/transformer&gt;
```

“ value”选项强制使用一个显式值，而“ defaultValue”选项提供了一个默认值，仍然可以使用“ v”和“ t”本地参数覆盖。  

### [explain] - ExplainAugmenterFactory

与 "调试" 部分中每个文档的可用信息完全一样，对每个文档的分数进行一个内联解释：  
```
q=features:cache&amp;fl=id,[explain style=nl]
```

用于 style 支持的值是 text、html 和 nl，它以结构化数据的形式返回信息：  
```
{ "response":{"numFound":2,"start":0,"docs":[
      {
        "id":"6H500F0",
        "[explain]":{
          "match":true,
          "value":1.052226,
          "description":"weight(features:cache in 2) [DefaultSimilarity], result of:",
          "details":[{
}]}}]}}
```

默认样式可以通过在您的配置中指定一个“args” 参数进行配置：  
```
&lt;transformer name="explain" class="org.apache.solr.response.transform.ExplainAugmenterFactory" &gt;
  &lt;str name="args"&gt;nl&lt;/str&gt;
&lt;/transformer&gt;
```


### [child] - ChildDocTransformerFactory

此转换器返回与您的查询匹配的每个父文档的所有子代文档，该列表嵌套在匹配的父文档内。当您为嵌套的子文档编制索引并且想要为任何类型的搜索查询检索相关父文档的子文档时，这非常有用。  
```
fl=id,[child parentFilter=doc_type:book childFilter=doc_type:chapter limit=100]
```

请注意，即使查询本身不是块联接查询，也可以使用此转换器。
      
  
使用此变换器时，必须指定 parentFilter 参数，并且与所有块联接查询的工作方式相同，其他可选参数为：  

    - childFilter - 查询过滤应包含哪些子文档，当您有多层次的分层文档（默认：所有子项）时，这可能特别有用。
    - limit - 每个父文档要返回的最大子文档数量（默认值：10）


### [shard] - ShardAugmenterFactory

这个转换器在分布式请求中添加有关每个文档碎片的信息。
      
  
ShardAugmenterFactory 不支持任何请求参数或配置选项。  

### [docid] - DocIdAugmenterFactory

这个转换器将 Lucene 的内部文档 ID 添加到每个文档 - 这仅用于调试目的。  
DocIdAugmenterFactory 不支持任何请求参数或配置选项。  

### [elevated] 和 [excluded]

这些转换器仅在使用查询高程组件时可用。  

    - [elevated] 注释每个文件以指示是否升高。
    - [excluded] 注释每个文档以指示是否已被排除 - 仅当您也使用 markExcludes 参数时才支持。
```
fl=id,[elevated],[excluded]&amp;excludeIds=GB18030TEST&amp;elevateIds=6H500F0&amp;markExcludes=true
```
- 
```
{ "response":{"numFound":32,"start":0,"docs":[
      {
        "id":"6H500F0",
        "[elevated]":true,
        "[excluded]":false},
      {
        "id":"GB18030TEST",
        "[elevated]":false,
        "[excluded]":true},
      {
        "id":"SP2514N",
        "[elevated]":false,
        "[excluded]":false},
]}}
```


### [json] / [xml]

这些转换器将包含有效 XML 或 JSON 结构的字符串表示形式的字段值替换为实际的原始 XML 或 JSON 结构，而不仅仅是字符串值。每一种 [json] 都只适用于特定的编写器，这样只适用于wt=json 并且 [xml] 只适用于 wt=xml。  
```
fl=id,source_s:[json]&amp;wt=json
```


### [subquery]

此转换器将每个转换文档传递文档字段的单独查询作为子查询参数的输入执行。它通常与 {! join} 和 {! 父} 查询解析器一起使用，旨在改进 [child]。  
  

    - 必须给它一个唯一的名字： fl=*,children:[subquery]
    - 可能会有几个，例如：fl=*,sons:[subquery],daughters:[subquery]。
    - 每个 [subquery] 出现都会在给定名称的结果文档中添加一个字段，该字段的值是一个文档列表，这是使用文档字段作为输入执行子查询的结果。

以下是各种格式的外观：  
```
 &lt;result name="response" numFound="2" start="0"&gt;
      &lt;doc&gt;
         &lt;int name="id"&gt;1&lt;/int&gt;
         &lt;arr name="title"&gt;
            &lt;str&gt;vdczoypirs&lt;/str&gt;
         &lt;/arr&gt;
         &lt;result name="children" numFound="1" start="0"&gt;
            &lt;doc&gt;
               &lt;int name="id"&gt;2&lt;/int&gt;
               &lt;arr name="title"&gt;
                  &lt;str&gt;vdczoypirs&lt;/str&gt;
               &lt;/arr&gt;
            &lt;/doc&gt;
         &lt;/result&gt;
      &lt;/doc&gt;
  ...
```
```
{ "response":{
    "numFound":2, "start":0,
    "docs":[
      {
        "id":1,
        "subject":["parentDocument"],
        "title":["xrxvomgu"],
        "children":{
           "numFound":1, "start":0,
           "docs":[
              { "id":2,
                "cat":["childDocument"]
              }
            ]
      }}]}}
```
```
SolrDocumentList subResults = (SolrDocumentList)doc.getFieldValue("children");
```


#### 子查询结果字段

为了出现在子查询文档列表中，一个字段应该同时指定两个 fl 参数，在主要的一个 fl（尽管主要的结果文档没有这个字段）和在子查询的一个，例如：foo.fl。当然，您可以在这些参数中的任何一个或两个中使用通配符。例如，如果字段标题应该出现在类别子查询中，则可以通过以下方式之一来完成。  
```
fl=...title,categories:[subquery]&amp;categories.fl=title&amp;categories.q=...
fl=...title,categories:[subquery]&amp;categories.fl=*&amp;categories.q=...
fl=...*,categories:[subquery]&amp;categories.fl=*&amp;categories.q=...
fl=...*,categories:[subquery]&amp;categories.fl=*&amp;categories.q=...
```


#### 子查询参数 Shift

如果子查询被声明为：fl=*,foo:[subquery]，则子查询参数以给定的名称和句点作为前缀。例如：  
```
q=:&amp;fl=*,foo:[subquery]&amp;foo.q=to be continued&amp;foo.rows=10&amp;foo.sort=id desc
```

#### 文档字段作为子查询参数的输入

必须将某些文档字段值作为子查询的参数传递。它通过隐式 row.fieldname 参数支持，可以（但可能不只是）通过本地参数语法来引用：  
  
```
q=namne:john&amp;fl=name,id,depts:[subquery]&amp;depts.q={!terms f=idv=$row.dept_id}&amp;depts.rows=10
```
搜索结果中，每个员工都会找到门卫。我们可以说它就像 SQL 加入 ON emp.dept_id=dept.id 一样。  
  
请注意，当文档字段有多个值时，它们默认与逗号连接，可以通过本地参数 foo:[subquery separator=' '] 进行更改，这样模拟{!terms}可以使用它。  
要记录替代的子查询请求参数，请添加相应的参数名称，如：depts.logParamsList=q,fl,rows,row.dept_id  

#### SolrCloud 中的核心和集合

使用 foo:[subquery fromIndex=departments] 来调用同一节点上的另一个核心子查询，这是 {! join} 用于非 SolrCloud 模式。但是在 SolrCloud 的情况下（只有）明确地指定了其子查询的本地参数 collection、shards ，例如：  
  
```
q=:&amp;fl=*,foo:[subquery]&amp;foo.q=cloud&amp;foo.collection=departments
```
如果子查询集合具有不同的唯一键字段名称 (假设 foo_id 与主集合中的 id 相反)，则添加以下参数以适应此差异：foo.fl=id:foo_id&amp;foo.distrib.singlePass=true。否则你就会从 QueryComponent.mergeIds 得到 NullPoniterException。  

### [geo] - Geospatial formatter

使用指定的格式类型名称，从空间字段中设置空间数据的格式。需要两个内部参数：f 用于字段名，w 用于格式名。例如：geojson:[geo f=mySpatialField w=GeoJSON]。  
  
通常情况下，通过将 format 空间字段类型上的属性设置为 WKT 或者 GeoJSON，您可以简单地选择所需的格式类型。有关更多信息，请参阅空间搜索部分。如果是一致的话，它会以你存储它的方式出来。这个转换器提供了一个方便的空间格式转换为不同的检索。  
另外，这个功能对于 RptWithGeometrySpatialField 避免双向存储潜在的大型矢量几何体非常有用。该转换器将检测该字段类型并从磁盘上的内部紧凑二进制表示（在 docValues 中）获取几何图形，然后根据需要对其进行格式化。因此，您不需要将该字段标记为已存储，这将是多余的。从某种意义上说，docValue 和存储值存储之间的双重存储并不是空间唯一的，而是具有多边形的几何体，它可以是大量的数据，而且您希望避免将其存储为冗长的格式（如 GeoJSON 或 WKT）。  

### [features] - LTRFeatureLoggerTransformerFactory

“LTR”前缀代表[学习排序](https://www.w3cschool.cn/solr_doc/solr_doc-v4yj2gyw.html)。这个转换器返回函数的值，它可以用于函数提取和函数记录。  
```
fl=id,[features store=yourFeatureStore]
```

这将返回 yourFeatureStore 存储中函数的值。  
```
fl=id,[features]&amp;rq={!ltr model=yourModel}
```

如果 [features] 与“学习排序”重新排​​列查询一起使用，则将返回重新排序模型（yourModel ）中的函数值。  
  
  
