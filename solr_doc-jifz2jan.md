## MIGRATESTATEFORMAT：迁移群集状态 
<div class="content-intro view-box ">
## MIGRATESTATEFORMAT
专业的实用程序API，用来将集合从共享的clusterstate.json zookeeper节点（在所有的Solr默认5.0以前的版本，创建stateFormat=1）到存储在ZooKeeper （使用stateFormat = 2创建，当前默认设置）的每个集合state.json，无需任何应用程序的停机时间。  
  
/admin/collections?action=MIGRATESTATEFORMAT&amp;collection=&lt;collection_name&gt;  
### MIGRATESTATEFORMAT参数

**collection**
要从<code>clusterstate.json</code>迁移到其自身<code>state.json</code>ZooKeeper节点的集合的名称。该参数是必需的。  
**async**
请求ID来跟踪这个将被异步处理的动作。  

此API在将Solr 5.0之前创建的任何集合迁移到默认情况下现在使用的更具可伸缩性的集群状态格式中非常有用。如果在任何Solr 5.x版本或更高版本中创建集合，则不需要执行此命令。  
