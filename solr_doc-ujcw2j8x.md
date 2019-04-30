## DELETEREPLICAPROP：删除副本属性 
<div class="content-intro view-box ">从特定副本中删除任意属性。  
/admin/collections?action=DELETEREPLICAPROP&amp;collection=collectionName&amp;shard=shardName&amp;replica=replicaName&amp;property=propertyName  

### DELETEREPLICAPROP参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#deletereplicaprop-parameters"/>

- collection  

    
        副本所属的集合的名称。该参数是必需的。  
    
- shard
    
        副本所属分片的名称。该参数是必需的。  
    

### DELETEREPLICAPROP响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#deletereplicaprop-response"/>

响应将包括请求的状态。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用DELETEREPLICAPROP的示例<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-deletereplicaprop"/>

DELETEREPLICAPROP示例输入：  
这个命令会从core_node1中删除preferredLeader（property.preferredLeader）。  
```
http://localhost:8983/solr/admin/collections?action=DELETEREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node1&amp;property=preferredLeader
```

DELETEREPLICAPROP示例输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;9&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
