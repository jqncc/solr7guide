## SPLITSHARD：分割碎片 
<div class="content-intro view-box ">/admin/collections?action=SPLITSHARD&amp;collection=name&amp;shard=shardID
      
  
分割碎片将使用现有的碎片并将其拆分成两个碎片，这些碎片作为两个（新的）碎片写入磁盘。原始碎片将继续包含相同的数据，但是会启动将请求重新路由到新的碎片。新的碎片将具有与原始碎片一样多的副本。分割碎片后自动发出软提交，以便文档在子分片上可见。在分割操作之后，显式提交（hard 或者 soft）不是必需的，因为索引在分割操作期间会自动保留到磁盘。  
这个命令允许无缝拆分，不需要停机。被拆分的碎片将继续接受查询和索引请求，并且一旦完成该操作，将自动开始将请求路由到新的碎片。此命令只能用于使用numShards参数创建的SolrCloud集合，这意味着依赖于Solr的基于哈希的路由机制的集合。  
分割是通过将原始分片的哈希范围划分为两个相等的分区，并根据新的子分区来划分原始碎片中的文档。下面讨论的两个参数：ranges和split.key提供对分割的方式的进一步控制。  
碎片分割可能是一个漫长的过程。为了避免超时，你应该将其作为异步调用运行。  

### SPLITSHARD参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#splitshard-parameters"/>

- collection  

   
        包含要分割的碎片的集合的名称。该参数是必需的。  
    
- shard  

  
        要分割的碎片的名称。未指定<code>split.key</code>时需要此参数。  
    
- ranges  

    
        用十六进制表示哈希范围的逗号分隔列表，如<code>ranges=0-1f4,1f5-3e8,3e9-5dc</code>。  
        
            此参数可用于将原始分片的哈希范围划分为以十六进制指定的任意哈希范围区间。例如，如果原始哈希范围<code>0-1500</code>然后添加参数：<code>ranges=0-1f4,1f5-3e8,3e9-5dc</code>将原始碎片分成三个碎片，分别为哈希范围<code>0-500</code>，<code>501-1000</code>和<code>1001-1500</code>。  
          
    
- split.key  

    
        用于分割索引的关键。  
        
            此参数可用于使用路由密钥分割碎片，以使指定的路由密钥的所有文档都在一个专用的子分片中结束。在这种情况下不需要提供<code>shard</code>参数，因为路由密钥足以找出正确的分片。不支持跨越多个分片的路由密钥。  
          
        
            例如，假设<code>split.key=A!</code>哈希到该范围，<code>12-15</code>并且属于带范围<code>0-20</code>的分片“shard1”。分割经此路由密钥将产生三个子碎片与范围<code>0-11</code>，<code>12-15</code>以及<code>16-20</code>。请注意，具有路由密钥哈希范围的子分片也可能包含哈希范围重叠的其他路由密钥的文档。  
          
    
- property.name=value  

   
        将核心属性name设置为value。有关受支持的属性和值的详细信息，请参阅定义core.properties一节。  
    
- async  

    
        请求ID来跟踪这个将被异步处理的动作
            <a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#asynchronous-calls"/>
          
    


### SPLITSHARD响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#splitshard-response"/>

输出将包括请求的状态和新的分片名称，它们将使用原始碎片作为基础，添加一个下划线和一个数字。例如，“shard1”将变成“shard1_0”和“shard1_1”。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。
      
  

### 使用SPLITSHARD的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-splitshard"/>

输入：  
拆分“anotherCollection”集合的shard1。  
```
http://localhost:8983/solr/admin/collections?action=SPLITSHARD&amp;collection=anotherCollection&amp;shard=shard1
```

输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;6120&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;3673&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;anotherCollection_shard1_1_replica1&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;3681&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;anotherCollection_shard1_0_replica1&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;6008&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;6007&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;71&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;0&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;anotherCollection_shard1_1_replica1&lt;/str&gt;
      &lt;str name="status"&gt;EMPTY_BUFFER&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;0&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;anotherCollection_shard1_0_replica1&lt;/str&gt;
      &lt;str name="status"&gt;EMPTY_BUFFER&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
