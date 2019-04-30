## 使用Collection API的异步调用 
<div class="content-intro view-box ">
## 异步调用<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#asynchronous-calls"/>
由于某些集合API调用可能是长时间运行的任务（如SPLITSHARD），因此可以选择异步运行这些调用。指定async=&lt;request-id&gt;使您可以进行异步调用，在任何时候都可以使用 [REQUESTSTATUS](https://www.w3cschool.cn/solr_doc/solr_doc-81pm2j6f.html) 调用请求其状态。  
  
截至目前，REQUESTSTATUS不会自动清理跟踪数据结构，这意味着完成或失败任务的状态保持存储在ZooKeeper中，除非手动清除。DELETESTATUS可以用来清除存储的状态。但是，存储在群集中的异步调用响应数量有10,000个限制。  
### 异步请求的示例<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-of-async-requests"/>
示例输入：  
```
http://localhost:8983/solr/admin/collections?action=SPLITSHARD&amp;collection=collection1&amp;shard=shard1&amp;async=1000
```
得到输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;99&lt;/int&gt;
  &lt;/lst&gt;
  &lt;str name="requestid"&gt;1000&lt;/str&gt;
&lt;/response&gt;
```
