## SolrCloud查询路由和读取容错 
SolrCloud在读取和写入中具有高度的可用性和容错性。  
  
## 读取侧容错<a href="http://lucene.apache.org/solr/guide/7_0/solrcloud-query-routing-and-read-tolerance.html#read-side-fault-tolerance"/>
在SolrCloud集群中，每个单独的节点负载平衡在集合中的所有副本之间平衡读取请求。您仍然需要一个与集群交谈的“外部”负载平衡器，或者您需要一个智能客户端，了解如何读取ZooKeeper中的Solr元数据并与其交互，并只请求ZooKeeper集合的地址开始发现它应该到达哪个节点发送请求。（Solr提供了一个称为CloudSolrClient的智能Java SolrJ客户端。）  
  
即使在集群中的某些节点处于离线状态或无法访问，Solr 节点也能够正确响应搜索请求，只要它可以与每一个碎片的至少一个副本进行通信，或每个相关碎片的一个副本，如果用户碎片通过shards或_route_参数限制搜索。每个分片越多的副本，Solr集群就越有可能在发生节点故障时处理搜索结果。  
### zkConnected<a href="http://lucene.apache.org/solr/guide/7_0/solrcloud-query-routing-and-read-tolerance.html#zkconnected"/>
只要Solr节点能够与所知道的每个碎片的至少一个副本进行通信，就会返回搜索请求的结果，即使它在收到请求时不能与ZooKeeper进行通信。从容错的角度来看，这通常是首选的行为，但如果对该节点尚未通过ZooKeeper通知的集合结构有重大更改，则可能会导致过时或不正确的结果（即碎片可能已被添加或删除，或分割成分片）  
  
每个搜索响应中都包含一个zkConnected标题，指示处理请求的节点是否与ZooKeeper在以下时间连接：  
Solr响应与部分结果  
```
{
  "responseHeader": {
    "status": 0,
    "zkConnected": true,
    "QTime": 20,
    "params": {
      "q": "*:*"
    }
  },
  "response": {
    "numFound": 107,
    "start": 0,
    "docs": [ "..." ]
  }
}
```
### shards.tolerant<a href="http://lucene.apache.org/solr/guide/7_0/solrcloud-query-routing-and-read-tolerance.html#shards-tolerant"/>
如果查询的一个或多个碎片完全不可用，则Solr的默认行为是使请求失败。但是，有很多用例可以接受部分结果，所以Solr提供了一个布尔shards.tolerant参数（默认值false）。  
  
如果shards.tolerant=true，那么部分结果可能会被返回。如果返回的响应不包含所有合适的碎片的结果，那么响应头部包含一个称为partialResults的特殊的调用标志。  
客户端可以指定shards.info以及shards.tolerant参数以检索更多 fine-grained 的详细信息。  
将partialResults标志设置为“true”的示例响应：  
Solr响应与部分结果：  
```
{
  "responseHeader": {
    "status": 0,
    "zkConnected": true,
    "partialResults": true,
    "QTime": 20,
    "params": {
      "q": "*:*"
    }
  },
  "response": {
    "numFound": 77,
    "start": 0,
    "docs": [ "..." ]
  }
}
```
