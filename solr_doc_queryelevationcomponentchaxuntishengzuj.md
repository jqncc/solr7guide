## QueryElevationComponent：查询提升组件 
<div class="content-intro view-box ">查询提升组件允许您为给定的查询配置最佳结果，而不管正常的 Lucene 评分如何。  
  
这有时被称为“赞助搜索”，“编辑提升”或“最佳匹配”。此组件将用户查询文本与配置的顶层结果映射进行匹配。文本可以是任何字符串或非字符串的 ID，只要它被索引。尽管此组件可以与任何QueryParser一起使用，但最适合与[DisMax](https://www.w3cschool.cn/solr_doc/solr_doc-vpyf2gn1.html)或[eDisMax](https://www.w3cschool.cn/solr_doc/solr_doc-usk22gqk.html)一起使用。  
查询提升组件也支持分布式搜索。  
本节中使用的所有示例配置和查询都假定您正在运行 Solr 的 techproducts 示例，如下：  
```
bin/solr -e techproducts
```

## 配置查询提升组件<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#configuring-the-query-elevation-component"/>

您可以在 solrconfig.xml 文件中配置查询提升组件。类似 QueryElevationComponent 的搜索组件可能会被添加到任何请求处理程序；这里使用了一个专门的请求处理程序。  
```
&lt;searchComponent name="elevator" class="solr.QueryElevationComponent" &gt;
  &lt;!-- pick a fieldType to analyze queries --&gt;
  &lt;str name="queryFieldType"&gt;string&lt;/str&gt;
  &lt;str name="config-file"&gt;elevate.xml&lt;/str&gt;
&lt;/searchComponent&gt;
&lt;requestHandler name="/elevate" class="solr.SearchHandler" startup="lazy"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="echoParams"&gt;explicit&lt;/str&gt;
  &lt;/lst&gt;
  &lt;arr name="last-components"&gt;
    &lt;str&gt;elevator&lt;/str&gt;
  &lt;/arr&gt;
&lt;/requestHandler&gt;
```
或者，在“查询提升组件”配置中，还可以指定以下内容以将编辑结果与“正常”结果区分开来：  
```
&lt;str name="editorialMarkerFieldName"&gt;foo&lt;/str&gt;
```
查询提升搜索组件采用以下参数：  

**queryFieldType 参数**
    
        指定应使用哪个 fieldType 来分析传入的文本。例如，使用 LowerCaseFilter 的 fieldType 可能是合适的。  
    
**config-file 参数**
    
        定义查询提升的文件的路径。该文件必须存在于<code>&lt;instanceDir&gt;/conf/&lt;config-file&gt;</code>或<code>&lt;dataDir&gt;/&lt;config-file&gt;</code>。如果文件存在于<code>conf/</code>目录中，它将在启动时加载一次。如果它存在于<code>data/</code>目录中，它将被重载为每个 IndexReader。  
    
**forceElevation 参数**
    
        默认情况下，该组件遵守所请求的<code>sort</code>参数：如果请求要求按日期排序，它将按日期排序结果。如果为<code>forceElevation=true</code>（默认），结果将首先返回提升的文档，然后按日期排序。  
    


### elevate.xml文件<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-elevate-xml-file"/>

提升查询结果可以在 config-file 参数中指定的外部 XML 文件中进行配置。一个 elevate.xml 文件可能看起来像这样：  
```
&lt;elevate&gt;
  &lt;query text="foo bar"&gt;
    &lt;doc id="1" /&gt;
    &lt;doc id="2" /&gt;
    &lt;doc id="3" /&gt;
  &lt;/query&gt;
  &lt;query text="ipod"&gt;
    &lt;doc id="MA147LL/A" /&gt;  &lt;!-- put the actual ipod at the top --&gt;
    &lt;doc id="IW-02" exclude="true" /&gt; &lt;!-- exclude this cable --&gt;
  &lt;/query&gt;
&lt;/elevate&gt;
```
在这个例子中，查询 “foo bar” 将首先返回文档1，2和3，然后是对同一个查询通常出现的任何情况。对于查询 “ipod”，它将首先返回 “MA147LL / A”，并确保 “IW-02” 不在结果集中。  
  
如果要升级的文档没有在 elevate.xml 文件中定义，则应该在查询时使用 elevateIds 参数传入。  

## 使用查询提升组件<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#using-the-query-elevation-component"/>

### enableElevation 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-enableelevation-parameter"/>

对于调试，查看有和没有提升的文档的结果可能是有用的。要隐藏结果，请使用 enableElevation=false：  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;debugQuery=true&amp;enableElevation=true
```
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;debugQuery=true&amp;enableElevation=false
```

### forceElevation 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-forceelevation-parameter"/>

您可以在运行时通过添加 forceElevation=true 到查询网址来强制提升：  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;debugQuery=true&amp;enableElevation=true&amp;forceElevation=true
```

### 独有的参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-exclusive-parameter"/>

您可以强制 Solr 通过添加 exclusive=true 到 URL 来仅返回在提升文件中指定的结果：  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;debugQuery=true&amp;exclusive=true
```

### 文档转换器和 markExcludes 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#document-transformers-and-the-markexcludes-parameter"/>

该 [elevated] 文档转换器可用于对每个文档进行注释，并提供有关是否提升的信息：  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;fl=id,[elevated]
```
同样，在故障排除时查看所有匹配的文档 (包括提升配置通常会排除的文档) 也很有用。这可以通过使用 markExcludes = true 参数，然后使用 [excluded] 转换器:  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;markExcludes=true&amp;fl=id,[elevated],[excluded]
```

### elevateIds和excludeIds 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-elevateids-and-excludeids-parameters"/>

在使用提升组件时，可以在请求时重写预先配置的查询高程列表，以使用这些请求参数中指定的唯一键。  
例如，在下面的请求中，文件 3007WFP 和 9885A004 将被提升，文件 IW-02 将被排除 - 无论在 elevate.xml 中为查询“cable”配置了哪些提升或排除：  
```
http://localhost:8983/solr/techproducts/elevate?q=cable&amp;df=text&amp;excludeIds=IW-02&amp;elevateIds=3007WFP,9885A004
```
如果在请求时指定了这些参数中的任何一个，则将忽略查询的整个提升配置。  
例如，在下面的请求文件中，IW-02 和 F8V7067-APL-KIT 将被提升，并且不会排除任何文档 - 无论在 elevate.xml 中为查询 “ipod” 配置了什么样的提升或排除：  
```
http://localhost:8983/solr/techproducts/elevate?q=ipod&amp;df=text&amp;elevateIds=IW-02,F8V7067-APL-KIT
```

### 具有提升的 fq 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-query-elevation-component.html#the-fq-parameter-with-elevation"/>

查询提升尊重标准过滤器查询（fq）参数。也就是说，如果查询包含 fq 参数，即使 elevate.xml 将其他文档添加到结果集，所有结果也将在该过滤器内。  
