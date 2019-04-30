## SolrCloud群集状态：CLUSTERSTATUS 
<div class="content-intro view-box ">CLUSTERSTATUS操作可以获取集群状态，包括集合、分片、副本、配置名称以及集合别名和集群属性。  
/admin/collections?action=CLUSTERSTATUS  

### CLUSTERSTATUS参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#clusterstatus-parameters"/>

- collection  

    
        请求信息的集合名称。如果省略，则将返回集群中所有集合的信息。  
    
- shard  

   
        要求信息的分片。可以将多个分片名称指定为以逗号分隔的列表。  
    
- _route_  

    
        如果您需要特定文档所属的分片的详细信息，并且您不知道它属于哪个分片，则可以使用此选项。  
    


### CLUSTERSTATUS响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#clusterstatus-response"/>

响应将包括请求的状态和集群的状态。  

### 使用CLUSTERSTATUS的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-clusterstatus"/>

在这个例子中我们的输入如下：  
```
http://localhost:8983/solr/admin/collections?action=CLUSTERSTATUS
```

得到输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":333},
  "cluster":{
    "collections":{
      "collection1":{
        "shards":{
          "shard1":{
            "range":"80000000-ffffffff",
            "state":"active",
            "replicas":{
              "core_node1":{
                "state":"active",
                "core":"collection1",
                "node_name":"127.0.1.1:8983_solr",
                "base_url":"http://127.0.1.1:8983/solr",
                "leader":"true"},
              "core_node3":{
                "state":"active",
                "core":"collection1",
                "node_name":"127.0.1.1:8900_solr",
                "base_url":"http://127.0.1.1:8900/solr"}}},
          "shard2":{
            "range":"0-7fffffff",
            "state":"active",
            "replicas":{
              "core_node2":{
                "state":"active",
                "core":"collection1",
                "node_name":"127.0.1.1:7574_solr",
                "base_url":"http://127.0.1.1:7574/solr",
                "leader":"true"},
              "core_node4":{
                "state":"active",
                "core":"collection1",
                "node_name":"127.0.1.1:7500_solr",
                "base_url":"http://127.0.1.1:7500/solr"}}}},
        "maxShardsPerNode":"1",
        "router":{"name":"compositeId"},
        "replicationFactor":"1",
        "znodeVersion": 11,
        "autoCreated":"true",
        "configName" : "my_config",
        "aliases":["both_collections"]
      },
      "collection2":{
        "..."
      }
    },
    "aliases":{ "both_collections":"collection1,collection2" },
    "roles":{
      "overseer":[
        "127.0.1.1:8983_solr",
        "127.0.1.1:7574_solr"]
    },
    "live_nodes":[
      "127.0.1.1:7574_solr",
      "127.0.1.1:7500_solr",
      "127.0.1.1:8983_solr",
      "127.0.1.1:8900_solr"]
  }
}
```
