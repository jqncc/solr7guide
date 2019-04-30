## SolrCloud 
<div class="content-intro view-box ">Apache Solr包括设置结合了容错和高可用性的Solr服务器集群的能力。这些称为SolrCloud的功能提供分布式索引和搜索功能，支持以下功能：  
  

    - 整个集群的中心配置
    - 自动负载平衡和故障切换的查询
    - ZooKeeper集成，用于集群协调和配置

SolrCloud是灵活的分布式搜索和索引，没有主节点分配节点、分片和副本。相反，Solr使用ZooKeeper来管理这些位置，具体取决于配置文件和架构。查询和更新可以发送到任何服务器。Solr将使用ZooKeeper数据库中的信息来确定哪些服务器需要处理请求。  
在本节中，我们将介绍在SolrCloud模式下使用Solr时需要了解的一切。我们已经将细节分成以下主题：  

    - SolrCloud入门
    
    - SolrCloud的工作原理SolrCloud中的碎片和索引数据
        分布式请求
    
    - SolrCloud韧性SolrCloud恢复和写入容差
        SolrCloud查询路由和读取容差
    
    - SolrCloud配置和参数设置一个外部ZooKeeper集合
        使用ZooKeeper管理配置文件ZooKeeper访问控制集合API参数引用
            命令行实用程序SolrCloud与传统的配置文件ConfigSets API
    
    - 基于规则的副本放置
    
    - 跨数据中心复制（CDCR）
    
    - SolrCloud Autoscaling
