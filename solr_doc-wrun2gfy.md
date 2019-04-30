## Solr内容流 
<div class="content-intro view-box ">内容流是通过对 Solr 的请求传递的大容量数据。  
  
当使用基于路径的 URL 访问 Solr RequestHandler 时，包含请求参数的 SolrQueryRequest 对象也可能包含包含请求的大容量数据的 ContentStreams 列表。（名称 SolrQueryRequest 有点误导：无论是查询请求还是更新请求，都涉及所有请求。）  

## 内容流来源<a href="http://lucene.apache.org/solr/guide/7_0/content-streams.html#content-stream-sources"/>

当前请求处理程序可以通过多种方式获取内容流：  
  

    - 对于多部分文件上传，每个文件都以流的形式传递。
    - 对于内容类型不是 application/x-www-form-urlencoded 的 POST 请求，原始的 POST 主体将作为流传递。完整的 POST 主体被解析为参数并包含在 Solr 参数中。
    - 参数 stream.body 的内容作为流传递。
    - 如果启用了远程流并在请求处理期间调用了 URL 内容，则每个参数 stream.url 和 stream.file 参数的内容将作为流获取并传递。

默认情况下，curl 发送一个 contentType="application/x-www-form-urlencoded" 头。如果您需要测试 SolrContentHeader 内容流，则需要使用 curl -H 标志来设置内容类型。  

## 使用 RemoteStreaming<a href="http://lucene.apache.org/solr/guide/7_0/content-streams.html#remotestreaming"/>

通过远程流传输，您可以将 URL 的内容作为流发送到给定的 SolrRequestHandler。您可以使用远程流发送远程或本地文件到更新插件。  
  
默认情况下禁用远程流。在生产环境中不建议启用它，但是在您与不可信的远程客户端之间不需要额外的安全性。  
```
*** WARNING ***
在启用远程流处理之前, 应确保系统已启用身份验证。
&lt;requestParsers enableRemoteStreaming="false"...&gt;
```
如果未在 solrconfig. xml 中指定 enableRemoteStreaming，则默认行为是不允许远程流 (即 enableRemoteStreaming = "false")。  
  
远程流也可以通过 Config API 启用，如下所示：  
```
curl -d '
{
  "set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true}
}
' http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json'
```
注意：如果使用 enableRemoteStreaming = "true"，请注意，这允许任何人向任何 URL 或本地文件发送请求。如果启用了 DumpRequestHandler，它将允许任何人查看您系统上的任何文件。  

## 调试请求<a href="http://lucene.apache.org/solr/guide/7_0/content-streams.html#debugging-requests"/>

隐式“转储” RequestHandler（请参阅 Implicit RequestHandlers）只是使用指定的编写器类型 wt 来输出 SolrQueryRequest 的内容。这是一个有用的工具，有助于了解哪些流可用于 RequestHandlers。  
