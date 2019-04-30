## RELOAD：重新加载一个集合 
<div class="content-intro view-box ">/admin/collections?action=RELOAD&amp;name=name  
  
在更改ZooKeeper中的配置时将使用RELOAD操作。  

### RELOAD参数

**name**
    
        要重新加载的集合的名称。该参数是必需的。  
    
**async**
    
        请求ID来跟踪这个将被异步处理的操作。  
    


### RELOAD响应

响应将包括请求的状态和重新加载的核心。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 使用RELOAD的例子

您输入以下内容：  
```
http://localhost:8983/solr/admin/collections?action=RELOAD&amp;name=newCollection
```
则会得到如下输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1551&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst name="10.0.1.6:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;761&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
    &lt;lst name="10.0.1.4:8983_solr"&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;1527&lt;/int&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
