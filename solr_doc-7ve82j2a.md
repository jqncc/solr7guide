## Solr群集属性：CLUSTERPROP 
<div class="content-intro view-box ">在Solr中，通过CLUSTERPROP操作能够添加、编辑或删除群集范围的属性。  
  
/admin/collections?action=CLUSTERPROP&amp;name=propertyName&amp;val=propertyValue  

### CLUSTERPROP参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#clusterprop-parameters"/>

**name**
    
        属性的名称。支持的属性名称是<code>urlScheme</code>和<code>autoAddReplicas and location</code>。其他名称被拒绝并出现错误。  
    
**val**
    
        属性的值。如果值为空或为null，则该属性未设置。  
    


### CLUSTERPROP响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#clusterprop-response"/>

CLUSTERPROP响应将包括请求的状态以及更新或删除的属性。如果状态不是“0”，则会显示一条错误消息，说明请求失败的原因。  

### 使用CLUSTERPROP的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-clusterprop"/>

这个例子的输入为：  
```
http://localhost:8983/solr/admin/collections?action=CLUSTERPROP&amp;name=urlScheme&amp;val=https
```
将得到如下输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
