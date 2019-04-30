## SolrCloud添加一个角色：ADDROLE 
<div class="content-intro view-box ">/admin/collections?action=ADDROLE&amp;role=roleName&amp;node=nodeName  
为集群中的给定节点分配一个角色。唯一受支持的角色是overseer。  
  
使用这个命令将一个特定的节点专用为Overseer。多次调用它以添加更多的节点。这在Overseer可能超载的大型集群中很有用。如果可用的话，被分配给“overseer”角色的节点列表中的一个将成为overseer。如果没有任何指定的节点处于运行状态，则系统会将该角色分配给任何其他节点。  

### ADDROLE参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#addrole-parameters"/>

**role**
    
        角色的名称。目前唯一受支持的角色是<code>overseer</code>。该参数是必需的。  
    
**node**
    
        将被分配角色的节点的名称。即使在该节点启动之前，也可以分配一个角色。该参数已启动。  
    


### ADDROLE响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#addrole-response"/>

响应将包括请求的状态以及更新或删除的属性。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用ADDROLE的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-addrole"/>

输入如下：  
```
http://localhost:8983/solr/admin/collections?action=ADDROLE&amp;role=overseer&amp;node=192.167.1.2:8983_solr
```
得到的输出是：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
