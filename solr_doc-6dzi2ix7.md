## CREATESHARD：创建一个分片 
<div class="content-intro view-box ">对于使用“隐式”路由器的集合（即创建集合时，router.name=implicit），只能使用此API创建分片。可以为现有的“隐式”集合创建具有名称的新分片。
      
  
对使用“compositeId”路由器（router.key=compositeId）创建的集合使用SPLITSHARD 。  
/admin/collections?action=CREATESHARD&amp;shard=shardName&amp;collection=name  

### CREATESHARD参数

- collection  

   
        包含要分割的分片的集合的名称。该参数是必需的。  
    
- shard  

   
        要创建的分片的名称。该参数是必需的。  
    
- createNodeSet  

   
        允许定义节点来传播新的集合。如果未提供，则CREATESHARD操作将在所有活动的Solr节点上创建分片副本。  
        
            格式是以逗号分隔的node_name列表，例如<code>localhost:8983_solr,localhost:8984_solr,localhost:8985_solr</code>。  
          
    
- property.name=value  

    
        将核心属性name设置为value。有关受支持的属性和值的详细信息，请参阅定义core.properties一节。  
    
- async  

   
        请求ID来跟踪这个将被异步处理的动作。  
    


### CREATESHARD响应

输出将包含请求的状态。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 使用CREATESHARD的例子

输入：  
为“anImplicitCollection”集合创建“shard-z”。  
```
http://localhost:8983/solr/admin/collections?action=CREATESHARD&amp;collection=anImplicitCollection&amp;shard=shard-z
```

输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;558&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
