## BALANCESHARDUNIQUE: 在节点之间平衡属性 
<div class="content-intro view-box ">
## BALANCESHARDUNIQUE<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#balanceshardunique"/>

/admin/collections?action=BALANCESHARDUNIQUE&amp;collection=collectionName&amp;property=propertyName
      
  
确保某个特定属性在组成集合的物理节点之间均匀分布。如果复制副本上已存在该属性，则尽一切努力将其保留在那里。如果该属性不在碎片上的任何副本上，则选择一个，并添加该属性。  

### BALANCESHARDUNIQUE参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#balanceshardunique-parameters"/>

- collection  
    
        用于平衡属性的集合的名称。此参数是必需的。  
    
- property  
   
        要平衡的属性。如果未明确指定，则该字面值<code>property.</code>将被预置为该属性。该参数是必需的。  
    
- onlyactivenodes  
    
        默认为<code>true</code>。通常，该属性仅在活动节点上实例化。如果将此参数指定为<code>false</code>，则也会包含非活动节点进行分发。  
    
- shardUnique  
    
        一些安全值。有一个预定义的属性（<code>preferredLeader</code>），默认这个值为<code>true</code>。对于所有其他平衡的属性，必须将其设置为<code>true</code>，否则将返回错误消息。  
    

### BALANCESHARDUNIQUE响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#balanceshardunique-response"/>

响应将包括请求的状态。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用BALANCESHARDUNIQUE的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-balanceshardunique"/>

在这个例子中输入：  
这两个命令中的任何一个都会将“preferredLeader”属性放在“collection1”集合中每个分片的一个副本上。  
```
http://localhost:8983/solr/admin/collections?action=BALANCESHARDUNIQUE&amp;collection=collection1&amp;property=preferredLeader
http://localhost:8983/solr/admin/collections?action=BALANCESHARDUNIQUE&amp;collection=collection1&amp;property=property.preferredLeader
```
得到输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;9&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
在发出此调用后检查 clusterstate 应在每个具有此属性的碎片中显示一个副本。  
