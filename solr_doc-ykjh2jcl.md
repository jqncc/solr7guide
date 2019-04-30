## REPLACENODE：将节点中的所有副本移到另一个节点 
<div class="content-intro view-box ">
## REPLACENODE

REPLACENODE命令将一个节点（源）中的副本重新创建到另一个节点（目标）。在复制每个副本之后，将删除源节点中的副本。
      
  
对于也是碎片leader程序的源副本，操作将等待使用该timeout参数设置的秒数，以确保存在可成为leader的活动副本（现有副本成为leader或新副本完成恢复并成为leader）。  
```
/admin/collections?action=REPLACENODE&amp;sourceNode=source-node&amp;targetNode=target-node
```

### REPLACENODE参数

- sourceNode  
    
        需要从中复制副本的源节点。该参数是必需的。  
    
- targetNode  
   
        将复制副本的目标节点。该参数是必需的。  
    
- parallel  
    
        如果此标志设置为<code>true</code>，则所有副本都在不同的线程中创建。请记住，如果副本具有非常大的索引，这可能会导致非常高的网络和磁盘I / O。默认是<code>false</code>。  
    
- async  
    
        请求ID来跟踪这个将被异步处理的动作。  
    
- timeout  
   
        以秒为单位的时间，直到创建新的副本，直到完全恢复主副本。默认值是<code>300</code>5分钟。  
    

此操作不保留属于源节点上的副本所需的锁定。所以在此期间不要进行其他集合操作。  
