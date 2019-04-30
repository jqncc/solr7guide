## SolrConfig中的RequestDispatcher 
<div class="content-intro view-box ">solrconfig.xml的requestDispatcher元素，控制了Solr HTTP RequestDispatcher实现对请求的响应方式。  
其中包括用于定义是否应该处理/select的url的参数(对于Solr 1.1兼容性)，如果它支持远程流，文件上传的最大大小以及它将如何响应HTTP缓存头的请求。  

## handleSelect元素

注意：<span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: nowrap;">handleSelect </span>是为了传统的后向兼容性；那些新来的Solr不需要改变默认配置的方式  
第一个可配置项是&lt; requestDispatcher &gt;元素本身的handleSelect当选属性。该属性可以设置为“true”或“false”两个值之一。它管理Solr如何响应请求，如：/select?qt=XXX。如果requestHandler没有显式注册/select名称，默认值“false”将忽略请求/select。值“true”将查询请求路由到定义为qt值的解析器。
      
  
在Solr的最新版本中，/selectrequestHandler是默认定义的，因此值“false”将正常工作。有关更多信息，请参阅SolrConfig中的RequestHandlers和SearchComponents部分。  
```
&lt;requestDispatcher handleSelect="true" &gt;
  ...
&lt;/requestDispatcher&gt;
```

## requestParsers元素<a href="http://lucene.apache.org/solr/guide/7_0/requestdispatcher-in-solrconfig.html#requestparsers-element"/>

该&lt;requestParsers&gt;子元素控制与解析请求的相关值。这是一个空的XML元素，没有任何内容，只有属性。
      
  
该属性enableRemoteStreaming控制是否允许远程传输内容。如果省略或设置为false（默认），则不允许流式传输。将其设置为true允许您指定要使用stream.file和stream.url参数进行流式传输的内容或位置。  
如果启用远程流式传输，请确保您已启用身份验证。否则，有人可能通过访问任意的URL访问您的内容。将Solr放置在防火墙后面以防止从不可信的客户端访问Solr也是一个好主意。  
该属性multipartUploadLimitInKB以可以在多部分HTTP POST请求中提交的文档的大小为单位设置以千字节为单位的上限。指定的值乘以1024来确定以字节为单位的大小。-1意味着MAX_INT 的值，如果省略，它也是系统默认值。  
该属性formdataUploadLimitInKB在HTTP POST请求中提交的表单数据（application / x-www-form-urlencoded）的大小上设置了一个限制（千字节），可用于传递不适合URL的请求参数。-1意味着MAX_INT 的值，如果省略，也是系统默认值。  
该属性addHttpRequestToContext可以用来指示原始HttpServletRequest对象应该包含在SolrQueryRequest使用httpRequest键的上下文映射。HttpServletRequest不是任何Solr组件所使用的，但在开发自定义插件时可能会有用。  
```
&lt;requestParsers enableRemoteStreaming="false"
                multipartUploadLimitInKB="2048"
                formdataUploadLimitInKB="2048"
                addHttpRequestToContext="false" /&gt;
```
以下命令是如何通过[Config API](http://lucene.apache.org/solr/guide/7_0/config-api.html#creating-and-updating-common-properties)启用RemoteStreaming和BodyStreaming的示例：  
```
curl http://localhost:8983/solr/gettingstarted/config -H 'Content-type:application/json' -d'{
    "set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true},
    "set-property" : {"requestDispatcher.requestParsers.enableStreamBody":true}
}'
```

## httpCaching元素<a href="http://lucene.apache.org/solr/guide/7_0/requestdispatcher-in-solrconfig.html#httpcaching-element"/>

该&lt;httpCaching&gt;元素控制HTTP缓存控制标头。不要将这些设置与Solr的内部缓存配置混淆。该元素控制由W3C HTTP规范定义的HTTP响应的缓存。
      
  
这个元素允许三个属性和一个子元素。&lt;httpCaching&gt;元素的属性控制是否允许对GET请求的304响应，如果是，则应该是什么类型的响应。当一个HTTP客户端应用程序发出一个GET时，它可以选择性地指定一个304响应是可接受的，如果该资源从上次被提取以来没有被修改过。  
- never304  
    
        如果存在值<code>true</code>，则GET请求将永远不会响应304代码，即使请求的资源未被修改。当此属性设置为true时，接下来的两个属性将被忽略。将其设置为true对于开发是非常方便的，因为当通过Web浏览器或其他支持缓存头的客户端修改Solr响应时，304响应可能会造成混淆。  
    
- lastModFrom  
    
        该属性可以设置为<code>openTime</code>（默认）或<code>dirLastMod</code>。该值<code>openTime</code>表示与由客户端发送的If-Modified-Since报头相比，最后修改时间应该相对于搜索器开始的时间来计算。使用<code>dirLastMod</code>，如果您想要时间精确地对应于磁盘上最后更新的索引。  
    
- etagSeed  
    
        这个属性的值作为<code>ETag</code>头的值发送。即使索引没有改变，更改此值也可能有助于强制客户重新获取内容 - 例如，当您对配置进行了一些更改时。  
```
&lt;httpCaching never304="false"
             lastModFrom="openTime"
             etagSeed="Solr"&gt;
  &lt;cacheControl&gt;max-age=30, public&lt;/cacheControl&gt;
&lt;/httpCaching&gt;
```
    

### cacheControl元素<a href="http://lucene.apache.org/solr/guide/7_0/requestdispatcher-in-solrconfig.html#cachecontrol-element"/>

除了这些属性之外，&lt;httpCaching&gt;接受一个子元素：&lt;cacheControl&gt;。这个元素的内容将作为HTTP响应的Cache-Control头的值发送。此标头用于修改请求客户端的默认缓存行为。Cache-Control头的可能值由第14.9节中的HTTP 1.1规范定义。
      
  
设置max-age字段控制客户端在从服务器再次请求之前可以重新使用缓存响应的时间。此时间间隔应根据您更新索引的频率以及您的应用程序是否可接受使用已过时的内容来设置。设置must-revalidate将告诉客户端在服务器上重新使用之前确认其缓存副本仍然正常。这将确保使用最及时的结果，同时避免在不需要的情况下再次获取内容，则请求服务器进行检查。  
