# Cloud界面

在 SolrCloud 模式下运行时，“Cloud” 选项将会出现在 Logging 和 Collections / Core Admin之间的管理界面。  

此 “Cloud” 界面提供了有关集群中每个Collections和Core的状态信息，以及对存储在ZooKeeper中的低级别数据的访问。  

>Tip：Cloud 界面只有在使用 SolrCloud 时才是可见的： “Cloud” 菜单选项仅在以 SolrCloud 模式运行的 Solr 实例上可用。Solr 的单节点或主/从复制实例不会显示此选项


点击左侧导航栏中的 Cloud 选项，会出现一个小的子菜单，其中有 “Tree”、“Graph”、“Graph（Radial）” 和 “Dump” 选项。默认视图（“Graph”）显示每个集合的图形、组成这些集合的分片以及每个分片的每个副本的地址。  
  
此示例显示使用 bin/solr -e cloud -noprompt 示例命令创建的非常简单的双节点群集。除了2个碎片、2个副本 “gettingstarted” 集合之外，还有一个额外的 “films” 集合，由单个碎片/副本组成：  
![solr cloud-graph](http://lucene.apache.org/solr/guide/7_0/images/cloud-screens/cloud-graph.png)  
“Graph（Radial）” 选项为每个节点提供了不同的可视视图。使用相同的示例集群，径向图形视图如下所示： 
![solr cloud-radial](http://lucene.apache.org/solr/guide/7_0/images/cloud-screens/cloud-radial.png) 

“Tree” 选项显示了 ZooKeeper 中数据的目录结构，包括关于 live_nodes 和 overseer 状态的集群信息，以及 state.json、当前分片前导集和所使用的配置文件等的集合特定信息。在这个例子中，我们看到为 “films” 集合定义的 state.json 文件：  
![solr cloud-tree](http://lucene.apache.org/solr/guide/7_0/images/cloud-screens/cloud-tree.png)  
最后的选项是 “Dump”，它返回一个包含所有节点、它们的内容和子节点（递归）的 JSON 文档。这可以用来导出 Solr 保存在 ZooKeeper 中的所有数据的快照，并且可以帮助调试 SolrCloud 问题。  
