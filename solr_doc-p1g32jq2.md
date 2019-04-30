## Solr合并索引 
<div class="content-intro view-box ">
## 合并索引
如果您需要合并来自两个不同项目的索引或以前在分布式配置中使用的多个服务器，则可以使用包含在lucene-misc或CoreAdminHandler中的IndexMergeTool 。  
  
要合并索引，它们必须满足以下要求：  
- 这两个索引必须兼容：它们的架构应该包含相同的字段，并且它们应该以相同的方式分析字段。
- 索引不得包含重复的数据。
理想情况下，两个索引应该使用相同的架构来构建。  
## 使用IndexMergeTool<a href="http://lucene.apache.org/solr/guide/7_0/merging-indexes.html#using-indexmergetool"/>
要合并索引，请执行以下操作：  
  
1 <li>确保您要合并的两个索引都已关闭。</li>2 <li>将这个新目录复制到应用程序的solr索引的位置（当然，先将旧目录移到一边），然后启动Solr。</li>
## 使用CoreAdmin<a href="http://lucene.apache.org/solr/guide/7_0/merging-indexes.html#using-coreadmin"/>
CoreAdminHandler的MERGEINDEXES命令可以用来将索引合并到一个新的核心中 - 可以从一个或多个任意的indexDir目录中合并，也可以通过一个或多个现有的 srcCore 核心名称进行合并。  
  
有关详细信息，请参阅CoreAdminHandler部分。  
