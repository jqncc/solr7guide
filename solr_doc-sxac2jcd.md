## RESTORE: 还原集合 
<div class="content-intro view-box ">
## <h2>RESTORE

RESTORE用于还原Solr索引和相关配置。
      
  
```
/admin/collections?action=RESTORE&amp;name=myBackupName&amp;location=/path/to/my/shared/drive&amp;collection=myRestoredCollectionName
```
RESTORE操作将在collection参数中创建一个具有指定名称的集合。您无法将备份还原到同一个集合中。此外，目标集合不应该在调用API时出现，因为Solr会为您创建它。  
创建的集合将具有与原始集合相同数量的分片和副本，保留路由信息等。可选地，您可以覆盖下面记录的一些参数。  
在还原的同时，如果ZooKeeper中存在一个名称相同的configSet，则Solr将重新使用该名称，否则它将上载备份的ZooKeeper中的configSet并使用它。  
您可以使用集合CREATEALIAS命令来确保客户端不需要更改端点以针对新还原的集合进行查询或编制索引。  

### RESTORE参数

- collection  
    
        索引将被恢复到的集合。该参数是必需的。  
    
- location  
    
        共享驱动器上的RESTORE命令的读取位置。或者，可以将其设置为群集属性。  
    
- async  
    
        请求ID来跟踪这个将被异步处理的动作。  
    
- repository  
    
        要用于备份的存储库的名称。如果没有指定仓库，那么本地文件系统仓库将被自动使用。  
    

<b>覆盖参数</b>
  
另外，在还原备份时，可能会覆盖原始集合上可能已经设置的几个参数：  
- collection.configName  
    
        定义用于此集合的配置的名称。这些必须已经存储在ZooKeeper中。如果没有提供，Solr将默认为集合名称作为配置名称。  
    
- replicationFactor  
    
        要为每个分片创建的副本数量。  
    
- maxShardsPerNode  
    
        创建集合时，分片或副本分布在所有可用的（即活动的）节点上，并且同一分片的两个副本永远不会在同一个节点上。  
        
            如果一个节点在调用CREATE操作时不存在，它将不会获得新集合的任何部分，这可能会导致在单个活动节点上创建太多副本。定义<code>maxShardsPerNode</code>来设置复制CREATE操作的数量的限制将扩展到每个节点。。如果整个集合不能适应活节点，则根本不会创建任何集合。  
          
    
- autoAddReplicas  
    
        设置<code>true</code>为时，将启用自动添加共享文件系统上的副本。有关设置和覆盖的详细信息，请参阅 SolrCloud 中的 "自动添加副本" 部分。  
    
- property.name=value  
   
        将核心属性name设置为value。有关受支持的属性和值的详细信息，请参阅定义core.properties一节。  
    
  
  
