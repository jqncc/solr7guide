## RequestHandler中的提交和优化操作 
<div class="content-intro view-box ">
## 提交和优化操作<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#commit-and-optimize-operations"/>
在主服务器上执行提交或优化操作时，RequestHandler将读取与每个提交点关联的文件名列表。这依赖于replicateAfter配置中的参数来决定哪些类型的事件应该触发复制。  
以下的这些操作是支持的：  
- commit：只要在主索引上执行提交就会触发复制。
- optimize：每当主索引被优化时触发复制。
- startup：每当主索引启动时触发复制。
该replicateAfter参数可以接受多个参数。例如：  
```
&lt;str name="replicateAfter"&gt;startup&lt;/str&gt;
&lt;str name="replicateAfter"&gt;commit&lt;/str&gt;
&lt;str name="replicateAfter"&gt;optimize&lt;/str&gt;
```
