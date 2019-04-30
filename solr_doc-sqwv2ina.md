## Collections API创建一个集合：CREATE 
<div class="content-intro view-box ">
## <span style="font-size: 14px;">CREATE</span>：创建一个集合

/admin/collections?action=CREATE&amp;name=name  

### 创建参数

CREATE操作允许以下参数：  
- name  
    
        要创建的集合的名称。该参数是必需的。  
    
- router.name  
 
        将使用的路由器名称。路由器定义文件如何在碎片之间分配。可能的值是<code>implicit</code>或者 <code>compositeId</code>，这是默认值。  
        
            该<code>implicit</code>路由器不会自动将文档路由到不同的碎片。无论您在索引请求中（或在每个文档中）指明的分片都将被用作这些文档的目标。  
          
        
            该<code>compositeId</code>路由器对 uniqueKey 字段中的值进行哈希处理，并在集合的 clusterstate 中查找该哈希，以确定哪些碎片会收到该文件，并具有手动引导路由的额外功能。  
          
        
            使用<code>implicit</code>路由器时，该<code>shards</code>参数是必需的。使用<code>compositeId</code>路由器时，该<code>numShards</code>参数是必需的。  
          
        
            有关更多信息，另请参阅文档路由一节。  
          
     
- numShards  
  
        要作为集合的一部分创建的分片的数量。当<code>router.name</code>是<code>compositeId</code>时，这是一个必需的参数。  
     
- shards  
   
        以逗号分隔的分片名称列表，例如<code>shard-x,shard-y,shard-z</code>。当<code>router.name</code>是<code>implicit</code>时，这是一个必需的参数。  
     
- replicationFactor  
  
        要为每个分片创建的副本数量。默认是<code>1</code>。这将创建一个NRT类型的副本。如果您需要其他类型的副本，请参阅<code>tlogReplicas</code>和<code>pullReplica</code>参数。有关副本类型的更多信息，请参阅副本类型。  
     
- nrtReplicas  
    
        为此集合创建的NRT（近实时）副本的数量。这种副本维护一个事务日志，并在本地更新其索引。如果你想要所有的副本都是这种类型的，你可以直接使用<code>replicationFactor</code>。  
    
- tlogReplicas  
    
        要为此集合创建的TLOG副本的数量。这种副本维护一个事务日志，但只通过从leader的副本来更新其索引。有关副本类型的更多信息，请参阅副本类型。  
     
- pullReplicas  
    
        为此集合创建的PULL副本的数量。这种类型的副本不维护事务日志，只通过从leader的副本来更新其索引。这种类型没有资格成为leader，不应该是集合中唯一的副本。有关副本类型的更多信息，请参阅副本类型。  
     
- maxShardsPerNode  
    
        创建集合时，分片或副本分布在所有可用（即活动）节点上，并且同一分片的两个副本永远不会在同一个节点上。  
        
            如果一个节点在调用CREATE操作时不存在，它将不会获得新集合的任何部分，这可能会导致在单个活动节点上创建太多副本。定义<code>maxShardsPerNode</code>对CREATE操作将传播到每个节点的副本数量设置限制。  
          
        
            如果整个集合不能适应实时节点，则根本不会创建任何集合。默认<code>maxShardsPerNode</code>值是<code>1</code>。  
          
     
- createNodeSet  
   
        允许定义节点来传播新的集合。格式是以逗号分隔的node_name列表，例如<code>localhost:8983_solr,localhost:8984_solr,localhost:8985_solr</code>。  
        
            如果未提供，则CREATE操作将在所有的活动 Solr 节点上创建碎片副本。  
          
        
            或者，使用特殊值<code>EMPTY</code>在新集合中初始创建无碎片副本，然后使用ADDREPLICA操作在需要时添加分片副本。  
          
    
- createNodeSet.shuffle  
  
        控制是否为该集合创建的分片副本将按照顺序分配给由<code>createNodeSet</code>所指定的节点，或者在创建单个副本之前应该对节点列表进行分配。  
        
            一个<code>false</code>值，使得集合创造可预见的结果，并对单个碎片副本的位置提供更精确的控制，但如果为<code>true</code>可以确保副本跨节点分布均匀更好的选择。默认是<code>true</code>。  
          
        
            如果<code>createNodeSet</code>没有指定，则忽略此参数。  
          
     
- collection.configName  
    
        定义用于此集合的配置的名称（<strong>必须已经存储在ZooKeeper中</strong>）。如果没有提供，Solr将默认为集合名称作为配置名称。  
    
- router.field  
    
        如果指定了这个参数，则路由器将查看输入文档中字段的值来计算哈希，并标识分片而不是查看<code>uniqueKey</code>字段。如果文档中指定的字段为空，则文档将被拒绝。  
        
            请注意，通过文档ID实时获取或检索也需要参数<code>_route_</code>（或<code>shard.keys</code>）来避免分布式搜索。  
          
     
- property.name=value  
   
        将核心属性名称设置为值。有关受支持的属性和值的详细信息，请参阅定义core.properties一节。  
    
- autoAddReplicas  
    
        设置为<code>true</code>时，仅允许在共享文件系统（如HDFS）上自动添加副本。有关设置和覆盖的更多详细信息，请参阅autoAddReplicas设置一节。默认值是<code>false</code>。  
     
- async  
   
        请求ID来跟踪这个将被异步处理的动作。  
     
- rule  
    
        副本放置规则。有关详细信息，请参阅基于规则的副本放置部分。  
     
- snitch  
   
        snitch 提供者的细节。有关详细信息，请参阅基于规则的副本放置部分。  
    
- policy  
    
        集合级别策略的名称。有关详细信息，请参阅定义集合特定的策略  
    

### 创建响应

响应将包括请求的状态和新的核心名称。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。
      
  

### 使用CREATE的例子

输入：  
```
http://localhost:8983/solr/admin/collections?action=CREATE&amp;name=newCollection&amp;numShards=2&amp;replicationFactor=1
```
输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;3764&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;3450&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;newCollection_shard1_replica1&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;3597&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;newCollection_shard2_replica1&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
