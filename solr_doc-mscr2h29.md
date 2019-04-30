## TermsComponent组件介绍 
TermsComponent组件提供对字段中的索引词的访问以及与每个词匹配的文档数量。这对于构建 auto-suggest 功能或在 term 级别而不是搜索或文档级别操作的任何其他功能都很有用。检索索引顺序中的 term 非常快，因为实现直接使用 Lucene 的 TermEnum 来遍历术语字典。
      
  
从某种意义上说，这个搜索组件在整个索引上提供了快速的 field-faceting，不受基本查询或任何过滤器的限制。返回的文档频率是与该词匹配的文档数量，包括任何已被标记为删除但尚未从索引中删除的文档。  

## 配置TermsComponent组件<a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#configuring-the-terms-component"/>

默认情况下，TermsComponent组件已在 solrconfig.xml 针对每个集合进行了配置。
      
  

### 定义TermsComponent组件<a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#defining-the-terms-component"/>

定义 Term 搜索组件很简单：简单地给它一个名称并使用 solr.TermsComponent 类。  
```
&lt;searchComponent name="terms" class="solr.TermsComponent"/&gt;
```

这使得该组件可供使用，但是直到包含在请求处理程序中才会被使用。  

### 在请求处理程序中使用TermsComponent组件

### <a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#using-the-terms-component-in-a-request-handler"/>

TermsComponent 组件包含在 Solr 的现成的请求处理程序中的 /terms 请求处理程序中 - 请参阅 Implicit RequestHandlers。
      
  
请注意，此请求处理程序的默认值将参数 “terms” 设置为 true，这允许根据请求返回条件。参数 “distrib” 被设置为 false，这使得这个处理程序只能在一个 Solr 内核上使用。  
如果您愿意的话，您可以将这个组件添加到另一个处理程序中，并且在 HTTP 请求中传递 “terms = true” 以获得条件。如果仅在单独的处理程序中定义它，则在查询时必须使用该处理程序，以获取条件而不是将常规文档作为结果。  

### TermsComponent 组件参数<a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#terms-component-parameters"/>

以下参数允许您控制返回的 term。如果您想永久设置它们，也可以使用请求处理程序来配置其中的任何一个。或者，您可以将它们添加到查询请求中。这些参数是：
      
  
- terms 参数  

    
        如果设置为<code>true</code>，则启用条款组件。默认情况下，条款组件处于关闭状态（<code>false</code>）。例： <code>terms=true</code>
          
    
- terms.fl 参数  

  
        指定从中检索 term 的字段。如果<code>terms=true</code>，则该参数是必需的。例： <code>terms.fl=title</code>
          
    
- terms.list 参数  

    
        获取逗号分隔的 term 列表的文档频率。词总是以索引顺序返回。如果<code>terms.ttf</code>设置为 true，则返回它们的总词频。如果定义了多个<code>terms.fl</code>，那么这些统计数据将在每个请求的字段中为每个词返回。  
        
            例： <code>terms.list=termA,termB,termC</code>
              
          
    
- terms.limit 参数  

   
        指定要返回的最大 term 数。默认是<code>10</code>。如果限制设置为小于0的数字，则不执行最大限制。虽然这不是必需的，但是这个参数或者<code>terms.upper</code>必须被定义。  
        
            例： <code>terms.limit=20</code>
              
          
    
- terms.lower 参数  

    
        指定开始的 term。如果未指定，则使用空字符串，从而导致 Solr 从字段的开始处开始。  
        
            例： <code>terms.lower=orange</code>
              
          
    
- terms.lower.incl 参数  

        如果设置为 true，则包含下限项（在结果中指定<code>terms.lower</code>）。  
        
            例： <code>terms.lower.incl=false</code>
              
          
    
- terms.mincount 参数  

    
        指定要返回的最小文档频率，以便将术语包含在查询响应中。结果包括小数（即 &gt;= mincount）。  
        
            例： <code>terms.mincount=5</code>
              
          
    
- terms.maxcount 参数  

  
        指定一个 term，为了包含在查询响应中而必须具有的最大文档频率。默认设置是-1，它不设置上限。结果包含 maxcount（即&lt;= maxcount）。
              
          
   
        
            例： <code>terms.maxcount=25</code>
              
          
    
- terms.prefix 参数  

   
        限制匹配以指定字符串开头的 term。  
        
            例： <code>terms.prefix=inter</code>
              
          
    
- terms.raw 参数  

    
        如果设置为 true，则返回索引项的原始字符，而不管其是否可读。例如，索引形式的数字是不可读的。  
        
            例： <code>terms.raw=true</code>
              
          
    
- terms.regex 参数  

    
        限制符合正则表达式的条件。  
        
            例： <code>terms.regex=.*pedist</code>
              
          
    
- terms.regex.flag 参数  

    
        定义一个 Java 正则表达式标志，用于计算<code>terms.regex</code>定义的表达式。有关每个标志的详细信息，请参见：http://docs.oracle.com/javase/tutorial/essential/regex/pattern.html。有效的选项是：
              
          
    
        
            <ul>
                <li>
                    <code>case_insensitive</code>
                      
                
                - 
                    <code>comments</code>
                      
                
                - 
                    <code>multiline</code>
                      
                
                - 
                    <code>literal</code>
                      
                
                - 
                    <code>dotall</code>
                      
                
                - 
                    <code>unicode_case</code>
                      
                
                - 
                    <code>canon_eq</code>
                      
                
                - 
                    <code>unix_lines</code>
                      
                    
                        例： <code>terms.regex.flag=case_insensitive</code>
                          
                      
                
            
          
    </li><li>terms.stats 参数  

    
        在结果中包含索引统计信息。目前只返回一个集合的 numDocs。当与<code>terms.list</code>结合时它提供足够的信息来计算术语列表的逆文件频率（IDF）。  
    </li><li>terms.sort 参数  

   
        定义如何对返回的条件进行排序。有效选项<code>count</code>按频率排序，首先选择最高频率，或<code>index</code>按索引顺序排序。
              
          
   
        
            例： <code>terms.sort=index</code>
              
          
    </li><li>terms.ttf 参数  

    
        如果设置为 true，那么同时返回<code>df</code>（docFreq）和<code>ttf</code>（totalTermFreq）统计信息，在<code>terms.list</code>请求 term 中。在这种情况下，响应格式是：  
```
&lt;lst name="terms"&gt;
  &lt;lst name="field"&gt;
    &lt;lst name="termA"&gt;
      &lt;long name="df"&gt;22&lt;/long&gt;
      &lt;long name="ttf"&gt;73&lt;/long&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/lst&gt;
```

    </li><li>terms.upper 参数  

   
        指定要停止的术语。虽然此参数不是必需的，但是该参数或<code>terms.limit</code>必须定义。  
        
            例： <code>terms.upper=plum</code>
              
          
    </li><li>terms.upper.incl 参数  

    
        如果设置为 true，则结果集合中将包含上限项目。默认值是 false。  
        
            例： <code>terms.upper.incl=true</code>
              
          
    </li>
</ul>
对 term 请求的回应是 term 及其文档频率值的列表。  

## TermsComponent组件示例

以下所有示例查询均适用于 Solr 的 “bin / solr -e techproducts” 示例。  

### 如何获得前10 的排序<a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#get-top-10-terms"/>

该查询请求名称字段中的前十个 term：  
```
http://localhost:8983/solr/techproducts/terms?terms.fl=name&amp;wt=xml
```

结果如下：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;2&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="terms"&gt;
    &lt;lst name="name"&gt;
      &lt;int name="one"&gt;5&lt;/int&gt;
      &lt;int name="184"&gt;3&lt;/int&gt;
      &lt;int name="1gb"&gt;3&lt;/int&gt;
      &lt;int name="3200"&gt;3&lt;/int&gt;
      &lt;int name="400"&gt;3&lt;/int&gt;
      &lt;int name="ddr"&gt;3&lt;/int&gt;
      &lt;int name="gb"&gt;3&lt;/int&gt;
      &lt;int name="ipod"&gt;3&lt;/int&gt;
      &lt;int name="memory"&gt;3&lt;/int&gt;
      &lt;int name="pc"&gt;3&lt;/int&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```


### 从字母 "a" 开始获取前10 的 term

这个查询按索引顺序（而不是按文档数计算的前10个项）请求名称字段中的前十个项：  
```
http://localhost:8983/solr/techproducts/terms?terms.fl=name&amp;terms.lower=a&amp;terms.sort=index&amp;wt=xml
```

结果如下：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="terms"&gt;
    &lt;lst name="name"&gt;
      &lt;int name="a"&gt;1&lt;/int&gt;
      &lt;int name="all"&gt;1&lt;/int&gt;
      &lt;int name="apple"&gt;1&lt;/int&gt;
      &lt;int name="asus"&gt;1&lt;/int&gt;
      &lt;int name="ata"&gt;1&lt;/int&gt;
      &lt;int name="ati"&gt;1&lt;/int&gt;
      &lt;int name="belkin"&gt;1&lt;/int&gt;
      &lt;int name="black"&gt;1&lt;/int&gt;
      &lt;int name="british"&gt;1&lt;/int&gt;
      &lt;int name="cable"&gt;1&lt;/int&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```


### SolrJ 调用

```
    SolrQuery query = new SolrQuery();
    query.setRequestHandler("/terms");
    query.setTerms(true);
    query.setTermsLimit(5);
    query.setTermsLower("s");
    query.setTermsPrefix("s");
    query.addTermsField("terms_s");
    query.setTermsMinCount(1);
    QueryRequest request = new QueryRequest(query);
    List&lt;Term&gt; terms = request.process(getSolrClient()).getTermsResponse().getTerms("terms_s");
```


## 将 TermsComponent 组件用于自动建议功能

## <a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#using-the-terms-component-for-an-auto-suggest-feature"/>

如果 [Suggester](https://www.w3cschool.cn/solr_doc/solr_doc-7yp92gzc.html) 不符合您的需求，则可以使用 Solr 中的 Terms 组件为您自己的搜索应用程序构建一个类似的功能。只需提交一个查询，指定用户键入的前缀的任何字符。例如，如果用户输入“at”，则搜索引擎的接口将提交以下查询：
      
  
```
http://localhost:8983/solr/techproducts/terms?terms.fl=name&amp;terms.prefix=at&amp;wt=xml
```

结果如下：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="terms"&gt;
    &lt;lst name="name"&gt;
      &lt;int name="ata"&gt;1&lt;/int&gt;
      &lt;int name="ati"&gt;1&lt;/int&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```

您可以使用参数 omitHeader=true 从查询响应中省略响应头，就像在这个例子中一样，它也以 JSON 格式返回响应：  
```
http://localhost:8983/solr/techproducts/terms?terms.fl=name&amp;terms.prefix=at&amp;omitHeader=true
```

结果如下：  
```
{
  "terms": {
    "name": [
      "ata",
      1,
      "ati",
      1
    ]
  }
}
```


## 分布式搜索支持<a href="http://lucene.apache.org/solr/guide/7_0/the-terms-component.html#distributed-search-support"/>

TermsComponent 也支持分布式索引。对于 /terms 请求处理程序，您必须提供以下两个参数：  
- shards 参数  

   
        指定分布式索引配置中的分片。有关分布式索引的更多信息，请参见使用索引分片的分布式搜索。  
    
- shards.qt 参数  

    
        指定 Solr 用于请求分片的请求处理程序。  
    

