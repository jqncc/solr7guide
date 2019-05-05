## Solr实时获取功能 
<div class="content-intro view-box ">为了让Solr索引更新可见（可搜索），某些类型的提交必须重新打开搜索程序到索引的新时间点视图。  
  
Solr实时获取功能使检索（通过 unique-key）任何文档的最新版本，而无需重新打开一个搜索的相关成本。在使用 Solr 作为 NoSQL 数据存储而不仅仅是搜索索引时，这是非常有用的。  
实时获取依赖于 "更新日志" 功能，该功能默认启用，并且可以在 solrconfig.xml 中进行配置，如下所示：  
```
&lt;updateLog&gt;
  &lt;str name="dir"&gt;${solr.ulog.dir:}&lt;/str&gt;
&lt;/updateLog&gt;
```
实时获取请求可以使用在 Solr 隐式存在的 /get 处理程序来执行-请参见隐式 RequestHandlers，它等效于以下配置:  
```
&lt;requestHandler name="/get" class="solr.RealTimeGetHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="omitHeader"&gt;true&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
例如, 如果您使用 bin/solr-techproducts 示例命令启动了Solr，那么您就可以对一个新文档（不需要提交它）进行索引 (以提交它)，如下所示:  
```
curl 'http://localhost:8983/solr/techproducts/update/json?commitWithin=10000000' \
  -H 'Content-type:application/json' -d '[{"id":"mydoc","name":"realtime-get test!"}]'
```
如果您进行正常搜索，则不应该找到该文档：  
```
http://localhost:8983/solr/techproducts/query?q=id:mydoc
...
"response":
{"numFound":0,"start":0,"docs":[]}
```
但是，如果使用公开的实时获取处理程序/get，则仍然可以检索该文档：  
```
http://localhost:8983/solr/techproducts/get?id=mydoc
...
{"doc":{"id":"mydoc","name":"realtime-get test!", "_version_":1487137811571146752}}
```
您也可以通过 ids 参数和以逗号分隔的 id 列表或使用多个 id 参数同时指定多个文档。如果您指定了多个 ID 或使用 ids 参数，则响应将模拟正常的查询响应，以便现有客户端更容易解析。  
  
例如：  
```
http://localhost:8983/solr/techproducts/get?ids=mydoc,IW-02
http://localhost:8983/solr/techproducts/get?id=mydoc&amp;id=IW-02
...
{"response":
  {"numFound":2,"start":0,"docs":
    [ { "id":"mydoc",
        "name":"realtime-get test!",
        "_version_":1487137811571146752},
      {
        "id":"IW-02",
        "name":"iPod &amp; iPod Mini USB 2.0 Cable",
        ...
    ]
 }
}
```
实时获取请求也可以与过滤器查询结合使用，用 fq 参数指定，就像搜索请求一样：  
```
http://localhost:8983/solr/techproducts/get?id=mydoc&amp;id=IW-02&amp;fq=name:realtime-get
...
{"response":
  {"numFound":1,"start":0,"docs":
    [ { "id":"mydoc",
        "name":"realtime-get test!",
        "_version_":1487137811571146752}
    ]
 }
}
```
注意：如果您使用的是 SolrCloud，则不要禁用实时获取处理程序<code>/get</code>，否则任何首项选择都将导致有关碎片的所有副本的完全同步。  
同样，复制副本恢复也总是从首项那里获取完整的索引，因为在没有这个处理程序的情况下，部分同步是不可能的。  
