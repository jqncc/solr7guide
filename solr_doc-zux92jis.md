## CDCR设置ZooKeeper与升级 
<div class="content-intro view-box ">
## ZooKeeper设置<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#zookeeper-settings"/>

使用CDCR，目标ZooKeepers将具有来自Target cloud和Source cloud的连接。您可能需要在zoo.cfg中增加maxClientCnxns设置：
      
  
```
## set numbers of connection to 200 from client
## is maxClientCnxns=0 that means no limit
maxClientCnxns=800
```

## CDCR升级和修补生产<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#upgrading-and-patching-production"/>

在升级到索引器或应用程序时，应关闭Source（生产）和Target（DR）。根据您的设置，您可能需要暂停/停止索引。部署发行版或修补程序以及可重建索引。然后启动Target（DR）。
      
  

    - 无需重新发出DISABLEBUFFERS或START命令。这些都是持续的。
    - 启动Target后，运行一个简单的测试。将测试文档添加到每个Source cloud。然后在目标上检查它。
```
#send to the Source
curl http://&lt;Source&gt;/solr/cloud1/update -H 'Content-type:application/json' -d '[{"SKU":"ABC"}]'
#check the Target
curl "http://&lt;Target&gt;:8983/solr/&lt;collection_name&gt;/select?q=SKU:ABC&amp;indent=true"
```

