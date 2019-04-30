## SolrCloud删除状态：DELETESTATUS 
<div class="content-intro view-box ">删除已经失败或已经完成的Asynchronous Collection API调用的存储响应。  
  
/admin/collections?action=DELETESTATUS&amp;requestid=request-id  

### DELETESTATUS参数

**requestid**
    
        存储响应应该被清除的异步调用的请求ID。  
    
**flush**
    
        设置为<code>true</code>来清除所有存储的完成和失败的异步请求响应。  
    


### 使用DELETESTATUS的示例

在本例中输入：有效的请求ID  
```
http://localhost:8983/solr/admin/collections?action=DELETESTATUS&amp;requestid=foo
```
得到的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;str name="status"&gt;successfully removed stored response for [foo]&lt;/str&gt;
&lt;/response&gt;
```
或者输入：无效的请求ID  
```
http://localhost:8983/solr/admin/collections?action=DELETESTATUS&amp;requestid=bar
```
得到输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;str name="status"&gt;[bar] not found in stored responses&lt;/str&gt;
&lt;/response&gt;
```
或者输入：清除所有存储的状态  
```
http://localhost:8983/solr/admin/collections?action=DELETESTATUS&amp;flush=true
```
得到输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;str name="status"&gt; successfully cleared stored collection api responses &lt;/str&gt;
&lt;/response&gt;
```
