## SolrCloud将文档迁移到另一个集合：MIGRATE 
<div class="content-intro view-box ">/admin/collections?action=MIGRATE&amp;collection=name&amp;split.key=key1!&amp;target.collection=target_collection&amp;forward.timeout=60  
在SolrCloud中MIGRATE命令用于将具有给定路由秘钥的所有文档迁移到另一个集合。源集合将继续具有相同的数据，但是它将开始将写入请求重新路由到目标集合，持续时间由forward.timeout参数指定的秒数。在MIGRATE操作完成之后，用户有责任切换到目标集合进行读取和写入操作。  
  
由split.key参数指定的路由密钥可能跨越源集合和目标集合上的多个分片。迁移是在单个线程中按分片执行的。在“迁移（migrate）”过程中，可以通过此命令创建一个或多个临时集合，但它们会自动清除。  
这是一个长时间运行的操作，因此强烈建议使用该async参数。如果未指定该async参数，则操作默认为同步，并建议在调用时保持较长的读取超时。即使读取超时时间很长，请求仍可能超时，但这并不一定意味着操作失败。用户应在再次调用该操作之前检查日志、群集状态、源和目标集合。  
该命令仅适用于使用compositeId路由器的集合。目标集合在MIGRATE命令运行期间不能接收任何写入，否则有些写入可能会丢失。  
请注意，MIGRATE API不会对文档执行任何重复数据删除，因此如果目标集合包含具有与正在迁移的文档相同的uniqueKey的文档，则目标集合将最终使用重复文档。  

### MIGRATE参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#migrate-parameters"/>

**collection**
    
        将从中分离文档的源集合的名称。该参数是必需的。  
    
**target.collection**
    
        要将文档迁移到的目标集合的名称。该参数是必需的。  
    
**split.key**
    
        路由秘钥的前缀。例如，如果文档的uniqueKey是“a！123”，那么你会使用<code>split.key=a!</code>。该参数是必需的。  
    
**forward.timeout**
    
        以秒为单位的超时，直到给定源集合的写请求<code>split.key</code>将被转发到目标分片。默认值是60秒。  
    
**property.name=value**
    
        将核心属性name设置为value。有关受支持的属性和值的详细信息，请参阅“定义core.properties”一节。  
**async**
    
        请求ID来跟踪这个将被异步处理的操作。  
    


### MIGRATE响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#migrate-response"/>

响应将包括请求的状态。  

### 使用MIGRATE的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-migrate"/>

在这个例子中输入如下：  
```
http://localhost:8983/solr/admin/collections?action=MIGRATE&amp;collection=test1&amp;split.key=a!&amp;target.collection=test2
```
得到以下的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;19014&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;test2_shard1_0_replica1&lt;/str&gt;
      &lt;str name="status"&gt;BUFFERING&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;2479&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;split_shard1_0_temp_shard1_0_shard1_replica1&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1002&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;21&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1655&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;split_shard1_0_temp_shard1_0_shard1_replica2&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;4006&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;17&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;test2_shard1_0_replica1&lt;/str&gt;
      &lt;str name="status"&gt;EMPTY_BUFFER&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="192.168.43.52:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;31&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst name="192.168.43.52:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;31&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;test2_shard1_1_replica1&lt;/str&gt;
      &lt;str name="status"&gt;BUFFERING&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1742&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;split_shard1_1_temp_shard1_1_shard1_replica1&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1002&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;15&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1917&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;split_shard1_1_temp_shard1_1_shard1_replica2&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;5007&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;8&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;test2_shard1_1_replica1&lt;/str&gt;
      &lt;str name="status"&gt;EMPTY_BUFFER&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="192.168.43.52:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;30&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst name="192.168.43.52:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;30&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
