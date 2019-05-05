## Solr：V2 API 
v2 API是一个现代化的自我记录API接口，覆盖了大多数当前的Solr API。预计一旦v2 API达到完全覆盖范围，Solr内部的API用法（如SolrJ和Admin UI）已经从旧的API转换到v2 API，旧的API将最终被淘汰。  
  
现在两种API风格将共存，所有旧的API将继续工作，没有任何改变。您可以通过此-Ddisable.v2.api=true系统属性启动您的服务器以禁用所有V2 API端点。  
旧的API和v2 API的不同在三个主要方面体现：  
1 <li>命令格式：通过HTTP GET请求的URL请求参数提供旧的API命令和相关的参数，而在v2 API中，大多数API命令是通过JSON体POST到v2 API端点提供的。v2 API还在适当的地方支持HTTP方法：GET和DELETE。</li>2 <li>端点结构：v2 API端点结构已合理化和规范化。</li>3 <li>文档：v2 API是自我记录的：附加/_introspect到任何有效的v2 API路径，API规范将以JSON格式返回。</li>
## v2 API路径前缀<a href="http://lucene.apache.org/solr/guide/7_0/v2-api.html#v2-api-path-prefixes"/>
以下是一些v2 API URL路径和路径前缀，以及在这些路径及其子路径中支持的一些操作。  
<table class=""><colgroup><col/><col/></colgroup><thead><tr><th style="text-align: center;">路径前缀</th><th style="text-align: center;">一些支持的操作</th></tr></thead><tbody><tr><td><p style="text-align: center;"><code>/api/collections</code> 或 <code>/api/c</code>  
</td><td><p style="text-align: center;">创建，别名，备份和恢复集合  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/update</code>  
</td><td><p style="text-align: center;">更新请求  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/config</code>  
</td><td><p style="text-align: center; ">配置请求  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/schema</code>  
</td><td><p style="text-align: center;">架构请求  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/<em>handler-name</em></code>  
</td><td><p style="text-align: center;">处理程序特定的请求  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/shards</code>  
</td><td><p style="text-align: center;">拆分碎片，创建碎片，添加副本  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/shards/<em>shard-name</em></code>  
</td><td><p style="text-align: center;">删除碎片，强制领导人选举  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/<em>collection-name</em>/shards/<em>shard-name</em>/<em>replica-name</em></code>  
</td><td><p style="text-align: center;">删除副本  
</td></tr><tr><td><p style="text-align: center;"><code>/api/cores</code>  
</td><td><p style="text-align: center;">创建一个核心  
</td></tr><tr><td><p style="text-align: center;"><code>/api/cores/<em>core-name</em></code>  
</td><td><p style="text-align: center;">重新加载，重命名，删除和卸载核心  
</td></tr><tr><td><p style="text-align: center;"><code>/api/node</code>  
</td><td><p style="text-align: center;">执行监督操作，重新组织领导选举  
</td></tr><tr><td><p style="text-align: center;"><code>/api/cluster</code>  
</td><td><p style="text-align: center;">添加角色，删除角色，设置集群属性  
</td></tr><tr><td><p style="text-align: center;"><code>/api/c/.system/blob</code>  
</td><td><p style="text-align: center;">上传和下载blob和元数据  
</td></tr></tbody></table>
## Introspect<a href="http://lucene.apache.org/solr/guide/7_0/v2-api.html#introspect"/>
附加/_introspect到任何有效的v2 API路径，API规范将以JSON格式返回：  
  
```
http://localhost:8983/api/c/_introspect
```
为了限制introspect输出到仅包括一个特定的HTTP方法中，请添加具有值GET、POST或DELETE的请求参数method：  
```
http://localhost:8983/api/c/_introspect?method=POST
```
大多数端点支持通过POST发送的主体中提供的命令。要将反省输出限制为只有一个命令，请添加请求参数：command=command-name，  
```
http://localhost:8983/api/c/gettingstarted/_introspect?method=POST&amp;command=modify
```
### 解释Introspect输出

### <a href="http://lucene.apache.org/solr/guide/7_0/v2-api.html#interpreting-the-introspect-output"/>
例如：http://localhost:8983/api/c/gettingstarted/get/_introspect  
```
{
  "spec":[{
      "documentation":"https://lucene.apache.org/solr/guide/real-time-get.html",
      "description":"RealTime Get allows retrieving documents by ID before the documents have been committed to the index. It is useful when you need access to documents as soon as they are indexed but your commit times are high for other reasons.",
      "methods":["GET"],
      "url":{
        "paths":["/c/gettingstarted/get"],
        "params":{
          "id":{
            "type":"string",
            "description":"A single document ID to retrieve."},
          "ids":{
            "type":"string",
            "description":"One or more document IDs to retrieve. Separate by commas if more than one ID is specified."},
          "fq":{
            "type":"string",
            "description":"An optional filter query to add to the query. One use case for this is security filtering, in case users or groups should not be able to retrieve the document ID requested."}}}}],
  "WARNING":"This response format is experimental.  It is likely to change in the future.",
  "availableSubPaths":{}
}
```
文档: 此 API 的联机 Solr 参考指南部分的 URL  
描述: 特征/变量/命令等的文本描述。  
规范/方法: 此 API 支持的 HTTP 方法  
规范/url/路径: 此 API 支持的 url 路径  
规范/url/参数: 支持的 url 请求参数列表  
availableSubPaths: 有效 URL 路径和 HTTP 方法的列表每个支持  
  
  
  
对上面的例子中的一些键的描述：  
- documentation ：该API的联机Solr参考指南部分的URL
- description ：特征/变量/命令（feature/variable/command）的文本描述等  
- spec/methods ：此API支持的HTTP方法
- spec/url/paths ：此API支持的URL路径
- spec/url/params ：支持的URL请求参数列表
- availableSubPaths ：有效的URL子路径列表和每个支持的HTTP方法
introspect的一个POST API的例子： http://localhost:8983/api/c/gettingstarted/_introspect?method=POST&amp;command=modify  
```
{
  "spec":[{
      "documentation":"https://lucene.apache.org/solr/guide/collections-api.html",
      "description":"Several collection-level operations are supported with this endpoint: modify collection attributes; reload a collection; migrate documents to a different collection; rebalance collection leaders; balance properties across shards; and add or delete a replica property.",
      "methods":["POST"],
      "url":{"paths":["/collections/{collection}",
          "/c/{collection}"]},
      "commands":{"modify":{
          "documentation":"https://lucene.apache.org/solr/guide/collections-api.html#modifycollection",
          "description":"Modifies specific attributes of a collection. Multiple attributes can be changed at one time.",
          "type":"object",
          "properties":{
            "rule":{
              "type":"array",
              "documentation":"https://lucene.apache.org/solr/guide/rule-based-replica-placement.html",
              "description":"Modifies the rules for where replicas should be located in a cluster.",
              "items":{"type":"string"}},
            "snitch":{
              "type":"array",
              "documentation":"https://lucene.apache.org/solr/guide/rule-based-replica-placement.html",
              "description":"Details of the snitch provider",
              "items":{"type":"string"}},
            "autoAddReplicas":{
              "type":"boolean",
              "description":"When set to true, enables auto addition of replicas on shared file systems (such as HDFS). See https://lucene.apache.org/solr/guide/running-solr-on-hdfs.html for more details on settings and overrides."},
            "replicationFactor":{
              "type":"string",
              "description":"The number of replicas to be created for each shard. Replicas are physical copies of each shard, acting as failover for the shard. Note that changing this value on an existing collection does not automatically add more replicas to the collection. However, it will allow add-replica commands to succeed."},
            "maxShardsPerNode":{
              "type":"integer",
              "description":"When creating collections, the shards and/or replicas are spread across all available, live, nodes, and two replicas of the same shard will never be on the same node. If a node is not live when the collection is created, it will not get any parts of the new collection, which could lead to too many replicas being created on a single live node. Defining maxShardsPerNode sets a limit on the number of replicas can be spread to each node. If the entire collection can not be fit into the live nodes, no collection will be created at all."}}}}}],
  "WARNING":"This response format is experimental.  It is likely to change in the future.",
  "availableSubPaths":{
    "/c/gettingstarted/select":["POST", "GET"],
    "/c/gettingstarted/config":["POST", "GET"],
    "/c/gettingstarted/schema":["POST", "GET"],
    "/c/gettingstarted/export":["POST", "GET"],
    "/c/gettingstarted/admin/ping":["POST", "GET"],
    "/c/gettingstarted/update":["POST"]}
}
```
以上示例中的"commands"部分对于此端点支持的每个命令都有一个条目。关键是命令名，值是使用JSON架构描述命令结构的json对象（参见[http://json-schema.org/](http://json-schema.org/)）。  
## 调用示例<a href="http://lucene.apache.org/solr/guide/7_0/v2-api.html#invocation-examples"/>
对于“gettingstarted”集合，请设置复制因子以及是否自动添加副本（请参阅上面的"modify"用于此处使用的命令的introspect输出）：  
```
$ curl http://localhost:8983/api/c/gettingstarted -H 'Content-type:application/json' -d '
{ modify: { replicationFactor: "3", autoAddReplicas: false } }'
{"responseHeader":{"status":0,"QTime":842}}
```
查看集群的状态：  
```
$ curl http://localhost:8983/api/cluster
{"responseHeader":{"status":0,"QTime":0},"collections":["gettingstarted",".system"]}
```
设置一个集群属性：  
```
$ curl http://localhost:8983/api/cluster -H 'Content-type: application/json' -d '
{ set-property: { name: autoAddReplicas, val: "false" } }'
{"responseHeader":{"status":0,"QTime":4}}
```
