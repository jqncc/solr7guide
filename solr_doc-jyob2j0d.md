## Solr删除集合别名：DELETEALIAS 
<div class="content-intro view-box ">/admin/collections?action=DELETEALIAS&amp;name=name  

### DELETEALIAS参数

**name**
    
        要删除的别名的名称。该参数是必需的。  
    
**async**
    
        请求ID来跟踪这个将被异步处理的动作。  
    


### DELETEALIAS响应

输出将只是一个responseHeader，其中包含处理请求所花费的时间的详细信息。要确认删除别名，您可以查看Solr Admin UI中Cloud部分下的aliases.json文件。  

### 使用DELETEALIAS的例子

输入如下：  
删除名为“testalias”的别名：  
```
http://localhost:8983/solr/admin/collections?action=DELETEALIAS&amp;name=testalias
```
将会有如下的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;117&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
