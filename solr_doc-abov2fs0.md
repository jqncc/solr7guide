## IndexUpgrader工具 
<div class="content-intro view-box ">Lucene 发行版包含一个 IndexUpgrader 工具，它可以将以前 Lucene 版本的索引升级到当前的文件格式。  
该工具可以从命令行使用，也可以在 Java 中实例化和执行。  
在 Solr 发行版中，Lucene 文件位于 ./server/solr-webapp/webapp/WEB-INF/lib。运行该工具时，需要在类路径中包含 lucene-core-&lt;version&gt;.jar 和 lucene-backwards-codecs-&lt;version&gt;.jar。  
```
java -cp lucene-core-6.0.0.jar:lucene-backward-codecs-6.0.0.jar org.apache.lucene.index.IndexUpgrader [-delete-prior-commits] [-verbose] /path/to/index
```
这个工具只保留索引中的最后一个 commit。由于这个原因，如果传入的索引有多个提交，工具默认拒绝运行。指定 -delete-prior-commits 以重写此操作，允许该工具删除除了最后一个提交之外的所有操作。  
升级大型索引可能需要很长时间。作为一个经验法则，升级过程大约是每分钟1GB。  
```
注意：如果索引在执行之前部分升级（例如添加了文档），则该工具可能会对文档重新排序。如果您的应用程序依赖于文档 ID 的单调性(这意味着文档将被添加到索引中的顺序被保留)，那么请改为使用完整的 forceMerge。
```
