## Solr删除分片：DELETESHARD 
<div class="content-intro view-box ">DELETESHARD删除分片将卸载分片的所有副本，将其从clusterstate.json中删除，并且（默认情况下）删除每个副本的instanceDir和dataDir。它只会删除不活动的分片，或者没有为自定义分片赋予范围的分片。
      
  
/admin/collections?action=DELETESHARD&amp;shard=shardID&amp;collection=name  

### DELETESHARD参数


    - collection
          
        包含要删除的分片的集合的名称。该参数是必需的。  
    
    - shard
          
        要删除的分片的名称。该参数是必需的。  
    
    - deleteInstanceDir
          
        默认情况下，Solr将删除每个被删除副本的整个实例目录。将其设置为<code>false</code>，以防止实例目录被删除。  
    

- deleteDataDir
      
    默认情况下，Solr将删除每个被删除副本的dataDir。将其设置为<code>false</code>，以防止数据目录被删除。  

- deleteIndex
      
    默认情况下，Solr将删除每个被删除副本的索引。将其设置为<code>false</code>，以防止索引目录被删除。  

- async
      
    请求ID来跟踪这个将被异步处理的操作。  


### DELETESHARD响应

输出将包含请求的状态。如果状态不是“成功”，则会显示错误消息，说明请求失败的原因。  

### 使用DELETESHARD的示例

在该实例中具有如下输入：  
删除“anotherCollection”集合的“shard1”。  
```
http://localhost:8983/solr/admin/collections?action=DELETESHARD&amp;collection=anotherCollection&amp;shard=shard1
```

产量输出如下：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;558&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst name="10.0.1.4:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;27&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
