## REQUESTSTATUS：异步呼叫的请求状态 
<div class="content-intro view-box ">REQUESTSTATUS操作可以请求已提交的异步集合API（以下）调用的状态和响应。此调用也用于清除存储的状态。  
  
/admin/collections?action=REQUESTSTATUS&amp;requestid=request-id  

### REQUESTSTATUS参数

**requestid**
    
        用户为请求定义的请求ID。这可以用来跟踪提交的异步任务的状态。该参数是必需的。  
    


### 使用REQUESTSTATUS的例子

在该例中输入：有效的请求ID  
```
http://localhost:8983/solr/admin/collections?action=REQUESTSTATUS&amp;requestid=1000
```
得到的输出为：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="status"&gt;
    &lt;str name="state"&gt;completed&lt;/str&gt;
    &lt;str name="msg"&gt;found 1000 in completed tasks&lt;/str&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
或者输入：无效的请求ID  
```
http://localhost:8983/solr/admin/collections?action=REQUESTSTATUS&amp;requestid=1004
```
得到的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;1&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="status"&gt;
    &lt;str name="state"&gt;notfound&lt;/str&gt;
    &lt;str name="msg"&gt;Did not find taskid [1004] in any tasks queue&lt;/str&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
