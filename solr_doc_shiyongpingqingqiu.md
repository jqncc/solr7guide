## 使用Ping请求 
<div class="content-intro view-box ">在核心名称下选择 Ping 会发出一个 ping 请求来检查核心是否启动并响应请求。  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510217589638867.png)
  
由 Ping 执行的搜索是使用请求参数 API 进行配置的。请参阅 Implicit RequestHandlers，以了解用于 /admin/ping 端点的参数集。  
  
Ping 选项不打开页面，但是在点击集合名称时显示的核心概览页面上可以看到请求的状态。请求的时间长度显示在 Ping 选项旁边，以毫秒为单位。  

## API 示例<a href="http://lucene.apache.org/solr/guide/7_0/ping.html#api-examples"/>

虽然在 UI 界面上可以很容易地看到 ping 响应时间，但是当由远程监视工具执行时，底层 ping 命令会更加有用：  
  
输入如下：  
```
http://localhost:8983/solr/&lt;core-name&gt;/admin/ping
```
这个命令将 ping 一个响应的核心名称。  
输入如下：  
```
http://localhost:8983/solr/&lt;collection-name&gt;/admin/ping?distrib=true
```
此命令将为响应 ping 给定集合名称的所有副本。  
示例输出：  
```
&lt;response&gt;
   &lt;lst name="responseHeader"&gt;
      &lt;int name="status"&gt;0&lt;/int&gt;
      &lt;int name="QTime"&gt;13&lt;/int&gt;
      &lt;lst name="params"&gt;
         &lt;str name="q"&gt;{!lucene}*:*&lt;/str&gt;
         &lt;str name="distrib"&gt;false&lt;/str&gt;
         &lt;str name="df"&gt;_text_&lt;/str&gt;
         &lt;str name="rows"&gt;10&lt;/str&gt;
         &lt;str name="echoParams"&gt;all&lt;/str&gt;
      &lt;/lst&gt;
   &lt;/lst&gt;
   &lt;str name="status"&gt;OK&lt;/str&gt;
&lt;/response&gt;
```
这两个 API 调用都有相同的输出。status=OK 表示节点正在响应。  
SolrJ 示例：  
```
SolrPing ping = new SolrPing();
ping.getParams().add("distrib", "true"); //To make it a distributed request against a collection
rsp = ping.process(solrClient, collectionName);
int status = rsp.getStatus();
```
