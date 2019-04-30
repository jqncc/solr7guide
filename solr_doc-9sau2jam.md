## FORCELEADER：强制碎片leader 
<div class="content-intro view-box ">
## FORCELEADER<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#forceleader"/>
在不太可能发生的情况下，如果碎片失去leader，则可以调用这个FORCELEADER命令来强制选取新的leader。  
  
/admin/collections?action=FORCELEADER&amp;collection=&lt;collectionName&gt;&amp;shard=&lt;shardName&gt;  
### FORCELEADER参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#forceleader-parameters"/>

**collection**
集合的名称。该参数是必需的。  
**shard**
leader选取应该发生的碎片的名称。该参数是必需的。  

这是一个专业级别的命令，只有当常规的leader选取不起作用时才应该调用。如果新leader没有某些更新，可能会导致数据丢失，可能是最近的，这是在不使用旧的leader在前所承认的。  
