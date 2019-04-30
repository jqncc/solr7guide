## 使用ReplicationHandler设置Repeater 
<div class="content-intro view-box ">
## 使用ReplicationHandler设置Repeater
一个master可能只能服务那么多的slave而不影响性能。一些组织已经在多个数据中心部署了从属（slave）服务器。如果每个从属都从远程数据中心下载索引，则由此产生的下载可能会消耗太多的网络带宽。为避免这种情况下的性能下降，可以将一个或多个从属配置为中继器。中继器只是一个既充当主机又充当从机的节点。  
- 要将服务器配置为中继器，solrconfig. xml 文件中的复制 requestHandler 的定义必须包括用于主控形状和从属项的文件列表。
- 即使replicateAfter将主参数设置为优化，也要确保将replicateAfter参数设置为提交。这是因为在中继器（或任何从属）上，只有在下载索引后才会调用提交。优化命令永远不会在从属上被调用。
- 可选地，可以配置中继器通过压缩参数从主服务器获取压缩文件以减少索引下载时间。
以下是中继器的ReplicationHandler配置示例：  
```
&lt;requestHandler name="/replication" class="solr.ReplicationHandler"&gt;
  &lt;lst name="master"&gt;
    &lt;str name="replicateAfter"&gt;commit&lt;/str&gt;
    &lt;str name="confFiles"&gt;schema.xml,stopwords.txt,synonyms.txt&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="slave"&gt;
    &lt;str name="masterUrl"&gt;http://master.solr.company.com:8983/solr/core_name/replication&lt;/str&gt;
    &lt;str name="pollInterval"&gt;00:00:60&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
