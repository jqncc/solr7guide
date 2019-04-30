## SolrCloud删除角色：REMOVEROLE 
<div class="content-intro view-box ">REMOVEROLE命令可以用来删除指定的角色。该API用于撤消使用ADDROLE操作分配的角色。  
/admin/collections?action=REMOVEROLE&amp;role=roleName&amp;node=nodeName  

### REMOVEROLE参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#removerole-parameters"/>

**role**
    
        角色的名称。目前唯一受支持的角色是<code>overseer</code>。该参数是必需的。  
    
**node**
    
        应该删除角色的节点的名称。  
    


### REMOVEROLE响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#removerole-response"/>

响应将包括请求的状态以及更新或删除的属性。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用REMOVEROLE的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-removerole"/>

输入如下：  
```
http://localhost:8983/solr/admin/collections?action=REMOVEROLE&amp;role=overseer&amp;node=192.167.1.2:8983_solr
```
得到的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
