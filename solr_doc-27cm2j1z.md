## Solr删除一个集合：DELETE 
<div class="content-intro view-box ">/admin/collections?action=DELETE&amp;name=collection  

### DELETE参数

**name**
    
        要删除的集合的名称。该参数是必需的。  
    
**async**
    
        请求ID来跟踪这个将被异步处理的动作。  
    


### DELETE响应

响应将包括请求的状态和被删除的核心。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 使用DELETE的例子

在该例子中的输入：  
删除名为“newCollection”的集合。  
```
http://localhost:8983/solr/admin/collections?action=DELETE&amp;name=newCollection
```
会得到如下的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;603&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst name="10.0.1.6:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;19&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst name="10.0.1.4:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;67&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
