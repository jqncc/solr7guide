## MOVEREPLICA：将副本移到新节点 
<div class="content-intro view-box ">
## MOVEREPLICA<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#movereplica-move-a-replica-to-a-new-node"/>

MOVEREPLICA命令将副本从一个节点移动到新节点。在共享文件系统的情况下，dataDir将被重用。
      
  
```
/admin/collections?action=MOVEREPLICA&amp;collection=collection&amp;shard=shard&amp;replica=replica&amp;sourceNode=nodeName&amp;targetNode=nodeName
```

### MOVEREPLICA参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#movereplica-parameters"/>

- collection  
    
        集合的名称。该参数是必需的。  
    
- shard  
    
        该副本所属的分片的名称。该参数是必需的。  
    
- replica  
    
        副本的名称。该参数是必需的。  
    
- sourceNode  
    
        包含副本的节点的名称。该参数是必需的。  
    
- targetNode  
    
        目标节点的名称。该参数是必需的。  
    
- async  
   
        请求ID来跟踪这个将被异步处理的动作。  

