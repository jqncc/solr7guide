## 在Solr中学习排序（LTR） 
<div class="content-intro view-box ">通过学习排序（或 LTR）contrib 模块，您可以在 Solr 中配置和运行机器学习排序模型。
      
  
该模块还支持 Solr 内部的特征提取。您需要在 Solr 之外做的唯一事情就是训练你自己的排序模型。  

## 学习排序概念

学习排序包含了三个部分，分别是：重新排序、学习排序模型以及训练模式，接下来我们将一一介绍。  

### 重新排序

通过重新排序，您可以运行一个简单的查询来匹配文档，然后使用来自不同的、更复杂的查询中的分数重新排列前 N 个文档。本页面描述了 LTR 复杂查询的使用，Solr 发行版中包含的其他等级查询的信息可以在查询重新排序页面上找到。  

### 学习排序模型

在信息检索系统中，学习排序被用来排名使用经过训练的机器学习模型来检索前 N 个文档。希望这种复杂的模型可以比 TF-IDF 或 BM25 等标准排序函数做出更细致的排序决定。  
排序模型计算用于重新排列文档的分数。不管任何特定的算法或实现，排序模型的计算可以使用三种类型的输入：  

    - 表示评分算法的参数
    - 表示文档被评分的功能
    - 表示要对其进行评分的查询的功能


#### 特征

特征是一个值，一个数字，表示被评分的文件的数量或质量，或者是要对其进行记录的查询。例如，文档通常具有“新近度”质量，“过去购买次数”可能是作为搜索查询的一部分传递给 Solr 的数量。
      
  

#### 正规化

一些排名模式预计在特定的规模的功能。标准化器可用于将任意特征值转换为标准化值，例如：0..1 或 0..100 的标度。
      
  

### 训练模式


#### 函数工程

LTR contrib 模块包含几个函数类以及对自定义功能的支持。每个函数类的 javadoc 都包含一个例子来说明该类的使用方法。函数工程本身的过程完全取决于您的领域专业知识和创造力。
      
  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">函数</th>
            <th style="text-align: center;">类</th>
            <th style="text-align: center;">示例参数</th>
            <th style="text-align: center;">额外的函数信息</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">字段长度  
            </td>
            <td>
                <p style="text-align: center;">FieldLengthFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"field":"title"}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">尚未支持  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">字段值  
            </td>
            <td>
                <p style="text-align: center;">FieldValueFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"field":"hits"}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">尚未支持  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">原始分数  
            </td>
            <td>
                <p style="text-align: center;">OriginalScoreFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">不适用  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">solr 查询  
            </td>
            <td>
                <p style="text-align: center;">SolrFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"q":"{!func}</code> <code>recip(ms(NOW,last_modified)</code><code>,3.16e-11,1,1)"}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">支持的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">solr 过滤器查询  
            </td>
            <td>
                <p style="text-align: center;">SolrFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"fq":["{!terms f=category}book"]}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">支持的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">solr 查询 + 过滤器查询  
            </td>
            <td>
                <p style="text-align: center;">SolrFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"q":"{!func}</code> <code>recip(ms(NOW,last_modified),</code><code>3.16e-11,1,1)",</code> <code>"fq":["{!terms f=category}book"]}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">支持的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">值  
            </td>
            <td>
                <p style="text-align: center;">ValueFeature  
            </td>
            <td>
                <p style="text-align: center;"><code>{"value":"${userFromMobile}","required":true}</code>
                  
            </td>
            <td>
                <p style="text-align: center;">支持的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">（自定义）  
            </td>
            <td>
                <p style="text-align: center;">（自定义类扩展函数）  
            </td>
            <td/>
            <td/>
        </tr>
    </tbody>
</table>
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">正规化</th>
            <th style="text-align: center;">类</th>
            <th style="text-align: center;">示例参数</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">Identity
                      
                  
            </td>
            <td>
                <p style="text-align: center;">IdentityNormalizer  
            </td>
            <td>
                <p style="text-align: center;"><code>{}</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">MINMAX  
            </td>
            <td>
                <p style="text-align: center;">MinMaxNormalizer  
            </td>
            <td>
                <p style="text-align: center;"><code>{"min":"0", "max":"50" }</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">Standard
                      
                  
            </td>
            <td>
                <p style="text-align: center;">StandardNormalizer  
            </td>
            <td>
                <p style="text-align: center;"><code>{"avg":"42","std":"6"}</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">（自定义）  
            </td>
            <td>
                <p style="text-align: center;">（自定义类扩展 Normalizer）  
            </td>
            <td/>
        </tr>
    </tbody>
</table>

#### 函数提取

LTR contrib 模块包含一个[函数转换器]，用于支持函数值的计算和返回以便进行函数提取，特别是当您还没有真正的重新排序模型时。  

#### 函数选择和模型训练

函数选择和模型训练在离线和 Solr 之外进行。ltr contrib 模块支持两种广义形式的模型以及自定义模型。每个模型类的 javadoc 包含一个示例来说明该类的配置。以 JSON 文件的形式，可以使用提供的 REST API 直接将经过培训的一个或多个模型（例如针对不同客户地区的不同模型）上传到 Solr。
      
  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">一般形式</th>
            <th style="text-align: center;">类</th>
            <th style="text-align: center;">具体的例子</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">Linear
                      
                  
            </td>
            <td>
                <p style="text-align: center;">LinearModel
                      
                  
            </td>
            <td>
                <p style="text-align: center;">RankSVM, Pranking
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">Multiple Additive Trees
                      
                  
            </td>
            <td>
                <p style="text-align: center;">MultipleAdditiveTreesModel  
            </td>
            <td>
                <p style="text-align: center;">LambdaMART, Gradient Boosted Regression Trees (GBRT)
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">（自定义）  
            </td>
            <td>
                <p style="text-align: center;">（自定义类扩展 LTRScoringModel）  
            </td>
            <td>
                <p style="text-align: center;">（不适用）  
            </td>
        </tr>
    </tbody>
</table>

## LTR 快速入门

Solr 附带的 "techproducts" 示例预先配置了学习排序所需的插件，但默认情况下它们是禁用的。
      
  
要启用插件，请在运行示例时指定 solr.ltr.enabled JVM 系统属性：  
```
bin/solr start -e techproducts -Dsolr.ltr.enabled=true
```


### 上传函数

要上传 /path/myFeatures.json 文件中的函数，请运行：  
```
curl -XPUT 'http://localhost:8983/solr/techproducts/schema/feature-store' --data-binary "@/path/myFeatures.json" -H 'Content-type:application/json'
```

要查看刚刚上传的函数，请在浏览器中打开以下 URL：  
```
http://localhost:8983/solr/techproducts/schema/feature-store/_DEFAULT_
```

例如：/path/myFeatures.json  
```
[
  {
    "name" : "documentRecency",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "q" : "{!func}recip( ms(NOW,last_modified), 3.16e-11, 1, 1)"
    }
  },
  {
    "name" : "isBook",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "fq": ["{!terms f=cat}book"]
    }
  },
  {
    "name" : "originalScore",
    "class" : "org.apache.solr.ltr.feature.OriginalScoreFeature",
    "params" : {}
  }
]
```


### 提取函数

要将函数作为查询的一部分提取，请添加 [features] 到 fl 参数中，例如：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;fl=id,score,%5Bfeatures%5D
```

输出 XML 将包含作为逗号分隔列表的函数值，类似于此处显示的输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":0,
    "params":{
      "q":"test",
      "fl":"id,score,[features]"}},
  "response":{"numFound":2,"start":0,"maxScore":1.959392,"docs":[
      {
        "id":"GB18030TEST",
        "score":1.959392,
        "[features]":"documentRecency=0.020893794,isBook=0.0,originalScore=1.959392"},
      {
        "id":"UTF8TEST",
        "score":1.5513437,
        "[features]":"documentRecency=0.020893794,isBook=0.0,originalScore=1.5513437"}]
  }}
```


### 上传模型

要将 /path/myModel.json 文件中的模型上传，请运行：  
```
curl -XPUT 'http://localhost:8983/solr/techproducts/schema/model-store' --data-binary "@/path/myModel.json" -H 'Content-type:application/json'
```

要查看刚刚上传的模型，请在浏览器中打开以下URL：  
```
http://localhost:8983/solr/techproducts/schema/model-store
```

例如：/path/myModel.json  
```
{
  "class" : "org.apache.solr.ltr.model.LinearModel",
  "name" : "myModel",
  "features" : [
    { "name" : "documentRecency" },
    { "name" : "isBook" },
    { "name" : "originalScore" }
  ],
  "params" : {
    "weights" : {
      "documentRecency" : 1.0,
      "isBook" : 0.1,
      "originalScore" : 0.5
    }
  }
}
```


### 运行重新查询

要重新查询查询的结果，请将该 rq 参数添加到搜索中，例如：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq=%7B!ltr%20model=myModel%20reRankDocs=100%7D&amp;fl=id,score[http://localhost:8983/solr/techproducts/query?q=test&amp;rq=\{!ltr model=myModel reRankDocs=100}&amp;fl=id,score]`
```

rq 参数的添加不会改变搜索的输出 XML。  
要获得重新排序时计算的函数值，请添加 [features] 到 fl 参数中，例如：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq=%7B!ltr%20model=myModel%20reRankDocs=100%7D&amp;fl=id,score,%5Bfeatures%5D[http://localhost:8983/solr/techproducts/query?q=test&amp;rq=\{!ltr model=myModel reRankDocs=100}&amp;fl=id,score,[features]]
```

输出 XML 将包含作为逗号分隔列表的函数值，类似于此处显示的输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":0,
    "params":{
      "q":"test",
      "fl":"id,score,[features]",
      "rq":"{!ltr model=myModel reRankDocs=100}"}},
  "response":{"numFound":2,"start":0,"maxScore":1.0005897,"docs":[
      {
        "id":"GB18030TEST",
        "score":1.0005897,
        "[features]":"documentRecency=0.020893792,isBook=0.0,originalScore=1.959392"},
      {
        "id":"UTF8TEST",
        "score":0.79656565,
        "[features]":"documentRecency=0.020893792,isBook=0.0,originalScore=1.5513437"}]
  }}
```


### 外部函数信息

该 ValueFeature 和 SolrFeature 类支持使用外部函数信息，简称 efi。  

#### 上传函数

要上传 /path/myEfiFeatures.json 文件中的函数，请运行：  
```
curl -XPUT 'http://localhost:8983/solr/techproducts/schema/feature-store' --data-binary "@/path/myEfiFeatures.json" -H 'Content-type:application/json'
```

要查看刚刚上传的函数，请在浏览器中打开以下URL：  
```
http://localhost:8983/solr/techproducts/schema/feature-store/myEfiFeatureStore
```

例如：/path/myEfiFeatures.json  
```
[
  {
    "store" : "myEfiFeatureStore",
    "name" : "isPreferredManufacturer",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : { "fq" : [ "{!field f=manu}${preferredManufacturer}" ] }
  },
  {
    "store" : "myEfiFeatureStore",
    "name" : "userAnswerValue",
    "class" : "org.apache.solr.ltr.feature.ValueFeature",
    "params" : { "value" : "${answer:42}" }
  },
  {
    "store" : "myEfiFeatureStore",
    "name" : "userFromMobileValue",
    "class" : "org.apache.solr.ltr.feature.ValueFeature",
    "params" : { "value" : "${fromMobile}", "required" : true }
  },
  {
    "store" : "myEfiFeatureStore",
    "name" : "userTextCat",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : { "q" : "{!field f=cat}${text}" }
  }
]
```

另外，您可能已经注意到该 myEfiFeatures.json 示例使用了 "store":"myEfiFeatureStore" 属性：store，请在本页面的 LTR 生命周期部分阅读更多关于函数存储的内容。  

#### 提取函数

要将 myEfiFeatureStore 函数作为查询的一部分提取，请将 efi.* 参数添加到 fl 参数的 [features] 部分，例如：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;fl=id,cat,manu,score,[features store=myEfiFeatureStore efi.text=test efi.preferredManufacturer=Apache efi.fromMobile=1]
```

```
http://localhost:8983/solr/techproducts/query?q=test&amp;fl=id,cat,manu,score,[features store=myEfiFeatureStore efi.text=test efi.preferredManufacturer=Apache efi.fromMobile=0 efi.answer=13]
```
    
#### 上传模型

    要将 /path/myEfiModel.json 文件中的模型上传，请运行：  
```
curl -XPUT 'http://localhost:8983/solr/techproducts/schema/model-store' --data-binary "@/path/myEfiModel.json" -H 'Content-type:application/json'
```

    要查看刚刚上传的模型，请在浏览器中打开以下URL：http://localhost:8983/solr/techproducts/schema/model-store  
    例如：/path/myEfiModel.json  
```
{
  "store" : "myEfiFeatureStore",
  "name" : "myEfiModel",
  "class" : "org.apache.solr.ltr.model.LinearModel",
  "features" : [
    { "name" : "isPreferredManufacturer" },
    { "name" : "userAnswerValue" },
    { "name" : "userFromMobileValue" },
    { "name" : "userTextCat" }
  ],
  "params" : {
    "weights" : {
      "isPreferredManufacturer" : 0.2,
      "userAnswerValue" : 1.0,
      "userFromMobileValue" : 1.0,
      "userTextCat" : 0.1
    }
  }
}
```

    
#### 运行重新排序查询

    要获得重新排序期间计算的函数值，请将 [features] 参数添加到 fl 参数中并且将 efi.* 参数添加到 rq 参数中，例如：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq=\{!ltr model=myEfiModel efi.text=test efi.preferredManufacturer=Apache efi.fromMobile=1}&amp;fl=id,cat,manu,score,[features]] link:[]
```

    
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq={!ltr model=myEfiModel efi.text=test efi.preferredManufacturer=Apache efi.fromMobile=0 efi.answer=13}&amp;fl=id,cat,manu,score,[features]    
```
        请注意，在 fl 参数的 [features] 部分中没有 efi. * 参数。  
        
#### 在重新排序是提取函数

        要提取 myEfiFeatureStore 函数的功能，同时仍然使用 myModel 进行重新排序：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq={!ltr model=myModel}&amp;fl=id,cat,manu,score,[features store=myEfiFeatureStore efi.text=test efi.preferredManufacturer=Apache efi.fromMobile=1]
```

        注意 rq 参数中没有 efi.* 参数（因为 myModel 没有使用 efi 函数）以及 fl 参数的 [features] 部分存在 efi.* 参数（因为 myEfiFeatureStore 包含了 efi 函数）。
              
          
        在此页面的LTR生命周期部分阅读有关模型演化的更多信息。  
        
### 训练示例

        示例训练数据和演示“训练和上传模型”脚本可以在 [Apache lucene-solr git repository](https://git-wip-us.apache.org/repos/asf?p=lucene-solr.git)（镜像在[github.com](https://github.com/apache/lucene-solr/tree/releases/lucene-solr/6.4.0/solr/contrib/ltr/example)）的 solr/contrib/ltr/example 文件夹中找到（solr/contrib/ltr/example 文件夹不是在 solr 二进制文件释放）。
              
          
        
## 安装 LTR

        LTR contrib 模块需要：dist/solr-ltr-*.jarJAR。  
        
## LTR 配置

        Learning-To-Rank（学习排序）是一个 contrib 模块，因此它的插件必须在 solrconfig.xml 配置。  
        
### 最低要求

        
            - 包括所需的 contrib JAR。请注意，默认路径是相对于 Solr 核心的，所以他们可能需要调整您的配置，或一个明确的规范是：$solr.install.dir。
```
&lt;lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-ltr-\d.*\.jar" /&gt;
```

            
            - LTR 查询解析器的声明。
```
&lt;queryParser name="ltr" class="org.apache.solr.ltr.search.LTRQParserPlugin"/&gt;
```

            
            - 配置函数值缓存。
```
&lt;cache name="QUERY_DOC_FV"
       class="solr.search.LRUCache"
       size="4096"
       initialSize="2048"
       autowarmCount="4096"
       regenerator="solr.search.NoOpRegenerator" /&gt;
```

            
            - [features] 转换器的声明。
```
&lt;transformer name="features" class="org.apache.solr.ltr.response.transform.LTRFeatureLoggerTransformerFactory"&gt;
  &lt;str name="fvCacheName"&gt;QUERY_DOC_FV&lt;/str&gt;
&lt;/transformer&gt;
```

            
        
        
### 高级选项

        
#### LTRThreadModule

        可以为查询解析器和转换器配置线程模块，以并行化函数权重的创建。有关详细信息，请参阅 LTRThreadModule javadocs。  
        
#### 函数向量定制

        函数转换器返回密集的 CSV 值，如：featureA=0.1、featureB=0.2、featureC=0.3、featureD=0.0。
              
          
        对于稀疏的 CSV 输出，例如：featureA:0.1、featureB:0.2、featureC:0.3，您可以在 solrconfig 中自定义函数记录器转换器声明，如下所示：  
```
&lt;transformer name="features" class="org.apache.solr.ltr.response.transform.LTRFeatureLoggerTransformerFactory"&gt;
  &lt;str name="fvCacheName"&gt;QUERY_DOC_FV&lt;/str&gt;
  &lt;str name="defaultFormat"&gt;sparse&lt;/str&gt;
  &lt;str name="csvKeyValueDelimiter"&gt;:&lt;/str&gt;
  &lt;str name="csvFeatureSeparator"&gt; &lt;/str&gt;
&lt;/transformer&gt;
```

        
## LTR 生命周期

        
### 函数存储

        建议您将所有函数组织到类似于命名空间的存储中：
              
          
        
            - 存储区中的函数必须唯一地命名。
            - 跨存储相同或相似的函数可以共享相同的名称。
                  
            
            - 如果未指定存储名，则将使用默认的 _DEFAULT_ 函数存储。
        
        查看所有函数存储的名称：  
```
http://localhost:8983/solr/techproducts/schema/feature-store
```

        检查 commonFeatureStore 函数存储的内容：  
```
http://localhost:8983/solr/techproducts/schema/feature-store/commonFeatureStore
```

        
### 模型

        
            - 模型使用来自一个函数存储的函数。
            - 如果未指定存储，则将使用默认的 _DEFAULT_ 函数存储。
                  
            
            - 模型不需要使用函数存储中定义的所有函数。
            - 多个模型可以使用相同的函数存储。
        
        提取 currentFeatureStore 函数的函数：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;fl=id,score,[features store=currentFeatureStore]
```

        提取 nextFeatureStore 函数的函数，同时 reranking 和 currentModel 基于 currentFeatureStore：  
```
http://localhost:8983/solr/techproducts/query?q=test&amp;rq={!ltr model=currentModel reRankDocs=100}&amp;fl=id,score,[features store=nextFeatureStore]
```

        要查看所有型号：  
```
http://localhost:8983/solr/techproducts/schema/model-store
```

        删除 currentModel 模型：  
```
curl -XDELETE 'http://localhost:8983/solr/techproducts/schema/model-store/currentModel'
```

        注意：只有在没有使用模型的情况下，才能删除函数存储。  
        删除 currentFeatureStore 函数存储：  
```
curl -XDELETE 'http://localhost:8983/solr/techproducts/schema/feature-store/currentFeatureStore'
```

        
### 应用更改

        函数存储和模型存储都是托管资源。在 Solr 集合（或单服务器模式下的 Solr 核心）重新加载之前，对托管资源所做的更改不会应用到活动的 Solr 组件。  
        
### LTR 示例

        
#### 一个函数存储，多个排序模型

        
            - leftModel 和 rightModel 都使用来自 commonFeatureStore 的函数，并且两个模型之间唯一的不同是每个函数附加的权重。
            - 使用的约定：<ul><li>commonFeatureStore.json 文件包含 commonFeatureStore 函数存储的函数；
- leftModel.json 文件包含名为 leftModel 的模型；
- rightModel.json 文件包含名为 rightModel 的模型；
- 模型的函数和权重按照名称的字母顺序进行排序，这样可以很容易地看出两个模型之间的共同点和差异。存储函数按名称按字母顺序排序，这使得查找模型中使用的函数变得很容易
</li>
        </ul>
        例如：/path/commonFeatureStore.json  
```
[
  {
    "store" : "commonFeatureStore",
    "name" : "documentRecency",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "q" : "{!func}recip( ms(NOW,last_modified), 3.16e-11, 1, 1)"
    }
  },
  {
    "store" : "commonFeatureStore",
    "name" : "isBook",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "fq": [ "{!terms f=category}book" ]
    }
  },
  {
    "store" : "commonFeatureStore",
    "name" : "originalScore",
    "class" : "org.apache.solr.ltr.feature.OriginalScoreFeature",
    "params" : {}
  }
]
```

        例如：/path/leftModel.json  
```
{
  "store" : "commonFeatureStore",
  "name" : "leftModel",
  "class" : "org.apache.solr.ltr.model.LinearModel",
  "features" : [
    { "name" : "documentRecency" },
    { "name" : "isBook" },
    { "name" : "originalScore" }
  ],
  "params" : {
    "weights" : {
      "documentRecency" : 0.1,
      "isBook" : 1.0,
      "originalScore" : 0.5
    }
  }
}
```

        例如：/path/rightModel.json  
```
{
  "store" : "commonFeatureStore",
  "name" : "rightModel",
  "class" : "org.apache.solr.ltr.model.LinearModel",
  "features" : [
    { "name" : "documentRecency" },
    { "name" : "isBook" },
    { "name" : "originalScore" }
  ],
  "params" : {
    "weights" : {
      "documentRecency" : 1.0,
      "isBook" : 0.1,
      "originalScore" : 0.5
    }
  }
}
```

        
#### 模型演变

        
            - linearModel201701 使用 featureStore201701 的函数
            - treesModel201702 使用  featureStore201702 的函数
            - linearModel201701 和 treesModel201702 他们的函数存储可以共存，而两者都是必要的。
            - 当 linearModel201701 被删除时，那么 featureStore201701 也可以被删除。
            - 使用的约定：<ul><li>&lt;store&gt;.json 文件包含 &lt;store&gt; 函数存储的函数；
- &lt;model&gt;.json 文件包含模型名称 &lt;model&gt;；
- “generation” ID（例如，YYYYMM 年 - 月）是函数存储和模型名称的一部分；
- 模型的函数和权重按照名称的字母顺序进行排序，这样可以很容易地看出两个模型之间的共同点和差异。存储函数按名称按字母顺序排序，这样可以很容易地看出两个函数存储之间的共同点和差异。
</li>
        </ul>
        例如：/path/featureStore201701.json  
```
[
  {
    "store" : "featureStore201701",
    "name" : "documentRecency",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "q" : "{!func}recip( ms(NOW,last_modified), 3.16e-11, 1, 1)"
    }
  },
  {
    "store" : "featureStore201701",
    "name" : "isBook",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "fq": [ "{!terms f=category}book" ]
    }
  },
  {
    "store" : "featureStore201701",
    "name" : "originalScore",
    "class" : "org.apache.solr.ltr.feature.OriginalScoreFeature",
    "params" : {}
  }
]
```

        例如：/path/linearModel201701.json  
```
{
  "store" : "featureStore201701",
  "name" : "linearModel201701",
  "class" : "org.apache.solr.ltr.model.LinearModel",
  "features" : [
    { "name" : "documentRecency" },
    { "name" : "isBook" },
    { "name" : "originalScore" }
  ],
  "params" : {
    "weights" : {
      "documentRecency" : 0.1,
      "isBook" : 1.0,
      "originalScore" : 0.5
    }
  }
}
```

        例如：/path/featureStore201702.json  
```
[
  {
    "store" : "featureStore201702",
    "name" : "isBook",
    "class" : "org.apache.solr.ltr.feature.SolrFeature",
    "params" : {
      "fq": [ "{!terms f=category}book" ]
    }
  },
  {
    "store" : "featureStore201702",
    "name" : "originalScore",
    "class" : "org.apache.solr.ltr.feature.OriginalScoreFeature",
    "params" : {}
  }
]
```

        例如：/path/treesModel201702.json  
```
{
  "store" : "featureStore201702",
  "name" : "treesModel201702",
  "class" : "org.apache.solr.ltr.model.MultipleAdditiveTreesModel",
  "features" : [
    { "name" : "isBook" },
    { "name" : "originalScore" }
  ],
  "params" : {
    "trees" : [
      {
        "weight" : "1",
        "root" : {
          "feature" : "isBook",
          "threshold" : "0.5",
          "left" : { "value" : "-100" },
          "right" : {
            "feature" : "originalScore",
            "threshold" : "10.0",
            "left" : { "value" : "50" },
            "right" : { "value" : "75" }
          }
        }
      },
      {
        "weight" : "2",
        "root" : {
          "value" : "-10"
        }
      }
    ]
  }
}
```

        
## 额外的 LTR 资源

        
            - 在 Austin 中的在 Lucene/Solr Revolution 2015 上的 “Solr 学习排名”：
                <ul>
                    <li>
                        幻灯片[: http://www.slideshare.net/lucidworks/learning-to-rank-in-solr-presented-by-michael-nilsson-diego-ceccarelli-bloomberg-lp](https://ssl.microsofttranslator.com/bv.aspx?from=&amp;to=zh-CHS&amp;a=%E5%B9%BB%E7%81%AF%E7%89%87%3A%20http%3A%2F%2Fwww.slideshare.net%2Flucidworks%2Flearning-to-rank-in-solr-presented-by-michael-nilsson-diego-ceccarelli-bloomberg-lp)
                          
                    
                    - 视频:[ https://www.youtube.com/watch？v=M7BKwJoh96s](https://ssl.microsofttranslator.com/bv.aspx?from=&amp;to=zh-CHS&amp;a=%E8%A7%86%E9%A2%91%3A%20https%3A%2F%2Fwww.youtube.com%2Fwatch%EF%BC%9Fv%3DM7BKwJoh96s)
                    
                
            </li>
        </ul>
