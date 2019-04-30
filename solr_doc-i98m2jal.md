## REBALANCELEADERS：重新平衡leader 
<div class="content-intro view-box ">
## REBALANCELEADERS<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#rebalanceleaders"/>

根据活动节点上的preferredLeader属性重新分配集合中的leader。
      
  
/admin/collections?action=REBALANCELEADERS&amp;collection=collectionName  
根据活动节点上的preferredLeader属性将leader分配到一个集合中。在通过BALANCESHARDUNIQUE或ADDREPLICAPROP命令分配preferredLeader属性之后，应执行此命令。  
并不要求集合中的所有碎片都有<code>preferredLeader</code>属性。再平衡将只尝试重新分配领导那些将<code>preferredLeader</code>属性设置为<code>true</code> 的副本，而不是当前的碎片leader和当前的活动状态。  

### REBALANCELEADERS参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#rebalanceleaders-parameters"/>

- collection  
    
        要重新平衡的集合的名称<code>preferredLeaders</code>。该参数是必需的。  
    
- maxAtOnce  
    
        一次排队的最大重新分配数量。值&lt;= 0使用默认值Integer.MAX_VALUE。  
        
            当达到这个数字时，该过程等待一个或多个领导者被成功分配，然后再增加更多的队列。  
          
    
- maxWaitSeconds  
    
        默认为<code>60</code>。这是等待领导人重新分配时的超时值。如果<code>maxAtOnce</code>小于将要发生的重新分配的数量，则这是任何<em>一次</em>等待至少一次重新分配的最大间隔。  
        
            例如，如果10和再分配是发生和<code>maxAtOnce</code>是<code>1</code>与<code>maxWaitSeconds</code>是<code>60</code>，在该命令可以等待的时间的上限是10分钟。  
          
    

### REBALANCELEADERS响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#rebalanceleaders-response"/>

响应将包括请求的状态。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用REBALANCELEADERS的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-rebalanceleaders"/>

在这个例子中输入：  
这些命令中的任何一个都会导致所有具有preferredLeader属性设置的活动副本，并且不是已经成为首选的leader。  
```
http://localhost:8983/solr/admin/collections?action=REBALANCELEADERS&amp;collection=collection1
http://localhost:8983/solr/admin/collections?action=REBALANCELEADERS&amp;collection=collection1&amp;maxAtOnce=5&amp;maxWaitSeconds=30
```
得到输出：  
在这个例子中，“alreadyLeaders”部分中的两个副本已经将leader分配给与该preferredLeader属性相同的节点，因此没有采取任何操作。
      
  
“inactivePreferreds”部分中的副本具有preferredLeader属性集，但节点已关闭且不采取任何操作。“successes”部分的三个节点是leader，因为他们拥有preferredLeader属性集，但不是leader，他们是活跃的。  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;123&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="alreadyLeaders"&gt;
    &lt;lst name="core_node1"&gt;
      &lt;str name="status"&gt;success&lt;/str&gt;
      &lt;str name="msg"&gt;Already leader&lt;/str&gt;
      &lt;str name="nodeName"&gt;192.168.1.167:7400_solr&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="core_node17"&gt;
      &lt;str name="status"&gt;success&lt;/str&gt;
      &lt;str name="msg"&gt;Already leader&lt;/str&gt;
      &lt;str name="nodeName"&gt;192.168.1.167:7600_solr&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
  &lt;lst name="inactivePreferreds"&gt;
    &lt;lst name="core_node4"&gt;
      &lt;str name="status"&gt;skipped&lt;/str&gt;
      &lt;str name="msg"&gt;Node is a referredLeader, but it's inactive. Skipping&lt;/str&gt;
      &lt;str name="nodeName"&gt;192.168.1.167:7500_solr&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
  &lt;lst name="successes"&gt;
    &lt;lst name="_collection1_shard3_replica1"&gt;
      &lt;str name="status"&gt;success&lt;/str&gt;
      &lt;str name="msg"&gt;
        Assigned 'Collection: 'collection1', Shard: 'shard3', Core: 'collection1_shard3_replica1', BaseUrl:
        'http://192.168.1.167:8983/solr'' to be leader
      &lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="_collection1_shard5_replica3"&gt;
      &lt;str name="status"&gt;success&lt;/str&gt;
      &lt;str name="msg"&gt;
        Assigned 'Collection: 'collection1', Shard: 'shard5', Core: 'collection1_shard5_replica3', BaseUrl:
        'http://192.168.1.167:7200/solr'' to be leader
      &lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="_collection1_shard4_replica2"&gt;
      &lt;str name="status"&gt;success&lt;/str&gt;
      &lt;str name="msg"&gt;
        Assigned 'Collection: 'collection1', Shard: 'shard4', Core: 'collection1_shard4_replica2', BaseUrl:
        'http://192.168.1.167:7300/solr'' to be leader
      &lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
发出这个调用之后检查clustertate应该显示每个拥有该preferredLeader属性的活动节点，也应该将“leader”属性设置为true。  
