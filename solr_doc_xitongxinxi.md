## Solr系统信息 
<div class="content-intro view-box ">
## Solr 版本信息

version 命令只是返回当前安装的 Solr 版本，并且立即存在。  
  
```
$ bin/solr version
X.Y.0
```

## Solr 状态信息

### <a href="http://lucene.apache.org/solr/guide/7_0/solr-control-script-reference.html#status"/>

status 命令显示在本地系统上运行的任何 Solr 节点的基本 JSON 格式的信息。  
  
status 命令使用 SOLR_PID_DIR 环境变量来定位 Solr 进程 ID 文件以查找正在运行的 Solr 实例，默认情况下是 bin 目录。  
```
bin/solr status
```
输出将包含群集中每个节点的状态，如下例所示：  
```
找到2个 Solr 节点:
Solr process 39920 running on port 7574
{
  "solr_home":"/Applications/Solr/example/cloud/node2/solr/",
  "version":"X.Y.0",
  "startTime":"2015-02-10T17:19:54.739Z",
  "uptime":"1 days, 23 hours, 55 minutes, 48 seconds",
  "memory":"77.2 MB (%15.7) of 490.7 MB",
  "cloud":{
    "ZooKeeper":"localhost:9865",
    "liveNodes":"2",
    "collections":"2"}}
Solr process 39827 running on port 8865
{
  "solr_home":"/Applications/Solr/example/cloud/node1/solr/",
  "version":"X.Y.0",
  "startTime":"2015-02-10T17:19:49.057Z",
  "uptime":"1 days, 23 hours, 55 minutes, 54 seconds",
  "memory":"94.2 MB (%19.2) of 490.7 MB",
  "cloud":{
    "ZooKeeper":"localhost:9865",
    "liveNodes":"2",
    "collections":"2"}}
```

## Healthcheck
在 SolrCloud 模式下运行时，healthcheck 命令将为集合生成 JSON 格式的运行状况报告。运行状况报告提供有关集合中所有碎片的每个副本状态的信息，包括已提交文档的数量及其当前状态。  
```
bin/solr healthcheck [options]
bin/solr healthcheck -help
```

### 健康检查参数

#### <a href="http://lucene.apache.org/solr/guide/7_0/solr-control-script-reference.html#healthcheck-parameters"/>

**-c &lt;collection&gt;**
    
        （要求）运行健康检查的集合的名称。例如：  
```
bin/solr healthcheck -c gettingstarted
```
          
    
**-z &lt;zkhost&gt;**
    
        ZooKeeper 连接字符串，默认为<code>localhost:9983</code>。如果您在8983以外的端口上运行 Solr，则必须指定 ZooKeeper 连接字符串。默认情况下，这将是 Solr 端口 + 1000。例如：  
```
bin/solr healthcheck -z localhost:2181
```
          
    

以下是使用非标准 ZooKeeper 连接字符串运行的 healthcheck 请求和响应示例，其中包含2个节点：  
```
$ bin/solr healthcheck -c gettingstarted -z localhost:9865
```
    
  
```
{
  "collection":"gettingstarted",
  "status":"healthy",
  "numDocs":0,
  "numShards":2,
  "shards":[
    {
      "shard":"shard1",
      "status":"healthy",
      "replicas":[
        {
          "name":"core_node1",
          "url":"http://10.0.1.10:8865/solr/gettingstarted_shard1_replica2/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 48 seconds",
          "memory":"25.6 MB (%5.2) of 490.7 MB",
          "leader":true},
        {
          "name":"core_node4",
          "url":"http://10.0.1.10:7574/solr/gettingstarted_shard1_replica1/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 42 seconds",
          "memory":"95.3 MB (%19.4) of 490.7 MB"}]},
    {
      "shard":"shard2",
      "status":"healthy",
      "replicas":[
        {
          "name":"core_node2",
          "url":"http://10.0.1.10:8865/solr/gettingstarted_shard2_replica2/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 48 seconds",
          "memory":"25.8 MB (%5.3) of 490.7 MB"},
        {
          "name":"core_node3",
          "url":"http://10.0.1.10:7574/solr/gettingstarted_shard2_replica1/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 42 seconds",
          "memory":"95.4 MB (%19.4) of 490.7 MB",
          "leader":true}]}]}
```
