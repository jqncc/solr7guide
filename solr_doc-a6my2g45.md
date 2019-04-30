## Solr检索架构信息 
<div class="content-intro view-box ">以下端点允许您阅读如何定义架构。您可以根据需要获取 Solr 的整个架构，或者仅部分架构。  
  
如果要修改架构，请参阅上一节[修改架构](https://www.w3cschool.cn/solr_doc/solr_doc-8v5b2g40.html)。  

## 检索整个架构

```
GET /collection/schema
```

### 检索架构参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    

查询参数  
查询参数应该在'？'之后添加到 API 请求中。  

**wt**
    
        定义响应的格式。选项是 <strong>json</strong>，<strong>xml </strong>或 <strong>schema.xml</strong>。如果未指定，则默认返回 JSON。  
    


### 检索架构响应

输出内容：输出将包括所请求格式（JSON 或 XML）的所有字段、字段类型、动态规则和复制字段规则。架构名称和版本也包括在内。  

### 检索架构示例

用 JSON 获取整个架构。  
```
curl http://localhost:8983/solr/gettingstarted/schema
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":5},
  "schema":{
    "name":"example",
    "version":1.5,
    "uniqueKey":"id",
    "fieldTypes":[{
        "name":"alphaOnlySort",
        "class":"solr.TextField",
        "sortMissingLast":true,
        "omitNorms":true,
        "analyzer":{
          "tokenizer":{
            "class":"solr.KeywordTokenizerFactory"},
          "filters":[{
              "class":"solr.LowerCaseFilterFactory"},
            {
              "class":"solr.TrimFilterFactory"},
            {
              "class":"solr.PatternReplaceFilterFactory",
              "replace":"all",
              "replacement":"",
              "pattern":"([^a-z])"}]}}],
    "fields":[{
        "name":"_version_",
        "type":"long",
        "indexed":true,
        "stored":true},
      {
        "name":"author",
        "type":"text_general",
        "indexed":true,
        "stored":true},
      {
        "name":"cat",
        "type":"string",
        "multiValued":true,
        "indexed":true,
        "stored":true}],
    "copyFields":[{
        "source":"author",
        "dest":"text"},
      {
        "source":"cat",
        "dest":"text"},
      {
        "source":"content",
        "dest":"text"},
      {
        "source":"author",
        "dest":"author_s"}]}}
```
用 XML 获取整个架构。  
```
curl http://localhost:8983/solr/gettingstarted/schema?wt=xml
```
```
&lt;response&gt;
&lt;lst name="responseHeader"&gt;
  &lt;int name="status"&gt;0&lt;/int&gt;
  &lt;int name="QTime"&gt;5&lt;/int&gt;
&lt;/lst&gt;
&lt;lst name="schema"&gt;
  &lt;str name="name"&gt;example&lt;/str&gt;
  &lt;float name="version"&gt;1.5&lt;/float&gt;
  &lt;str name="uniqueKey"&gt;id&lt;/str&gt;
  &lt;arr name="fieldTypes"&gt;
    &lt;lst&gt;
      &lt;str name="name"&gt;alphaOnlySort&lt;/str&gt;
      &lt;str name="class"&gt;solr.TextField&lt;/str&gt;
      &lt;bool name="sortMissingLast"&gt;true&lt;/bool&gt;
      &lt;bool name="omitNorms"&gt;true&lt;/bool&gt;
      &lt;lst name="analyzer"&gt;
        &lt;lst name="tokenizer"&gt;
          &lt;str name="class"&gt;solr.KeywordTokenizerFactory&lt;/str&gt;
        &lt;/lst&gt;
        &lt;arr name="filters"&gt;
          &lt;lst&gt;
            &lt;str name="class"&gt;solr.LowerCaseFilterFactory&lt;/str&gt;
          &lt;/lst&gt;
          &lt;lst&gt;
            &lt;str name="class"&gt;solr.TrimFilterFactory&lt;/str&gt;
          &lt;/lst&gt;
          &lt;lst&gt;
            &lt;str name="class"&gt;solr.PatternReplaceFilterFactory&lt;/str&gt;
            &lt;str name="replace"&gt;all&lt;/str&gt;
            &lt;str name="replacement"/&gt;
            &lt;str name="pattern"&gt;([^a-z])&lt;/str&gt;
          &lt;/lst&gt;
        &lt;/arr&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
...
    &lt;lst&gt;
      &lt;str name="source"&gt;author&lt;/str&gt;
      &lt;str name="dest"&gt;author_s&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/arr&gt;
&lt;/lst&gt;
&lt;/response&gt;
```
以 “schema.xml” 格式获取整个架构。  
```
curl http://localhost:8983/solr/gettingstarted/schema?wt=schema.xml
```
```
&lt;schema name="example" version="1.5"&gt;
  &lt;uniqueKey&gt;id&lt;/uniqueKey&gt;
  &lt;types&gt;
    &lt;fieldType name="alphaOnlySort" class="solr.TextField" sortMissingLast="true" omitNorms="true"&gt;
      &lt;analyzer&gt;
        &lt;tokenizer class="solr.KeywordTokenizerFactory"/&gt;
        &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
        &lt;filter class="solr.TrimFilterFactory"/&gt;
        &lt;filter class="solr.PatternReplaceFilterFactory" replace="all" replacement="" pattern="([^a-z])"/&gt;
      &lt;/analyzer&gt;
    &lt;/fieldType&gt;
...
  &lt;copyField source="url" dest="text"/&gt;
  &lt;copyField source="price" dest="price_c"/&gt;
  &lt;copyField source="author" dest="author_s"/&gt;
&lt;/schema&gt;
```

## 列表字段

- GET /collection/schema/fields  
- GET /collection/schema/fields/fieldname  


### 列表字段参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    
**fieldname**
    
        特定的字段名（如果将请求限制为单个字段）。  
    

查询参数  
查询参数可以在 '？' 后添加到 API 请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回 JSON。  
    
**fl**
    
        以逗号或空格分隔的一个或多个要返回的字段的列表。如果未指定，所有字段将默认返回。  
    
**includeDynamic**
    
        如果<code>true</code>并且如果<code>fl</code>指定了查询参数或者使用了<code>fieldname</code>路径参数，则匹配的动态字段被包括在响应中并且与该<code>dynamicBase</code>属性一起标识。  
        
            如果<code>fl</code>查询参数和<code>fieldname</code>路径参数均未指定，则<code>includeDynamic</code>查询参数将被忽略。  
          
        
            如果<code>false</code>默认，匹配的动态字段将不会被返回。  
          
    
**showDefaults**
    
        如果<code>true</code>，每个字段的字段类型的所有默认字段属性都将包含在响应中（例如<code>tokenized</code>for <code>solr.TextField</code>）。如果<code>false</code>默认只包含明确指定的字段属性。  
    


### 列表字段响应

输出将包括每个字段和每个字段的任何定义的配置。定义的配置可能会因每个字段而异, 但将最小程度地包括字段名称、类型 (如果它是索引的) 和存储的。  
如果多值被定义为真或假 (最可能为真), 也将显示。有关每个参数的详细信息, 请参阅定义字段一节。  
  
  
  
输出将包括每个字段和每个字段的任何定义的配置。所定义的配置可能会因每个字段而异，但最低限度地将包括字段名称、类型 ，如果是 indexed 和 stored。  
如果 multiValued 被定义为真或假（很可能是真的），那么也将被显示。有关每个参数的更多信息，请参阅[定义字段](https://www.w3cschool.cn/solr_doc/solr_doc-n2h12g1c.html)部分。  

### 列表字段示例

获取所有字段的列表。  
```
curl http://localhost:8983/solr/gettingstarted/schema/fields
```
下面的示例输出已被截断，只显示几个字段。  
```
{
    "fields": [
        {
            "indexed": true,
            "name": "_version_",
            "stored": true,
            "type": "long"
        },
        {
            "indexed": true,
            "name": "author",
            "stored": true,
            "type": "text_general"
        },
        {
            "indexed": true,
            "multiValued": true,
            "name": "cat",
            "stored": true,
            "type": "string"
        },
"..."
    ],
    "responseHeader": {
        "QTime": 1,
        "status": 0
    }
}
```

## 列出动态字段

- GET /collection/schema/dynamicfields  
- GET /collection/schema/dynamicfields/name  


### 列出动态字段参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    
**name**
    
        动态字段规则的名称（如果将请求限制为单个动态字段规则）。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回JSON。  
    
**showDefaults**
    
        如果<code>true</code>，每个动态字段的字段类型的所有默认字段属性都将包含在响应中（例如<code>tokenized</code>for <code>solr.TextField</code>）。如果<code>false</code>默认只包含明确指定的字段属性。  
    


### 列出动态字段响应

输出将包括每个动态字段规则和每个规则的定义配置。所定义的配置可以为每个规则而变化，但将最低限度地包括动态字段的 name，type，如果是 indexed 和 stored。有关每个参数的更多信息，请参阅[动态字段](https://www.w3cschool.cn/solr_doc/solr_doc-qymb2g1g.html)部分。  

### 列出动态字段示例

获取所有动态字段声明的列表：  
```
curl http://localhost:8983/solr/gettingstarted/schema/dynamicfields
```
下面的示例输出已被截断。  
```
{
    "dynamicFields": [
        {
            "indexed": true,
            "name": "*_coordinate",
            "stored": false,
            "type": "tdouble"
        },
        {
            "multiValued": true,
            "name": "ignored_*",
            "type": "ignored"
        },
        {
            "name": "random_*",
            "type": "random"
        },
        {
            "indexed": true,
            "multiValued": true,
            "name": "attr_*",
            "stored": true,
            "type": "text_general"
        },
        {
            "indexed": true,
            "multiValued": true,
            "name": "*_txt",
            "stored": true,
            "type": "text_general"
        }
"..."
    ],
    "responseHeader": {
        "QTime": 1,
        "status": 0
    }
}
```

## 列出字段类型

- GET /collection/schema/fieldtypes  
- GET /collection/schema/fieldtypes/name  


### 列出字段类型参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    
**name**
    
        字段类型的名称（如果将请求限制为单个字段类型）。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回JSON。  
    
**showDefaults**
    
        如果<code>true</code>，每个动态字段的字段类型的所有默认字段属性都将包含在响应中（例如<code>tokenized</code>for <code>solr.TextField</code>）。如果<code>false</code>默认只包含明确指定的字段属性。  
    


### 列表字段类型响应

输出将包括每个字段类型和该类型的任何定义的配置。定义的配置可以为每个类型，但最低限度包括字段类型 name 和 class。如果查询或索引分析器、标记器或过滤器已定义，那么也将显示其他已定义的参数。有关如何配置各种类型的字段的更多信息，请参见 Solr 字段类型一节。  

### 列出字段类型示例

获取所有字段类型的列表。  
```
curl http://localhost:8983/solr/gettingstarted/schema/fieldtypes
```
下面的示例输出已被截断，以显示列表的不同部分的几个不同的字段类型。  
```
{
    "fieldTypes": [
        {
            "analyzer": {
                "class": "solr.TokenizerChain",
                "filters": [
                    {
                        "class": "solr.LowerCaseFilterFactory"
                    },
                    {
                        "class": "solr.TrimFilterFactory"
                    },
                    {
                        "class": "solr.PatternReplaceFilterFactory",
                        "pattern": "([^a-z])",
                        "replace": "all",
                        "replacement": ""
                    }
                ],
                "tokenizer": {
                    "class": "solr.KeywordTokenizerFactory"
                }
            },
            "class": "solr.TextField",
            "dynamicFields": [],
            "fields": [],
            "name": "alphaOnlySort",
            "omitNorms": true,
            "sortMissingLast": true
        },
        {
            "class": "solr.FloatPointField",
            "dynamicFields": [
                "*_fs",
                "*_f"
            ],
            "fields": [
                "price",
                "weight"
            ],
            "name": "float",
            "positionIncrementGap": "0",
        }]
}
```

## 列表复制字段

GET /collection/schema/copyfields  

### 列表复制字段参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回JSON。  
    
**source.fl**
    
        在响应中包含一个或多个copyField源字段的逗号或空格分隔列表 - 包含所有其他源字段的copyField指令将被排除在响应之外。如果没有指定，所有copyField-s将被包含在响应中。  
    
**dest.fl**
    
        一个或多个copyField目标字段的逗号或空格分隔列表，以包含在响应中。所有其他<code>dest</code>字段的copyField指令将被排除。如果没有指定，所有copyField-s将被包含在响应中。  
    


### 列表复制字段响应

输出将包括在 schema.xml 中定义的每个复制字段规则的 source 和 dest（目的地）。有关复制字段的更多信息，请参见复制字段部分。  

### 列表复制字段示例

获取所有 copyField 的列表。  
```
curl http://localhost:8983/solr/gettingstarted/schema/copyfields
```
下面的示例输出已被截断为前几个复制定义。  
```
{
    "copyFields": [
        {
            "dest": "text",
            "source": "author"
        },
        {
            "dest": "text",
            "source": "cat"
        },
        {
            "dest": "text",
            "source": "content"
        },
        {
            "dest": "text",
            "source": "content_type"
        },
    ],
    "responseHeader": {
        "QTime": 3,
        "status": 0
    }
}
```

## 显示架构名称

GET /collection/schema/name  

### 显示模式参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回JSON。  
    


### 显示模式响应

输出将只是给架构的名称。  

### 显示架构示例

获取架构名称。  
```
curl http://localhost:8983/solr/gettingstarted/schema/name
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":1},
  "name":"example"}  
```

## 显示架构版本
GET /collection/schema/version  

### 显示架构版本参数

路径参数  
   collection
    
        集合（或核心）名称。  
    

查询参数  
查询参数可以在 '？' 后添加到 API 请求中。  

<dt><span style="font-weight: normal;">wt  
</span></dt>
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回 JSON。  
    


### 显示架构版本响应

输出将只是正在使用的架构版本。  

### 显示架构版本示例

获取架构版本  
```
curl http://localhost:8983/solr/gettingstarted/schema/version
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":2},
  "version":1.5}
```

## 列出 UniqueKey

GET /collection/schema/uniquekey  

### 列出 UniqueKey 参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回 JSON。  
    


### 列出 UniqueKey 响应

输出将只包含被定义为索引的uniqueKey的字段名称。  

### 列表 UniqueKey 示例

列出 uniqueKey。  
```
curl http://localhost:8983/solr/gettingstarted/schema/uniquekey
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":2},
  "uniqueKey":"id"}
```

## 显示全局相似性

GET /collection/schema/similarity  

### 显示全局相似性参数

路径参数  

**collection**
    
        集合（或核心）名称。  
    

查询参数  
查询参数可以在'？'后添加到API请求中。  

**wt**
    
        定义响应的格式。选项是<code>json</code>或<code>xml</code>。如果未指定，则默认返回 JSON。  
    


### 显示全局相似的回应

输出将包括定义的全局相似性的类名称（如果有的话）。  

### 显示全局相似性示例

获取相似性实现。  
```
curl http://localhost:8983/solr/gettingstarted/schema/similarity
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":1},
  "similarity":{
    "class":"org.apache.solr.search.similarities.DefaultSimilarityFactory"}}
```

## 管理资源数据

该托管资源 REST API 为任何 Solr 插件提供了一种机制，用于公开应支持 CRUD (创建、读取、更新、删除) 操作的资源。根据架构中配置的字段类型和分析器，可能会存在其他的/schema/  REST API 路径。有关更多信息和示例，请参阅 "托管资源" 部分。  
