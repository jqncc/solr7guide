## Solr请求参数API 
<div class="content-intro view-box ">请求参数API允许创建参数集（又名 paramsets），它可以覆盖或取代在solrconfig.xml中定义的参数。  
  
使用此API定义的参数集可以用于对Solr的请求，也可以直接在 solrconfig.xml 请求处理程序定义中引用。  
它实际上是Config API的另一个端点，而不是一个单独的API，并且具有不同的命令。它不会替换或修改solrconfig.xml的任何部分，而是提供处理请求中使用的参数的另一种方法。它的行为方式与Config API相同，通过将参数存储在另一个将在运行时使用的文件中。在这种情况下，参数存储在一个名为params.json的文件中。该文件保存在ZooKeeper中或独立的Solr实例的conf目录中。  
params.json查询时使用存储的设置来覆盖solrconfig.xml在某些情况下定义的设置，如下所述。  
什么时候可以使用这个功能？请参考以下的几种情况：  

    - 为了避免频繁编辑您的solrconfig.xml来更新经常更改的请求参数。
    - 在各种请求处理程序中重用参数。
    - 在请求时混合和匹配参数集。
    - 为了避免重新加载您的集合，以便进行小的参数更改。

## 请求参数端点
所有请求都被发送到Config API的/config/params端点。  

## 设置请求参数<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#setting-request-parameters"/>

要设置、取消设置或更新请求参数的请求将作为一组包含名称的地图发送。这些对象可以直接用于请求或请求处理程序定义中使用。  
  
可用的命令有：  

    - set：创建或覆盖参数集映射。
    - unset：删除一个参数集映射。
    - update：更新参数集映射。这相当于一个map.putAll（newMap）。这两个地图都合并，如果新地图具有与旧地图相同的密钥，则会被覆盖。

如有必要，您可以将这些命令混合成一个请求。  
  
每个映射都必须包含一个名称，以便以后可以在对 Solr 的直接请求或请求处理程序定义中引用它。  
在下面的例子中，我们设置了两组名为“myFacets”和“myQueries”的参数：  
```
curl http://localhost:8983/solr/techproducts/config/params -H 'Content-type:application/json'  -d '{
  "set":{
    "myFacets":{
      "facet":"true",
      "facet.limit":5}},
  "set":{
    "myQueries":{
      "defType":"edismax",
      "rows":"5",
      "df":"text_all"}}
}'
```
在上面的示例中，所有参数都等效于 solrconfig. xml 中的 "defaults"。可以添加不变量并追加如下:  
```
curl http://localhost:8983/solr/techproducts/config/params -H 'Content-type:application/json'  -d '{
  "set":{
    "my_handler_params":{
      "facet.limit":5,
      "_invariants_": {
        "facet":true,
       },
      "_appends_":{"facet.field":["field1","field2"]
     }
   }}
}'
```

## 使用请求参数和RequestHandlers<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#using-request-parameters-with-requesthandlers"/>

在上面的章节中创建了my_handler_params 参数集之后，可以像下面这样定义一个请求处理程序：  
```
&lt;requestHandler name="/my_handler" class="solr.SearchHandler" useParams="my_handler_params"/&gt;
```
它将相当于一个标准的请求处理程序定义，例如：  
```
&lt;requestHandler name="/my_handler" class="solr.SearchHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;int name="facet.limit"&gt;5&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="invariants"&gt;
    &lt;bool name="facet"&gt;true&lt;/bool&gt;
  &lt;/lst&gt;
  &lt;lst name="appends"&gt;
    &lt;arr name="facet.field"&gt;
      &lt;str&gt;field1&lt;/str&gt;
      &lt;str&gt;field2&lt;/str&gt;
    &lt;/arr&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```

### 使用请求参数 API 的隐式 RequestHandlersSolr<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#implicit-requesthandlers-with-the-request-parameters-api"/>

Solr附带有许多现成的请求处理程序，这些处理程序只能通过请求参数API进行配置，因为它们的配置不在solrconfig.xml中。在配置隐式请求处理程序时，请参阅Implicit RequestHandlers以使用paramset。  
  

### 使用RequestHandlers查看扩展参数集和有效参数
要查看扩展的 paramset 和 useParams 定义的 RequestHandler 的有效参数, 请使用 expandParams 请求参数。例如对于/export请求处理程序：  
```
curl http://localhost:8983/solr/techproducts/config/requestHandler?componentName=/export&amp;expandParams=true
```

## 查看请求参数<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#viewing-request-parameters"/>

要查看已创建的参数集，可以使用/config/params端点来读取params.json请求中的内容或使用该名称：  
```
curl http://localhost:8983/solr/techproducts/config/params
#Or use the paramset name
curl http://localhost:8983/solr/techproducts/config/params/myQueries
```

## useParams参数<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#the-useparams-parameter"/>

发出请求时，useParams参数将应用发送给请求的请求参数。这是在请求时间转换为实际参数。  
例如（使用我们在前面例子中设置的名字，请用您自己的名字替换）：  
```
http://localhost/solr/techproducts/select?useParams=myQueries
```
可以在相同的请求中传递多个参数集。例如：  
```
http://localhost/solr/techproducts/select?useParams=myFacets,myQueries
```
在上面的例子中，参数集“myQueries”被应用在“myFacets”的顶部。因此，“myQueries”中的值优先于“myFacets”中的值。此外，请求中传递的任何值都优先于useParams参数。这就像在 solrconfig. xml 中的 &lt;requestHandler&gt; 定义中所指定的 "defaults"。  
  
参数集可以直接在请求处理程序定义中使用，如下所示。请注意，即使请求包含 useParams，也始终应用指定的 useParams。  
```
&lt;requestHandler name="/terms" class="solr.SearchHandler" useParams="myQueries"&gt;
  &lt;lst name="defaults"&gt;
    &lt;bool name="terms"&gt;true&lt;/bool&gt;
    &lt;bool name="distrib"&gt;false&lt;/bool&gt;
  &lt;/lst&gt;
  &lt;arr name="components"&gt;
    &lt;str&gt;terms&lt;/str&gt;
  &lt;/arr&gt;
&lt;/requestHandler&gt;
```
总而言之，安照以下顺序应用参数：  

    - 在solrconfig.xml的&lt;invariants&gt;中定义的参数。
    - 应用于params.json中的invariants的参数，并且在requesthandler定义中指定或者甚至在请求中。
    - 直接在请求中定义的参数。
    - 在请求中定义的参数集，其顺序与 useParams 一起列出。
    - 在params.json中定义的参数集已在请求处理程序中定义。
    - 在solrconfig.xml中的&lt;defaults&gt;中定义的参数。

## 公共API<a href="http://lucene.apache.org/solr/guide/7_0/request-parameters-api.html#public-apis"/>

可以使用SolrConfig#getRequestParams()方法访问RequestParams对象。每个参数可以通过使用RequestParams#getRequestParams(String name)方法的名称来访问。  
  

## 使用请求参数API示例

    Solr 的 "films" 示例演示了参数 API 的使用。您可以在 Solr 安装中使用此示例 (在example/films目录中)，或在 https://github.com/apache/lucene-solr/tree/master/solr/example/films 中查看 Apache GitHub 镜像中的文件。  
