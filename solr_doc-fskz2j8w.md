## ADDREPLICAPROP：添加副本属性 
<div class="content-intro view-box ">将任意属性分配给特定副本，并为其指定值。如果该属性已经存在，则会被新值覆盖。
      
  
/admin/collections?action=ADDREPLICAPROP&amp;collection=collectionName&amp;shard=shardName&amp;replica=replicaName&amp;property=propertyName&amp;property.value=value  

### ADDREPLICAPROP参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#addreplicaprop-parameters"/>

- collection  

    
        副本所属的集合的名称。该参数是必需的。  
    
- shard  

    
        副本所属分片的名称。该参数是必需的。  
    
- replica  

    
        副本，例如<code>core_node1</code>。该参数是必需的。  
    
- property  

    
        要添加的属性的名称。此属性是必需的。  
        
            这将具有<code>property.</code>将其与系统维护属性区分开来的字面意思。所以这两种形式是等价的：  
          
        
            <code>property=special</code>和 <span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: nowrap;">property=property.special</span>
              
          
    
- property.value  

   
        要分配给该属性的值。该参数是必需的。  
    
- shardUnique  

    
        如果为<code>true</code>，那么在一个副本中设置此属性将从该分片中的所有其他副本中删除该属性。默认是<code>false</code>。  
        
            有一个预先定义的属性<code>preferredLeader</code>，其中<code>shardUnique</code>被要求为<code>true</code>如果<code>shardUnique</code>明确设置为<code>false</code>，则返回一个错误。  
          
        
            <code>PreferredLeader</code>是一个布尔属性。任何赋值不相等（不区分大小写）的值<code>true</code>将被解释为<code>preferredLeader</code>的<code>false</code>。  
          
    


### ADDREPLICAPROP响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#addreplicaprop-response"/>

响应将包括请求的状态。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用ADDREPLICAPROP的示例<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-addreplicaprop"/>

<b>ADDREPLICAPROP示例输入</b>
  
这个命令会在“core_node1”上设置“preferredLeader”属性（property.preferredLeader）为“true”，并从该分片中的任何其他副本中删除该属性。  
```
http://localhost:8983/solr/admin/collections?action=ADDREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node1&amp;property=preferredLeader&amp;property.value=true
```

<b>ADDREPLICAPROP示例输出</b>
  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;46&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
<b>ADDREPLICAPROP示例输入</b>
      
  
这对命令将把 "testprop" 属性 （property.testprop）分别设置为 "value1" 和 "value2"，用于同一碎片中的两个节点。  
```
http://localhost:8983/solr/admin/collections?action=ADDREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node1&amp;property=testprop&amp;property.value=value1
http://localhost:8983/solr/admin/collections?action=ADDREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node3&amp;property=property.testprop&amp;property.value=value2
```

<b>ADDREPLICAPROP示例输入</b>
      
  
这一对命令将导致“core_node_3”具有“testprop”属性（property.testprop）值的设置，因为第二个命令指定shardUnique=true，这将导致属性从“core_node_1”中删除。
      
  
```
http://localhost:8983/solr/admin/collections?action=ADDREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node1&amp;property=testprop&amp;property.value=value1
http://localhost:8983/solr/admin/collections?action=ADDREPLICAPROP&amp;shard=shard1&amp;collection=collection1&amp;replica=core_node3&amp;property=testprop&amp;property.value=value2&amp;shardUnique=true
```
