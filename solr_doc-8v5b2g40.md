## Solr修改架构 
<div class="content-intro view-box ">
```
POST /collection/schema
```
Solr 如果要添加、删除或替换字段、动态字段规则、复制字段规则或新字段类型，可以将 POST 请求发送到 /collection/schema/ 端点，并使用一系列命令以执行请求的操作。支持以下命令：  

    - add-field：用你提供的参数添加一个新的字段。
    - delete-field：删除一个字段。
    - replace-field：用一个不同的配置替换现有的字段。
    - add-dynamic-field：使用您提供的参数添加新的动态字段规则。
    - delete-dynamic-field：删除一个动态的字段规则。
    - replace-dynamic-field：用一个配置不同的现有动态字段规则替换。
    - add-field-type：用你提供的参数添加一个新的字段类型。
    - delete-field-type：删除一个字段类型。
    - replace-field-type：用不同的配置替换现有的字段类型。
    - add-copy-field：添加一个新的复制字段规则。
    - delete-copy-field：删除复制字段规则。

这些命令可以在不同的 POST 请求或相同的 POST 请求中发出。命令按照指定的顺序执行。  
  
在每种情况下，响应将包括处理请求的状态和时间，但不包括整个架构。  
在使用 API​​ 修改架构时，将自动发生核心重新加载，以便随后对其索引的文档立即进行更改。以前索引的文档不会自动处理 - 如果他们使用了您更改的架构元素，则必须重新编制索引。  

## 添加一个新的字段
<h3/>
该 add-field 命令将新的字段定义添加到您的架构中。如果同名的字段存在，则会引发错误。  
  
当使用手动 schema.xml 编辑定义字段时可用的所有属性都可以通过 API 传递。这些请求属性在“ 定义字段 ”一节中详细介绍。  
例如，要定义一个名为 “sell-by”，类型为 “pdate” 的新存储字段，您可以发送以下请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"sell-by",
     "type":"pdate",
     "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 删除一个字段

该 delete-field 命令从架构中删除字段定义。如果该字段在架构中不存在，或者该字段是复制字段规则的源或目标，则会引发错误。  
例如，要删除名为 “sell-by” 的字段，可以发送以下请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field" : { "name":"sell-by" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 替换一个字段

该 replace-field 命令替换字段的定义。请注意，您必须提供字段的完整定义 - 此命令不会部分修改字段的定义。如果该架构中不存在该字段，则会引发错误。  
使用手动 schema.xml 编辑定义字段时可用的所有属性都可以通过 API 传递。这些请求属性在“ 定义字段 ”一节中详细介绍。  
例如，要替换现有字段 “sell-by” 的定义，使其为 “date” 类型而不被存储，则会发送以下请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field":{
     "name":"sell-by",
     "type":"date",
     "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 添加动态字段规则

该 add-dynamic-field 命令将新的动态字段规则添加到您的架构中。  
  
当使用 schema.xml 编辑时的所有可用的属性都可以通过 POST 请求传递。“ 动态字段 ”部分详细介绍了可以为动态字段规则定义的所有属性。  
例如，要创建一个新的动态字段规则，其中所有以“_s”结尾的传入字段将被存储并且具有字段类型“string”，您可以像这样发送请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-dynamic-field":{
     "name":"*_s",
     "type":"string",
     "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 删除动态字段规则

该 delete-dynamic-field 命令从架构中删除一个动态的字段规则。如果架构中不存在动态字段规则，或者架构包含的目标或目标仅与此动态字段规则匹配的副本字段规则，则会引发错误。  
例如，要删除匹配“* _s”的动态字段规则，可以像这样发布请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-dynamic-field":{ "name":"*_s" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 替换动态字段规则

该 replace-dynamic-field 命令将替换模式中的动态字段规则。请注意，您必须提供动态字段规则的完整定义 - 此命令不会部分修改动态字段规则的定义。如果模式中不存在动态字段规则，则会引发错误。  
当使用 schema.xml 编辑时的所有可用的属性都可以通过 POST 请求传递。“ 动态字段 ”部分详细介绍了可以为动态字段规则定义的所有属性。  
例如，要将“* _s”动态字段规则的定义替换为字段类型为“text_general”且未存储的规则，可以像这样发布请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-dynamic-field":{
     "name":"*_s",
     "type":"text_general",
     "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 添加一个新的字段类型

该 add-field-type 命令将新的字段类型添加到您的架构中。  
schema.xml 手动编辑时可用的所有字段类型属性都可用于 POST 请求。命令的结构是标准字段类型定义的 JSON 映射，包括名称、类、索引和查询分析器定义等。有关所有可用选项的详细信息，请参见“ [Solr字段类型](https://www.w3cschool.cn/solr_doc/solr_doc-cw8g2fzo.html) ”一节。  
例如，要创建一个名为 “myNewTxtField” 的新字段类型，可以按如下方式发布请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type" : {
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer" : {
        "charFilters":[{
           "class":"solr.PatternReplaceCharFilterFactory",
           "replacement":"$1$1",
           "pattern":"([a-zA-Z])\\\\1+" }],
        "tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}}
}' http://localhost:8983/solr/gettingstarted/schema
```
注意在这个例子中，我们只定义了一个分析器部分，它将应用于索引分析和查询分析。如果我们想定义单独的分析，我们将使上面的例子中的 analyzer 部分用单独的部分 indexAnalyzer 和 queryAnalyzer 取代。正如在下面这个例子所述：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTextField",
     "class":"solr.TextField",
     "indexAnalyzer":{
        "tokenizer":{
           "class":"solr.PathHierarchyTokenizerFactory",
           "delimiter":"/" }},
     "queryAnalyzer":{
        "tokenizer":{
           "class":"solr.KeywordTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

## 删除一个字段类型

该 delete-field-type 命令从模式中删除字段类型。如果模式中不存在字段类型，或模式中的任何字段或动态字段规则使用字段类型，则会引发错误。  
例如，要删除名为 “myNewTxtField” 的字段类型，可以按如下所示进行 POST 请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field-type":{ "name":"myNewTxtField" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 替换一个字段类型

该 replace-field-type 命令将替换模式中的字段类型。请注意，您必须提供字段类型的完整定义 - 此命令不会部分修改字段类型的定义。如果模式中不存在字段类型，则会引发错误。  
schema.xml 手动编辑时可用的所有字段类型属性都可用于 POST 请求。命令的结构是标准字段类型定义的 JSON 映射，包括名称、类、索引和查询分析器定义等。有关所有可用选项的详细信息，请参见“ [Solr字段类型](https://www.w3cschool.cn/solr_doc/solr_doc-cw8g2fzo.html) ”一节。  
例如，要替换名为“myNewTxtField”的字段类型的定义，可以按如下方式创建POST请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

## 添加新的复制字段规则

该 add-copy-field 命令将新的复制字段规则添加到您的架构中。  
该命令所支持的属性与在创建复制字段规则时通过手动编辑 schema.xml 相同，如下所示:  

**source**
    
        源字段。该参数是必需的。  
    
**dest**
    
        源字段将被复制到的字段或字段数组。该参数是必需的。  
    
**maxChars**
    
        要复制的字符数的上限。[复制字段](https://www.w3cschool.cn/solr_doc/solr_doc-1qmz2g1d.html)部分有更多的细节。  
    

例如，要定义将字段 “shelf” 复制到 “location” 和 “catchall” 字段的规则，可以发送以下请求：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-copy-field":{
     "source":"shelf",
     "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```

## 删除复制字段规则

该 delete-copy-field 命令从模式中删除复制字段规则。如果模式中不存在复制字段规则，则会引发错误。  
该命令要求 source 和 dest 属性。  
例如, 要删除一条将字段 "shelf" 复制到 "location" 字段的规则，您可以发布以下请求:  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-copy-field":{ "source":"shelf", "dest":"location" }
}' http://localhost:8983/solr/gettingstarted/schema
```

## 单个 POST 中的多个命令

可以在单个命令中执行一个或多个添加请求。API 是事务性的，单个调用中的所有命令要么一起成功，要么一起失败。  
  
这些命令按照指定的顺序执行。这意味着如果你想创建一个新的字段类型并且在同一请求中使用新字段的字段类型，那么创建字段类型的请求部分必须位于创建新字段的部分之前。类似地，由于字段必须存在才能用于复制字段规则，因此添加字段的请求必须出现在请求字段用作复制字段规则的源或目标之前。  
进行多个请求的语法支持多种方法。首先，命令可以简单地按顺序进行，就像在这个请求中创建一个新的字段类型，然后是一个使用该类型的字段：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{
        "charFilters":[{
           "class":"solr.PatternReplaceCharFilterFactory",
           "replacement":"$1$1",
           "pattern":"([a-zA-Z])\\\\1+" }],
        "tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}},
   "add-field" : {
      "name":"sell-by",
      "type":"myNewTxtField",
      "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```
或者，可以重复相同的命令，如下例所示：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"shelf",
     "type":"myNewTxtField",
     "stored":true },
  "add-field":{
     "name":"location",
     "type":"myNewTxtField",
     "stored":true },
  "add-copy-field":{
     "source":"shelf",
      "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```
最后，重复的命令可以作为数组发送：  
```
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":[
     { "name":"shelf",
       "type":"myNewTxtField",
       "stored":true },
     { "name":"location",
       "type":"myNewTxtField",
       "stored":true }]
}' http://localhost:8983/solr/gettingstarted/schema
```

## 副本之间的架构变化

在 SolrCloud 模式下运行时，对一个节点上的架构所做的更改将传播到集合中的所有副本。  
  
您可以将 updateTimeoutSecs 参数与您的请求一起传递，以设置等待的秒数，直到所有副本确认它们应用了架构更新。这有助于您的客户端应用程序更加健壮，因为您可以确保所有副本在指定的时间内都有给定的架构更改。  
如果在指定时间内没有达成所有副本的协议，则请求将失败，并且错误消息将包含有关哪些副本有问题的信息。在大多数情况下，唯一的选择是在等待一段时间后重新尝试更改。如果问题仍然存在，那么您可能需要调查在应用更改时遇到问题的副本上的服务器日志。  

  
如果您没有提供 updateTimeoutSecs 参数，则默认行为是在将更新持久化到 ZooKeeper 之后，接收节点立即返回。所有其他副本将异步应用更新。因此，如果不提供超时，则客户端应用程序无法确定所有副本都应用了更改。  
