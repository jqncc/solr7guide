## Solr API：Blob Store 
<div class="content-intro view-box ">Blob Store REST API提供REST方法来存储、检索或列出Lucene索引中的文件。  
  
它可以用来上传包含标准solr组件（如 RequestHandlers、SearchComponents 或其他为 solr 编写的自定义代码）的 jar 文件。架构组件不还支持 Blob 存储。  
当使用blob存储时，请注意，如果上传了新名称，则API不会删除或覆盖之前的对象。它总是向索引添加一个新的blob版本。删除可以使用标准的REST删除命令来执行。  
Blob存储只能在SolrCloud模式下运行。独立模式下的Solr不支持使用Blob存储。  
blob store API是作为 requestHandler 实现的。使用名为“.system”的特殊集合来存储 blob。这个集合可以被预先创建，但是如果它不存在，它将被自动创建。  

## 关于.system集合<a href="http://lucene.apache.org/solr/guide/7_0/blob-store-api.html#about-the-system-collection"/>

在将blob上传到blob存储区之前，必须创建一个特殊的集合并且必须命名为.system。如果尚未存在，Solr将自动创建该集合，但是如果您选择，则您也可以手动创建它。  
  
BlobHandler 自动注册在.system集合中。该集合的 solrconfig.xml、架构和其他配置文件由系统自动提供，并不需要进行特别限定。  
如果不使用-shards或-replicationFactor选项，则将使用默认值：numShards = 1 和 replicationFactor = 3（或群集中的最大节点数）。  
您可以使用集合 API 创建 .system集合，如下例所示：  
```
curl http://localhost:8983/solr/admin/collections?action=CREATE&amp;name=.system&amp;replicationFactor=2
```
注意：该 <span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: nowrap;">bin/solr </span>脚本不能用于创建 <span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: nowrap;">.system </span>集合  

## 将文件上传到Blob Store<a href="http://lucene.apache.org/solr/guide/7_0/blob-store-api.html#upload-files-to-blob-store"/>
创建.system集合之后，可以将文件上传到blob store，请求类似于以下内容：  
```
curl -X POST -H 'Content-Type: application/octet-stream' --data-binary @{filename} http://localhost:8983/solr/.system/blob/{blobname}
```
例如，要将名为“test1.jar”的文件上传到名为“test”的blob，您可以发出POST请求，如：  
```
curl -X POST -H 'Content-Type: application/octet-stream' --data-binary @test1.jar http://localhost:8983/solr/.system/blob/test
```
GET请求将返回blob列表和其他详细信息：  
```
curl http://localhost:8983/solr/.system/blob?omitHeader=true
```
输出如下所示：  
```
{
  "response":{"numFound":1,"start":0,"docs":[
      {
        "id":"test/1",
        "md5":"20ff915fa3f5a5d66216081ae705c41b",
        "blobName":"test",
        "version":1,
        "timestamp":"2015-02-04T16:45:48.374Z",
        "size":13108}]
  }
}
```
有关各个blob的详细信息可以通过类似于以下的请求来访问：  
```
curl http://localhost:8983/solr/.system/blob/{blobname}
```
例如，这个请求将只返回名为“test”的blob：  
```
curl http://localhost:8983/solr/.system/blob/test?omitHeader=true
```
输出如下所示：  
```
{
  "response":{"numFound":1,"start":0,"docs":[
      {
        "id":"test/1",
        "md5":"20ff915fa3f5a5d66216081ae705c41b",
        "blobName":"test",
        "version":1,
        "timestamp":"2015-02-04T16:45:48.374Z",
        "size":13108}]
  }
}
```
文件流响应编写器可以返回一个特定版本的blob进行下载，如下所示：  
```
curl http://localhost:8983/solr/.system/blob/{blobname}/{version}?wt=filestream &gt; {outputfilename}
```
对于最新版本的blob，{version}可以省略，  
```
curl http://localhost:8983/solr/.system/blob/{blobname}?wt=filestream &gt; {outputfilename}
```

## 在处理程序或组件中使用Blob<a href="http://lucene.apache.org/solr/guide/7_0/blob-store-api.html#use-a-blob-in-a-handler-or-component"/>

要使用blob作为请求处理程序或搜索组件的类，可以在solrconfig.xml中照常创建请求处理程序。您将需要定义以下参数：  

**class**
    
        完全合格的类名。例如，如果您创建了一个名为CRUDHandler的新的请求处理程序类，则可以输入<code>org.apache.solr.core.CRUDHandler</code>。  
    
**runtimeLib**
    
        设置为true，要求该组件应从加载运行时 jar 的加载器中加载。。  
    

例如，要使用名为test的blob，您可以在solrconfig.xml中像这样配置：  
```
&lt;requestHandler name="/myhandler" class="org.apache.solr.core.myHandler" runtimeLib="true" version="1"&gt;
&lt;/requestHandler&gt;
```
如果自定义处理程序中有可用的参数, 则可以使用与任何其他请求处理程序定义相同的方式来定义它们。  
注意：Blob store 只能用于动态加载在 solrconfig. xml 中配置的组件。在 solrconfig. xml 中指定的组件不能从 blob store 加载。  
