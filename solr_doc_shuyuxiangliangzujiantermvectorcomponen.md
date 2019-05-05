## 术语向量组件：TermVectorComponent 
<div class="content-intro view-box ">TermVectorComponent 是一个搜索组件，用于返回关于与您的搜索匹配的文档的附加信息。  
  
对于响应中的每个文档，TermVectorCcomponent 可以返回术语向量、术语频率、逆文档频率、位置和偏移量信息。  

## 术语向量组件配置<a href="http://lucene.apache.org/solr/guide/7_0/the-term-vector-component.html#term-vector-component-configuration"/>

TermVectorComponent 未在 Solr 中隐式启用 - 它必须在 solrconfig.xml 文件中显式配置。本页中的示例显示了如何在 Solr 的 "techproducts" 示例中进行配置：  
```
bin/solr -e techproducts
```
要启用这个组件，您需要使用一个 searchComponent 元素来配置它：  
```
&lt;searchComponent name="tvComponent" class="org.apache.solr.handler.component.TermVectorComponent"/&gt;
```
然后必须将请求处理程序配置为使用此组件名称。在这个 techproducts 例子中，该组件与一个名为 /tvrh 的特殊请求处理程序相关联，默认情况下，使用 tv=true 参数启用术语向量；但您可以将其与任何请求处理程序关联：  
```
&lt;requestHandler name="/tvrh" class="org.apache.solr.handler.component.SearchHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;bool name="tv"&gt;true&lt;/bool&gt;
  &lt;/lst&gt;
  &lt;arr name="last-components"&gt;
    &lt;str&gt;tvComponent&lt;/str&gt;
  &lt;/arr&gt;
&lt;/requestHandler&gt;
```
一旦您的处理程序被定义了，您可以和任何模式一起使用（uniqueKeyField)对于用 termVector 属性配置的字段，例如在 techproducts 示例模式中：  
```
&lt;field name="includes"
       type="text_general"
       indexed="true"
       stored="true"
       multiValued="true"
       termVectors="true"
       termPositions="true"
       termOffsets="true" /&gt;
```

## 调用术语向量组件<a href="http://lucene.apache.org/solr/guide/7_0/the-term-vector-component.html#invoking-the-term-vector-component"/>

下面的例子显示了使用上面的配置调用这个 TermVectorComponent 组件：  
```
http://localhost:8983/solr/techproducts/tvrh?q=*:*&amp;start=0&amp;rows=10&amp;fl=id,includes&amp;wt=xml
```
```
...
&lt;lst name="termVectors"&gt;
  &lt;lst name="GB18030TEST"&gt;
    &lt;str name="uniqueKey"&gt;GB18030TEST&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="EN7800GTX/2DHTV/256M"&gt;
    &lt;str name="uniqueKey"&gt;EN7800GTX/2DHTV/256M&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="100-435805"&gt;
    &lt;str name="uniqueKey"&gt;100-435805&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="3007WFP"&gt;
    &lt;str name="uniqueKey"&gt;3007WFP&lt;/str&gt;
    &lt;lst name="includes"&gt;
      &lt;lst name="cable"/&gt;
      &lt;lst name="usb"/&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
  &lt;lst name="SOLR1000"&gt;
    &lt;str name="uniqueKey"&gt;SOLR1000&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="0579B002"&gt;
    &lt;str name="uniqueKey"&gt;0579B002&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="UTF8TEST"&gt;
    &lt;str name="uniqueKey"&gt;UTF8TEST&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="9885A004"&gt;
    &lt;str name="uniqueKey"&gt;9885A004&lt;/str&gt;
    &lt;lst name="includes"&gt;
      &lt;lst name="32mb"/&gt;
      &lt;lst name="av"/&gt;
      &lt;lst name="battery"/&gt;
      &lt;lst name="cable"/&gt;
      &lt;lst name="card"/&gt;
      &lt;lst name="sd"/&gt;
      &lt;lst name="usb"/&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
  &lt;lst name="adata"&gt;
    &lt;str name="uniqueKey"&gt;adata&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="apple"&gt;
    &lt;str name="uniqueKey"&gt;apple&lt;/str&gt;
  &lt;/lst&gt;
&lt;/lst&gt;
```

### 术语向量请求参数<a href="http://lucene.apache.org/solr/guide/7_0/the-term-vector-component.html#term-vector-request-parameters"/>

下面的例子显示了这个组件的一些可用的请求参数：  
```
http://localhost:8983/solr/techproducts/tvrh?q=includes:[* TO *]&amp;rows=10&amp;indent=true&amp;tv=true&amp;tv.tf=true&amp;tv.df=true&amp;tv.positions=true&amp;tv.offsets=true&amp;tv.payloads=true&amp;tv.fl=includes
```

**tv 参数**
    
        如果为<code>true</code>，将运行术语向量组件。  
    
**tv.docIds 参数**
    
        对于给定的以逗号分隔的 Lucene 文档 ID 列表（不是 Solr 唯一键），将返回术语向量。  
**tv.fl 参数**
    
        对于给定的以逗号分隔的字段列表，将返回术语向量。如果未指定，则使用<code>fl</code>参数。  
    
**tv.all 参数**
    
        如果为<code>true</code>将启用以下列出的所有布尔参数：<code>tv.df</code>，<code>tv.offsets</code>，<code>tv.positions</code>，<code>tv.payloads</code>，<code>tv.tf</code>和<code>tv.tf_idf</code>。  
    
**tv.df 参数**
    
        如果为<code>true</code>，将返回集合中该术语的文档频率（DF）。这可能在计算上是昂贵的。  
    
**tv.offsets 参数**
    
        如果为<code>true</code>，将返回文档中每个术语的偏移信息。  
    
**tv.positions 参数**
    
        如果为<code>true</code>，则返回位置信息。  
    
**tv.payloads 参数**
    
        如果为<code>true</code>，则返回有效载荷信息。  
    
**tv.tf 参数**
    
        如果为<code>true</code>，返回文档中每个术语的文档术语频率信息。  
    
**tv.tf_idf 参数**
    
        如果为<code>true</code>计算每个术语的 TF/DF（即：TF * IDF）。请注意，这是“ 字频乘以逆文档频率” 的文字计算，而不是经典的 TF-IDF 相似性度量。  
        
            这个参数需要<code>tv.tf</code>和<code>tv.df</code>是 true。这可能在计算上是昂贵的。（结果不显示在示例输出中）  
          
    

要查看 TermVector 组件输出的示例，请参阅Wiki页面：http://wiki.apache.org/solr/TermVectorComponentExampleOptions  

## SolrJ 和术语向量组件

SolrQuery 类和 QueryResponse 类都不提供特定的方法调用来设置术语向量组件参数或获取 "termVectors" 输出。但是，它有一个补丁：[SOLR-949](https://issues.apache.org/jira/browse/SOLR-949)。  
