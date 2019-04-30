## Solr删除副本：DELETEREPLICA 
<div class="content-intro view-box ">从指定的集合和分片中删除命名副本。
      
  
如果相应的核心正在运行并且正在运行的核心被卸载，则从 clusterstate 中删除该项，并且（默认情况下）删除instanceDir和dataDir。如果节点/核心（node/core）处于关闭状态，则将该条目从 clusterstate 中取出，如果核心稍后出现，则会自动取消注册。  
/admin/collections?action=DELETEREPLICA&amp;collection=collection&amp;shard=shard&amp;replica=replica  

### DELETEREPLICA参数

- collection  

    
        集合的名称。该参数是必需的。  
    
- shard  

    
        包含要删除的副本的分片的名称。该参数是必需的。  
    
- replica  

  
        要删除的副本的名称。  
        
            如果使用<code>count</code>，则不需要此参数。否则，必须提供此参数。  
          
    
- count  

    
        要删除的副本数量。如果请求的数量超过副本数量，则不会删除副本。如果只有一个副本，则不会被删除。  
        
            如果使用<code>replica</code>，则不需要此参数。否则，必须提供此参数。  
          
    
- deleteInstanceDir  

    
        默认情况下，Solr将删除被删除副本的整个instanceDir。将其设置为<code>false</code>以防止实例目录被删除。  
    
- deleteDataDir  

    
        默认情况下，Solr将删除被删除副本的dataDir。将其设置为<code>false</code>以防止数据目录被删除。  
    
- deleteIndex  

    
        默认情况下，Solr将删除被删除副本的索引。将其设置为<code>false</code>以防止索引目录被删除。  
    
- onlyIfDown  

    
        设置为<code>true</code>时，如果副本处于活动状态，则不会执行任何操作。默认为<code>false</code>。  
    
- async  

    
        请求ID来跟踪这个将被异步处理的动作。  
    


### 使用DELETEREPLICA的例子

在该例子中输入如下：  
```
http://localhost:8983/solr/admin/collections?action=DELETEREPLICA&amp;collection=test2&amp;shard=shard2&amp;replica=core_node3
```

将得到的输出是：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;110&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
