## Solr配置API：Config API 
<div class="content-intro view-box ">Config API可以使用类似REST的API调用来处理您的solrconfig.xml的各个方面。  
  
此功能默认启用，并且在SolrCloud和独立模式下的工作方式类似。许多通常编辑的属性（如缓存大小和提交设置）和请求处理程序定义可以使用此API进行更改。  
使用此API时，solrconfig.xml不会更改。相反，所有编辑的配置都存储在一个名为configoverlay.json的文件中。该configoverlay.json中值覆盖solrconfig.xml中的值。  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">配置API入口点</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#config-api-entry-points"/>

    - /config：检索或修改配置。GET检索和POST执行命令
    - /config/overlay：单独检索configoverlay.json细节
    - /config/params：允许创建参数集，可以覆盖或取代在solrconfig.xml中定义的参数。请参阅请求参数API部分以获取更多详细信息。

## 检索配置<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#retrieving-the-config"/>

所有配置项，都可以通过向/config端点发送GET请求被检索 - 其结果将是configoverlay.json 与在 solrconfig.xml 中合并设置而产生的有效配置：  
```
curl http://localhost:8983/solr/techproducts/config
```
如果要将返回的结果限制到顶级部分，例如query，requestHandler或者updateHandler，那么请将该节的名称追加到斜线之后的/config端点。例如，检索所有请求处理程序的配置：  
```
curl http://localhost:8983/solr/techproducts/config/requestHandler
```
为了进一步限制返回的结果为顶层部分中的单个组件，请使用componentName请求参数，例如返回/select请求处理程序的配置：  
```
curl http://localhost:8983/solr/techproducts/config/requestHandler?componentName=/select
```

## 修改配置的命令<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#commands-to-modify-the-config"/>

此API使用特定的命令来告诉Solr要添加到configoverlay.json的属性或类型的属性。这些命令作为与请求一起发送的数据的一部分传递。  
  
配置命令分为3个不同的部分，它们在 solrconfig. xml 中操作各种数据结构，这些都在下面的内容中进行描述。  

    - 通用属性
    
    - 组件
    
    - 用户定义的属性
    

### 通用属性的命令
常见的属性是那些经常需要在Solr实例中自定义的属性。它们使用两个命令进行操作：  

    - set-property：设置一个众所周知的属性。属性的名称是预定义的并且是固定的。如果该属性已经设置，该命令将覆盖以前的设置。
    - unset-property：使用该set-property命令删除一个属性集。

使用这些命令配置的属性是预定义的，并在下面列出。这些属性的名称是从它们在 solrconfig. xml 中找到的 xml 路径派生的。  
  

    - updateHandler.autoCommit.maxDocs
    - updateHandler.autoCommit.maxTime
    - updateHandler.autoCommit.openSearcher
    - updateHandler.autoSoftCommit.maxDocs
    - updateHandler.autoSoftCommit.maxTime
    - updateHandler.commitWithin.softCommit
    - updateHandler.indexWriter.closeWaitsForMerges
    - query.filterCache.class
    - query.filterCache.size
    - query.filterCache.initialSize
    - query.filterCache.autowarmCount
    - query.filterCache.regenerator
    - query.queryResultCache.class
    - query.queryResultCache.size
    - query.queryResultCache.initialSize
    - query.queryResultCache.autowarmCount
    - query.queryResultCache.regenerator
    - query.documentCache.class
    - query.documentCache.size
    - query.documentCache.initialSize
    - query.documentCache.autowarmCount
    - query.documentCache.regenerator
    - query.fieldValueCache.class
    - query.fieldValueCache.size
    - query.fieldValueCache.initialSize
    - query.fieldValueCache.autowarmCount
    - query.fieldValueCache.regenerator
    - query.useFilterForSortedQuery
    - query.queryResultWindowSize
    - query.queryResultMaxDocCached
    - query.enableLazyFieldLoading
    - query.boolToFilterOptimizer
    - query.maxBooleanClauses
    - jmx.agentId
    - jmx.serviceUrl
    - jmx.rootName
    - requestDispatcher.handleSelect
    - requestDispatcher.requestParsers.multipartUploadLimitInKB
    - requestDispatcher.requestParsers.formdataUploadLimitInKB
    - requestDispatcher.requestParsers.enableRemoteStreaming
    - requestDispatcher.requestParsers.addHttpRequestToContext

### 自定义处理程序和本地组件的命令<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#commands-for-custom-handlers-and-local-components"/>

自定义请求处理程序、搜索组件和其他类型的本地化Solr组件（例如自定义查询解析器、更新处理器等）可以使用特定命令添加、更新和删除，以便修改组件。  
  
语法在每种情况下都是类似的：add-&lt;component-name&gt;，update-&lt;component-name&gt;，和delete-&lt;component-name&gt;。命令名不区分大小写，因此Add-RequestHandler，ADD-REQUESTHANDLER和add-requesthandler都是等效的。  
在每种情况下，add- 命令都会将新配置添加到configoverlay.json，这将覆盖solrconfig.xml组件中的任何其他设置；update- 命令覆盖configoverlay.json中的现有设置；delete-命令从configoverlay.json中删除设置。  
从configoverlay.json删除的设置不会从solrconfig.xml中删除。  
可用命令的完整列表如下所示：  
这些命令是最常用的：  

    - add-requesthandler
    - update-requesthandler
    - delete-requesthandler
    - add-searchcomponent
    - update-searchcomponent
    - delete-searchcomponent
    - add-initparams
    - update-initparams
    - delete-initparams
    - add-queryresponsewriter
    - update-queryresponsewriter
    - delete-queryresponsewriter

<h4>这些命令允许向Solr注册更高级的定制：  
</h4>
    - add-queryparser
    - update-queryparser
    - delete-queryparser
    - add-valuesourceparser
    - update-valuesourceparser
    - delete-valuesourceparser
    - add-transformer
    - update-transformer
    - delete-transformer
    - add-updateprocessor
    - update-updateprocessor
    - delete-updateprocessor
    - add-queryconverter
    - update-queryconverter
    - delete-queryconverter
    - add-listener
    - update-listener
    - delete-listener
    - add-runtimelib
    - update-runtimelib
    - delete-runtimelib

有关使用这些命令的示例，请参见下面的“创建和更新请求处理程序”一节。  

#### 什么是updateRequestProcessorChain？<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#what-about-updaterequestprocessorchain"/>

配置API不允许您创建或编辑updateRequestProcessorChain元素。但是，可以创建updateProcessor条目并按名称使用它们来创建链。  
  
例如：  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json' -d '{
"add-updateprocessor" : { "name" : "firstFld",
                          "class": "solr.FirstFieldValueUpdateProcessorFactory",
                          "fieldName":"test_s"}}'
```
您可以直接在您的请求中使用此功能，方法是在updateRequestProcessorChain中的特定更新处理器中添加一个名为processor=firstFld的参数。  
  

### 用户定义属性的命令<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#commands-for-user-defined-properties"/>

Solr允许用户使用占位符格式：${variable_name:default_val}对solrconfig.xml进行模板化。例如，您可以使用系统属性，如：-Dvariable_name= my_customvalue来设置这些值。使用这些命令可以在运行时实现相同的功能：  
  

    - set-user-property：设置用户定义的属性。如果该属性已经设置，则该命令将覆盖以前的设置。
    - unset-user-property：删除用户定义的属性。

请求的结构类似于使用其他命令的请求的结构，格式为"command":{"variable_name":"property_value"}。如有需要，您可以一次添加多个变量。  
  
有关用户定义属性的更多信息，请参阅core.properties中的“用户定义属性”部分。  
有关如何使用此类型命令的示例，另请参阅下面的“创建和更新用户定义的属性”部分。  

## 如何将solrconfig.xml属性映射到JSON<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#how-to-map-solrconfig-xml-properties-to-json"/>

通过使用此API，您将生成在solrconfig.xml中定义的属性的JSON表示。为了理解API如何表示属性，我们来看几个例子。  
  
以下是一个请求处理程序在 solrconfig 中的样子：  
```
&lt;requestHandler name="/query" class="solr.SearchHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="echoParams"&gt;explicit&lt;/str&gt;
    &lt;int name="rows"&gt;10&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
使用Config API定义的相同请求处理程序如下所示：  
```
{
  "add-requesthandler":{
    "name":"/query",
    "class":"solr.SearchHandler",
    "defaults":{
      "echoParams":"explicit",
      "rows": 10
    }
  }
}
```
solrconfig.xml中的QueryElevationComponent searchComponent 如下所示：  
```
&lt;searchComponent name="elevator" class="solr.QueryElevationComponent" &gt;
  &lt;str name="queryFieldType"&gt;string&lt;/str&gt;
  &lt;str name="config-file"&gt;elevate.xml&lt;/str&gt;
&lt;/searchComponent&gt;
```
与Config API相同的 searchComponent：  
```
{
  "add-searchcomponent":{
    "name":"elevator",
    "class":"QueryElevationComponent",
    "queryFieldType":"string",
    "config-file":"elevate.xml"
  }
}
```
使用Config API删除searchComponent：  
```
{
  "delete-searchcomponent":"elevator"
}
```
一个简单的高亮在solrconfig.xml中看起来是像下面这样（例如被截断的空间）：  
```
&lt;searchComponent class="solr.HighlightComponent" name="highlight"&gt;
    &lt;highlighting&gt;
      &lt;fragmenter name="gap"
                  default="true"
                  class="solr.highlight.GapFragmenter"&gt;
        &lt;lst name="defaults"&gt;
          &lt;int name="hl.fragsize"&gt;100&lt;/int&gt;
        &lt;/lst&gt;
      &lt;/fragmenter&gt;
      &lt;formatter name="html"
                 default="true"
                 class="solr.highlight.HtmlFormatter"&gt;
        &lt;lst name="defaults"&gt;
          &lt;str name="hl.simple.pre"&gt;&lt;![CDATA[&lt;em&gt;]]&gt;&lt;/str&gt;
          &lt;str name="hl.simple.post"&gt;&lt;![CDATA[&lt;/em&gt;]]&gt;&lt;/str&gt;
        &lt;/lst&gt;
      &lt;/formatter&gt;
      &lt;encoder name="html" class="solr.highlight.HtmlEncoder" /&gt;
...
    &lt;/highlighting&gt;
```
与Config API相同的高亮：  
```
{
    "add-searchcomponent": {
        "name": "highlight",
        "class": "solr.HighlightComponent",
        "": {
            "gap": {
                "default": "true",
                "name": "gap",
                "class": "solr.highlight.GapFragmenter",
                "defaults": {
                    "hl.fragsize": 100
                }
            }
        },
        "html": [{
            "default": "true",
            "name": "html",
            "class": "solr.highlight.HtmlFormatter",
            "defaults": {
                "hl.simple.pre": "before-",
                "hl.simple.post": "-after"
            }
        }, {
            "name": "html",
            "class": "solr.highlight.HtmlEncoder"
        }]
    }
}
```
在solrconfig.xml以下位置设置autoCommit属性：  
```
&lt;autoCommit&gt;
  &lt;maxTime&gt;15000&lt;/maxTime&gt;
  &lt;openSearcher&gt;false&lt;/openSearcher&gt;
&lt;/autoCommit&gt;
```
使用Config API定义相同的属性：  
```
{
  "set-property": {
    "updateHandler.autoCommit.maxTime":15000,
    "updateHandler.autoCommit.openSearcher":false
  }
}
```

### 为Config API的组件命名<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#name-components-for-the-config-api"/>

Config API始终允许通过名称更改任何组件的配置。然而，一些配置，如listener或initParams不需要 solrconfig. xml 中的名称。为了能够update和delete在configoverlay.json中相同的项目，必须使用 name 属性。  

## <span style="font-family: inherit;">Config API</span>示例<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#config-api-examples"/>

### 创建和更新通用属性<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#creating-and-updating-common-properties"/>

此更改将 query.filterCache.autowarmCount 设置为1000项，并取消设置 query.filterCache.size。  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json' -d'{
    "set-property" : {"query.filterCache.autowarmCount":1000},
    "unset-property" :"query.filterCache.size"}'
```
使用/config/overlay端点，您可以使用如下请求验证更改：  
```
curl http://localhost:8983/solr/gettingstarted/config/overlay?omitHeader=true
```
您应该会得到这样的回应：  
```
{
  "overlay":{
    "znodeVersion":1,
    "props":{"query":{"filterCache":{
          "autowarmCount":1000,
          "size":25}}}}}
```

### 创建和更新请求处理程序<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#creating-and-updating-request-handlers"/>

要创建请求处理程序，我们可以使用以下add-requesthandler命令：  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json'  -d '{
  "add-requesthandler" : {
    "name": "/mypath",
    "class":"solr.DumpRequestHandler",
    "defaults":{ "x":"y" ,"a":"b", "rows":10 },
    "useParams":"x"
  }
}'
```
调用新的请求处理程序来检查它是否被注册：  
```
curl http://localhost:8983/solr/techproducts/mypath?omitHeader=true
```
您应该会看到下面的输出：  
```
{
  "params":{
    "indent":"true",
    "a":"b",
    "x":"y",
    "rows":"10"},
  "context":{
    "webapp":"/solr",
    "path":"/mypath",
    "httpMethod":"GET"}}
```
要更新请求处理程序，您应该使用以下update-requesthandler命令：  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json'  -d '{
  "update-requesthandler": {
    "name": "/mypath",
    "class":"solr.DumpRequestHandler",
    "defaults": {"x":"new value for X", "rows":"20"},
    "useParams":"x"
  }
}'
```
作为另一个例子，我们将创建另一个请求处理程序，这次将“terms”组件添加为定义的一部分：  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json' -d '{
  "add-requesthandler": {
    "name": "/myterms",
    "class":"solr.SearchHandler",
    "defaults": {"terms":true, "distrib":false},
    "components": [ "terms" ]
  }
}'
```

### 创建和更新用户定义的属性<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#creating-and-updating-user-defined-properties"/>

这个命令设置一个用户属性。  
```
curl http://localhost:8983/solr/techproducts/config -H'Content-type:application/json' -d '{
    "set-user-property" : {"variable_name":"some_value"}}'
```
我们依然可以使用/config/overlay端点来验证所做的更改：  
```
curl http://localhost:8983/solr/techproducts/config/overlay?omitHeader=true
```
我们希望看到这样的输出：  
```
{"overlay":{
   "znodeVersion":5,
   "userProps":{
     "variable_name":"some_value"}}
}
```
要取消设置变量，请执行如下命令：  
```
curl http://localhost:8983/solr/techproducts/config -H'Content-type:application/json' -d '{"unset-user-property" : "variable_name"}'
```

## Config API的工作原理<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#how-the-config-api-works"/>

每个内核都监视与该内核一起使用的配置集的ZooKeeper目录。然而，在独立模式下，没有监视（因为ZooKeeper没有运行）。如果同一个节点中有多个核心使用相同的配置集，则只使用一个ZooKeeper监视。例如，如果一个核心使用了configset'myconf'，那么节点就会监视/configs/myconf。每个通过API执行的写入操作都会“触摸”目录（设置一个空字节[]来触发监视），并通知所有监视器。每个内核会检查Schema文件，solrconfig.xml或者configoverlay.json通过比较znode版本进行修改，如果修改，则重新加载内核。  
  
如果params.json修改，则params对象只是在没有核心重新加载的情况下更新（请参阅请求参数API部分了解更多有关params.json的信息）。  

### 空命令<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#empty-command"/>

如果一个空的命令发送到/config端点，那么使用这个配置集在所有内核上触发监视。例如：  
```
curl http://localhost:8983/solr/techproducts/config -H'Content-type:application/json' -d '{}'
```
直接编辑任何文件而不“接触”该目录将不会使其对所有节点可见。  
  
通过SolrCore#registerConfListener()注册监听器，组件可以监视configset “touch”事件。  

### 听取配置更改<a href="http://lucene.apache.org/solr/guide/7_0/config-api.html#listening-to-config-changes"/>

任何组件都可以使用以下方法注册侦听器  
```
SolrCore#addConfListener(Runnable listener)
```
通知配置更改。如果修改的文件导致核心重新加载（即configoverlay.xml或架构），这不是非常有用。组件可以使用它来重新加载他们感兴趣的文件。  
