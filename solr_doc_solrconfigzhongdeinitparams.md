## SolrConfig中的InitParams 
<div class="content-intro view-box ">solrconfig 的 &lt;initParams&gt; 部分允许您定义处理程序配置之外的请求处理程序参数。
      
  
有几个用例可能是需要的：  

    - 一些处理程序在代码中隐式定义 - 请参阅隐式RequestHandlers - 应该有一种方法来添加/追加/重写一些隐式定义的属性。
    - 在处理程序中使用了一些属性。这有助于您只保留这些属性的单个定义，并将其应用于多个处理程序。

例如，如果您希望多个搜索处理程序返回相同的字段列表，则可以创建一个&lt;initParams&gt;部分，而无需在每个请求处理程序定义中定义相同的一组参数。如果您有一个单一的请求处理程序，该处理程序应该返回不同的字段，那么您可以像往常一样在个别&lt;requestHandler&gt;部分定义重写参数。
      
  
一个&lt;initParams&gt;部分的属性和配置镜像了请求处理程序的属性和配置。它可以包含用于默认、附加和不变的部分，与任何请求处理程序相同。  
例如，这里是在_default示例中默认定义的&lt; initParams &gt;部分：  
```
&lt;initParams path="/update/**,/query,/select,/tvrh,/elevate,/spell,/browse"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="df"&gt;_text_&lt;/str&gt;
  &lt;/lst&gt;
&lt;/initParams&gt;
```

这会将默认搜索字段（“df”）设置为路径部分中指定的所有请求处理程序的“文本”。如果我们稍后想要更改/query请求处理程序以在默认情况下搜索不同的字段，则可以通过定义/query中的&lt;requestHandler&gt;部分的参数来重写 &lt;initParams&gt;。
      
  
语法和语义与&lt;requestHandler&gt;类似。以下是属性：  

    - path
