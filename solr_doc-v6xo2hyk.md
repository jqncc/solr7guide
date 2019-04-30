## Solr托管资源 
<div class="content-intro view-box ">Solr托管资源公开了一个REST API端点，用于在Solr对象上执行Create-Read-Update-Delete（CRUD）操作。  
具有配置设置或数据的任何长期存在的Solr对象都是被管理资源的好候选。托管资源是对Solr中其他可编程管理的组件的补充，例如用于将字段添加到托管模式的RESTful模式API。  
请考虑一个基于web的用户界面，该UI提供"Solr"服务，用户需要配置一组停用词和同义词映射作为其搜索应用程序的初始设置过程的一部分。使用Solr提供的托管停止过滤器和托管的同义词图过滤器工厂，可以通过托管资源REST API轻松地支持此类用例。  
用户也可以编写自己的自定义插件，利用相同的内部钩子来管理额外的资源REST。  
本节中的所有示例都假设您正在运行“techproducts”Solr示例：  
```
bin/solr -e techproducts
```

## 托管资源概述

我们通过查看Solr提供的几个示例来开始学习托管资源，以使用REST API管理停用词和同义词的示例。阅读本节后，您将准备好深入了解如何在Solr中实施托管资源的细节，以便您可以开始构建自己的实施。  
  

### 管理停用词

首先，您需要定义一个使用ManagedStopFilterFactory的字段类型，例如：  
```
&lt;fieldType name="managed_en" positionIncrementGap="100"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.ManagedStopFilterFactory" 
            managed="english" /&gt; 
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
关于这个字段类型的定义，有两个重要的事情要注意：  
<p/>
用于管理 techproducts 集合中英文stop词的REST端点是：  
  
```
/solr/techproducts/schema/analysis/stopwords/english。
```
示例资源路径应该大多是不言自明的。应该注意的是，ManagedStopFilterFactory实现确定了/schema/analysis/stopwords路径的一部分，这是合理的，因为这是一个由架构定义的分析组件。  
接下来是使用以下过滤器的字段类型：  
```
&lt;filter class="solr.ManagedStopFilterFactory"
        managed="french" /&gt;
```
将会解决路径：  
```
/solr/techproducts/schema/analysis/stopwords/french
```
所以现在让我们看一下这个API，从一个简单的GET请求开始：  
```
curl "http://localhost:8983/solr/techproducts/schema/analysis/stopwords/english"
```
假设您将此请求发送到Solr，则响应正文是一个JSON文档：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":1
  },
  "wordSet":{
    "initArgs":{"ignoreCase":true},
    "initializedOn":"2014-03-28T20:53:53.058Z",
    "managedList":[
      "a",
      "an",
      "and",
      "are",
       ]
  }
}
```
该sample_techproducts_configs configset附带了一组预置的托管stop词，但是您应该只使用此文件中使用API进行交互，而不是直接进行编辑。  
  
在这个回应中应该突出的一件事：它包含了一些词汇的managedList以及initArgs。这是这个框架中的一个重要概念 - 被管理的资源通常具有配置和数据。对于stop词，唯一的配置参数是一个布尔值，用于决定是否在stop词过滤期间忽略标记的大小写（ignoreCase = true | false）。数据是一个单词列表，它表示为响应中名为 managedList 的 JSON 数组。  
现在，我们使用HTTP PUT为英stop词列表添加一个新词：  
```
curl -X PUT -H 'Content-type:application/json' --data-binary '["foo"]' "http://localhost:8983/solr/techproducts/schema/analysis/stopwords/english"
```
在这里，我们使用curl将一个包含单个单词“foo”的JSON列表放到托管英语stop词集中。如果请求成功，Solr将返回200。您也可以将多个单词放在一个PUT请求中。  
  
您可以通过发送该单词的GET请求作为该集的子资源来测试是否存在特定的单词，例如：  
```
curl "http://localhost:8983/solr/techproducts/schema/analysis/stopwords/english/foo"
```
如果子资源（foo）存在，此请求将返回状态码200；如果不存在托管列表，则返回404。  
要删除一个停止词，您需要这样做：  
```
curl -X DELETE "http://localhost:8983/solr/techproducts/schema/analysis/stopwords/english/foo"
```
放置/POST 用于将术语添加到现有列表中, 而不是完全替换该列表。这是因为在现有列表中添加一个术语比完全替换一个列表更常见, 因此 API 更倾向于递增地添加术语, 特别是因为也支持删除各个术语。  
PUT/POST用于将词添加到现有列表，而不是完全替换列表。这是因为向现有列表添加一个词比将其全部替换为一个列表更为常见，因此 API 更倾向于递增地添加术语，特别是由于删除单个词也是受支持的。  
## <span style="font-family: inherit; font-weight: 600;">管理同义词</span>

大多数情况下，用于管理同义词的API的行为类似于用于stop词的API，除了使用单词列表之外，它也使用映射，其中映射中每个条目的值都是一个词的一组同义词。与 stop 词一样，sample_techproducts_configs configset包含一组预先构建的同义映射集，这些映射集适用于由schema.xml中的以下字段类型定义激活的示例数据：  
```
&lt;fieldType name="managed_en" positionIncrementGap="100"&gt;
  &lt;analyzer type="index"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.ManagedStopFilterFactory" managed="english" /&gt;
    &lt;filter class="solr.ManagedSynonymGraphFilterFactory" managed="english" /&gt;
    &lt;filter class="solr.FlattenGraphFilterFactory"/&gt; &lt;!-- required on index analyzers after graph filters --&gt;
  &lt;/analyzer&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.ManagedStopFilterFactory" managed="english" /&gt;
    &lt;filter class="solr.ManagedSynonymGraphFilterFactory" managed="english" /&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
要获取受管同义词的映射，请发送一个GET请求到：  
```
curl "http://localhost:8983/solr/techproducts/schema/analysis/synonyms/english"
```
此请求将返回如下所示的响应：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":3},
  "synonymMappings":{
    "initArgs":{
      "ignoreCase":true,
      "format":"solr"},
    "initializedOn":"2014-12-16T22:44:05.33Z",
    "managedMap":{
      "GB":
        ["GiB",
         "Gigabyte"],
      "TV":
        ["Television"],
      "happy":
        ["glad",
         "joyful"]}}}
```
托管的同义词在managedMap属性下返回，其中包含一个JSON映射，其中每个条目的值是一个词的一组同义词，例如上面的示例中的“happy”具有同义词“glad”和“joyful”。  
  
要添加一个新的同义词映射，您可以PUT / POST一个映射，如：  
```
curl -X PUT -H 'Content-type:application/json' --data-binary '{"mad":["angry","upset"]}' "http://localhost:8983/solr/techproducts/schema/analysis/synonyms/english"
```
如果PUT请求成功，API将返回状态码200。要确定特定词的同义词，请对子资源发送GET请求，例如：/schema/analysis/synonyms/english/mad，将返回["angry","upset"]。  
您也可以PUT一个对称同义词列表，它将被扩展成列表中每个词的映射。例如，可以使用 JSON 列表语法而不是映射来PUT下面的对称同义词列表：  
```
curl -X PUT -H 'Content-type:application/json' --data-binary '["funny", "entertaining", "whimiscal", "jocular"]' "http://localhost:8983/solr/techproducts/schema/analysis/synonyms/english"
```
请注意，扩展是在处理PUT请求时执行的，因此基础持久化状态仍然是托管映射。因此，如果在发送先前的PUT请求之后，您做了GET /schema/analysis/synonyms/english/jocular，那么您将收到一个包含["funny", "entertaining", "whimiscal"]的列表。一旦使用列表创建同义词映射，每个词都必须单独管理。  
  
最后，您可以通过向托管端点发送DELETE请求来删除映射。  

## 应用托管资源更改

在Solr集合（或单一服务器模式下的Solr核心）重新加载之前，通过此REST API对受管资源所做的更改不会应用于活动的Solr组件。  
  
例如：在添加或删除stop词之后，必须在更改变为活动状态之前重新加载核心/集合；相关的API：CoreAdmin API和Collections API。  
在分布式模式下运行时，这种方法是必需的，这样我们才能确保同时对集合中的所有内核进行更改，从而保证行为的一致性和可预测性。不言而喻，您不希望您的一个副本使用不同于其他的stop词或同义词组。  
这个apply-changes-at-reload方法的一个细微结果就是，一旦您对API进行了修改，就无法读取活动的数据。换句话说，API从API的角度返回最新的数据，这可能与当前Solr组件使用的数据不同。  
然而，这个API实现的目的是在做出更改之后，在短时间内使用重新加载来应用这些更改，以使API返回的数据与服务器中的活动不同的时间可以忽略不计。  
注意：更改诸如 stop 字词和同义词映射之类的内容通常需要索引现有文档 （如果被索引时间分析器使用）。RestManager 框架并没有保护你, 它只是使得有可能以编程方式建立一组stop字、同义词等。  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">RestManager端点</span>

关于注册的ManagedResources的元数据可以使用每个集合的/schema/managed端点。  
假设您有managed_enschema.xml中定义的字段类型，那么向以下资源发送GET请求将返回关于RestManager管理哪些与模式相关的资源的元数据：  
```
curl "http://localhost:8983/solr/techproducts/schema/managed"
```
响应正文是一个JSON文档，其中包含有关/schema根目录下的受管资源的元数据：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":3
  },
  "managedResources":[
    {
      "resourceId":"/schema/analysis/stopwords/english",
      "class":"org.apache.solr.rest.schema.analysis.ManagedWordSetResource",
      "numObservers":"1"
    },
    {
      "resourceId":"/schema/analysis/synonyms/english",
      "class":"org.apache.solr.rest.schema.analysis.ManagedSynonymGraphFilterFactory$SynonymManager",
      "numObservers":"1"
    }
  ]
}
```
您还可以在配置使用这些资源的任何内容之前，使用PUT/POST创建新的托管资源到相应的URL。  
  
例如，假设我们想要建立一组德语停用词。在我们开始添加停用词之前，我们需要创建端点：  
```
/solr/techproducts/schema/analysis/stopwords/german
```
要创建此端点，请将以下PUT/POST请求发送给我们希望创建的端点：  
```
curl -X PUT -H 'Content-type:application/json' --data-binary \
'{"class":"org.apache.solr.rest.schema.analysis.ManagedWordSetResource"}' \
"http://localhost:8983/solr/techproducts/schema/analysis/stopwords/german"
```
如果请求成功，Solr将以状态码200回应。实际上，此操作在RestManager中为托管资源注册新端点。从这里开始，您可以开始添加德语停用词，如上所述：  
```
curl -X PUT -H 'Content-type:application/json' --data-binary '["die"]' \
"http://localhost:8983/solr/techproducts/schema/analysis/stopwords/german"
```
对于大多数用户来说，以这种方式创建资源不应该是必需的，因为托管资源是在配置时自动创建的。  
  
但是，如果受管资源不再由Solr组件使用，则可能需要显式删除受管资源。  
例如，我们上面创建的德语的托管资源可以被删除，因为没有使用它的Solr组件，而英语停用词的托管资源不能被删除，因为在schema.xml中声明了一个标记过滤器使用它。  
```
curl -X DELETE "http://localhost:8983/solr/techproducts/schema/analysis/stopwords/german"
```
