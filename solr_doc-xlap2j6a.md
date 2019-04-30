## SolrCloud监督状态和统计：OVERSEERSTATUS 
<div class="content-intro view-box ">返回监督器（overseer）的当前状态，各种监督器API的性能统计信息以及每种操作类型的最近10次故障。   
/admin/collections?action=OVERSEERSTATUS  

### 使用OVERSEERSTATUS的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-overseerstatus"/>

在这个例子中输入：  
```
http://localhost:8983/solr/admin/collections?action=OVERSEERSTATUS
```
得到的输出为：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":33},
  "leader":"127.0.1.1:8983_solr",
  "overseer_queue_size":0,
  "overseer_work_queue_size":0,
  "overseer_collection_queue_size":2,
  "overseer_operations":[
    "createcollection",{
      "requests":2,
      "errors":0,
      "avgRequestsPerSecond":0.7467088842794136,
      "5minRateRequestsPerSecond":7.525069023276674,
      "15minRateRequestsPerSecond":10.271274280947182,
      "avgTimePerRequest":0.5050685,
      "medianRequestTime":0.5050685,
      "75thPcRequestTime":0.519016,
      "95thPcRequestTime":0.519016,
      "99thPcRequestTime":0.519016,
      "999thPcRequestTime":0.519016},
    "removeshard",{
      "..."
  }],
  "collection_operations":[
    "splitshard",{
      "requests":1,
      "errors":1,
      "recent_failures":[{
          "request":{
            "operation":"splitshard",
            "shard":"shard2",
            "collection":"example1"},
          "response":[
            "Operation splitshard caused exception:","org.apache.solr.common.SolrException:org.apache.solr.common.SolrException: No shard with the specified name exists: shard2",
            "exception",{
              "msg":"No shard with the specified name exists: shard2",
              "rspCode":400}]}],
      "avgRequestsPerSecond":0.8198143044809885,
      "5minRateRequestsPerSecond":8.043840552427673,
      "15minRateRequestsPerSecond":10.502079828515368,
      "avgTimePerRequest":2952.7164175,
      "medianRequestTime":2952.7164175000003,
      "75thPcRequestTime":5904.384052,
      "95thPcRequestTime":5904.384052,
      "99thPcRequestTime":5904.384052,
      "999thPcRequestTime":5904.384052},
    "..."
  ],
  "overseer_queue":[
    "..."
  ],
  "..."
 }
```
